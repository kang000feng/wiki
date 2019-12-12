# Ansible

## What is Ansible

>  Ansible is a radically simple IT automation system. It handles configuration management, application deployment, cloud provisioning, ad-hoc task execution, network automation, and multi-node orchestration. Ansible makes complex changes like zero-downtime rolling updates with load balancers easy. [1]

It seems that ansible focus on application publishing and updates, with some features of load balancing and service orchestration, which seems to be a subset of k8s. These functions can be provided by k8s, but many companies are currently using ansible. Maybe some places are different. Let’s take a closer look.



## Install Ansible

If you use MacOS or Linux, then you can follow the installation guide to install ansible, there are many ways to install it on your machine. My server use Ubuntu, So the command I executed is as follows.

```shell
$ sudo apt update
$ sudo apt install software-properties-common
$ sudo apt-add-repository --yes --update ppa:ansible/ansible
$ sudo apt install ansible
```



## Config

I can find `ansible.cfg` at `/etc/ansible/ansible.cfg` because I install ansible with apt-get, if you use other installation ways, you can read  Configuring Ansible[3] to find the config file.



## Reference

1. Ansible GitHub:  https://github.com/ansible/ansible 
2. Installation Guide: https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html 
3. 