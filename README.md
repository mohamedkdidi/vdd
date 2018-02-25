Vagrant Drupal Development
==========================

Vagrant Drupal Development (VDD) is a fully configured and ready to use
development environment built on Linux (Ubuntu 12) with Vagrant, VirtualBox and
Chef Solo provisioner. VDD is a virtualized environment, so your base system will
not be changed and remain clean after installation. You can create and delete as
many environments as you wish without any consequences.

Note: VDD works great with 6, 7 and 8 versions of Drupal.

The main goal of the project is to provide an easy to use fully functional and
highly customizable Linux based environment for Drupal development.

Setup is very simple, fast and can be performed on Windows, Linux or Mac
platforms. It's simple to clone an existing environment to your Laptop or home
computer and then keep it synchronized.

If you are not familiar with Vagrant, please read about it. Documentation is very
simple to read and understand. http://docs.vagrantup.com/v2/

To start VDD you don't need to write your Vagrantfile. All configuration can be
done a inside simple JSON configuration file.


Out of the box features
=======================

  * Configured Linux, Apache, MySQL and PHP stack
  * Drush with site aliases
  * PhpMyAdmin
  * Xdebug

TODO:

  * Webgrind
  * Xhprof
  * Mailing system
  * Other development tools and useful software


Getting Started
===============

VDD uses Chef Solo provisioner. It means that your environment is built from
the source code. All you need is to get base system, the latest code and build
your environment.

  1. Install VirtualBox
     https://www.virtualbox.org/wiki/Downloads

  1. Install Vagrant
     http://docs.vagrantup.com/v2/installation/index.html

  1. Install vagrant host updater plugin. Please note on this one that "plugin" is not available in some lower version       of Vagrant. The solution so far is to upgrade to higher versions.
     `$ vagrant plugin install vagrant-hostsupdater`

  1. Prepare VDD source code
     Download and unpack VDD source code and place it inside your local development folder.
     `git clone CLONE_URL_OF_THIS_REPO`

  1. Adjust configuration | Copy `config.example.json` to `config.json`
     You can edit `config.json` file to adjust your settings. If you use VDD first
     time it's recommended to leave the original config as is.

  1. Build your environment (Note : You might want to add your project before doing this step)
     To build your environment execute next command inside your VDD copy:
     `$ vagrant up --provision`

     Vagrant will start to build your environment. You'll see green status
     messages while Chef is configuring the system.

  1. Visit `192.168.44.44` address OR the hostname specified in the config.json
     If you didn't change the default IP address in `config.json` file you'll see
     VDD's main page. Main page has links to configured sites, development tools
     and list of frequently asked questions. Follow instruction on how to quickly install Drupal

  1. Configure Drupal Code sniffer with Coder module inside bin directory
     Reference : https://drupal.org/node/1419988

It's now time to add your default project inside the `www` folder.

Basic Usage
===========

Inside your VDD copy's directory you can find `data` directory. It's
synchronized with virtual machine. You should place your application's files
inside sub folders with the name of your project. You can edit your application's
files on the host machine using your favorite editor or connect to virtual
machine by SSH. VDD will never delete data from data directory, but you should
backup it.

Vagrant's basic commands (should be executed inside VDD directory):

  * `$ vagrant ssh`
    _SSH into virtual machine._

  * `$ vagrant up`
    _Start virtual machine._

  * `$ vagrant halt`
    _Halt virtual machine._

  * `$ vagrant destroy`
    _Destroy you virtual machine. Source code and content of data directory will
    remain unchangeable. VirtualBox machine instance will be destroyed only. You
    can build your machine again with `vagrant up` command. The command is
    useful if you want to save disk space._

  * `$ vagrant provision`
    _Reconfigure virtual machine after source code change._

  * `$ vagrant reload`
    _Reload virtual machine. Useful when you need to change network or
    synced folder settings._

Official Vagrant site has beautiful documentation.
http://docs.vagrantup.com/v2/

Drush Integration
=================

If your `Host` machine has Drush already you can use it to manage your VDD environment. All you need to do is create a file called `vdd.aliases.drushrc.php` in `~/.drush` in your host machine. The file should contain the below script.

```
<?php
$aliases['vdd'] = array(
  'parent' => '@parent',
  'site' => 'vdd',
  'env' => 'local',
  'uri' => 'vdd/NAME_OF_FOLDER',
  'root' => '/var/www/NAME_OF_FOLDER/',
  'remote-host' => '192.168.44.44',
  'remote-user' => 'vagrant',
  'php' => '/usr/bin/php',
);
```

To be able to execute Drush commands against VDD, while on your host machine:

`$ drush @vdd [DRUSH_COMMAND]`

Examples:

`$ drush @vdd cc all`<br>
`$ drush @vdd en views -y`<br>
`$ drush @vdd vset preprocess_css 1 -y`<br>
`$ drush @vdd status`<br>

N.B. Make sure that you uploaded the host public key on Vagrant.


Customizations
==============

You should understand that every time you start virtual machine Vagrant will
fire Chef provisioner. If you want to customize your VDD copy you should do it
right way.

Templates override

If you want to change some configuration files, for example, php.ini you should
override default VDD's template. All templates a located in
`cookbooks/core/vdd/templates/default`

All you need is to copy template file into `cookbooks/core/vdd/templates/ubuntu`
directory and edit it.

Writing custom role

If you want to make serious modifications you should write your custom role and
add it in `config.json` file. Please, see `vdd_example.json` file inside roles
directory.

config.json description
=======================

`config.json` is the main configuration file. Data from `config.json` is used to
configure virtual machine. After editing file make sure that your JSON syntax is
valid. http://jsonlint.com/ can help to check it.

  * `hostname (string, required)`
    _URL of the machine automatically added to the /etc/hosts if you have vagrant
    host updater plugin (vagrant plugin install vagrant-hostsupdater)_

  * `ip (string, required)`
    _Static IP address of virtual machine. It is up to the users to make sure
    that the static IP doesn't collide with any other machines on the same
    network. While you can choose any IP you'd like, you should use an IP from
    the reserved private address space._

  * `memory (string, required)`
    _RAM available to virtual machine. Minimum value is 1024._

  * `synced_folder (object of strings, required)`
    _Synced folder configuration._

      * `host_path (string, required)`
        _A path to a directory on the host machine. If the path is relative, it
        is relative to VDD root._

      * `guest_path (string, required)`
        _Must be an absolute path of where to share the folder within the guest
        machine._

  * `php (object of strings, required)`
    _PHP configuration._

      * `version (string or false, required)`
        _Desired PHP version. Please, see http://www.php.net/releases for proper
        version numbers. If you would like to use standard Ubuntu package you
        should set number to "false". Example: "version": false._

  * `mysql (object of strings, required)_`
    _MySQL configuration._

      * `server_root_password (string, required)`
        _MySQL server root password._

  * `sites (object ob objects, required)`
    _List of sites (similar to virtual hosts) to configure. At least one site is
    required._

      * `Key (string, required)`
        _Machine name of a site. Name should fit expression '[^a-z0-9_]+'. Will
        be used for creating subdirectory for site, Drush alias name, database
        name, etc._

          * `account_name (string, required)`
            _Drupal administrator user name._

          * `account_pass (string, required)`
            _Drupal administrator password._

          * `account_mail (string, required)`
            _Drupal administrator email._

          * `site_name (string, required)`
            _rupal site name._

          * `site_mail (string, required)`
            _Drupal site email._

  * `xdebug (object of strings, optional)`
    _Xdebug configuration._

    * `remote_host (string, required)`
      _Selects the host where the debug client is running._

  * `git (object of strings, optional)`
    _Git configuration._

  * `custom_roles (array, required)`
    _List of custom roles. Key is required, but can be empty array ([])._

If you find a problem, incorrect comment, obsolete or improper code or such,
please let us know by creating a new issue at
http://drupal.org/project/issues/vdd

Maintained by the [developers at x-team](https://www.x-team.com) | [developer blog](https://www.x-team.com/blog/)

