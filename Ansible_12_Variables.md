# Exercice

Placez-vous dans le répertoire du dixième atelier pratique :

```
$ cd ~/formation-ansible/atelier-14
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

* Écrivez un playbook myvars1.yml qui affiche respectivement votre voiture et votre moto préférée en utilisant le module debug et deux variables mycar et mybike définies en tant que play vars.
```
---  # myvars1.yml

- hosts: localhost
  gather_facts: false


  vars:
    mycar: renault super 5
    mybike: honda XR125

  tasks:
    - debug:
        msg: "voiture: {{mycar}}, moto: {{mybike}}"

...


$ ansible-playbook myvars1.yml 

PLAY [localhost] ***************************************************************************************************

TASK [debug] *******************************************************************************************************
ok: [localhost] => {
    "msg": "voiture: renault super 5, moto: honda XR125"
}

PLAY RECAP *********************************************************************************************************
localhost                  : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```
* En utilisant les extra vars, remplacez successivement l’une et l’autre marque – puis les deux à la fois – avant d’exécuter le play.
```
$ ansible-playbook myvars1.yml -e mycar=kangoo

PLAY [localhost] ***************************************************************************************************

TASK [debug] *******************************************************************************************************
ok: [localhost] => {
    "msg": "voiture: kangoo, moto: honda XR125"
}

PLAY RECAP *********************************************************************************************************
localhost                  : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   




$ ansible-playbook myvars1.yml -e mybike=yamaha

PLAY [localhost] ***************************************************************************************************

TASK [debug] *******************************************************************************************************
ok: [localhost] => {
    "msg": "voiture: renault super 5, moto: yamaha"
}

PLAY RECAP *********************************************************************************************************
localhost                  : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   




$ ansible-playbook myvars1.yml -e mycar=kangoo -e mybike=yamaha

PLAY [localhost] ***************************************************************************************************

TASK [debug] *******************************************************************************************************
ok: [localhost] => {
    "msg": "voiture: kangoo, moto: yamaha"
}

PLAY RECAP *********************************************************************************************************
localhost                  : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```
* Écrivez un playbook myvars2.yml qui fait essentiellement la même chose que myvars1.yml, mais en utilisant une tâche avec set_fact pour définir les deux variables.
```
---  # myvars2.yml

- hosts: localhost
  gather_facts: false

  tasks:
    - name: Define variables
      set_fact:
        mycar: renault super 5
        mybike: honda XR125


    - debug:
        msg: "voiture: {{mycar}}, moto: {{mybike}}"

...


$ ansible-playbook myvars2.yml 

PLAY [localhost] ***************************************************************************************************

TASK [Define variables] ********************************************************************************************
ok: [localhost]

TASK [debug] *******************************************************************************************************
ok: [localhost] => {
    "msg": "voiture: renault super 5, moto: honda XR125"
}

PLAY RECAP *********************************************************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   



```
* Là aussi, essayez de remplacer les deux variables en utilisant des extra vars avant l’exécution du play.
```
$ ansible-playbook myvars2.yml -e mycar=kangoo -e mybike=yamaha

PLAY [localhost] ***************************************************************************************************

TASK [Define variables] ********************************************************************************************
ok: [localhost]

TASK [debug] *******************************************************************************************************
ok: [localhost] => {
    "msg": "voiture: kangoo, moto: yamaha"
}

PLAY RECAP *********************************************************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```
* Écrivez un playbook myvars3.yml qui affiche le contenu des deux variables mycar et mybike mais sans les définir. Avant d’exécuter le playbook, définissez VW et BMW comme valeurs par défaut pour mycar et mybike pour tous les hôtes, en utilisant l’endroit approprié.
```
---  # myvars3.yml

- hosts: all
  gather_facts: false

  tasks:
    - debug:
        msg: "voiture: {{mycar}}, moto: {{mybike}}"

...


$ mkdir -v ~/ansible/projets/ema/group_vars
mkdir: created directory '/home/vagrant/ansible/projets/ema/group_vars'


---  # group_vars/all.yml

mycar: VW                       
mybike: BMW

...

$ ansible-playbook myvars3.yml 

PLAY [all] *********************************************************************************************************

TASK [debug] *******************************************************************************************************
ok: [target02] => {
    "msg": "voiture: VW, moto: BMW"
}
ok: [target01] => {
    "msg": "voiture: VW, moto: BMW"
}
ok: [target03] => {
    "msg": "voiture: VW, moto: BMW"
}

PLAY RECAP *********************************************************************************************************
target01                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target02                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target03                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```
* Effectuez le nécessaire pour remplacer VW et BMW par Mercedes et Honda sur l’hôte target02.
```
$ mkdir -v ~/ansible/projets/ema/host_vars
mkdir: created directory '/home/vagrant/ansible/projets/ema/host_vars'


---  # host_vars/target02.yml

mycar: mercedes
mybike: honda

...




$ ansible-playbook myvars3.yml 

PLAY [all] *********************************************************************************************************

TASK [debug] *******************************************************************************************************
ok: [target01] => {
    "msg": "voiture: VW, moto: BMW"
}
ok: [target02] => {
    "msg": "voiture: mercedes, moto: honda"
}
ok: [target03] => {
    "msg": "voiture: VW, moto: BMW"
}

PLAY RECAP *********************************************************************************************************
target01                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target02                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target03                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```
* Écrivez un playbook display_user.yml qui affiche un utilisateur et son mot de passe correspondant à l’aide des variables user et password. Ces deux variables devront être saisies de manière interactive pendant l’exécution du playbook. Les valeurs par défaut seront microlinux pour user et yatahongaga pour password. Le mot de passe ne devra pas s’afficher pendant la saisie.
```
---  # display_user.yml    

- hosts: localhost
  gather_facts: false

  vars_prompt:

    - name: user
      prompt: Please enter your username
      default: microlinux
      private: false

    - name: password
      prompt: And now your password (secret)
      default: yatahongaga
      private: true

  tasks:
    - debug:
        msg: "user is {{user}}, password is {{password}}"

...



$ ansible-playbook display_user.yml 
Please enter your username [microlinux]: 
And now your password (secret) [yatahongaga]: 

PLAY [localhost] ***************************************************************************************************

TASK [debug] *******************************************************************************************************
ok: [localhost] => {
    "msg": "user is microlinux, password is yatahongaga"
}

PLAY RECAP *********************************************************************************************************
localhost                  : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

$ ansible-playbook display_user.yml 
Please enter your username [microlinux]: usr
And now your password (secret) [yatahongaga]: 

PLAY [localhost] ***************************************************************************************************

TASK [debug] *******************************************************************************************************
ok: [localhost] => {
    "msg": "user is usr, password is bonjour"
}

PLAY RECAP *********************************************************************************************************
localhost                  : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   


```


Quittez le Control Host :
```
$ exit
```
Supprimez toutes les VM :
```
$ vagrant destroy -f
```
