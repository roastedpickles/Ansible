# Exercice

Placez-vous dans le répertoire du dixième atelier pratique :

```
$ cd ~/formation-ansible/atelier-10
```
Voici les quatre machines virtuelles de cet atelier :

|Machine virtuelle |	Adresse IP|
| ------------- |:-------------:|
|ansible | 192.168.56.10|
|rocky | 192.168.56.20|
|debian | 192.168.56.30|
|suse |	192.168.56.40|

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

Écrivez trois playbooks :

* Un premier playbook apache-debian.yml qui installe Apache sur l’hôte debian avec une page personnalisée Apache web server running on Debian Linux. Nous le sauvegarderons dans playbooks/apache-debian.yml
```
---  # apache-debian.yml

- hosts: debian

  tasks:

    - name: Update package information
      apt:
        update_cache: true
        cache_valid_time: 3600

    - name: Install Apache
      apt:
        name: apache2

    - name: Start & enable Apache
      service:
        name: apache2
        state: started
        enabled: true

    - name: Install custom web page
      copy:
        dest: /var/www/html/index.html
        mode: 0644
        content: |
          <!doctype html>
          <html>
            <head>
              <meta charset="utf-8">
              <title>Test</title>
            </head>
            <body>
              <h1>Apache web server running on Debian Linux</h1>
            </body>
          </html>

...

```
* lancement et test du playbook :
```
$ yamllint playbooks/apache-debian.yml
$ ansible-playbook playbooks/apache-debian.yml 

PLAY [debian] ******************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************
ok: [debian]

TASK [Update package information] **********************************************************************************
changed: [debian]

TASK [Install Apache] **********************************************************************************************
changed: [debian]

TASK [Start & enable Apache] ***************************************************************************************
ok: [debian]

TASK [Install custom web page] *************************************************************************************
changed: [debian]

PLAY RECAP *********************************************************************************************************
debian                     : ok=5    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

$ curl debian
<!doctype html>
<html>
  <head>
    <meta charset="utf-8">
    <title>Test</title>
  </head>
  <body>
    <h1>Apache web server running on Debian Linux</h1>
  </body>
</html>

```
* Un deuxième playbook apache-rocky.yml qui installe Apache sur l’hôte rocky avec une page personnalisée Apache web server running on Rocky Linux. Nous le sauvegarderons dans playbooks/apache-rocky.yml
```

---  # apache-rocky.yml

- hosts: rocky

  tasks:

    - name: Install Apache
      dnf:
        name: httpd

    - name: Start & enable Apache
      service:
        name: httpd
        state: started
        enabled: true

    - name: Install custom web page
      copy:
        dest: /var/www/html/index.html
        mode: 0644
        content: |
          <!doctype html>
          <html>
            <head>
              <meta charset="utf-8">
              <title>Test</title>
            </head>
            <body>
              <h1>Apache web server running on Rocky Linux</h1>
            </body>
          </html>

...

```
* lancement et test du playbook :
```
$ yamllint playbooks/apache-rocky.yml 
$ ansible-playbook playbooks/apache-rocky.yml 

PLAY [rocky] *******************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************
ok: [rocky]

TASK [Install Apache] **********************************************************************************************
ok: [rocky]

TASK [Start & enable Apache] ***************************************************************************************
ok: [rocky]

TASK [Install custom web page] *************************************************************************************
changed: [rocky]

PLAY RECAP *********************************************************************************************************
rocky                      : ok=4    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

$ curl rocky
<!doctype html>
<html>
  <head>
    <meta charset="utf-8">
    <title>Test</title>
  </head>
  <body>
    <h1>Apache web server running on Rocky Linux</h1>
  </body>
</html>
```
* Un troisième playbook apache-suse.yml qui installe Apache sur l’hôte suse avec une page personnalisée Apache web server running on SUSE Linux. Nous le sauvegarderons dans playbooks/apache-suse.yml
```
---  # apache-suse.yml

- hosts: suse

  tasks:

    - name: Install Apache
      community.general.zypper:
        name: apache2

    - name: Start & enable Apache
      service:
        name: apache2
        state: started
        enabled: true

    - name: Install custom web page
      copy:
        dest: /srv/www/htdocs/index.html
        mode: 0644
        content: |
          <!doctype html>
          <html>
            <head>
              <meta charset="utf-8">
              <title>Test</title>
            </head>
            <body>
              <h1>Apache web server running on SUSE Linux</h1>
            </body>
          </html>

...

```
* lancement et test du playbook :
```
$ yamllint playbooks/apache-suse.yml
$ ansible-playbook playbooks/apache-suse.yml 

PLAY [suse] ********************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************
ok: [suse]

TASK [Install Apache] **********************************************************************************************
ok: [suse]

TASK [Start & enable Apache] ***************************************************************************************
ok: [suse]

TASK [Install custom web page] *************************************************************************************
changed: [suse]

PLAY RECAP *********************************************************************************************************
suse                       : ok=4    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

$ curl suse
<!doctype html>
<html>
  <head>
    <meta charset="utf-8">
    <title>Test</title>
  </head>
  <body>
    <h1>Apache web server running on SUSE Linux</h1>
  </body>
</html>

```
