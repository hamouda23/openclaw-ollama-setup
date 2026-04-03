# Référence des commandes

Guide complet de toutes les commandes utiles pour OpenClaw, Ollama et le dépannage système.

## Table des matières

- [Commandes Ollama](#commandes-ollama)
- [Commandes OpenClaw](#commandes-openclaw)
- [Configuration](#configuration)
- [Dépannage](#dépannage)
- [Logs et monitoring](#logs-et-monitoring)

---

## Commandes Ollama

### Gestion du serveur

```bash
# Démarrer le serveur Ollama
ollama serve

# Démarrer en arrière-plan
ollama serve &

# Arrêter le serveur
pkill -f ollama
```

### Gestion des modèles

```bash
# Lister les modèles installés
ollama list

# Télécharger un modèle
ollama pull <nom_du_modele>

# Exemples
ollama pull qwen3.5:cloud
ollama pull llama3.2
ollama pull mistral

# Voir les modèles en cours d'exécution
ollama ps

# Supprimer un modèle
ollama rm <nom_du_modele>

# Afficher les informations d'un modèle
ollama show <nom_du_modele>
```

### Utilisation interactive

```bash
# Lancer un modèle en mode interactif
ollama run <nom_du_modele>

# Exemple
ollama run qwen3.5:cloud

# Envoyer une requête unique
ollama run <nom_du_modele> "Votre question ici"

# Exemple
ollama run qwen3.5:cloud "Explique-moi l'IA en 3 phrases"
```

### API et tests

```bash
# Tester la connexion à l'API
curl http://localhost:11434/api/tags

# Générer une réponse via API
curl http://localhost:11434/api/generate -d '{
  "model": "qwen3.5:cloud",
  "prompt": "Why is the sky blue?",
  "stream": false
}'

# Obtenir les embeddings
curl http://localhost:11434/api/embeddings -d '{
  "model": "qwen3.5:cloud",
  "prompt": "Here is an article about llamas..."
}'
```

### Informations système

```bash
# Version d'Ollama
ollama --version

# Aide générale
ollama --help

# Aide sur une sous-commande
ollama pull --help
```

---

## Commandes OpenClaw

### Commandes principales

```bash
# Version
openclaw --version

# Aide générale
openclaw --help

# Aide sur une sous-commande
openclaw tui --help
```

### Interface utilisateur

```bash
# Lancer l'interface TUI (Terminal User Interface)
openclaw tui

# Lancer en mode debug
openclaw tui --debug

# Quitter l'interface TUI
# Appuyer sur Ctrl+C ou Ctrl+D
```

### Gestion du Gateway

```bash
# Démarrer le gateway
openclaw gateway start

# Arrêter le gateway
openclaw gateway stop

# Redémarrer le gateway
openclaw gateway restart

# Vérifier le statut
openclaw gateway status

# Statut du service (si installé comme service)
systemctl status openclaw-gateway.service
```

### Logs

```bash
# Voir les logs en temps réel
openclaw logs --follow

# Afficher les 50 dernières lignes
openclaw logs --tail 50

# Afficher tous les logs
openclaw logs

# Logs avec horodatage
openclaw logs --timestamps

# Filtrer par niveau
openclaw logs --level error
openclaw logs --level info
openclaw logs --level debug
```

### Diagnostic

```bash
# Lancer le diagnostic automatique
openclaw doctor

# Vérifier la configuration
openclaw config validate

# Afficher la configuration actuelle
openclaw config show

# Lister les agents disponibles
openclaw agents list

# Informations sur un agent spécifique
openclaw agents show main
```

### Gestion des agents

```bash
# Créer un nouvel agent
openclaw agents create <nom>

# Supprimer un agent
openclaw agents delete <nom>

# Exporter un agent
openclaw agents export <nom> -o agent.tar.gz

# Importer un agent
openclaw agents import agent.tar.gz
```

---

## Configuration

### Fichiers de configuration

```bash
# Afficher la configuration globale
cat ~/.openclaw/openclaw.json

# Afficher la configuration de l'agent main
cat ~/.openclaw/agents/main/agent/models.json

# Afficher les profils d'authentification
cat ~/.openclaw/agents/main/agent/auth-profiles.json

# Lister tous les fichiers de configuration
find ~/.openclaw -name "*.json" -type f
```

### Modification rapide

```bash
# Éditer la configuration principale
nano ~/.openclaw/openclaw.json

# Éditer les modèles de l'agent
nano ~/.openclaw/agents/main/agent/models.json

# Éditer l'authentification
nano ~/.openclaw/agents/main/agent/auth-profiles.json
```

### Modification via sed (automatique)

```bash
# Changer le modèle primaire
sed -i 's|"primary": ".*"|"primary": "ollama/qwen3.5:cloud"|' ~/.openclaw/openclaw.json

# Activer Telegram
sed -i 's|"enabled": false|"enabled": true|' ~/.openclaw/openclaw.json

# Backup avant modification
cp ~/.openclaw/openclaw.json ~/.openclaw/openclaw.json.backup
```

### Validation

```bash
# Valider la syntaxe JSON
cat ~/.openclaw/openclaw.json | jq .

# Si jq n'est pas installé
sudo apt install jq  # Ubuntu/Debian
brew install jq      # macOS
```

---

## Dépannage

### Vérification des processus

```bash
# Lister tous les processus Ollama
ps aux | grep ollama

# Lister tous les processus OpenClaw
ps aux | grep openclaw

# Trouver le PID d'un processus
pgrep ollama
pgrep openclaw

# Tuer un processus par PID
kill <PID>

# Tuer tous les processus d'un nom
pkill -f ollama
pkill -f openclaw

# Forcer l'arrêt
pkill -9 -f ollama
```

### Vérification des ports

```bash
# Vérifier si le port 11434 (Ollama) est utilisé
sudo lsof -i :11434

# Vérifier si le port 18789 (OpenClaw Gateway) est utilisé
sudo lsof -i :18789

# Voir tous les ports en écoute
sudo netstat -tulpn

# Alternative avec ss
sudo ss -tulpn
```

### Nettoyage

```bash
# Nettoyer les modèles Ollama non utilisés
ollama rm <nom_du_modele>

# Vider le cache OpenClaw
rm -rf ~/.openclaw/cache/*

# Réinitialiser complètement OpenClaw (ATTENTION : perte de données)
rm -rf ~/.openclaw
openclaw tui  # Recréera les fichiers par défaut
```

### Redémarrage complet

```bash
# Script de redémarrage complet
#!/bin/bash

echo "Arrêt des services..."
pkill -f ollama
pkill -f openclaw

echo "Attente..."
sleep 3

echo "Démarrage d'Ollama..."
ollama serve &
sleep 2

echo "Démarrage d'OpenClaw..."
openclaw gateway restart
sleep 2

echo "Vérification..."
curl http://localhost:11434/api/tags
openclaw gateway status

echo "Terminé !"
```

### Tests de connectivité

```bash
# Tester Ollama
curl http://localhost:11434/api/tags

# Tester avec un modèle
curl http://localhost:11434/api/generate -d '{
  "model": "qwen3.5:cloud",
  "prompt": "Test",
  "stream": false
}'

# Tester OpenClaw Gateway (WebSocket)
# Nécessite websocat ou wscat
websocat ws://127.0.0.1:18789
```

---

## Logs et monitoring

### Emplacement des logs

```bash
# Logs OpenClaw
~/.openclaw/logs/

# Logs Ollama (selon l'installation)
journalctl -u ollama  # Si installé comme service
~/.ollama/logs/       # Logs locaux
```

### Surveillance en temps réel

```bash
# Surveiller les logs OpenClaw
tail -f ~/.openclaw/logs/gateway.log

# Surveiller plusieurs fichiers
tail -f ~/.openclaw/logs/*.log

# Avec grep pour filtrer
tail -f ~/.openclaw/logs/gateway.log | grep ERROR
```

### Analyse des logs

```bash
# Compter les erreurs
grep -c ERROR ~/.openclaw/logs/gateway.log

# Afficher uniquement les erreurs
grep ERROR ~/.openclaw/logs/gateway.log

# Erreurs des dernières 24h
find ~/.openclaw/logs -type f -mtime -1 -exec grep ERROR {} +

# Statistiques
grep -E "(ERROR|WARN|INFO)" ~/.openclaw/logs/gateway.log | cut -d' ' -f2 | sort | uniq -c
```

### Monitoring système

```bash
# Utilisation CPU/RAM par Ollama
top -p $(pgrep ollama)

# Alternative avec htop
htop -p $(pgrep ollama)

# Utilisation mémoire
ps aux | grep ollama | awk '{print $6/1024 " MB"}'

# Espace disque utilisé par les modèles
du -sh ~/.ollama/models/
```

---

## Changer de modèle rapidement

### Script de basculement de modèle

Pour faciliter le changement de modèle OpenClaw, voici un script automatisé.

#### Modèles disponibles

```bash
ollama list
```

Sortie exemple :
```
NAME                       ID              SIZE      MODIFIED
qwen3.5:cloud              a7bf6f7891c3    -         30 hours ago
qwen2.5:7b                 845dbda0ea48    4.7 GB    4 days ago
nomic-embed-text:v1.5      0a109f422b47    274 MB    2 months ago
nomic-embed-text:latest    0a109f422b47    274 MB    2 months ago
deepseek-coder:6.7b        ce298d984115    3.8 GB    2 months ago
mistral:latest             6577803aa9a0    4.4 GB    2 months ago
```

#### Créer le script de basculement

```bash
# Créer le script
cat > ~/switch-model.sh << 'EOF'
#!/bin/bash

# Script de changement de modèle OpenClaw
# Usage: ./switch-model.sh <nom-du-modele>

if [ -z "$1" ]; then
  echo "❌ Usage: switch-model.sh <nom-du-modele>"
  echo ""
  echo "Modèles disponibles :"
  ollama list
  echo ""
  echo "Exemple: switch-model.sh qwen3.5:cloud"
  exit 1
fi

MODEL=$1

echo "🔄 Changement vers le modèle : $MODEL"

# Vérifier que le modèle existe
if ! ollama list | grep -q "^$MODEL "; then
  echo "❌ Erreur: Le modèle '$MODEL' n'est pas installé"
  echo ""
  echo "Pour l'installer :"
  echo "  ollama pull $MODEL"
  exit 1
fi

# Backup de la config
cp ~/.openclaw/openclaw.json ~/.openclaw/openclaw.json.backup
echo "💾 Backup créé : ~/.openclaw/openclaw.json.backup"

# Changer la config globale
sed -i "s|\"primary\": \"ollama/.*\"|\"primary\": \"ollama/$MODEL\"|" ~/.openclaw/openclaw.json

# Changer tous les "model" dans la config globale
sed -i "s|\"model\": \"ollama/.*\"|\"model\": \"ollama/$MODEL\"|" ~/.openclaw/openclaw.json

# Changer la config de l'agent
if [ -f ~/.openclaw/agents/main/agent/models.json ]; then
  sed -i "s|\"defaultModel\": \".*\"|\"defaultModel\": \"$MODEL\"|" ~/.openclaw/agents/main/agent/models.json
fi

# Redémarrer le gateway
echo "🔄 Redémarrage du gateway..."
openclaw gateway restart

echo ""
echo "✅ Modèle changé vers : ollama/$MODEL"
echo "✅ Gateway redémarré"
echo ""
echo "Vérification :"
grep "primary" ~/.openclaw/openclaw.json
EOF

# Rendre exécutable
chmod +x ~/switch-model.sh

echo "✅ Script créé : ~/switch-model.sh"
```

#### Utiliser le script

```bash
# Afficher l'aide
~/switch-model.sh

# Changer vers qwen3.5:cloud
~/switch-model.sh qwen3.5:cloud

# Changer vers mistral
~/switch-model.sh mistral:latest

# Changer vers deepseek pour le code
~/switch-model.sh deepseek-coder:6.7b

# Changer vers qwen2.5
~/switch-model.sh qwen2.5:7b
```

#### Sortie exemple

```
🔄 Changement vers le modèle : qwen3.5:cloud
💾 Backup créé : ~/.openclaw/openclaw.json.backup
🔄 Redémarrage du gateway...

✅ Modèle changé vers : ollama/qwen3.5:cloud
✅ Gateway redémarré

Vérification :
  "primary": "ollama/qwen3.5:cloud",
```

#### Restaurer la config précédente

```bash
# Si le changement pose problème
cp ~/.openclaw/openclaw.json.backup ~/.openclaw/openclaw.json
openclaw gateway restart
```

#### Alias pratiques

Ajouter dans `~/.bashrc` :

```bash
# Alias pour changer de modèle
alias switch-qwen='~/switch-model.sh qwen3.5:cloud'
alias switch-mistral='~/switch-model.sh mistral:latest'
alias switch-deepseek='~/switch-model.sh deepseek-coder:6.7b'
alias switch-qwen25='~/switch-model.sh qwen2.5:7b'

# Alias pour voir le modèle actuel
alias current-model='grep "primary" ~/.openclaw/openclaw.json'
```

Puis :
```bash
source ~/.bashrc

# Utilisation
switch-qwen
switch-mistral
current-model
```

---

## Scripts utiles

### Script de vérification complète

```bash
#!/bin/bash

echo "=== VÉRIFICATION OPENCLAW + OLLAMA ==="

# Ollama
echo -e "\n[Ollama]"
if curl -s http://localhost:11434/api/tags > /dev/null; then
    echo "✅ Serveur actif"
    echo "Modèles installés :"
    ollama list
else
    echo "❌ Serveur inactif"
fi

# OpenClaw
echo -e "\n[OpenClaw]"
if command -v openclaw &> /dev/null; then
    echo "✅ Installé ($(openclaw --version))"
    openclaw gateway status
else
    echo "❌ Non installé"
fi

# Configuration
echo -e "\n[Configuration]"
if [ -f ~/.openclaw/openclaw.json ]; then
    echo "✅ Fichier de config présent"
    cat ~/.openclaw/openclaw.json | jq -r '.primary' 2>/dev/null || grep primary ~/.openclaw/openclaw.json
else
    echo "❌ Fichier de config manquant"
fi

# Processus
echo -e "\n[Processus actifs]"
ps aux | grep -E "(ollama|openclaw)" | grep -v grep || echo "Aucun processus trouvé"

echo -e "\n=== FIN ==="
```

### Script de démarrage automatique

```bash
#!/bin/bash
# Sauvegarder dans ~/start-openclaw.sh

# Démarrer Ollama si nécessaire
if ! curl -s http://localhost:11434/api/tags > /dev/null; then
    echo "Démarrage d'Ollama..."
    ollama serve &
    sleep 2
fi

# Démarrer OpenClaw
echo "Démarrage d'OpenClaw..."
openclaw gateway restart

# Lancer l'interface
echo "Lancement de l'interface..."
openclaw tui
```

Rendre exécutable :
```bash
chmod +x ~/start-openclaw.sh
```

---

## Raccourcis utiles

### Aliases bash

Ajouter dans `~/.bashrc` ou `~/.zshrc` :

```bash
# Ollama
alias ol='ollama list'
alias orun='ollama run qwen3.5:cloud'
alias opull='ollama pull'

# OpenClaw
alias oc='openclaw tui'
alias ocr='openclaw gateway restart'
alias ocl='openclaw logs --follow'
alias ocd='openclaw doctor'

# Tests
alias test-ollama='curl http://localhost:11434/api/tags'
alias test-openclaw='openclaw gateway status'
```

Puis :
```bash
source ~/.bashrc
```

---

## Résumé des commandes essentielles

| Tâche | Commande |
|-------|----------|
| Démarrer Ollama | `ollama serve &` |
| Lister modèles | `ollama list` |
| Télécharger modèle | `ollama pull qwen3.5:cloud` |
| Lancer OpenClaw TUI | `openclaw tui` |
| Redémarrer gateway | `openclaw gateway restart` |
| Voir logs | `openclaw logs --follow` |
| Diagnostic | `openclaw doctor` |
| Tester Ollama | `curl http://localhost:11434/api/tags` |
| Tuer processus | `pkill -f ollama && pkill -f openclaw` |
| Config globale | `nano ~/.openclaw/openclaw.json` |

---

[← Retour au README](../README.md) | [Guide de troubleshooting →](TROUBLESHOOTING.md)
