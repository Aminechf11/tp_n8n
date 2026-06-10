# Guide de configuration détaillé

## 1. Installer n8n

### Option A — npx (recommandé pour tester)

```bash
npx n8n
```

Accéder à : `http://localhost:5678`

### Option B — Docker

```bash
docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  n8nio/n8n
```

### Option C — npm global

```bash
npm install -g n8n
n8n start
```

---

## 2. Configurer Gmail OAuth2

### Créer un projet Google Cloud

1. Aller sur [console.cloud.google.com](https://console.cloud.google.com)
2. Créer un nouveau projet (ex: `n8n-assistant-rh`)
3. Activer les APIs suivantes :
   - **Gmail API**
   - **Google Sheets API**

### Créer les identifiants OAuth2

1. `APIs & Services` → `Credentials` → `Create Credentials` → `OAuth 2.0 Client IDs`
2. Type : **Web application**
3. Nom : `n8n Local`
4. Authorized redirect URIs : `http://localhost:5678/rest/oauth2-credential/callback`
5. Copier le **Client ID** et le **Client Secret**

### Configurer dans n8n

1. `Settings` → `Credentials` → `Add Credential`
2. Rechercher `Gmail OAuth2 API`
3. Coller le Client ID et Client Secret
4. Cliquer **Connect my account** et autoriser l'accès

---

## 3. Configurer Google Sheets

### Créer le Google Sheets

1. Aller sur [sheets.google.com](https://sheets.google.com)
2. Créer un nouveau fichier
3. Renommer l'onglet `Feuille 1` en `Candidatures`
4. Ajouter les en-têtes en ligne 1 :

```
A1: Date
B1: Nom
C1: Email
D1: Sujet
E1: Resume
F1: Mots cles
G1: Statut
H1: Email envoye
```

### Récupérer l'ID du Spreadsheet

L'ID se trouve dans l'URL :
```
https://docs.google.com/spreadsheets/d/[SPREADSHEET_ID]/edit
```

### Configurer dans n8n

1. Ouvrir le nœud `Enregistrer dans Sheets`
2. Dans `Document ID`, coller l'ID du Spreadsheet
3. Dans `Sheet Name`, vérifier que `Candidatures` est bien sélectionné
4. Credential : sélectionner le compte Google configuré à l'étape 2

---

## 4. Configurer le webhook Discord

### Créer le webhook

1. Dans Discord, aller sur votre serveur
2. `Paramètres du serveur` → `Intégrations` → `Webhooks`
3. Cliquer `Nouveau webhook`
4. Nom : `Assistant RH n8n`
5. Choisir le canal où poster les notifications
6. Copier l'**URL du webhook**

### Configurer dans n8n

1. Ouvrir le nœud `Notifier Discord`
2. Dans `Webhook URI`, coller l'URL du webhook Discord
3. Aucun credential n8n nécessaire pour les webhooks Discord

---

## 5. Importer et configurer le workflow

### Importer

1. Dans n8n, cliquer `+` pour créer un nouveau workflow
2. Menu `⋮` (en haut à droite) → `Import from file`
3. Sélectionner `workflows/assistant-candidatures.json`

### Connecter les credentials

Chaque nœud avec une icône d'avertissement nécessite une configuration :

| Nœud | Credential nécessaire |
|------|-----------------------|
| `Nouveau Email` | Gmail OAuth2 |
| `Confirmation Candidat` | Gmail OAuth2 |
| `Reponse Generique` | Gmail OAuth2 |
| `Enregistrer dans Sheets` | Google Sheets OAuth2 |

Pour chaque nœud :
1. Cliquer sur le nœud
2. Dans le menu déroulant `Credential`, sélectionner le compte configuré

### Activer le workflow

Basculer le toggle **Inactive → Active** en haut à droite.

---

## 6. Tester le workflow

### Test manuel depuis n8n

1. Cliquer sur `Execute Workflow` (bouton ▶)
2. Envoyer un email de test à l'adresse Gmail configurée
3. Observer l'exécution en temps réel dans n8n

### Vérifier le résultat

- **Google Sheets** : une nouvelle ligne doit apparaître dans `Candidatures`
- **Gmail** : l'adresse expéditrice doit recevoir l'email de confirmation
- **Discord** : le canal doit afficher la notification

---

## Dépannage

### Le trigger ne détecte pas les emails

- Vérifier que le workflow est **Active** (pas juste Execute Workflow)
- S'assurer que les emails sont **non lus** dans la boîte Gmail
- Attendre jusqu'à 1 minute (le polling se fait chaque minute)

### Erreur d'authentification Gmail

- Revérifier que l'API Gmail est bien activée dans Google Cloud
- Régénérer les credentials OAuth2 si nécessaire
- S'assurer que l'URI de redirection est exactement `http://localhost:5678/rest/oauth2-credential/callback`

### Google Sheets retourne une erreur

- Vérifier que l'onglet s'appelle exactement `Candidatures` (sensible à la casse)
- S'assurer que les en-têtes correspondent exactement aux noms configurés dans le nœud
- Vérifier que l'API Google Sheets est activée dans Google Cloud

### Discord ne reçoit pas les notifications

- Tester l'URL du webhook directement avec curl :
  ```bash
  curl -X POST -H "Content-Type: application/json" \
    -d '{"content": "Test webhook n8n"}' \
    VOTRE_WEBHOOK_URL
  ```
