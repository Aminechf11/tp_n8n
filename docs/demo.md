# Script de démonstration

## Présentation du cas d'usage (1 min)

> "Nous avons automatisé la gestion des candidatures par email pour une petite équipe RH. Chaque email reçu est analysé automatiquement : si c'est une candidature, elle est enregistrée dans un tableau de suivi, le candidat reçoit une confirmation, et l'équipe RH est notifiée sur Discord — sans aucune intervention manuelle."

---

## Démo 1 — Candidature valide (scénario principal)

### Email à envoyer

Depuis une adresse email différente de celle configurée dans n8n, envoyer :

**Destinataire :** l'adresse Gmail connectée à n8n

**Objet :**
```
Candidature pour un poste de développeur web full-stack
```

**Corps :**
```
Bonjour,

Je me permets de vous adresser ma candidature spontanée pour un poste 
de développeur web au sein de votre équipe.

Diplômé en informatique (Bac+3), j'ai 2 ans d'expérience en React et 
Node.js. Mon CV et ma lettre de motivation sont disponibles sur demande.

Dans l'attente de votre retour,
Cordialement,

Marie Martin
marie.martin@exemple.fr
```

### Ce qui doit se passer (dans l'ordre)

1. **n8n détecte l'email** (~1 minute)
   - Le nœud `Nouveau Email` s'allume en vert
   
2. **Classification** — le Code node détecte : `candidature`, `poste`, `cv`
   - `estCandidature = true`
   - `scoreDetection = 3`
   
3. **IF node** — branche TRUE activée

4. **Google Sheets** — nouvelle ligne ajoutée :
   ```
   Date         | Nom          | Email                   | Sujet               | Statut
   10/06/2026   | Marie Martin | marie.martin@exemple.fr | Candidature pour... | Nouveau
   ```

5. **Gmail** — Marie reçoit :
   > ✅ Candidature bien reçue
   > Bonjour Marie Martin, nous avons bien reçu votre candidature concernant : **Candidature pour un poste de développeur web full-stack**...

6. **Discord** — notification dans le canal RH :
   > 📬 **Nouvelle candidature reçue !**
   > 👤 **Candidat :** Marie Martin
   > ✉️ marie.martin@exemple.fr
   > 📝 Candidature pour un poste de développeur web...
   > ✅ Enregistré dans Google Sheets | ✅ Email envoyé

---

## Démo 2 — Email hors sujet (branche false)

**Objet :**
```
Question sur vos tarifs de formation
```

**Corps :**
```
Bonjour, j'aimerais avoir des informations sur vos formations disponibles.
Merci d'avance.
```

### Ce qui doit se passer

1. Classification : aucun mot-clé détecté → `estCandidature = false`
2. **IF node** — branche FALSE activée
3. **Gmail** — l'expéditeur reçoit la réponse générique :
   > 📩 Message bien reçu — Notre équipe le traitera dans les meilleurs délais.
4. **Google Sheets et Discord** — non déclenchés (email ignoré pour le suivi RH)

---

## Démo 3 — Version IA (si configurée)

Importer `workflows/assistant-candidatures-ia.json` et utiliser le même email de test.

**Différences observables :**
- GPT-4o-mini analyse le contenu en langage naturel
- La réponse envoyée au candidat est **personnalisée** selon le contenu exact de l'email
- Google Sheets contient également : **Type de poste** et **Compétences clés**
- Fonctionne même si les mots-clés ne sont pas présents (l'IA comprend le contexte)

**Exemple de réponse générée par l'IA :**
> Bonjour Marie, votre candidature pour un poste de développeur web full-stack a bien été reçue. Votre profil en React et Node.js est particulièrement intéressant pour nos projets en cours. Notre équipe technique l'examinera dans les 5 à 7 jours ouvrables.

---

## Points à souligner pendant la démo

1. **Zéro code serveur** — tout est configuré visuellement dans n8n
2. **Extensibilité** — on peut facilement ajouter : Notion, Trello, Slack, Google Calendar...
3. **Routing conditionnel** — le nœud IF permet de gérer plusieurs cas d'usage
4. **Version IA** — GPT-4o-mini permet des réponses contextualisées sans règles manuelles
5. **Observabilité** — n8n garde un historique de chaque exécution avec les données traitées

---

## Questions fréquentes

**Q : Que se passe-t-il si le workflow est désactivé ?**
> Les emails s'accumulent dans la boîte Gmail. Quand le workflow est réactivé, n8n ne traite que les emails non lus depuis la dernière exécution.

**Q : Peut-on traiter les pièces jointes (CV PDF) ?**
> Oui. Le nœud Gmail peut extraire les pièces jointes. On pourrait les sauvegarder dans Google Drive et ajouter le lien dans le Sheets.

**Q : Peut-on envoyer les emails depuis une adresse différente ?**
> Oui, si Gmail est configuré avec une adresse d'envoi alias, ou en utilisant un nœud SMTP.

**Q : Comment éviter de traiter le même email deux fois ?**
> Le Gmail Trigger filtre automatiquement les emails non lus. Après traitement, l'email peut être marqué comme lu via un nœud Gmail supplémentaire.
