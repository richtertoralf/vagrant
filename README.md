# vagrant
erste Schritte mit vagrant

Schnell mal mit vagrant paar virtuelle Maschinen in VirtualBox erstellen. (Siehe Vagrantfile)

>Vagrant ist eine freie Ruby-Anwendung zum Erstellen und Verwalten virtueller Maschinen. Vagrant ermöglicht einfache Softwareverteilung (englisch Deployment) insbesondere in der Software- und Webentwicklung und dient als Wrapper zwischen Virtualisierungssoftware wie VirtualBox, KVM/QEMU, VMware und Hyper-V und Software-Configuration-Management-Anwendungen beziehungsweise Systemkonfigurationswerkzeugen wie Chef, Saltstack und Puppet.

Nach der Erstellung der Maschinen will ich mit Ansible die Maschinen konfigurieren und verwalten.

getestet 11/2022 auf meinem PC mit Windows 11 (AMD Ryzen 7 2700 Eight-Core Processor 3.20 GHz / Windows 11 Home Version 22H2)
installiert ist ORACLE VirtualBox 6.1

```
C:\Users\toral>git version
git version 2.37.3.windows.1

C:\Users\toral>git clone https://github.com/richtertoralf/vagrant.git

C:\Users\toral>vagrant -v
Vagrant 2.3.3

C:\Users\toral>cd vagrant

C:\Users\toral\vagrant>vagrant up
```
```
vagrant@ansiblehost:~$ 
ssh-keygen -t rsa
ssh-copy-id vagrant@192.168.50.10
ssh-copy-id vagrant@192.168.50.20
ssh-copy-id vagrant@192.168.50.30
```
`mkdir /etc/ansible`
`sudo nano /etc/ansible/hosts`
```
[servers]
machine1 ansible_host=192.168.50.10
machine2 ansible_host=192.168.50.20
machine3 ansible_host=192.168.50.30

[all:vars]
ansible_python_interpreter=/usr/bin/python3
```
`sudo nano /etc/ansible/inventory`
```
192.168.50.10
192.168.50.20
192.168.50.30
```

```
vagrant@ansiblehost:/etc/ansible$ ansible-inventory --list -y
all:
  children:
    servers:
      hosts:
        machine1:
          ansible_host: 192.168.50.10
          ansible_python_interpreter: /usr/bin/python3
        machine2:
          ansible_host: 192.168.50.20
          ansible_python_interpreter: /usr/bin/python3
        machine3:
          ansible_host: 192.168.50.30
          ansible_python_interpreter: /usr/bin/python3
    ungrouped: {}
```

```
vagrant@ansiblehost:/etc/ansible$ ansible all -m ping -u vagrant
machine3 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
machine2 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
machine1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```
