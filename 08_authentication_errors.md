# Authentication Errors

Avec tout le code écrit jusqu'à maintenant, si on se connecte avec une
adresse e-mail non existante en bdd, alors le code retourne une erreur
"Cannot redirect to an empty URL".  
En fait c'est parce que dans le code de `AbstractFormLoginAuthenticator`, si 
une erreur se produit lors de l'authentification, le code appelle la méthode
`getLoginUrl` : si l'authentification a échoué, l'appli redirige l'utilisateur
vers le formulaire de connexion. Et du coup comme notre méthode `getLoginUrl()`
n'était pas remplie, on s'est pris une erreur.  

Une fois cette méthode remplie avec une simple redirection vers la page de login :
```PHP
public function getLoginUrl()
{
    return $this->router->generate('app_login');
}
```

On peut alors vérifier qu'une erreur lors de la connexion redirige l'utilisateur
sur la page de connexion avec un message d'erreur, vraisemblablement affichée 
par le code ci-dessous dans le formulaire `login.html.twig` :
```HTML
{% if error %}
    <div class="alert alert-danger">{{ error.messageKey|trans(error.messageData, 'security') }}</div>
{% endif %}
```
Une erreur aussi précise (dans le tuto, ils ont "Username could not be found") 
a été obtenue grâce au fait que l'Authenticator peut 
savoir quand l'authentification a échoué : ici parce qu'il n'a pas reçu d'utilisateur
de `getUser()`.  

Si on met `false` dans `checkCredentials()`, on peut aussi remarquer qu'on a une erreur
"Invalid Credentials" même en mettant une adresse qui existe en base.  

On rappelle que la dernière erreur est obtenue dans `SecurityController` via un appel à 
`getLastAuthenticationError()` :
```PHP
$error = $authenticationUtils->getLastAuthenticationError();
```

Lorsqu'une authentification échoue, la méthode `onAuthenticationFailure()` de `AbstractFormLoginAuthenticator`
est appelée :
```PHP
public function onAuthenticationFailure(Request $request, AuthenticationException $exception)
{
    if ($request->hasSession()) {
        $request->getSession()->set(Security::AUTHENTICATION_ERROR, $exception);
    }

    $url = $this->getLoginUrl();

    return new RedirectResponse($url);
}
```

Le deuxième paramètre de cette méthode est une `AuthenticationException` : à chaque erreur dans 
une authentification, une exception de ce type est levée. Cette exception est stockée en session,
à une clé correspondant à la constante statique `Security::AUTHENTICATION_ERROR`.  

Puis dans le code de `getLastAuthenticationError()`, on peut voir que la valeur à cette clé de session 
est récupérée pour être affichée sur l'interface.  

On souhaite à présent conserver l'e-mail entré dans le champ "Email address" même après une erreur 
d'authentification (au lieu de tout supprimer lors d'une erreur de connexion comme c'est le cas en 
ce moment).  
En fait on peut déjà voir dans `SecurityController` que le dernier username entré est déjà stocké dans `$lastUsername` :
```PHP
$lastUsername = $authenticationUtils->getLastUsername();

return $this->render('security/index.html.twig', [
    'last_username' => $lastUsername,
    'error' => $error
]);
```

Il suffit donc juste de l'afficher dans le template Twig :
```HTML
<input type="email" value="{{ last_username }}" name="email" id="inputEmail" class="form-control" placeholder="Email address" required autofocus>
```

La méthode `getLastUsername()` fonctionnant d'une manière similaire à `getLastAuthenticationError()`, on doit
stocker le dernier username entré en session dans la méthode `getCredentials()` de `LoginFormAuthenticator` :
```PHP
public function getCredentials(Request $request)
{
//        dump($request->request->all());die;
        $credentials = [
            'email' => $request->request->get('email'),
            'password' => $request->request->get('password')
        ];

        $request->getSession()->set(
            Security::LAST_USERNAME,
            $credentials['email']
        );

        return $credentials;
}
```

Et le tour est joué (je n'ai eu que des erreurs "Invalid Credentials" dans mon cas au lieu d'avoir 
"Username could not be found" dans certains cas décrits par le tuto, mais pour le moment je considère 
que ce n'est pas important).