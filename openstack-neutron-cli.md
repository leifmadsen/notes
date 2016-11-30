# Create a public-private network
Create a public/private network that is routable using the Openstack Unified
CLI.

## create networks
It is assumed the external network was already created, as I'm using oooq.

```
openstack network create private --internal --project admin --enable

openstack subnet create int-subnet --project admin --network private \
  --allocation-pool start=192.168.100.100,end=192.168.100.199 --subnet-range \
  192.168.100.0/24

openstack network list ; openstack subnet list

openstack router create router

openstack router add subnet router 7f1a5beb-b450-4593-a0fd-09ec3e5c03fa

# NOTE: need to use neutron directly because there is an outstanding TODO in
# the python-openstackclient to add enable-gateway-net configuration.

neutron router-gateway-set router public
```

# create server
Create the server and assign it to the private network

```
openstack server create --nic net-id=c2a672e8-ad19-4e17-a6ef-650258307408 \
  --image d5777196-36e8-4b90-b8e8-dd7523640e11 --flavor m1.small testing

```

# floating ip create and assign
Create a floating IP on the public network and assign to a server

```
openstack floating ip create public

openstack server add floating ip testing 192.168.24.109
```

# setup security group rules
Create the rules in the default security group for the project

```
openstack security group list ;  openstack project list

openstack security group rule create 6697f5f1-0e83-450f-a403-fe52fa6de003 \
  --protocol icmp --src-ip 0.0.0.0/0

openstack security group rule create 6697f5f1-0e83-450f-a403-fe52fa6de003 \
  --protocol tcp --dst-port 22:22 --src-ip 0.0.0.0/0
```
