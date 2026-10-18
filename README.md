# REBUILD LAB

Projet personnel d’infrastructure : déployer un petit service web, automatiser sa configuration, sauvegarder ses données et prouver sa reconstruction après une panne simulée.

Projet en cours : le service et le stockage de sauvegarde sont déployés. La sauvegarde, la restauration et la reconstruction ne sont pas encore validées.

## Objectifs

- Administrer un serveur Linux.
- Déployer un service avec Ansible et Docker Compose.
- Limiter et tester ses accès réseau.
- Sauvegarder les données avec restic sur un support distinct.
- Tester une restauration puis une reconstruction et mesurer les résultats.

## Architecture actuelle

- Raspberry Pi 4 B avec 4 Go de RAM.
- Système Debian 13 Trixie ARM64 sans bureau graphique, sur microSD de 32 Go.
- Poste de contrôle macOS avec Ansible Core 2.21.5 exécuté sous Python 3.13.
- Service Nginx avec Docker Compose ; Docker Engine observé : 29.9.1.
- Image déclarée : nginx:stable-alpine, non figée par digest à ce stade.
- Données persistantes : /srv/rebuild-lab/data, montées en lecture seule dans le conteneur.
- Clé USB SanDisk Ultra de 32 Go, formatée en ext4, montée sur /mnt/rebuild-backup.
- Administration interactive avec Raspberry Pi Connect et SSH ; automatisation par SSH depuis le Mac.

```text
Mac : Ansible ── SSH ──> Raspberry Pi 4
Navigateur Mac ── tunnel SSH ──> 127.0.0.1:8080 ──> Nginx
                                                    │
                                      /srv/rebuild-lab/data

Clé USB distincte : /mnt/rebuild-backup
Destination prévue pour restic ; sauvegarde non encore réalisée.
```

## Résultats observés le 18 octobre 2026

| Vérification | Résultat |
|---|---|
| Accès SSH et module Ansible ping | Réussis |
| Montage de la clé USB | Actif, configuré par UUID dans fstab |
| Réexécution du playbook stockage | ok=4, changed=0, unreachable=0, failed=0 |
| Premier déploiement web | ok=14, changed=7, unreachable=0, failed=0 |
| Deuxième exécution du playbook web | ok=14, changed=0, unreachable=0, failed=0 |
| Publication du port web | 127.0.0.1:8080 vers le port 80 du conteneur |
| Accès HTTP depuis le Mac par tunnel SSH | HTTP 200 OK, contenu attendu |
| Accès direct par l’adresse Ethernet du Pi au port 8080 | Connexion refusée, code curl 7 |

L’absence de changement à la réexécution est observée dans l’état testé ; elle ne garantit pas tous les scénarios futurs.

## Organisation du dépôt

```text
ansible/
  requirements.yml
  inventory/example.yml
  playbooks/storage.yml
  playbooks/web.yml
compose/docker-compose.yml
docs/validation-stockage.md
docs/validation-web.md
```

## Utilisation des playbooks

Ces instructions correspondent au Pi Debian 13 ARM64 testé, pas à un déploiement universel. Le playbook web installe Docker et modifie le système cible.

Étapes initiales manuelles : installation du système, configuration des accès, vérification de l’identité SSH, partitionnement et formatage du support de sauvegarde. Ne jamais formater un disque sans identifier la cible et confirmer la perte de son contenu.

Depuis le poste de contrôle, à la racine du dépôt :

```bash
ansible-galaxy collection install -r ansible/requirements.yml
cp ansible/inventory/example.yml ansible/inventory/local.yml
```

Créer local.yml seulement s’il n’existe pas déjà. Renseigner l’adresse du Pi, l’utilisateur SSH et l’UUID de la partition de sauvegarde. Aucun mot de passe ne doit figurer dans l’inventaire.

Vérifier d’abord la syntaxe :

```bash
ansible-playbook -i ansible/inventory/local.yml ansible/playbooks/storage.yml --syntax-check
ansible-playbook -i ansible/inventory/local.yml ansible/playbooks/web.yml --syntax-check
```

Après vérification du matériel, du support monté et de l’absence de conflits Docker :

```bash
ansible-playbook -i ansible/inventory/local.yml ansible/playbooks/storage.yml --ask-pass --ask-become-pass
ansible-playbook -i ansible/inventory/local.yml ansible/playbooks/web.yml --ask-pass --ask-become-pass
```

Le playbook stockage monte une partition existante ; il ne la formate pas. Le playbook web conserve la page initiale si elle existe déjà.

## Accès au service

Le service écoute sur la boucle locale du Pi. Depuis le Mac, remplacer UTILISATEUR et ADRESSE_DU_PI :

```bash
ssh -N -o ExitOnForwardFailure=yes -L 127.0.0.1:18080:127.0.0.1:8080 UTILISATEUR@ADRESSE_DU_PI
```

Ouvrir ensuite http://127.0.0.1:18080 dans le navigateur du Mac. Fermer le tunnel avec Ctrl+C. Aucune redirection de port sur le routeur n’a été nécessaire pour ce test.

## Sauvegarde et reconstruction : travail restant

- Installer et configurer restic.
- Conserver le secret de restauration hors du dépôt et hors de la seule microSD.
- Vérifier le montage de la clé avant toute sauvegarde et refuser l’écriture si elle est absente.
- Créer une sauvegarde, contrôler son intégrité et restaurer dans un dossier temporaire.
- Comparer les fichiers récupérés aux fichiers d’origine.
- Préparer un scénario non destructif de panne selon le matériel disponible.
- Mesurer le temps de reconstruction, de restauration, l’âge de la sauvegarde et les interventions manuelles.

## Limites actuelles

- Aucune sauvegarde, restauration ou reconstruction complète encore démontrée.
- Persistance après redémarrage du Pi non testée.
- Tests réseau limités au tunnel SSH et à l’accès direct Ethernet depuis le Mac ; Wi-Fi et IPv6 non testés.
- Santé et endurance de la clé USB non établies.
- Support de sauvegarde local distinct, pas une sauvegarde hors site.
- Image Docker non figée par digest ; compatibilité testée uniquement sur le Pi ARM64.
- Aucune haute disponibilité ni basculement automatique.
- Extension cloud facultative, non engagée.

## Confidentialité

L’inventaire réel ansible/inventory/local.yml est exclu de Git. Ne publier aucun secret ni donnée personnelle dans les fichiers, les captures, les journaux ou les métadonnées de commits. Utiliser uniquement des données de démonstration fictives.
