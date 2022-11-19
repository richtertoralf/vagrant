# vagrant
erste Schritte mit vagrant

Schnell mal mit vagrant paar virtuelle Maschinen in VirtualBox erstellen. (Siehe Vagrantfile)  
Ziel ist eine Laborumgebung zum Testen und Üben.

>Vagrant ist eine freie Ruby-Anwendung zum Erstellen und Verwalten virtueller Maschinen. Vagrant ermöglicht eine einfache Softwareverteilung (englisch Deployment) insbesondere in der Software- und Webentwicklung und dient als Wrapper zwischen Virtualisierungssoftware wie VirtualBox, KVM/QEMU, VMware und Hyper-V und Software-Configuration-Management-Anwendungen beziehungsweise Systemkonfigurationswerkzeugen wie Chef, Saltstack und Puppet.

Nach der Erstellung der Maschinen mittels Vagrant will ich mit Ansible die Maschinen konfigurieren und verwalten.
Ansible soll auf einer Linux-Maschine mit dem Hostname "ansiblehost" und der IP 192.168.50.100 in meinem privaten Netzwerk "mynetwork" laufen. Dies wird mit den folgenden Zeilen im Vagrantfile konfiguriert:
```
    anshost.vm.network "private_network", ip: "192.168.50.100", virtualbox__intnet: "mynetwork"
    anshost.vm.hostname = "ansiblehost"
```

getestet 11/2022 auf meinem PC mit Windows 11 (AMD Ryzen 7 2700 Eight-Core Processor 3.20 GHz / Windows 11 Home Version 22H2)
installiert ist ORACLE VirtualBox 6.1

```
C:\Users\toral>git version
git version 2.37.3.windows.1

C:\Users\toral>git clone https://github.com/richtertoralf/vagrant-ansible.git

C:\Users\toral>vagrant -v
Vagrant 2.3.3

C:\Users\toral>cd vagrant-ansible

C:\Users\toral\vagrant-ansible>vagrant up
```

# ansible
erste Schritte mit ansible
## SSH Zugriff einrichten und ping testen
```
vagrant@ansiblehost:~$ sudo nano /etc/hosts
# insert:
192.168.50.10 machine1
192.168.50.20 machine2
192.168.50.30 machine3
```
```
vagrant@ansiblehost:~$ 
ssh-keygen -t rsa
ssh-copy-id vagrant@192.168.50.10
ssh-copy-id vagrant@192.168.50.20
ssh-copy-id vagrant@192.168.50.30
```
```
vagrant@ansiblehost:~$
sudo apt install sshpass
mkdir -p ~/.ssh
ssh-keyscan -t rsa machine1 machine2 machine3 > ~/.ssh/known_hosts
```
erster Test mit Ansible-ping, ob die Zielmachinen erreichbar sind:
```
ansible all -i machine1,machine2,machine3 -u vagrant -m ping -k
machine1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
machine3 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
machine2 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```
## Erstellen eines inventory
`mkdir /etc/ansible`  
`sudo nano /etc/ansible/hosts`  
```
[my_machines]
machine1 ansible_host=192.168.50.10
machine2 ansible_host=192.168.50.20
machine3 ansible_host=192.168.50.30

# [all:vars]
# ansible_python_interpreter=/usr/bin/python3
```
```
vagrant@ansiblehost:~$ ansible all --list-hosts
  hosts (3):
    machine1
    machine2
    machine3
```
```
vagrant@ansiblehost:~$ ansible all -m ping
machine2 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
machine1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
machine3 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

```
vagrant@ansiblehost:~$ ansible-inventory --list -y
all:
  children:
    my_machines:
      hosts:
        machine1:
          ansible_host: 192.168.50.10
        machine2:
          ansible_host: 192.168.50.20
        machine3:
          ansible_host: 192.168.50.30
    ungrouped: {}
```

```
vagrant@ansiblehost:~$ ansible all -m ping -u vagrant
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

## Playbook
Ich nutze meine schon vorhandene inventory-Datei `/etc/ansible/hosts`:
```
[my_machines]
machine1 ansible_host=192.168.50.10
machine2 ansible_host=192.168.50.20
machine3 ansible_host=192.168.50.30

# [all:vars]
# ansible_python_interpreter=/usr/bin/python3
```
Erstelle aber im Userverzeichnis von `vagrant` zuerst einen neuen Ordner, kopiere meine inventory-Datei dorthin und lege dort meine neuen Playbooks ab. 
```
cd ~
mkdir ansible-playbooks
cd ansible-playbooks/
cp /etc/ansible/hosts ~/ansible-playbooks/inventory
```
### Hallo Welt
```
cd ~/ansible-playbooks
nano playbook-helloworld.yml
```
und Folgenden Inhalt in playbook-helloworld.yml einfügen:
```
- hosts: all
  tasks:
    - name: Print message
      debug:
        msg: Hello Ansible World
```
Playbook ausführen:
```
ansible-playbook -i inventory playbook-helloworld.yml -u vagrant
```
