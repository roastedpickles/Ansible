# Ansible par la pratique

## Exercice 1


* Démarrez la VM ubuntu depuis le répertoire atelier-01.
```
$ cd ~/formation-ansible/atelier-01
$ vagrant up ubuntu
```
* Connectez-vous à cette VM.
```
$ vagrant ssh ubuntu
```
* Rafraîchissez les informations sur les paquets.
```
$ sudo apt update
```
* Recherchez le paquet ansible avec les options qui vont bien.
```
$ apt-cache search ansible
```
* Installez le paquet officiel fourni par la distribution.
```
$ sudo apt install -y ansible
```
* Vérifiez si l’installation s’est bien déroulée.
```
$ dpkg-query -l | grep "ansible"
$ type ansible
```
* Notez la version d’Ansible.
```
$ ansible --version | head -n 1
```
* Déconnectez-vous et supprimez la VM.
```
$ exit
$ vagrant destroy -f ubuntu
```

## Exercice 2
* Démarrez la VM ubuntu depuis le répertoire atelier-01.
```
$ cd ~/formation-ansible/atelier-01
$ vagrant up ubuntu
```
* Connectez-vous à cette VM.
```
$ vagrant ssh ubuntu
```
* Rafraîchissez les informations sur les paquets.
```
$ sudo apt update
```
* Recherchez le paquet ansible avec les options qui vont bien.
```
$ sudo apt-add-repository ppa:ansible/ansible
$ sudo apt update
```
* Installez le paquet officiel fourni par la distribution.
```
$ sudo apt install -y ansible
```
* Vérifiez si l’installation s’est bien déroulée.
```
$ dpkg-query -l | grep "ansible"
$ type ansible
```
* Notez la version d’Ansible.
```
$ ansible --version | head -n 1
```
* Déconnectez-vous et supprimez la VM.
```
$ exit
$ vagrant destroy -f ubuntu
```

## Exercice 3
* Lancez une VM Rocky Linux et installez Ansible en utilisant PIP et Virtualenv.
```
$ vagrant up rocky
$ vagrant ssh rocky
$ sudo dnf install -y epel-release
$ sudo crb enable
$ sudo dnf install -y python3-pip
$ python3 -m venv ~/.venv/ansible
$ source ~/.venv/ansible/bin/activate
(ansible) $ pip install --upgrade pip
(ansible) $ pip install ansible
(ansible) $ type ansible
(ansible) $ ansible --version
ansible [core 2.15.13]
  config file = None
  configured module search path = ['/home/vagrant/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /home/vagrant/.venv/ansible/lib64/python3.9/site-packages/ansible
  ansible collection location = /home/vagrant/.ansible/collections:/usr/share/ansible/collections
  executable location = /home/vagrant/.venv/ansible/bin/ansible
  python version = 3.9.18 (main, Sep  7 2023, 00:00:00) [GCC 11.4.1 20230605 (Red Hat 11.4.1-2)] (/home/vagrant/.venv/ansible/bin/python3)
  jinja version = 3.1.6
  libyaml = True

(ansible) $ deactivate
$ vagrant destroy -f rocky
```
