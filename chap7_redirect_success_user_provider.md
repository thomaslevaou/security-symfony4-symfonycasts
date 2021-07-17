# Redirecting on Success & the User Provider

On va à présent chercher à remplir le contenu de `onAuthenticationSuccess` : 
maintenant que notre utilisateur est connecté, que fait-on après sa connexion ?  
Plusieurs possibilités sont ouvertes : on peut ne rien faire, ou retourner un objet de réponse,
ou rediriger l'utilisateur connecté vers une autre page. Ici, on va choisir cette dernière option.  

Afin d'assurer proprement les redirections, on va injecter le service `RouterInterface` dans le
constructeur du `LoginFormAuthenticator`. On peut alors coder `onAuthenticationSuccess` de la 
manière suivante :
```PHP
public function onAuthenticationSuccess(Request $request, TokenInterface $token, $providerKey)
{
//        dd('Success');
    return new RedirectResponse($this->router->generate('app_homepage'));
}
```
`app_homepage` étant pour rappel le nom utilisé pour identifier la méthode `homepage()` de
`ArticleController`.  

En testant la page, on peut alors constater qu'une fois qu'on a entré un identifiant de la bdd
et un mdp quelconque, on est alors redirigé sur la page d'accueil en tant que 
`spacebar9@example.com` (c'est visible via l'identifiant affiché en bas du profiler).  
Les informations d'authentification sont stockées en session.  

Dans le fichier `security.yaml`, on peut y voir les données suivantes :
```yaml
app_user_provider:
  entity:
    class: App\Entity\User
    property: email
```

Elles sont là pour indiquer qu'au rechargement d'une page, l'**User Provider** peut
jeter un oeil au User en base (représenté par l'entité User) pour remettre à jour 
ses données si besoin. Par exemple, si on est connecté à deux endroits en même temps
et qu'un des deux comptes connectés met à jour l'identifiant, alors l'autre identifiant
connecté sera mis à jour directement au prochain rafraîchissement de la page aussi.  

Du coup intuitivement je trouve que la session perd de l'intérêt et que ça risque 
de causer des soucis de perf, mais je verrai ça plus tard.