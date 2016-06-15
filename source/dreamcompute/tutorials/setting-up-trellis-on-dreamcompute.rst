======================================================
Step-by-step guide to set up Trellis on DreamCompute
======================================================

Trellis
~~~~~~~~~~~~

In this tutorial we are going to use `Trellis
<https://roots.io/trellis/>`_
to install a very sold modern LEMP stack on DreamCompute. This stack is great to run WordPress websites and works well with `Bedrock
<https://roots.io/bedrock/>`_
, the modern WordPress stack. 

Modern LEMP Stack
~~~~~~~~~~~~~~~

Trellis is a set of Ansible playbooks that help you setup a full local, staging and development environment for your project. With it you will have a WordPress ready server running locally or remotely (intention in this totorial) with:

* Nginx
* MariaDB (MySQL drop-in replacement)
* PHP 7 (+ extensions)
* Composer
* WP-CLI
* sSMTP
* Memcached
* Fail2ban
* ferm (firewall)
* Mailhog

Locally it works with an automated Vagrant setup and remotely for staging it sets you up with a full fledged WordPress server. It also helps you to deploy your WordPress site once you are ready to do so.


Requirements
~~~~~~~~~~~~~~~

Locally there are several `requirements
<https://roots.io/trellis/docs/installing-trellis/>`_
to work with Trellis and to be able to work remotely:

* Ansible 2.0.2.0
* Virtualbox >= 4.3.10
* Vagrant >= 1.5.4
* vagrant-bindfs >= 0.3.1 (Windows users may skip this if not using vagrant-winnfsd for folder sync)
* vagrant-hostmanager

**NB** Ansible is both needed for setting up a remote server for staging or production on DreamCompute. Git will also be needed as well as the latest Python 2.x version for running Ansible 2.0.2.0

Local Setup
~~~~~~~~~~~~~

How you install things locally depends partly on your operating system (OSX, Linux, Windows) and is not really part of this tutorial as we focus on the DreamCompute part of things. I recommend following the Trellis documentation on this `here
<https://roots.io/trellis/docs/local-development-setup/>`_
. Just remember the earlier mentioned prerequitsites. Without those on your local server or PC you won't be able to get things started. This and the proper changes in the Trellis configuration files for setting up a site locally and remotely the way you want. See documentation on this `here
<https://roots.io/trellis/docs/wordpress-sites/>`_ . But a bare minimal example of wordpress_sites.yml for development would be:

.. code::

    # group_vars/development/wordpress_sites.yml
    wordpress_sites:
      example.com:
        site_hosts:
          - example.dev
        local_path: ../site # path targeting local Bedrock site directory (relative to Ansible root)
        admin_email: admin@example.dev
        multisite:
          enabled: false
        ssl:
          enabled: false
        cache:
          enabled: false

For the other files like vault.yml we again refer to the documentation.          


Bedrock
*************

And for working with Bedrock - Modern WordPress Stack - which is really recommended we recommend you checking out the docs `here
<https://roots.io/bedrock/>`_ .


Setting Up Your Instance
~~~~~~~~~~~~~

Go to your DreamCompute Dashboard and pick Ubuntu from the list under images. This Trellis server setup on a DreamCompute instance is best done using a Ubuntu 14.0.4 image on DreamCompute. You can also use a more recent version of Ubuntu, Ubuntu 16.0.4. However, you will then be forced to install an older version of Python side by side with Python 3 on your DreamCompute instance. This you can do using: 

.. code:: 

    sudo apt-get install python

If you do not mind this extra step then do go ahead. Always nice to run a more recent Ubuntu version, isn't it?

Just make sure you use Ubuntu as the Ansible playbooks used by Trellis to run the LEMP setup are built with Ubuntu/Debian in mind.
Whichever Ubuntu version you pick, remember it's better to boot volume backed instances as they are permanent as opposed to ephemeral disks.


Provisoning Your DreamCompute Instance
~~~~~~~~~~~~~

Provisioning Trellis means setting up the actual LEMP stack for your staging or production website. Staging and Production do not differ much. Do remember you need a separate instance for both though! Provisioning you normally do once you have worked out the proper site setup and have setup things locally.

Trellis works best with passwordless SSH login so do make sure you have added your public SSH key in the DreamCompute Dashboard.

    "We assume that when you first create your server you've already added your SSH key to the root account. Digital Ocean will add this for you when you create a droplet. If you don't want to use an SSH key, you will need to add the --ask-pass option each time you run the server.yml playbook."



Configuration Files
~~~~~~~~~~

Let's say you work locally and on production only and have worked out things locally. Then you only need to set up / edit a couple of files for provisioning your remote server - setting up remote server documention `here
<https://roots.io/trellis/docs/remote-server-setup/>`_

WordPress Sites
*************

The first one to begin with is wordpress_sites.yml. This file is located under trellis/group_vars/production. In this file you need to change the following:

* name of site
* site_hosts
* repo (Github repo for example)
* multisite or not
* ssl or not and which provider
* cache enabled or not

Vault
*************

Once that is done you also need to edit vault.yml - extended documention `here
<https://roots.io/trellis/docs/vault/>`_ under trellis/group_vars/production. There you have to add:


* vault_mysql_root_password
* vault_wordpress_sites (same as in wordpress_sites.yml)
* db_password
* auth_key
* secure_auth_key
* logged_in_key
* nonce_key
* auth_salt
* secure_auth_salt
* logged_in_salt
* nonce_salt

Generate your keys here: https://roots.io/salts.html


Hosts
*************

Now under trellis open hosts/production. That is a file where you add your host details for making the real connection. If you do forget it you will net be able to connect and sometimes not get any errors at all. Here is an example

.. code::

    # Add each host to the [production] group and to a "type" group such as [web] or [db].
    # List each machine only once per [group], even if it will host multiple sites.

    [production]
    domain.com

    [web]
    domain.com


Users
*************

Wait, there is one more important file to attend to located in trellis/group_vars/all. That is users.yml. DreamCompute does not work with root but with the user dhc-user and that should be reflected in this file:

.. code::

    # Documentation: https://roots.io/trellis/docs/ssh-keys/
    admin_user: dhc-user
    # Also define 'vault_sudoer_passwords' (`group_vars/staging/vault.yml`, `group_vars/production/vault.yml`)
    users:
      - name: "{{ web_user }}"
        groups:
          - "{{ web_group }}"
        keys:
          - "{{ lookup('file', '~/.ssh/id_rsa.pub') }}"
          # - https://github.com/username.keys
      - name: "{{ admin_user }}"
        groups:
          - sudo
        keys:
          - "{{ lookup('file', '~/.ssh/id_rsa.pub') }}"
          # - https://github.com/username.keys
    web_user: web
    web_group: www-data
    web_sudoers:
      - "/usr/sbin/service php7.0-fpm *"

Everything else in this file can stay the same.

Push to Remote DreamCompute Instance
*************

Once you have the remote setup configured properly you can go ahead and push to the remote server using

.. code::

    ansible-playbook server.yml -e env=<environment>

Here *environment* will be production if you are pushing to production. staging is the other option.

**Note** Please understand that provisioning will take quite some time as a full stack server will be installed with Nginx, MariaDB, PHP 7 and beautiful things such as SSL, HTTP2 and so on. Also it takes care of setting up WordPress on the server. All in all a pretty great feat.


Deploying your site to DreamCompute
~~~~~~~~~~~~~

You have to realize that provisioning is just setting up your server for working with WordPress really well and at lightning speed. The instance is still not loading a site at all and going to the ip address or domain will show you a nice Nginx 404 as nothing can be found. You simply need to push your locally deployed WordPress site to the server still. Once that is done you will see remotely what you see locally.

Issues setting up Trellis
~~~~~~~~~~~~~

If you do run into issues ask a question at Roots Discourse `here
<https://discourse.roots.io/c/trellis>`_
This is the dedicated forum sub section for Trellis and that is where you can find the experts you need debuggig issues. Many errors with possible solution can also be found `here
<https://imagewize.com/web-development/roots-trellis-errors/>`_ .
