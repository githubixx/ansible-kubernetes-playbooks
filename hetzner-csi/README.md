Install Hetzner CSI driver
==========================

This playbook is used in my blog post [Kubernetes the not so hard way with Ansible - Persistent storage - Part 1](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-persistent-storage-part-1/)

This playbook uses the Ansible's [k8s](https://docs.ansible.com/ansible/2.6/modules/k8s_module.html) module. That means you need at least Ansible v2.6 as it was added with this version (the module was formerly known as `openshift_raw` or `k8s_raw` ;-) ). This modules uses the OpenShift Python client to perform CRUD operations on Kubernetes objects. That means you need to install `openshift` pip e.g.: `pip3 install openshift` (also see [requirements](https://docs.ansible.com/ansible/2.6/modules/k8s_module.html#requirements).

To install all resources needed for Hetzner CSI driver into Kubernetes `kube-system` namespace run (for further options read on):

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
