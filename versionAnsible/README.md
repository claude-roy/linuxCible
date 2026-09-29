# Version ansible de la VM Linux cible  

Ce document vous donne la version automatisée, avec Ansible, de créer la VM Linux cible.  

## Prérequis  

### Linux cible  

Vous devez avoir une VM d'installée avec les spécifications suivantes:  

- installation minimale d'une distribution Linux (il est possible d'utiliser une version serveur);  
- mémoire : 4 Go minimum;  
- disque : 25 Go minimum;  
- serveur ssh;  
- un utilisateur avec le nom "jim", l'utilisateur doit être dans les groupes d'administrations de votre distribution : par exemple les groupes wheel, sudo, adm.  

**Note 1 :** Pour créer ce document, j'ai utilisé un XUbuntu 24.04 avec une installation minimale. Si, vous utilisez une autre distribution ou version de Linux, il se peut que vous deviez faire des ajustements aux fichiers.  

**Attention :** il semble avoir un problème avec Ubuntu 25.10 (Timeout for privilege escalation).  

Pour l'installation du serveur SSH :  

```bash
sudo apt update && sudo apt install ssh -y
sudo systemctl enable --now ssh
sudo systemctl status ssh
```  

### Poste de contrôle ou installation local

Vous pouvez utiliser un poste de contrôle avec Ansible d'installé. Le poste de contrôle doit être un système Linux.  

Vous pouvez également faire l'installation localement.  

## Installation d'Ansible  

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
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/home/jim/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /home/jim/.local/share/pipx/venvs/ansible/lib/python3.12/site-packages/ansible
  ansible collection location = /home/jim/.ansible/collections:/usr/share/ansible/collections
  executable location = /home/jim/.local/bin/ansible
  python version = 3.12.7 (main, Nov  8 2024, 17:55:36) [GCC 14.2.0] (/home/prof/.local/share/pipx/venvs/ansible/bin/python)
  jinja version = 3.1.4
  libyaml = True
```  

Si vous utilisez un hôte de contrôle, vous devez copier la clé publique SSH de l'utilisateur du poste de contrôle dans l'utilisateur *jim* de la VM cible.

Voici un exemple avec une clé spécifique :  

```bash  
ssh-copy-id -i ~/.ssh/linuxcible jim@adresse_ip
```  

Si vous travaillez localement, je vous recommande d'installer git et de cloner le dépôt linuxCible.  

```bash
sudo apt update && sudo apt install git -y
git clone https://github.com/claude-roy/linuxCible.git
```  


## Fichiers d'automatisation Ansible  
### Ficher `ansible.conf`  

Le fichier `ansible.conf` est le fichier de configuration d'Ansible.  

### Fichier `hosts.yaml`  

Le fichier `hosts.yaml` est le fichier des appareils à utiliser.  

### Utilisation d'un poste de contrôle   

Vous devez ajuster l'entrée *ansible_host* à l'adresse IP de votre VM.  
Vous devez ajuster l'entrée *ansible_ssh_private_key_file* à votre clé SSH. Si vous n'utilisez pas une clé spécifique, vous pouvez commenter cette ligne.  

### Utilisation local  

Pour un déploiement local, vous devez changer la variable ```hosts``` du fichier `deploy.yaml` pour ```control```.  

### Variable `ansible_sudo_pass`    

Le fichier `deploy.yaml` contient la variable `ansible_sudo_pass` que vous devez changer pour le mot de passe de l'utilisateur de la cible Linux.  

### Fichier `deploy.yaml`  

Avant de faire un déploiement, il est recommandé de vérifier la fonctionnalité d'Ansible et du fait même la connectivité.

```bash
ansible -m ping all  
# Pour une installation local
ansible -m ping control

```  

Le déploiement est regroupé par étape en utilisant les `tags`. Le déploiement avec l'utilisation des tags se fait de la manière suivant :  

```bash
ansible-playbook deploy.yaml --tags docker # vous remplacer le tag docker par celui de l'étape.
```  

Les étapes et les `tags` sont les suivants :  

1. Installation des applications : tag apps.  
2. Installation de Docker : tag docker.  
3. Ajout des utilisateurs : tag add_users
4. Création des répertoires : tag reps.  
5. Clone du dépôt Mutillidae : tag clone_git.  
6. Copie des fichiers Docker Compose, script et service : tag copy_files.  
7. Le lancement des conteneurs : tag compose_up.  
8. L'arrêt des conteneurs : tag compose_stop.  
9. L'arrêt et le retrait des conteneurs : tag compose_down.
10. Pour installer les applications comme un service : tag set_as_service.  

## Configuration des applications après l'installation  

Référez-vous à la page [README.md](https://github.com/claude-roy/linuxCible/blob/main/README.md#configuration-des-applications) de la configuration d'un Linux cible pour la configuration des applications.

## Références  
[https://docs.ansible.com/]  
[https://docs.ansible.com/projects/ansible/latest/installation_guide/installation_distros.html#installing-distros]  
