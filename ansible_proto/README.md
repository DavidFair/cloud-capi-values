Ansible Prototype for driving CAPI Cluster Creation
===================================================

This is a prototype for driving the creation of a CAPI cluster using Ansible.

Setup:
======

Requirements:
-------------

- A clouds.yaml file with application credentials in inventory/clouds.yaml , the project ID does not need to be added.
- A clouds.yaml file in ~/.config/openstack/clouds.yaml such that `openstack --os-cloud openstack server list` works.
- Ansible and Ansible Galaxy

Prep
----

- Set a venv with `python3.9 -m venv venv`
- Activate the venv with `source venv/bin/activate`
- Install the requirements with `pip install -r requirements.txt`
- Install the Ansible Galaxy requirements with `ansible-galaxy install -r requirements.yml`

Configuration
-------------

- Allocate two floating IPs for the worker and management clusters and add them to inventory/group_vars/all.yml
- Add the name of your SSH key to inventory/group_vars/all.yml
- Set the cluster names in inventory/group_vars/all.yml

This can be run in one of two modes:

With a seed node
-----------------

This mode creates a VM automatically to drive the creation of the cluster. This is the preferred mode.

| Ansible Machine | -> | Seed Node (K8s + CAPI) | -> | Cluster Nodes (K8s) |

To do this the inventory/static-inventory file needs to have a definition for the local
machine. This is shipped by default.

```
[local]
localhost  ansible_connection=local
```

- Run `ansible-playbook playbooks/deploy_seed.yml` to deploy the seed node and k3s.
- Run `ansible-playbook playbooks/mgmt_cluster.yml` to deploy a self-managed cluster.
- Run `ansible-playbook playbooks/worker_cluster.yml` to deploy a managed cluster.`

On this Machine
---------------

Or you can run with the Ansible Machine and Seed Node on the same machine:
| Ansible + K8s | -> | Cluster Nodes (K8s) |

This requires you to have a K8s cluster already running on the machine.

- In static inventory add the following:
```
[seed]
localhost  ansible_connection=local
```
- Run `ansible-playbook playbooks/mgmt_cluster.yml` to deploy a self-managed cluster.
- Run `ansible-playbook playbooks/worker_cluster.yml` to deploy a managed cluster.`

Kubeconfigs
===========

A copy of each kubeconfig is copied into inventory/.kube for safety.

Destroying Cluster
==================

Each resource can be destroyed with the `destroy` tag:

This will destroy the seed VM:
- `ansible-playbook playbooks/deploy_seed.yml --tags destroy`

This will destroy the worker cluster:
- `ansible-playbook playbooks/worker_cluster.yml --tags destroy`

This will partially clean up the management cluster (but will not remove the worker cluster which would be orphaned if it exists)
- `ansible-playbook playbooks/mgmt_cluster.yml --tags destroy`