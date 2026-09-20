# Network Engineering Automation Lab

## Project Overview

This project is a containerized network engineering lab built with **Containerlab, FRRouting, Ansible, and Jinja2**.

The project demonstrates how a small multi-router network can be deployed, configured, and verified using network automation rather than manually configuring each router.

The lab consists of three FRRouting routers running OSPF in a full-mesh topology.

## Architecture

```text
                 R1
              /      \
             /        \
          R2 ---------- R3

        OSPF Area 0
```

The topology contains three Layer 3 routers:

* R1
* R2
* R3

Each router has two point-to-point links and a loopback interface.

## Technologies

* Linux / WSL2
* Docker
* Containerlab
* FRRouting (FRR) 10.4.1
* Ansible
* Jinja2
* YAML
* Git

## Network Topology

| Router | Interface | IP Address   | Connection |
| ------ | --------- | ------------ | ---------- |
| R1     | eth1      | 10.0.0.1/30  | R2 eth1    |
| R1     | eth2      | 10.0.0.9/30  | R3 eth2    |
| R1     | lo        | 1.1.1.1/32   | Loopback   |
| R2     | eth1      | 10.0.0.2/30  | R1 eth1    |
| R2     | eth2      | 10.0.0.5/30  | R3 eth1    |
| R2     | lo        | 2.2.2.2/32   | Loopback   |
| R3     | eth1      | 10.0.0.6/30  | R2 eth2    |
| R3     | eth2      | 10.0.0.10/30 | R1 eth2    |
| R3     | lo        | 3.3.3.3/32   | Loopback   |

## OSPF

OSPF is configured as the dynamic routing protocol.

All routers operate in **OSPF Area 0**.

Each router uses its loopback address as the OSPF router ID:

| Router | OSPF Router ID |
| ------ | -------------- |
| R1     | 1.1.1.1        |
| R2     | 2.2.2.2        |
| R3     | 3.3.3.3        |

OSPF cost is configured on the point-to-point interfaces.

The resulting topology provides multiple paths between the routers.

## Ansible Automation

Ansible is used to automate router configuration.

The automation workflow is:

```text
Ansible Variables
       |
       v
Jinja2 Template
       |
       v
FRR Configuration
       |
       v
Containerlab
       |
       v
FRRouting Routers
       |
       v
OSPF
```

The main configuration playbook is:

```text
ansible/configure.yml
```

It generates the FRR configuration files for all routers and automatically reconfigures Containerlab when a configuration changes.

## Jinja2 Configuration Generation

The template:

```text
ansible/frr.conf.j2
```

is used to generate router-specific FRR configurations.

Router-specific parameters are stored in:

```text
ansible/variables.yml
```

This separates configuration data from the configuration template.

For example, router addressing and OSPF parameters are defined in the variables file rather than being hard-coded separately for every router.

## Inventory

The Ansible inventory is:

```text
ansible/inventory.yml
```

It defines the three routers:

```text
r1
r2
r3
```

and maps them to their corresponding Containerlab containers.

## Automated Verification

The verification playbook:

```text
ansible/playbook-verification.yml
```

automatically checks:

* OSPF neighbor relationships
* Routing tables
* Loopback connectivity
* Connectivity between all routers

The verification performs connectivity tests between all three loopbacks:

```text
R1 -> 1.1.1.1
R1 -> 2.2.2.2
R1 -> 3.3.3.3

R2 -> 1.1.1.1
R2 -> 2.2.2.2
R2 -> 3.3.3.3

R3 -> 1.1.1.1
R3 -> 2.2.2.2
R3 -> 3.3.3.3
```

All tested connections completed successfully with **0% packet loss**.

## OSPF Verification

The lab successfully established full OSPF adjacencies.

Each router sees the other two routers as OSPF neighbors.

Example:

```text
R1
 |
 +-- R2  Full
 |
 +-- R3  Full
```

The routing tables also contain OSPF-learned routes to the remote loopbacks.

## Idempotency

The configuration playbook was executed multiple times without changing the desired configuration.

The final execution reported:

```text
r1 changed: False
r2 changed: False
r3 changed: False
```

Containerlab reconfiguration was therefore skipped.

This demonstrates **idempotent configuration management**: running the automation repeatedly does not create unnecessary changes.

## Deployment

Start the lab with:

```bash
sudo containerlab deploy --topo topology.yml
```

The routers can then be inspected with Docker:

```bash
sudo docker ps
```

## Configure the Network

Run:

```bash
ansible-playbook -i ansible/inventory.yml ansible/configure.yml
```

For a syntax check:

```bash
ansible-playbook -i ansible/inventory.yml ansible/configure.yml --syntax-check
```

## Verify the Network

Run:

```bash
ansible-playbook -i ansible/inventory.yml ansible/playbook-verification.yml
```

Manual OSPF verification can also be performed with:

```bash
sudo docker exec clab-network-engineering-lab-r1 vtysh -c "show ip ospf neighbor"
```

Routing table:

```bash
sudo docker exec clab-network-engineering-lab-r1 vtysh -c "show ip route"
```

## Project Structure

```text
network-engineering-project/
│
├── .gitignore
├── README.md
├── topology.yml
│
├── ansible/
│   ├── configure-generate.yml
│   ├── configure.yml
│   ├── frr.conf.j2
│   ├── inventory.yml
│   ├── playbook-verification.yml
│   ├── playbook.yml
│   └── variables.yml
│
└── configs/
    ├── r1/
    │   ├── daemons
    │   ├── frr.conf
    │   └── vtysh.conf
    │
    ├── r2/
    │   ├── daemons
    │   ├── frr.conf
    │   └── vtysh.conf
    │
    └── r3/
        ├── daemons
        ├── frr.conf
        └── vtysh.conf
```

## Skills Demonstrated

This project demonstrates practical experience with:

* Network engineering
* Dynamic routing
* OSPF
* IP addressing
* Linux networking
* Containerlab
* Docker
* FRRouting
* Ansible
* Jinja2
* Configuration templating
* Infrastructure automation
* Network verification
* Idempotent automation
* Git-based project management

## Future Improvements

Possible extensions include:

* BGP configuration
* Ansible automated fault detection
* Automated OSPF metric changes
* Network state validation
* CI/CD pipeline for configuration testing
* Nornir integration
* Network telemetry
* Prometheus/Grafana monitoring
* Automated configuration backup
* Automated topology testing

## Project Status

**Completed**

The current implementation successfully deploys a three-router FRRouting topology, establishes OSPF adjacencies, distributes loopback routes, verifies end-to-end connectivity, and automates configuration using Ansible and Jinja2.
