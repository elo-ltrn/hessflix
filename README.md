# 📽️ HessFlix

## Préambule

**Podman** est un outil de gestion de conteneurs qui fonctionne sans nécessiter un démon central, contrairement à Docker. Il permet de créer, gérer et exécuter des conteneurs et des pods. Un **pod** est un groupe de conteneurs qui partagent le même réseau et peuvent communiquer entre eux de manière transparente. Chaque conteneur dans un pod peut avoir des volumes de stockage et des configurations propres, tout en étant lié aux autres conteneurs du pod.

Sur **macOS**, Podman peut être installé via **Homebrew** et utilise une machine virtuelle pour exécuter les conteneurs, car macOS ne supporte pas nativement les technologies de conteneurisation de type Linux. Cela permet de simuler un environnement Linux, dans lequel les conteneurs fonctionnent comme s'ils étaient exécutés sur une machine physique.

Les conteneurs que nous allons configurer sont utilisés pour des services spécifiques de gestion et de téléchargement de médias :
- **Jellyfin** : Un serveur multimédia open-source pour la gestion et la diffusion de films, séries, et musique.
- **Jellyseerr** : Une interface front-end pour gérer les téléchargements de séries TV via Radarr ou Sonarr.
- **qBittorrent** : Un client BitTorrent pour le téléchargement de fichiers.
- **Prowlarr** : Un indexeur pour gérer les moteurs de recherche de médias et s'intégrer avec Radarr, Sonarr, et Lidarr.
- **Radarr** : Un gestionnaire de films automatique, similaire à Sonarr, mais pour les films.
- **Sonarr** : Un gestionnaire de séries TV, similaire à Radarr.
## 🅰️ - **Méthode ligne de commande (Step-by-step)**

### 1. **Installation de Podman sur macOS**

#### a. **Installation de Podman via Homebrew**
Pour installer **Podman** sur votre Mac, vous devez d'abord installer **Homebrew**, un gestionnaire de paquets pour macOS. Si vous ne l'avez pas encore installé, exécutez cette commande dans votre terminal :

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Ensuite, installez **Podman** via **Homebrew** :

```bash
brew install podman
```

#### b. **Configurer la Machine Virtuelle pour Podman**
Podman utilise une machine virtuelle pour fonctionner sur macOS. Une fois Podman installé, vous devez initialiser cette machine virtuelle en utilisant la commande suivante :

```bash
podman machine init
podman machine start
```

Cela lancera la machine virtuelle où vos conteneurs Podman s'exécuteront. Vous pouvez vérifier que la machine fonctionne correctement avec :

```bash
podman machine list
```

### 2. **Création des Pods**

Pour chaque service, nous allons créer un **pod** et y exécuter les conteneurs correspondants.

#### a. **Pod Jellyfin**
```bash
podman pod create --name jellyfin-pod -p 8096:8096
podman run -dt --name jellyfin --pod jellyfin-pod \
    -v ~/Documents/IluApp/jellyfin/config:/config \
    -v ~/Documents/IluApp/jellyfin/cache:/cache \
    -v ~/Documents/IluApp/jellyfin/media:/media \
    docker.io/jellyfin/jellyfin:latest
```

**URL Web**: `http://localhost:8096`

#### b. **Pod Jellyseerr**
```bash
podman pod create --name jellyseerr-pod -p 5055:5055
podman run -dt --name jellyseerr --pod jellyseerr-pod \
    -v ~/Documents/IluApp/jellyseerr/config:/config \
    -v ~/Documents/IluApp/jellyseerr/downloads:/downloads \
    docker.io/fallenbagel/jellyseerr:latest
```

**URL Web**: `http://localhost:5055`

#### c. **Pod qBittorrent**
```bash
podman pod create --name qbittorrent-pod -p 8080:8080
podman run -dt --name qbittorrent --pod qbittorrent-pod \
    -v ~/Documents/IluApp/qbittorrent/config:/config \
    -v ~/Documents/IluApp/qbittorrent/downloads:/downloads \
    docker.io/linuxserver/qbittorrent:latest
```

**URL Web**: `http://localhost:8080`

#### d. **Pod Prowlarr**
```bash
podman pod create --name prowlarr-pod -p 9696:9696
podman run -dt --name prowlarr --pod prowlarr-pod \
    -v ~/Documents/IluApp/prowlarr/config:/config \
    -v ~/Documents/IluApp/prowlarr/downloads:/downloads \
    docker.io/linuxserver/prowlarr:latest
```

**URL Web**: `http://localhost:9696`

#### e. **Pod Radarr**
```bash
podman pod create --name radarr-pod -p 7878:7878
podman run -dt --name radarr --pod radarr-pod \
    -v ~/Documents/IluApp/radarr/config:/config \
    -v ~/Documents/IluApp/radarr/movies:/movies \
    -v ~/Documents/IluApp/prowlarr/downloads:/downloads \
    docker.io/linuxserver/radarr:latest
```

**URL Web**: `http://localhost:7878`

#### f. **Pod Sonarr**
```bash
podman pod create --name sonarr-pod -p 8989:8989
podman run -dt --name sonarr --pod sonarr-pod \
    -v ~/Documents/IluApp/sonarr/config:/config \
    -v ~/Documents/IluApp/sonarr/tv:/tv \
    -v ~/Documents/IluApp/prowlarr/downloads:/downloads \
    docker.io/linuxserver/sonarr:latest
```

**URL Web**: `http://localhost:8989`

### 3. **Vérifier l'IP interne des Pods pour l'intercommunication**

Pour que les conteneurs de différents pods puissent communiquer entre eux, vous pouvez obtenir l'adresse IP interne d'un pod avec la commande suivante :

```bash
podman pod inspect <nom_du_pod> --format '{{.IP}}'
```

Cela vous donne l'IP interne du pod, permettant ainsi aux conteneurs d'interagir.

### 4. **Commandes pour la gestion des Pods**

- **Démarrer un pod** :
  ```bash
  podman pod start <nom_du_pod>
  ```
  
- **Arrêter un pod** :
  ```bash
  podman pod stop <nom_du_pod>
  ```

- **Supprimer un pod** (en arrêtant les conteneurs associés) :
  ```bash
  podman pod rm -f <nom_du_pod>
  ```

### 5. **Script `start_all.sh` et `stop_all.sh` pour démarrer et arrêter tous les pods**

Vous pouvez créer deux scripts pour démarrer et arrêter tous vos pods en une seule commande.

#### a. **Script `start_all.sh`** (pour démarrer tous les pods) :
```bash
#!/bin/bash

# Démarrer tous les pods
podman pod start jellyfin-pod
podman pod start jellyseerr-pod
podman pod start qbittorrent-pod
podman pod start prowlarr-pod
podman pod start radarr-pod
podman pod start sonarr-pod

echo "Tous les pods sont démarrés."
```

#### b. **Script `stop_all.sh`** (pour arrêter tous les pods) :
```bash
#!/bin/bash

# Arrêter tous les pods
podman pod stop jellyfin-pod
podman pod stop jellyseerr-pod
podman pod stop qbittorrent-pod
podman pod stop prowlarr-pod
podman pod stop radarr-pod
podman pod stop sonarr-pod

echo "Tous les pods sont arrêtés."
```

#### c. **Rendre les scripts exécutables**
```bash
chmod +x start_all.sh
chmod +x stop_all.sh
```

### Résumé

- **Podman** permet de gérer des conteneurs et des pods sur macOS via une machine virtuelle.
- Nous avons installé Podman, créé des pods pour **Jellyfin**, **Jellyseerr**, **qBittorrent**, **Prowlarr**, **Radarr**, et **Sonarr**.
- Chaque service possède son propre port d'interface web, accessible via `http://localhost:<port>`.
- Vous pouvez gérer les pods avec des commandes simples (`start`, `stop`, `rm`).
- Un script `start_all.sh` et `stop_all.sh` permet de démarrer et d'arrêter tous les pods en une seule commande.

Cela vous permet de gérer facilement l'ensemble de vos services multimédia en conteneurs !

---
---
---

## 🅱️ - **Méthode Podman-Compose (Fast)**

Voici une version mise à jour du tutoriel avec une précision sur le bon répertoire à utiliser.  

---

### 1️⃣ **Installation de Podman et Podman Compose sur macOS**  
Sur macOS, **Podman** fonctionne via une machine virtuelle. Commençons par l’installer avec **Homebrew**.

#### ➜ **Installer Homebrew** (si ce n'est pas déjà fait)  
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

#### ➜ **Installer Podman et Podman Compose**  
```bash
brew install podman podman-compose
```

#### ➜ **Configurer la machine virtuelle Podman**  
```bash
podman machine init
podman machine start
```

---

### 2️⃣ **Créer le Fichier `podman-compose.yml`**
Dans un dossier dédié (`~/Documents/HesFlix`), crée un fichier nommé `podman-compose.yml` et copie-colle ceci :

```yaml
version: '3.8'

services:
  jellyfin:
    image: docker.io/jellyfin/jellyfin:latest
    container_name: jellyfin
    restart: always
    ports:
      - "8096:8096"
    volumes:
      - ~/Documents/HesFlix/jellyfin/config:/config
      - ~/Documents/HesFlix/jellyfin/cache:/cache
      - ~/Documents/HesFlix/jellyfin/media:/media

  jellyseerr:
    image: docker.io/fallenbagel/jellyseerr:latest
    container_name: jellyseerr
    restart: always
    ports:
      - "5055:5055"
    volumes:
      - ~/Documents/HesFlix/jellyseerr/config:/config
      - ~/Documents/HesFlix/jellyseerr/downloads:/downloads

  qbittorrent:
    image: docker.io/linuxserver/qbittorrent:latest
    container_name: qbittorrent
    restart: always
    ports:
      - "8080:8080"
    volumes:
      - ~/Documents/HesFlix/qbittorrent/config:/config
      - ~/Documents/HesFlix/qbittorrent/downloads:/downloads

  prowlarr:
    image: docker.io/linuxserver/prowlarr:latest
    container_name: prowlarr
    restart: always
    ports:
      - "9696:9696"
    volumes:
      - ~/Documents/HesFlix/prowlarr/config:/config
      - ~/Documents/HesFlix/prowlarr/downloads:/downloads

  radarr:
    image: docker.io/linuxserver/radarr:latest
    container_name: radarr
    restart: always
    ports:
      - "7878:7878"
    volumes:
      - ~/Documents/HesFlix/radarr/config:/config
      - ~/Documents/HesFlix/radarr/movies:/movies
      - ~/Documents/HesFlix/prowlarr/downloads:/downloads

  sonarr:
    image: docker.io/linuxserver/sonarr:latest
    container_name: sonarr
    restart: always
    ports:
      - "8989:8989"
    volumes:
      - ~/Documents/HesFlix/sonarr/config:/config
      - ~/Documents/HesFlix/sonarr/tv:/tv
      - ~/Documents/HesFlix/prowlarr/downloads:/downloads
```

---

### 3️⃣ **Démarrer les Services**
#### 📍 **Se placer dans le bon répertoire**
Avant de lancer les services, assure-toi d’être dans le bon dossier :
```bash
cd ~/Documents/HesFlix
```

#### 🚀 **Lancer tous les services**
```bash
podman-compose up -d
```

- **L'option `-d` permet d’exécuter les conteneurs en arrière-plan**.
- **Vérifier que tout fonctionne** avec :
  ```bash
  podman ps
  ```

---

### 4️⃣ **Gérer les Services**
#### 🔴 **Arrêter tous les services**  
```bash
podman-compose down
```

#### 🟢 **Redémarrer tous les services**  
```bash
podman-compose up -d
```

---

### 5️⃣ **Automatisation avec des Scripts**
Crée deux scripts pour simplifier le démarrage et l’arrêt des services.

📌 **Script `start_all.sh` :**
```bash
#!/bin/bash
cd ~/Documents/HesFlix
podman-compose up -d
echo "Tous les services sont démarrés."
```

📌 **Script `stop_all.sh` :**
```bash
#!/bin/bash
cd ~/Documents/HesFlix
podman-compose down
echo "Tous les services sont arrêtés."
```

#### ➜ **Rendre les scripts exécutables**
```bash
chmod +x start_all.sh stop_all.sh
```

#### ➜ **Exécuter les scripts**
- Démarrer tous les services :  
  ```bash
  ./start_all.sh
  ```
- Arrêter tous les services :  
  ```bash
  ./stop_all.sh
  ```

---

### 🎯 **Résumé**
✅ **Installation** de Podman et Podman Compose  
✅ **Création du fichier `podman-compose.yml`** pour simplifier la configuration  
✅ **Démarrage des services** avec `podman-compose up -d`  
✅ **Gestion facile** des services avec `start_all.sh` et `stop_all.sh`  

---

#### 🌍 **Accès aux interfaces web**
| Service    | URL Web                      |
|------------|------------------------------|
| Jellyfin   | `http://localhost:8096`      |
| Jellyseerr | `http://localhost:5055`      |
| qBittorrent | `http://localhost:8080`     |
| Prowlarr   | `http://localhost:9696`      |
| Radarr     | `http://localhost:7878`      |
| Sonarr     | `http://localhost:8989`      |

---

🚀 **Avec cette configuration, ton serveur multimédia est opérationnel en quelques commandes !**  
Dis-moi si tu veux d'autres ajustements. 😊
