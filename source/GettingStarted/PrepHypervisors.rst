Prepping Hypervisors
======================

With Fetch hypervisor inventories are not dynamically generated. While technically possible to have the inventory of hypervisors generated dynamically we find it best to a manually process. Additionally we find it best practice to use password authentication for hypervisors.

.. Warning::

   With configuration management things happen FAST! It is too easy to make bad things happen quickly. With manual inventories it is very clear what machines will be affected. The rational for password authentication is that if Ansible starts prompting for a password it forces user intervention and pause. 

Creating hypervisor inventory
------------------------------

In the inventory directory of Fetch create a file called ``hypervisors``.

.. Note::
   If Ansible is given a directory for the inventory ``-i`` argument it will execute everything in that directory in search of hosts to use.


.. code-block:: YAML

   [hypervisors]
   10.0.0.20 ansible_connection=ssh  ansible_ssh_user=root ansible_python_interpreter=/opt/local/bin/python2.7
   10.0.0.21 ansible_connection=ssh  ansible_ssh_user=root ansible_python_interpreter=/opt/local/bin/python2.7
   10.0.0.22 ansible_connection=ssh  ansible_ssh_user=root ansible_python_interpreter=/opt/local/bin/python2.7
   10.0.0.23 ansible_connection=ssh  ansible_ssh_user=root ansible_python_interpreter=/opt/local/bin/python2.7
    
   [hold]
   10.0.0.24 ansible_connection=ssh  ansible_ssh_user=root ansible_python_interpreter=/opt/local/bin/python2.7

There are a few things to note about this file.
   - Hosts can be grouped. 
   - Connection paramaters can be specified in this file (rather than when running Ansible)

.. Note::
   You will notice a Python interpreter is specified in the inventory file. By default SmartOS global zone does not include a python interpreter. Don't wory about this, as we will deploy python using "raw" commands


Use inventory groups to your advantage. If you have a few host that you dont want to deploy to make a new group.

Now that we have our inventory created we can run a quick test to see if everything works. 

.. code-block:: bash

   ansible -i inventory/hypervisors hypervisors -m raw -a "/usr/bin/echo ok" --ask-pass


We are specifing a couple of things:

-i           The location of the inventory file
-m raw       The method to use
-a           The actaul command to run
--ask-pass   Becasue we are not using key auth we need to specify a password

.. code-block:: bash

   10.0.0.20 | success | rc=0 >>
   ok

   10.0.0.21 | success | rc=0 >>
   ok
   
   10.0.0.22 | success | rc=0 >>
   ok
   
   10.0.0.23 | success | rc=0 >>
   ok

**Good news!** Its that easy! You just ran a commmand on all of you hosts and saw the response. Of course we want to do a whole lot more than that, so lets take a look a roles


.. toctree::
   :maxdepth: 2