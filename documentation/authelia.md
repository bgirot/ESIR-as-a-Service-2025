# Authelia - Serveur d'Authentification & SSO

## Description
Authelia est un serveur d'authentification open-source. Il fournit un portail de connexion unique (SSO), gère l'authentification à deux facteurs (2FA) et agit comme fournisseur d'identité OpenID Connect (OIDC) pour l'ensemble de mes applications.

## Pourquoi j'ai choisi cette solution
J'ai retenu Authelia pour plusieurs raisons stratégiques par rapport à des solutions comme Keycloak ou Authentik :

1. Légèreté : Écrit en Go, il consomme très peu de ressources (RAM/CPU), ce qui est crucial pour mon déploiement sur une VM unique.
2. Configuration déclarative : Tout se configure via un fichier YAML unique (configuration.yml).
3. Intégration Nginx : Il s'intègre nativement avec le module auth_request de Nginx, permettant de protéger n'importe quelle URL (comme mon Dashboard) sans modifier l'application cible.

## Ma Configuration

Les fonctionnalités que j'ai déployées :
* Single Sign-On (SSO) : Une seule connexion permet d'accéder à tous les services.
* Backend Utilisateurs : J'utilise un fichier plat (users_database.yml) pour gérer les utilisateurs, évitant la complexité d'un annuaire LDAP pour une petite structure.
* Fournisseur OIDC : Configuration des clients pour permettre la connexion fédérée.

Extrait de mon fichier Docker Compose :

```yaml
  authelia:
    container_name: esir_authelia
    image: authelia/authelia
    volumes:
      - ./authelia/config:/config
    environment:
      - TZ=Europe/Paris
    networks:
      - esir-network
    expose:
      - 9091
    restart: unless-stopped
    healthcheck:
      disable: true
```

### Configuration des Clients OIDC (Extrait de configuration.yml)

C'est ici que j'ai défini les règles pour chaque service connecté. Sur la VM, on peut retrouver les secrets. Cependant, pour des raisons de sécurité, le code présent dans le repo sont des fichiers d'exemple, les fichiers exemple étant de le `.gitignore`.

```yaml
identity_providers:
  oidc:
    clients:
      - client_id: gitea
        client_name: Gitea
        client_secret: '$pbkdf2-sha512$...' # Hash du secret
        redirect_uris:
          - [https://git.komquest.cc/user/oauth2/authelia/callback](https://git.komquest.cc/user/oauth2/authelia/callback)
        scopes: [openid, profile, email, groups]
        
      - client_id: memos
        client_name: Memos Notes
        token_endpoint_auth_method: client_secret_post # Spécifique pour Memos
        redirect_uris:
          - [https://memos.komquest.cc/auth/callback](https://memos.komquest.cc/auth/callback)
          
      - client_id: seafile
        client_name: Seafile
        redirect_uris:
          - [https://files.komquest.cc/oauth/callback/](https://files.komquest.cc/oauth/callback/)
        # ... autres clients (Photoview, WordPress)
```

## Points d'attention


- Redirections strictes : Le protocole OIDC est très strict sur les URL de redirection (redirect_uris). Une simple erreur de slash final (ex: /callback vs /callback/) suffisait à briser l'authentification, notamment pour Seafile.
- Hashage des secrets : Les secrets des clients ne sont pas stockés en clair dans Authelia. J'ai dû utiliser la commande docker run authelia/authelia authelia crypto hash generate pour sécuriser chaque secret client.

## Accès

* Portail d'authentification : https://auth.komquest.cc
* Accès réseau interne : http://esir_authelia:9091
* Endpoints OIDC :
    * Authorization : /api/oidc/authorization
    * Token : /api/oidc/token
    * Userinfo : /api/oidc/userinfo