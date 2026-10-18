# REBUILD LAB

Projet personnel d’infrastructure autour d’un Raspberry Pi : déployer un service web avec Ansible et Docker Compose, sauvegarder ses données, puis démontrer sa reconstruction après une panne simulée.

Le service web est opérationnel. La sauvegarde avec restic, la restauration et la reconstruction constituent les prochaines étapes.

## Architecture

- Raspberry Pi 4 B avec 4 Go de RAM, sous Linux ARM64 sur microSD de 32 Go.
- Mac comme poste de contrôle, avec Ansible et SSH.
- Nginx déployé avec Docker Compose, à partir de l’image nginx:stable-alpine.
- Données du service dans /srv/rebuild-lab/data, montées en lecture seule dans le conteneur.
- Support USB distinct prévu pour les sauvegardes restic.

Le port web est publié uniquement sur 127.0.0.1:8080 côté Pi. L’accès depuis le Mac passe par un tunnel SSH.

```text
Mac ── Ansible / SSH ──> Raspberry Pi 4
                         └── Docker Compose
                              └── Nginx
                                   └── /srv/rebuild-lab/data

Mac ── tunnel SSH ──> 127.0.0.1:8080 ──> Nginx

Sauvegarde prévue : données du service ── restic ──> support USB
```

## Avancement

### Réalisé

- Accès SSH depuis le Mac et communication Ansible avec le Pi.
- Déploiement du service web avec Ansible et Docker Compose.
- Données persistantes séparées du conteneur.
- Deuxième exécution du playbook web sans changement : changed=0.
- Accès par tunnel SSH : HTTP 200 OK et contenu attendu.
- Accès direct au port 8080 par l’adresse Ethernet du Pi : connexion refusée.

### À réaliser

- Configurer restic et effectuer une première sauvegarde sur le support USB.
- Tester la restauration et comparer les fichiers récupérés aux originaux.
- Simuler une panne et reconstruire le service.
- Mesurer le temps de reconstruction et les interventions manuelles nécessaires.

## Périmètre des tests

Les tests web ont été réalisés le 18 octobre 2026 sur le Pi ARM64. Ils couvrent le déploiement, sa réexécution et l’accès réseau depuis le Mac.

Le redémarrage du Pi, les accès Wi-Fi et IPv6 ainsi que la restauration ne sont pas encore testés. L’image Docker n’est pas encore figée par digest.

## Code et documentation

- [Collections Ansible](ansible/requirements.yml)
- [Playbook web](ansible/playbooks/web.yml)
- [Configuration Docker Compose](compose/docker-compose.yml)
- [Compte rendu de validation web](docs/validation-web.md)
- [Exemple d’inventaire](ansible/inventory/example.yml)
