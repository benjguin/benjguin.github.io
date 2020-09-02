---
title: deploying containers to one or several VMs
date: 2016-08-31
tags: 
- Azure
- docker
---

Here is a sample test to show how Docker Swarm and Docker Compose simplify network management. 

We'll deploy a topology of 3 containers and see how they can reach each other. In the first case, we deploy it on a single node. In the second case, we deploy it on a 2 host swarm cluster.

The Swarm cluster is created as a Swarm Azure Container Service instance with the main following parameters: 
- Orchestrator configuration: Swarm (btw, other option for ACS is DC/OS)
- Agent count: 2
- Agent virtual machine size: (leave default)
- Master count: 1
- DNS prefix for container service: myacs

Two files are created

The Dockerfile contains: 

```
FROM busybox

ENTRYPOINT ["init"]
```

The docker-compose.yml file has the following content: 

```
version: '2'
services:
  node1:
    build: .
    container_name: n1
  node2:
    build: .
    container_name: n2
  node3:
    build: .
    container_name: n3
```

## on a single box

Here is what I get on the single box: 

```
benjguin@benjguinu1605a:~/simpletest$ docker-compose up -d
Creating network "simpletest_default" with the default driver
Creating n1
Creating n3
Creating n2
benjguin@benjguinu1605a:~/simpletest$ docker-compose ps
Name   Command   State   Ports
------------------------------
n1     init      Up
n2     init      Up
n3     init      Up
benjguin@benjguinu1605a:~/simpletest$ docker exec n1 ping -c 2 n2
PING n2 (172.19.0.4): 56 data bytes
64 bytes from 172.19.0.4: seq=0 ttl=64 time=0.086 ms
64 bytes from 172.19.0.4: seq=1 ttl=64 time=0.067 ms

--- n2 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.067/0.076/0.086 ms
benjguin@benjguinu1605a:~/simpletest$ docker exec n1 ping -c 2 n3
PING n3 (172.19.0.3): 56 data bytes
64 bytes from 172.19.0.3: seq=0 ttl=64 time=0.072 ms
64 bytes from 172.19.0.3: seq=1 ttl=64 time=0.072 ms

--- n3 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.072/0.072/0.072 ms
benjguin@benjguinu1605a:~/simpletest$ docker exec n3 ping -c 2 n1
PING n1 (172.19.0.2): 56 data bytes
64 bytes from 172.19.0.2: seq=0 ttl=64 time=0.051 ms
64 bytes from 172.19.0.2: seq=1 ttl=64 time=0.060 ms

--- n1 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.051/0.055/0.060 ms
```

## on a Swarm cluster 

Let's now do the same from the master node of my ACS cluster.

network information was added to the docker-compose.yml file: 

```
version: '2'
services:
  node1:
    build: .
    container_name: n1
    networks:
      - net34
  node2:
    build: .
    container_name: n2
    networks:
      - net34
  node3:
    build: .
    container_name: n3
    networks:
      - net34
networks:
  net34:
    driver: overlay
```

and here is the result of the test:

```
export DOCKER_HOST=172.16.0.5:2375

benjguin@swarm-master-B295EC2C-0:~$ docker-compose up -d
Creating network "benjguin_net34" with driver "overlay"
Creating n1
Creating n3
Creating n2
benjguin@swarm-master-B295EC2C-0:~$ docker inspect n1 | grep swarm-agent
            "Name": "swarm-agent-B295EC2C000001",
benjguin@swarm-master-B295EC2C-0:~$ docker inspect n2 | grep swarm-agent
            "Name": "swarm-agent-B295EC2C000000",
benjguin@swarm-master-B295EC2C-0:~$ docker inspect n3 | grep swarm-agent
            "Name": "swarm-agent-B295EC2C000000",

benjguin@swarm-master-B295EC2C-0:~$ docker exec n2 ping -c 2 n1
PING n1 (10.0.0.2): 56 data bytes
64 bytes from 10.0.0.2: seq=0 ttl=64 time=1.088 ms
64 bytes from 10.0.0.2: seq=1 ttl=64 time=0.830 ms

--- n1 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.830/0.959/1.088 ms
benjguin@swarm-master-B295EC2C-0:~$ docker exec n1 ping -c 2 n2
PING n2 (10.0.0.4): 56 data bytes
64 bytes from 10.0.0.4: seq=0 ttl=64 time=0.671 ms
64 bytes from 10.0.0.4: seq=1 ttl=64 time=0.695 ms

--- n2 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.671/0.683/0.695 ms
benjguin@swarm-master-B295EC2C-0:~$ docker exec n1 ping -c 2 n3
PING n3 (10.0.0.3): 56 data bytes
64 bytes from 10.0.0.3: seq=0 ttl=64 time=1.050 ms
64 bytes from 10.0.0.3: seq=1 ttl=64 time=0.783 ms

--- n3 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.783/0.916/1.050 ms
benjguin@swarm-master-B295EC2C-0:~$ docker exec n2 ping -c 2 n3
PING n3 (10.0.0.3): 56 data bytes
64 bytes from 10.0.0.3: seq=0 ttl=64 time=0.068 ms
64 bytes from 10.0.0.3: seq=1 ttl=64 time=0.076 ms

--- n3 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.068/0.072/0.076 ms
```

## Conclusion

We have containers in a common network which is described only thru docker means. 
This works on a single host, and also on multiple hosts (in the example, a host had 1 container, another host had 2 of the 3 containers).
