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
