# Guide de configuration

Documentation complète de tous les fichiers et paramètres de configuration pour OpenClaw avec Ollama.

## Table des matières

- [Vue d'ensemble](#vue-densemble)
- [Configuration globale](#configuration-globale)
- [Configuration de l'agent](#configuration-de-lagent)
- [Authentification](#authentification)
- [Configuration Telegram](#configuration-telegram)
- [Modèles disponibles](#modèles-disponibles)

---

## Vue d'ensemble

### Arborescence des fichiers

```
~/.openclaw/
├── openclaw.json                          # Configuration globale
├── logs/                                  # Fichiers de logs
├── cache/                                 # Cache temporaire
└── agents/
    └── main/
        └── agent/
            ├── models.json                # Modèles de l'agent
            ├── auth-profiles.json         # Profils d'authentification
            └── config.json                # Config spécifique à l'agent
```

### Hiérarchie de configuration

1. **Configuration globale** (`openclaw.json`) → Paramètres système
2. **Configuration de l'agent** (`models.json`) → Modèles et providers
3. **Authentification** (`auth-profiles.json`) → Clés API et credentials

---

## Configuration globale

### Fichier : `~/.openclaw/openclaw.json`

Ce fichier contient la configuration principale d'OpenClaw.

#### Structure complète

```json
{
  "primary": "ollama/qwen3.5:cloud",
  "gateway": {
    "host": "127.0.0.1",
    "port": 18789,
    "protocol": "ws"
  },
  "channels": {
    "telegram": {
      "enabled": false,
      "token": "",
      "allowFrom": []
    },
    "discord": {
      "enabled": false
    },
    "slack": {
      "enabled": false
    }
  },
  "logging": {
    "level": "info",
    "file": true,
    "console": true
  },
  "storage": {
    "type": "local",
    "path": "~/.openclaw/data"
  }
}
```

#### Paramètres principaux

| Paramètre | Type | Description | Valeur par défaut |
|-----------|------|-------------|-------------------|
| `primary` | string | Modèle principal utilisé | `"modelstudio/qwen3.5-plus"` |
| `gateway.host` | string | Adresse du gateway | `"127.0.0.1"` |
| `gateway.port` | number | Port du gateway | `18789` |
| `logging.level` | string | Niveau de log (debug/info/warn/error) | `"info"` |

#### Configuration minimale pour Ollama

```json
{
  "primary": "ollama/qwen3.5:cloud"
}
```

#### Modifier la configuration

##### Méthode 1 : Édition manuelle

```bash
nano ~/.openclaw/openclaw.json
```

##### Méthode 2 : Via sed

```bash
# Changer le modèle primaire
sed -i 's|"primary": ".*"|"primary": "ollama/qwen3.5:cloud"|' ~/.openclaw/openclaw.json

# Changer le niveau de logs
sed -i 's|"level": ".*"|"level": "debug"|' ~/.openclaw/openclaw.json
```

##### Méthode 3 : Via jq

```bash
# Installer jq si nécessaire
sudo apt install jq

# Modifier le modèle primaire
jq '.primary = "ollama/qwen3.5:cloud"' ~/.openclaw/openclaw.json > tmp.$$.json && mv tmp.$$.json ~/.openclaw/openclaw.json
```

---

## Configuration de l'agent

### Fichier : `~/.openclaw/agents/main/agent/models.json`

Ce fichier définit les modèles et providers utilisés par l'agent.

#### Structure complète

```json
{
  "defaultModel": "qwen3.5:cloud",
  "providers": {
    "ollama": {
      "enabled": true,
      "endpoint": "http://localhost:11434",
      "models": [
        "qwen3.5:cloud",
        "llama3.2",
        "mistral"
      ],
      "timeout": 30000
    },
    "openai": {
      "enabled": false,
      "endpoint": "https://api.openai.com/v1",
      "models": [
        "gpt-4",
        "gpt-3.5-turbo"
      ]
    },
    "anthropic": {
      "enabled": false,
      "endpoint": "https://api.anthropic.com/v1",
      "models": [
        "claude-3-opus",
        "claude-3-sonnet"
      ]
    },
    "modelstudio": {
      "enabled": false,
      "endpoint": "https://api.modelstudio.ai/v1",
      "models": [
        "qwen3.5-plus"
      ]
    }
  },
  "routing": {
    "defaultProvider": "ollama",
    "rules": [
      {
        "pattern": ".*code.*",
        "provider": "ollama",
        "model": "qwen3.5:cloud"
      }
    ]
  }
}
```

#### Configuration minimale pour Ollama

```json
{
  "defaultModel": "qwen3.5:cloud",
  "providers": {
    "ollama": {
      "enabled": true,
      "endpoint": "http://localhost:11434",
      "models": ["qwen3.5:cloud"]
    }
  }
}
```

#### Paramètres des providers

| Provider | Endpoint par défaut | Authentification requise |
|----------|---------------------|--------------------------|
| `ollama` | `http://localhost:11434` | Non (local) |
| `openai` | `https://api.openai.com/v1` | Oui (clé API) |
| `anthropic` | `https://api.anthropic.com/v1` | Oui (clé API) |
| `modelstudio` | `https://api.modelstudio.ai/v1` | Oui (clé API) |

#### Ajouter un modèle Ollama

```bash
# 1. Télécharger le modèle
ollama pull llama3.2

# 2. Ajouter dans models.json
nano ~/.openclaw/agents/main/agent/models.json

# Ajouter "llama3.2" dans la liste des modèles
```

---

## Authentification

### Fichier : `~/.openclaw/agents/main/agent/auth-profiles.json`

Ce fichier stocke les profils d'authentification pour chaque provider.

#### Structure complète

```json
{
  "profiles": {
    "ollama:default": {
      "type": "api_key",
      "provider": "ollama",
      "key": "ollama-local",
      "endpoint": "http://localhost:11434"
    },
    "openai:default": {
      "type": "api_key",
      "provider": "openai",
      "key": "sk-...",
      "organization": "org-..."
    },
    "anthropic:default": {
      "type": "api_key",
      "provider": "anthropic",
      "key": "sk-ant-..."
    },
    "modelstudio:default": {
      "type": "api_key",
      "provider": "modelstudio",
      "key": "ms-..."
    }
  },
  "lastGood": {
    "ollama": "ollama:default",
    "openai": "openai:default",
    "anthropic": "anthropic:default",
    "modelstudio": "modelstudio:default"
  }
}
```

#### Configuration minimale pour Ollama

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

#### Ajouter une clé OpenAI

```bash
nano ~/.openclaw/agents/main/agent/auth-profiles.json
```

Ajouter :

```json
{
  "profiles": {
    "ollama:default": { ... },
    "openai:production": {
      "type": "api_key",
      "provider": "openai",
      "key": "sk-votre-cle-ici"
    }
  },
  "lastGood": {
    "ollama": "ollama:default",
    "openai": "openai:production"
  }
}
```

#### Sécurité des clés

⚠️ **IMPORTANT** : Ne jamais committer les fichiers contenant des clés API !

```bash
# Ajouter au .gitignore
echo "auth-profiles.json" >> .gitignore

# Permissions restrictives
chmod 600 ~/.openclaw/agents/main/agent/auth-profiles.json
```

---

## Configuration Telegram

### Prérequis

1. Un compte Telegram
2. Un bot créé via [@BotFather](https://t.me/BotFather)
3. Votre ID utilisateur numérique

### Étape 1 : Créer un bot

```
1. Ouvrir Telegram
2. Chercher @BotFather
3. Envoyer /newbot
4. Suivre les instructions
5. Copier le token (format: 1234567890:ABCdefGHIjklMNOpqrsTUVwxyz)
```

### Étape 2 : Obtenir votre ID

```
1. Chercher @userinfobot
2. Envoyer /start
3. Copier votre ID (ex: 987654321)
```

### Étape 3 : Configuration dans OpenClaw

```bash
nano ~/.openclaw/openclaw.json
```

Ajouter/modifier :

```json
{
  "primary": "ollama/qwen3.5:cloud",
  "channels": {
    "telegram": {
      "enabled": true,
      "token": "1234567890:ABCdefGHIjklMNOpqrsTUVwxyz",
      "allowFrom": ["987654321"],
      "polling": {
        "enabled": true,
        "interval": 1000
      },
      "webhook": {
        "enabled": false,
        "url": "",
        "port": 8443
      }
    }
  }
}
```

### Paramètres Telegram

| Paramètre | Type | Description |
|-----------|------|-------------|
| `enabled` | boolean | Activer/désactiver le bot |
| `token` | string | Token du bot depuis BotFather |
| `allowFrom` | array | Liste d'IDs autorisés à utiliser le bot |
| `polling.enabled` | boolean | Utiliser le polling (recommandé) |
| `webhook.enabled` | boolean | Utiliser un webhook (avancé) |

### Autoriser plusieurs utilisateurs

```json
{
  "allowFrom": [
    "987654321",
    "123456789",
    "555666777"
  ]
}
```

### Redémarrer après configuration

```bash
openclaw gateway restart
```

### Tester

Envoyer un message à votre bot sur Telegram :
```
Hello, bot!
```

---

## Modèles disponibles

### Modèles Ollama recommandés

| Modèle | Taille | Cas d'usage | Commande |
|--------|--------|-------------|----------|
| `qwen3.5:cloud` | ~5 GB | Usage général, conversationnel | `ollama pull qwen3.5:cloud` |
| `llama3.2` | ~3 GB | Rapide, léger, bon pour le code | `ollama pull llama3.2` |
| `mistral` | ~4 GB | Raisonnement, analyse | `ollama pull mistral` |
| `phi3` | ~2 GB | Très léger, rapide | `ollama pull phi3` |
| `codellama` | ~4 GB | Spécialisé code | `ollama pull codellama` |

### Télécharger et configurer un nouveau modèle

```bash
# 1. Télécharger
ollama pull mistral

# 2. Tester
ollama run mistral "Hello"

# 3. Ajouter dans OpenClaw
nano ~/.openclaw/agents/main/agent/models.json

# Ajouter "mistral" dans la liste des modèles ollama
```

### Changer le modèle par défaut

```bash
# Dans openclaw.json
sed -i 's|"primary": "ollama/qwen3.5:cloud"|"primary": "ollama/mistral"|' ~/.openclaw/openclaw.json

# Dans models.json
sed -i 's|"defaultModel": "qwen3.5:cloud"|"defaultModel": "mistral"|' ~/.openclaw/agents/main/agent/models.json

# Redémarrer
openclaw gateway restart
```

---

## Configuration avancée

### Routing intelligent

Utiliser différents modèles selon le type de requête :

```json
{
  "routing": {
    "defaultProvider": "ollama",
    "rules": [
      {
        "pattern": ".*code.*|.*program.*|.*debug.*",
        "provider": "ollama",
        "model": "codellama"
      },
      {
        "pattern": ".*image.*|.*photo.*",
        "provider": "openai",
        "model": "gpt-4-vision"
      },
      {
        "pattern": ".*",
        "provider": "ollama",
        "model": "qwen3.5:cloud"
      }
    ]
  }
}
```

### Logging avancé

```json
{
  "logging": {
    "level": "debug",
    "file": true,
    "console": true,
    "rotation": {
      "enabled": true,
      "maxSize": "10M",
      "maxFiles": 5
    },
    "format": "json"
  }
}
```

### Mémoire et contexte

```json
{
  "memory": {
    "enabled": true,
    "maxTokens": 4096,
    "persistence": true,
    "ttl": 86400
  },
  "context": {
    "systemPrompt": "Tu es un assistant IA utile et amical.",
    "temperature": 0.7,
    "maxTokens": 2048
  }
}
```

---

## Exemples de configurations

### Configuration complète pour Ollama uniquement

```json
// ~/.openclaw/openclaw.json
{
  "primary": "ollama/qwen3.5:cloud",
  "gateway": {
    "host": "127.0.0.1",
    "port": 18789
  },
  "channels": {
    "telegram": {
      "enabled": true,
      "token": "VOTRE_TOKEN",
      "allowFrom": ["VOTRE_ID"]
    }
  },
  "logging": {
    "level": "info"
  }
}
```

```json
// ~/.openclaw/agents/main/agent/models.json
{
  "defaultModel": "qwen3.5:cloud",
  "providers": {
    "ollama": {
      "enabled": true,
      "endpoint": "http://localhost:11434",
      "models": ["qwen3.5:cloud", "llama3.2", "mistral"]
    }
  }
}
```

```json
// ~/.openclaw/agents/main/agent/auth-profiles.json
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

### Configuration multi-provider

Pour utiliser à la fois Ollama (local) et OpenAI (cloud) :

```json
// models.json
{
  "defaultModel": "qwen3.5:cloud",
  "providers": {
    "ollama": {
      "enabled": true,
      "models": ["qwen3.5:cloud"]
    },
    "openai": {
      "enabled": true,
      "models": ["gpt-4", "gpt-3.5-turbo"]
    }
  },
  "routing": {
    "defaultProvider": "ollama",
    "rules": [
      {
        "pattern": ".*urgent.*|.*important.*",
        "provider": "openai",
        "model": "gpt-4"
      }
    ]
  }
}
```

---

## Validation et sauvegarde

### Valider la configuration

```bash
# Valider la syntaxe JSON
jq . ~/.openclaw/openclaw.json

# Validation OpenClaw
openclaw config validate

# Test complet
openclaw doctor
```

### Sauvegarder la configuration

```bash
# Créer un backup
mkdir -p ~/openclaw-backup
cp -r ~/.openclaw ~/openclaw-backup/openclaw-$(date +%Y%m%d)

# Ou avec tar
tar -czf ~/openclaw-backup-$(date +%Y%m%d).tar.gz ~/.openclaw
```

### Restaurer une sauvegarde

```bash
# Depuis un dossier
cp -r ~/openclaw-backup/openclaw-20260401 ~/.openclaw

# Depuis un tar.gz
tar -xzf ~/openclaw-backup-20260401.tar.gz -C ~/
```

---

## Troubleshooting de configuration

### La configuration n'est pas appliquée

```bash
# Vérifier la syntaxe
jq . ~/.openclaw/openclaw.json

# Redémarrer le gateway
openclaw gateway restart

# Vérifier les logs
openclaw logs --tail 50
```

### Conflit de configuration

```bash
# Lister tous les fichiers de config
find ~/.openclaw -name "*.json" -type f

# Afficher toutes les valeurs de "primary"
grep -r "primary" ~/.openclaw/*.json
```

---

[← Retour au README](../README.md) | [Guide d'installation →](INSTALLATION.md)
