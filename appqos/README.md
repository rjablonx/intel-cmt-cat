# Introduction

AppQoS is a proof-of-concept software created to demonstrate the use of
Intel(R) RDT technologies (CAT, MBA) to improve QoS for applications
via partitioning system resources.

_NOTE: This is just a quick start-up guide. For mor informations about AppQoS please see [AppQoS README](https://raw.githubusercontent.com/intel/intel-cmt-cat/master/appqos/README)_

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

For evaluation purposes, the appqos/ca/ directory contains sample script called 'gen_test_certs.sh' - it can be used for generating certifacates for the AppQoS and the client (e.g: curl) - keep in mind that those keys are not for production use.

```
$ cd appqos/ca
$ ./gen_test_certs.sh
```

This script will create:
- ca certificate
- appqos certificate
- client certificate

_NOTE: Generated keys are not for production use - you should obtain production certificate and keys from your administrator. More information about requirements for AppQoS mTLS certificates can be found in  [Advanced_AppQoS_Usage_Guide.txt](https://raw.githubusercontent.com/intel/intel-cmt-cat/master/appqos/doc/Advanced_AppQoS_Usage_Guide.txt)_
