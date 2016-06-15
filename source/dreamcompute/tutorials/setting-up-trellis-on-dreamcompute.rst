======================================================
Step-by-step guide to set up Trellis on DreamCompute
======================================================

Trellis
~~~~~~~~~~~~

In this tutorial we are going to use `Trellis
<https://roots.io/trellis/>`_
to install a very sold modern LEMP stack on DreamCompute. This stack is great to run WordPress websites and works well with `Bedrock
<https://roots.io/bedrock/>`_
, the modern WordPress stack. Trellis is a set of Ansible playbooks that help you setup a full local, staging and development environment for your project. Locally it works with an automated Vagrant setup and remotely for staging it sets you up with a full fledged WordPress server.


Preparation
~~~~~~~~~~~~~~~

This Trellis server setup on a DreamCompute instance is best done using a Ubuntu 14.0.4 image on DreamCompute. You can also use a more recent version of Ubuntu, Ubuntu 16.0.4. However, you will then be forced to install an older version of Python side by side with Python 3 on your DreamCompute instance. This you can do using 
 .. code:: 
     sudo apt-get install python^

If you do not mind this extra step then do go ahead. Always nice to run a more recent Ubuntu version, isn't it?

Just make sure you use Ubuntu as the Ansible playbooks used by Trellis to run the LEMP setup are built with Ubuntu/Debian in mind.
Whichever Ubuntu version you pick, remember it's better to boot volume backed instances as they are permanent as opposed to ephemeral disks.

Locally you do also have to have several things running and to be able to push to the remote server:

* Ansible 2.0.2.0
* Virtualbox >= 4.3.10
* Vagrant >= 1.5.4
* vagrant-bindfs >= 0.3.1 (Windows users may skip this if not using vagrant-winnfsd for folder sync)
* vagrant-hostmanager

Ansible is both needed for setting up a remote server for staging or production on DreamCompute. Git will also be needed as well as the latest Python 2.x version for running Ansible

Installing LAMP
~~~~~~~~~~~~~~~

While you certainly can install everything on it's own, the LAMP stack for
Ubuntu is perfect for WordPress. If necessary, you can follow the directions
on `how to configure LAMP
<215879467-How-to-Configure-LAMP-on-DreamCompute-running-Debian-or-Ubuntu>`_
but the overview is as follows:

.. code::

    sudo apt-get install lamp-server^

This is interactive, so you’ll be asked ‘are you sure?’ in some places, and
in others it will want a password for SQL. Make up a secure password for SQL
and save it as you will need this later to set up SQL for WordPress.

After it runs, you’ll want to add mod_rewrite so WordPress can make pretty
pages:

.. code::

    sudo a2enmod rewrite

Finally restart apache:

.. code::

    sudo service apache2 restart

At this point, your server will be accessible via its IP address.

Create a User
~~~~~~~~~~~~~

While it's not required to make a separate user ID for WordPress, it's strongly
recommended for security. Having a user who only has access to the one website
will limit the havoc caused should that account be hacked. This also limits the
danger caused by rogue plugins or themes. The server itself will be safe,
containing the damage just to that user account.

To do this, we make a folder for the website:

.. code::

    sudo mkdir /var/www/example.com

And we create a user and give them access:

.. code::

    sudo adduser wp_example
    sudo adduser wp_example www-data
    sudo chown -R wp_example:www-data /var/www/example.com/

The reason we add the users to the www-data group is to allow Ubuntu to properly
manage WordPress updates and images.

Add SSH Access
~~~~~~~~~~~~~~

WordPress users often need SSH access in order to do extra configuration with
WordPress. By default, this is disabled, so you will need to edit your config.

.. code::

    sudo vi /etc/ssh/sshd_config

Look for the setting of PasswordAuthentication, change it to "yes", and save
your file. Remember to restart SSHD once you've done this.

.. code::

    sudo service sshd restart

Will this make your server less secure? Not significantly. As this new account
only has access to itself, it can only hack itself.

Add Your Domain
~~~~~~~~~~~~~~~

There are a few steps to set up your domain. First you'll need to `Setup DNS
for DreamCompute <218672058>`_ for all your domains.

Next you'll want to configure VirtualHosts so your server knows how to handle
the domain.

To do this, you need to make a .conf file:

.. code::

    sudo touch /etc/apache2/sites-available/example.com.conf

It's recommended you name the file after your domain, so you can always know
what file is for what domain.

Edit that file and put this in:

.. code::

    <VirtualHost *:80>
        ServerName example.com
        ServerAdmin admin@example.com
        DocumentRoot /var/www/example.com
        <Directory /var/www/example.com>
                AllowOverride all
        </Directory>

        ErrorLog ${APACHE_LOG_DIR}/example.com-error.log
        CustomLog ${APACHE_LOG_DIR}/example.com-access.log combined
    </VirtualHost>

Once the site is added, we'll need to enable it via a command called a2ensite
(if you want to disable, it’s a2dissite):

.. code::

    sudo a2ensite

This will prompt you to pick what site you want to enable. Type it in, hit
enter, and you’ll be told what’s next.

.. code::

    Your choices are: 000-default default-ssl example.com
    Which site(s) do you want to enable (wildcards ok)?
    example.com
    Enabling site example.com.
    To activate the new configuration, you need to run:
      service apache2 reload

Remember this command. It's a fast way to enable sites without having to rename
or mess with files. Finally bounce your apache service so it reads the changes:

.. code::

    sudo service apache2 reload

Create the Database and Users
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

WordPress absolutely requires a database. You'll want to create one

.. code::

    mysql -u root -p

Remember the password we set earlier? That’s what it’s for.

Your command prompt will be “mysql>” so let’s make the database:

.. code::

    mysql> CREATE DATABASE examplecom_wordpress;
    mysql> GRANT ALL ON examplecom_wordpress.* TO examplecom@localhost IDENTIFIED by 'PASSWORD';

Remember to change PASSWORD to an actually secure password.

You can check this by running the following command:

.. code::

    mysql -u examplecom -p examplecom_wordpress

Install WP-CLI
~~~~~~~~~~~~~~

While this is optional, we strongly recommend this. DreamHost includes `WP-CLI
<http://wp-cli.org/>`_ on all servers due to it's usefulness. To install, log
in as your default user (not the web user we created earlier) and run the
following:

.. code::

    cd ~
    curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar

Check that it works:

.. code::

    php wp-cli.phar --info

And if it does move it so everyone can use it!

.. code::

    chmod +x wp-cli.phar
    sudo mv wp-cli.phar /usr/local/bin/wp

That will make it accessible for all users.

Install WordPress
~~~~~~~~~~~~~~~~~

Log into your server as your WordPress SSH account (wp_example) and go to your
webfolder. If you've installed WP-CLI, then all you have to do is this:

.. code::

    wp core download

If you go to http://example.com now you’ll get that 5 minute install page.

Of course since you have wp-cli you can also do this:

.. code::

    wp core config --dbname=examplecom_wordpress --dbuser=examplecom --dbpass=PASSWORD
    wp core install --url=http://example.com --title=DreamComputePress --admin_user=YOURUSERNAME --admin_password=PASSWORD --admin_email=admin@example.com --skip-email

If you use secure passwords like cWG8j8FPPj{T9UDL_PW8 then you MUST put quotes
around the password.

I chose to skip-emails since I’m making it right there.

Miscellaneous Stuff
~~~~~~~~~~~~~~~~~~~

The following will make WordPress run even better, but aren't required.

Make sure apt has the latest and greatest.

.. code::

    sudo apt-get -y update

Make PHP Better

If you use a lot of media, install these to make PHP process images more better.

.. code::

    sudo apt install php-imagick php7.0-gd

Run a restart of apache when you’re done:

Troubleshooting
~~~~~~~~~~~~~~~

If WordPress can’t save files, you probably forgot to put your user in the right
group:

.. code::

    sudo adduser wp_example www-data
    sudo chown -R wp_example:www-data /var/www/example.com/

If that still doesn’t work, try this:

.. code::

    sudo chgrp -R www-data /var/www/example.com/
    sudo chmod -R g+w /var/www/example.com/

If pretty permalinks don't work, make sure you installed rewrite:

.. code::

    sudo a2enmod rewrite && sudo service apache2 restart

And make absolutely sure you have AllowOverride set to All in your Virtual Host:

.. code::

    <Directory /var/www/example.com>
        AllowOverride all
    </Directory>

It won’t work without it.

.. meta::
    :labels: wordpress
