Roles
========

Roles are the foundation for large scale Ansible based deployments. Each host can be part of one or more roles. Roles are a collection of tasks, files, handlers, and variables.

The Hypervisor Role
---------------------

The hypervisor role included with Fetch does the following:

  - Deploy Python
  - Check for Shell Shock vulnerability (and fix it if there is one)

Obviously you will want to do more on each hypervisor, but we like each role to stay minimal. As time goes on we will introduce other roles that can be used on the hypervisors.

 .. Note:: 
    If you have not made an inventory file for your hypervisors you will need to do that. :doc:`PrepHypervisors`

To run the hypervisor role playbook all we have to do is run:

.. code-block:: bash

   ansible-playbook -i inventory/hypervisors hypervisors.yml --ask-pass

Explain this command to me 
"""""""""""""""""""""""""""""

We are telling Ansible that we are going to give it a playbook ``ansible-playbook``. Just like when we where setting up the hypervisor inventory we pass the in inventory file we want to use ``-i inventory/hypervisors``. We point Ansible to the playbook we want to use ``hypervisors.yml`` (located in the root of Fetch). Lastly because these are hypervisors (and we use password auth on hypervisors) we tell Ansible that it will need to prompt for the ssh password ``--ask-pass``.

Great, but what does it do?
""""""""""""""""""""""""""""""

Ansible looks for the playbook and opens it up only to find a couple simple lines

.. code-block:: yaml

    - hosts: hypervisors
      gather_facts: no
      roles:
       - hypervisor

This file is simply saying that we are going to take the hosts in the *hypervisors* group and apply the hypervisor role to them.

.. Note::
   You will notice that there is an additional line ``gather_facts: no``. Typically this would not be included as it provides some useful info for making decisions, but because the global zone does not have python (which is used for fact collection) it will cause the task to fail before it has a chance to deploy.


Ok, so about this role thing
""""""""""""""""""""""""""""""

When a role is referenced Ansible simply looks in the roles/{role_name} directory for instructions. 

Looking at the hypervisor role directory we can see there are a couple of directories but only two have anything in them. 


   :tasks: In this directory we place execution instructions
   :files: The directory where payloads are placed

Inside of the *tasks* directory this is a single file ``main.yml``. This is the default task file. Ansible will run this against any host it is directed to. Because our top level ``hypervisors.yml`` specified ``hosts: hypervisors`` Only host in the hypervisor group will get the instructions in ``main.yml``.

The file ``main.yaml`` is pretty well commented so go ahead and `open it up to see what it's all about <https://TODO>`_.

**Good Job! That wasnt so bad! Lets make our own role...** :doc:`../FirstRole/index`

.. toctree::
   :maxdepth: 2