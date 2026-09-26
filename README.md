# qBittorrent WebUI Toolbox

Userscript (Violentmonkey / Tampermonkey) qui ajoute au WebUI de qBittorrent :

- **Créateur de torrent amélioré** — remplace la fenêtre native : taille de pièce, format v1/v2/hybride, privé par défaut, trackers enregistrés (nom, URL, champ *source*), navigateur de fichiers avec autocomplétion, préremplissage depuis le torrent sélectionné, téléchargement du `.torrent`.
- **Éditeur de trackers en masse** — remplace une URL d'announce sur tous les torrents concernés (sélection, filtre tracker de la barre latérale, ou toute la bibliothèque).
- **MediaInfo** *(optionnel)* — affiche la sortie `mediainfo` d'un fichier d'un torrent, copie ou export `.nfo`. Nécessite le petit conteneur `mediainfo-api`.

## Prérequis

- qBittorrent **5.0+** (API `torrentcreator`). **5.1+** recommandé : l'éditeur de trackers récupère tout en une seule requête (sinon, repli sur un scan torrent par torrent, plus lent).
- Violentmonkey (recommandé) ou Tampermonkey.
- Aucun identifiant à fournir : le script tourne dans la page du WebUI et réutilise ta session.

## Installation

1. Clique sur **[qbt-webui-toolbox.user.js](https://raw.githubusercontent.com/Gusdezup/qbt-webui-toolbox/main/qbt-webui-toolbox.user.js)** : l'extension propose l'installation.
2. Déclare l'adresse de **ton** WebUI :
   - **Violentmonkey** : tableau de bord → *Modifier* le script → onglet *Paramètres* → *Règles @match* → ajoute par ex. `http://192.168.1.10:8080/*` (garde « Garder l'original » coché) → Enregistrer.
   - **Tampermonkey** : *Paramètres* du script → *Correspondances utilisateur* → même valeur.

   Le `/*` final est obligatoire. Ce réglage survit aux mises à jour.
3. Recharge le WebUI.

Les mises à jour sont automatiques (via `@updateURL`).

## Réglages

Menu de l'extension → **Réglages…**, ou bouton **⚙ Réglages** dans la fenêtre du créateur :

| Réglage | Défaut |
|---|---|
| URL du sidecar MediaInfo (vide = bouton masqué) | vide |
| Taille des pièces | 4 MiB |
| Format | v1 |
| Torrent privé / démarrer le seed | oui / oui |
| Masquer les flèches de file d'attente | non |

## MediaInfo (optionnel)

Le navigateur ne peut pas lire les fichiers du serveur : un conteneur `mediainfo-api` exécute `mediainfo` à sa place.

Ajoute ce service au `docker-compose.yml` qui contient qBittorrent (voir [`docker-compose.example.yml`](docker-compose.example.yml)) :

```yaml
  mediainfo-api:
    image: ghcr.io/Gusdezup/qbt-mediainfo-api:latest
    container_name: mediainfo-api
    user: "1000:1000"            # mêmes PUID/PGID que qBittorrent
    environment:
      - ALLOWED_ROOT=/media
    volumes:
      - /chemin/vers/tes/medias:/media:ro
    ports:
      - "8765:8765"
    restart: unless-stopped
```

Puis `docker compose up -d mediainfo-api`, et dans **Réglages…** du script : `http://IP-DU-SERVEUR:8765` → **Tester** → **Enregistrer**.

**Point essentiel :** le volume doit être monté **au même chemin dans le conteneur** que dans qBittorrent. Si qBittorrent voit tes fichiers sous `/downloads`, monte-les aussi sous `/downloads` et mets `ALLOWED_ROOT=/downloads`.

Variables : `ALLOWED_ROOT` (défaut `/media`), `PORT` (8765), `MEDIAINFO_TIMEOUT` en secondes (60).

### Sécurité

- Le sidecar n'a **pas d'authentification** : ne l'expose **qu'en réseau local**, jamais sur Internet.
- Il refuse tout chemin hors de `ALLOWED_ROOT` (liens symboliques et `..` résolus) et ne lit que des fichiers. Monte les médias en lecture seule (`:ro`).

## Dépannage

- **Aucun bouton n'apparaît** : l'adresse du WebUI n'est pas dans les règles `@match`.
- **MediaInfo : « Injoignable »** : vérifie IP/port et que le conteneur tourne (`curl http://IP:8765/health` → `ok`).
- **MediaInfo : « Fichier introuvable »** : chemins de montage différents entre qBittorrent et le sidecar.
- **Éditeur de trackers lent** : qBittorrent < 5.1.

## Licence

MIT
