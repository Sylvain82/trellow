Trellow

Un tableau Kanban collaboratif en temps réel, inspiré de Trello, tenu dans un seul fichier HTML. Il a été conçu pour un usage interne (« Trello BT JVA ») : pas de serveur à maintenir, pas de build, juste une page statique hébergée sur GitHub Pages et une base Firebase.

Projet indépendant, non affilié à Trello ni à Atlassian.

Aperçu des fonctionnalités
Tableau et colonnes
Colonnes et cartes déplaçables par glisser-déposer, à la souris comme au doigt (appui long sur mobile)
Couleur personnalisable par colonne
Limite WIP par colonne, avec jauge de remplissage et blocage du déplacement ou de la création quand la limite est atteinte
Tri par colonne : manuel, date de création, échéance, priorité, ordre alphabétique
Suppression d'une colonne avec confirmation (annulable)
Cartes
Titre, description (mise en forme légère : **gras**, *italique*, - listes), priorité, date limite, couverture colorée
Labels dynamiques, créés, renommés, recolorés et supprimés depuis l'interface
Cartes assignables à un ou plusieurs utilisateurs
Sous-tâches avec barre de progression, cochables directement depuis la carte
Commentaires horodatés avec auteur
Réactions par emoji (👍 ❤️ 🔥 ⚠️ 🎉 👀)
Récurrence (tous les N jours, semaines ou mois) : à la clôture d'une carte récurrente, la suivante est créée automatiquement
Duplication de carte
Historique des déplacements et temps de cycle par colonne
Vieillissement visuel : les cartes ouvertes qui n'ont pas bougé depuis plus de 7, 14 puis 30 jours changent d'apparence
Mise en évidence des cartes en retard ou à échéance sous 3 jours
Recherche et filtres
Recherche plein texte (titre, description, commentaires, sous-tâches) avec surlignage des résultats
Filtres par priorité, label, assigné et statut (ouvert, clos, en retard), option « Masquer clos »
Affichage des filtres actifs sous forme de pastilles
Collaboration et suivi
Synchronisation en temps réel entre tous les utilisateurs connectés
Indicateur de présence (qui est en ligne)
Journal d'activité (100 dernières actions)
Tableau de bord : totaux, cartes en retard, priorités hautes, répartition par colonne et par label, taux de complétion, derniers commentaires
Annulation (Ctrl+Z, 30 niveaux) pour les modifications du tableau
Bandeau hors ligne et reconnexion automatique
Pastille sur le favicon indiquant le nombre de cartes en retard
Confort
Thème clair / sombre (suit la préférence du système au premier lancement)
Affichage adapté aux mobiles
Mise en page dédiée à l'impression
Export JSON du tableau
Archives (affichage des cartes terminées)
Raccourcis clavier
Touche	Action
F	Ouvrir les filtres / la recherche
N	Nouvelle colonne
D	Tableau de bord
A	Journal d'activité
T	Changer de thème
Ctrl+Z	Annuler
?	Afficher l'aide des raccourcis
Échap	Fermer le panneau ou la fenêtre ouverte
Architecture
trellow/
├── index.html                  # Toute l'application (HTML + CSS + JS)
└── .github/workflows/
    └── deploy.yml              # Déploiement sur GitHub Pages
Front : HTML, CSS et JavaScript natifs (modules ES), sans framework ni étape de build.
Backend : Firebase 10.12 chargé depuis le CDN de Google : Authentication (e-mail / mot de passe) et Realtime Database.
Hébergement : GitHub Pages, alimenté par GitHub Actions.
Préférences locales (thème, tri des colonnes, dernier e-mail utilisé) : localStorage du navigateur.
Structure des données (Realtime Database)

Les écritures sont granulaires (par carte ou par colonne) pour limiter les conflits entre utilisateurs simultanés.

Chemin	Contenu
board/columns/<id>	titre, couleur, limite WIP, ordre
board/cards/<id>	carte complète (champs, labels, sous-tâches, commentaires, réactions, historique de cycle)
board/meta	informations de migration
labels/<id>	nom et couleur des labels
journal/<id>	entrées du journal d'activité
presence/<uid>	présence en ligne (effacée à la déconnexion)
users/<uid>	e-mail de l'utilisateur

Au premier lancement, une migration automatique convertit l'ancienne structure (boards/shared) vers la structure actuelle, si elle existe.

Installation
1. Créer le projet Firebase
Créez un projet dans la console Firebase.
Ajoutez une application Web pour récupérer la configuration.
Dans Authentication, activez le fournisseur E-mail / Mot de passe, puis créez les comptes de vos utilisateurs. L'application n'offre pas d'inscription : seuls les comptes créés depuis la console peuvent se connecter.
Dans Realtime Database, créez une base de données.
Dans Authentication > Paramètres > Domaines autorisés, ajoutez le domaine GitHub Pages (<utilisateur>.github.io).
2. Sécuriser la base

La configuration Firebase d'une application web est visible par quiconque ouvre la page : la protection des données repose donc sur les règles de sécurité de la base, pas sur le secret de la clé. À minima, réservez la lecture et l'écriture aux utilisateurs authentifiés :

json
{
  "rules": {
    ".read": "auth != null",
    ".write": "auth != null"
  }
}

Adaptez ces règles à votre contexte (validation des champs, restriction par e-mail, etc.).

3. Configurer les secrets GitHub

Dans Settings > Secrets and variables > Actions du dépôt, créez les quatre secrets suivants avec les valeurs de votre configuration Firebase :

Secret	Valeur
FIREBASE_API_KEY	apiKey
FIREBASE_AUTH_DOMAIN	authDomain
FIREBASE_DB_URL	databaseURL
FIREBASE_PROJECT_ID	projectId

Le fichier index.html du dépôt ne contient que des espaces réservés (%%FIREBASE_API_KEY%%, etc.) : aucune configuration n'est versionnée.

4. Activer GitHub Pages

Le workflow publie le site sur la branche gh-pages. Une fois le premier déploiement terminé, allez dans Settings > Pages et choisissez Deploy from a branch, branche gh-pages, dossier / (root).

Déploiement

Chaque push sur la branche main déclenche le workflow Deploy to GitHub Pages :

injection des secrets Firebase à la place des espaces réservés dans index.html ;
copie du fichier dans un dossier dist/ ;
publication de dist/ sur la branche gh-pages.
Développement local

Comme les valeurs Firebase ne sont pas dans le dépôt, travaillez sur une copie du fichier pour ne jamais committer vos clés :

bash
mkdir -p /tmp/trellow-dev
sed -e 's|%%FIREBASE_API_KEY%%|VOTRE_API_KEY|' \
    -e 's|%%FIREBASE_AUTH_DOMAIN%%|VOTRE_PROJET.firebaseapp.com|' \
    -e 's|%%FIREBASE_DB_URL%%|https://VOTRE_PROJET-default-rtdb.firebaseio.com|' \
    -e 's|%%FIREBASE_PROJECT_ID%%|VOTRE_PROJET|' \
    index.html > /tmp/trellow-dev/index.html

cd /tmp/trellow-dev && python3 -m http.server 8000

Ouvrez ensuite http://localhost:8000. Si l'initialisation de Firebase échoue, la page affiche le bandeau « Firebase non configuré ».

Personnalisation
Utilisateurs assignables : la liste est définie dans la constante HARDCODED_USERS de index.html (identifiant et nom affiché). Elle est indépendante des comptes Firebase.
Labels par défaut : créés au premier lancement si la base n'en contient aucun ; ils se gèrent ensuite depuis le bouton 🏷️.
Couleurs de couverture et de colonne : tableaux coverColors et COL_COLORS.
