# Ansible playbook to deploy Consul and Nomad

## Host(VM) Preparation

Put your SSH identity public key as file `id_ed25519.pub`. Execute one of the following commands to create VM and complete VM setup for ansible execution 

- ```ansible-playbook setup-consul.yaml```

When complete VM setup, execute the following command to check readiness.

```ansible-playbook setup-consul.yaml -i hosts```
