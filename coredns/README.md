Install CoreDNS in Kubernetes with Ansible
==========================================

This playbook is used in my blog post [Kubernetes the not so hard way with Ansible - The worker](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-worker/). Since Kubernetes v1.11 [CoreDNS](https://coredns.io/) has reached [General Availability](https://kubernetes.io/blog/2018/07/10/coredns-ga-for-kubernetes-cluster-dns/) (GA) for Kubernetes Cluster DNS as an alternative to the kube-dns addon.

This playbook uses the Ansible's [k8s](https://docs.ansible.com/ansible/2.6/modules/k8s_module.html) module. That means you need at least Ansible v2.6 as it was added with this version (the module was formerly known as `openshift_raw` or `k8s_raw` ;-) ). This modules uses the OpenShift Python client to perform CRUD operations on Kubernetes objects. That means you need to install `openshift` pip e.g.: `pip3 install openshift` (also see [requirements](https://docs.ansible.com/ansible/2.6/modules/k8s_module.html#requirements).

Before you install and use CoreDNS for your Kubernetes cluster you may find this links useful:  
[CoreDNS GA for Kubernetes Cluster DNS](https://kubernetes.io/blog/2018/07/10/coredns-ga-for-kubernetes-cluster-dns/)  
[Customizing DNS Service](https://kubernetes.io/docs/tasks/administer-cluster/dns-custom-nameservers/)  
[Migration from kube-dns to CoreDNS](https://coredns.io/2018/05/21/migration-from-kube-dns-to-coredns/)  
[deploy.sh and coredns.yaml.sed](https://github.com/coredns/deployment/tree/master/kubernetes) This guide helps you converting from kube-dns to CoreDNS. It includes a script to generate a manifest for running CoreDNS on a cluster that is currently running standard kube-dns.  
[CoreDNS website](https://coredns.io/)
[CoreDNS Corefile Migration for Kubernetes](https://blogs.infoblox.com/community/coredns-corefile-migration-for-kubernetes/)

[configmap.yml.j2](https://github.com/githubixx/ansible-kubernetes-playbooks/blob/master/coredns/templates/configmap.yml.j2) used in this playbook contains the CoreDNS configuration file the so called `Corefile`. If you use `cluster.local` as your internal Kubernetes domain then propability is high that you can use the `Corefile` as is. Domain names that can't be resolved by CoreDNS locally are upstreamed to `1.1.1.1` ([1.1.1.1](https://1.1.1.1/) provided by Cloudflare and APNIC) and `9.9.9.9` ([Quad9](https://quad9.net/), a alternative to Google's 8.8.8.8 provided by Packet Clearing House (PCH), Global Cyber Alliance (GCA), IBM and other partners). If you want even more DNS privacy you can use [configmap_quad9_dot.yml.j2](https://github.com/githubixx/ansible-kubernetes-playbooks/blob/master/coredns/templates/configmap_quad9_dot.yml.j2). This uses [Quad9](https://quad9.net/) DNS-over-TLS.

The following CoreDNS plugins are used:  
[kubernetes](https://coredns.io/plugins/kubernetes/)  
[loop](https://coredns.io/plugins/loop/)  
[errors](https://coredns.io/plugins/errors/)  
[health](https://coredns.io/plugins/health/)  
[prometheus](https://coredns.io/plugins/metrics/)  
[proxy](https://coredns.io/plugins/proxy/)  
[cache](https://coredns.io/plugins/cache/)  
[reload](https://coredns.io/plugins/reload/)  
[loadbalance](https://coredns.io/plugins/loadbalance/)

To install all parts needed for CoreDNS into Kubernetes `kube-system` namespace run (for further options read on):

```
ansible-playbook coredns.yml
```

The Ansible [k8s](https://docs.ansible.com/ansible/2.6/modules/k8s_module.html) module reads the Kubernetes authorization information from `${HOME}/.kube/config` by default. If you've the variables `ansible_become_user: root` and/or `ansible_become: true` defined then `${HOME}` will become `/root` during execution of the playbook and the probability that the [k8s](https://docs.ansible.com/ansible/2.6/modules/k8s_module.html) module will find the Kubernetes authorization information at `/root/.kube/config` isn't that high ;-) So you may need to specify the correct user like in this example (of course replace `<username>` with the correct username):

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

