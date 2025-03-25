# Exercice

Pour vous familiariser avec la notion d’idempotence, exécutez une série de tâches administratives sur toutes les machines cibles. Pour ce faire, servez-vous des commandes ad hoc que nous avons vues dans le précédent article. Prenez soin d’exécuter chaque commande deux fois et observez ce qui se passe dans l’affichage du résultat.

* Installez successivement les paquets tree, git et nmap sur toutes les cibles.
```
$ ansible all -m package -a "name=tree state=present"
$ ansible all -m package -a "name=git state=present"
$ ansible all -m package -a "name=nmap state=present"
```
* Désinstallez successivement ces trois paquets en utilisant le paramètre supplémentaire state=absent.
```
$ ansible all -m package -a "name=tree state=absent"
$ ansible all -m package -a "name=git state=absent"
$ ansible all -m package -a "name=nmap state=absent"
```
* Copier le fichier /etc/fstab du Control Host vers tous les Target Hosts sous forme d’un fichier /tmp/test3.txt.
```
$ ansible all -m copy -a "src=/etc/fstab dest=/tmp/test3.txt"
```
* Supprimez le fichier /tmp/test3.txt sur les Target Hosts en utilisant le module file avec le paramètre state=absent.
```
$ ansible all -m file -a "dest=/tmp/test3.txt state=absent"
```
* Enfin, affichez l’espace utilisé par la partition principale sur tous les Target Hosts.
```
ansible all -m command -a "df -h /"
debian | CHANGED | rc=0 >>
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda3       124G  2.3G  115G   2% /
suse | CHANGED | rc=0 >>
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda3       124G  2.8G  118G   3% /
rocky | CHANGED | rc=0 >>
Filesystem                  Size  Used Avail Use% Mounted on
/dev/mapper/rl_rocky9-root   70G  2.4G   68G   4% /
```
* Que remarquez-vous ?

La commande n'affecte pas l'état de la VM mais affiche un état "CHANGED".
