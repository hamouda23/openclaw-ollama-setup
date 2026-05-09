# OpenClaw + Ollama + Telegram

> Framework d'agents IA autonomes avec interface Telegram et modèles locaux

[![OpenClaw](https://img.shields.io/badge/OpenClaw-2026.5.6-blue)](https://github.com/openclaw/openclaw)
[![Ollama](https://img.shields.io/badge/Ollama-0.17+-green)](https://ollama.com)
[![Status](https://img.shields.io/badge/Status-Operational-success)](https://github.com)

## 📋 Vue d'ensemble

Ce projet documente la configuration complète d'**OpenClaw**, un framework open-source pour créer et déployer des agents IA autonomes. L'installation utilise **Ollama** pour l'exécution locale de modèles d'IA et **Telegram** comme interface de messagerie.

### ✨ Fonctionnalités

- 🤖 Agent IA autonome avec modèles locaux via Ollama
- 💬 Interface Telegram via bot personnalisé
- 🖥️ Interface TUI (Terminal User Interface)
- 🔒 Exécution 100% locale — aucune clé API requise
- 🔌 Architecture modulaire et extensible

## 🏗️ Architecture

```
┌─────────────────────────────────────────────┐
│         COUCHE UTILISATEUR                  │
│  • Terminal TUI                             │
│  • Telegram Bot                             │
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
│  • Ollama (local)                           │
│  • OpenAI, Anthropic (cloud, optionnel)     │
│  • Sélection dynamique des modèles          │
└─────────────────────────────────────────────┘
```

## 🚀 Installation

### Prérequis

```bash
# Vérifier Node.js (v18+ requis)
node --version

# Installer Ollama
curl -fsSL https://ollama.com/install.sh | sh

# Installer OpenClaw
npm install -g openclaw
```

### Configuration rapide

```bash
# Télécharger un modèle local
ollama pull qwen2.5:7b

# Configurer OpenClaw
openclaw onboard --non-interactive \
  --auth-choice ollama \
  --accept-risk

# Démarrer le gateway
openclaw gateway start

# Lancer l'interface
openclaw tui
```

## 🧠 Choisir le bon modèle local

Le choix du modèle dépend directement de ton matériel. Voici les concepts clés :

### VRAM vs RAM vs Contexte

- **VRAM** — mémoire du GPU, là où le modèle est chargé. Un modèle 100% en VRAM est 5 à 10x plus rapide qu'un modèle qui déborde en RAM.
- **RAM** — utilisée pour le contexte (historique de la conversation en cours). Plus le contexte est grand, plus la RAM est consommée.
- **Contexte** — nombre de tokens que le modèle peut garder en mémoire dans une conversation. Pour les agents avec outils, un minimum de 32 000 tokens est recommandé.

> 💡 **Règle de base** : À la quantization Q4_K_M, prévoir environ 0.6 GB par milliard de paramètres. Un modèle 7B nécessite donc environ 4 à 6 GB de VRAM.

### Tableau des modèles testés (GPU Quadro P4000 — 8GB VRAM)

| Modèle | Taille | Contexte | GPU | Recommandé |
|--------|--------|----------|-----|------------|
| `qwen2.5:7b` | 4.7 GB | 32 768 | ✅ 100% | ⭐ Usage général |
| `mistral:latest` | 4.4 GB | 32 768 | ✅ 100% | ✅ Raisonnement |
| `qwen3:8b` | 5.2 GB | 40 960 | ✅ 100% | ⚠️ Thinking mode lent |
| `deepseek-coder:6.7b` | 3.8 GB | 16 384 | ✅ 100% | ✅ Code |
| `gemma4:e4b` | 5.0 GB | 8 192 | ⚠️ 76% | ❌ Contexte trop petit |
| `gemma4:latest` | 9.6 GB | 131 072 | ❌ 33% | ❌ Dépasse la VRAM |
| `llama3.1:8b` | 4.9 GB | 131 072 | ❌ — | ❌ Nécessite 20 GB RAM |

> ⚠️ **Attention au thinking mode** : Les modèles `qwen3:8b` ont un mode de réflexion activé par défaut (`thinking=medium`) qui génère des tokens internes avant de répondre. Cela ralentit significativement les réponses dans OpenClaw. Utiliser `qwen2.5:7b` ou `mistral:latest` pour éviter ce problème.

> ⚠️ **Attention au contexte et RAM** : Un grand contexte (131 072 tokens) peut nécessiter jusqu'à 20 GB de RAM même si le modèle est petit en VRAM. Toujours vérifier avec `ollama ps` après chargement.

### Modèle recommandé pour 8GB VRAM

```bash
ollama pull qwen2.5:7b
```

- 100% GPU ✅
- 32 768 tokens de contexte ✅
- Supporte les outils (tool calling) ✅
- Pas de thinking mode ✅

## 📚 Documentation

- [Guide d'installation complet](docs/Installation.md)
- [Structure d'OpenClaw](docs/STRUCTURE.md) — arborescence complète
- [Configuration détaillée](docs/configuration.md)
- [Résolution de problèmes](docs/Troubleshooting.md)
- [Référence des commandes](docs/Commands.md)
- [Scripts utiles](docs/Scripts.md)

## 🔧 Configuration

### Fichier principal

| Fichier | Description |
|---------|-------------|
| `~/.openclaw/openclaw.json` | Configuration globale (gateway, modèles, canaux) |

### Modèle par défaut

```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "ollama/qwen2.5:7b"
      }
    }
  }
}
```

## 💡 Utilisation

### Interface Terminal (TUI)

```bash
openclaw tui
```

### Via Telegram

1. Créer un bot via [@BotFather](https://t.me/BotFather)
2. Obtenir votre ID numérique via [@getmyid_bot](https://t.me/getmyid_bot)
3. Configurer via CLI :

```bash
openclaw config set channels.telegram.botToken "VOTRE_TOKEN"
openclaw config set channels.telegram.allowFrom '["VOTRE_ID"]'
openclaw config set channels.telegram.enabled true
openclaw gateway restart
```

4. Envoyer un message à votre bot — approuver le pairing si nécessaire :

```bash
openclaw pairing approve telegram CODE
```

### Commandes essentielles

```bash
# Statut du gateway
openclaw gateway status

# Logs en temps réel
openclaw logs --follow

# Diagnostic complet
openclaw doctor

# Lister les modèles disponibles
ollama list

# Voir le modèle actif et l'utilisation GPU
ollama ps
```

### 🔄 Changer de modèle

```bash
openclaw config set agents.defaults.model.primary "ollama/mistral:latest"
openclaw gateway restart
```

**Modèles disponibles sur ce système :**

| Modèle | Usage |
|--------|-------|
| `qwen2.5:7b` | Usage général (par défaut) |
| `mistral:latest` | Raisonnement |
| `qwen3:8b` | Avancé (thinking mode) |
| `deepseek-coder:6.7b` | Code |
| `nomic-embed-text:latest` | Embeddings RAG |

## 🐛 Problèmes courants

### Missing gateway auth token / Missing gateway.mode

```bash
openclaw onboard --non-interactive \
  --auth-choice ollama \
  --accept-risk
```

### Modèle trop lent (thinking mode)

Si le modèle met plusieurs minutes à répondre, c'est probablement le thinking mode. Changer vers `qwen2.5:7b` ou `mistral:latest`.

### Modèle dépasse la RAM disponible

```
model requires more system memory (X GiB) than is available
```

Choisir un modèle plus petit. Vérifier avec `ollama ps` que le modèle tourne à 100% GPU.

### Réinstallation complète

```bash
pkill -f openclaw
rm -f ~/.npm-global/bin/openclaw
rm -rf ~/.npm-global/lib/node_modules/openclaw
rm -rf ~/.openclaw
npm install -g openclaw
openclaw onboard --non-interactive --auth-choice ollama --accept-risk
```

## 💾 Sauvegarde de la configuration

La configuration d'OpenClaw est versionnée avec Git dans `~/.openclaw` :

```bash
# Avant toute modification
cd ~/.openclaw
git add openclaw.json
git commit -m "avant modification - description"

# Restaurer en cas de problème
git log --oneline
git checkout <commit-id> -- openclaw.json
openclaw gateway restart
```

> ⚠️ Le dépôt Git de `~/.openclaw` est **local uniquement** — ne jamais pousser sur GitHub car il contient des tokens sensibles.

## 📦 Versions

- **OpenClaw** : 2026.5.6 (c97b9f7)
- **Node.js** : v22+ recommandé
- **Modèle par défaut** : qwen2.5:7b (100% GPU, 32k contexte)

## 🔗 Liens utiles

- [OpenClaw GitHub](https://github.com/openclaw/openclaw)
- [Documentation OpenClaw](https://docs.openclaw.ai)
- [Ollama Official](https://ollama.com)
- [Telegram Bot API](https://core.telegram.org/bots/api)

## 📝 Statut

✅ **Opérationnel** — OpenClaw 2026.5.6 + Ollama + Telegram

- [x] Installation via `openclaw onboard`
- [x] Modèle local `qwen2.5:7b` (100% GPU)
- [x] Interface TUI opérationnelle
- [x] Bot Telegram configuré
- [x] Sauvegarde Git locale de `~/.openclaw`

## 👤 Auteur

**Samir**

Dernière mise à jour : Mai 2026

## 📄 License

Documentation libre d'utilisation et de modification.

---

💡 **Astuce** : Avant toute modification de configuration, faire un commit Git dans `~/.openclaw` pour pouvoir revenir en arrière facilement.