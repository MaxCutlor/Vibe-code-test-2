# Guide : quitter Obsidian Sync avec un serveur maison

Configuration retenue : **PC Windows + iPhone**, plus peut-être un Mac plus tard.
Solution : **Self-hosted LiveSync** (plugin Obsidian) + **CouchDB** sur un **Raspberry Pi ou un NAS**, accessible partout via **Tailscale**.

> Un futur Mac ne change rien : il suffira d'y installer Obsidian et le plugin. Pas besoin de basculer sur iCloud.
>
> **Garde Obsidian Sync actif jusqu'à l'étape 6.** Rien n'est coupé avant que la nouvelle synchro soit en place.

---

## 0. Le matériel

| Option | Ce qu'il faut | Remarque |
|---|---|---|
| **Raspberry Pi 4 ou 5** | 2 Go de RAM minimum (4 Go conseillés), alimentation officielle, **SSD en USB** (environ 30 €) | Évite la carte SD seule : CouchDB écrit beaucoup et use les cartes SD. |
| **NAS Synology / QNAP** | Un modèle qui gère Docker (« Container Manager » chez Synology, souvent les modèles « + ») | Encore plus simple si le NAS a déjà des disques et des sauvegardes. |
| Vieux PC ou portable | Linux (Debian/Ubuntu) ou Windows avec Docker | Consomme plus d'électricité. |

Consommation d'un Pi : environ 3 à 5 W, soit quelques euros d'électricité par an.

---

## 1. Préparer le serveur (exemple : Raspberry Pi)

1. Avec **Raspberry Pi Imager** sur ton PC, installe **Raspberry Pi OS Lite (64-bit)**. Dans les réglages de l'imager, active SSH et définis un utilisateur et un mot de passe.
2. Connecte-toi depuis le PC : dans PowerShell, `ssh tonuser@raspberrypi.local`.
3. Installe Docker :
   ```bash
   curl -fsSL https://get.docker.com | sh
   sudo usermod -aG docker $USER   # puis déconnecte-toi et reconnecte-toi
   ```

Sur un **NAS Synology**, ignore cette étape : installe « Container Manager » depuis le Centre de paquets, puis crée un « Projet » à l'étape 2 en y collant le `docker-compose.yml`.

---

## 2. Installer CouchDB

Crée un dossier `~/couchdb` qui contient les deux fichiers suivants.

**`docker-compose.yml`**
```yaml
services:
  couchdb:
    image: couchdb:3.4
    container_name: couchdb-obsidian
    restart: unless-stopped
    environment:
      - COUCHDB_USER=obsidian
      - COUCHDB_PASSWORD=CHANGE-MOI-mot-de-passe-long
    volumes:
      - ./data:/opt/couchdb/data
      - ./local.ini:/opt/couchdb/etc/local.d/local.ini
    ports:
      - "127.0.0.1:5984:5984"   # accessible seulement depuis le Pi : Tailscale s'occupe de l'extérieur
```

**`local.ini`** (réglages demandés par LiveSync)
```ini
[couchdb]
single_node = true
max_document_size = 50000000

[chttpd]
require_valid_user = true
max_http_request_size = 4294967296
enable_cors = true

[chttpd_auth]
require_valid_user = true
authentication_redirect = /_utils/session.html

[httpd]
WWW-Authenticate = Basic realm="couchdb"
bind_address = 0.0.0.0

[cors]
origins = app://obsidian.md, capacitor://localhost, http://localhost
credentials = true
headers = accept, authorization, content-type, origin, referer
methods = GET, PUT, POST, HEAD, DELETE
max_age = 3600
```

Ensuite, lance CouchDB et vérifie qu'il répond :
```bash
cd ~/couchdb && docker compose up -d
curl -u obsidian:CHANGE-MOI-mot-de-passe-long http://127.0.0.1:5984/
# doit répondre {"couchdb":"Welcome",...}
```

---

## 3. Accès depuis partout avec Tailscale (HTTPS inclus)

L'app Obsidian sur iPhone exige une adresse **HTTPS**. Tailscale la fournit gratuitement, sans ouvrir de port sur ta box.

1. Crée un compte gratuit sur tailscale.com.
2. Sur le Pi :
   ```bash
   curl -fsSL https://tailscale.com/install.sh | sh
   sudo tailscale up
   ```
3. Dans la console d'administration Tailscale (onglet **DNS**), active **MagicDNS** et **HTTPS Certificates**.
4. Sur le Pi, publie CouchDB en HTTPS sur ton réseau Tailscale :
   ```bash
   sudo tailscale serve --bg 5984
   ```
   Tu obtiens une adresse du type `https://raspberrypi.ton-reseau.ts.net`.
5. Installe **Tailscale sur le PC et sur l'iPhone**, connectés au même compte. Sur l'iPhone, active « VPN on Demand » pour que la connexion reste toujours active.

Test : ouvre `https://raspberrypi.ton-reseau.ts.net/_utils` dans le navigateur du PC. La page de CouchDB doit apparaître.

---

## 4. Configurer le PC (source de référence)

1. **Sauvegarde** : zippe le dossier du vault.
2. Dans Obsidian : Réglages → Plugins communautaires → installe **Self-hosted LiveSync** et active-le.
3. Lance l'assistant de configuration du plugin :
   - **URI** : `https://raspberrypi.ton-reseau.ts.net`
   - **Utilisateur** et **mot de passe** : ceux du `docker-compose.yml`
   - **Base de données** : `obsidian` (elle est créée automatiquement)
   - Active le **chiffrement de bout en bout** et choisis une phrase secrète. Note-la dans ton gestionnaire de mots de passe : sans elle, les données sont irrécupérables.
   - Mode de synchro : **LiveSync** (temps réel).
4. Dès que LiveSync fonctionne, **désactive Obsidian Sync sur le PC** : Réglages → Plugins principaux → Synchronisation, en position désactivée. Ne laisse jamais deux synchros actives sur le même vault.
5. Lance l'envoi initial du vault vers le serveur (bouton proposé à la fin de l'assistant).
6. Dans le plugin, **copie l'« URI de configuration »** (Setup URI), protégée par un mot de passe. Elle sert à configurer l'iPhone en un clic.

> Les libellés exacts de l'assistant évoluent selon les versions. En cas de doute, suis la doc officielle : https://github.com/vrtmrz/obsidian-livesync

---

## 5. Configurer l'iPhone

1. Désactive Obsidian Sync sur l'iPhone.
2. Dans Obsidian, **crée un vault neuf et vide**, stocké sur l'iPhone et **pas dans iCloud**.
3. Installe Self-hosted LiveSync, puis choisis « utiliser une URI de configuration » et colle l'URI copiée à l'étape 4.6.
4. Laisse le vault se télécharger, puis vérifie quelques notes.
5. Supprime l'ancien vault de l'iPhone une fois que tout est correct.

Test croisé : modifie une note sur le PC, elle doit apparaître sur l'iPhone en quelques secondes, et inversement.

---

## 6. Sauvegardes (pour remplacer l'historique d'Obsidian Sync)

- **Sur le PC** : installe le plugin **Obsidian Git**, avec un commit automatique toutes les 30 minutes vers un **dépôt GitHub privé**. Tu retrouves ainsi l'historique de chaque note.
- **Sur le serveur** : sauvegarde de temps en temps le dossier `~/couchdb/data`, sur le NAS, un disque externe ou ailleurs.
- Garde le plugin principal **« Récupération de fichiers »** activé.

Après **une à deux semaines sans problème**, tu peux **résilier Obsidian Sync**.

---

## 7. Plus tard

- **Un Mac** : installe Obsidian, Tailscale et le plugin, puis colle l'URI de configuration. C'est tout.
- **L'app de tâches** : ce même serveur pourra héberger le service de rappels (option D de l'étude). `livesync-bridge` y recopie le vault en fichiers lisibles, et les rappels arrivent sur l'iPhone via l'app **ntfy**.

## En cas de souci

| Symptôme | Piste |
|---|---|
| L'iPhone ne synchronise pas | Tailscale est-il connecté sur l'iPhone ? |
| Erreur CORS ou 401 | Vérifie `local.ini` et le mot de passe, puis `docker compose restart` |
| « Document too large » | Pièce jointe très lourde : augmente `max_document_size` ou exclus le dossier dans le plugin |
| Doublons ou conflits | Le plugin propose une résolution des conflits ; vérifie qu'Obsidian Sync est bien coupé partout |
