# Exercice

Placez-vous dans le répertoire du dixième atelier pratique :

```
$ cd ~/formation-ansible/atelier-17
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
Écrivez successivement deux playbooks pour mettre en place la synchronisation NTP avec Chrony sur vos quatre Target Hosts sous Debian, Rocky Linux, SUSE Linux et Ubuntu. Il vous faudra identifier le nom du paquet, le service correspondant et le chemin vers le fichier de configuration par défaut pour chacune des distributions.

Voici la configuration qu’il faudra installer sur chacune des quatre cibles :

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

* Le premier playbook chrony-01.yml utilisera les modules de gestion de paquets natifs apt, dnf et zypper et s’inspirera de la méthode « gros sabots » utilisée plus haut dans cet article.
```
---  # chrony-01.yml

- hosts: all

  tasks:

    - name: Update package information on Debian/Ubuntu
      apt:
        update_cache: true
        cache_valid_time: 3600
      when: ansible_os_family == "Debian"

    - name: Install Chrony on Debian/Ubuntu
      apt:
        name: chrony
      when: ansible_os_family == "Debian"


    - name: Install Chrony on Rocky Linux
      dnf:
        name: chrony
      when: ansible_distribution == "Rocky"


    - name: Install Chrony on SUSE Linux
      zypper:
        name: chrony
      when: ansible_distribution == "openSUSE Leap"


    - name: Start & enable Chrony for debian family
      service:
        name: chrony
        state: started
        enabled: true
      when: ansible_distribution in ["Debian", "Ubuntu"]      

    - name: Configuration du fichier chrony.conf for debian
      copy:
        dest: /etc/chrony/chrony.conf
        content: |
          # /etc/chrony/chrony.conf
          server 0.fr.pool.ntp.org iburst
          server 1.fr.pool.ntp.org iburst
          server 2.fr.pool.ntp.org iburst
          server 3.fr.pool.ntp.org iburst
          driftfile /var/lib/chrony/drift
          makestep 1.0 3
          rtcsync
          logdir /var/log/chrony
      notify: Restart Chronydebian
      when: ansible_distribution in ["Debian", "Ubuntu"]

    - name: Start & enable Chrony
      service:
        name: chronyd
        state: started
        enabled: true
      when: ansible_distribution in ["Rocky", "openSUSE Leap"]

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
      notify: Restart Chrony!debian
      when: ansible_distribution in ["Rocky", "openSUSE Leap"]

  handlers:

    - name: Restart Chrony!debian
      service:
        name: chronyd
        state: restarted


    - name: Restart Chronydebian
      service:
        name: chrony
        state: restarted

...



```
* Le deuxième playbook chrony-02.yml définira trois variables chrony_package, chrony_service et chrony_confdir et utilisera le module de gestion de paquets générique package.
```
---  # chrony-02.yml

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

    - name: Configure file
      copy:
        dest: "{{chrony_config_directory}}/chrony.conf"
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

Quittez le Control Host :
```
$ exit
```
Supprimez toutes les VM :
```
$ vagrant destroy -f
```
