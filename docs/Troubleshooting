# Guide de résolution de problèmes

Ce guide répertorie **toutes les erreurs rencontrées** lors de la configuration d'OpenClaw avec Ollama et Telegram, ainsi que leurs solutions détaillées.

## Table des matières

1. [Erreur de commande TUI](#erreur-1--openclaw-tui-command-not-found)
2. [Validation de configuration](#erreur-2--config-validation-failed)
3. [Erreur HTTP 403](#erreur-3--http-403-access-to-model-denied)
4. [Clé API manquante](#erreur-4--no-api-key-found-for-provider-ollama)
5. [Résolution Telegram](#erreur-5--telegram-could-not-resolve-username)
6. [Connexion Ollama refusée](#erreur-6--connection-refused-sur-ollama)
7. [Port déjà utilisé](#erreur-7--address-already-in-use)
8. [Modèle introuvable](#erreur-8--model-not-found)
9. [Configuration non appliquée](#erreur-9--configuration-non-appliquée)

---

## ERREUR 1 : `openclaw-tui: command not found`

### 🔴 Symptôme

```bash
$ openclaw-tui
-bash: openclaw-tui: command not found
```

### 🔍 Cause

La commande correcte n'est pas `openclaw-tui` (avec tiret) mais `openclaw tui` (avec espace).

### ✅ Solution

```bash
# ❌ MAUVAIS
openclaw-tui

# ✅ BON
openclaw tui
```

### 📝 Note

OpenClaw utilise une structure de sous-commandes :
- `openclaw tui` → Interface utilisateur
- `openclaw gateway restart` → Redémarrer le gateway
- `openclaw logs` → Voir les logs

---

## ERREUR 2 : `Config validation failed: Unrecognized key: "provider"`

### 🔴 Symptôme

```bash
$ openclaw config set provider ollama
Error: Config validation failed: Unrecognized key: "provider"
```

### 🔍 Cause

Les commandes `openclaw config set provider/model` n'existent pas dans cette version. La configuration se fait via modification directe des fichiers JSON.

### ✅ Solution

Modifier manuellement les fichiers de configuration :

```bash
# Changer le provider principal
sed -i 's|"primary": "modelstudio/.*"|"primary": "ollama/qwen3.5:cloud"|' ~/.openclaw/openclaw.json

# Ou éditer manuellement
nano ~/.openclaw/openclaw.json
```

### 📁 Fichiers à modifier

- Configuration globale : `~/.openclaw/openclaw.json`
- Modèles de l'agent : `~/.openclaw/agents/main/agent/models.json`
- Authentification : `~/.openclaw/agents/main/agent/auth-profiles.json`

---

## ERREUR 3 : `HTTP 403: Access to model denied`

### 🔴 Symptôme

```bash
HTTP 403: Access to model denied.
agent main (main) | modelstudio/qwen3.5-plus | tokens ?/1.0m
```

### 🔍 Cause

L'agent utilise toujours **ModelStudio** (provider cloud payant) qui nécessite :
- Une clé API valide
- Des droits d'accès que nous n'avons pas

Le problème vient de la configuration par défaut d'OpenClaw.

### ✅ Solution

#### Étape 1 : Modifier la configuration globale

```bash
sed -i 's|"primary": "modelstudio/qwen3.5-plus"|"primary": "ollama/qwen3.5:cloud"|' ~/.openclaw/openclaw.json
```

Vérifier :
```bash
cat ~/.openclaw/openclaw.json | grep primary
# Doit afficher : "primary": "ollama/qwen3.5:cloud"
```

#### Étape 2 : Modifier la configuration de l'agent

```bash
# Ouvrir le fichier
nano ~/.openclaw/agents/main/agent/models.json

# Ajouter :
{
  "defaultModel": "qwen3.5:cloud",
  "providers": {
    "ollama": {
      "models": ["qwen3.5:cloud"]
    }
  }
}
```

#### Étape 3 : Redémarrer le gateway

```bash
openclaw gateway restart
```

#### Étape 4 : Vérifier

```bash
openclaw tui
# Le modèle devrait maintenant être "qwen3.5:cloud"
```

### 🎯 Vérification rapide

```bash
# Vérifier qu'Ollama fonctionne
curl http://localhost:11434/api/tags

# Vérifier que le modèle est disponible
ollama list | grep qwen3.5
```

---

## ERREUR 4 : `No API key found for provider "ollama"`

### 🔴 Symptôme

```bash
Agent failed before reply: No API key found for provider "ollama".
Auth store: /home/samir/.openclaw/agents/main/agent/auth-profiles.json
```

### 🔍 Cause

Le fichier `auth-profiles.json` ne contient pas de profil d'authentification pour Ollama, seulement pour ModelStudio.

### ✅ Solution complète

#### Méthode 1 : Édition manuelle (recommandée)

```bash
nano ~/.openclaw/agents/main/agent/auth-profiles.json
```

Remplacer tout le contenu par :

```json
{
  "profiles": {
    "ollama:default": {
      "type": "api_key",
      "provider": "ollama",
      "key": "ollama-local"
    },
    "modelstudio:default": {
      "type": "api_key",
      "provider": "modelstudio",
      "key": "YOUR_KEY_IF_NEEDED"
    }
  },
  "lastGood": {
    "ollama": "ollama:default",
    "modelstudio": "modelstudio:default"
  }
}
```

#### Méthode 2 : Commande automatique

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

#### Vérification

```bash
# Afficher le fichier
cat ~/.openclaw/agents/main/agent/auth-profiles.json

# Redémarrer
openclaw gateway restart

# Tester
openclaw tui
```

### 📝 Explication

- `"ollama:default"` : Nom du profil
- `"type": "api_key"` : Type d'authentification
- `"provider": "ollama"` : Provider utilisé
- `"key": "ollama-local"` : Clé (fictive pour Ollama local)
- `"lastGood"` : Dernier profil utilisé avec succès

---

## ERREUR 5 : `Telegram Could not resolve: @username`

### 🔴 Symptôme

```bash
Telegram allowFrom (numeric sender id; @username resolves to id)
@ouais23bot
Could not resolve: @ouais23bot
```

### 🔍 Cause

Le champ `allowFrom` dans la configuration Telegram attend un **ID numérique** d'utilisateur, pas un nom d'utilisateur (`@username`).

### ✅ Solution

#### Étape 1 : Obtenir votre ID utilisateur

1. Ouvrir Telegram
2. Chercher [@userinfobot](https://t.me/userinfobot)
3. Envoyer `/start`
4. Le bot retourne votre ID (ex: `123456789`)

#### Étape 2 : Modifier la configuration

```bash
nano ~/.openclaw/openclaw.json
```

Modifier la section `channels` :

```json
{
  "channels": {
    "telegram": {
      "enabled": true,
      "token": "VOTRE_TOKEN_BOT",
      "allowFrom": ["123456789"]
    }
  }
}
```

⚠️ **IMPORTANT** : Utiliser l'ID **numérique**, PAS le `@username` !

#### Étape 3 : Redémarrer

```bash
openclaw gateway restart
```

### 📝 Exemple complet

```json
{
  "primary": "ollama/qwen3.5:cloud",
  "channels": {
    "telegram": {
      "enabled": true,
      "token": "1234567890:ABCdefGHIjklMNOpqrsTUVwxyz",
      "allowFrom": ["987654321", "123456789"]
    }
  }
}
```

---

## ERREUR 6 : `connection refused` sur Ollama

### 🔴 Symptôme

```bash
$ curl http://localhost:11434/api/tags
curl: (7) Failed to connect to localhost port 11434: Connection refused
```

### 🔍 Cause

Le serveur Ollama n'est pas démarré.

### ✅ Solution

#### Démarrer Ollama

```bash
# En arrière-plan
ollama serve &

# Ou dans un terminal séparé
ollama serve
```

#### Vérifier qu'il tourne

```bash
curl http://localhost:11434/api/tags
```

Sortie attendue :
```json
{
  "models": [
    {
      "name": "qwen3.5:cloud",
      "modified_at": "2026-04-01T10:30:00Z",
      "size": 4700000000
    }
  ]
}
```

#### Vérifier le processus

```bash
ps aux | grep ollama
```

### 🔧 Automatiser le démarrage

#### Linux (systemd)

```bash
# Créer un service
sudo systemctl enable ollama
sudo systemctl start ollama
```

#### macOS (launchd)

Ollama se lance automatiquement après installation.

---

## ERREUR 7 : `address already in use` sur le port 11434

### 🔴 Symptôme

```bash
$ ollama serve
Error: listen tcp 127.0.0.1:11434: bind: address already in use
```

### 🔍 Cause

Ollama tourne **déjà en arrière-plan**. Pas besoin de le relancer !

### ✅ Solution

#### Option 1 : Ne rien faire (recommandé)

```bash
# Vérifier qu'Ollama fonctionne
curl http://localhost:11434/api/tags

# Si ça marche, tout va bien !
```

#### Option 2 : Redémarrer complètement

```bash
# Tuer le processus existant
pkill -f ollama

# Attendre 2 secondes
sleep 2

# Relancer
ollama serve &
```

#### Trouver le processus

```bash
# Voir le PID
ps aux | grep ollama

# Ou avec lsof
sudo lsof -i :11434
```

---

## ERREUR 8 : `model not found`

### 🔴 Symptôme

```bash
Error: model "qwen3.5:cloud" not found
```

### 🔍 Cause

Le modèle n'est pas téléchargé localement ou n'est pas accessible.

### ✅ Solution

#### Télécharger le modèle

```bash
ollama pull qwen3.5:cloud
```

Sortie attendue :
```
pulling manifest
pulling 8934d96d3f08... 100% ▕████████████▏ 4.7 GB
pulling 8c17c2ebb0ea... 100% ▕████████████▏ 7.0 KB
pulling 7c23fb36d801... 100% ▕████████████▏ 4.8 KB
pulling 2e0493f67d0c... 100% ▕████████████▏   59 B
pulling fa304d675061... 100% ▕████████████▏   91 B
pulling 42ba7f8a01dd... 100% ▕████████████▏  557 B
verifying sha256 digest
writing manifest
success
```

#### Vérifier les modèles disponibles

```bash
ollama list
```

#### Tester le modèle

```bash
ollama run qwen3.5:cloud "Hello, world!"
```

### 📝 Modèles alternatifs

Si `qwen3.5:cloud` ne fonctionne pas :

```bash
# Llama 3.2 (3B)
ollama pull llama3.2

# Mistral 7B
ollama pull mistral

# Phi-3
ollama pull phi3
```

Puis modifier la configuration OpenClaw en conséquence.

---

## ERREUR 9 : Configuration non appliquée après modification

### 🔴 Symptôme

Après modification des fichiers JSON, OpenClaw utilise **toujours l'ancienne configuration**.

### 🔍 Cause

Le Gateway OpenClaw garde la configuration en **mémoire** et ne la recharge pas automatiquement.

### ✅ Solution

#### Méthode 1 : Redémarrer le gateway (recommandée)

```bash
openclaw gateway restart
```

#### Méthode 2 : Tuer tous les processus

```bash
# Tuer OpenClaw
pkill -f openclaw

# Attendre 2 secondes
sleep 2

# Relancer l'interface
openclaw tui
```

#### Méthode 3 : Redémarrage complet

```bash
# Tuer Ollama et OpenClaw
pkill -f ollama
pkill -f openclaw

# Attendre
sleep 3

# Redémarrer Ollama
ollama serve &

# Attendre qu'Ollama démarre
sleep 2

# Redémarrer OpenClaw
openclaw gateway restart

# Lancer TUI
openclaw tui
```

### 🔍 Vérifier que la config est chargée

```bash
# Voir les logs de démarrage
openclaw logs --tail 50 | grep "primary"

# Devrait afficher quelque chose comme :
# "Loaded primary model: ollama/qwen3.5:cloud"
```

---

## 🛠️ Outils de diagnostic

### Commande complète de vérification

```bash
#!/bin/bash

echo "=== DIAGNOSTIC OPENCLAW + OLLAMA ==="

echo -e "\n1. Vérification Ollama..."
if curl -s http://localhost:11434/api/tags > /dev/null; then
    echo "✅ Ollama accessible"
    ollama list
else
    echo "❌ Ollama non accessible"
fi

echo -e "\n2. Vérification OpenClaw..."
if command -v openclaw &> /dev/null; then
    echo "✅ OpenClaw installé"
    openclaw --version
else
    echo "❌ OpenClaw non installé"
fi

echo -e "\n3. Configuration principale..."
cat ~/.openclaw/openclaw.json | grep -A2 "primary"

echo -e "\n4. Processus actifs..."
ps aux | grep -E "(ollama|openclaw)" | grep -v grep

echo -e "\n5. Ports utilisés..."
sudo lsof -i :11434 2>/dev/null || echo "Port 11434 libre"
sudo lsof -i :18789 2>/dev/null || echo "Port 18789 libre"

echo -e "\n=== FIN DU DIAGNOSTIC ==="
```

### Diagnostic intégré

```bash
openclaw doctor
```

---

## 📞 Besoin d'aide ?

Si aucune de ces solutions ne fonctionne :

1. Vérifier les [logs détaillés](COMMANDS.md#logs)
2. Consulter la [documentation officielle](https://docs.openclaw.dev)
3. Ouvrir une issue sur [GitHub](https://github.com/openclaw/openclaw/issues)

---

[← Retour au README](../README.md) | [Guide de configuration →](CONFIGURATION.md)
