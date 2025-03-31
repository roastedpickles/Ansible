# Exercice

Placez-vous dans le répertoire du dixième atelier pratique :

```
$ cd ~/formation-ansible/atelier-18
```
Voici les quatre machines virtuelles de cet atelier :

|Machine virtuelle |	Adresse IP|
| ------------- |:-------------:|
|ansible | 192.168.56.10|
|rocky | 192.168.56.20|
|debian | 192.168.56.30|
|suse |	192.168.56.40|
|ubuntu |	192.168.56.50|

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

Dans notre précédent exercice, nous avons mis en place la synchronisation NTP avec Chrony sur nos Target Hosts, en installant le fichier de configuration ci-dessous sur chacune des quatre cibles :
```
# chrony.conf
server 0.fr.pool.ntp.org iburst
server 1.fr.pool.ntp.org iburst
server 2.fr.pool.ntp.org iburst
server 3.fr.pool.ntp.org iburst
driftfile /var/lib/chrony/drift
makestep 1.0 3
rtcsync
logdir /var/log/chrony
```

Écrivez un playbook chrony.yml qui installe un fichier de configuration personnalisé sur vos cibles. La première ligne de commentaire devra indiquer le chemin complet vers le fichier :

* Dans certains cas ce sera /etc/chrony/chrony.conf.
* Dans d’autres cas ce sera simplement /etc/chrony.conf.
```
---  # chrony.yml

- hosts: all

  tasks:

    - name: Load distribution-specific parameters
      include_vars: >
        chrony01_{{ansible_distribution|lower|replace(" ", "-") }}.yml

    - name: Update package information on Debian/Ubuntu
      apt:
        update_cache: true
        cache_valid_time: 3600
      when: ansible_os_family == "Debian"

    - name: Install Chrony
      package:
        name: "{{chrony_package_name}}"

    - name: Start chrony & enable it on boot
      service:
        name: "{{chrony_service_name}}"
        state: started
        enabled: true

    - name: Configure custom chrony.conf file
      template:
        dest: "{{chrony_config_directory}}/chrony.conf"
        src: chrony-1.conf.j2
      notify: Reload Chrony


  handlers:

    - name: Reload Chrony
      service:
        name: "{{chrony_service_name}}"
        state: restarted
...



```




Et les différents fichiers de configuration :
```
---  # vars/chrony01_debian.yml

chrony_package_name: chrony
chrony_service_name: chronyd
chrony_config_directory: /etc/chrony

...
```
```
---  # vars/chrony01_ubuntu.yml

chrony_package_name: chrony
chrony_service_name: chrony
chrony_config_directory: /etc/chrony

...

```
```
---  # vars/chrony01_opensuse-leap.yml

chrony_package_name: chrony
chrony_service_name: chronyd
chrony_config_directory: /etc

...
```
```
---  # vars/chrony01_rocky.yml

chrony_package_name: chrony
chrony_service_name: chronyd
chrony_config_directory: /etc

...

```
```
{# templates/chrony-1.conf.j2 #}
server 0.fr.pool.ntp.org iburst
server 1.fr.pool.ntp.org iburst
server 2.fr.pool.ntp.org iburst
server 3.fr.pool.ntp.org iburst
driftfile /var/lib/chrony/drift
makestep 1.0 3
rtcsync
logdir /var/log/chrony

```

* résultats de l'execution :

```
$ ansible-playbook chrony.yml 

PLAY [all] *********************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************
ok: [debian]
ok: [ubuntu]
ok: [rocky]
ok: [suse]

TASK [Load distribution-specific parameters] ***********************************************************************
ok: [rocky]
ok: [debian]
ok: [suse]
ok: [ubuntu]

TASK [Update package information on Debian/Ubuntu] *****************************************************************
skipping: [rocky]
skipping: [suse]
ok: [ubuntu]
changed: [debian]

TASK [Install Chrony] **********************************************************************************************
ok: [debian]
ok: [suse]
ok: [ubuntu]
ok: [rocky]

TASK [Start chrony & enable it on boot] ****************************************************************************
ok: [ubuntu]
ok: [debian]
ok: [rocky]
ok: [suse]

TASK [Configure custom chrony.conf file] ***************************************************************************
ok: [ubuntu]
ok: [debian]
ok: [suse]
ok: [rocky]

PLAY RECAP *********************************************************************************************************
debian                     : ok=6    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
rocky                      : ok=5    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
suse                       : ok=5    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
ubuntu                     : ok=6    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   



$ ansible rocky,suse -a "cat /etc/chrony.conf"
suse | CHANGED | rc=0 >>
# chrony.conf
server 0.fr.pool.ntp.org iburst
server 1.fr.pool.ntp.org iburst
server 2.fr.pool.ntp.org iburst
server 3.fr.pool.ntp.org iburst
driftfile /var/lib/chrony/drift
makestep 1.0 3
rtcsync
logdir /var/log/chrony
rocky | CHANGED | rc=0 >>
# chrony.conf
server 0.fr.pool.ntp.org iburst
server 1.fr.pool.ntp.org iburst
server 2.fr.pool.ntp.org iburst
server 3.fr.pool.ntp.org iburst
driftfile /var/lib/chrony/drift
makestep 1.0 3
rtcsync
logdir /var/log/chrony


$ ansible debian,ubuntu -a "cat /etc/chrony/chrony.conf"
ubuntu | CHANGED | rc=0 >>
# chrony.conf
server 0.fr.pool.ntp.org iburst
server 1.fr.pool.ntp.org iburst
server 2.fr.pool.ntp.org iburst
server 3.fr.pool.ntp.org iburst
driftfile /var/lib/chrony/drift
makestep 1.0 3
rtcsync
logdir /var/log/chrony
debian | CHANGED | rc=0 >>
# chrony.conf
server 0.fr.pool.ntp.org iburst
server 1.fr.pool.ntp.org iburst
server 2.fr.pool.ntp.org iburst
server 3.fr.pool.ntp.org iburst
driftfile /var/lib/chrony/drift
makestep 1.0 3
rtcsync
logdir /var/log/chrony

```

Quittez le Control Host :
```
$ exit
```
Supprimez toutes les VM :
```
$ vagrant destroy -f
```
