Adding Content
--------------

After the installation of your Dynamo data management system, it is of course empty. In the the following we show how to fill content into your data management system. We need to add Storage Sites, and Data and then we need to setup the access.

**Once the server is running and user(s) have been added (last section) we are now only working as the owner of the above given certificate(s).**

.. _addstorage-ref:

Add Storage Sites
.................

We create a json file which specifies the storage sites. Here is a typical example of such a json file called mydynamo-storage-sites.json:
::
  
    {"site":
     [
      {"name": "T2_US_XYZ", "host": "t2-storage.xyz.edu", "backend": "gsiftp://t2-storage.xyz.edu:2811/cms", "status": "ready"},
      {"name": "T3_US_XYZ", "host": "t3-storage.xyz.edu", "backend": "srm://t3-storage.xyz.edu:8443/srm/v2/server?SFN=/mnt/hadoop/cms", "status": "ready"}
     ]
    }

Note that the LFN to PFN translation is very basic by attaching the LFN to the backend at the storage site. Now we upload this json to the inventory server:
::

   dynamo-inject mydynamo-storage-sites.json

Verify it was correctly uploaded:
::

   dynamo
   >>> inventory.sites
   {'T3_US_XYZ': Site('T3_US_XYZ','t3-storage.xyz.edu','disk','srm://t3-storage.xyz.edu:8443/srm/v2/server?SFN=/mnt/hadoop/cms','ready',2), 'T2_US_XYZ': Site('T2_US_XYZ','t2-storage.xyz.edu','disk','gsiftp://t2-storage.xyz.edu:2811/cms','ready',1)}


Initial Data Injection
......................

Assuming you have already a large amount of data that you would like Dynamo to manage, here is a way of injecting this data. Here is a simple example of a dataset that has one block which contains 2 files. The dataset is kept open because there will be more files added eventually.
::

    {"dataset":
      [
        {"name": "pandaf/010/DoubleMuon+Run2017F-31Mar2018-v1+MINIAOD",
         "is_open": true,
         "last_update": 1528406008,
         "software_version": ["pandaf/010"],
         "status": "production",
	 "data_type": "unknown",
         "blocks":
          [
            {"name": "48dbe0c6-36fa-11e8-a5e2-ac1f6b05e848",
             "is_open": true,
             "last_update": 1528406008,
             "num_files": 2,
             "size": 165438,
             "files":
              [
                {"name": "/store/user/paus/pandaf/010/DoubleMuon+Run2017F-31Mar2018-v1+MINIAOD/6C7F843A-4837-E811-93AC-14187741208F.root", "size": 57495},
                {"name": "/store/user/paus/pandaf/010/DoubleMuon+Run2017F-31Mar2018-v1+MINIAOD/6EABBD4C-2E37-E811-AB8D-1866DAEA7F94.root", "size": 107943}
              ]
            }
          ]
        }
      ],
     "datasetreplica":
      [
        {"dataset": "pandaf/010/DoubleMuon+Run2017F-31Mar2018-v1+MINIAOD",
         "site": "T2_US_XYZ",
         "blockreplicas":
          [
            {"block": "48dbe0c6-36fa-11e8-a5e2-ac1f6b05e848", "last_update": 1528406008}
          ]
        }
      ]
    }
  

Verify it was correctly uploaded:
::

   dynamo
   >>> inventory.datasets
   {'pandaf/010/DoubleMuon+Run2017F-31Mar2018-v1+MINIAOD': Dataset('pandaf/010/DoubleMuon+Run2017F-31Mar2018-v1+MINIAOD','production','unknown','pandaf/010',1528406008,True,1)}
   >>> inventory.datasets['pandaf/010/DoubleMuon+Run2017F-31Mar2018-v1+MINIAOD'].blocks
   set([Block('48dbe0c6-36fa-11e8-a5e2-ac1f6b05e848','pandaf/010/DoubleMuon+Run2017F-31Mar2018-v1+MINIAOD',165438,2,True,1528406008,1,False)])


.. _applicationauthorization-ref:

Application Authorization
.........................

Any python script can be run as a Dynamo application by
::

  dynamo '<script> [<options>]'

The script will be run as a subprocess of the server, inheriting the memory image of the inventory from the parent process. A script executed this way becomes a *readonly* process, where any changes made to the inventory during the execution will be discarded at the end. To execute a script as a *read/write* process, it first needs to be authorized by the super-user.
::

  su -
  source /usr/local/dynamo/etc/profile.d/init.sh
  dynamo-exec-auth --executable <script> --title <title> [--user <user>]

`<title>` can be any string. With the `--user` option, only the specified user will be allowed to execute the script with read/write permission. To execute a read/write application, as an authorized user do
::

  dynamo '<script> [<options>]' --write-request --title <title>

The authorization step must be performed every time there is a change to the `<script>`.
