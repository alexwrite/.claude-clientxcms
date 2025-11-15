# Workflow Git - ClientXCMS

Ce document décrit le workflow de travail entre Alexandre et Claude Code pour le développement de ClientXCMS.

## Structure des branches

| Branche | Rôle | Qui commit ? | Push vers |
|---------|------|--------------|-----------|
| **`dev`** (local/home) | Développement continu, corrections, features | Claude + Alexandre | `home/dev` (fork) |
| **`master`** (home) | Préparation Pull Request vers production | Alexandre uniquement | `home/master` (fork) |
| **`master`** (origin) | Production ClientXCMS officiel | Après validation PR | `origin/master` (repo principal) |
| **`old/dev`** | Archive de l'ancienne branche dev avec config perso | Lecture seule | `home/old/dev` |

---

## Workflow complet

### 1️⃣ Claude travaille sur `dev`

Quand Alexandre demande une correction ou une amélioration :

**Claude fait :**
- Modifications du code
- Commits avec messages propres (1 ligne)
- Informe Alexandre quand c'est terminé

**⚠️ Important :** Claude ne push JAMAIS automatiquement. Alexandre décide quand pusher.

**Commandes (Alexandre) :**
```bash
# Vérifier qu'on est sur dev
git status

# Voir les commits de Claude
git log --oneline -10
```

---

### 2️⃣ Alexandre valide et push sur `dev`

Une fois satisfait des modifications de Claude :

```bash
# Push sur le fork
git push home dev
```

**État après cette étape :**
- ✅ Commits sur `home/dev` (fork)
- ❌ Pas encore sur `master`
- ❌ Pas encore en production

---

### 3️⃣ Alexandre prépare la mise en production

#### Option A : TOUS les commits de dev sont bons

```bash
git checkout master
git merge dev --ff-only    # Fast-forward merge
git push home master
```

#### Option B : SÉLECTIONNER certains commits

```bash
git checkout master

# Voir les commits disponibles sur dev
git log dev --oneline -20

# Cherry-pick les commits voulus
git cherry-pick <commit1-sha>
git cherry-pick <commit2-sha>
git cherry-pick <commit3-sha>

# Ou en une seule commande
git cherry-pick <commit1> <commit2> <commit3>

# Push vers le fork
git push home master
```

---

### 4️⃣ Alexandre crée la Pull Request

#### Via GitHub CLI (si installé)
```bash
gh pr create --base master \
             --head alexwrite:master \
             --repo ClientXCMS/ClientXCMS \
             --title "Titre de la PR" \
             --body "Description"
```

#### Via l'interface GitHub
1. Aller sur https://github.com/alexwrite/ClientXCMS
2. Cliquer sur "Compare & pull request"
3. Configurer :
   - Base: `ClientXCMS/master`
   - Head: `alexwrite/master`
4. Remplir titre et description
5. Créer la PR

---

## Schéma du workflow

```
┌─────────────────────────────────────────────────┐
│  Claude corrige/améliore sur dev (local)        │
└─────────────────┬───────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────┐
│  Alexandre valide les commits                   │
│  git log --oneline -10                          │
└─────────────────┬───────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────┐
│  git push home dev                              │
│  (Commits sur le fork alexwrite/ClientXCMS)     │
└─────────────────┬───────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────┐
│  Alexandre sélectionne pour la prod             │
│  git checkout master                            │
│  git cherry-pick <commits> OU git merge dev     │
└─────────────────┬───────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────┐
│  git push home master                           │
│  (Préparation PR sur le fork)                   │
└─────────────────┬───────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────┐
│  Création Pull Request sur GitHub               │
│  alexwrite/master → ClientXCMS/master           │
└─────────────────┬───────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────┐
│  🎉 En production après validation !            │
└─────────────────────────────────────────────────┘
```

---

## Cas d'usage courants

### Claude corrige un bug

```bash
# Alexandre demande
"Claude, corrige le bug dans EmailController"

# Claude fait les modifs et commit
[Claude modifie, commit, informe]

# Alexandre valide et push
git push home dev
```

### Mise en production sélective

```bash
# Voir les commits de dev
git log dev --oneline -15

# Sélectionner uniquement les commits de bug fix (pas les features)
git checkout master
git cherry-pick abc1234 def5678 ghi9012
git push home master

# Créer la PR sur GitHub
```

### Nettoyer dev après mise en prod

Quand `master` est à jour et qu'on veut repartir d'une base propre :

```bash
git checkout dev
git reset --hard master       # Dev = Master
git push home dev --force     # Force update du fork
```

### Récupérer un commit de old/dev

Si besoin de récupérer un ancien commit :

```bash
# Voir les commits de old/dev
git log old/dev --oneline -20

# Cherry-pick sur dev
git checkout dev
git cherry-pick <commit-sha-from-old-dev>
```

---

## Règles importantes

### ✅ À faire

- **Alexandre** : Toujours vérifier les commits de Claude avant de push
- **Alexandre** : Utiliser des messages de commit clairs (1 ligne)
- **Claude** : Ne jamais push automatiquement
- **Claude** : Toujours informer Alexandre quand un travail est terminé
- **Les deux** : Garder `master` propre (seulement code production)

### ❌ À ne pas faire

- **Claude** : Ne jamais push sur `dev` ou `master` sans autorisation
- **Alexandre** : Ne jamais push directement sur `origin/master` (toujours via PR)
- **Les deux** : Ne jamais force push sur `origin/master`
- **Les deux** : Ne pas commiter des fichiers de config perso sur `master` (devcontainer, etc.)

---

## Configuration des remotes

```bash
# Vérifier les remotes
git remote -v

# Devrait afficher :
# home    git@github.com:alexwrite/ClientXCMS.git (fetch/push)
# origin  git@github.com:ClientXCMS/ClientXCMS.git (fetch/push)
```

**home** : Fork personnel d'Alexandre (lecture + écriture)
**origin** : Repo officiel ClientXCMS (lecture seule pour Alexandre, écriture via PR)

---

## Commandes utiles

### Voir l'état des branches

```bash
# Branches locales
git branch

# Branches locales + distantes
git branch -a

# Voir les commits en avance/retard
git log --oneline --graph --decorate --all -20
```

### Comparer des branches

```bash
# Voir les commits de dev qui ne sont pas sur master
git log master..dev --oneline

# Voir les différences de fichiers
git diff master..dev

# Voir les commits de master qui ne sont pas sur origin/master
git log origin/master..master --oneline
```

### Synchroniser avec origin

```bash
# Récupérer les dernières modifications d'origin
git fetch origin

# Voir ce qui a changé sur origin/master
git log HEAD..origin/master --oneline

# Mettre à jour master depuis origin (si besoin)
git checkout master
git merge origin/master --ff-only
```

---

## Gestion des conflits

Si un conflit survient lors d'un merge/cherry-pick :

```bash
# Voir les fichiers en conflit
git status

# Éditer les fichiers pour résoudre les conflits
# (Supprimer les marqueurs <<<< ==== >>>>)

# Marquer comme résolu
git add <fichier-resolu>

# Continuer le merge/cherry-pick
git cherry-pick --continue
# ou
git merge --continue

# Ou annuler en cas de problème
git cherry-pick --abort
# ou
git merge --abort
```

---

## Notes importantes

- **Commits de dev** : Peuvent contenir config perso, outils, docs Claude
- **Commits de master** : Uniquement code production, pas de config perso
- **Pull Requests** : Toujours depuis `alexwrite/master` vers `ClientXCMS/master`
- **Branches Claude** : Ne pas les garder sur le remote (supprimer après utilisation)

---

## Historique des branches

- **2025-11-11** : Création de `old/dev` (archive de l'ancienne dev avec config perso)
- **2025-11-11** : Création de `dev` propre à partir de `master`
- **2025-11-11** : Suppression des branches `claude/analyze-*` du remote

---

## Contact et support

- **Dépôt officiel** : https://github.com/ClientXCMS/ClientXCMS
- **Fork personnel** : https://github.com/alexwrite/ClientXCMS
- **Documentation** : Voir `.claude/CLAUDE.md` pour les instructions du projet
