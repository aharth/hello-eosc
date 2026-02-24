# Hello European Open Science Cloud

EOSC EU Node


## Set Up Environment/Virtual Machine

First, log in at https://open-science-cloud.ec.europa.eu/.

Next, select Virtual Machines (an Environment).

Get access to a Small for a few days.

After a while you have an allocated Environment.

Clicking on "View externally" leads to the OpenStack interface.



## Application Credentials

Next, create an application credential to authenticate OpenStack scripts.

In the OpenStack interface, select Application Credentials.

Create credential, select Roles reader and member (no idea what they do).

Download the `clouds.yaml`.



## Install OpenStack

```
python3 -m venv .
```

```
./bin/pip install python-openstackclient
```


## Work with OpenStack

Now, try to list the available servers:

```
$ ./bin/openstack server list
The request you have made requires authentication. (HTTP 401) (Request-ID: req-64510208-016d-4e59-bdd5-2fe1be4cd3c9)
```

Ok, debug output:

```
$ ./bin/openstack --debug server list
...
```
