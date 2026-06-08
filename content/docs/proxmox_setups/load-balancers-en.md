+++
title = "Load balancers"
description = "Simple walkthrough of our current HA setup"
template = "docs/page.html"
sort_by = "weight"
weight = 10001
draft = false

[extra]
lang = "en"
translation = "docs/proxmox_setups/Load-balancers.md"
+++

## What is load balancing?

Load balancing is done by receiving requests on one IP/endpoint and forwarding
them in a balanced way to a distributed system.

This is useful for things like Kubernetes, since all services are available
through all master nodes (`hutre`, `petter`, and so on).

## What do we have?

Today we have 2 load balancers. They are hosted on Pluto and Fergus
respectively.

`load-balancer-1` is the master and `load-balancer-2` is the slave.

These two proxy traffic to the Kubernetes cluster, which lets us use IPv4 to
access the Kubernetes cluster even though it is pure IPv6. We also have load
balancing between the master nodes in the cluster. This also means that it is
important that these two VMs have IPv6 addresses.

Today we use a system with HAProxy and Keepalived to proxy traffic onward.

Both load balancers also run a WireGuard server that is connected to
Keepalived. This gives us access to internal services such as the Proxmox UI,
Rancher, and any future services that we do not necessarily want to expose to
the internet.

In theory, all public services should live as an ingress in the Kubernetes
cluster and automatically be available through these 2 load balancers. This goes
through HAProxy to ports 80 and 443 on any of the master nodes in the
Kubernetes cluster. This gives us automatic load balancing between the master
nodes in the cluster.

## Synchronizing configuration between lb-1 and lb-2

Today this is done with a script located on `load-balancer-1`.

This means that all changes should be made on `load-balancer-1`, and then you
run the script to migrate the changes over to `load-balancer-2`.

You can find the script at `/home/fribyte/propagate_haproxy_wg.sh` on
`load-balancer-1`.

## Keepalived

Keepalived uses a protocol called VRRP to provide a floating IP between the two
servers.

This is on IP `158.37.6.28`.

`load-balancer-1` is configured as master and continuously checks that HAProxy
is up and running.

**NB** There is no liveness check on WireGuard, so if the WireGuard service
crashes, it will not automatically switch to `load-balancer-2`. **NB**

If HAProxy goes down, or `load-balancer-1` goes down, `load-balancer-2` will
take over the shared IP.

This is completely seamless for the user.

Here is the config used today:

```
MASTER:

vrrp_script chk_haproxy {
    script 'killall -0 haproxy' # faster than pidof
    interval 2
}

vrrp_instance VI_1 {
  state MASTER
  interface eth0
  virtual_router_id 55
  priority 150
  advert_int 1
  unicast_src_ip MASTER_IP
  unicast_peer {
    SLAVE_IP
  }
  authentication {
    auth_type PASS
    auth_pass """INSERT PASSWORD HERE"""
  }
  virtual_ipaddress {
    SHARED_IP
  }
  track_script {
    chk_haproxy
  }
}

SLAVE:

vrrp_script chk_haproxy {
    script 'killall -0 haproxy' # faster than pidof
    interval 2
}

vrrp_instance VI_1 {
  state BACKUP
  interface eth0
  virtual_router_id 55
  priority 100
  advert_int 1
  unicast_src_ip SLAVE_IP
  unicast_peer {
    MASTER_IP
  }
  authentication {
    auth_type PASS
    auth_pass """INSERT PASSWORD HERE"""
  }
  virtual_ipaddress {
    SHARED_IP
  }
  track_script {
    chk_haproxy
  }
}
```

## HAProxy

HAProxy is used to proxy the traffic onward.

The plan is to use this both as a proxy for external services in the Kubernetes
cluster and for internal services.

This is done by connecting both load balancer VMs to the same WireGuard VPN
network. All internal services are then bound to the VPN network, while all
external services are bound directly to the floating IP.

You can find the HAProxy config at `/etc/haproxy/haproxy.cfg` on either
`load-balancer-1` or `load-balancer-2`.

## WireGuard

This is set up exactly like it used to be on Skaftetrynet.

Information about how to connect to it can be found under
[nytt_medlem](/docs/innmelding/nytt-medlem#wireguard).

You can find the config for this in `/etc/wireguard/wg0.conf` on
`load-balancer-1` or `load-balancer-2`.
