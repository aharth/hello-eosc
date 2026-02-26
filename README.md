# Hello European Open Science Cloud

EOSC EU Node (on eu-1).


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

No, not enough, need to create a network.




## Router/Network Connectivity

```
./bin/openstack network create eosc-net
```

```
./bin/openstack subnet create --network eosc-net --subnet-range 192.168.10.0/24 --dns-nameserver 8.8.8.8 eosc-subnet
```

List the networks.

```
./bin/openstack network list
```

Create router.

```
./bin/openstack router create eosc-router
```

Attach subnet to router (for traffic to flow out).

```
./bin/openstack router add subnet eosc-router eosc-subnet
```

Set gateway.

```
./bin/openstack router set eosc-router --external-gateway PSNC-EXT-PUB1-EDU
```

Boot VM on private network.

```
./bin/openstack server create --flavor m1-nvme-1-4-50 thevm --image debian-13 --key-name ed25519 --network eosc-net
```

Allocate floating IP.

```
./bin/openstack floating ip create PSNC-EXT-PUB1-EDU
```

Attach IP.

```
./bin/openstack server add floating ip thevm <IP>
```

## Log into VM via SSH

Check routing.

```
./bin/openstack router show eosc-router
```


Check VM IP address.

```
./bin/openstack server show thevm -c addresses
```

Finally, log into the machine.

```
ssh debian@<IP>
```



## Security Group

Not sure that is needed...

Allow ICMP for ping:

```
./bin/openstack security group rule create --proto icmp --ethertype IPv6 default
```

Allow SSH:

```
./bin/openstack security group rule create --proto tcp --dst-port 22 --remote-ip ::/0 --ethertype IPv6 default
```
