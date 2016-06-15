======================================================
Step-by-step guide to set up Trellis on DreamCompute
======================================================

Trellis
~~~~~~~~~~~~

In this tutorial we are going to use `Trellis
<https://roots.io/trellis/>`_
to install a very sold modern LEMP stack on DreamCompute. This stack is great to run WordPress websites and works well with `Bedrock
<https://roots.io/bedrock/>`_
, the modern WordPress stack. Trellis is a set of Ansible playbooks that help you setup a full local, staging and development environment for your project. Locally it works with an automated Vagrant setup and remotely for staging it sets you up with a full fledged WordPress server. It also helps you to deploy your WordPress site once you are ready to do so.



Ansible is both needed for setting up a remote server for staging or production on DreamCompute. Git will also be needed as well as the latest Python 2.x version for running Ansible

Installing Locally
~~~~~~~~~~~~~~~

Locally you do have to have several things running to work with Trellis:

* Ansible 2.0.2.0
* Virtualbox >= 4.3.10
* Vagrant >= 1.5.4
* vagrant-bindfs >= 0.3.1 (Windows users may skip this if not using vagrant-winnfsd for folder sync)
* vagrant-hostmanager


How you install things locally depends partly on your operating system and is not really part of this tutorial as we focus on the DreamCompute part of things. I recommend following the Trellis documentation on this `here
<https://roots.io/trellis/docs/local-development-setup/>`_
. Just remember the earlier mentioned prerequitsites. Without those on your local server or PC you won't be able to get things started. This and the proper changes in the Trellis configuration files for setting up a site locally and remotely the way you want. See documentation on this `here
<https://roots.io/trellis/docs/wordpress-sites/>`_


Setting Up Your Instance
~~~~~~~~~~~~~

Go to your DreamCompute Dashboard and pick Ubuntu from the list under images.This Trellis server setup on a DreamCompute instance is best done using a Ubuntu 14.0.4 image on DreamCompute. You can also use a more recent version of Ubuntu, Ubuntu 16.0.4. However, you will then be forced to install an older version of Python side by side with Python 3 on your DreamCompute instance. This you can do using: 

.. code:: 

    sudo apt-get install python^

If you do not mind this extra step then do go ahead. Always nice to run a more recent Ubuntu version, isn't it?

Just make sure you use Ubuntu as the Ansible playbooks used by Trellis to run the LEMP setup are built with Ubuntu/Debian in mind.
Whichever Ubuntu version you pick, remember it's better to boot volume backed instances as they are permanent as opposed to ephemeral disks.


Provisoning Your DreamCompute Instance
~~~~~~~~~~~~~

Provisioning Trellis means setting up the actual LEMP stack for your staging or production website. Staging and Production do not differ much. Do remember you need a separate instance for both though! Provisioning you normally do once you have worked out the proper site setup and have setup things locally.

Issues setting up Trellis
~~~~~~~~~~~~~
If you do run into issues ask a question at Roots Discourse `here
<https://discourse.roots.io/c/trellis>`_
This is the dedicated forum sub section for Trellis and that is where you can find the experts you need debuggig issues. Many errors with possible solution can also be found `here
<https://imagewize.com/web-development/roots-trellis-errors/>`_
