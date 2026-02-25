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

No, not enough, need to create a network.




## Router/Network Connectivity

```
./bin/openstack network create thenet
```

```
./bin/openstack router create theway
```

List the networks.

```
./bin/openstack network list
```

Then use e.g., `PSNC-EXT-IPV6-PUB2-EDU`.

```
./bin/openstack router set --external-gateway PSNC-EXT-IPV6-PUB2-EDU theway
```

List the subnet pools.

```
./bin/openstack subnet pool list
```

Then use e.g., `bgp-vlan92-pub2v6-pool1`.

```
./bin/openstack subnet create thev6 --network thenet --ip-version 6 --subnet-pool bgp-vlan92-pub2v6-pool1 --prefix-length 64 --ipv6-ra-mode dhcpv6-stateless --ipv6-address-mode dhcpv6-stateless
```

```
./bin/openstack router add subnet theway thev6
```



## Security Group

Allow ICMP for ping:

```
./bin/openstack security group rule create --proto icmp --ethertype IPv6 default
```

Allow SSH:

```
./bin/openstack security group rule create --proto tcp --dst-port 22 --remote-ip ::/0 --ethertype IPv6 default
```



## Re-create the VM


Now with the network:

```
./bin/openstack server create --flavor m1-nvme-1-4-50 thevm --image debian-13 --key-name ed25519 --network thenet
```

Show the console:

```
./bin/openstack console log show thevm
```

Says:

```
ci-info: no authorized SSH keys fingerprints found for user debian.
```

Ping the machine:

```
$ ping 2001:808:3:60a:f816:3eff:fe3a:644d
PING 2001:808:3:60a:f816:3eff:fe3a:644d (2001:808:3:60a:f816:3eff:fe3a:644d) 56 data bytes
64 bytes from 2001:808:3:60a:f816:3eff:fe3a:644d: icmp_seq=1 ttl=240 time=85.6 ms
64 bytes from 2001:808:3:60a:f816:3eff:fe3a:644d: icmp_seq=2 ttl=240 time=30.8 ms
^C
```

SSH into the machine:

```
$ ssh debian@2001:808:3:60a:f816:3eff:fe3a:644d
The authenticity of host '2001:808:3:60a:f816:3eff:fe3a:644d (2001:808:3:60a:f816:3eff:fe3a:644d)' can't be established.
ED25519 key fingerprint is SHA256:/hw0pqEHTfoD+dbTs3Ykw3dmSeKh8+gxncd0veRgkSo.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '2001:808:3:60a:f816:3eff:fe3a:644d' (ED25519) to the list of known hosts.
debian@2001:808:3:60a:f816:3eff:fe3a:644d: Permission denied (publickey).
```
