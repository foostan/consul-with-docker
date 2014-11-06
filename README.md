consul-with-docker
=================

Consul Template and Registrator(https://github.com/progrium/registrator) have good chemistry, and combine Consul and Docker effectively.
For example, they enable to update HAProxy node that is running on Docker easy and quickly, in real time.

This repository is environment of Consul with Docker include Consul Template and Registrator.
Usage is here in Japanese http://fstn.hateblo.jp/entry/2014/10/26/153247

## Overview
![Overview Figure](http://cdn-ak.f.st-hatena.com/images/fotolife/f/foostan/20141026/20141026021814.png)

A Virtual Machine include these packages

- Docker
- Consul
- Consul Template
- Registrator
- HAProxy

and, the endpoint of web services is `192.168.33.11:80` in this environment.

## Prepare environment

### Build by Vagrant and Ansible

```
$ git clone https://github.com/foostan/consul-with-docker.git
$ cd consul-with-docker
$ vagrant up
```

### Run Consul agent

```
vagrant@node-1:~$ consul agent -data-dir=/tmp/consul -server -bootstrap
==> WARNING: Bootstrap mode enabled! Do not enable unless necessary
==> WARNING: It is highly recommended to set GOMAXPROCS higher than 1
==> Starting Consul agent...
==> Starting Consul agent RPC...
==> Consul agent running!
         Node name: 'node-1'
        Datacenter: 'dc1'
            Server: true (bootstrap: true)
       Client Addr: 127.0.0.1 (HTTP: 8500, DNS: 8600, RPC: 8400)
      Cluster Addr: 10.0.2.15 (LAN: 8301, WAN: 8302)
    Gossip encrypt: false, RPC-TLS: false, TLS-Incoming: false
```

### Run Consul template
```
vagrant@node-1:~$ sudo consul-template\
 -consul=localhost:8500\
 -template="/vagrant/haproxy.ctmpl:/etc/haproxy/haproxy.cfg:service haproxy reload"
```

## Run Registrator
```
vagrant@node-1:~$ registrator consul:
```

## Build tinyweb and Run a container

### Build tinyweb
Tinyweb is a simple web server for a test environment by nc command.

```
vagrant@node-1:~$ docker build -t foostan/tinyweb /vagrant/tinyweb/
```

### Run a container

```
docker run -p 80 -d foostan/tinyweb
c437b2717b76dde8c31211fea09bc5bb60b8c16892161db24cb31a1b49543ae1
vagrant@node-1:~$ docker ps
CONTAINER ID        IMAGE                    COMMAND                CREATED             STATUS              PORTS                   NAMES
c437b2717b76        foostan/tinyweb:latest   "\"/bin/sh -c 'while   5 minutes ago       Up 5 minutes        0.0.0.0:49178->80/tcp   romantic_carson
```

## Check
Add a tinyweb service in consul by Registrator and update HAProxy configuration by Consul Template after running a container.

Check Consul services

```
$ curl -s localhost:8500/v1/catalog/services | jq .
{
  "consul": [],
  "tinyweb": []
}
vagrant@node-1:~$ curl -s localhost:8500/v1/catalog/service/tinyweb | jq .
[
  {
    "Node": "node-1",
    "Address": "10.0.2.15",
    "ServiceID": "node-1:romantic_carson:80",
    "ServiceName": "tinyweb",
    "ServiceTags": null,
    "ServicePort": 49178
  }
]
```
Added tinyweb and registrated address and port of a node.

Check HAProxy

```
vagrant@node-1:~$ tail -n 7 /etc/haproxy/haproxy.cfg

frontend  main *:80
    default_backend    tinyweb

backend tinyweb
    balance roundrobin
    server node-1:romantic_carson:80 10.0.2.15:49178 check
```

Updated the HAProxy configuration.

Access check

```
vagrant@node-1:~$ curl localhost
c437b2717b76
```

Without a virtual machine
```
$ curl 192.168.33.11
c437b2717b76
```

### Run and kill multiple containers
Run 5 containers

```
vagrant@node-1:~$ docker run -d -p 80 foostan/tinyweb
df9d19738e7610c448478c94ec10ab63259a2c78cfbe6437de2d3e7a8870b63e
vagrant@node-1:~$ docker run -d -p 80 foostan/tinyweb
b7ddc05c645490c21c18b98161bef0994b9c2ad366ff6e85a0b45652ff204e6d
vagrant@node-1:~$ docker run -d -p 80 foostan/tinyweb
0c31735035d4da4519ebbe7a6957ddb9bc1f865e5448c9ab838a742bff3178d3
vagrant@node-1:~$ docker run -d -p 80 foostan/tinyweb
8be6547a4a418faf34be1727b40386c149dd34d41ce5f8efb8c37d37050849ee
vagrant@node-1:~$ docker run -d -p 80 foostan/tinyweb
a7969a2dfa63d137d77369b5e9c519b5565ec0bc2b14126c300b24c11c25deb5
```

Updated the HAProxy configuration

```
vagrant@node-1:~$ tail -n 12 /etc/haproxy/haproxy.cfg

frontend  main *:80
    default_backend    tinyweb

backend tinyweb
    balance roundrobin
    server node-1:hopeful_newton:80 10.0.2.15:49181 check
    server node-1:hopeful_torvalds:80 10.0.2.15:49179 check
    server node-1:naughty_elion:80 10.0.2.15:49182 check
    server node-1:romantic_carson:80 10.0.2.15:49178 check
    server node-1:sad_darwin:80 10.0.2.15:49180 check
    server node-1:thirsty_kowalevski:80 10.0.2.15:49183 check
```

Access check
```
$ curl 192.168.33.11
0c31735035d4
$ curl 192.168.33.11
df9d19738e76
$ curl 192.168.33.11
8be6547a4a41
$ curl 192.168.33.11
c437b2717b76
$ curl 192.168.33.11
b7ddc05c6454
$ curl 192.168.33.11
a7969a2dfa63
```

Kill containers and Updated the HAProxy configuration
```
vagrant@node-1:~$ docker kill a7969a2dfa63 b7ddc05c6454 c437b2717b76
a7969a2dfa63
b7ddc05c6454
c437b2717b76
vagrant@node-1:~$ tail -n 9 /etc/haproxy/haproxy.cfg

frontend  main *:80
    default_backend    tinyweb

backend tinyweb
    balance roundrobin
    server node-1:hopeful_newton:80 10.0.2.15:49181 check
    server node-1:hopeful_torvalds:80 10.0.2.15:49179 check
    server node-1:naughty_elion:80 10.0.2.15:49182 check
```
