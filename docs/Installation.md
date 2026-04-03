# Guide d'installation

Ce guide vous accompagne étape par étape dans l'installation et la configuration d'OpenClaw avec Ollama et Telegram.

## Table des matières

- [Prérequis](#prérequis)
- [Installation d'Ollama](#installation-dollama)
- [Installation d'OpenClaw](#installation-dopenclaw)
- [Configuration initiale](#configuration-initiale)
- [Configuration Telegram](#configuration-telegram)
- [Vérification](#vérification)

## Prérequis

### Système d'exploitation

- Linux (Ubuntu, Debian, Fedora, etc.)
- macOS
- Windows (via WSL2)

### Logiciels requis

```bash
# Node.js (v16 ou supérieur)
node --version

# npm (installé avec Node.js)
npm --version

# curl (pour les téléchargements)
curl --version
```

Si Node.js n'est pas installé :

```bash
# Ubuntu/Debian
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs

# macOS
brew install node

# Fedora
sudo dnf install nodejs
```

## Installation d'Ollama

### Méthode automatique (recommandée)

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

### Vérification de l'installation

```bash
# Vérifier la version
ollama --version
# Sortie attendue : ollama version is 0.15.2

# Démarrer le serveur
ollama serve &

# Tester la connexion
curl http://localhost:11434/api/tags
```

### Télécharger le modèle

```bash
# Télécharger qwen3.5:cloud
ollama pull qwen3.5:cloud

# Vérifier les modèles installés
ollama list
```

Sortie attendue :
```
NAME              ID              SIZE      MODIFIED
qwen3.5:cloud     abc123def456    4.7 GB    2 minutes ago
```

## Installation d'OpenClaw

### Installation via npm

```bash
# Installation globale
npm install -g openclaw

# Vérifier l'installation
openclaw --version
# Sortie attendue : 2026.4.1 (da64a97)
```

### Vérifier l'emplacement

```bash
which openclaw
# Sortie : ~/.npm-global/bin/openclaw ou /usr/local/bin/openclaw
```

## Configuration initiale

### Étape 1 : Configuration globale

Modifier le fichier de configuration principal :

```bash
nano ~/.openclaw/openclaw.json
```

Changer la ligne suivante :

```json
{
  "primary": "ollama/qwen3.5:cloud"
}
```

**Ou via commande automatique :**

```bash
sed -i 's|"primary": "modelstudio/qwen3.5-plus"|"primary": "ollama/qwen3.5:cloud"|' ~/.openclaw/openclaw.json
```

### Étape 2 : Configuration de l'agent

Modifier les modèles de l'agent :

```bash
nano ~/.openclaw/agents/main/agent/models.json
```

Ajouter :

```json
{
  "defaultModel": "qwen3.5:cloud",
  "providers": {
    "ollama": {
      "models": ["qwen3.5:cloud"]
    }
  }
}
```

**Ou via commande :**

```bash
cat > ~/.openclaw/agents/main/agent/models.json << 'EOF'
{
  "defaultModel": "qwen3.5:cloud",
  "providers": {
    "ollama": {
      "models": ["qwen3.5:cloud"]
    }
  }
}
EOF
```

### Étape 3 : Configuration de l'authentification

Créer le profil d'authentification Ollama :

```bash
nano ~/.openclaw/agents/main/agent/auth-profiles.json
```

Contenu :

```json
{
  "profiles": {
    "ollama:default": {
      "type": "api_key",
      "provider": "ollama",
      "key": "ollama-local"
    }
  },
  "lastGood": {
    "ollama": "ollama:default"
  }
}
```

**Ou via commande :**

```bash
cat > ~/.openclaw/agents/main/agent/auth-profiles.json << 'EOF'
{
  "profiles": {
    "ollama:default": {
      "type": "api_key",
      "provider": "ollama",
      "key": "ollama-local"
    }
  },
  "lastGood": {
    "ollama": "ollama:default"
  }
}
EOF
```

### Étape 4 : Redémarrage du Gateway

```bash
# Redémarrer OpenClaw
openclaw gateway restart

# Vérifier le statut
openclaw gateway status
```

## Configuration Telegram

### Étape 1 : Créer un bot Telegram

1. Ouvrir Telegram et chercher [@BotFather](https://t.me/BotFather)
2. Envoyer `/newbot`
3. Suivre les instructions pour nommer votre bot
4. Copier le token API fourni

### Étape 2 : Obtenir votre ID utilisateur

1. Chercher [@userinfobot](https://t.me/userinfobot) dans Telegram
2. Envoyer `/start`
3. Copier votre ID numérique (ex: `123456789`)

### Étape 3 : Configurer OpenClaw pour Telegram

Modifier la configuration :

```bash
nano ~/.openclaw/openclaw.json
```

Ajouter la section Telegram :

```json
{
  "channels": {
    "telegram": {
      "enabled": true,
      "token": "VOTRE_TOKEN_BOT_ICI",
      "allowFrom": ["VOTRE_ID_NUMERIQUE_ICI"]
    }
  }
}
```

### Étape 4 : Redémarrer et tester

```bash
# Redémarrer
openclaw gateway restart

# Envoyer un message à votre bot sur Telegram
```

## Vérification

### Test de l'interface TUI

```bash
openclaw tui
```

Vous devriez voir :

```
┌─────────────────────────────────────────┐
│ OpenClaw TUI                            │
│ Agent: main                             │
│ Model: qwen3.5:cloud                    │
└─────────────────────────────────────────┘

> _
```

### Test via Telegram

1. Ouvrir Telegram
2. Chercher votre bot
3. Envoyer : `Hello!`
4. Attendre la réponse de l'agent

### Vérification des logs

```bash
# Logs en temps réel
openclaw logs --follow

# Dernières lignes
openclaw logs --tail 50
```

### Diagnostic automatique

```bash
openclaw doctor
```

Sortie attendue :
```
✓ Gateway running
✓ Ollama accessible
✓ Model qwen3.5:cloud available
✓ Configuration valid
```

## Résolution de problèmes

Si vous rencontrez des erreurs, consultez le [guide de troubleshooting](TROUBLESHOOTING.md).

### Problèmes courants

#### Ollama ne démarre pas

```bash
# Vérifier si le port est déjà utilisé
sudo lsof -i :11434

# Tuer le processus si nécessaire
pkill -f ollama

# Redémarrer
ollama serve &
```

#### OpenClaw ne trouve pas le modèle

```bash
# Vérifier que le modèle est téléchargé
ollama list

# Télécharger si absent
ollama pull qwen3.5:cloud
```

#### Gateway ne démarre pas

```bash
# Tuer tous les processus OpenClaw
pkill -f openclaw

# Redémarrer
openclaw gateway restart
```

## Prochaines étapes

- ✅ Installation terminée
- 📖 Consulter la [documentation de configuration](CONFIGURATION.md)
- 🐛 Lire le [guide de troubleshooting](TROUBLESHOOTING.md)
- 💡 Explorer les [commandes utiles](COMMANDS.md)

---

[← Retour au README](../README.md)
