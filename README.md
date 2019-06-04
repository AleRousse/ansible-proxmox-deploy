Proxmox Deploy
=========

Creates virtual machines with Proxmox and installs (unattended) a distribution on it.
=========
This role is based on Inoxio proxmox_vms (https://github.com/inoxio/ansible-role-proxmox-vms) 

Role Variables
--------------
Available variables are listed below, along with default values (see defaults/main.yml for more informations):
```
#Basic VM settings
type: vm
hypervisor: pve
node: prmx0
memory_size: 2048
balloon: '1024'
cpu: "host"
cores: '2'
net: '{"net0":"virtio,bridge=vmbr0,tag=10,firewall=1"}'
virtio: '{"virtio0":"local-lvm:24,cache=writeback,discard=on"}'
sscsihw: 'virtio-scsi-pci'
ostype: 'l26'
onboot: 'yes'

#Distribution and mirror
distribution: 'debian-stretch'
mirror: debian.mirror.rafal.ca

#Language and timezone
locale: en_US
timezone: America/Toronto

#User
user_username: your_user
user_password: <user_password> #vault encrypted

#Network
network:
  ip: 192.168.1.2
  netmask: 255.255.255.0 
  gateway: 192.168.1.1
domainname: yourdomain.com
nameserver: 1.1.1.1
ntp_pools:
  - 0.ca.pool.ntp.org
  - 1.ca.pool.ntp.org
  - 2.ca.pool.ntp.org
  
```
  *  `ansible_hostname`: (Required/No Defaults) Define the vm name in proxmox and in the vm 
  #### Basic VM settings
  *  `type`: (Required/No Defaults)
  *Only proxmox virtual environnement is supported for now*
  *  `hypervisor`: (Required/No Defaults) Specifies the hypervisor type from the following: `pve`  
  * `node`: (Required/No Defaults) Specifies the name of the node on the Proxmox server. Under the
node the VM will be installed.
  * `memory_size`: (Default: 2048) Specifies the size of memory in MB for the VM.
  * `ballooon`: (Default: 1024) The virtio balloon device allows KVM guests to reduce their memory size (thus relinquishing memory to the host) and  to increase it back (thus taking memory from the host).
  * `cpu` (Default: host) Specifies the cpu type 
  * `cores`: (Default: 2) Specifies the number of cores of the VM
  * `net`: (Default: {"net0":"virtio,bridge=vmbr0"}) Specifies a hash/dictionary of network interfaces for the VM. net='{"key":"value", "key":"value"}'.
    * `virtio` choosen type of interface
    * `bridge` Specifies the interface to bridge
    * `tag` Specifies a vlan id tag
    * `firewall` (1-On/0-Off) Speicifes if the proxmox's firewall  should be enable for this VM.
  * `virtio` (Default: {"virtio0":"local-lvm:10,cache=writeback,discard=on"}) A hash/dictionary of volume used as VIRTIO hard disk. virtio='{"key":"value", "key":"value"}'
    * `local-lvm`: Size of the disk in GB
  * `sscsihw`: (Default: virtio-scsi-pci) Specifies the SCSI controller model.
  * `ostype`: (Default: l26) Specifies the os type of the VM
  * `onboot`: (Default: yes) Sepcifies if the vm should boot when the host start
  #### Distribution and Mirror
  * `distribution`: (Default: debian-stretch) Specifies the distribution to be installed on the VM from the following list: `debian-stretch, ubuntu-bionic`
  * `mirror`: (Default: debian.mirror.rafal.ca) Specifies the mirror use to fetch package
  #### Language and Timezone
  * `locale`: (Default: en_US) Locale is a set of parameters that defines the user language. Need to be set in the following format language_contry
  * `timezone`: (Default: America/New York) Specifies the timezone of the VM
  #### User
  * `user_username`: (Required/No Defaults) Specifies the username of a root account.
  * `user_password`: (Required/No Defaults)  Specifies the password of a root account.
  #### Nerwork
  * `network`:
      * `ip`:(Required/No Defaults)  Specifies the ip address of the VM.
      * `netmask`:(Required/No Defaults)  Specifies the netmask to be used by the VM.
      * `gateway`:(Required/No Defaults)  Specifies the gateway ip to be used by the VM.
  * `nameserver`: (Required/No Defaults) Specifies the nameserver ips to be used bthe VM.
  * `domainname`:(Required/No Defaults)  Specifies the domain name of the VM. 
  * `ntp_pools`: (Defaults: US ntp servers: 0.us.pool.ntp.org) Specifies the pool of ntp servers to use as a list. Because of preseed limitation only one ntp can be use. The first one in the list will be use.
  * `scripts`: (Optional/No Default) Scripts is a list of filewhoconten
will be inserted into the finish-installation file.
              E.g. copying ssh keys from AW

Example Playbook
----------------
```
- hosts: proxmox # see hosts file
  remote_user: root

  #This is necessary because proxmoxer need python3 
  vars:
    ansible_python_interpreter: /usr/bin/python3


  roles:
    - role: proxmox_deploy
      proxmox:
        api_user: 
        api_password:
        api_host:
```
 * `proxmox`: Contains login data for the Proxmox server which should bencrypted. You need a file which contains  
             the password for the Ansible vault.
             Encrypting is done by the following command on the terminal:  
             `ansible-vault encrypt_string --vault-i<path_to_the_password_file> '<password>' --nam'<variable_name>'`  
             Example: `ansible-vault encrypt_string --vault-id .ansible.secret 'some_password' --name 'api_password'`
    * `api_user`: Specifies the user-name for the Proxmox login.
    * `api_password`: Specifies the password of a user for Proxmox login.
    * `api_host`: Specifies the host name or ip of the Proxmox server.
  
Example deploy command
----------------
```
$ ansible-playbook playbooks/proxmox.yml -i inventories/production --ask-pass --ask-vault -K
```
Or specify a inventory group to be deploy **by default all hosts will be deployed**
```
ansible-playbook playbooks/proxmox.yml -i inventories/production --ask-pass --ask-vault -K --extra-vars "vmsgroup=webserver"
```

Testing
--------------
Automatic testing this role is difficult because you need a VM with Proxmox on it which creates the VMs within the VM itself. Thus a hypervisor is needed which supports nested virtualization.

License
-------
Apache 2.0