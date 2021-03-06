# Lab 03 - Examine OSA Configuration Files

## Goals

Understand the preconfigured OSA Configuration

## Prerequisites

* Access to the running host instances

## Overview

Before installing OpenStack-Ansible (OSA), the OSA configuration files need to be generated. 

https://docs.openstack.org/project-deploy-guide/openstack-ansible/rocky/configure.html

We've taken the liberty to generate a configuration file for your specific lab environment. We're going to walk you through that configuration file.

Keep in mind that these OSA configuration files were automatically generated by a Terraform OSA config file generator. It handled all the subnet generating for us specific to the IP address subnets that were assigned to each physical host by the underlying bare metal cloud provider. The generator also left comments in the configuration file with more general details about the networks assigned.

## Key Maintenance

(If you haven't already)
Before we start, we need some permission cleanup on the lab files. Some files are currently owned by root and needs to be owned by your user. We'll run ```chmod``` to give you access to them.

```
osa02@osa-lab-master:~/terraform$ sudo chown `whoami` *
```

## Examine OSA Configuration File

You'll be looking at the OSA configuration file on infra0 to see how it is setup. Take note of the IP address while you're logged into the osa-master as output from Terraform.

```
osa02@osa-lab-master:~/terraform$ terraform output
osa02@osa-lab-master:~/terraform$ ssh root@<your infra0 IP> -i default.pem
```

Pull up the OSA configuration file.

```
root@infra0:~# more /etc/openstack_deploy/openstack_user_config.yml
```

### Assigned Subnets

A /25 subnet is divided up across all of the physical hosts as /28 subnets to be used for the containers. OSA runs the various OpenStack services in these containers.

```
cidr_networks:
  infra0_container_subnet:   10.99.213.16/28
  infra0_tunnel_subnet:      10.99.213.48/28
  compute0_container_subnet: 10.99.213.32/28
  compute0_tunnel_subnet:    10.99.213.64/28
```

### Provider Networks

Walk through all of the network definitions taking a look at which virtual bridge, the assigned subnet, and the gateway address. The gateway is the IP address, first in the subnet, that was setup in the networking configuration from the previous step where the networks were configured.

Notice how there is a different subnet of IPs for each of the physical hosts. Since these hosts are in a diffenent L2 network segment, they need to use a different L3 subnets. For physical hosts on the same L2 network, they could share the same pool of IP addresses.

```
  provider_networks:
    - network:
        container_bridge: "br-mgmt"
        container_type: "veth"
        container_interface: "eth1"
...
```

### Host Groups

Further down in the configuration file we can see where all the host groups are defined. These are used to determine which OpenStack services to run atop which physical hosts.


```
# horizon
dashboard_hosts:
  infra0:
    ip: 27.75.67.197

# neutron server, agents (L3, etc)
network_hosts:
  infra0:
    ip: 27.75.67.197

# nova hypervisors
compute_hosts:
  compute0:
    ip: 27.75.192.173
```



## Next Steps

Once you're comfortable with the configuration files proceed to [Lab 4](Lab04.md)

(C) JHL Consulting LLC 2019
