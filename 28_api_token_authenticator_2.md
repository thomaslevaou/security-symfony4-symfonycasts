# API Token Authenticator Part 2!

Si on entre dans l'appel Postman un token qui n'existe pas en base, Postman 
retourne dans sa réponse le formulaire de connexion Web au format HTML.  
En effet, comme la méthode `onAuthentificationFailure` n'est pas remplie et 
que `getUser()` n'a pas retourné d'utilisateur, l'`ApiTokenAuthenticator` n'a 
identifié personne, et donc le `AccountController` ne reconnaît pas de compte. 
Du coup Symfony a redirigé l'utilisateur sur l'entry point.

Pour éviter d'avoir un résultat aussi inattendu, on va donc remplir la méthode 
`onAuthenticationFailure` de notre `ApiTokenAuthenticator` de la manière
suivante : 
```PHP
public function onAuthenticationFailure(Request $request, AuthenticationException $exception)
{
    return new JsonResponse([
       'message' => $exception->getMessageKey()
    ], 401);
}
```
Notons qu'on utilise l'instance d'`AuthenticationException` passée en paramètre, 
qui retourne directement une erreur en cas d'échec d'authentification, qu'on peut
appliquer directement dans notre méthode ici. Ceci corrige le retour inattendu
que nous avions dans Postman.  

Notons cependant que la traduction que nous avions mise en place côté formulaire
n'est pas prise en compte ici (car la traduction est gérée dans la partie Twig).  

Donc ici, comme annoncé dans le chapitre 9, on va faire appel à la classe
d'exception `CustomUserMessageAuthenticationException` dans la méthode
`getUser()` de notre `ApiTokenAuthenticator` :
```PHP
        if (!$token) {
            throw new CustomUserMessageAuthenticationException('Invalid API Token');
        }
```

Et c'est bien cette exception qui sera considérée comme le deuxième 
paramètre de `onAuthenticationFailure()`.  

Profitons de cette situation pour également gérer les situations
où le token a expiré. Dans l'entité `ApiToken`, on peut 
ajouter une nouvelle méthode `isExpired()` avec le code suivant :
```PHP
    public function isExpired(): bool
    {
        return $this->getExpiresAt() <= new \DateTime();
    }
```

On peut alors directement utiliser cette méthode dans 
`ApiTokenAuthenticator`, en-dessous de la vérification "Invalid
API Token" que nous venons d'ajouter :
```PHP
        if (!$token) {
            throw new CustomUserMessageAuthenticationException('Invalid API Token');
        }

        if ($token->isExpired()) {
            throw new CustomUserMessageAuthenticationException('Token Expired');
        }
```

Notons qu'on aurait pu mettre ces vérifications dans `checkCredentials()`, 
mais c'était un poil plus simple de le faire dans `getUser()` de part le 
fait qu'on avait directement accès à notre objet `$token`.  

On peut se permettre de mettre un simple `return true;` dans `checkCredentials()` 
à présent :
```PHP
public function checkCredentials($credentials, UserInterface $user)
{
    return true;
}
```

Dans `onAuthenticationSuccess`, on peut ne rien mettre, car chaque appel
de route doit retourner directement la route appelée (on n'a pas de page 
par défaut ou page précédente à mettre comme sur une IHM) :
```PHP
public function onAuthenticationSuccess(Request $request, TokenInterface $token, $providerKey)
{
    // allow the request to continue
}
```

La méthode `start()` de l'`ApiTokenAuthenticator` ne sera jamais 
appelée car notre entry point est celui du `LoginFormAuthenticator`,
on peut la laisser vide telle quelle.  

En API, le remember me n'est pas gérée, la méthode `supportsRememberMe`
peut donc retourner systématiquement `false` :
```PHP
public function supportsRememberMe()
{
    return false;
}
```

Ce qui termine à présent notre `ApiTokenAuthenticator`, et nous permet
une connexion à l'API via Postman. L'appel à `/api/Account` retourne
comme prévu les données du groupe "main" de l'utilisateur.  

Notons que notre système d'authentification rend `/api/account` 
accessible par API ou id/mot de passe sans que le code d'AccountController
ait besoin de savoir quel type d'authentification est utilisé.
