# Architecture d'OpenClaw

Documentation technique de l'architecture et du flux de données d'OpenClaw.

## Table des matières

- [Vue d'ensemble](#vue-densemble)
- [Architecture en couches](#architecture-en-couches)
- [Flux de communication](#flux-de-communication)
- [Composants détaillés](#composants-détaillés)
- [Séquences d'exécution](#séquences-dexécution)

---

## Vue d'ensemble

OpenClaw est un framework d'orchestration d'agents IA qui connecte des interfaces utilisateur (Telegram, Terminal, Discord) à des modèles d'IA (Ollama, OpenAI, Anthropic) via un gateway central.

### Schéma général

```
┌─────────────────────────────────────────────────────────────┐
│                    COUCHE UTILISATEUR                       │
│                                                             │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐            │
│  │   TUI    │    │ Telegram │    │ Discord  │    ...     │
│  │ Terminal │    │   Bot    │    │   Bot    │            │
│  └────┬─────┘    └────┬─────┘    └────┬─────┘            │
│       │               │               │                    │
└───────┼───────────────┼───────────────┼────────────────────┘
        │               │               │
        └───────────────┴───────────────┘
                        │
┌───────────────────────▼────────────────────────────────────┐
│                   COUCHE GATEWAY                           │
│                                                            │
│  ┌──────────────────────────────────────────────────┐    │
│  │           WebSocket Server                       │    │
│  │         ws://127.0.0.1:18789                     │    │
│  ├──────────────────────────────────────────────────┤    │
│  │  • Gestion des sessions                          │    │
│  │  • Mémoire et contexte                           │    │
│  │  • Routage des messages                          │    │
│  │  • Load balancing                                │    │
│  └────────────────────┬─────────────────────────────┘    │
│                       │                                    │
└───────────────────────┼────────────────────────────────────┘
                        │
        ┌───────────────┴───────────────┐
        │                               │
┌───────▼──────────┐          ┌─────────▼────────┐
│  AGENT "main"    │          │  Autres agents   │
│                  │          │                  │
│ ┌──────────────┐ │          │                  │
│ │ Models       │ │          │                  │
│ │ Auth         │ │          │                  │
│ │ Routing      │ │          │                  │
│ └──────┬───────┘ │          │                  │
└────────┼─────────┘          └──────────────────┘
         │
         │
┌────────▼──────────────────────────────────────────────────┐
│                   COUCHE PROVIDER                         │
│                                                           │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐            │
│  │  Ollama  │   │  OpenAI  │   │ Anthropic│    ...     │
│  │  Local   │   │   API    │   │   API    │            │
│  └──────────┘   └──────────┘   └──────────┘            │
│                                                           │
│  Modèles disponibles :                                   │
│  • qwen3.5:cloud                                         │
│  • llama3.2                                              │
│  • gpt-4                                                 │
│  • claude-3-opus                                         │
└───────────────────────────────────────────────────────────┘
```

---

## Architecture en couches

### Couche 1 : Utilisateur (Interface)

**Responsabilité** : Réception des entrées utilisateur et affichage des réponses

**Composants** :
- **TUI (Terminal User Interface)** : Interface texte dans le terminal
- **Bot Telegram** : Interface via messagerie Telegram
- **Bot Discord** : Interface via Discord
- **Bot Slack** : Interface via Slack
- **API REST** : Interface pour applications tierces

**Communication** : WebSocket vers le Gateway

```javascript
// Exemple de connexion TUI → Gateway
const ws = new WebSocket('ws://127.0.0.1:18789');
ws.send(JSON.stringify({
  type: 'message',
  content: 'Hello, bot!',
  session_id: 'user-session-123'
}));
```

---

### Couche 2 : Gateway (Orchestration)

**Responsabilité** : Routage intelligent, gestion de sessions, mémoire

**Composants** :
- **WebSocket Server** : Point d'entrée unique
- **Session Manager** : Gestion des sessions utilisateur
- **Memory Store** : Contexte et historique
- **Message Router** : Aiguillage vers les agents
- **Load Balancer** : Distribution de charge

**Architecture interne** :

```
Gateway
├── WebSocket Server (port 18789)
├── Session Manager
│   ├── Session Store (Redis/Memory)
│   └── TTL Management
├── Message Router
│   ├── Pattern Matching
│   └── Agent Selection
└── Memory Manager
    ├── Short-term (session)
    └── Long-term (persistent)
```

**Flux de traitement** :

```
1. Message entrant → WebSocket
2. Session lookup/create
3. Context retrieval
4. Route to agent
5. Wait for response
6. Update context
7. Send back to user
```

---

### Couche 3 : Agent (Intelligence)

**Responsabilité** : Traitement des requêtes, sélection du modèle

**Composants d'un agent** :
- **Models Config** : Liste des modèles disponibles
- **Auth Profiles** : Credentials pour les providers
- **Routing Rules** : Règles de sélection du modèle
- **Prompt Templates** : Templates de prompts système

**Exemple d'agent "main"** :

```
agents/main/
├── agent/
│   ├── models.json          # Configuration des modèles
│   ├── auth-profiles.json   # Authentification
│   ├── routing.json         # Règles de routage
│   └── prompts/
│       ├── system.txt       # Prompt système
│       └── templates/       # Templates de prompts
├── memory/
│   └── conversations/       # Historique
└── logs/
    └── agent.log            # Logs spécifiques
```

**Processus de décision** :

```
Agent reçoit message
↓
Analyse du contenu
↓
Matching routing rules
↓
Sélection du provider/modèle
↓
Récupération auth profile
↓
Appel au provider
↓
Traitement réponse
↓
Retour au Gateway
```

---

### Couche 4 : Provider (Exécution)

**Responsabilité** : Génération des réponses IA

**Providers disponibles** :

#### Ollama (Local)
- **Endpoint** : `http://localhost:11434`
- **Auth** : Aucune (local)
- **Modèles** : qwen3.5:cloud, llama3.2, mistral, etc.
- **Avantages** : Gratuit, privé, rapide

#### OpenAI (Cloud)
- **Endpoint** : `https://api.openai.com/v1`
- **Auth** : Clé API
- **Modèles** : GPT-4, GPT-3.5-turbo
- **Avantages** : Performant, vision, fonctions

#### Anthropic (Cloud)
- **Endpoint** : `https://api.anthropic.com/v1`
- **Auth** : Clé API
- **Modèles** : Claude 3 Opus, Sonnet, Haiku
- **Avantages** : Contexte long, raisonnement

#### ModelStudio (Cloud)
- **Endpoint** : `https://api.modelstudio.ai/v1`
- **Auth** : Clé API
- **Modèles** : qwen3.5-plus
- **Avantages** : Modèles chinois optimisés

---

## Flux de communication

### Flux complet d'un message

```mermaid
sequenceDiagram
    participant U as Utilisateur
    participant T as TUI/Telegram
    participant G as Gateway
    participant A as Agent
    participant P as Provider (Ollama)

    U->>T: Tape "Hello!"
    T->>G: WebSocket: {message: "Hello!", session: "123"}
    G->>G: Récupère session
    G->>G: Récupère contexte
    G->>A: Route vers agent "main"
    A->>A: Analyse message
    A->>A: Sélectionne modèle
    A->>A: Récupère auth
    A->>P: POST /api/generate
    P->>P: Génère réponse
    P-->>A: {response: "Hi! How can I help?"}
    A-->>G: Retourne réponse
    G->>G: Met à jour contexte
    G-->>T: WebSocket: {response: "Hi! How..."}
    T-->>U: Affiche réponse
```

### Détails du protocole WebSocket

**Message utilisateur → Gateway** :

```json
{
  "type": "message",
  "content": "Explain quantum physics",
  "session_id": "user-123",
  "metadata": {
    "timestamp": 1680000000,
    "channel": "telegram",
    "user_id": "987654321"
  }
}
```

**Réponse Gateway → Utilisateur** :

```json
{
  "type": "response",
  "content": "Quantum physics is...",
  "session_id": "user-123",
  "metadata": {
    "model": "qwen3.5:cloud",
    "provider": "ollama",
    "tokens": {
      "prompt": 15,
      "completion": 150
    },
    "latency_ms": 1234
  }
}
```

---

## Composants détaillés

### Session Manager

**Responsabilité** : Gestion des sessions utilisateur

**Structure de session** :

```json
{
  "session_id": "user-123",
  "created_at": 1680000000,
  "last_activity": 1680001000,
  "ttl": 3600,
  "user": {
    "id": "987654321",
    "channel": "telegram",
    "name": "Samir"
  },
  "context": {
    "messages": [
      {"role": "user", "content": "Hello"},
      {"role": "assistant", "content": "Hi!"}
    ],
    "metadata": {
      "conversation_id": "conv-456",
      "topic": "general"
    }
  }
}
```

**Opérations** :
- `create_session()` : Nouvelle session
- `get_session()` : Récupérer session
- `update_session()` : Mettre à jour
- `delete_session()` : Supprimer
- `cleanup_expired()` : Nettoyage auto

---

### Memory Manager

**Types de mémoire** :

#### Mémoire à court terme (Session)
- Stockage : RAM
- Durée : Session active
- Contenu : Conversation en cours

#### Mémoire à long terme (Persistent)
- Stockage : Disque/DB
- Durée : Illimitée
- Contenu : Historique, préférences

**Structure** :

```
Memory Store
├── Short-term (Redis/Memory)
│   └── sessions/
│       ├── user-123
│       ├── user-456
│       └── ...
└── Long-term (SQLite/PostgreSQL)
    └── conversations/
        ├── conv-abc
        ├── conv-def
        └── ...
```

---

### Message Router

**Règles de routage** :

```json
{
  "rules": [
    {
      "id": "code-requests",
      "pattern": ".*code.*|.*program.*|.*debug.*",
      "agent": "main",
      "provider": "ollama",
      "model": "codellama",
      "priority": 10
    },
    {
      "id": "general-chat",
      "pattern": ".*",
      "agent": "main",
      "provider": "ollama",
      "model": "qwen3.5:cloud",
      "priority": 1
    }
  ]
}
```

**Algorithme de sélection** :

```python
def route_message(message, rules):
    matching_rules = []
    
    for rule in rules:
        if regex_match(rule.pattern, message):
            matching_rules.append(rule)
    
    # Trier par priorité
    matching_rules.sort(key=lambda r: r.priority, reverse=True)
    
    # Retourner la règle avec la plus haute priorité
    return matching_rules[0] if matching_rules else default_rule
```

---

## Séquences d'exécution

### Démarrage du système

```
1. Démarrer Ollama
   ollama serve &

2. Charger modèles en mémoire
   ollama list

3. Démarrer Gateway
   openclaw gateway start
   └── Charger configuration
   └── Initialiser WebSocket
   └── Charger agents
   └── Connecter providers

4. Lancer interface
   openclaw tui
   └── Établir connexion WebSocket
   └── Créer session
   └── Prêt à recevoir messages
```

### Traitement d'un message

```
[Interface] Message utilisateur
    ↓
[Gateway] Réception WebSocket
    ↓
[Session] Lookup ou création session
    ↓
[Memory] Récupération contexte
    ↓
[Router] Sélection agent/modèle
    ↓
[Agent] Préparation prompt
    ↓
[Auth] Récupération credentials
    ↓
[Provider] Génération réponse
    ↓
[Agent] Post-traitement
    ↓
[Memory] Mise à jour contexte
    ↓
[Gateway] Envoi réponse WebSocket
    ↓
[Interface] Affichage à l'utilisateur
```

### Gestion d'erreur

```
[Erreur détectée]
    ↓
[Retry Logic]
    ├─ Provider timeout → Retry 3x
    ├─ Auth failure → Fallback provider
    ├─ Model not found → Fallback model
    └─ Network error → Queue message
    ↓
[Logging]
    ↓
[User Notification]
```

---

## Performance et scalabilité

### Optimisations actuelles

- **Connection pooling** : Réutilisation connexions HTTP
- **Session caching** : Sessions en RAM
- **Lazy loading** : Chargement modèles à la demande
- **Streaming** : Réponses en temps réel

### Métriques de performance

```
Latence typique :
├── WebSocket round-trip : ~5ms
├── Session lookup : ~1ms
├── Routing : ~2ms
├── Ollama local : ~500-2000ms
└── Total : ~500-2010ms
```

### Scalabilité

**Limites actuelles** :
- Gateway : Single instance
- Sessions : Memory-based
- Concurrent users : ~100

**Améliorations futures** :
- Gateway clustering
- Redis pour sessions
- Load balancer externe

---

[← Retour au README](../README.md)
