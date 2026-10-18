# Validation du service web

Date : 18 octobre 2026

## Configuration
- Service Nginx déployé avec Docker Compose et Ansible.
- Image déclarée : nginx:stable-alpine.
- Version Docker Engine observée : 29.9.1.
- Données persistantes : /srv/rebuild-lab/data.
- Données montées en lecture seule dans le conteneur.
- Port publié : 127.0.0.1:8080 vers le port 80 du conteneur.
- Accès depuis le Mac par tunnel SSH sur le port local 18080.

## Résultats observés
- Premier déploiement : ok=14, changed=7, unreachable=0, failed=0.
- Deuxième exécution : ok=14, changed=0, unreachable=0, failed=0.
- Conteneur en cours d'exécution.
- Accès par tunnel : HTTP 200 OK.
- Identifiant fictif affiché : rebuild-demo-001.
- Accès direct au port 8080 par l'adresse Ethernet :
  connexion refusée, code curl 7.

## Limites
- Accès par l'adresse Wi-Fi et IPv6 non testés.
- Redémarrage du Pi non testé.
- Image non figée par digest.
- Compatibilité testée uniquement sur le Pi ARM64.
- Aucune sauvegarde ni restauration du service encore réalisée.
