# vagrant
erste Schritte mit vagrant

Schnell mal paar virtuelle Maschinen in VirtualBox erstellen. (Siehe Vagrantfile)

>Vagrant ist eine freie Ruby-Anwendung zum Erstellen und Verwalten virtueller Maschinen. Vagrant ermöglicht einfache Softwareverteilung (englisch Deployment) insbesondere in der Software- und Webentwicklung und dient als Wrapper zwischen Virtualisierungssoftware wie VirtualBox, KVM/QEMU, VMware und Hyper-V und Software-Configuration-Management-Anwendungen beziehungsweise Systemkonfigurationswerkzeugen wie Chef, Saltstack und Puppet.

Nach der Erstellung der Maschinen will ich mit Ansible die Maschinen konfigurieren und verwalten.

getestet auf meinem PC mit Windows 11 (AMD Ryzen 7 2700 Eight-Core Processor 3.20 GHz / Windows 11 Home Version 22H2)

```
C:\Users\toral>git version
git version 2.37.3.windows.1

C:\Users\toral>git clone https://github.com/richtertoralf/vagrant.git

C:\Users\toral>vagrant -v
Vagrant 2.3.3

C:\Users\toral>cd vagrant

C:\Users\toral\vagrant>vagrant up
```
