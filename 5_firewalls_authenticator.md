# Firewalls & Authenticator


Contrairement à ce qu'on pourrait penser a priori, nous n'allons pas écrire 
la logique de vérification des identifiants du formulaire de connexion dans
`SecurityController`.  

En Symfony, chaque requête fait appel à un **Authenticator**, aussi appelé
**Authentication Listener**. Le but de celui-ci est de repérer des données
d'authentification qui pourraient passer dans une requête (des e-mails / 
mots de passe envoyées, ou un token API, ...), et d'authentifier avec ces 
données si nécessaire. Nous allons écrire cet Authenticator.  

Avant cela, notons que la sécurité Symfony se compose de deux parties :
- L'Authentification, pour montrer qui on est et le prouver;
- L'Autorisation, qui détermine si on a accès ou pas à un truc une fois
  authentifié. 

Et dans tout ça, on a des **Firewalls** pour aider à l'authentification.  
Même si on a plusieurs manières de se connecter (ex. par API et login/mdp
en même temps), il vaut mieux ne garder qu'un seul firewall.  

Dans le fichier `config/packages/security.yaml`, la partie `firewalls:` se 
présente de la manière suivante :
```YAML
firewalls:
        dev:
            pattern: ^/(_(profiler|wdt)|css|images|js)/
            security: false
        main:
            anonymous: lazy
            provider: app_user_provider
```
La partie `dev` est là pour ne pas bloquer l'accès au profiler (entre autres)
en dev quelque soit notre sécurité.  
Mais pour le reste, nous n'utilisons qu'un seul "vrai" firewall : celui
de la partie `main`. Sans clé `pattern:`, ce sera le firewall utilisé partout 
sauf dans les URL concernées par `dev`.
Le `anonymous: lazy` dans `main` est là pour dire que les requêtes "anonymes" 
peuvent passer le firewall, de sorte que les pages publiques puissent être
accessibles sans login. En fait, il vaut mieux le garder même si on veut que 
tout le site ne soit accessible qu'après un login. En effet on verra comment 
on peut restreindre les accès sur tout le site via `access_control` (plus bas
dans le `security.yaml`).  

Avant de revenir plus en détail sur l'utilisation concrète du firewall, on va 
créer notre premier Authenticator. Pour ce faire, on va lancer la commande
`./bin/console make:auth`.  
En réponse aux questions posées par la console, on va créer un
`Empty authenticator` (mais faire un `Login form authenticator` peut bien 
marcher aussi la prochaine fois), et on va l'appeler `LoginFormAuthenticator`.  

La commande a alors créée une nouvelle classe `LoginFormAuthenticator`, visible
dans `src/Security`, avec une méthode pour chaque étape du processus
d'authentification.  
Mais pour avoir moins de taff dans notre cas, au lieu de faire hériter notre
classe de `AbstractGuardAuthenticator`, on va la faire hériter de 
`AbstractFormLoginAuthenticator`. Ainsi, nous pouvons supprimer les 
méthodes `onAuthenticationFailure`, `start` et `supportsRememberMe`, qui vont
être gérées automatiquement par cette classe parente.  
On les utilisera quand on fera une connexion par API. En revanche ici, on va 
ajouter a méthode `getLoginUrl`, qu'on va implémenter.  

Notons que dans le fichier `security`, notre authenticator a été activé
via l'ajout des balises Yaml ci-dessous: 

```YAML
guard:
  authenticators:
    - App\Security\LoginFormAuthenticator
```

Tout le système d'authenticator provient de la partie du composant Security
appelé **guard**, d'où le nom de cette nouvelle balise dans security.yaml.  

Et du coup maintenant à chaque requête, Symfony va appeler la méthode `supports()`
de notre Authenticator (vérifiable en mettant un `die()` dedans).