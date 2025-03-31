# Exercice

Placez-vous dans le répertoire du dixième atelier pratique :

```
$ cd ~/formation-ansible/atelier-15
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

* Écrivez un playbook kernel.yml qui affiche les infos détaillées du noyau sur tous vos Target Hosts. Utilisez la commande uname -a et le module debug avec le paramètre msg.
```
---  # kernel.yml

- hosts: all
  gather_facts: false

  tasks:

    - name: Report kernel information
      command: uname -a
      changed_when: false
      register: uname_cmd

    - debug:
        msg: "{{uname_cmd.stdout_lines}}"
...



$ ansible-playbook kernel.yml 

PLAY [all] *********************************************************************************************************

TASK [Report kernel information] ***********************************************************************************
ok: [debian]
ok: [suse]
ok: [rocky]

TASK [debug] *******************************************************************************************************
ok: [rocky] => {
    "msg": [
        "Linux rocky 5.14.0-362.13.1.el9_3.x86_64 #1 SMP PREEMPT_DYNAMIC Wed Dec 13 14:07:45 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux"
    ]
}
ok: [debian] => {
    "msg": [
        "Linux debian 6.1.0-17-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.69-1 (2023-12-30) x86_64 GNU/Linux"
    ]
}
ok: [suse] => {
    "msg": [
        "Linux suse 5.14.21-150500.55.39-default #1 SMP PREEMPT_DYNAMIC Tue Dec 5 10:06:35 UTC 2023 (2e4092e) x86_64 x86_64 x86_64 GNU/Linux"
    ]
}

PLAY RECAP *********************************************************************************************************
debian                     : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
rocky                      : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
suse                       : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```
* Essayez d’obtenir le même résultat en utilisant le paramètre var du module debug.
```
---  # kernel.yml

- hosts: all
  gather_facts: false

  tasks:

    - name: Report kernel information
      command: uname -a
      changed_when: false
      register: uname_cmd

    - debug:
        var: uname_cmd.stdout_lines

...




$ ansible-playbook kernel.yml 

PLAY [all] *********************************************************************************************************

TASK [Report kernel information] ***********************************************************************************
ok: [debian]
ok: [rocky]
ok: [suse]

TASK [debug] *******************************************************************************************************
ok: [rocky] => {
    "uname_cmd.stdout_lines": [
        "Linux rocky 5.14.0-362.13.1.el9_3.x86_64 #1 SMP PREEMPT_DYNAMIC Wed Dec 13 14:07:45 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux"
    ]
}
ok: [debian] => {
    "uname_cmd.stdout_lines": [
        "Linux debian 6.1.0-17-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.69-1 (2023-12-30) x86_64 GNU/Linux"
    ]
}
ok: [suse] => {
    "uname_cmd.stdout_lines": [
        "Linux suse 5.14.21-150500.55.39-default #1 SMP PREEMPT_DYNAMIC Tue Dec 5 10:06:35 UTC 2023 (2e4092e) x86_64 x86_64 x86_64 GNU/Linux"
    ]
}

PLAY RECAP *********************************************************************************************************
debian                     : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
rocky                      : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
suse                       : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   


```
* Écrivez un playbook packages.yml qui affiche le nombre total de paquets RPM installés sur les hôtes rocky et suse (rpm -qa | wc -l).
  * Modifiez d'abord le fichier inventory pour créer un groupe d'hôtes :
```
[testing]
rocky
debian
suse

[rpm_hosts]
rocky
suse

[testing:vars]
ansible_python_interpreter=/usr/bin/python3
ansible_user=vagrant
ansible_become=yes
```
  * puis créez le script (utilisez le module "shell" plutot que "command" car ce dernier ne prend pas en compte les "|") :
```
---  # packages.yml

- hosts: rpm_hosts
  gather_facts: false

  tasks:

    - name: Report the total number of packages on the rocky and suse vm
      shell: rpm -qa | wc -l
      changed_when: false
      register: nb_packages

    - debug:
        var: nb_packages.stdout_lines

...




$ ansible-playbook packages.yml 

PLAY [rpm_hosts] ***************************************************************************************************

TASK [Report the total number of packages on the rocky and suse vm] ************************************************
ok: [rocky]
ok: [suse]

TASK [debug] *******************************************************************************************************
ok: [rocky] => {
    "nb_packages.stdout_lines": [
        "671"
    ]
}
ok: [suse] => {
    "nb_packages.stdout_lines": [
        "917"
    ]
}

PLAY RECAP *********************************************************************************************************
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
