# Scripts utiles

Collection de scripts shell pour faciliter l'utilisation d'OpenClaw avec Ollama.

## Table des matières

- [Script de basculement de modèle](#script-de-basculement-de-modèle)
- [Script de démarrage automatique](#script-de-démarrage-automatique)
- [Script de diagnostic](#script-de-diagnostic)
- [Script de backup](#script-de-backup)

---

## Script de basculement de modèle

### Vue d'ensemble

Ce script permet de changer facilement le modèle Ollama utilisé par OpenClaw en une seule commande.

**Modèles disponibles sur ce système :**

```
NAME                       ID              SIZE      MODIFIED
qwen3.5:cloud              a7bf6f7891c3    -         30 hours ago
qwen2.5:7b                 845dbda0ea48    4.7 GB    4 days ago
nomic-embed-text:v1.5      0a109f422b47    274 MB    2 months ago
nomic-embed-text:latest    0a109f422b47    274 MB    2 months ago
deepseek-coder:6.7b        ce298d984115    3.8 GB    2 months ago
mistral:latest             6577803aa9a0    4.4 GB    2 months ago
```

### Installation

```bash
# Créer le script
cat > ~/switch-model.sh << 'EOF'
#!/bin/bash

# ════════════════════════════════════════════════════════════
# Script de changement de modèle OpenClaw
# Auteur: Samir
# Usage: ./switch-model.sh <nom-du-modele>
# ════════════════════════════════════════════════════════════

# Couleurs
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'

# Vérifier les arguments
if [ -z "$1" ]; then
  echo -e "${RED}❌ Usage: switch-model.sh <nom-du-modele>${NC}"
  echo ""
  echo -e "${BLUE}Modèles disponibles :${NC}"
  ollama list
  echo ""
  echo -e "${YELLOW}Exemple:${NC} switch-model.sh qwen3.5:cloud"
  exit 1
fi

MODEL=$1

echo -e "${BLUE}═══════════════════════════════════════════${NC}"
echo -e "${BLUE}   OpenClaw - Changement de modèle${NC}"
echo -e "${BLUE}═══════════════════════════════════════════${NC}"
echo ""
echo -e "${YELLOW}🔄 Changement vers le modèle : $MODEL${NC}"

# Vérifier que le modèle existe
if ! ollama list | grep -q "^$MODEL "; then
  echo -e "${RED}❌ Erreur: Le modèle '$MODEL' n'est pas installé${NC}"
  echo ""
  echo -e "${YELLOW}Pour l'installer :${NC}"
  echo "  ollama pull $MODEL"
  exit 1
fi

# Vérifier que les fichiers de config existent
if [ ! -f ~/.openclaw/openclaw.json ]; then
  echo -e "${RED}❌ Erreur: ~/.openclaw/openclaw.json non trouvé${NC}"
  exit 1
fi

# Backup de la config
BACKUP_FILE=~/.openclaw/openclaw.json.backup.$(date +%Y%m%d_%H%M%S)
cp ~/.openclaw/openclaw.json "$BACKUP_FILE"
echo -e "${GREEN}💾 Backup créé : $BACKUP_FILE${NC}"

# Afficher le modèle actuel
CURRENT=$(grep -oP '"primary": "ollama/\K[^"]+' ~/.openclaw/openclaw.json 2>/dev/null || echo "non défini")
echo -e "${YELLOW}📌 Modèle actuel : $CURRENT${NC}"

# Changer la config globale
sed -i "s|\"primary\": \"ollama/.*\"|\"primary\": \"ollama/$MODEL\"|" ~/.openclaw/openclaw.json

# Changer tous les "model" dans la config globale
sed -i "s|\"model\": \"ollama/.*\"|\"model\": \"ollama/$MODEL\"|" ~/.openclaw/openclaw.json

# Changer la config de l'agent
if [ -f ~/.openclaw/agents/main/agent/models.json ]; then
  sed -i "s|\"defaultModel\": \".*\"|\"defaultModel\": \"$MODEL\"|" ~/.openclaw/agents/main/agent/models.json
  echo -e "${GREEN}✅ Configuration de l'agent mise à jour${NC}"
fi

# Redémarrer le gateway
echo -e "${YELLOW}🔄 Redémarrage du gateway...${NC}"
if openclaw gateway restart > /dev/null 2>&1; then
  echo -e "${GREEN}✅ Gateway redémarré avec succès${NC}"
else
  echo -e "${RED}❌ Erreur lors du redémarrage du gateway${NC}"
  echo -e "${YELLOW}Restauration du backup...${NC}"
  cp "$BACKUP_FILE" ~/.openclaw/openclaw.json
  exit 1
fi

echo ""
echo -e "${GREEN}═══════════════════════════════════════════${NC}"
echo -e "${GREEN}✅ Modèle changé vers : ollama/$MODEL${NC}"
echo -e "${GREEN}═══════════════════════════════════════════${NC}"
echo ""
echo -e "${BLUE}Vérification :${NC}"
grep "primary" ~/.openclaw/openclaw.json

EOF

# Rendre exécutable
chmod +x ~/switch-model.sh

echo "✅ Script créé : ~/switch-model.sh"
```

### Utilisation

#### Afficher l'aide

```bash
~/switch-model.sh
```

Sortie :
```
❌ Usage: switch-model.sh <nom-du-modele>

Modèles disponibles :
NAME                       ID              SIZE      MODIFIED
qwen3.5:cloud              a7bf6f7891c3    -         30 hours ago
qwen2.5:7b                 845dbda0ea48    4.7 GB    4 days ago
mistral:latest             6577803aa9a0    4.4 GB    2 months ago

Exemple: switch-model.sh qwen3.5:cloud
```

#### Changer de modèle

```bash
# Basculer vers qwen3.5:cloud
~/switch-model.sh qwen3.5:cloud

# Basculer vers mistral
~/switch-model.sh mistral:latest

# Basculer vers deepseek pour le code
~/switch-model.sh deepseek-coder:6.7b

# Basculer vers qwen2.5
~/switch-model.sh qwen2.5:7b
```

#### Sortie exemple

```
═══════════════════════════════════════════
   OpenClaw - Changement de modèle
═══════════════════════════════════════════

🔄 Changement vers le modèle : qwen3.5:cloud
💾 Backup créé : ~/.openclaw/openclaw.json.backup.20260402_143022
📌 Modèle actuel : qwen2.5:7b
✅ Configuration de l'agent mise à jour
🔄 Redémarrage du gateway...
✅ Gateway redémarré avec succès

═══════════════════════════════════════════
✅ Modèle changé vers : ollama/qwen3.5:cloud
═══════════════════════════════════════════

Vérification :
  "primary": "ollama/qwen3.5:cloud",
```

### Cas d'usage

#### Pour la conversation générale
```bash
~/switch-model.sh qwen3.5:cloud
```

#### Pour le développement de code
```bash
~/switch-model.sh deepseek-coder:6.7b
```

#### Pour le raisonnement
```bash
~/switch-model.sh mistral:latest
```

#### Pour les embeddings
```bash
~/switch-model.sh nomic-embed-text:latest
```

### Restaurer un backup

Si le changement pose problème :

```bash
# Voir les backups disponibles
ls -lh ~/.openclaw/openclaw.json.backup.*

# Restaurer le dernier backup
LAST_BACKUP=$(ls -t ~/.openclaw/openclaw.json.backup.* | head -1)
cp "$LAST_BACKUP" ~/.openclaw/openclaw.json
openclaw gateway restart

echo "✅ Configuration restaurée depuis : $LAST_BACKUP"
```

---

## Script de démarrage automatique

Script pour démarrer Ollama et OpenClaw automatiquement.

```bash
cat > ~/start-openclaw.sh << 'EOF'
#!/bin/bash

echo "🚀 Démarrage d'OpenClaw..."

# Démarrer Ollama si nécessaire
if ! curl -s http://localhost:11434/api/tags > /dev/null 2>&1; then
    echo "⏳ Démarrage d'Ollama..."
    ollama serve > /dev/null 2>&1 &
    sleep 3
    echo "✅ Ollama démarré"
else
    echo "✅ Ollama déjà actif"
fi

# Démarrer OpenClaw Gateway
echo "⏳ Démarrage du Gateway OpenClaw..."
openclaw gateway restart

echo "✅ OpenClaw prêt !"
echo ""
echo "Lancer l'interface avec : openclaw tui"

EOF

chmod +x ~/start-openclaw.sh
```

Utilisation :
```bash
~/start-openclaw.sh
```

---

## Script de diagnostic

Script complet pour diagnostiquer les problèmes.

```bash
cat > ~/diagnose-openclaw.sh << 'EOF'
#!/bin/bash

echo "═══════════════════════════════════════════"
echo "   DIAGNOSTIC OPENCLAW + OLLAMA"
echo "═══════════════════════════════════════════"

# 1. Vérification Ollama
echo -e "\n[1/5] Vérification Ollama..."
if curl -s http://localhost:11434/api/tags > /dev/null 2>&1; then
    echo "✅ Serveur Ollama actif"
    echo "Modèles installés :"
    ollama list
else
    echo "❌ Serveur Ollama inactif"
    echo "Pour démarrer : ollama serve &"
fi

# 2. Vérification OpenClaw
echo -e "\n[2/5] Vérification OpenClaw..."
if command -v openclaw &> /dev/null; then
    echo "✅ OpenClaw installé ($(openclaw --version))"
    openclaw gateway status 2>&1 || echo "⚠️  Gateway non actif"
else
    echo "❌ OpenClaw non installé"
fi

# 3. Configuration
echo -e "\n[3/5] Configuration..."
if [ -f ~/.openclaw/openclaw.json ]; then
    echo "✅ Fichier de config présent"
    CURRENT_MODEL=$(grep -oP '"primary": "ollama/\K[^"]+' ~/.openclaw/openclaw.json 2>/dev/null)
    echo "Modèle actuel : $CURRENT_MODEL"
else
    echo "❌ Fichier de config manquant"
fi

# 4. Processus actifs
echo -e "\n[4/5] Processus actifs..."
OLLAMA_PID=$(pgrep ollama)
OPENCLAW_PID=$(pgrep openclaw)

if [ -n "$OLLAMA_PID" ]; then
    echo "✅ Ollama actif (PID: $OLLAMA_PID)"
else
    echo "❌ Ollama non actif"
fi

if [ -n "$OPENCLAW_PID" ]; then
    echo "✅ OpenClaw actif (PID: $OPENCLAW_PID)"
else
    echo "❌ OpenClaw non actif"
fi

# 5. Ports
echo -e "\n[5/5] Ports réseau..."
if sudo lsof -i :11434 &> /dev/null; then
    echo "✅ Port 11434 (Ollama) : utilisé"
else
    echo "⚠️  Port 11434 (Ollama) : libre"
fi

if sudo lsof -i :18789 &> /dev/null; then
    echo "✅ Port 18789 (Gateway) : utilisé"
else
    echo "⚠️  Port 18789 (Gateway) : libre"
fi

echo -e "\n═══════════════════════════════════════════"
echo "   FIN DU DIAGNOSTIC"
echo "═══════════════════════════════════════════"

EOF

chmod +x ~/diagnose-openclaw.sh
```

---

## Script de backup

Sauvegarder toute la configuration OpenClaw.

```bash
cat > ~/backup-openclaw.sh << 'EOF'
#!/bin/bash

BACKUP_DIR=~/openclaw-backups
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="$BACKUP_DIR/openclaw-backup-$TIMESTAMP.tar.gz"

# Créer le répertoire de backup
mkdir -p "$BACKUP_DIR"

echo "💾 Création du backup OpenClaw..."

# Créer l'archive
tar -czf "$BACKUP_FILE" \
    ~/.openclaw/openclaw.json \
    ~/.openclaw/agents/main/agent/models.json \
    ~/.openclaw/agents/main/agent/auth-profiles.json \
    2>/dev/null

if [ -f "$BACKUP_FILE" ]; then
    SIZE=$(du -h "$BACKUP_FILE" | cut -f1)
    echo "✅ Backup créé : $BACKUP_FILE ($SIZE)"
    
    # Lister tous les backups
    echo ""
    echo "Backups disponibles :"
    ls -lh "$BACKUP_DIR"
else
    echo "❌ Erreur lors de la création du backup"
    exit 1
fi

EOF

chmod +x ~/backup-openclaw.sh
```

Restaurer un backup :
```bash
# Lister les backups
ls -lh ~/openclaw-backups/

# Restaurer
tar -xzf ~/openclaw-backups/openclaw-backup-20260402_143022.tar.gz -C ~/
openclaw gateway restart
```

---

## Aliases pratiques

Ajouter dans `~/.bashrc` ou `~/.zshrc` :

```bash
# ═══════════════════════════════════════════
# Aliases OpenClaw + Ollama
# ═══════════════════════════════════════════

# Changement rapide de modèle
alias switch-qwen='~/switch-model.sh qwen3.5:cloud'
alias switch-mistral='~/switch-model.sh mistral:latest'
alias switch-deepseek='~/switch-model.sh deepseek-coder:6.7b'
alias switch-qwen25='~/switch-model.sh qwen2.5:7b'

# Commandes fréquentes
alias oc='openclaw tui'
alias ocr='openclaw gateway restart'
alias ocl='openclaw logs --follow'
alias ocd='openclaw doctor'
alias current-model='grep "primary" ~/.openclaw/openclaw.json'

# Ollama
alias ol='ollama list'
alias orun='ollama run'

# Scripts utiles
alias start-oc='~/start-openclaw.sh'
alias diag-oc='~/diagnose-openclaw.sh'
alias backup-oc='~/backup-openclaw.sh'
```

Activer :
```bash
source ~/.bashrc
```

Utilisation :
```bash
switch-qwen       # Basculer vers qwen3.5:cloud
switch-mistral    # Basculer vers mistral
current-model     # Voir le modèle actuel
oc               # Lancer OpenClaw TUI
ocr              # Redémarrer le gateway
diag-oc          # Diagnostic complet
```

---

[← Retour au README](../README.md) | [Référence des commandes →](COMMANDS.md)
