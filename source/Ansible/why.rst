Why Ansible?
===============

We chose Ansible for three simple reasons:

  - **Agentless** We wanted a configuration managment system that did not need any special agents installed on our servers. Sure others can deploy agents as part of a first step, but it fits our model of thinking better that a special tool should not have to be deployed, and if we stop using the tool it doesnt leave any garbage behind.

  - **No Central Servers** Things fail. We don't want our configuration management to rely on a failure point. Again this fits our model of thinking we would rather push instructions to our servers rather than having them pull them from a central point. 

  - **Simple Recipies** This is likely more subjective, but to us Ansible recipies are easier to read. We can make a play, and then come back to in months later and still understand what it is doing. Never underestimate how imporant 2am comprehensibility is!

  .. toctree::
   :maxdepth: 2