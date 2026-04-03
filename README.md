# OpenClaw + Ollama + Telegram

> Framework d'agents IA autonomes avec interface Telegram et modèles locaux

[![OpenClaw](https://img.shields.io/badge/OpenClaw-2026.4.1-blue)](https://github.com/openclaw/openclaw)
[![Ollama](https://img.shields.io/badge/Ollama-0.15.2-green)](https://ollama.com)
[![Status](https://img.shields.io/badge/Status-Operational-success)](https://github.com)

## 📋 Vue d'ensemble

Ce projet documente la configuration complète d'**OpenClaw**, un framework open-source pour créer et déployer des agents IA autonomes. L'installation utilise **Ollama** pour l'exécution locale de modèles d'IA et **Telegram** comme interface de messagerie.

### ✨ Fonctionnalités

- 🤖 Agent IA autonome avec le modèle `qwen3.5:cloud`
- 💬 Interface Telegram via bot personnalisé
- 🖥️ Interface TUI (Terminal User Interface)
- 🔒 Exécution 100% locale avec Ollama
- 🔌 Architecture modulaire et extensible

## 🏗️ Architecture

```
┌─────────────────────────────────────────────┐
│         COUCHE UTILISATEUR                  │
│  • Terminal TUI                             │
│  • Telegram Bot (@ouais23bot)               │
│  • Discord, Slack (supportés)               │
└──────────────────┬──────────────────────────┘
                   │
┌──────────────────▼──────────────────────────┐
│         COUCHE GATEWAY                      │
│  • WebSocket local (ws://127.0.0.1:18789)   │
│  • Gestion sessions & mémoire               │
│  • Routage des messages                     │
└──────────────────┬──────────────────────────┘
                   │
┌──────────────────▼──────────────────────────┐
│         COUCHE PROVIDER                     │
│  • Ollama (local/cloud)                     │
│  • OpenAI, Anthropic, ModelStudio           │
│  • Sélection dynamique des modèles          │
└─────────────────────────────────────────────┘
```

## 🚀 Installation

### Prérequis

```bash
# Vérifier Node.js
node --version

# Installer Ollama
curl -fsSL https://ollama.com/install.sh | sh

# Installer OpenClaw
npm install -g openclaw
```

### Configuration rapide

```bash
# 1. Télécharger le modèle
ollama pull qwen3.5:cloud

# 2. Configurer OpenClaw
sed -i 's|"primary": "modelstudio/qwen3.5-plus"|"primary": "ollama/qwen3.5:cloud"|' ~/.openclaw/openclaw.json

# 3. Démarrer le gateway
openclaw gateway restart

# 4. Lancer l'interface
openclaw tui
```

## 📚 Documentation

- [Guide d'installation complet](docs/INSTALLATION.md)
- [Configuration détaillée](docs/CONFIGURATION.md)
- [Résolution de problèmes](docs/TROUBLESHOOTING.md)
- [Référence des commandes](docs/COMMANDS.md)
- [Scripts utiles](docs/SCRIPTS.md) ⭐ **Nouveau : script de changement de modèle**

## 🔧 Configuration

### Fichiers principaux

| Fichier | Description |
|---------|-------------|
| `~/.openclaw/openclaw.json` | Configuration globale |
| `~/.openclaw/agents/main/agent/models.json` | Modèles de l'agent |
| `~/.openclaw/agents/main/agent/auth-profiles.json` | Authentification |

### Modèle par défaut

```json
{
  "primary": "ollama/qwen3.5:cloud",
  "defaultModel": "qwen3.5:cloud"
}
```

## 💡 Utilisation

### Interface Terminal (TUI)

```bash
openclaw tui
```

### Via Telegram

1. Configurer votre bot Telegram
2. Obtenir votre ID utilisateur via [@userinfobot](https://t.me/userinfobot)
3. Ajouter l'ID dans la configuration
4. Envoyer un message à votre bot !

### Commandes essentielles

```bash
# Démarrer Ollama
ollama serve

# Lister les modèles
ollama list

# Redémarrer OpenClaw
openclaw gateway restart

# Logs en temps réel
openclaw logs --follow

# Diagnostic
openclaw doctor
```

### 🔄 Changer de modèle rapidement

Utilisez le [script de basculement de modèle](docs/SCRIPTS.md#script-de-basculement-de-modèle) :

```bash
# Créer le script (voir docs/SCRIPTS.md pour le code complet)
chmod +x ~/switch-model.sh

# Changer de modèle en une commande
~/switch-model.sh qwen3.5:cloud    # Conversation générale
~/switch-model.sh deepseek-coder:6.7b  # Développement
~/switch-model.sh mistral:latest   # Raisonnement
~/switch-model.sh qwen2.5:7b       # Alternative
```

**Modèles disponibles sur ce système :**
- `qwen3.5:cloud` - Conversation générale (par défaut)
- `qwen2.5:7b` - Alternative performante
- `deepseek-coder:6.7b` - Spécialisé code
- `mistral:latest` - Raisonnement
- `nomic-embed-text:latest` - Embeddings

## 🐛 Problèmes courants

### Erreur 403: Access denied

```bash
# Changer le provider de ModelStudio vers Ollama
sed -i 's|"primary": "modelstudio/.*"|"primary": "ollama/qwen3.5:cloud"|' ~/.openclaw/openclaw.json
openclaw gateway restart
```

### No API key found

Voir le [guide de troubleshooting](docs/TROUBLESHOOTING.md#erreur-4--no-api-key-found-for-provider-ollama) complet.

## 📦 Versions

- **OpenClaw** : 2026.4.1 (da64a97)
- **Ollama** : 0.15.2
- **Node.js** : Compatible avec versions récentes
- **Modèle principal** : qwen3.5:cloud

## 🔗 Liens utiles

- [OpenClaw GitHub](https://github.com/openclaw/openclaw)
- [Documentation OpenClaw](https://docs.openclaw.dev)
- [Ollama Official](https://ollama.com)
- [Qwen Models](https://ollama.com/library/qwen)
- [Telegram Bot API](https://core.telegram.org/bots/api)

## 📝 Statut

✅ **Opérationnel** - OpenClaw + Ollama + Telegram fonctionnels

- [x] Installation et configuration
- [x] Interface TUI opérationnelle
- [x] Bot Telegram actif
- [x] Modèle qwen3.5:cloud validé

## 👤 Auteur

**Samir**

Dernière mise à jour : Avril 2026

## 📄 License

Documentation libre d'utilisation et de modification.

---

💡 **Astuce** : Consultez le [troubleshooting complet](docs/TROUBLESHOOTING.md) pour toutes les erreurs rencontrées et leurs solutions !
