# Ansible par la pratique
##Exercice

Placez-vous dans le répertoire du troisième atelier pratique :
```
$ cd ~/formation-ansible/atelier-03
```
Voici les quatre machines virtuelles Ubuntu 22.04 de cet atelier :
|Machine virtuelle |	Adresse IP|
| ------------- |:-------------:|
|control |	192.168.56.10|
|target01 |	192.168.56.20|
|target02 |	192.168.56.30|
|target03 |	192.168.56.40|

Démarrez les VM :
```
$ vagrant up
```
Connectez-vous au Control Host :
```
$ vagrant ssh control
```
Ansible est déjà installé sur cette machine :
```
$ type ansible
ansible is /usr/bin/ansible
```

Faites le nécessaire pour réussir un ping Ansible comme ceci :
```
$ ansible all -i target01,target02,target03 -m ping
target03 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
target02 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
target01 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

* Ajoutez dans le fichier /etc/hosts à l'aide de nano ou vim les hosts suivants afin d'avoir une résolution de nom :
```
# /etc/hosts
127.0.0.1 localhost
127.0.1.1 vagrant

192.168.56.10 control
192.168.56.20 target01
192.168.56.30 target02
192.168.56.40 target03
```
* Faites un test de connectivité avec :
```
$ for HOST in target01 target02 target03; do ping -c 1 -q $HOST; done

PING target01 (192.168.56.20) 56(84) bytes of data.
--- target01 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms

PING target02 (192.168.56.30) 56(84) bytes of data.
--- target02 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms

PING target03 (192.168.56.40) 56(84) bytes of data.
--- target03 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms


```
* Puis recupérez les clés SSH des hotes :
```
$ ssh-keyscan -t rsa target01 target02 target03 >> .ssh/known_hosts
# target03:22 SSH-2.0-OpenSSH_8.9p1 Ubuntu-3ubuntu0.11
# target01:22 SSH-2.0-OpenSSH_8.9p1 Ubuntu-3ubuntu0.11
# target02:22 SSH-2.0-OpenSSH_8.9p1 Ubuntu-3ubuntu0.11
```
* Vous pouvez vérifier la connexion SSH avec :
```
$ ssh target0X      #remplacez X par 1, 2 ou 3 et le mot de passe est vagrant

```
* Générez la clé SSH :
```
$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/vagrant/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/vagrant/.ssh/id_rsa
Your public key has been saved in /home/vagrant/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:b/nr6c58Hr5xsmU97wxPhob0mJxlkPB5HtWvKdVizFY vagrant@control
The key's randomart image is:
+---[RSA 3072]----+
|          .     o|
|           o o .E|
|            =ooo.|
|             +B.o|
|        S   .+++ |
|         . +.Oo..|
|          + *.O.B|
|         . + +.#o|
|           +X+=o=|
+----[SHA256]-----+

```
* Distribuez la clé publique sur les Target Hosts avec le mot de passe _vagrant_ :
```
$ ssh-copy-id vagrant@target01
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/vagrant/.ssh/id_rsa.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
vagrant@target01's password: 
Number of key(s) added: 1
Now try logging into the machine, with:   "ssh 'vagrant@target01'"
and check to make sure that only the key(s) you wanted were added.

$ ssh-copy-id vagrant@target02
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/vagrant/.ssh/id_rsa.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
vagrant@target02's password: 
Number of key(s) added: 1
Now try logging into the machine, with:   "ssh 'vagrant@target02'"
and check to make sure that only the key(s) you wanted were added.

$ ssh-copy-id vagrant@target03
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/vagrant/.ssh/id_rsa.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
vagrant@target03's password: 
Number of key(s) added: 1
Now try logging into the machine, with:   "ssh 'vagrant@target03'"
and check to make sure that only the key(s) you wanted were added.

```
* Finalement vous pouvez lancer le ping :
```

$ ansible all -i target01,target02,target03 -m ping
target01 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
target02 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
target03 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}

```
