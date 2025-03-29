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

```
* En utilisant les extra vars, remplacez successivement l’une et l’autre marque – puis les deux à la fois – avant d’exécuter le play.
```

```
* Écrivez un playbook myvars2.yml qui fait essentiellement la même chose que myvars1.yml, mais en utilisant une tâche avec set_fact pour définir les deux variables.
```

```
* Là aussi, essayez de remplacer les deux variables en utilisant des extra vars avant l’exécution du play.
```

```
* Écrivez un playbook myvars3.yml qui affiche le contenu des deux variables mycar et mybike mais sans les définir. Avant d’exécuter le playbook, définissez VW et BMW comme valeurs par défaut pour mycar et mybike pour tous les hôtes, en utilisant l’endroit approprié.
```

```
* Effectuez le nécessaire pour remplacer VW et BMW par Mercedes et Honda sur l’hôte target02.
```

```
* Écrivez un playbook display_user.yml qui affiche un utilisateur et son mot de passe correspondant à l’aide des variables user et password. Ces deux variables devront être saisies de manière interactive pendant l’exécution du playbook. Les valeurs par défaut seront microlinux pour user et yatahongaga pour password. Le mot de passe ne devra pas s’afficher pendant la saisie.
```

```


Quittez le Control Host :
```
$ exit
```
Supprimez toutes les VM :
```
$ vagrant destroy -f
```
