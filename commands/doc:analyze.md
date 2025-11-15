# Documentation : Analyser et décomposer la documentation Docusaurus

Analyse une page web (URL) pour améliorer la documentation Docusaurus locale avec **décomposition intelligente par sujets**.

**OBJECTIF** : Identifier les écarts entre documentation et code, puis décomposer le contenu en fichiers modulaires cohérents.

**RÉSULTAT** : Documentation Docusaurus organisée en fichiers thématiques après validation interactive

---

## RÈGLES STRICTES

1. **TOUJOURS WebFetch** l'URL fournie
2. **ANALYSER LA STRUCTURE COMPLÈTE** : Tous les fichiers du dossier, pas un seul
3. **DÉCOMPOSER PAR SUJETS** : Proposer une organisation modulaire
4. **ÉVITER LA DUPLICATION** : Identifier et éliminer les doublons entre fichiers
5. **VALIDATION FICHIER PAR FICHIER** : Demander validation pour chaque fichier modifié/créé
6. **RESPECTER LA HIÉRARCHIE** : Maintenir la structure Docusaurus (sidebar, liens)

---

## FORMAT D'APPEL

```bash
/doc:analyze https://docs.clientxcms.com/developpers/extensions/
```

---

## PROCESSUS DÉTAILLÉ

### PHASE 1 : ANALYSE DE LA STRUCTURE (Automatique)

#### Étape 1.1 : WebFetch de l'URL

```
WebFetch: [URL fournie]
Prompt: "Extraire tout le contenu markdown"
```

**Identifier** :
- Sujet principal de la page
- Sections principales (H1, H2, H3)
- Sujets abordés
- Longueur du contenu

#### Étape 1.2 : Mapping et découverte de la structure

**Règle de mapping** :
```
https://docs.clientxcms.com/[path]/
→ .claude/docs/docs.clientxcms.com/docs/[path]/
```

**Actions** :
1. Extraire le path : `/developpers/extensions/`
2. Chercher le dossier : `.claude/docs/docs.clientxcms.com/docs/developpers/extensions/`
3. **Lister TOUS les fichiers .md du dossier** (pas seulement le fichier principal)
4. Créer une carte de la structure existante

**Exemple de découverte** :
```
Dossier trouvé : docs/developpers/extensions/
Fichiers existants :
  1. extensions.md (fichier principal)
  2. create.md
  3. configuration.md
  4. routes.md
  5. database.md
  6. schedules.md
  7. definitions/
     - permissions.md
     - translations.md
     - models.md
     - events.md
  8. implementation_guides/
     - gateway.md
     - product/
       - product.md
       - server.md
       - configuration.md
       - data.md
       - panel.md
     - settings.md
     - navigation.md
     - email.md
```

#### Étape 1.3 : Lecture de TOUS les fichiers du dossier

```
Pour chaque fichier trouvé :
  Read: [chemin du fichier]

Extraire :
  - Titre principal (H1)
  - Sections (H2, H3)
  - Sujets traités
  - Exemples de code présents
  - Longueur (nombre de lignes)
  - Références à d'autres fichiers
```

#### Étape 1.4 : Création de la carte structurelle

```markdown
📍 STRUCTURE DÉCOUVERTE

URL analysée : https://docs.clientxcms.com/developpers/extensions/
Dossier local : docs/developpers/extensions/

## FICHIERS EXISTANTS (10 fichiers)

### Fichier principal
- extensions.md (33 lignes)
  Sujets : Introduction, différence addon/module, activation

### Fichiers thématiques
- create.md (61 lignes)
  Sujets : Commande artisan, structure de fichiers

- configuration.md (67 lignes)
  Sujets : addon.json, composer.json

- routes.md (74 lignes)
  Sujets : Routes admin, routes publiques, middlewares

- database.md (127 lignes)
  Sujets : Migrations, seeders, modèles

- schedules.md (88 lignes)
  Sujets : Tâches planifiées, cron jobs

### Sous-dossiers
- definitions/ (4 fichiers)
- implementation_guides/ (8 fichiers)

Prêt à analyser tous ces fichiers ? [Continuer]
```

---

### PHASE 2 : ANALYSE DU CODE CLIENTXCMS (Automatique)

**Pour CHAQUE sujet identifié dans la documentation** :

#### Étape 2.1 : Recherche dans le code

| Sujet doc | Recherche code |
|-----------|----------------|
| BaseAddonServiceProvider | `Grep "class BaseAddonServiceProvider"` |
| Exemple addon Contact | `Read addons/contact/src/ContactServiceProvider.php` |
| Structure addon | `Glob pattern="**/addon.json" path="addons/"` |
| Widgets admin | `Grep "AdminCountWidget"` |
| Settings | `Grep "setDefaultValue\|addCardItem"` |
| Permissions | `Read addons/*/permissions.json` |
| Routes | `Glob pattern="**/routes/*.php" path="addons/"` |
| Migrations | `Glob pattern="**/migrations/*.php" path="addons/"` |

#### Étape 2.2 : Analyse de 2-3 addons de référence

```
Read: addons/contact/src/ContactServiceProvider.php
Read: addons/faq/src/FaqServiceProvider.php
Read: addons/contact/addon.json
Read: addons/contact/permissions.json
Read: addons/contact/emails.json
```

**Créer une base de connaissances** :
- Quelles méthodes sont disponibles ?
- Quels patterns sont utilisés en pratique ?
- Quels fichiers sont présents dans les addons réels ?

---

### PHASE 3 : ANALYSE DES DOUBLONS ET INCOHÉRENCES (Automatique)

#### Étape 3.1 : Détection des doublons entre fichiers

**Algorithme** :
1. Pour chaque sujet identifié, noter dans quels fichiers il apparaît
2. Identifier les doublons (même sujet traité dans plusieurs fichiers)
3. Identifier les incohérences (informations contradictoires entre fichiers)

**Exemple de rapport de doublons** :

| Sujet | Fichier 1 | Fichier 2 | Statut |
|-------|-----------|-----------|--------|
| Structure addon.json | extensions.md | configuration.md | 🟡 Doublon partiel |
| Méthodes ServiceProvider | extensions.md | create.md | 🟡 Doublon partiel |
| Routes admin | extensions.md | routes.md | ✅ Pas de doublon (juste mention) |
| Widgets AdminCountWidget | Absent | Absent | 🔴 Manquant |
| Fichier permissions.json | Absent | configuration.md | 🟡 Incomplet |

#### Étape 3.2 : Identification des sujets manquants

**Scan du code vs documentation** :

Sujets dans le code mais absents de la doc :
1. `AdminCountWidget` : Utilisé dans `addons/contact/` mais non documenté
2. `addProtectedRoute()` : Méthode disponible mais non documentée
3. `permissions.json` : Format non documenté complètement
4. `emails.json` : Format non documenté
5. Commandes artisan : `clientxcms:create-extension` mentionné mais pas détaillé

#### Étape 3.3 : Comparaison page web ↔ fichiers locaux ↔ code

| Élément | Page web | Fichier local | Code réel | Statut |
|---------|----------|---------------|-----------|--------|
| BaseAddonServiceProvider | Mentionné | Mentionné (33 lignes) | Existe (72 lignes) | ⚠️ Incomplet |
| Structure fichiers | Partielle | Partielle | Complète (addons/contact) | ⚠️ Incomplet |
| Widgets admin | Absent | Absent | Existe (AdminCountWidget) | 📝 Non documenté |

---

### PHASE 4 : PROPOSITION DE DÉCOMPOSITION (Interactive)

#### Étape 4.1 : Présentation du plan de réorganisation

```markdown
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📊 PLAN DE DÉCOMPOSITION
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## ANALYSE ACTUELLE

**Problèmes identifiés** :
1. ❌ Fichier extensions.md trop court (33 lignes) pour être un hub
2. 🟡 Doublons entre extensions.md et configuration.md (addon.json)
3. 📝 5 fonctionnalités non documentées (widgets, emails.json, etc.)
4. ⚠️ Informations manquantes dans plusieurs fichiers existants

**Contenu manquant total** : ~300 lignes à ajouter

## STRATÉGIE DE DÉCOMPOSITION PROPOSÉE

### Option 1 : Organisation par niveaux (RECOMMANDÉ)

**extensions.md** (fichier hub - 120 lignes)
→ Rôle : Vue d'ensemble, par où commencer
- Qu'est-ce qu'une extension ?
- Addon vs Module (tableau comparatif)
- Cas d'usage avec exemples réels
- Arbre de navigation vers les autres guides
- Par où commencer selon l'objectif

**create.md** (guide de création - 150 lignes)
→ Rôle : Comment créer son addon pas à pas
- Commande artisan clientxcms:create-extension
- Structure de fichiers complète (obligatoires + optionnels)
- Exemple minimal fonctionnel
- ServiceProvider détaillé

**configuration.md** (fichiers de config - 120 lignes)
→ Rôle : Format des fichiers de configuration
- addon.json / module.json (complet)
- composer.json
- permissions.json (NOUVEAU)
- emails.json (NOUVEAU)

**advanced.md** (fonctionnalités avancées - NOUVEAU - 180 lignes)
→ Rôle : Features avancées du ServiceProvider
- Widgets admin (AdminCountWidget)
- Settings personnalisés (setDefaultValue, addCardItem)
- Routes protégées (addProtectedRoute)
- Commandes artisan personnalisées
- Tâches planifiées (référence vers schedules.md)

**troubleshooting.md** (dépannage - NOUVEAU - 80 lignes)
→ Rôle : Déboguer une extension
- 5 problèmes courants + solutions
- Logs utiles
- Checklist de vérification

### Option 2 : Organisation par fonctionnalité

[Alternative si Option 1 ne convient pas]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## PLAN D'ACTION

Si tu valides l'Option 1 :

1. ✏️  Enrichir extensions.md (33 → 120 lignes)
2. ✏️  Enrichir create.md (61 → 150 lignes)
3. ✏️  Enrichir configuration.md (67 → 120 lignes)
4. ➕ Créer advanced.md (nouveau - 180 lignes)
5. ➕ Créer troubleshooting.md (nouveau - 80 lignes)
6. 🔄 Mettre à jour les liens internes
7. 📋 Mettre à jour le sidebar.ts

**Total** : 5 fichiers modifiés/créés, ~650 lignes ajoutées

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Quelle option préfères-tu ?
[1] Option 1 (organisation par niveaux)
[2] Option 2 (organisation par fonctionnalité)
[M] Proposer une autre organisation
[Q] Quitter

Ton choix :
```

#### Étape 4.2 : Validation de la stratégie

Attendre la validation de l'utilisateur avant de continuer.

---

### PHASE 5 : MODIFICATIONS FICHIER PAR FICHIER (Interactive)

**Pour CHAQUE fichier à modifier/créer** :

#### Template de présentation d'une modification de fichier

```markdown
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
FICHIER 1/5 : extensions.md
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📂 Type : Modification (enrichissement)
📍 Chemin : docs/developpers/extensions/extensions.md
📏 Taille : 33 lignes → 120 lignes (+87 lignes)

## RÔLE DE CE FICHIER

Fichier hub servant de point d'entrée pour toute la documentation des extensions.
Doit permettre de comprendre rapidement les concepts et orienter vers les guides détaillés.

## MODIFICATIONS PROPOSÉES

### 1. Enrichir la section "Addon vs Module" (+30 lignes)

**Actuellement** :
- 2 paragraphes courts
- Pas d'exemples concrets

**Proposition** :
- Tableau comparatif détaillé
- Cas d'usage avec exemples réels du code (Contact, Pterodactyl)
- Références aux fichiers sources

### 2. Ajouter section "Que peut-on créer ?" (+25 lignes)

**Contenu** :
- Liste des types d'extensions possibles
- Exemples réels pour chaque type avec chemins
  - Passerelles : Stripe (app/Core/Gateway/StripeType.php)
  - Pages : FAQ (addons/faq/), Contact (addons/contact/)
  - Panels : Pterodactyl (modules/pterodactyl/)

### 3. Ajouter section "Par où commencer ?" (+20 lignes)

**Contenu** :
- Parcours recommandé pour débutants
- Tableau "Vous voulez créer X ? → Suivez le guide Y"
- Références aux addons à étudier

### 4. Améliorer la navigation (+12 lignes)

**Contenu** :
- Tableau des guides disponibles avec descriptions
- Colonne "Référence code" pour chaque guide

## APERÇU DU CONTENU AJOUTÉ

```markdown
## Que peut-on créer avec les extensions ?

### Addons : Fonctionnalités et intégrations

**Exemples d'addons existants dans le projet** :

- **Passerelles de paiement** : Stripe (`app/Core/Gateway/StripeType.php`), PayPal, Revolut
- **Pages personnalisées** : FAQ (`addons/faq/`), Contact (`addons/contact/`)
- **Fonctionnalités métier** : Système de crédit (`addons/fund/`)

**Cas d'usage typiques** :
- Ajouter une méthode de paiement → Implémenter `AbstractGatewayType`
- Créer des pages front-end → Routes + Vues Blade
- Ajouter des widgets admin → `AdminCountWidget`

➡️ [Voir le guide détaillé](/developpers/extensions/create)
```

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Actions :
[A] Appliquer ces modifications
[P] Prévisualiser le fichier complet
[M] Modifier les propositions
[S] Sauter ce fichier
[Q] Quitter

Ton choix :
```

#### Traitement des réponses

**Si [A] Appliquer** :

```
Edit: docs/developpers/extensions/extensions.md
old_string: [contenu actuel exact]
new_string: [contenu enrichi exact]
```

Afficher :
```markdown
✅ FICHIER MODIFIÉ

Fichier : docs/developpers/extensions/extensions.md
Modifications : 4 sections ajoutées/enrichies
Lignes : 33 → 120 (+87 lignes)

Passage au fichier suivant...
```

**Si [P] Prévisualiser** :

```
Afficher le fichier complet après modifications (avec numéros de ligne)
```

Puis redemander l'action.

**Si [M] Modifier** :

```markdown
📝 MODIFICATION DES PROPOSITIONS

Quelle partie veux-tu modifier ?
[1] Section "Addon vs Module"
[2] Section "Que peut-on créer ?"
[3] Section "Par où commencer ?"
[4] Navigation
[T] Tout (donner des instructions globales)

Ton choix :
```

**Si [S] Sauter** :

```markdown
⏭️  FICHIER SAUTÉ

Ce fichier ne sera pas modifié.
```

---

### PHASE 6 : CRÉATION DE NOUVEAUX FICHIERS (Interactive)

**Pour chaque nouveau fichier à créer** :

```markdown
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
NOUVEAU FICHIER 4/5 : advanced.md
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📂 Type : Création (nouveau fichier)
📍 Chemin : docs/developpers/extensions/advanced.md
📏 Taille : 180 lignes
🎯 Frontmatter requis : Oui

## JUSTIFICATION

Ce fichier regroupe les fonctionnalités avancées du ServiceProvider qui sont actuellement
non documentées mais présentes dans le code (AdminCountWidget, addProtectedRoute, etc.).

Ces fonctionnalités ne trouvent pas leur place dans les guides existants :
- create.md : Trop basique pour ces features
- configuration.md : Concerne les fichiers de config, pas le ServiceProvider
- implementation_guides/ : Trop spécifique (gateway, product, etc.)

## CONTENU DU FICHIER

### Structure proposée

```markdown
---
sidebar_position: 6
---

# Fonctionnalités avancées

Ce guide couvre les fonctionnalités avancées disponibles dans le ServiceProvider
de vos extensions.

## Prérequis

- Avoir créé une extension de base (voir [Créer une extension](/developpers/extensions/create))
- Comprendre le rôle du ServiceProvider

## Widgets dans le dashboard admin

Les widgets permettent d'afficher des compteurs dans le tableau de bord administrateur.

### Créer un widget de comptage

```php
use App\Core\Admin\Dashboard\AdminCountWidget;
use App\Models\Admin\Permission;

public function boot()
{
    $widget = new AdminCountWidget(
        'contacts',                    // UUID unique
        'bi bi-envelope',             // Icône Bootstrap Icons
        'contact::lang.pending',      // Titre (traduction)
        function () {                 // Valeur (callable ou statique)
            return Contact::where('read', false)->count();
        },
        Permission::MANAGE_EXTENSIONS // Permission requise
    );

    $this->app['extension']->addAdminCountWidget($widget);
}
```

**Exemple réel** : `addons/contact/src/ContactServiceProvider.php:59`

[... 150 lignes supplémentaires avec sections Settings, Routes protégées, Commandes artisan]
```

## APERÇU PARTIEL (50 premières lignes)

[Afficher le début du fichier]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Actions :
[A] Créer ce fichier
[P] Prévisualiser le fichier complet
[M] Modifier le contenu
[S] Ne pas créer ce fichier
[Q] Quitter

Ton choix :
```

**Si [A] Créer** :

```
Write: docs/developpers/extensions/advanced.md
content: [contenu complet]
```

---

### PHASE 7 : MISE À JOUR DE LA NAVIGATION (Automatique si fichiers créés)

Si de nouveaux fichiers ont été créés, proposer de mettre à jour `sidebars.ts` :

```markdown
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔄 MISE À JOUR DE LA NAVIGATION
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

De nouveaux fichiers ont été créés. Je dois mettre à jour le fichier de navigation
Docusaurus pour qu'ils soient accessibles dans le sidebar.

## MODIFICATIONS À APPORTER

Fichier : sidebars.ts

**Nouveaux items à ajouter** dans la section `developpers/extensions` :

```typescript
{
  type: 'category',
  label: 'Extensions',
  items: [
    'developpers/extensions/extensions',
    'developpers/extensions/create',
    'developpers/extensions/configuration',
    'developpers/extensions/routes',
    'developpers/extensions/database',
    'developpers/extensions/schedules',
    'developpers/extensions/advanced',        // ← NOUVEAU
    'developpers/extensions/troubleshooting', // ← NOUVEAU
    {
      type: 'category',
      label: 'Définitions',
      items: [...]
    },
    {
      type: 'category',
      label: 'Guides d\'implémentation',
      items: [...]
    }
  ]
}
```

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[A] Appliquer la mise à jour
[M] Modifier l'ordre des items
[S] Je le ferai manuellement
[V] Voir le fichier sidebars.ts complet

Ton choix :
```

---

### PHASE 8 : RÉSUMÉ FINAL (Automatique)

```markdown
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🎉 DÉCOMPOSITION ET AMÉLIORATION TERMINÉES
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## STATISTIQUES FINALES

### Fichiers traités
- ✏️  Modifiés : 3 fichiers
- ➕ Créés : 2 fichiers
- ⏭️  Sautés : 0 fichiers
- 🔄 Navigation : Mise à jour

### Détail des modifications

**1. extensions.md** (✏️  Modifié)
- Lignes : 33 → 120 (+87)
- Ajouts : 3 nouvelles sections
- Rôle : Fichier hub avec vue d'ensemble

**2. create.md** (✏️  Modifié)
- Lignes : 61 → 150 (+89)
- Ajouts : Détails structure, exemple complet
- Rôle : Guide de création pas à pas

**3. configuration.md** (✏️  Modifié)
- Lignes : 67 → 120 (+53)
- Ajouts : permissions.json, emails.json
- Rôle : Référence des fichiers de config

**4. advanced.md** (➕ Créé)
- Lignes : 180 (nouveau)
- Contenu : Widgets, settings avancés, commandes artisan
- Rôle : Features avancées du ServiceProvider

**5. troubleshooting.md** (➕ Créé)
- Lignes : 80 (nouveau)
- Contenu : 5 problèmes courants + solutions
- Rôle : Guide de dépannage

### Total
- Fichiers modifiés/créés : 5
- Lignes ajoutées : ~489
- Doublons éliminés : 3
- Fonctionnalités documentées : 8 (nouvelles)

## ORGANISATION FINALE

```
docs/developpers/extensions/
├── extensions.md (hub - 120 lignes)
├── create.md (création - 150 lignes)
├── configuration.md (config - 120 lignes)
├── routes.md (routing - 74 lignes)
├── database.md (BDD - 127 lignes)
├── schedules.md (cron - 88 lignes)
├── advanced.md (avancé - 180 lignes) ← NOUVEAU
├── troubleshooting.md (debug - 80 lignes) ← NOUVEAU
├── definitions/
│   ├── permissions.md
│   ├── translations.md
│   ├── models.md
│   └── events.md
└── implementation_guides/
    ├── gateway.md
    ├── settings.md
    ├── navigation.md
    ├── email.md
    └── product/
        ├── product.md
        ├── server.md
        ├── configuration.md
        ├── data.md
        └── panel.md
```

## AVANTAGES DE CETTE ORGANISATION

✅ **Modularité** : Chaque fichier a un rôle clair et ciblé
✅ **Pas de duplication** : Informations uniques par fichier
✅ **Navigation claire** : Hub → Guides spécifiques
✅ **Maintenabilité** : Facile de mettre à jour un sujet précis
✅ **Complétude** : Toutes les features du code sont documentées
✅ **Progression logique** : Du basique (create) à l'avancé (advanced)

## PROCHAINES ÉTAPES

1. 🧪 Tester la documentation localement
   ```bash
   cd .claude/docs/docs.clientxcms.com
   npm run start
   ```

2. 📸 Vérifier la navigation Docusaurus
   - Sidebar correctement mis à jour
   - Liens internes fonctionnels
   - Frontmatter correct

3. ✅ Valider le contenu
   - Relire les modifications
   - Tester les exemples de code
   - Vérifier les références aux fichiers sources

4. 💾 Commit si satisfait
   ```bash
   git add docs/developpers/extensions/
   git commit -m "docs(extensions): décomposition modulaire et enrichissement

   - Enrichi extensions.md (hub avec vue d'ensemble)
   - Enrichi create.md (guide détaillé de création)
   - Enrichi configuration.md (permissions.json, emails.json)
   - Ajouté advanced.md (widgets, settings avancés, commandes)
   - Ajouté troubleshooting.md (guide de dépannage)
   - Mis à jour sidebars.ts"
   ```

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Veux-tu :
[R] Voir le rapport complet détaillé
[D] Voir les diffs de chaque fichier
[T] Tester localement (npm run start)
[G] Générer un fichier CHANGELOG.md
[F] Terminer

Ton choix :
```

---

## CAS SPÉCIAUX À GÉRER

### Cas 1 : Contenu trop long pour un seul fichier

Si après enrichissement, un fichier dépasse 400 lignes :

```markdown
⚠️  FICHIER TROP LONG DÉTECTÉ

Fichier : create.md
Taille après modifications : 520 lignes

Un fichier de documentation au-delà de 400 lignes devient difficile à maintenir.

PROPOSITION DE SPLIT :

Option 1 : Diviser par niveau de complexité
- create.md (150 lignes) : Création basique
- create-advanced.md (180 lignes) : Création avancée
- create-examples.md (190 lignes) : Exemples complets

Option 2 : Diviser par composants
- create.md (200 lignes) : Vue d'ensemble et structure
- create-serviceprovider.md (170 lignes) : ServiceProvider détaillé
- create-examples.md (150 lignes) : Exemples d'addons

Quelle option préfères-tu ?
[1] Option 1
[2] Option 2
[M] Autre organisation
[G] Garder en un seul fichier (non recommandé)

Ton choix :
```

### Cas 2 : Doublons détectés entre fichiers

```markdown
🔍 DOUBLON DÉTECTÉ

Sujet : Format de addon.json

**Présent dans** :
1. extensions.md (lignes 45-60) : Exemple basique
2. configuration.md (lignes 12-35) : Documentation complète

**Problème** : Duplication partielle de l'information

PROPOSITION :

- extensions.md : Garder uniquement une mention + lien vers configuration.md
  ```markdown
  Chaque addon doit avoir un fichier `addon.json` définissant ses métadonnées.

  ➡️ [Voir la documentation complète de addon.json](/developpers/extensions/configuration#addon-json)
  ```

- configuration.md : Garder la documentation complète avec tous les champs

[A] Appliquer cette correction
[M] Modifier la proposition
[S] Garder le doublon (non recommandé)

Ton choix :
```

### Cas 3 : Fichier avec frontmatter existant

Lors de la modification d'un fichier avec frontmatter :

```markdown
ℹ️  FRONTMATTER DÉTECTÉ

Fichier : extensions.md

Frontmatter actuel :
```yaml
---
sidebar_position: 1
title: Extensions
---
```

Je vais préserver le frontmatter existant et modifier uniquement le contenu.

[OK] Continuer
```

---

## RÈGLES DE DÉCOMPOSITION INTELLIGENTE

### Taille optimale des fichiers

- **Fichier hub** (intro générale) : 100-150 lignes
- **Guide spécifique** : 150-300 lignes
- **Référence technique** : 200-400 lignes
- **⚠️ Au-delà de 400 lignes** : Proposer un split

### Critères de séparation

**Séparer en fichiers différents si** :
1. Sujets indépendants (création vs configuration vs routing)
2. Niveaux différents (basique vs avancé)
3. Audiences différentes (débutant vs expert)
4. Types différents (guide vs référence vs exemples)

**Garder dans le même fichier si** :
1. Sujets fortement liés (impossible de comprendre l'un sans l'autre)
2. Contexte partagé nécessaire
3. Fichier court (<150 lignes après enrichissement)

### Hiérarchie des fichiers

```
Niveau 1 : Hub / Vue d'ensemble
  └─ extensions.md : Introduction, navigation, par où commencer

Niveau 2 : Guides fondamentaux
  ├─ create.md : Comment créer une extension
  ├─ configuration.md : Fichiers de configuration
  └─ routes.md : Système de routing

Niveau 3 : Guides spécialisés
  ├─ database.md : Migrations et modèles
  ├─ schedules.md : Tâches planifiées
  └─ advanced.md : Fonctionnalités avancées

Niveau 4 : Guides d'implémentation
  └─ implementation_guides/
      ├─ gateway.md : Passerelles de paiement
      ├─ product/ : Types de produits
      └─ ...

Niveau 5 : Utilitaires
  └─ troubleshooting.md : Dépannage
```

---

## NOTES IMPORTANTES

### Ce que cette commande FAIT

- ✅ Analyse la structure complète du dossier
- ✅ Identifie les doublons entre fichiers
- ✅ Propose une décomposition modulaire intelligente
- ✅ Modifie/crée les fichiers un par un avec validation
- ✅ Élimine les duplications
- ✅ Met à jour la navigation Docusaurus
- ✅ Respecte les frontmatters existants

### Ce que cette commande NE FAIT PAS

- ❌ Ne fusionne pas plusieurs fichiers en un seul
- ❌ Ne supprime pas de fichiers existants
- ❌ Ne modifie pas les fichiers sans validation
- ❌ Ne touche pas au code ClientXCMS

### Limitations

- Analyse un dossier à la fois (pas récursif profond)
- Ne gère pas les images (signale si manquantes)
- Ne traduit pas (reste dans la langue d'origine)

---

**C'EST PARTI !**

Fournis l'URL à analyser :
```
/doc:analyze <url>
```

La commande va :
1. Analyser la structure complète du dossier
2. Proposer une décomposition modulaire
3. Appliquer les modifications fichier par fichier avec validation
4. Créer de nouveaux fichiers si nécessaire
5. Éliminer les doublons
6. Mettre à jour la navigation
