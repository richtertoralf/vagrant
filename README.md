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
Das Passwort meiner mit vagrant erstellten Maschinen ist übrigens `vagrant`. 
```
ansible-playbook -i inventory playbook-helloworld.yml -u vagrant
```
### Zugriff auf Systeminformationen
```
ansible all -i inventory -m setup -u vagrant
```
IPV4 Adressen auslesen
```
ansible all -i inventory -m setup -a "filter=*ipv4*" -u vagrant
```
```
nano playbook-ipv4.yml
```
Einfügen:
```
- hosts: all
  tasks:
    - name: print facts
      debug:
        msg: "IPv4 address: {{ ansible_all_ipv4_addresses }}"
```
Playbook ausführen:
```
ansible-playbook -i inventory playbook-ipv4.yml -u vagrant
```
### Playbook update & upgrade
`nano playbook-update.yml`
```
- hosts: all
  become: yes
  tasks:
    - name: Update apt repo and cache on all Debian/Ubuntu boxes
      apt: update_cache=yes force_apt_get=yes cache_valid_time=3600

    - name: Upgrade all packages on servers
      apt: upgrade=dist force_apt_get=yes
```
`ansible-playbook -i inventory playbook-update.yml -u vagrant -K`
```
vagrant@ansiblehost:~/ansible-playbooks$ ansible-playbook -i inventory playbook-update.yml -u vagrant -K
BECOME password:

PLAY [all] ****************************************************
TASK [Gathering Facts] ****************************************
ok: [machine2]
ok: [machine1]
ok: [machine3]

TASK [Update apt repo and cache on all Debian/Ubuntu boxes] ***
changed: [machine2]
changed: [machine1]
changed: [machine3]

TASK [Upgrade all packages on servers] ************************
changed: [machine3]
changed: [machine1]
changed: [machine2]

PLAY RECAP ****************************************************
machine1                   : ok=3    changed=2 
machine2                   : ok=3    changed=2 
machine3                   : ok=3    changed=2 
```
### Playbook install programs
`nano playbook-install_programs.yml`
```
- hosts: all
  become: yes
  tasks:
    - name: Update apt cache and make sure curl, nano, nginx and php-fpm are installed
      apt:
        name: "{{ item }}"
        update_cache: yes
      loop:
        - curl
        - nano
        - nginx
        - php-fpm
```
```
vagrant@ansiblehost:~/ansible-playbooks$ ansible-playbook -i inventory playbook-install_programs.yml -u vagrant -K
BECOME password:

PLAY [all] ***************************

TASK [Gathering Facts] ***************
ok: [machine2]
ok: [machine3]
ok: [machine1]

TASK [Update apt cache and make sure curl, nano, nginx and php-fpm are installed] *****
ok: [machine1] => (item=curl)
ok: [machine2] => (item=curl)
ok: [machine3] => (item=curl)
ok: [machine1] => (item=nano)
ok: [machine3] => (item=nano)
ok: [machine2] => (item=nano)
changed: [machine1] => (item=nginx)
changed: [machine3] => (item=nginx)
changed: [machine2] => (item=nginx)
changed: [machine1] => (item=php-fpm)
changed: [machine2] => (item=php-fpm)
changed: [machine3] => (item=php-fpm)

PLAY RECAP ****************************
machine1                   : ok=2    changed=1
machine2                   : ok=2    changed=1
machine3                   : ok=2    changed=1
```
### Playbook Install-Webserver
```
- hosts: all
  become: yes
  tasks:
    - name: Update apt cache and make sure curl, nano, nginx and php-fpm are installed
      apt:
        name: "{{ item }}"
        update_cache: yes
      loop:
        - curl
        - nano
        - nginx
        - libnginx-mod-rtmp
        - php-fpm

    - name: delete default nginx.conf
      file:
        path: /etc/nginx/nginx.conf
        state: absent
      notify: restart nginx

    - name: copy new /etc/nginx/nginx.conf
      copy:
        src: ~/ansible-playbooks/webserver/nginx.conf
        dest: /etc/nginx/
      notify: restart nginx

    - name: copying new default file from local to remote
      copy:
        src: ~/ansible-playbooks/webserver/default
        dest: /etc/nginx/sites-available/
      notify:
        - restart nginx

    - name: copying new rtmp.conf file from local to remote
      copy:
        src: ~/ansible-playbooks/webserver/rtmp.conf
        dest: /etc/nginx/sites-available/
      notify:
        - restart nginx

    - name: Creates directory
      file:
        path: /var/www/html/stream
        state: directory

    - name: create symbolic link
      file:
        src: /etc/nginx/sites-available/rtmp.conf
        dest: /etc/nginx/sites-enabled/rtmp.conf
        state: link
      notify:
        - restart nginx

    - name: copying info.php
      copy:
        src: ~/ansible-playbooks/webserver/info.php
        dest: /var/www/html
        owner: www-data
        group: www-data
        mode: 0775

  handlers:
    - name: restart nginx
      service: 
        name: nginx
        state: restarted

```
Das obige Beispiel funktioniert so nur, wenn folgende Dateien im lokalen Order ~/ansible-playbooks/webserver fertig bereit liegen:  
default
info.php
inventory
nginx.conf
rtmp.conf
