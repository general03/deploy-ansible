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

Il faut lancer la commande `ssh-keygen -t rsa -b 4096`

/!\ Ne pas mettre de passphrase

Il gènère une clé privée (sans .pub) et une clé public

La clé privée doit OBLIGATOIREMENT avoir des droits `600`

Il faut bien vérifier que la clé public est dans le fichier  ~/.ssh/authorized_keys du serveur (fait par la commande scaleway)

## Inventory Ansible

Il est nécessaire de configurer l'inventory avec votre clé SSH créée juste avant
```
[server]
51.159.186.197 ansible_ssh_user=root ansible_ssh_private_key_file=id_rsa
```

## Variables 

### env

Il est nécessaire de configurer les variables d'env dans la CD de Github/Gitlab

- TOKEN_GITHUB => Personnal Access Token de Github ou Personal Access Token de Gitlab (dans les settings utilisateur)
- PRIVATE_KEY_ANSIBLE => Clé privée pour se connecter au serveur (généré à la 1er étape)

### local

Il faut éventuellement modifier les variables du playbook

Les étapes d'ajout du token et du clone doivent être adaptées pour pointer sur le bon repository

## Github Action

Il faut aller dans l'onglet Action de Github et choisir un workflow custom. Faites le commit.

A chaque push sur la branche (précisée dans le yml) un workflow sera lancé avec le playbook Ansible.

## Gitlab Runner

Il faut ajouter le fichier ansible.cfg avec   
`interpreter_python = auto`

Cela évite les erreurs 
`Failed to import the required Python library (python-gitlab) on bloog's Python /usr/bin/python3. Please read the module documentation and install it in the appropriate location. If the required library is installed, but Ansible is using the wrong Python interpreter, please consult the documentation on ansible_python_interpreter`
