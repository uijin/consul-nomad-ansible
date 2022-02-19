# Ansible playbook to deploy Consul and Nomad

## Provision VM

Put your SSH identity public key as file `id_ed25519.pub`. Execute one of the following commands to create VM and setup VM for Ansible execution 

- ```ansible-playbook setup-multipass.yaml```

When complete VM setup, execute the following command to check readiness.

```
ansible-playbook setup-consul.yaml -i hosts --tags check-ssh
```

## Setup Consul Cluster

Execute below command to install, setup and start Consul cluster.

```
ansible-playbook setup-consul.yaml -i hosts --become
```

If necessary, launch Consul UI on port 8500 in first VM

```
ansible-playbook setup-consul.yaml -i hosts --become --tags consul-ui,never
```

## Setup Nomad Server

Execute below command to install, setup and start Nomad server cluster. These Nomad servers are running on the same VMs which host Consul service.

```
ansible-playbook setup-nomad.yaml -i hosts --become
```

Nomad cluster would ready after a while.

## Provision Nomad Client VM

Execute one of the following commands to create VM and setup VM for Ansible execution 

- ```ansible-playbook setup-multipass-nomad-client.yaml```

When complete VM setup, execute the following command to check readiness.

```
ansible-playbook setup-nomad-client.yaml -i hosts-client --tags check-ssh
```

## Setup Nomad Client

Execute below command to install, setup and start Nomad client node. These Nomad servers are running on the same VMs which host Consul service. There are two Ansible inventory files specified in command arguments.

```
ansible-playbook setup-nomad-client.yaml -i hosts -i hosts-client --become
```

## Run a example application

WIP
