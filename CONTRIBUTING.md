# Guide de contribution

Merci de votre intérêt pour ce projet de documentation OpenClaw ! Ce guide vous aidera à contribuer efficacement.

## Comment contribuer

### Types de contributions acceptées

- 🐛 **Corrections de bugs** : Erreurs dans les commandes, configurations incorrectes
- 📝 **Amélioration de la documentation** : Clarifications, exemples supplémentaires
- ✨ **Nouvelles sections** : Cas d'usage, intégrations, tutoriels
- 🌍 **Traductions** : Documentation en d'autres langues
- 💡 **Suggestions** : Idées d'amélioration via Issues

### Ce qui n'est PAS accepté

- ❌ Modifications du code source d'OpenClaw (voir le [repo officiel](https://github.com/openclaw/openclaw))
- ❌ Ajout de clés API ou credentials
- ❌ Modifications sans explication

## Processus de contribution

### 1. Forker le projet

```bash
# Cliquer sur "Fork" sur GitHub
# Puis cloner votre fork
git clone https://github.com/VOTRE-USERNAME/openclaw-ollama-telegram.git
cd openclaw-ollama-telegram
```

### 2. Créer une branche

```bash
# Créer une branche descriptive
git checkout -b fix/correction-commande-tui
# ou
git checkout -b docs/ajouter-exemple-discord
```

Conventions de nommage :
- `fix/` : Corrections
- `docs/` : Documentation
- `feat/` : Nouvelles fonctionnalités
- `refactor/` : Réorganisation

### 3. Faire vos modifications

```bash
# Éditer les fichiers
nano docs/TROUBLESHOOTING.md

# Vérifier les changements
git status
git diff
```

### 4. Tester vos modifications

**Pour la documentation Markdown** :

```bash
# Vérifier la syntaxe
# Installer un linter Markdown si besoin
npm install -g markdownlint-cli

# Vérifier les fichiers
markdownlint docs/*.md
```

**Pour les commandes shell** :

```bash
# Tester les commandes dans un environnement de test
# S'assurer qu'elles fonctionnent réellement
```

### 5. Commiter vos changements

```bash
# Ajouter les fichiers modifiés
git add docs/TROUBLESHOOTING.md

# Commit avec message descriptif
git commit -m "docs: ajouter solution pour erreur port 11434"
```

**Format des messages de commit** :

```
<type>: <description courte>

[Description détaillée optionnelle]

[Footer optionnel]
```

Types :
- `fix:` → Correction
- `docs:` → Documentation
- `feat:` → Nouvelle fonctionnalité
- `refactor:` → Réorganisation
- `style:` → Formatage
- `test:` → Tests

Exemples :
```
fix: corriger la commande sed dans TROUBLESHOOTING.md

La commande sed pour modifier openclaw.json contenait
une erreur de syntaxe qui empêchait son exécution.

Fixes #42
```

```
docs: ajouter exemple de configuration multi-provider

Ajoute un exemple complet montrant comment configurer
OpenClaw avec Ollama et OpenAI simultanément.
```

### 6. Pousser vers GitHub

```bash
# Pousser vers votre fork
git push origin fix/correction-commande-tui
```

### 7. Créer une Pull Request

1. Aller sur GitHub
2. Cliquer sur "Pull Request"
3. Sélectionner votre branche
4. Remplir la description :

```markdown
## Description
Correction de la commande `sed` dans TROUBLESHOOTING.md qui contenait une erreur de syntaxe.

## Type de changement
- [x] Correction de bug
- [ ] Nouvelle fonctionnalité
- [ ] Documentation

## Tests effectués
- [x] Commande testée dans Ubuntu 22.04
- [x] Syntaxe Markdown validée
- [x] Liens vérifiés

## Checklist
- [x] Mon code suit les conventions du projet
- [x] J'ai testé mes changements
- [x] J'ai mis à jour la documentation si nécessaire
- [x] Mes commits ont des messages descriptifs
```

5. Soumettre la PR

## Standards de documentation

### Format Markdown

```markdown
# Titre principal (H1)

Description brève.

## Section (H2)

Contenu de la section.

### Sous-section (H3)

Détails.
```

### Blocs de code

````markdown
```bash
# Commande shell avec commentaires
ollama pull qwen3.5:cloud
```

```json
{
  "config": "exemple"
}
```
````

### Liens

```markdown
[Texte du lien](URL)
[Guide de troubleshooting](docs/TROUBLESHOOTING.md)
```

### Tableaux

```markdown
| Colonne 1 | Colonne 2 | Colonne 3 |
|-----------|-----------|-----------|
| Valeur 1  | Valeur 2  | Valeur 3  |
```

### Alertes et notes

```markdown
⚠️ **ATTENTION** : Message important

💡 **ASTUCE** : Conseil utile

✅ **BON** : Bonne pratique

❌ **MAUVAIS** : À éviter
```

## Structure du projet

```
openclaw-ollama-telegram/
├── README.md                 # Point d'entrée
├── docs/
│   ├── INSTALLATION.md       # Guide d'installation
│   ├── CONFIGURATION.md      # Configuration détaillée
│   ├── TROUBLESHOOTING.md    # Résolution de problèmes
│   ├── COMMANDS.md           # Référence des commandes
│   └── ARCHITECTURE.md       # Architecture technique
├── .gitignore                # Fichiers à ignorer
└── CONTRIBUTING.md           # Ce fichier
```

## Guidelines de style

### Langue

- Documentation principale : **Français**
- Code et commandes : **Anglais** (conventions shell/JSON)
- Commentaires de code : **Français**

### Ton

- Technique mais accessible
- Utiliser "vous" (pas "tu")
- Exemples concrets
- Pas de jargon inutile

### Exemples

**BON** ✅ :
```markdown
Pour démarrer Ollama, exécutez la commande suivante :

```bash
ollama serve &
```

Cette commande lance le serveur en arrière-plan.
```

**MAUVAIS** ❌ :
```markdown
Lancez le daemon Ollama avec un fork process :

```bash
ollama serve &
```
```

## Checklist avant de contribuer

- [ ] J'ai lu ce guide de contribution
- [ ] J'ai cherché si une issue similaire existe
- [ ] J'ai testé mes modifications
- [ ] Mon code suit les conventions du projet
- [ ] J'ai mis à jour la documentation si nécessaire
- [ ] Mes commits ont des messages descriptifs
- [ ] J'ai créé une branche dédiée
- [ ] Aucun credential ou donnée sensible n'est inclus

## Rapporter un bug

### Créer une issue

1. Aller dans l'onglet "Issues"
2. Cliquer sur "New Issue"
3. Utiliser le template :

```markdown
## Description du bug
Description claire et concise du problème.

## Étapes pour reproduire
1. Aller sur '...'
2. Exécuter '...'
3. Voir l'erreur

## Comportement attendu
Ce qui devrait se passer.

## Comportement actuel
Ce qui se passe réellement.

## Environnement
- OS : Ubuntu 22.04
- OpenClaw : 2026.4.1
- Ollama : 0.15.2

## Logs
```bash
# Coller les logs pertinents
```

## Captures d'écran
Si applicable.
```

## Proposer une amélioration

```markdown
## Description
Description claire de l'amélioration proposée.

## Motivation
Pourquoi cette amélioration est-elle nécessaire ?

## Solution proposée
Comment implémenteriez-vous cette amélioration ?

## Alternatives considérées
Autres approches envisagées.
```

## Questions fréquentes

### Q: Puis-je contribuer sans être développeur ?

**R:** Absolument ! Corrections de typos, clarifications, exemples, tout est bienvenu.

### Q: Comment savoir si ma contribution sera acceptée ?

**R:** Ouvrez d'abord une issue pour discuter des changements majeurs. Pour les petites corrections, créez directement une PR.

### Q: Puis-je traduire la documentation ?

**R:** Oui ! Créez un dossier `docs/[langue]/` (ex: `docs/en/`, `docs/es/`) et traduisez les fichiers.

### Q: J'ai trouvé une erreur de sécurité

**R:** **Ne créez PAS d'issue publique.** Contactez directement le mainteneur.

## Code de conduite

### Nos engagements

- ✅ Être respectueux et inclusif
- ✅ Accepter les critiques constructives
- ✅ Se concentrer sur ce qui est le mieux pour la communauté
- ✅ Montrer de l'empathie envers les autres

### Comportements inacceptables

- ❌ Commentaires insultants ou dégradants
- ❌ Trolling ou attaques personnelles
- ❌ Harcèlement public ou privé
- ❌ Publication d'informations privées sans permission

## Ressources utiles

- [Guide Markdown](https://www.markdownguide.org/)
- [Conventional Commits](https://www.conventionalcommits.org/)
- [GitHub Flow](https://guides.github.com/introduction/flow/)
- [OpenClaw GitHub](https://github.com/openclaw/openclaw)

## Remerciements

Merci à tous les contributeurs qui rendent ce projet meilleur ! 🎉

---

**Des questions ?** Ouvrez une issue avec le label `question`.

**Besoin d'aide ?** Consultez d'abord la [documentation complète](README.md).
