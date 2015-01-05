A Starring Role
=================

So you want to create your very first role? 
"""""""""""""""""""""""""""""""""""""""""""""
Thats great news! We are going to start out by creating a role to manage the Chunter installation state on our hypervisors. 

The end goal of our role is to:

.. code-block:: guess

   1. Check what version of Chunter is installed.
   2. If the wrong verion is installed: 
       a. Download and unpack the correct version.
       b. Modify the installation for unattendend install
       c. Run the install
   3. Ensure the settings are correct.
   4. Ensure the required services are enabled.


This section will introduce a couple new concepts.

- Playbook code management
- Role Templating
- Varaiable Files
- Conditional Execution
- File Templates
- Handlers
- Notifications


Code Management
""""""""""""""""""""

.. note::
	It is only a recomendation to to add your new role to source code management. If you do not want to, you can safely skip this paragraph.


It is generally best practice to keep all playbooks in source code management. This allows for easy change tracking and (hopefully never) roll backs. We find it very convenient to keep one git repository per role rather than one large repo for everything. This allows allows for easy grabbing of external repos (like from here!). As an added benifit you can easily update the *Fetch* repo to get updated utilities. So how do you do this?


	#. Create a git repo **NOT** in the Fetch directory. The easiest way to do this is just create a new repo on Github (but this is certianlly not the only place)
	#. Create a file in the repo. We usually just create a ``LICENSE.md`` file.
	#. On your local machine navigate to Fetch/roles
	#. Add your newly created repo as a submodule ``git submodule add git@github.com:{{owner}}/{{repo-name}}.git``

.. warning::
   It is **CRITICAL** that if you are using Github or some other public repo hosting service that you add the correct ignore settings so that you do not in advertently upload confidential information. Please take the time to read about ``.gitignore`` files. 

The Fetch Template Tool  (Lights)
"""""""""""""""""""""""""""""""""""
Fetch includes a templating tool for rapidly scaffolding a new role. The tool simply creates a directory and a few empty files. To use the tool:

	#. On your local machine naviage to ``Fetch``.
	#. Run template with the name of your new role: ``./template fifo-chunter``

Cameras
"""""""""""
Now that we have our Role templeted out we can start adding some lines!

Lets start in the ``tasks/main.yml`` file of our newly created role. A proper yaml file starts with ``----`` so go ahead and add that.

The first thing we need to do is check what version (if any) of Chunter is installed. Luckly for us Chunter has a file that contains one thing: the version number! Lets grab the contents of that file and assign it to a variable. We should also ignore any errors that might occur (becasue the file might not exist). 

.. code-block:: yaml

   - name: Lookup version of chunter
     shell: awk {'gsub(",",""); print $1'} /opt/chunter/etc/chunter.version
     register: chunter_version_installed
     changed_when: chunter_version_installed.stdout != '{{chunter_version_required}}'
     ignore_errors: True

We now have a variable ``chunter_version_installed`` that contains the contents of the Chunter version file (or the parts of the file we care about). With this variable we can use condition execution to decide if we should install Chunter or not. Ansible uses the ``when`` statement to decide if it should run a particular command. Unfortunately the installation of Chunter is not a single step process, so we will have to include the when statement multiple times.

**But wait!** Ansible has the ability to include one file inside of another. This will allow us to create a install task file and conditionally include it. This has the side effect of keeping our ``main.yml`` nice and clean and keeping all install taks in a dedicated file. 

To get started on the install process lets create a file called ``install.yml`` in the task directory of our role. Again a proper yaml file starts with ``----`` so add that. 

After you have created and prepped the ``install.yaml`` we can add a couple of task that download and extract Chunter.

.. code-block:: yaml

   - name: download latest chunter
     get_url: url=http://release.project-fifo.net/chunter/{{chunter_release}}/chunter-latest.gz dest=/opt/chunter-latest.gz

   - name: unpack chunter
     shell: gunzip -f /opt/chunter-latest.gz

TODO: MUST LINK BUILD # TO FILES IN CHUNTER DEPLOYMENT SERVER ^^ DOES NTO WORK RIGHT NOW ^^

Not so bad, right? The one thing you may notice is that in the get_url task there is a variable ``{{chunter_release}}`` that we have not yet created. Don't worry about it we will come back to that shortly.

The next thing we need to do is prep the Chunter install file for unattended installation.

.. code-block:: yaml

   - name: remove prompts
     shell: sed -i.orig '/read/d' /opt/chunter-latest 

   - name: remove exits
     shell: sed -i.bak -e '/exit[[:space:]]1/d' /opt/chunter-latest 

.. warning::
   Prompts and conditional exits where put into the installation script for a reason! We run an old version of SmartOS that is not tested with Fifo. If you are going to do something like this you should test every release before putting into production. *Modifing the installation script like this is 100% unsupported by Project-Fifo.*

Lastly we need to actually install Chunter.

.. code-block:: yaml

   - name: install chunter
     shell: sh /opt/chunter-latest -f
     notify: Bounce Chunter


.. note::
   If you include the *force* command (``-f``) with the chunter install command you likely do not need to modify the install script. Obviously test, test, test.

Eagle eyed readers will notice a new line that we have not seen before: ``notify: Bounce Chunter``. This line is informing Ansible that we will need to run the *Bounce Chutner* handler when the rest of the playbook is complete. We will create this handler shortly.

Now that we have the installation tasks complete we can hop back to the ``main.yml`` file. The next lines that we add in ``main.yml`` should include the installation file we created, but only if the node is not running the correct version of Chunter.

.. code-block:: yaml

   - include: install.yml
     when: chunter_version_installed.stdout != '{{chunter_version_required}}'

Again we are using a previously unmentioned variable (``{{chunter_version_required}}``). We will define that in a moment.

The last thing we want to do in our tasks file is make sure that the cookie that Chunter uses for communications matches the cookie that we use for the rest of our Project-Fifo core services, and that the necessary services are enabled. 

.. code-block:: yaml

   - name: set cookie
     lineinfile: dest=/opt/chunter/etc/chunter.conf regexp='^distributed_cookie' line='distributed_cookie = {{chunter_cookie}}'
     notify: Bounce Chunter

   - name: Enable epmd
     service: name=epmd state=started enabled=yes

   - name: Enable Chunter
     service: name=chunter state=started enabled=yes

There is another unmnetioned variable (``{{chunter_cookie}}``), same story as the last time. Again you will notice we are notifying ``Bounce Chunter``. Ansible will only notify the bounce task *if* the was a change that resulted from the ``set cookie`` task. 

.. note::
   You will notice the service task has both ``state=started`` and ``enabled=yes``. If ``enabled`` is not specified the service will only be started but not enabled. As a consequence if the host is rebooted the service will not auto start.

Remember all those variables that we kept using with out defining? Lets open the ``vars/main.yml`` and create those now.

.. code-block:: yaml

   ---
   chunter_version_required: "dev-41502a3"
   chunter_release: "dev"
   chunter_cookie: "COOKIE"

Lastly in the installation we notified a handler that has not yet been defined, so lets open up ``handlers/main.yml`` and define that.

.. code-block:: yaml

   ---
   - name: Bounce Chunter
     service: name=chunter state=restarted enabled=yes

Why do we need to restart Chunter on a fresh install? Well on a fresh install this task is not necessary but keep in mind the install runs if the version doesnt match, so if Chunter is upgraded we do want to restart. Also if the cookie changes we would want to restart Chunter.



Action
""""""""
Now that we are done creating all the necessary files we can run ansible-playbook 
apply the role to our hosts.

.. code-block:: bash

   ansible-playbook -i inventory/hypervisors hypervisors.yml --ask-pass

Hmmmm, something doesn't seem right! You will notice that all the tasks we just created did not actually run. Why is that? If you look at the above command you will notice that we are calling the playbook ``hypervisors.yml``. Open up ``Fetch/hypervisors.yaml`` and add the line ``- fifo-chunter`` to the roles list. This lets Ansible know that both the role ``hypervisor`` and ``fifo-chunter`` should be applied to our ``hypervisors`` inventory group. Run your ansible command again. 

.. code-block:: bash

   ansible-playbook -i inventory/hypervisors hypervisors.yml --ask-pass

This time you should see output indicating that chunter was deployed to your hosts. 


An Encore?
""""""""""""""""
If you are looking to hone your skills here are some ideas for improvements you can make to this role:
 
   - Configure Chunter to use Watchdog
   - Have Ansible send you a text when it updates or installs Chunter on a host.


tl;dr
""""""""
Look HERE (TODO) for the playbook that we created. 


Whats Next?
""""""""""""""
**I'm a star, I'm on top, somebody bring me some ham!** 

Next we look at how Project-Fifo can help us deploy roles quickly to virtual machines - Link (TODO)




.. toctree::
   :maxdepth: 2

   