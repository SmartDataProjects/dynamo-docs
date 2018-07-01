Advanced Usage
--------------

Here we cover how to expand and customize Dynamo.


Setting Up a Secondary Server (Horisontal Scaling)
--------------------------------------------------

Multiple machines can run Dynamo server processes (secondary servers) as one big instance of Dynamo. Each secondary server holds a RAM image of the inventory, which are synchronized to each other at real time. This horizontal scaling feature can be used for load distribution (all servers with identical configuration) and / or division of roles (different server for different tasks). Additionally, when the primary server becomes unavailable, one of the secondary servers can take up the role of the primary, thus providing a high-availability fallback mechanism.

The prerequisites and the installation instruction for the secondary server is mostly identical to the primary, as documented in :ref:`installation-ref` section. In particular, secondary servers also require a MySQL server locally. The only difference in the setting for a secondary server is the server configuration. The server host name within the master_conf parameter in the [server] section must specify the primary server:
::
   
   master_conf={"host": "primary-dynamo.dynamo.net", "user": "dynamosrv", "readuser": "dynamoread"}

Inter-server communications happen through MySQL tables. Therefore, all server processes must be able to write to each other's dynamo database tables. However, no additional ports need to be opened on participating machines. Note that it is in principle possible to set MySQL passwords for the secondary server differently from the primary, but it is highly recommended to use identical configurations for horizontally connected servers.

Because the initial primary server is statically defined in the secondary servers' configuation file, whenever the cluster of Dynamo servers are started, the primary server must be started first. Similarly, when shutting down the cluster, the primary must be the last process to terminate.

The secondary servers can be run with or without a local persistency store (i.e. the inventory MySQL database). A secondary server with a local persistency can serve as a database backup for the primary.

Writing a New Application
-------------------------

Any python script can become a Dynamo application. Scripts intended for making changes to the inventory must be authorized using the `dynamo-exec-auth` command as documented in :ref:`installation-ref`. Note that the authorization is based on the checksum of the script file, and therefore the authorization must be renewed whenever there is a change to the script.

The inventory object is available to all applications through an import.
::

    from dynamo.core.executable import inventory

Changes to the inventory are made in two ways.
::

    inventory.update(some_object)

adds a clone of `some_object` into the inventory, where `some_object` is an object defined in `dynamo.dataformat` (Block, BlockReplica, Dataset, DatasetReplica, File, Group, Site, SitePartition). If the object with the same identifier (name for Dataset, File, Group, and Site, name+dataset for Block, name+site for SitePartition, block+site for BlockReplica, and dataset+site for DatasetReplica) already exists in the inventory, its attribute values are copied from `some_object`.
::

    inventory.delete(some_object)

deletes an object from the inventory. Again the object identifier is used to find the object to delete from the inventory; `some_object` does not have to be an object contained in the inventory.

When a script is run in the read-only mode, `update()` and `delete()` methods of the inventory object only changes the state of the inventory within the child process executing the script. All changes are thrown away at the end of execution. In the read/write mode, the changes are collected in the order of calls to `update()` and `delete()` made during the execution, and are sent back to the server when the script terminates with return code 0.

Even for scripts that run in the read/write mode, it is advised to write it such that it can be run also in the read-only mode. To hide certain parts of the script from read-only execution, use the `read_only` boolean flag available as
::

    from dynamo.core.executable import read_only


Adding a New Web Interface
--------------------------

Web pages and REST API commands available under `http://<host>/web/` and `http://<host>/data/` are defined in `dynamo.web.modules`. The python module and package name under the `modules` directory are directly used as the web interface name:
::

    dynamo/lib/web/modules/mymod.py
    =>
    http://<host>/web/mymod
    or
    http://<host>/data/mymod

    dynamo/lib/web/modules/mypkg/__init__.py
                                /abc.py
    =>
    http://<host>/web/mypkg
    or
    http://<host>/data/mypkg

Note that in the above example, `abc.py` is not directly mapped to any URL. Mapping to specific URL is done by `export_web` and `export_data` variables defined in the module or package. For example, if `mymod.py` contains two lines
::

    export_web = {'myweb': WebClassDefinedInMyMod}    
    export_data = {'mydata': DataClassDefinedInMyMod}

then `http://<host>/web/mymod/myweb` returns the value computed by a `WebClassDefinedInMyMod` object, presumably defined in `mymod.py`. Similarly, `http://<host>/data/mymod/mydata` returns the value computed by a `DataClassDefinedInMyMod` object. Within a package, `export_web` and `export_data` must be defined in the `__init__.py` file. There are no other restrictions on how the package should be structured.

Classes that appear in `export_web` and `export_data` must inherit from `WebModule` in `dynamo.web.modules._base` and have at least two methods: a constructor that takes one argument and a `run` function that takes three arguments. The argument to the constructor is a `Configuration` object which carries the parameters defined in the web modules configuration file, as specified in the `modules_config` parameter in the `[web]` section of the server configuration. The signature of the `run` function is
::

    run(caller, request, inventory)

where `caller` is a User object defined within the WebServer class in dynamo.web.server, representing the user making the web request. This object carries meaningful information only for HTTPS requests. The second argument `request` is a python dictionary containing the HTTP GET and POST requests. Keys of `request` are all strings, and the values are either strings or lists of strings. The final argument is the inventory object, whose behavior is described in detail in the previous section.

The return value of the `run()` method must be an HTML document (for /web URLs) or a value that is serializable as a JSON (for /data URLs). For /web URLs, a handy class `HTMLMixin <https://github.com/SmartDataProjects/dynamo/blob/master/lib/web/modules/_html.py>`_ exists to handle various routines in the background.
