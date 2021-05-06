# Introduction

AppQoS is a proof-of-concept software created to demonstrate the use of
Intel(R) RDT technologies (CAT, MBA) to improve QoS for applications
via partitioning system resources.

_NOTE: This is just a quick start-up guide. For mor informations about AppQoS please see [Advanced_AppQoS_Usage_Guide.txt](https://raw.githubusercontent.com/intel/intel-cmt-cat/master/appqos/doc/Advanced_AppQoS_Usage_Guide.txt)_

# Installation

AppQoS is a part of intel-cmt-cat repo, available at GitHub.

### Clone the repository

```
$ git clone https://github.com/intel/intel-cmt-cat.git

Cloning into 'intel-cmt-cat'...
...
```
_NOTE: For further informations about cloning repository from github please see [Cloning a repository from GitHub](https://docs.github.com/en/free-pro-team@latest/github/creating-cloning-and-archiving-repositories/cloning-a-repository)_

### Install libpqos and virtualenv

Install libpqos
```
$ cd ./intel-cmt-cat
$ make && sudo make install
```

Create virtualenv
```
$ cd ./appqos/
$ make setup
...
Creating a virtualenv for this project...
...
Successfully created virtual environment!
...
```
All required dependencies are now installed in virtualenv.


### Creating evalutation mTLS certificate for testing

The appqos/ca/ directory contains sample script called 'gen_test_certs.sh' - it can be used for generating certifacates for the AppQoS and the client (e.g: curl) - keep in mind that those keys are not for production use.

```
$ cd appqos/ca
$ ./gen_test_certs.sh
```

This script will create:
- ca certificate
- appqos certificate
- client certificate

_NOTE: Generated keys are not for production use - you should obtain production certificate and keys from your administrator. More information about requirements for AppQoS mTLS certificates can be found in [Advanced_AppQoS_Usage_Guide.txt](https://raw.githubusercontent.com/intel/intel-cmt-cat/master/appqos/doc/Advanced_AppQoS_Usage_Guide.txt)_

# Configuration

AppQoS reads its initial configuration from configuration file. During the runtime it accepts commands via REST API.

_NOTE: None of configuration changes made via REST API are saved to configuration file._

### Configuration file
By default AppQoS will attempt to read from "appqos.conf" file from current folder.

Create ./appqos.conf file.
```
cat ./appqos.conf
{
    "auth": {
        "password": "password",
        "username": "admin"
    },

    "rdt_iface": {
        "interface": "os"
    },

    "mba_ctrl": {
        "enabled": false
    },

    "apps": [],

    "pools": []
}
```
Basic configuration file with essential configuration only:
 - REST API Authentication details,
 - RDT interface and MBA mode configuration

# Execution

"make run" command executes AppQoS in virtualenv with all required dependencies already installed.

_NOTE: AppQoS requires root privileges._

```
$ sudo make run

WORKON_HOME=../venv/appqos_silpixa00399791 pipenv run ./appqos.py
2020-12-14 19:30:52,497 INFO  Interface MSR, MBA BW: unsupported.
2020-12-14 19:30:52,527 INFO  Interface OS, MBA BW: supported.
2020-12-14 19:30:52,527 INFO  Supported RDT interfaces: ['msr', 'os']
2020-12-14 19:30:52,539 INFO  RDT initialized with 'os' interface
2020-12-14 19:30:52,701 INFO  Supported capabilities:
2020-12-14 19:30:52,701 INFO  ['cat', 'mba']
 * Serving Flask app "rest.rest_server" (lazy loading)
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: off
2020-12-14 19:30:52,727 INFO   * Running on https://127.0.0.1:5000/ (Press CTRL+C to quit)
2020-12-14 19:30:52,734 INFO  SST-BF not enabled
2020-12-14 19:30:52,734 INFO  Power Profiles/EPP not enabled
2020-12-14 19:30:52,734 INFO  Configuring RDT
2020-12-14 19:30:52,896 INFO  RDT MBA CTRL disabled
2020-12-14 19:30:52,903 INFO  Configuration changed, processing new config...
```

AppQoS is up and runnig ready for REST API commands.

### System capabilities detection
During start-up process AppQoS performs system capabilites detection and prints out info on the console.
```
2020-12-14 19:30:52,497 INFO  Interface MSR, MBA BW: unsupported.
2020-12-14 19:30:52,527 INFO  Interface OS, MBA BW: supported.
2020-12-14 19:30:52,527 INFO  Supported RDT interfaces: ['msr', 'os']
```
System supports RDT MSR and OS interfaces. MBA BW/MBA CTRL is supported only for OS interface.
```
2020-12-14 19:30:52,539 INFO  RDT initialized with 'os' interface
2020-12-14 19:30:52,701 INFO  Supported capabilities:
2020-12-14 19:30:52,701 INFO  ['cat', 'mba']
```
RDT is initialized with OS interface and supports both CAT and MBA.

### Preparing mTLS key and certificate for REST API client
Before establishing the connection from client side, we need to make sure that private key and the certificates needed for mTLS connection are accessible.

REST client needs AppQoS ca.crt certificate, client certificate signed by AppQoS ca.crt and a accompanying private key for connecting with AppQoS server. If you are using evaluation mTLS certificates, generated by the gen_test_certs.sh scripts, those files can be found in the appqos/ca directory

```
$ mkdir ~/appqos_client_workspace
$ cp appqos/ca/client_appqos.key appqos/ca/client_appqos.crt appqos/ca/ca.crt ~/appqos_client_workspace
$ cd ~/appqos_client_workspace
$ ls
ca.crt  client_appqos.crt  client_appqos.key
```

_NOTE: More information about requirements for AppQoS mTLS client certificates can be found in [Advanced_AppQoS_Usage_Guide.txt](https://raw.githubusercontent.com/intel/intel-cmt-cat/master/appqos/doc/Advanced_AppQoS_Usage_Guide.txt)_

### Capabilities queries via REST API
It is also possible to get supported capabilities information via REST API

_NOTE: 5000 is default REST API port. Is also assumed that every curl command will be running in directory where ca.crt, client_appqos.crt and client_appqos.key are locally accessible_

**List supported and current RDT interface**
```
$ curl https://localhost:5000/caps/rdt_iface -X GET --cert client_appqos.crt --key client_appqos.key --cacert ca.crt

{
  "interface": "os",
  "interface_supported": [
    "msr",
    "os"
  ]
}

```
_NOTE: OS is current RDT interface, but MSR interface is also supported._

**List MBA CTRL status**
```
$ curl https://localhost:5000/caps/mba_ctrl -X GET --cert client_appqos.crt --key client_appqos.key --cacert ca.crt

{
  "supported": true,
  "enabled": false
}
```
_NOTE: MBA CTRL is supported but not enabled_

Verify MBA CTRL status
```
$ mount | grep resctrl

resctrl on /sys/fs/resctrl type resctrl (rw,relatime)
```
_NOTE: resctrl fs is mounted but no "mba_MBps" option is used, MBA CTRL is not enabled._

**List RDT capabilities**
```
$ curl https://localhost:5000/caps -X GET --cert client_appqos.crt --key client_appqos.key --cacert ca.crt

{
  "capabilities": [
    "cat",
    "mba"
  ]
}
```
_NOTE: RDT supports CAT and MBA._

### Pool operations via REST API

**Default pool**

If there is no "Default" pool (with "id" equal to 0) defined in config file,
AppQoS will dynamically create one on start-up.

All unassigned cores will be assigned to "Default" pool,
MBA will be configured to "no throttling" (100% (default) or ~2^32MBps (MBA CTRL enabled))
and CAT CBM to all cache ways.

**List all configured pools**
```
$ curl https://localhost:5000/pools -X GET --cert client_appqos.crt --key client_appqos.key --cacert ca.crt

[
  {
    "id": 0,
    "mba": 100,
    "cbm": 2047,
    "name": "Default",
    "cores": [
      0,
      1,
      2,
...
      45,
      46,
      47
    ]
  }
]
```

_NOTE: The only configured pool is a "Default" one, with MBA configured to "no throttling" (100%) `"mba": 100`._

**Modify "Default" pool**

Modify "Default" pool to exclude multiple cores.

```
$ curl https://localhost:5000/pools/0 -X PUT --cert client_appqos.crt --key client_appqos.key --cacert ca.crt -H "Content-Type: application/json" -d '{"cores": [5,6,7,8,9,17,18,19,20,21,22,23,24,26,27,28,29,30,31,32,33,41,42,43,44,45,46,47]}'

{"message": "POOL 0 updated"}
```
**Check new "Default" pool configuration**

```
$ curl https://localhost:5000/pools/0 -X GET --cert client_appqos.crt --key client_appqos.key --cacert ca.crt

{
  "id": 0,
  "mba": 100,
  "cbm": 2047,
  "name": "Default",
  "cores": [
    5,
    6,
    7,
...
    45,
    46,
    47
  ]
}

```

_NOTE: New cores configuration `"cores": [5, 6, 7, ... , 45, 46, 47].`_

**Create new pool**

```
$ curl https://localhost:5000/pools -X POST --cert client_appqos.crt --key client_appqos.key --cacert ca.crt -H "Content-Type: application/json" -d '{"name": "HP", "cores": [0,1,2,3,4,10,11,12,13,14,15,16,25,34,35,36,37,38,39,40], "mba": 100, "cbm": 2047}'

{"id": 7, "message": "New POOL 7 added"}
```

_NOTE: New pool with "id=7" was created._

Get new pool details

```
$ curl https://localhost:5000/pools/7 -X GET --cert client_appqos.crt --key client_appqos.key --cacert ca.crt

{
  "name": "HP",
  "cores": [
    0,
    1,
    2,
...
    38,
    39,
    40
  ],
  "mba": 100,
  "cbm": 2047,
  "id": 7
}
```
_NOTE: New pool's MBA is configured to 100% `"mba": 100`._

**Modify new pool's MBA**

```
$ curl https://localhost:5000/pools/7 -X PUT --cert client_appqos.crt --key client_appqos.key --cacert ca.crt -H "Content-Type: application/json" -d '{"mba": 70}'

{"message": "POOL 7 updated"}
```
Get new pool details
```
$ curl https://localhost:5000/pools/7 -X GET --cert client_appqos.crt --key client_appqos.key --cacert ca.crt

{
  "name": "HP",
  "cores": [
    0,
    1,
    2,
...
    38,
    39,
    40
  ],
  "mba": 70,
  "cbm": 2047,
  "id": 7
}
```
_NOTE: New pool's MBA is configured to 70% `"mba": 70`._

### Enable MBA CTRL

**Enable RDT MBA CTRL**
```
$ curl https://localhost:5000/caps/mba_ctrl -X PUT --cert client_appqos.crt --key client_appqos.key --cacert ca.crt -H "Content-Type: application/json" -d '{"enabled": true}'

{"message": "Please remove all Pools first!"}
```
_NOTE: MBA CTRL state can only be changed when there are no Pools configured (other than "default" Pool #0)
(e.g.: all non-default, configured Pools were removed)._
```
$ curl https://localhost:5000/pools/7 -X DELETE --cert client_appqos.crt --key client_appqos.key --cacert ca.crt

{"message": "POOL 7 deleted"}
```

```
curl https://localhost:5000/caps/mba_ctrl -X PUT --cert client_appqos.crt --key client_appqos.key --cacert ca.crt -H "Content-Type: application/json" -d '{"enabled": true}'

{"message": "MBA CTRL status changed."}
```
_NOTE: After MBA CTRL state change, "Default" pool is reset to default values (all cores, full cache and MEM BW access)._

**Verify MBA CTRL status**
```
$ mount | grep resctrl

resctrl on /sys/fs/resctrl type resctrl (rw,relatime,mba_MBps)
```
_NOTE: resctrl fs mounted with "mba_MBps" option, MBA CTRL is now enabled._
```
$ curl https://localhost:5000/caps/mba_ctrl -X GET --cert client_appqos.crt --key client_appqos.key --cacert ca.crt

{
  "supported": true,
  "enabled": true
}

$ curl https://localhost:5000/caps/mba -X GET --cert client_appqos.crt --key client_appqos.key --cacert ca.crt

{
  "mba_enabled": false,
  "mba_bw_enabled": true
}
```
_NOTE: MBA CTRL is now enabled. User can use "mba_bw" to configure pool's MBA allocation._

**Verify MBA configuration for "Default" Pool**
```
$ curl https://localhost:5000/pools -X GET --cert client_appqos.crt --key client_appqos.key --cacert ca.crt

[
  {
    "id": 0,
    "mba_bw": 4294967295,
    "cbm": 2047,
    "name": "Default",
    "cores": [
      0,
      1,
      2,
...
      45,
      46,
      47
    ]
  }
]
```
_NOTE: MBA for Pool #0 set to maximum value 4294967295MBps (~2^32MBps) `"mba_bw": 4294967295`._

**Create new pool and limit its MBA to ~5GBps and then ~10GBps**

Modify "Default" pool to exclude multiple cores

```
$ curl https://localhost:5000/pools/0 -X PUT --cert client_appqos.crt --key client_appqos.key --cacert ca.crt -H "Content-Type: application/json" -d '{"cores": [5,6,7,8,9,17,18,19,20,21,22,23,24,26,27,28,29,30,31,32,33,41,42,43,44,45,46,47]}'

{"message": "POOL 0 updated"}
```

Create new pool with 5GBps MBA limit
```
$ curl https://localhost:5000/pools -X POST --cert client_appqos.crt --key client_appqos.key --cacert ca.crt -H "Content-Type: application/json" -d '{"name": "HP", "cores": [0,1,2,3,4,10,11,12,13,14,15,16,25,34,35,36,37,38,39,40], "mba_bw": 5000, "cbm": 2047}'

{"id": 7, "message": "New POOL 7 added"}
```

Verify MBA configuration for new pool, COS#7
```
$ curl https://localhost:5000/pools/7 -X GET --cert client_appqos.crt --key client_appqos.key --cacert ca.crt

{
  "name": "HP",
  "cores": [
    0,
    1,
    2,
...
    38,
    39,
    40
  ],
  "mba_bw": 5000,
  "cbm": 2047,
  "id": 7
}

cat /sys/fs/resctrl/COS7/schemata

    L3:0=7ff;1=7ff
    MB:0=5000;1=5000
```
_NOTE: Both AppQoS and Resctrl show new MBA configuration (5000MBps) for Pool #7._

Modify new pool MBA to 10GBps
```
$ curl https://localhost:5000/pools/7 -X PUT --cert client_appqos.crt --key client_appqos.key --cacert ca.crt -H "Content-Type: application/json" -d '{"mba_bw": 10000}'

{"message": "POOL 7 updated"}
```
Verify new pool MBA configuration

```
$ curl https://localhost:5000/pools/7 -X GET --cert client_appqos.crt --key client_appqos.key --cacert ca.crt

{
  "name": "HP",
  "cores": [
    0,
    1,
    2,
...
    38,
    39,
    40
  ],
  "mba_bw": 10000,
  "cbm": 2047,
  "id": 7
}

$ cat /sys/fs/resctrl/COS7/schemata

    L3:0=7ff;1=7ff
    MB:0=10000;1=10000
```
_NOTE: Both AppQoS and Resctrl show new MBA configuration (10000MBps) for Pool #7._
