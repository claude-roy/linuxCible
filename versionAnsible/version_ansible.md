# Version ansible de la VM Linux cible  

Ce document vous donne la version automatisée, avec Ansible, de créer la VM Linux cible.  

## Pré-requis  

### Linux cible  

Vous devez avoir une VM d'installée avec les spécifications suivantes:  

- installation minimale d'une distribution Linux (il est possible d'utiliser une version serveur);  
- mémoire : 4 Go minimum;  
- disque : 25 Go minimum;  
- serveur ssh;  
- un utilisateur avec le nom "jim", l'utilisateur doit être dans les groupes d'administrations de votre distributions : par exemple les groupes wheel, sudo, adm.  

**Note 1 :** Pour créer ce document, j'ai utilisé un XUbuntu 25.10 avec une installation minimale. Si, vous utilisez une autre distribution ou version de Linux, il se peut que vous deviez faire des ajustements aux fichiers.  

**Attention :** il semble avoir un problème avec Ubuntu 25.10 (Timeout for privilege escalation).  

Pour l'installation du serveur SSH :  

```bash
sudo apt update && sudo apt install ssh -y
sudo systemctl status sshd
```  

### Poste de contrôle  

Vous devez également avoir un poste de contrôle avec Ansible d'installé. Le poste de contrôle doit être un système Linux.  

L’installation d’Ansible peut se faire de plusieurs manières;
 
- par l’intermédiaire des packages du système d'exploitation utilisé;
- à l’aide de l’outil pip ou pipx de Python (éventuellement combiné avec virtualenv);
- par l’utilisation des archives contenant le code source d’Ansible;
- ou enfin, en interprétant directement le code source en provenance de Git.

Nous allons opter pour les packages système.  

<details>
	<summary>Pour avoir la version plus récente d'Ansible.</summary/>  

Le dépôt de package de votre distribution ne contient pas toujours la dernière version d'Ansible. Consultez le lien de la documentation d'Ansible pour l'installation d'Ansible sous différentes distributions de Linux : [https://docs.ansible.com/projects/ansible/latest/installation_guide/installation_distros.html#installing-distros].  

De manière générale, depuis votre poste de contrôle, exécutez la commande suivante pour inclure le PPA (personal package archive) du projet officiel dans la liste des sources de votre système :  

```bash
sudo apt-add-repository ppa:ansible/ansible
```  

Vous pouvez vérifier la liste de vos sources logicielles et tapant la commande suivante :

```bash
ls -l /etc/apt/sources.list.d/
```  
</details>  
<br>  

Pour, l'installation :

```bash
sudo apt update && sudo apt install ansible -y
```

Vérification de l'installation d’Ansible

```bash
ansible --version
```

Ci-dessous un exemple de sortie de cette commande (ici avec la version 2.18.1) :

```bash
ansible [core 2.18.1]
  config file = None
  configured module search path = ['/home/prof/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /home/prof/.local/share/pipx/venvs/ansible/lib/python3.12/site-packages/ansible
  ansible collection location = /home/prof/.ansible/collections:/usr/share/ansible/collections
  executable location = /home/prof/.local/bin/ansible
  python version = 3.12.7 (main, Nov  8 2024, 17:55:36) [GCC 14.2.0] (/home/prof/.local/share/pipx/venvs/ansible/bin/python)
  jinja version = 3.1.4
  libyaml = True
```
Vous devez copier la clé publique SSH de l'utilisateur du poste de contrôle dans l'utilisateur *jim* de la VM cible.

Voici un exemple avec une clé spécifique :  

```bash  
ssh-copy-id -i ~/.ssh/linuxcible jim@adresse_ip
```  

## Fichiers d'automatisation Ansible  
### Ficher `ansible.conf`  

Le fichier `ansible.conf` est le fichier de configuration d'Ansible.  

### Fichier `hosts.yaml`  

Le fichier `hosts.yaml` est le fichier des appareils à utiliser.  

Vous devez ajuster l'entrée *ansible_host* à l'adresse IP de votre VM.  
Vous devez ajuster l'entrée *ansible_ssh_private_key_file* à votre clé SSH. Si vous n'utilisez pas une clé spécifique, vous pouvez commenter cette ligne.  

### Fichier `group_vars/Cible.yaml`    

Le fichier `Cible.yaml` contient la variable `ansible_sudo_pass` que vous devez changer pour le mot de passe de l'utilisateur de la cible Linux.  

### Fichier `deploy.yaml`  

Avant de faire un déploiement, il est recommandé de vérifier la fonctionnalité d'Ansible et du fait même la connectivité.

```bash
ansible -m ping all  
```  

Le déploiment est regroupé par étape en utilisant les `tags`. Le déploiement avec l'utilisation des tags se fait de la manière suivant :  

```bash
ansible-playbook deploy --tags docker # vous remplacer le tag docker par celui de l'étape.
```  

Les étapes et les `tags` sont les suivants :  

1. Installation de Docker : tag docker.  
2. Création des répertoires : tag reps.  
3. Clône du dépôt Mutillidae : tag clone_git.  
4. Copie des fichier Docker Compose : tag copy_files.  
5. Le lancement des conteneurs : tag compose_up.  
6. L'arrêt des conteneurs : tag compose_stop.  
7. L'arrêt et effassage des conteneurs : tag compose_down.  

## Références  
[https://docs.ansible.com/]
[https://docs.ansible.com/projects/ansible/latest/installation_guide/installation_distros.html#installing-distros]  
