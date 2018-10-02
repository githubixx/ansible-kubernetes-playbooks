Install CoreDNS in Kubernetes with Ansible
==========================================

This playbook is used in my blog post [Kubernetes the not so hard way with Ansible - The worker](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-worker/). Since Kubernetes v1.11 [CoreDNS](https://coredns.io/) has reached [General Availability](https://kubernetes.io/blog/2018/07/10/coredns-ga-for-kubernetes-cluster-dns/) (GA) for Kubernetes Cluster DNS as an alternative to the kube-dns addon.

This role uses the Ansible's [k8s](https://docs.ansible.com/ansible/2.6/modules/k8s_module.html) module. That means you need at least Ansible v2.6 as it was added with this version (the module was formerly known as `openshift_raw` or `k8s_raw` ;-) ). This modules uses the OpenShift Python client to perform CRUD operations on Kubernetes objects. That means you need to install `openshift` pip e.g.: `pip3 install openshift` (also see [requirements](https://docs.ansible.com/ansible/2.6/modules/k8s_module.html#requirements).

To install all parts needed for CoreDNS into Kubernetes `kube-system` namespace run:

```
ansible-playbook coredns.yml
```

The [k8s](https://docs.ansible.com/ansible/2.6/modules/k8s_module.html) module reads the Kubernetes authorization information from `${HOME}/.kube/config` by default. If you've the variables `ansible_become_user: root` or `ansible_become: true` defined then `${HOME}` will become `/root` and the probability that the [k8s](https://docs.ansible.com/ansible/2.6/modules/k8s_module.html) module will find the Kubernetes authorization information at `/root/.kube/config` isn't that high ;-) So you may need to specify the correct user like in this example (of course replace `<username>` with the correct username):

```
ansible-playbook --become-user=<username> coredns.yml
```

All parts have a `tag` so e.g. if you only want to install the Kubernetes `Service` needed for CoreDNS use this command:

```
ansible-playbook --tags=install-service coredns.yml
```

Have a look at `tasks/install.yml` and `tasks/delete.yml` what tags are available (Sadly `ansible-playbook --list-tasks coredns.yml` doesn't show tags included with `import_tasks`). To delete all CoreDNS resources use

```
ansible-playbook -e delete=true coredns.yml 
```

To only delete a specific resource you can also use tags e.g.:

```
ansible-playbook -e delete=true --tags=delete-service coredns.yml 
```

