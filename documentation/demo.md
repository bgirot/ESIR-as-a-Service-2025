# Scénario de Démonstration - ESIR-as-a-Service

Ce document sert de guide pour tester l'intégralité de mon infrastructure. Il détaille les étapes pour vérifier le fonctionnement du SSO (Single Sign-On) et les fonctionnalités clés de chaque micro-service.

## Démo rapide et non exhaustive :
![Gif Présentation](documentation/demo.gif)

## Identifiants de Test

Pour cette démonstration, j'ai pré-créé un utilisateur standard dans l'annuaire (Authelia) :

* Utilisateur : guest
* Mot de passe : welcome2u!
* Email : guest@komquest.cc (utilisé par certains services comme identifiant)

---

## 1. Le Point d'Entrée : Dashy

Objectif : Montrer le tableau de bord centralisé et la protection "Proxy" (non-OIDC).

1. Accéder à : https://dashboard.komquest.cc
2. Observation : Vous devriez être bloqué par une page de connexion Authelia (Protection Nginx).
3. Action : Connectez-vous avec `guest` / `welcome2u!`.
4. Résultat : Vous accédez au Dashboard Dashy. Tous les liens vers les autres services sont disponibles ici.
5. Testez les différents thèmes proposés par Dashy, ou personnalisez votre vue en cliquant sur les icônes en haut à droite.

---

## 2. Service de Notes : Memos

Objectif : Tester le SSO natif (OAuth2) et la création de contenu.

1. Depuis le Dashboard, cliquez sur "Memos"
2. Connexion :
   - Cliquez sur le bouton "Se connecter avec SSO" (ou "Connexion Authelia").
   - Comme vous êtes déjà connecté sur Dashy, la redirection doit être immédiate (pas de mot de passe redemandé).
3. Fonctionnalités à tester :
   - Écrire une note simple : "Test de démo Memos".
   - Cocher la case "Task" pour créer une liste de tâches.
4. Validation : La note apparaît instantanément dans la timeline.

---

## 3. Forge Git : Gitea

Objectif : Montrer l'intégration OIDC

1. Accéder à : https://git.komquest.cc
2. Connexion :
   - Cliquez sur "Connexion" en haut à droite.
   - Le compte `guest` est créé automatiquement s'il n'existe pas.
3. Fonctionnalités à tester :
   - créer des repositories

---

## 4. Synchronisation de Fichiers : Seafile

Objectif : Tester le mappage d'attributs complexe et l'upload.

1. Accéder à : https://files.komquest.cc
2. Connexion :
   - Pas de SSO car pas fonctionnel (vous pouvez tester mais arriverez sur une page d'erreur).
   - Connectez-vous avec le compte local `guest@komquest.cc` / `welcome2u!`.
3. Fonctionnalités à tester :
   - Créer et éditer des fichiers.

---

## 5. Galerie Photo : Photoview

Objectif : Tester le service

1. Accéder à : https://photos.komquest.cc
2. Connexion :
   - Connectez vous
3. Fonctionnalités à tester :
   - L'utilisateur arrive sur sa timeline (vide initialement).
   - Aller dans Settings > Users. Vous verrez que l'utilisateur guest a un rôle "User" standard.

---

## 6. Site Web / Blog : WordPress

Objectif : Tester l'intégration OIDC via un Plugin tiers.

1. Accéder à : https://blog.komquest.cc
2. Connexion :
   - Allez sur https://blog.komquest.cc/wp-admin.
   - Cliquez sur le bouton "Login with OpenID Connect".
3. Fonctionnalités à tester :
   - Créer un nouvel article ("Brouillon rapide").
   - Visiter le site public pour voir l'article.

---

## 7. Vaultwarden - Gestionnaire de Mots de Passe
Vaultwarden n'accepte pas l'authentification via OIDC/SSO. L'accès se fait donc via un compte local. Je n'ai pas pris le temps de configurer ce service, à chaque utilisateur de créer son propre compte.
1. Accéder à : https://vault.komquest.cc


## 8 Test de Sécurité (Déconnexion)

Objectif : Montrer que la déconnexion est globale.

1. Retournez sur Dashy.
2. Déconnectez-vous via le bouton en haut à droite.