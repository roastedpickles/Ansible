# Exercice

Placez-vous dans le répertoire du dixième atelier pratique :

```
$ cd ~/formation-ansible/atelier-12
```
Voici les quatre machines virtuelles de cet atelier :

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
$ vagrant ssh ansible
```
L’environnement de cet atelier est préconfiguré et prêt à l’emploi :

* Ansible est installé sur le Control Host.
* Le fichier /etc/hosts du Control Host est correctement renseigné.
* L’authentification par clé SSH est établie sur les trois Target Hosts.
* Le répertoire du projet existe et contient une configuration de base et un inventaire.
* Direnv est installé et activé pour le projet.
* Le validateur de syntaxe yamllint est également disponible.

Rendez-vous dans le répertoire du projet :
```
$ cd ansible/projets/ema/
direnv: loading ~/ansible/projets/ema/.envrc
direnv: export +ANSIBLE_CONFIG
$ ls -l
total 8
-rw-r--r--. 1 vagrant vagrant  65 Sep 19 14:26 ansible.cfg
-rw-r--r--. 1 vagrant vagrant 128 Sep 19 14:26 inventory
drwxr-xr-x. 2 vagrant vagrant   6 Sep 19 14:26 playbooks
```

L’inventaire des machines définit un groupe [redhat] avec les trois Target Hosts :
```
$ cat inventory 
[redhat]
target01
target02
target03

[redhat:vars]
ansible_python_interpreter=/usr/bin/python3
ansible_user=vagrant
ansible_become=yes
```
Écrivez un playbook chrony.yml qui assure la synchronisation NTP de tous vos Target Hosts :

* Installez le paquet chrony.
```
--- #  chrony.yml

- hosts: redhat

  tasks:

    - name: Install Chrony
      dnf:
        name: chrony
...
```
* Activez et démarrez le service chronyd correspondant.
```
--- #  chrony.yml

- hosts: redhat

  tasks:

    - name: Install Chrony
      dnf:
        name: chrony

    - name: Start & enable Chrony
      service:
        name: chronyd
        state: started
        enabled: true

...
```
* Jetez un œil sur le fichier de configuration /etc/chrony.conf fourni par défaut.
```
ansible all -a "cat /etc/chrony.conf"
```
* Installez une configuration personnalisée (cf. ci-dessous).
```
---  # chrony.yml

- hosts: redhat

  tasks:

    - name: Install Chrony
      dnf:
        name: chrony

    - name: Start & enable Chrony
      service:
        name: chronyd
        state: started
        enabled: true

    - name: Configuration du fichier chrony.conf
      copy:
        dest: /etc/chrony.conf
        content: |
          # /etc/chrony.conf
          server 0.fr.pool.ntp.org iburst
          server 1.fr.pool.ntp.org iburst
          server 2.fr.pool.ntp.org iburst
          server 3.fr.pool.ntp.org iburst
          driftfile /var/lib/chrony/drift
          makestep 1.0 3
          rtcsync
          logdir /var/log/chrony
      notify: Restart Chrony

  handlers:

    - name: Restart Chrony
      service:
        name: chronyd
        state: restarted

...
```
* Vérifiez la syntaxe correcte de votre playbook chrony.yml.
```
$ yamllint chrony.yml
```
* Prenez en compte cette nouvelle configuration.
```
$ ansible-playbook chrony.yml 

PLAY [redhat] ******************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************
ok: [target02]
ok: [target01]
ok: [target03]

TASK [Install Chrony] **********************************************************************************************
ok: [target01]
ok: [target03]
ok: [target02]

TASK [Start & enable Chrony] ***************************************************************************************
ok: [target03]
ok: [target02]
ok: [target01]

TASK [Configuration du fichier chrony.conf] ************************************************************************
changed: [target02]
changed: [target01]
changed: [target03]

PLAY RECAP *********************************************************************************************************
target01                   : ok=4    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target02                   : ok=4    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target03                   : ok=4    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

$ ansible all -a "cat /etc/chrony.conf"
target02 | CHANGED | rc=0 >>
# /etc/chrony.conf
server 0.fr.pool.ntp.org iburst
server 1.fr.pool.ntp.org iburst
server 2.fr.pool.ntp.org iburst
server 3.fr.pool.ntp.org iburst
driftfile /var/lib/chrony/drift
makestep 1.0 3
rtcsync
logdir /var/log/chrony
target01 | CHANGED | rc=0 >>
# /etc/chrony.conf
server 0.fr.pool.ntp.org iburst
server 1.fr.pool.ntp.org iburst
server 2.fr.pool.ntp.org iburst
server 3.fr.pool.ntp.org iburst
driftfile /var/lib/chrony/drift
makestep 1.0 3
rtcsync
logdir /var/log/chrony
target03 | CHANGED | rc=0 >>
# /etc/chrony.conf
server 0.fr.pool.ntp.org iburst
server 1.fr.pool.ntp.org iburst
server 2.fr.pool.ntp.org iburst
server 3.fr.pool.ntp.org iburst
driftfile /var/lib/chrony/drift
makestep 1.0 3
rtcsync
logdir /var/log/chrony
```

* Vérifiez l’idempotence de votre playbook.
```
$ ansible-playbook chrony.yml 

PLAY [redhat] ******************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************
ok: [target03]
ok: [target01]
ok: [target02]

TASK [Install Chrony] **********************************************************************************************
ok: [target03]
ok: [target02]
ok: [target01]

TASK [Start & enable Chrony] ***************************************************************************************
ok: [target02]
ok: [target03]
ok: [target01]

TASK [Configuration du fichier chrony.conf] ************************************************************************
ok: [target01]
ok: [target02]
ok: [target03]

PLAY RECAP *********************************************************************************************************
target01                   : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target02                   : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target03                   : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```
Il y a idempotence : le contenu copié dans le fichier ne change pas donc il est considéré comme "unchanged". Il doit donc y avoir un checksum pour savoir si le fichier a été changé ou non.
