Installation
------------

Presently our installation relies on the person installing having root access and performing a git clone operation and than a number of steps to setup the database, performing some configuration and then importing the data and validating the full setup.

Prerequisites
.............

* A Linux machine with sufficient RAM (~20 GB used for CMS installation with 6 million objects)
  * The softsoftware itself does not depend on the distribution, but the server daemon scripts are only written for RHEL 6 and 7.
* Super-user access to the installation machine
* Python 2.6 or 2.7
* MySQL server with root access
* If REST API is enabled (recommended), an HTTP(S) server with FastCGI capabilities (`Lighttpd <https://www.lighttpd.net/>`_ is recommended)
* Host X509 certificate
* User X509 certificates
* Some non-default Python modules
  * `ssl` (required)
  * `sqlite3` (required)
  * `flup` (required for REST API)
  * `fts3` (required for file operations using `FTS3 <https://fts.web.cern.ch/>`_)
  * `gfal2` (required for file operations using `GFAL2 <https://dmc.web.cern.ch/projects/gfal-2/home>`_)
  * `lzma` (required for Detox)
  * `rrdtool` (required for Dealer monitoring)

Basic Installation
..................

The core software and most of the functionalities are contained in the `dynamo <https://github.com/SmartDataProjects/dynamo>`_ package. To start login as root and make a temporary directory:
::
   
   su -
   mkdir tmp   

To install, clone the package, configure, and run the installation script:
::

   git clone https://github.com/SmartDataProjects/dynamo
   
   # Before doing the install please go through configuration (described right below)
   
   ./dynamo/install.sh

   
Configuration
.............

There are a few steps to follow before running the installation script:

#. Copy configuration template to default location and edit the contents. Default configuration should work for most cases.
   ::

      cd dynamo
      cp dynamo.cfg.template dynamo.cfg 
   
   Installation target directories are configurable. With default settings, files are copied to

   - `/usr/local/dynamo`: For python libraries, daemon executables, standard applications, web templates, and an init script
   - `/etc/dynamo`: For configuration files
   - `/var/log/dynamo`: For server logs
   - `/var/spool/dynamo`: For scheduler work area
   - `/local/data/dynamo`: For data archival

   The server and all applications will be run under a normal UNIX user, which can be specified in the configuration file but must be created beforehand. (see 'user' line in the 'server' section in the configuration file dynamo.cfg)

#. Copy `dynamo/mysql/grants.json.template` to `dynamo/mysql/grants.json` and enter the user passwords. This file specifies what user accounts and permissiong grants should be created on the MySQL server. By default, four users are created with different usage classes:

   - `dynamosrv` is the MySQL user with full access to all relevant databases. This is the user used by the main Dynamo server. The password for `dynamosrv` should not be readable by normal users.
   - `dynamo` is the MySQL user with full access for all practical purposes running the Dynamo applications.
   - `dynamoread` is the restricted-access MySQL user designed for read-only applications.
   - `dynamofod` is a MySQL user specialized for performing file transfer and deletion operations.

   The MySQL users will be created on the fly during the installation if they do not exist already.
#. Copy `dynamo/defaults.json.template` to `dynamo/defaults.json` and edit
   - Passwords for MySQL users (three lines; must be identical to what is in grants.json)
   - X509 certificate to be used by the server user when accessing various external HTTPS REST resources (in `utils.interface.webservice:HTTPSCertKeyHandler` block).


Initial Data Import
...................

All users must be authorized before interacting with the Dynamo server. To add a user, use `dynamo-user-auth` as super-user:

::
  
  source /usr/local/dynamo/etc/profile.d/init.sh
  dynamo-user-auth --user <user name> --dn "<user certificate DN>"

Further application-specific authorization can be added using the same script. See the `--help` option for more details.



Validate Full Setup
...................

With the server running, use the `dynamo` command as one of the authorized users:

::

  $ dynamo

  +++++++++++++++++++++++++++++++++++++
  ++++++++++++++ DYNAMO +++++++++++++++
  ++++++++++++++  v2.1  +++++++++++++++
  +++++++++++++++++++++++++++++++++++++
  
  >>> 

An interactive session appears with an interface with the full functionality of the python interpreter. The only difference from the normal python interpreter is that the session loaded with a preset object `inventory`, which represents the Dynamo server inventory. Initial data injection can be validated by inspecting the inventory object:

::

  >>> inventory.datasets
  {}
  >>> inventory.sites
  {}
