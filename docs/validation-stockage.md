# Validation du stockage de sauvegarde

Date : 18 octobre 2026

## Configuration
- Support : clé USB SanDisk Ultra, capacité annoncée de 32 Go.
- Partition unique formatée en ext4.
- Point de montage : /mnt/rebuild-backup.
- Montage configuré par UUID avec Ansible.
- Support distinct de la microSD système, mais pas hors site.

## Résultats observés
- Simulation : ok=4, changed=3, unreachable=0, failed=0.
- Exécution réelle sur configuration en place :
  ok=4, changed=0, unreachable=0, failed=0.
- Montage actif vérifié avec findmnt.
- Environ 27 Go disponibles selon df -h.
- Entrée de montage présente dans /etc/fstab.

## Limites
- Récapitulatif de la première application avec changements non conservé.
- Persistance après redémarrage non testée.
- Santé et endurance de la clé non établies.
- Aucune sauvegarde ni restauration encore réalisée.
- Refus de sauvegarder lorsque la clé est absente non encore implémenté.
