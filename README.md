# REBUILD LAB

Projet personnel de déploiement reproductible et de reconstruction
d'un service après une panne simulée.

## Architecture prévue
- Raspberry Pi 4 B avec 4 Go de RAM
- Carte microSD de 32 Go
- Système Linux ARM64 sans bureau graphique
- Poste de contrôle sous macOS
- Configuration avec Ansible
- Service avec Docker Compose
- Sauvegarde et restauration avec restic
- Support de sauvegarde distinct : à confirmer

## Validé à ce stade
- Démarrage du Pi et diagnostic initial
- Accès SSH depuis le Mac
- Exécution distante du module Ansible ping

## Non encore validé
- Déploiement du service
- Règles et tests réseau
- Sauvegarde et restauration
- Reconstruction après panne simulée

## Inventaire
Copier ansible/inventory/example.yml vers
ansible/inventory/local.yml et adapter les paramètres.

Le fichier local.yml ne doit pas être versionné.
Aucun secret ni aucune donnée personnelle ne doit être publié.
