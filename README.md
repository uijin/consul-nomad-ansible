# Ansible playbook to deploy Consul and Nomad

This project demonstrate how to deploy Consul and Nomad quickly by using Ansible.

## Prerequisite

Operations would be taken on a hosts which includes

- [Multipass](https://multipass.run/)
- SSH client

Below are the files which be provided by yourself

- [SSH identity public key](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)

## Provision VM

Put your SSH identity public key as file `id_ed25519.pub`. Execute one of the following commands to create VM and setup VM for Ansible execution 

```bash
ansible-playbook setup-multipass.yaml
```

When complete VM setup, execute the following command to check readiness.

```bash
ansible-playbook setup-consul.yaml -i hosts --tags check-ssh
```

## Setup Consul Cluster

Execute below command to install, setup and start Consul cluster.

```bash
ansible-playbook setup-consul.yaml -i hosts --become
```

If necessary, launch Consul UI on port 8500 in first VM

```bash
ansible-playbook setup-consul.yaml -i hosts --become --tags consul-ui,never
```

## Setup Nomad Server

Execute below command to install, setup and start Nomad server cluster. These Nomad servers are running on the same VMs which host Consul service.

```bash
ansible-playbook setup-nomad.yaml -i hosts --become
```

Nomad cluster would ready after a while.

## Provision Nomad Client VM

Execute one of the following commands to create VM and setup VM for Ansible execution 

```bash
ansible-playbook setup-multipass-nomad-client.yaml
```

When complete VM setup, execute the following command to check readiness.

```bash
ansible-playbook setup-nomad-client.yaml -i hosts-client --tags check-ssh
```

## Setup Nomad Client

Execute below command to install, setup and start Nomad client node. These Nomad servers are running on the same VMs which host Consul service. There are two Ansible inventory files specified in command arguments.

```bash
ansible-playbook setup-nomad-client.yaml -i hosts -i hosts-client --become
```

## Run Example Job: hello-nomad

Copy the example job file `hello-nomad/hello-nomad.hcl` to the host which Nomad client executed. For example, host `nomad-client-01` is the host, and 192.168.64.20 is the IP address of the host.

``` bash
scp hello-nomad/hello-nomad.hcl ubuntu@192.168.64.20:
```

Login the host and submit hello-nomad job

``` bash
ssh ubuntu@192.168.64.20
ubuntu@nomad-client-01:~$ nomad job run hello-nomad.hcl
```

Show the job status, check whether job is running or not in `Allocations` section.

``` shell
ubuntu@nomad-client-01:~$ nomad job status hello-nomad
ID            = hello-nomad
Name          = hello-nomad
Submit Date   = 2022-02-20T08:18:49+08:00
Type          = service
Priority      = 50
Datacenters   = dc1
Status        = running
Periodic      = false
Parameterized = false

Summary
Task Group  Queued  Starting  Running  Failed  Complete  Lost
services    0       0         1        7       1         0

Allocations
ID        Node ID   Task Group  Version  Desired  Status    Created     Modified
2cf480df  e6f2a917  services    3        run      running   51s ago     37s ago
```

If the job is running, get service IP:PORT of it.

``` shell
ubuntu@nomad-client-01:~$ nomad job status hello-nomad | grep services | grep running | awk '{print $1}' | xargs -n 1 nomad status | grep -A 1 Addresses
CPU        Memory           Disk     IOPS  Addresses
0/100 MHz  7.6 MiB/100 MiB  300 MiB  0     http: 192.168.64.20:27248
```

Finally, access the service by HTTP request. It would be great to see **Hello Nomad !** on the screen :D

``` shell
ubuntu@nomad-client-01:~$ curl http://192.168.64.20:27248 && echo 
Hello Nomad !
```
