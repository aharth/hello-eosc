# Hello European Open Science Cloud

EOSC EU Node


## Install OpenStack

Use the OpenStack client over the OpenStack browser interface for reliable and reproducible results.

```
python3 -m venv .
```

```
./bin/pip install python-openstackclient
```


## Set Up Environment/Virtual Machine

Log in at https://open-science-cloud.ec.europa.eu/.

Next, select Virtual Machines (an Environment).

Get access to a Small for a few days.

After a while you have an allocated Environment.

Clicking on "View externally" leads to the OpenStack interface.



## Application Credentials

Next, create an application credential to authenticate OpenStack scripts.

In the OpenStack interface, select Application Credentials.

Create credential, select Role `member`.

Download the `clouds.yaml`.


## Check OpenStack

Now, try to list the available servers:

```
./bin/openstack server list
The request you have made requires authentication. (HTTP 401) (Request-ID: req-64510208-016d-4e59-bdd5-2fe1be4cd3c9)
```

If that does not work, use debug output:

```
./bin/openstack --debug server list
```

Or show configuration (attention: prints the Secret from `clouds.yaml`):

```
./bin/openstack configuration show
```



## Register Public Key

Register public key (for SSH login), from existing SSH keypair.

```
./bin/openstack keypair create --public-key ~/.ssh/id_ed25519.pub ed25519
```



## Create a Virtual Machine

List the available virtual machine hardware:

```
./bin/openstack flavor list
```

(should return types of VMs)


List the avilable operating system images:

```
./bin/openstack image list
```

(should return types of images/os)

Now, create a Virtual Machine flavour `m1-nvme-1-4-50` (a puny one) with name `thevm` with OS `debian-13`:

```
./bin/openstack server create --flavor m1-nvme-1-4-50 thevm --image debian-13 --key-name ed25519
```

Takes a while to provision and boot, check status with

```
./bin/openstack server list
```

or

```
./bin/openstack server show <server-id>
```