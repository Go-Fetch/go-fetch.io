Tip and Tricks
===============

Here are a couple of tips and tricks that we have come across. No particular order.


Ignore Host Key
"""""""""""""""""""""""
If you have issues with IP addresses being reused for new machines you can run (on your local machine):

 .. code-block:: bash

   export ANSIBLE_HOST_KEY_CHECKING=False



Add range of host to inventory file
""""""""""""""""""""""""""""""""""""""

Inventory files allow for ranges which allow us to refine the inventory file a bit.

 .. code-block:: python

    [eveything]
    a.loadbalancer.test.com
    b.loadbalancer.test.com
    c.loadbalancer.test.com
    webserver1.test.com
    webserver2.test.com
    database1.test.com
    database2.test.com
    database3.test.com

Is the same as:

 .. code-block:: python
    
    [a:c].loadbalancer.test.com
    webserver[1-2].test.com
    database[1-3].test.com

Inventory Groups Can Contain Other Groups
"""""""""""""""""""""""""""""""""""""""""""

 .. code-block:: python

    [proxies]
    [a:c].loadbalancer.test.com

    [webhosts]
    webserver[1-2].test.com

    [database]
    database[1-3].test.com

    [everything:children]
    proxies
    webhosts
    database


  .. toctree::
   :maxdepth: 2