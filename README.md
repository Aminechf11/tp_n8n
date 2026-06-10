# Assistant de Gestion des Candidatures — TP n8n

Workflow n8n qui traite automatiquement les emails de candidature : détection, enregistrement dans Google Sheets, réponse automatique au candidat et notification de l'équipe RH sur Discord.

## Cas d'usage

Un candidat envoie un email. n8n l'intercepte, détecte s'il s'agit d'une candidature, enregistre les informations dans un tableau de suivi, envoie une confirmation personnalisée au candidat, et notifie l'équipe RH sur Discord — le tout en moins de 60 secondes, sans intervention humaine.

## Schéma du workflow

```
┌─────────────────┐
│  Gmail Trigger  │  Vérifie les nouveaux emails toutes les minutes
│  Nouveau Email  │
└────────┬────────┘
         │
         ▼
┌─────────────────────┐
│  Code (JavaScript)  │  Extrait : nom, email, date
│  Extraire et        │  Détecte : mots-clés de candidature
│  Classifier         │  Calcule : score de détection
└────────┬────────────┘
         │
         ▼
┌──────────────────────┐
│  IF                  │
│  Est une Candidature?│
└──────┬───────────────┘
       │                     │
    OUI (true)            NON (false)
       │                     │
       ▼                     ▼
┌─────────────────┐   ┌──────────────────┐
│  Google Sheets  │   │  Gmail           │
│  Enregistrer    │   │  Réponse         │
│  dans Sheets    │   │  Générique       │
└────────┬────────┘   └──────────────────┘
         │
         ▼
┌─────────────────┐
│  Gmail          │
│  Confirmation   │
│  au Candidat   │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Discord        │
│  Notification   │
│  Équipe RH      │
└─────────────────┘
```

## Services utilisés

| Service | Rôle |
|---------|------|
| **Gmail** (trigger) | Réception des emails entrants |
| **JavaScript (Code)** | Extraction et classification automatique |
| **Google Sheets** | Suivi et stockage des candidatures |
| **Gmail** (action) | Réponse automatique au candidat |
| **Discord** | Notification temps réel de l'équipe RH |

## Prérequis

- [n8n](https://n8n.io) installé (version 1.0+) — `npx n8n` pour démarrer
- Un compte Gmail avec OAuth2 configuré
- Un Google Sheets avec un onglet nommé `Candidatures`
- Un webhook Discord pour les notifications

## Installation rapide

### 1. Lancer n8n

```bash
npx n8n
# ou avec Docker :
docker run -it --rm --name n8n -p 5678:5678 n8nio/n8n
```

### 2. Importer le workflow

1. Ouvrir n8n sur `http://localhost:5678`
2. Créer un nouveau workflow
3. Cliquer sur le menu `⋮` → **Import from file**
4. Sélectionner `workflows/assistant-candidatures.json`

### 3. Configurer les credentials

Dans n8n, ajouter les credentials suivants :

**Gmail OAuth2**
- `Settings` → `Credentials` → `Add Credential` → `Gmail OAuth2 API`
- Suivre le guide OAuth2 de Google Cloud Console

**Google Sheets OAuth2**
- `Settings` → `Credentials` → `Add Credential` → `Google Sheets OAuth2 API`
- Utiliser le même projet Google Cloud que Gmail

**Discord Webhook**
- Dans Discord : `Paramètres du serveur` → `Intégrations` → `Webhooks`
- Copier l'URL du webhook dans le nœud `Notifier Discord`

### 4. Configurer Google Sheets

Créer un Google Sheets avec un onglet `Candidatures` et les colonnes suivantes :

| Date | Nom | Email | Sujet | Resume | Mots cles | Statut | Email envoye |
|------|-----|-------|-------|--------|-----------|--------|--------------|

Mettre à jour le nœud `Enregistrer dans Sheets` avec l'ID de votre Spreadsheet (visible dans l'URL : `docs.google.com/spreadsheets/d/**SPREADSHEET_ID**/edit`).

### 5. Activer le workflow

Cliquer sur le toggle **Inactive → Active** en haut à droite.

## Structure du projet

```
n8n-assistant-email/
├── workflows/
│   ├── assistant-candidatures.json       # Workflow principal (règles)
│   └── assistant-candidatures-ia.json    # Version IA (OpenAI GPT-4o-mini)
├── docs/
│   ├── setup.md                          # Guide de configuration détaillé
│   └── demo.md                           # Script de démonstration
├── assets/
│   └── screenshots/                      # Captures d'écran du workflow
└── README.md
```

## Détection automatique des candidatures

Le workflow analyse le contenu de l'email à la recherche de ces mots-clés :

> candidature · stage · emploi · poste · recrutement · cv · curriculum vitae · lettre de motivation · postuler · alternance · apprentissage · internship · offre · application · apply

Si au moins un mot-clé est présent → email traité comme une candidature.

## Version IA (bonus)

Le fichier `workflows/assistant-candidatures-ia.json` propose une version enrichie qui utilise **GPT-4o-mini** (OpenAI) pour :

- Classifier l'email de façon plus intelligente
- Extraire le type de poste visé (CDI, Stage, Alternance…)
- Identifier les compétences mentionnées
- **Générer une réponse personnalisée** adaptée au contexte de la candidature

### Prérequis supplémentaires pour la version IA

- Clé API OpenAI
- Dans n8n : `Settings` → `Credentials` → `Add Credential` → `OpenAI API`

## Emails de test

### Test 1 — Candidature (doit déclencher le workflow complet)

```
Objet : Candidature pour un poste de développeur web
Corps  : Bonjour, je vous adresse ma candidature spontanée pour un poste 
         de développeur web au sein de votre entreprise. Vous trouverez 
         ci-joint mon CV et ma lettre de motivation.
         Cordialement, Jean Dupont
```

### Test 2 — Hors sujet (doit envoyer la réponse générique)

```
Objet : Question sur votre site web
Corps  : Bonjour, j'aimerais avoir des informations sur vos tarifs.
         Merci d'avance.
```

## Résultats attendus

Après envoi d'un email de candidature :

1. **Google Sheets** — nouvelle ligne ajoutée dans l'onglet Candidatures
2. **Gmail** — email de confirmation reçu par le candidat (< 1 minute)
3. **Discord** — notification dans le canal de l'équipe RH

## Auteur

TP n8n — Découverte de l'automatisation de workflows
