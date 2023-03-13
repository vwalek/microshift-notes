# How to run the etcdctl in container to check the status of the etcd.

### Issue
The `etcdctl` is not present currently on the microshift. If you need to run the etcdctl commands to backup or to check the state, you can run it as container.

### Prerequisities

- run the commands as root
- having pull-secret correctly configured
- having access to the internet or ability to pull and transfer the etcd image to the microshift host
- install podman

### Steps

1. Pull the image

~~~
# podman pull --authfile=/etc/crio/openshift-pull-secret registry.redhat.io/openshift4/ose-etcd:latest

Trying to pull registry.redhat.io/openshift4/ose-etcd:latest...
Getting image source signatures
Checking if image destination supports signatures
Copying blob d8190195889e skipped: already exists  
Copying blob 8b9370cb5925 done  
Copying blob 57e3e9b4305a done  
Copying blob 97da74cc6d8f skipped: already exists  
Copying blob b8c6fc8656cd done  
Copying config 7b298a4967 done  
~~~

2. Start the container

~~~
#  podman run --authfile=/etc/crio/openshift-pull-secret -it --rm --privileged \
     --network=host -v /var/lib/microshift/etcd:/var/lib/etcd -v /var/lib/microshift/certs/etcd-signer:/etc/etcd/ \
     --entrypoint="/bin/bash" registry.redhat.io/openshift4/ose-etcd:latest

[root@localhost ~]#
~~~

3. Check that you have access to the host network
~~~
# ip addr sh br-ex

7: br-ex: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN group default qlen 1000
    link/ether 52:54:00:7d:57:6f brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.138/24 brd 192.168.122.255 scope global dynamic noprefixroute br-ex
       valid_lft 3478sec preferred_lft 3478sec
    inet 169.254.169.2/29 brd 169.254.169.7 scope global br-ex
       valid_lft forever preferred_lft forever
    inet6 fe80::8282:8f0f:912c:ada4/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
~~~

4. Run the etcdctl command

~~~
# etcdctl --cacert /etc/etcd/ca.crt --cert /etc/etcd/etcd-peer/peer.crt \
  --key /etc/etcd/etcd-peer/peer.key --endpoints="https://localhost:2379" endpoint status -w table

+------------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
|        ENDPOINT        |        ID        | VERSION | DB SIZE | IS LEADER | IS LEARNER | RAFT TERM | RAFT INDEX | RAFT APPLIED INDEX | ERRORS |
+------------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
| https://localhost:2379 | 422356ad776e29f5 |   3.5.3 |  2.8 MB |      true |      false |       736 |      31383 |              31383 |        |
+------------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
~~~

5. You can run the `snapshot save`

~~~
# etcdctl --cacert /etc/etcd/ca.crt --cert /etc/etcd/etcd-peer/peer.crt \
  --key /etc/etcd/etcd-peer/peer.key --endpoints="https://localhost:2379" snapshot save /var/lib/etcd/backup.db
~~~
