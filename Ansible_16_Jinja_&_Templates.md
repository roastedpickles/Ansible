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
```

```
* Dans d’autres cas ce sera simplement /etc/chrony.conf.
```

```

À vous de trouver une solution en utilisant les éléments que nous avons abordés dans cet article.
