# Bootstrap OpenStack Deployment

In this document I explain some of the commands of how to bootstrap a
development environment on OpenStack (as deployed via TripleO). Note that these
commands are what work in my environment. Your mileage may vary :)

Start by logging into the undercloud, and sourcing your `overcloudrc` file.

    source overcloudrc

## Create project (tenant)

We'll use this project when defining our networks to somewhat simulate a "real
world" scenario.

    openstack project create tenantA

    -------------+----------------------------------+
    | Field       | Value                            |
    +-------------+----------------------------------+
    | description | None                             |
    | enabled     | True                             |
    | id          | f1ab19b3cb164467ac7890f499726af0 |
    | name        | tenantA                          |
    +-------------+----------------------------------+

## Create user for tenant

    openstack user create leifmadsen --project tenantA --password welcome --email leif@tenantA.tld

    +------------+----------------------------------+
    | Field      | Value                            |
    +------------+----------------------------------+
    | email      | leif@tenantA.tld                 |
    | enabled    | True                             |
    | id         | 275aee33823644a1ab196ba389af078e |
    | name       | leifmadsen                       |
    | project_id | f1ab19b3cb164467ac7890f499726af0 |
    | username   | leifmadsen                       |
    +------------+----------------------------------+


## Create networks

Create a public/private network that is routable using the Openstack Unified
CLI.

### public network (floating IPs)

In my network the `external` network is segregated to VLAN 10 (not a flat
network) because I used `single-nic-vlans` deployment with `network-isolation`.

```
openstack network create public --external --provider-network-type vlan \
    --provider-physical-network datacentre --provider-segment 10 --project tenantA

+---------------------------+--------------------------------------+
| Field                     | Value                                |
+---------------------------+--------------------------------------+
| admin_state_up            | UP                                   |
| availability_zone_hints   |                                      |
| availability_zones        |                                      |
| created_at                | 2017-03-22T18:00:04Z                 |
| description               |                                      |
| headers                   |                                      |
| id                        | 5d6b7a6e-ca1a-46d9-860f-32b07748ca29 |
| ipv4_address_scope        | None                                 |
| ipv6_address_scope        | None                                 |
| is_default                | False                                |
| mtu                       | 1496                                 |
| name                      | public                               |
| port_security_enabled     | True                                 |
| project_id                | f1ab19b3cb164467ac7890f499726af0     |
| project_id                | f1ab19b3cb164467ac7890f499726af0     |
| provider:network_type     | vlan                                 |
| provider:physical_network | datacentre                           |
| provider:segmentation_id  | 10                                   |
| qos_policy_id             | None                                 |
| revision_number           | 4                                    |
| router:external           | External                             |
| shared                    | True                                 |
| status                    | ACTIVE                               |
| subnets                   |                                      |
| tags                      | []                                   |
| updated_at                | 2017-03-22T18:00:04Z                 |
+---------------------------+--------------------------------------+
```

```
openstack subnet create sub_public --network public --dhcp \
    --allocation-pool start=192.168.10.64,end=192.168.10.127 \
    --gateway 192.168.10.1 --subnet-range 192.168.10.0/24

+-------------------+--------------------------------------+
| Field             | Value                                |
+-------------------+--------------------------------------+
| allocation_pools  | 192.168.10.64-192.168.10.127         |
| cidr              | 192.168.10.0/24                      |
| created_at        | 2017-03-22T18:04:15Z                 |
| description       |                                      |
| dns_nameservers   |                                      |
| enable_dhcp       | True                                 |
| gateway_ip        | 192.168.10.1                         |
| headers           |                                      |
| host_routes       |                                      |
| id                | 7186f884-ad8e-4f0a-97db-2b63df6ebf5d |
| ip_version        | 4                                    |
| ipv6_address_mode | None                                 |
| ipv6_ra_mode      | None                                 |
| name              | sub_public                           |
| network_id        | 5d6b7a6e-ca1a-46d9-860f-32b07748ca29 |
| project_id        | 353a3a8031fe48e79100031f58c29491     |
| project_id        | 353a3a8031fe48e79100031f58c29491     |
| revision_number   | 2                                    |
| service_types     | []                                   |
| subnetpool_id     | None                                 |
| updated_at        | 2017-03-22T18:04:15Z                 |
+-------------------+--------------------------------------+
```

### private network

```
openstack network create private --project tenantA --internal
+---------------------------+--------------------------------------+
| Field                     | Value                                |
+---------------------------+--------------------------------------+
| admin_state_up            | UP                                   |
| availability_zone_hints   |                                      |
| availability_zones        |                                      |
| created_at                | 2017-03-22T18:06:02Z                 |
| description               |                                      |
| headers                   |                                      |
| id                        | 0ed133ce-d162-432c-9724-127a7130865d |
| ipv4_address_scope        | None                                 |
| ipv6_address_scope        | None                                 |
| mtu                       | 1446                                 |
| name                      | private                              |
| port_security_enabled     | True                                 |
| project_id                | f1ab19b3cb164467ac7890f499726af0     |
| project_id                | f1ab19b3cb164467ac7890f499726af0     |
| provider:network_type     | vxlan                                |
| provider:physical_network | None                                 |
| provider:segmentation_id  | 85                                   |
| qos_policy_id             | None                                 |
| revision_number           | 3                                    |
| router:external           | Internal                             |
| shared                    | False                                |
| status                    | ACTIVE                               |
| subnets                   |                                      |
| tags                      | []                                   |
| updated_at                | 2017-03-22T18:06:02Z                 |
+---------------------------+--------------------------------------+
```

```
openstack subnet create sub_private_tenantA --project tenantA \
    --network private --allocation-pool start=10.0.0.100,end=10.0.0.199 \
    --dns-nameserver 8.8.8.8 --dns-nameserver 8.8.4.4 --subnet-range 10.0.0.0/24
+-------------------+--------------------------------------+
| Field             | Value                                |
+-------------------+--------------------------------------+
| allocation_pools  | 10.0.0.100-10.0.0.199                |
| cidr              | 10.0.0.0/24                          |
| created_at        | 2017-03-22T18:07:28Z                 |
| description       |                                      |
| dns_nameservers   |                                      |
| enable_dhcp       | True                                 |
| gateway_ip        | 10.0.0.1                             |
| headers           |                                      |
| host_routes       |                                      |
| id                | 401d2b34-b223-46b2-b83e-32bb884d41e6 |
| ip_version        | 4                                    |
| ipv6_address_mode | None                                 |
| ipv6_ra_mode      | None                                 |
| name              | sub_private_tenantA                  |
| network_id        | 0ed133ce-d162-432c-9724-127a7130865d |
| project_id        | f1ab19b3cb164467ac7890f499726af0     |
| project_id        | f1ab19b3cb164467ac7890f499726af0     |
| revision_number   | 2                                    |
| service_types     | []                                   |
| subnetpool_id     | None                                 |
| updated_at        | 2017-03-22T18:07:28Z                 |
+-------------------+--------------------------------------+
```

```
openstack network list ; openstack subnet list
+--------------------------------------+---------+--------------------------------------+
| ID                                   | Name    | Subnets                              |
+--------------------------------------+---------+--------------------------------------+
| 0ed133ce-d162-432c-9724-127a7130865d | private | 401d2b34-b223-46b2-b83e-32bb884d41e6 |
| 5d6b7a6e-ca1a-46d9-860f-32b07748ca29 | public  | 7186f884-ad8e-4f0a-97db-2b63df6ebf5d |
+--------------------------------------+---------+--------------------------------------+
+--------------------------------------+---------------------+--------------------------------------+-----------------+
| ID                                   | Name                | Network                              | Subnet          |
+--------------------------------------+---------------------+--------------------------------------+-----------------+
| 401d2b34-b223-46b2-b83e-32bb884d41e6 | sub_private_tenantA | 0ed133ce-d162-432c-9724-127a7130865d | 10.0.0.0/24     |
| 7186f884-ad8e-4f0a-97db-2b63df6ebf5d | sub_public          | 5d6b7a6e-ca1a-46d9-860f-32b07748ca29 | 192.168.10.0/24 |
+--------------------------------------+---------------------+--------------------------------------+-----------------+
```

# Routing

Create the router between the public and private network so that we can assign
floating IPs.

## Create router

```
openstack router create --project tenantA provider
+-------------------------+--------------------------------------+
| Field                   | Value                                |
+-------------------------+--------------------------------------+
| admin_state_up          | UP                                   |
| availability_zone_hints |                                      |
| availability_zones      |                                      |
| created_at              | 2017-03-22T18:09:56Z                 |
| description             |                                      |
| distributed             | False                                |
| external_gateway_info   | null                                 |
| flavor_id               | None                                 |
| ha                      | False                                |
| headers                 |                                      |
| id                      | 10358a47-c6dd-4260-b2c0-de6f323561c2 |
| name                    | provider                             |
| project_id              | f1ab19b3cb164467ac7890f499726af0     |
| project_id              | f1ab19b3cb164467ac7890f499726af0     |
| revision_number         | 3                                    |
| routes                  |                                      |
| status                  | ACTIVE                               |
| updated_at              | 2017-03-22T18:09:56Z                 |
+-------------------------+--------------------------------------+

openstack router add subnet provider sub_private_tenantA

# NOTE: need to use neutron directly because there is an outstanding TODO in
# the python-openstackclient to add enable-gateway-net configuration.

neutron router-gateway-set \
    --fixed-ip subnet_id=sub_public,ip_address=192.168.10.254 \
    provider public
```

# Images

Download the CirrOS image (for testing) and upload it to the Swift instance.

## Download image to undercloud

    curl -sSL https://download.cirros-cloud.net/0.3.5/cirros-0.3.5-x86_64-disk.img -o cirros-0.3.5.img

## Upload image to overcloud

```
openstack image create --file cirros-0.3.5.img --public "CirrOS 0.3.5"
+------------------+------------------------------------------------------------------------------+
| Field            | Value                                                                        |
+------------------+------------------------------------------------------------------------------+
| checksum         | f8ab98ff5e73ebab884d80c9dc9c7290                                             |
| container_format | bare                                                                         |
| created_at       | 2017-03-22T18:25:33Z                                                         |
| disk_format      | raw                                                                          |
| file             | /v2/images/fc7e9c2a-3fb8-4b1d-b88c-25e9f728acc1/file                         |
| id               | fc7e9c2a-3fb8-4b1d-b88c-25e9f728acc1                                         |
| min_disk         | 0                                                                            |
| min_ram          | 0                                                                            |
| name             | CirrOS 0.3.5                                                                 |
| owner            | 353a3a8031fe48e79100031f58c29491                                             |
| properties       | direct_url='swift+config://ref1/glance/fc7e9c2a-3fb8-4b1d-b88c-25e9f728acc1' |
| protected        | False                                                                        |
| schema           | /v2/schemas/image                                                            |
| size             | 13267968                                                                     |
| status           | active                                                                       |
| tags             |                                                                              |
| updated_at       | 2017-03-22T18:25:35Z                                                         |
| virtual_size     | None                                                                         |
| visibility       | public                                                                       |
+------------------+------------------------------------------------------------------------------+

```

# Flavours

Create flavours for our infrastructure. My baremetal lab environment has fairly
lightweight machines, so my flavours have been adjusted to account for that.
Machine specs are:

* 110GB disk
* 4 core CPU
* 16GB RAM

## Create flavours

```
openstack flavor create m1.tiny --ram 256 --disk 1 --vcpus 1
openstack flavor create m1.small --ram 512 --disk 6 --vcpus 1
openstack flavor create m1.medium --ram 1024 --disk 10 --vcpus 2
openstack flavor create m1.large --ram 2048 --disk 20 --vcpus 2
openstack flavor create m1.xlarge --ram 4096 --disk 30 --vcpus 4

openstack flavor list
+--------------------------------------+-----------+------+------+-----------+-------+-----------+
| ID                                   | Name      |  RAM | Disk | Ephemeral | VCPUs | Is Public |
+--------------------------------------+-----------+------+------+-----------+-------+-----------+
| 448286d9-e2d8-4f3c-8d4f-95101118e305 | m1.xlarge | 4096 |   30 |         0 |     4 | True      |
| 4ace1a33-c567-4966-847e-68cedd82d285 | m1.small  |  512 |    6 |         0 |     1 | True      |
| 4d24901e-61da-4137-bb08-ef5ead12befb | m1.large  | 2048 |   20 |         0 |     2 | True      |
| 519d2a14-197a-4547-9b91-c9ca8ea47f1d | m1.medium | 1024 |   10 |         0 |     2 | True      |
| 67a44924-43ea-439e-99e6-92e3fe4bf584 | m1.tiny   |  256 |    1 |         0 |     1 | True      |
+--------------------------------------+-----------+------+------+-----------+-------+-----------+
```

*********************
Do following steps from user terminal
*********************

# Get User `openrc` File

The following commands will be run as the user we created earlier. Login to the
Horizon dasboard, Select `Project > Compute > Access & Security > API Access >
Download OpenStack RC File v2.0`

Download the file, and then source it so that you can authenticate to the cloud
as a project user.

    source tenantA-openrc.sh

You'll be prompted for your password.

# Create Server

Create the server and assign it to the private network

```
openstack server create --nic net-id=0ed133ce-d162-432c-9724-127a7130865d \
    --image fc7e9c2a-3fb8-4b1d-b88c-25e9f728acc1 --flavor m1.small testing

+--------------------------------------+-----------------------------------------------------+
| Field                                | Value                                               |
+--------------------------------------+-----------------------------------------------------+
| OS-DCF:diskConfig                    | MANUAL                                              |
| OS-EXT-AZ:availability_zone          |                                                     |
| OS-EXT-STS:power_state               | NOSTATE                                             |
| OS-EXT-STS:task_state                | scheduling                                          |
| OS-EXT-STS:vm_state                  | building                                            |
| OS-SRV-USG:launched_at               | None                                                |
| OS-SRV-USG:terminated_at             | None                                                |
| accessIPv4                           |                                                     |
| accessIPv6                           |                                                     |
| addresses                            |                                                     |
| adminPass                            | Fc24XTUiV6a5                                        |
| config_drive                         |                                                     |
| created                              | 2017-03-22T18:54:21Z                                |
| flavor                               | m1.small (4ace1a33-c567-4966-847e-68cedd82d285)     |
| hostId                               |                                                     |
| id                                   | a1a8be31-531c-4b44-922a-da4f6731d32f                |
| image                                | CirrOS 0.3.5 (fc7e9c2a-3fb8-4b1d-b88c-25e9f728acc1) |
| key_name                             | None                                                |
| name                                 | testing                                             |
| os-extended-volumes:volumes_attached | []                                                  |
| progress                             | 0                                                   |
| project_id                           | f1ab19b3cb164467ac7890f499726af0                    |
| properties                           |                                                     |
| security_groups                      | [{u'name': u'default'}]                             |
| status                               | BUILD                                               |
| updated                              | 2017-03-22T18:54:21Z                                |
| user_id                              | 275aee33823644a1ab196ba389af078e                    |
+--------------------------------------+-----------------------------------------------------+
```

# Floating IPs

## Request and assign floating IP
Create a floating IP on the public network and assign to a server

```
openstack floating ip create public
+---------------------+--------------------------------------+
| Field               | Value                                |
+---------------------+--------------------------------------+
| created_at          | 2017-03-22T18:58:00Z                 |
| description         |                                      |
| fixed_ip_address    | None                                 |
| floating_ip_address | 192.168.10.65                        |
| floating_network_id | 5d6b7a6e-ca1a-46d9-860f-32b07748ca29 |
| headers             |                                      |
| id                  | f19b07c1-4d69-4d3b-b271-d015e23927af |
| port_id             | None                                 |
| project_id          | f1ab19b3cb164467ac7890f499726af0     |
| project_id          | f1ab19b3cb164467ac7890f499726af0     |
| revision_number     | 1                                    |
| router_id           | None                                 |
| status              | DOWN                                 |
| updated_at          | 2017-03-22T18:58:00Z                 |
+---------------------+--------------------------------------+
```

```
openstack server add floating ip testing 192.168.10.65

openstack server list
+--------------------------------------+---------+--------+--------------------+--------------+
| ID                                   | Name    | Status | Networks           | Image Name   |
+--------------------------------------+---------+--------+--------------------+--------------+
| a1a8be31-531c-4b44-922a-da4f6731d32f | testing | ACTIVE | private=10.0.0.107 | CirrOS 0.3.5 |
+--------------------------------------+---------+--------+--------------------+--------------+

openstack floating ip list
+-------------------------------------+---------------------+------------------+--------------------------------------+
| ID                                  | Floating IP Address | Fixed IP Address | Port                                 |
+-------------------------------------+---------------------+------------------+--------------------------------------+
| f19b07c1-4d69-4d3b-b271-d015e23927a | 192.168.10.65       | 10.0.0.107       | 767ff283-cfe5-4343-ad05-fd646257ac1a |
| f                                   |                     |                  |                                      |
+-------------------------------------+---------------------+------------------+--------------------------------------+
```

# Security Groups

Create the rules in the default security group for the project

## Setup security group rules

```
openstack security group list
+--------------------------------------+---------+------------------------+----------------------------------+
| ID                                   | Name    | Description            | Project                          |
+--------------------------------------+---------+------------------------+----------------------------------+
| 3e355f7b-8382-49a8-8cf5-eef90cb08682 | default | Default security group | f1ab19b3cb164467ac7890f499726af0 |
+--------------------------------------+---------+------------------------+----------------------------------+

openstack security group rule create 3e355f7b-8382-49a8-8cf5-eef90cb08682 \
  --protocol icmp --src-ip 0.0.0.0/0

openstack security group rule create 3e355f7b-8382-49a8-8cf5-eef90cb08682 \
  --protocol tcp --dst-port 22:22 --src-ip 0.0.0.0/0
```
