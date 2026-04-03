# Structure d'OpenClaw

Documentation détaillée de l'arborescence et des fichiers de configuration d'OpenClaw.

## Table des matières

- [Arborescence globale](#arborescence-globale)
- [Fichiers de configuration](#fichiers-de-configuration)
- [Workspace (Bureau de l'agent)](#workspace-bureau-de-lagent)
- [Hiérarchie et priorités](#hiérarchie-et-priorités)
- [Fichiers générés automatiquement](#fichiers-générés-automatiquement)

---

## Arborescence globale

```
~/.openclaw/
│
├── openclaw.json                    ← CONFIG GLOBALE (PRIORITAIRE)
│
├── agents/
│   └── main/
│       └── agent/
│           ├── models.json          ← Modèles et providers de l'agent
│           └── auth-profiles.json   ← Clés API et authentification
│
├── workspace/                       ← "BUREAU" DE L'AGENT
│   ├── AGENTS.md                    ← Règles opérationnelles
│   ├── SOUL.md                      ← Personnalité et ton
│   ├── USER.md                      ← Contexte sur l'utilisateur
│   ├── IDENTITY.md                  ← Identité de l'agent (nom, emoji)
│   ├── HEARTBEAT.md                 ← Status en temps réel
│   ├── MEMORY.md                    ← Mémoire long terme
│   ├── TOOLS.md                     ← Workflows et conventions
│   └── memory/
│       └── YYYY-MM-DD.md            ← Logs quotidiens
│
├── state/                           ← ÉTAT ET CACHE (généré auto)
│
├── logs/                            ← LOGS DU SYSTÈME
│   ├── gateway.log
│   └── gateway.error.log
│
└── credentials/                     ← DONNÉES SENSIBLES (tokens)
    └── telegram/
        └── default/
            └── session.json         ← Token du bot Telegram
```

---

## Fichiers de configuration

### 1. `~/.openclaw/openclaw.json` (CONFIG GLOBALE)

**Rôle** : Configuration principale, **PRIORITAIRE** sur tous les autres fichiers.

**Contenu** :
- Modèle par défaut global (`agents.defaults.model.primary`)
- Configuration de chaque agent (`agents.list[]`)
- Port du gateway (`gateway.port`)
- Configuration des canaux (Telegram, Discord, Slack)
- Outils et plugins activés

**Structure complète** :

```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "ollama/qwen3.5:cloud"
      }
    },
    "list": [
      {
        "id": "main",
        "model": "ollama/qwen3.5:cloud",
        "name": "Assistant Principal",
        "description": "Agent conversationnel général"
      }
    ]
  },
  "gateway": {
    "port": 18789,
    "host": "127.0.0.1",
    "protocol": "ws"
  },
  "channels": {
    "telegram": {
      "enabled": true,
      "botToken": "8692829207:AAFl...",
      "allowFrom": ["5517161958"]
    },
    "discord": {
      "enabled": false
    },
    "slack": {
      "enabled": false
    }
  },
  "tools": {
    "web_search": true,
    "code_execution": false,
    "file_operations": true
  },
  "plugins": []
}
```

**Paramètres clés** :

| Paramètre | Type | Description | Valeur par défaut |
|-----------|------|-------------|-------------------|
| `agents.defaults.model.primary` | string | Modèle par défaut pour tous les agents | `"modelstudio/qwen3.5-plus"` |
| `agents.list[].id` | string | ID unique de l'agent | `"main"` |
| `agents.list[].model` | string | Modèle spécifique à cet agent | Hérite de `defaults` |
| `gateway.port` | number | Port WebSocket du gateway | `18789` |
| `channels.telegram.enabled` | boolean | Activer le bot Telegram | `false` |
| `channels.telegram.botToken` | string | Token du bot depuis BotFather | `""` |
| `channels.telegram.allowFrom` | array | IDs utilisateurs autorisés | `[]` |

**Emplacement** :
```bash
~/.openclaw/openclaw.json
```

**Modifier** :
```bash
nano ~/.openclaw/openclaw.json
```

---

### 2. `~/.openclaw/agents/main/agent/models.json`

**Rôle** : Liste des modèles disponibles pour l'agent "main" et configuration des providers.

**Contenu** :
- Providers configurés (ollama, openai, anthropic, modelstudio)
- Liste des modèles par provider
- Paramètres de chaque modèle (contextWindow, maxTokens)
- Modèle par défaut de l'agent

**Structure complète** :

```json
{
  "providers": {
    "ollama": {
      "baseUrl": "http://127.0.0.1:11434",
      "api": "ollama",
      "defaultModel": "qwen3.5:cloud",
      "models": [
        {
          "id": "qwen3.5:cloud",
          "name": "Qwen 3.5 Cloud",
          "contextWindow": 32768,
          "maxTokens": 2048,
          "temperature": 0.7
        },
        {
          "id": "qwen2.5:7b",
          "name": "Qwen 2.5 7B",
          "contextWindow": 32768,
          "maxTokens": 2048
        },
        {
          "id": "mistral:latest",
          "name": "Mistral",
          "contextWindow": 8192,
          "maxTokens": 1024
        },
        {
          "id": "deepseek-coder:6.7b",
          "name": "DeepSeek Coder",
          "contextWindow": 16384,
          "maxTokens": 2048
        }
      ]
    },
    "openai": {
      "baseUrl": "https://api.openai.com/v1",
      "api": "openai",
      "models": [
        {
          "id": "gpt-4",
          "contextWindow": 8192,
          "maxTokens": 4096
        },
        {
          "id": "gpt-3.5-turbo",
          "contextWindow": 16384,
          "maxTokens": 4096
        }
      ]
    },
    "anthropic": {
      "baseUrl": "https://api.anthropic.com/v1",
      "api": "anthropic",
      "models": [
        {
          "id": "claude-3-opus-20240229",
          "contextWindow": 200000,
          "maxTokens": 4096
        }
      ]
    },
    "modelstudio": {
      "baseUrl": "https://api.modelstudio.ai/v1",
      "api": "modelstudio",
      "models": [
        {
          "id": "qwen3.5-plus",
          "contextWindow": 32768,
          "maxTokens": 2048
        }
      ]
    }
  },
  "defaultProvider": "ollama",
  "defaultModel": "qwen3.5:cloud"
}
```

**Emplacement** :
```bash
~/.openclaw/agents/main/agent/models.json
```

**Modifier** :
```bash
nano ~/.openclaw/agents/main/agent/models.json
```

---

### 3. `~/.openclaw/agents/main/agent/auth-profiles.json`

**Rôle** : Profils d'authentification pour chaque provider (clés API, tokens).

**⚠️ FICHIER SENSIBLE** : Contient des credentials, ne **JAMAIS** committer sur Git !

**Structure** :

```json
{
  "profiles": {
    "ollama:default": {
      "type": "api_key",
      "provider": "ollama",
      "key": "ollama-local",
      "endpoint": "http://localhost:11434"
    },
    "openai:production": {
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
    "openai": "openai:production",
    "anthropic": "anthropic:default",
    "modelstudio": "modelstudio:default"
  }
}
```

**Emplacement** :
```bash
~/.openclaw/agents/main/agent/auth-profiles.json
```

**Sécurité** :
```bash
# Permissions restrictives
chmod 600 ~/.openclaw/agents/main/agent/auth-profiles.json

# Ajouter au .gitignore
echo "auth-profiles.json" >> .gitignore
```

---

## Workspace (Bureau de l'agent)

Le dossier `~/.openclaw/workspace/` contient les fichiers qui définissent le **comportement**, la **personnalité** et la **mémoire** de l'agent.

### Fichiers Markdown du workspace

| Fichier | Rôle | Lu automatiquement ? | Priorité |
|---------|------|----------------------|----------|
| `AGENTS.md` | Règles opérationnelles et workflows | ✅ OUI | Haute |
| `SOUL.md` | Personnalité, ton, style de réponse | ✅ OUI | Haute |
| `USER.md` | Contexte sur l'utilisateur (toi) | ✅ OUI | Haute |
| `IDENTITY.md` | Nom, emoji, capabilities de l'agent | ❌ NON (sauf si demandé) | Moyenne |
| `HEARTBEAT.md` | Status en temps réel de l'agent | ✅ OUI (si existe) | Basse |
| `MEMORY.md` | Mémoire long terme persistante | ✅ Parfois | Moyenne |
| `TOOLS.md` | Workflows locaux et conventions | ✅ Parfois | Moyenne |

### Détail de chaque fichier

#### `AGENTS.md` - Règles opérationnelles

**Contenu typique** :
```markdown
# Règles opérationnelles de l'agent

## Comportement général
- Toujours répondre en français
- Être concis et précis
- Citer les sources quand nécessaire

## Workflows spécifiques
### Pour les questions de code
1. Analyser le problème
2. Proposer une solution
3. Expliquer le raisonnement

### Pour la recherche web
1. Formuler une requête claire
2. Synthétiser les résultats
3. Vérifier la fraîcheur des infos
```

#### `SOUL.md` - Personnalité

**Contenu typique** :
```markdown
# Personnalité de l'agent

## Ton
- Professionnel mais accessible
- Pédagogue et patient
- Optimiste et encourageant

## Style
- Phrases courtes et claires
- Exemples concrets
- Pas de jargon inutile

## Ce que je suis
- Un assistant technique compétent
- Un guide pédagogique
- Un partenaire de réflexion

## Ce que je ne suis pas
- Un expert infaillible
- Un simple exécuteur de commandes
- Un remplaçant à la réflexion humaine
```

#### `USER.md` - Contexte utilisateur

**Contenu typique** :
```markdown
# Contexte sur l'utilisateur : Samir

## Profil technique
- Technicien biomédical
- En formation AEC AI (Collège Bois-de-Boulogne)
- Compétences : réseau, IoT, avionique, ML/AI
- Stack : Ubuntu Server, Python, TensorFlow, Ollama

## Préférences
- Langue : Français
- Style : Étape par étape, pas de blocs de code massifs
- Environnement : Conda (pas venv)
- Approche : Builder hands-on, autodidacte

## Projets actuels
- Serveur HP Z800 (Quadro P4000, 16GB RAM)
- Pipeline RAG avec ChromaDB
- Infrastructure locale AI (Ollama + OpenClaw)
- Bot Telegram pour monitoring
```

#### `IDENTITY.md` - Identité de l'agent

**Contenu typique** :
```markdown
# Identité de l'agent

## Nom
Assistant OpenClaw

## Emoji/Avatar
🤖

## Capabilities
- Conversation générale
- Assistance technique
- Recherche web
- Génération de code
- Analyse de documents

## Limites
- Pas d'accès aux fichiers système sensibles
- Pas d'exécution de code arbitraire
- Knowledge cutoff : [date]
```

#### `HEARTBEAT.md` - Status temps réel

**Généré automatiquement**, contient :
```markdown
# Status - 2026-04-02 14:30:00

## État actuel
- Gateway : ✅ Actif
- Modèle : qwen3.5:cloud
- Provider : Ollama (local)
- Latence : 450ms

## Dernières interactions
- 14:29 : Question sur la config OpenClaw
- 14:25 : Changement de modèle
- 14:20 : Redémarrage du gateway

## Ressources
- CPU : 15%
- RAM : 2.3 GB / 16 GB
- Tokens utilisés aujourd'hui : 12,450
```

#### `MEMORY.md` - Mémoire long terme

**Contenu typique** :
```markdown
# Mémoire long terme

## Préférences apprises
- Samir préfère les explications étape par étape
- Ne pas donner de blocs de code complets
- Toujours expliquer le "pourquoi"

## Contexte récurrent
- Projet actuel : RAG avec ChromaDB
- Problème récurrent : Erreurs cuDNN avec TensorFlow
- Configuration : HP Z800, Quadro P4000

## Décisions importantes
- 2026-03-28 : Migration de ModelStudio vers Ollama (local)
- 2026-04-01 : Configuration Telegram bot réussie
```

#### `TOOLS.md` - Workflows locaux

**Contenu typique** :
```markdown
# Workflows et conventions

## Conventions de commit
`type: description`
- fix: corrections
- docs: documentation
- feat: nouvelles fonctionnalités

## Structure de projet
```
projet/
├── README.md
├── docs/
└── src/
```

## Checklist avant déploiement
- [ ] Tests passés
- [ ] Documentation à jour
- [ ] Credentials dans .gitignore
```

### `workspace/memory/` - Logs quotidiens

Fichiers générés automatiquement :
```
workspace/memory/
├── 2026-04-01.md
├── 2026-04-02.md
└── 2026-04-03.md
```

Contenu d'un fichier quotidien :
```markdown
# Log du 2026-04-02

## Conversations
### 09:15 - Configuration Telegram
- Problème : Could not resolve @username
- Solution : Utiliser l'ID numérique

### 14:20 - Changement de modèle
- Ancien : qwen2.5:7b
- Nouveau : qwen3.5:cloud
- Raison : Meilleure performance conversation

## Actions effectuées
- Redémarrage gateway : 3 fois
- Modifications config : 2 fichiers
- Scripts créés : switch-model.sh
```

---

## Hiérarchie et priorités

### Ordre de lecture des configurations

```
1. openclaw.json                      ← PRIORITÉ MAXIMALE
   ↓
2. agents/main/agent/models.json      ← Modèles disponibles
   ↓
3. workspace/AGENTS.md                ← Règles opérationnelles
   ↓
4. workspace/SOUL.md                  ← Personnalité
   ↓
5. workspace/USER.md                  ← Contexte utilisateur
```

### Priorités en cas de conflit

Si un paramètre est défini dans plusieurs fichiers :

```
openclaw.json > agents/.../models.json > workspace/*.md
```

**Exemple** :
- `openclaw.json` définit `model: "ollama/qwen3.5:cloud"`
- `models.json` définit `defaultModel: "mistral:latest"`
- **Résultat** : OpenClaw utilise `qwen3.5:cloud` (openclaw.json gagne)

---

## Fichiers générés automatiquement

### `~/.openclaw/state/`

Fichiers d'état et de cache, **ne pas modifier manuellement** :

```
state/
├── sessions/          ← Sessions actives
├── cache/             ← Cache des réponses
└── locks/             ← Verrous de processus
```

### `~/.openclaw/logs/`

Logs système :

```
logs/
├── gateway.log        ← Logs du gateway
├── gateway.error.log  ← Erreurs du gateway
└── agents/
    └── main.log       ← Logs de l'agent "main"
```

**Consulter les logs** :
```bash
# Logs en temps réel
tail -f ~/.openclaw/logs/gateway.log

# Erreurs uniquement
tail -f ~/.openclaw/logs/gateway.error.log

# Logs via OpenClaw CLI
openclaw logs --follow
openclaw logs --tail 50
```

### `~/.openclaw/credentials/`

**⚠️ TRÈS SENSIBLE** : Tokens et credentials

```
credentials/
└── telegram/
    └── default/
        └── session.json    ← Token du bot Telegram
```

**Sécurité** :
```bash
# Permissions restrictives
chmod 700 ~/.openclaw/credentials
chmod 600 ~/.openclaw/credentials/telegram/default/session.json

# .gitignore
echo "credentials/" >> .gitignore
```

---

## Commandes pour explorer la structure

### Voir l'arborescence complète

```bash
tree -L 4 ~/.openclaw/
```

### Trouver tous les fichiers JSON

```bash
find ~/.openclaw -name "*.json" -type f
```

### Trouver tous les fichiers Markdown du workspace

```bash
find ~/.openclaw/workspace -name "*.md" -type f
```

### Taille de chaque dossier

```bash
du -sh ~/.openclaw/*
```

### Vérifier les permissions

```bash
ls -la ~/.openclaw/
ls -la ~/.openclaw/agents/main/agent/
ls -la ~/.openclaw/credentials/
```

### Backup de toute la structure

```bash
tar -czf ~/openclaw-backup-$(date +%Y%m%d).tar.gz ~/.openclaw/
```

---

## Résumé visuel

```
CONFIGURATION
├── openclaw.json ────────────► PRIORITÉ #1 (Modèle, Gateway, Canaux)
├── models.json ──────────────► Modèles disponibles par provider
└── auth-profiles.json ───────► Credentials (SENSIBLE)

COMPORTEMENT
├── AGENTS.md ────────────────► Règles et workflows
├── SOUL.md ──────────────────► Personnalité et ton
└── USER.md ──────────────────► Contexte utilisateur

IDENTITÉ
├── IDENTITY.md ──────────────► Nom, emoji, capabilities
└── HEARTBEAT.md ─────────────► Status temps réel

MÉMOIRE
├── MEMORY.md ────────────────► Mémoire long terme
├── TOOLS.md ─────────────────► Conventions et workflows
└── memory/*.md ──────────────► Logs quotidiens

SYSTÈME (auto-généré)
├── state/ ───────────────────► Cache et sessions
├── logs/ ────────────────────► Logs système
└── credentials/ ─────────────► Tokens (TRÈS SENSIBLE)
```

---

[← Retour au README](../README.md) | [Configuration détaillée →](CONFIGURATION.md)
