## What are these ansible playbooks used for ?
- setting up a network bridge
- provisioning a virtual machine
- runing the virtual machine
- configuring some softwares on the newly created machine

Warning: This playbooks are used in Arch Linux

### Setting up a network bridge 
Before you run the playbook
```
ansible-playbook -K net/setup-bridge.yml
```
you need to change the value of variables in net/setup-bridge.yml
```
eth_original_ip: 192.168.0.140 # change this to the IP of your host machine  
eth_new_ip: 192.168.0.141  # setup a new IP for your host machine  
br_name: devops0 # the name of the new network bridge   
br_ip: "{{eth_original_ip}}" # let the original IP of your host machine be that of the new bridge   
br_mask: 24   
eth_if: enp34s0  # the network interface of your host machine   
gw: 192.168.0.1 # the IP of your router  

```
Use ip command to check the bridge info
```
ip addr show
```

### Provisioning a virtual machine  
vm/provision.yml   
```
#cloud init config 
cloudinit_userdata:
   user_name: "your_user_name"
   group_name: "your_group_name"
   user_pwd: "your_password"
   ssh_public_key: "your_password"
#qcow2
qcow2_file_size: "10G" # qcow2 disk size
qcow2_file_name: "devops_ubuntu7.qcow2" # qcow2 disk file name
qcow2_recreate: true # recreate a qcow2 disk file if an old qcow2 disk file exists, otherwise, this playbook will quit.
#ip,tap,mac_addr
gateway_ip: 192.168.0.1 # router IP
instance_bind_ip: 192.168.0.147 # static IP for the new vm  
instance_mac_addr: "DE:AD:BE:EF:B1:67" # Mac address for the new vm  
bridge_name: devops0 # before provisioning a new vm, you need to setup a virtual bridge
tap_name: devops_tap7 # a virtual tap  name for the new vm
```
```
instance_name: devops_ubuntu7 # name for the name option of qemu-system-x86_64 command, server name of ansible inventory, playbook name to run the created vm
ansible-playbook -K vm/provision.yml --extra-vars "instance_name=devops_ubuntu7"
```
### Runing the newly created machine  
```
ansible-playbook -K vm/run/{{instance_name}}.yml
```
Use SSH to Connect to the vm 
```
ssh -o StrictHostKeyChecking=no ryzen@192.168.0.147 -i ssh/rsa.key
```
Shutdown the running vm
```
ansible-playbook -K vm/run/shutdown.yml  -i host/{{instance_name}}.host --private-key ssh/rsa.key
```

### configuring some softwares on the newly created machine  

```
ansible-playbook -K dev-setup/setup-dev-tools.yml -i host/{{instance_name}}.host --private-key ssh/rsa.key
```