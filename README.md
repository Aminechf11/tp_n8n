# TP Découverte n8n – Assistant Support Client Automatique

## Présentation

Ce projet a pour objectif d'automatiser le traitement des demandes de support reçues par email grâce à n8n.

Lorsqu'un utilisateur envoie un email, le workflow :

* analyse le contenu du message ;
* classe automatiquement la demande ;
* enregistre les informations dans Google Sheets ;
* crée une tâche dans Trello ;
* notifie l'équipe sur Discord ;
* envoie un accusé de réception automatique.

## Technologies utilisées

* n8n
* Gmail
* OpenAI
* Google Sheets
* Trello
* Discord

## Workflow

1. Réception d'un email
2. Analyse du contenu avec OpenAI
3. Classification de la demande
4. Sauvegarde dans Google Sheets
5. Création d'un ticket Trello
6. Notification Discord
7. Réponse automatique à l'expéditeur

## Exemple

Email reçu :

Objet : Mot de passe oublié

Bonjour,

Je ne parviens plus à accéder à mon compte.

Résultat :

* catégorie = Authentification
* ligne ajoutée dans Google Sheets
* ticket créé dans Trello
* notification Discord envoyée
* email de confirmation envoyé

## Démonstration

1. Envoi d'un email de test
2. Exécution automatique du workflow
3. Vérification des actions réalisées

## Auteur

Nom Prénom
B3 Informatique
