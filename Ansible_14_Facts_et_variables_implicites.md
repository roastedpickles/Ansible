# Exercice

Placez-vous dans le répertoire du dixième atelier pratique :

```
$ cd ~/formation-ansible/atelier-16
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


Écrivez trois playbooks pour afficher des informations sur chacun des Target Hosts :

* pkg-info.yml pour afficher le gestionnaire de paquets utilisé
```
---  # pkg-info.yml

- hosts: all

  tasks:

    - name: Display packet manager
      debug:
        msg: "{{ansible_host}}'s packet manager: {{ansible_pkg_mgr}}"

...



$ ansible-playbook pkg-info.yml 

PLAY [all] *********************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************
ok: [debian]
ok: [suse]
ok: [rocky]

TASK [Display packet manager] **************************************************************************************
ok: [rocky] => {
    "msg": "rocky's packet manager: dnf"
}
ok: [debian] => {
    "msg": "debian's packet manager: apt"
}
ok: [suse] => {
    "msg": "suse's packet manager: zypper"
}

PLAY RECAP *********************************************************************************************************
debian                     : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
rocky                      : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
suse                       : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```
* python-info.yml pour afficher la version de Python installée
```
---  # python-info.yml
- hosts: all

  tasks:

    - name: Display Pyton version
      debug:
        msg: "{{ansible_host}}'s Python version: {{ansible_python.version_info}}"

...




$ ansible-playbook python-info.yml 

PLAY [all] *********************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************
ok: [debian]
ok: [rocky]
ok: [suse]

TASK [Display Pyton version] ***************************************************************************************
ok: [rocky] => {
    "msg": "rocky's Python version: [3, 9, 18, 'final', 0]"
}
ok: [debian] => {
    "msg": "debian's Python version: [3, 11, 2, 'final', 0]"
}
ok: [suse] => {
    "msg": "suse's Python version: [3, 6, 15, 'final', 0]"
}

PLAY RECAP *********************************************************************************************************
debian                     : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
rocky                      : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
suse                       : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```
* dns-info.yml pour afficher le(s) serveur(s) DNS utilisé(s)
```
---  # dns-info.yml
- hosts: all

  tasks:

    - name: Display DNS servers
      debug:
        msg: "{{ansible_host}}'s DNS server(s): {{ansible_dns.nameservers}}"

...




$ ansible-playbook dns-info.yml 

PLAY [all] *********************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************
ok: [debian]
ok: [rocky]
ok: [suse]

TASK [Display DNS servers] *****************************************************************************************
ok: [rocky] => {
    "msg": "rocky's DNS server(s): ['10.0.2.3']"
}
ok: [debian] => {
    "msg": "debian's DNS server(s): ['4.2.2.1', '4.2.2.2', '208.67.220.220']"
}
ok: [suse] => {
    "msg": "suse's DNS server(s): ['10.0.2.3']"
}

PLAY RECAP *********************************************************************************************************
debian                     : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
rocky                      : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
suse                       : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```




Quittez le Control Host :
```
$ exit
```
Supprimez toutes les VM :
```
$ vagrant destroy -f
```
