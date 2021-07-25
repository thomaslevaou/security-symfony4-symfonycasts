# Target Path: Redirecting an Anonymous User

On rappelle que la page d'erreur 403 "Access Denied" en dev avec les infos tech
est différente de celle de la prod. Si on tape sur Google "Symfony error pages",
on peut voir comment améliorer le rendu visuel de la page d'erreur 403 en prod.  
En tout cas, on peut avoir une page différente par code de status.  

Si on se déconnecte du site (`/logout`) et qu'on retourne sur `admin/comment`,
on est redirigé sur le formulaire de connexion.  
Ce fait provient de la classe `AbstractFormLoginAuthenticator` dont hérite
notre `LoginFormAuthenticator`. En effet la classe `AbstractFormLoginAuthenticator`
contient une méthode appelée `start()` qui détermine **l'Entry Point**, c'est-à-dire
ce que le site doit faire quand un utilisateur anonyme essaie d'accéder à des pages.  

Une fois connecté, l'utilisateur est redirigé sur la page d'accueil au lieu 
d'être redirigé sur la page à laquelle il souhaitait accéder (/admin/comment).  
Ceci est du au fait que dans notre `LoginFormAuthenticator`, on a écrit que 
la méthode `onAuthenticationSuccess` redirigeait toujours vers la page d'accueil.  

Pour récupérer la dernière adresse sur laquelle l'utilisateur a tenté de se connecter,
on va un peu modifier le code de cette méthode en utilisant le `TargetPathTrait` qui 
permet notamment de récupérer le dernier lien visité avant la redirection vers la connexion :
```PHP
public function onAuthenticationSuccess(Request $request, TokenInterface $token, $providerKey)
{
    if ($targetPath = $this->getTargetPath($request->getSession(), $providerKey)) {
        return new RedirectResponse($targetPath);
    }
    return new RedirectResponse($this->router->generate('app_homepage'));
}
```

Le `$providerKey` n'est que le nom du firewall, qui est passé automatiquement via le troisième
paramètre `$providerKey` de la méthode `onAuthenticationSuccess`.  
Le `if ($targetPath = $this->getTargetPath($request->getSession(), $providerKey))` est là
pour dire que si le dernier lien auquel a tenté d'accéder l'utilisateur a été trouvé et existe,
on le redirige dessus. Sinon (par ex. si c'est la première fois qu'il vient sur le site), on le redirige
sur la homepage.