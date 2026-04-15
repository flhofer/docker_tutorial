# 4 - Optional Advanced Networking

This module is optional and intended for learners who already completed modules 1-3.
It introduces Linux host networking operations that are commonly used for industrial or low-level network scenarios.

## Scope

This module covers:

* Docker `macvlan` networks on a physical NIC
* Static container IP assignment for deterministic endpoints
* Host-to-macvlan reachability workaround
* Optional network namespace NIC passthrough flow

## Safety and prerequisites

`STOP` if these are not true:

* Linux host (not Docker Desktop networking abstraction)
* Root/sudo privileges
* Dedicated physical NIC for testing
* You can access host console in case network access is disrupted

Strong recommendation:

* Use an isolated lab VLAN or test switch
* Do not run these steps on production NICs

## Before you start (2 minutes)

Identify your test NIC and current routes:

```sh
ip link
ip -4 addr
ip route
```

In the examples below, replace `enp3s0` with your real test interface.

## Quick mental model

* `macvlan`: gives containers L2 presence on the physical segment
* Parent NIC: host physical adapter used by macvlan
* Host shim interface: extra macvlan interface to let host talk to macvlan containers
* netns passthrough: move a physical NIC into another namespace (exclusive ownership)

## Lab A: macvlan with deterministic IPs

Create a user-defined macvlan network:

```sh
docker network create \
  --driver=macvlan \
  --subnet=192.168.50.0/24 \
  --gateway=192.168.50.1 \
  -o parent=enp3s0 \
  uc4-macvlan
```

Run two containers with fixed addresses:

```sh
docker run -d --name uc4-node1 --network uc4-macvlan --ip 192.168.50.10 alpine sleep infinity
docker run -d --name uc4-node2 --network uc4-macvlan --ip 192.168.50.11 alpine sleep infinity
```

Verify:

```sh
docker inspect uc4-node1 --format '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}'
docker exec uc4-node1 ping -c 3 192.168.50.11
```

## Host-to-container reachability workaround

By default, host and macvlan containers often cannot talk directly.
Create a host shim interface:

```sh
sudo ip link add uc4-shim link enp3s0 type macvlan mode bridge
sudo ip addr add 192.168.50.254/24 dev uc4-shim
sudo ip link set uc4-shim up
```

Test from host:

```sh
ping -c 3 192.168.50.10
```

## Lab B (optional): netns NIC reassignment pattern

This is more invasive than macvlan. Use only on a dedicated lab interface.

Example flow:

```sh
PID=$(docker inspect --format '{{.State.Pid}}' uc4-node1)
sudo ip netns attach uc4-node1-ns "$PID"
sudo ip link set enp3s0 netns uc4-node1-ns
sudo ip netns exec uc4-node1-ns ip link set enp3s0 up
```

Important:

* This gives exclusive NIC ownership to the target namespace
* Host may lose direct use of that NIC until you move it back

Move interface back to host namespace:

```sh
sudo ip netns exec uc4-node1-ns ip link set enp3s0 netns 1
sudo ip link set enp3s0 up
sudo ip netns del uc4-node1-ns
```

## Reading output quickly

* `docker network inspect uc4-macvlan`: check `Driver`, `Subnet`, and connected container endpoints
* `ip link`: verify parent NIC and shim state (`UP`)
* `ip route`: verify route to macvlan subnet after shim setup

## Beginner troubleshooting

* `operation not permitted`: run network commands with `sudo`
* no host<->container ping on macvlan: create the host shim interface
* IP conflicts: ensure static IPs are outside DHCP pool
* parent NIC not found: verify interface name with `ip link`
* network disruption: move NIC back to host namespace and re-enable it

## Cleanup / rollback

```sh
docker rm -f uc4-node1 uc4-node2 2>/dev/null || true
docker network rm uc4-macvlan 2>/dev/null || true
sudo ip link del uc4-shim 2>/dev/null || true
sudo ip netns del uc4-node1-ns 2>/dev/null || true
```

## Reference material

* UC3 reference workflow:
  * https://github.com/flhofer/real-time-containers/tree/master/test-monitor/usecase/uc3
* Docker macvlan docs:
  * https://docs.docker.com/engine/network/drivers/macvlan/
* Docker networking tutorials:
  * https://docs.docker.com/engine/network/tutorials/
