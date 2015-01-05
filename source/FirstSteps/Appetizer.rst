An Appetizer
=================

Now that we have a working cloud orchistrator we can start deploying virtual machines. There are a couple of things you will need first:

- If you haven't already installed the Project-Fifo cli tool on your local machine take a second to do that now. LINK
- You will also need networks, and packages (a definition of vm size), and at least one dataset (the latest base64 is a good choice) loaded into Fifo. If you are looking for some package deffinitions to get you up and running quickly look here (TODO)

One of the great features of Project-Fifo is that every entity (hypervisor, vm, network, package, ect) can have unlimited metadata attached to it. This allows us to extend the record model how ever we want. A simple example is 

.. code-block:: bash

   fifo vms metadata {{VM_UUID}} set keyName "value here"
     RESPONCE
   fifo vms metadata {{VM_UUID}} get keyName
     RESPONCE
   fifo vms metadata {{VM_UUID}} delete keyName
     RESPONCE
   fifo vms metadata {{VM_UUID}} get keyName
     RESPONCE

As you can see we created a meta record for a VM, read it back, deleated it, and tried to read it again to prove that it is really gone. 

Up until now we have been using static inventory files for Ansible. Obvioulsy that is a pain to manage for things that change dyamically like VMs. Included in the Fetch/inventory directory is a file name ``fifo-inventory``. This file will dynamically create a inventory based on vm metadata. We have decided to use the key ``scm.groups.{{GROUP_NAME}}.enabled`` for groups that the vm is part of. Additionally we use the key ``scm.hostvars`` to set any required variables needed for Ansible (mainly python location). Of course we wanted to make it a bit easier to use than having to type long commands so we have included three more tools with Fetch:

scm_prep
"""""""""""""""""
Adds default hostvars to the vm. This should be ran once (running more than once will not hurt) per vm that will be under SCM. Usage:

.. code-block:: bash

   scm_zone_prep 7b58dea3-1834-4259-9e1b-cb18d5dd9206


scm_add
"""""""""""""""""
Adds a VM to a group. This commmand can be used multiple times to add a VM to multiple groups. Usage:

.. code-block:: bash

   scm_add 7b58dea3-1834-4259-9e1b-cb18d5dd9206 pingable
   scm_add 7b58dea3-1834-4259-9e1b-cb18d5dd9206 webhosts
   

scm_remove
"""""""""""""""""
Removes a VM from a group. This commmand can be used multiple times to remove a VM from multiple groups. Usage:

.. code-block:: bash

   scm_remove 7b58dea3-1834-4259-9e1b-cb18d5dd9206 web-hosts


.. note:: 
	
	"SCM" stands for "Software Configuration Management"


Configurations at the Speed of Light
""""""""""""""""""""""""""""""""""""""
To see how this all works head over to your ``Fetch/roles`` directory. Lets add a sample role using the git submodule method

.. code-block:: bash

   git submodule add git@github.com:Go-Fetch/nginx-php.git

Unlike with the hypervisors we don't want to add this to the hypervisor playbook, so lets make a new file called ``webhosts.yml`` in the base directory of ``Fetch``. Add the following lines to your newly created file:
 
 .. code-block:: yaml

    - hosts: webhosts
      roles:
         - nginx-php


Now that your playbook and role are ready to go hop over to your Project-Fifo web interface and create a few VMs using the base-64 package. Pay attention to the UUID of each machine (TODO: ADD IMAGE) as we will need it in a second. 

Once you have created your VMs lets scm_prep them and add them to our groups. Run the following command for each VM you created.

.. code-block:: bash

   scm_zone_prep {{UUID}}
   scm_add {{UUID}} webhosts


We are now ready to rapidly configure machines!

.. code-block:: bash

   ansible-playbook -i inventory/fifo-inventory webhosts.yml --ask-pass

There are a couple of things to notice about this command. 
    
    * For the inventory file we are using ``inventory/fifo-inventory``
    * We have specified the webhosts playbook
    * Unlike last time we did not use the ``--ask-pass`` argument. This is because we assume you have setup key based authentication using Fifo.

.. warning::

   Fetch dynamic inventory uses your cached credentials of PyFi (the Project-Fifo local CLI tool). It is best practice to not use an account that has login rights to every vm. With configuration management things happen FAST! It is best to not allow a typo destory everything at once. Setting up multiple user accounts is best practice. That way when things go wrong you minimize dammage. Obviously this does slow down how fast you can deploy changes to your entire orginization. Find a balance, and stick to it.

Once we have run that command and it has completed you should notice that each host is now listening on port 80. Cool! Want to add a couple more host to your pool of webservers? Easy! Just spin up a few more VMs, use ``scm_add`` and run the same ``ansible-playbook`` again. It will everify that every host in the group is in the requested stat. As an added benifit it will bring any host that has creeped away from its original configuration back to its proper state. This role does not modify any files other than Nginx and PHP configuration files, so you can run it and feel safe knowing it will not mess with your ``htdocs`` directory.



Homework
""""""""""""""""
Want to improve your skills? Here are some ideas for improvements you can make to this role:
 
   - Create another role that deploys a website to the ``htdocs`` directory. Creat a new group on only a few VMs and deploy that role to the select few.


tl;dr
""""""""
From your ``Fetch`` base directory:

.. code-block:: bash

	cd roles
	git submodule add git@github.com:Go-Fetch/nginx-php.git
	cd ..

	cat >webhosts.yml <<EOL
	- hosts: webhosts
	      roles:
	         - nginx-php
	EOL

	PACKAGE="small" # Or whatever the name of your package is
	DATASET="base64-1.9.1" # Make sure this dataset is installed in your Fifo cloud
	NET="7df94bc3-6a9f-4c88-8f80-7a8f4086b79d"  # UUID of the network VMs should be palced on (hint: fifo networks list)

	for i in 1 2 3
	do
	  cat <<EOF | fifo vms create -p $PACKAGE -d $DATASET
	  {
	    "alias": "webhost${i}",
	    "networks": {"net0": "$NET"}
	  }
	EOF
	  UUID=`fifo vms get webhost$i`
	  scm_zone_prep $UUID
	  scm_add $UUID webhosts
	done

	sleep 30 # Give the vms time to spin up.

	ansible-playbook -i inventory/fifo-inventory webhosts.yml --ask-pass


You just created 3 virtual machines, added the webhosts role, and deployed Ngninx & PHP-FPM to them.


Whats Next?
""""""""""""""


Making a PaaS using clusters - Link (TODO)





.. toctree::
   :maxdepth: 2

   