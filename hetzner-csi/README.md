Install Hetzner CSI driver
==========================

This playbook is used in my blog post [Kubernetes the not so hard way with Ansible - Persistent storage - Part 1](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-persistent-storage-part-1/)

This playbook uses the Ansible's [k8s](https://docs.ansible.com/ansible/2.6/modules/k8s_module.html) module. That means you need at least Ansible v2.6 as it was added with this version (the module was formerly known as `openshift_raw` or `k8s_raw` ;-) ). This modules uses the OpenShift Python client to perform CRUD operations on Kubernetes objects. That means you need to install `openshift` pip e.g.: `pip3 install openshift` (also see [requirements](https://docs.ansible.com/ansible/2.6/modules/k8s_module.html#requirements).

The first thing we need for the [Hetzner CSI driver](https://github.com/hetznercloud/csi-driver) is an API token. You can create it in the [Hetzner Cloud Console](https://console.hetzner.cloud/). For the token description/name you can use `hcloud-csi` as value e.g. If you have this token created we can define an Ansible variables accordingly. 

So in `group_vars/all.yml` e.g. you need to define a few variables:

```
# The Hetzner API token
hcloud_csi_token: "abcdefghijklmnopqrstuvwz"

# The namespace where all CSI resources should be installed:
hcloud_namespace: "kube-system"

# The playbook installes a few resources like:
# Secret, StorageClass, ServiceAccount, ClusterRole,
# ClusterRoleBinding, StatefulSet and DaemonSet
# All this resources have names and the names will be prefixed
# with the prefix defined here:
hcloud_resource_prefix: "hcloud"

# Make {{ hcloud_resource_prefix }}-volumes the default
# storageClass. If you only have this storageClass then
# just set it to "true". If you've other storageClass'es
# already then "false" might be your choise.
hcloud_is_default_class: "true"

# The "volumeBindingMode" field controls when volume binding and
# dynamic provisioning should occur. The default value is "Immediate".
# The "Immediate" mode indicates that volume binding and dynamic
# provisioning occurs once the "PersistentVolumeClaim" is created.
# The "WaitForFirstConsumer" mode which will delay the binding and
# provisioning of a "PersistentVolume" until a Pod using the
# "PersistentVolumeClaim" is created. "PersistentVolumes" will be
# selected or provisioned conforming to the topology that is specified
# by the Pod’s scheduling constraints. These include, but are not
# limited to, resource requirements, node selectors, pod affinity
# and anti-affinity, and taints and tolerations.
hcloud_volume_binding_mode: "WaitForFirstConsumer"

# Directory where kubelet configuration is located
k8s_worker_kubelet_conf_dir: "/var/lib/kubelet"

# Versions/tags for the various container images needed. Besides
# the "csi_driver" you can find out about the recommened version
# of the container to use for a specific Kubernetes release in the
# CSI documentation: https://kubernetes-csi.github.io

# DaemonSet:
hcloud_csi_node_driver_registrar: "1.3.0"

# StatefulSet:
hcloud_csi_attacher: "2.2.0"
hcloud_csi_provisioner: "1.6.0"
hcloud_csi_resizer: "0.5.0"

# Hetzner CSI driver
hcloud_csi_driver: "1.1.4"
```

To install all resources needed for [Hetzner CSI driver](https://github.com/hetznercloud/csi-driver) run (for further options read on):

```
ansible-playbook hetzner-csi.yml
```

The Ansible [k8s](https://docs.ansible.com/ansible/2.6/modules/k8s_module.html) module reads the Kubernetes authorization information from `${HOME}/.kube/config` by default. If you've the variables `ansible_become_user: root` and/or `ansible_become: true` defined then `${HOME}` will become `/root` during execution of the playbook and the probability that the [k8s](https://docs.ansible.com/ansible/2.6/modules/k8s_module.html) module will find the Kubernetes authorization information at `/root/.kube/config` isn't that high ;-) So you may need to specify the correct user like in this example (of course replace `<username>` with the correct username):

```
ansible-playbook --become-user=<username> hetzner-csi.yml
```

All resources have a `tag` so e.g. if you only want to install the Kubernetes `DaemonSet` needed for Hetzner CSI driver use this command:

```
ansible-playbook --tags=install-daemonset hetzner-csi.yml
```

Have a look at `tasks/install.yml` and `tasks/delete.yml` what tags are available (Sadly `ansible-playbook --list-tasks hetzner-csi.yml` doesn't show tags included with `import_tasks`). To delete all Hetzner CSI driver resources use

```
ansible-playbook -e delete=true hetzner-csi.yml
```

To only delete a specific resource you can also use tags e.g.:

```
ansible-playbook -e delete=true --tags=delete-daemonset hetzner-csi.yml
```

Upgrading
---------

I've written a [blog post](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-csi-upgrade-notes/) how to upgrade CSI resources.
