# Version ansible de la VM Linux cible  

Ce document vous donne la version automatisée, avec Ansible, de créer la VM Linux cible.  

## Pré-requis  

Vous devez avoir une VM d'installée avec les spécifications suivantes:  

- installation minimale d'une distribution Linux (il est possible d'utiliser une version serveur);  
- mémoire : 4 Go minimum;  
- disque : 25 Go minimum;  
- serveur ssh;  
- utilisateur jim dans les groupes wheel, sudo, adm.  

**Note :** Pour créer ce document, j'ai utilisé un XUbuntu 25.10 avec une installation minimale. Si, vous utilisez une autre distribution ou version de Linux, il se peut que vous deviez faire des ajustements aux fichiers.