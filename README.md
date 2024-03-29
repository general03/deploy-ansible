# Schema de base

![schema](https://user-images.githubusercontent.com/2324694/151509329-36123ca0-9054-4416-9ef8-39d1eabc4133.png)

1. On initie une connexion au serveur déclaré dans l'inventory (dans .gitlab-ci.yml)
2. On génère une paire de clé SSH à la volé pour générer le deploy key (voir étape 3)
3. On crée une deploy key à partir de la cléé ssh de l'étape 2 afin de faire notre checkout
4. On récupère le code du repo sur le serveur Ansible

# Comment configurer Ansible avec les Actions Github

Il faut commencer par générer une paire de clé SSH. Garder le clé privée et pour la clé public suivre le procédure de Scaleway ci dessous

## Generate SSH key Scaleway

```
# To add a new key, you can:
#   -- Add keys on your Scaleway account https://cloud.scaleway.com/#/credentials
#   -- Add keys using server tags - https://cloud.scaleway.com/#/servers/5d05550d-0b16-4a51-9414-379679c56614
#        - i.e: "AUTHORIZED_KEY=ssh-rsa_XXXXXXXXXXX AUTHORIZED_KEY=ssh-rsa_YYYYYYYYYYYYYYY"
#        - Be sure to replace all spaces with underscores
#        - $> sed 's/ /_/g' ~/.ssh/id_rsa.pub
#   -- Add the keys to '/root/.ssh/instance_keys' which will be imported
#
# And recreate your 'authorized_keys' file with the new keys:
#   -- Run 'scw-fetch-ssh-keys --upgrade'
```

## Générer les clés SSH

Il faut lancer la commande `ssh-keygen -t rsa -b 4096` (dans la console scaleway)

/!\ Ne pas mettre de passphrase

Il gènère une clé privée (sans .pub) et une clé public

La clé privée doit OBLIGATOIREMENT avoir des droits `600`

Il faut bien vérifier que la clé public est dans le fichier  ~/.ssh/authorized_keys du serveur :
- fait par la commande scaleway
- ou par la commande `ssh-copy-id -i ~/.ssh/key.pub <USER SERVER>@<IP SERVER>`

## Inventory Ansible

Il est nécessaire de configurer l'inventory avec votre clé SSH créée juste avant
```
[server]
51.159.186.197 ansible_ssh_user=root ansible_ssh_private_key_file=id_rsa
```

## Variables 

### env

Il est nécessaire de configurer les variables d'env dans la CD de Github/Gitlab

- TOKEN_GITHUB => Personnal Access Token de Github ou Personal Access Token de Gitlab (dans les settings utilisateur) avec comme scope api et read_repository
Ce token va nous permettre de générer un "deploy key" à partir d'une clé SSH public (https://gitlab.itarverne.com/help/user/project/deploy_keys/index), et donc utiliser la clé privée pour faire notre checkout 
- PRIVATE_KEY_ANSIBLE => Clé privée pour se connecter au serveur (généré à la 1er étape)

### local

Il faut éventuellement modifier les variables du playbook

Les étapes d'ajout du token et du clone doivent être adaptées pour pointer sur le bon repository

## Github Action

Il faut aller dans l'onglet Action de Github et choisir un workflow custom. Faites le commit.

A chaque push sur la branche (précisée dans le yml) un workflow sera lancé avec le playbook Ansible.

Il faut ajouter le fichier ansible.cfg avec   
`interpreter_python = auto`

Cela évite les erreurs 
`Failed to import the required Python library (python-gitlab) on bloog's Python /usr/bin/python3. Please read the module documentation and install it in the appropriate location. If the required library is installed, but Ansible is using the wrong Python interpreter, please consult the documentation on ansible_python_interpreter`


# Création custom rôle

Pourquoi faire une rôle => https://docs.ansible.com/ansible/latest/user_guide/index.html#writing-tasks-plays-and-playbooks

- Il faut installer le module `pipenv install ansible`
- Créer le scrafold `ansible-galaxy init pipenv`
- Mettre le contenu du role dans le dossier `tasks`
- Ajouter dans le playbook 
```
  roles:
    - role: pipenv
```
ou alors créer une tâche
```
- name: Get pipenv
  import_role: 
    name: pipenv
```

# Errors

## `Failed to connect to the host via ssh: Host key verification failed` lors de la connexion SSH

- Si vous avez cette erreur, ajoutez dans `ansible.cfg`
```
[defaults]
host_key_checking = False
```
- OU encore mieux il faut ajouter le fingerprints depuis le runner gitlab (dans le fichier `.gitlab.yml`)
```
ssh-keyscan -H <IP SERVER> >> ~/.ssh/known_hosts
```