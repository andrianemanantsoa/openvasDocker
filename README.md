# OpenVAS avec Docker Compose

Déploiement simple d'OpenVAS (Greenbone Vulnerability Scanner) via Docker Compose, avec persistance des données et accès web.

## 🧰 Prérequis
- Docker et Docker Compose installés
- Ports libres sur l’hôte : 443 (HTTPS UI), 9390 (Manager)
- Droits suffisants pour exécuter des conteneurs

## 🚀 Démarrage rapide
1. Cloner ce dépôt
2. Lancer les services
   ```bash
   docker compose up -d
   ```
3. Accéder à l’interface Web GSA
   - URL: https://localhost/
   - Identifiant: admin
   - Mot de passe: admin (modifiable via variable d’env.)

> Remarque: Le premier démarrage peut être long (téléchargement/MAJ des NVTs).

## ⚙️ Configuration
Le fichier `docker-compose.yml` lance un seul service `openvas` basé sur l’image `mikesplain/openvas:latest`.

- Variables d’environnement:
  - `OV_PASSWORD`: mot de passe de l’utilisateur admin (par défaut: `admin`)
  - `PUBLIC_HOSTNAME`: nom d’hôte utilisé pour les certificats et l’accès (ex: `openvas.local`)

- Ports exposés:
  - `443:443` → Interface Web (GSA)
  - `9390:9390` → OpenVAS Manager (si vous avez des intégrations externes)

- Volumes:
  - `./openvas/data:/var/lib/openvas/mgr/` → persistance des données (DB, configs)

## 🔐 Mots de passe et sécurité
- Changez `OV_PASSWORD` avant le premier démarrage.
- Évitez d’exposer 443/9390 publiquement. Utilisez un reverse proxy et des ACL.
- Pour un hostname public différent, mettez à jour `PUBLIC_HOSTNAME`.

## 📦 Commandes utiles
- Démarrer: `docker compose up -d`
- Logs: `docker compose logs -f openvas`
- Arrêter: `docker compose down`
- Redémarrer: `docker compose restart openvas`
- Mettre à jour l’image:
  ```bash
  docker compose pull
  docker compose up -d --force-recreate
  ```

## 💾 Sauvegarde et restauration
Les données persistantes sont stockées dans `openvas/data`.
- Sauvegarde: archivez ce dossier (hors conteneur éteint de préférence)
- Restauration: replacez le dossier puis relancez `docker compose up -d`

## 🧪 Tests basiques
Après connexion à GSA:
- Créez un « Target » (ex: 127.0.0.1)
- Lancez un « Task » de scan
- Vérifiez les rapports générés

## 🛠️ Dépannage
- L’UI ne répond pas:
  - Vérifiez que le conteneur est "healthy" et écoute sur 443
  - Ports déjà utilisés ? `ss -tulpn | grep :443`
- Mises à jour NVT longues: attendez la fin au premier boot (consultez les logs)
- Erreurs de DB: supprimez prudemment `openvas/data/*` pour repartir (vous perdrez l’historique)

## ❓ FAQ
- Peut-on changer le port HTTPS ? Oui, modifiez le mapping `443:443` (ex: `8443:443`) dans `docker-compose.yml`.
- Où sont les rapports ? Dans l’UI GSA et en base dans le volume persistant.

## 📁 Arborescence
```
.
├── docker-compose.yml
└── openvas/
    └── data/
        ├── tasks.db
        ├── tasks.db-shm
        └── tasks.db-wal
```

## 📜 Licence
Ce projet fournit un fichier de composition Docker. Vérifiez la licence de l’image `mikesplain/openvas` avant usage en production.
