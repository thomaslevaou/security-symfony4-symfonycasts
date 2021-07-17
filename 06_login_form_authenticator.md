# Login Form Authenticator

Dans notre `LoginFormAuhenticator`, la méthode `supports()` a pour rôle de retourner `true`
si la requête contient des informations d'authentification que le `LoginFormAuthenticator`
sait traiter, et de retourner `false` sinon. Si la méthode `supports()` retourne false,
alors Symfony n'appelle aucune autre méthode de l'authenticator et la requête continue
comme si l'authenticator n'avait rien fait.  

Pour le moment, le code de notre implémentation de `supports()` va être le suivant :
```PHP
$request->attributes->get('_route') === 'app_login' && $request->isMethod('POST')
```
Si la requête a pour route demandée `app_login` (c'est-à-dire la route `/login` de 
notre `SecurityController`) et que les données envoyées sont en POST, alors la 
méthode `supports()` va retourner `true`, et `false` sinon.

Si la méthode `supports()` retourne `true`, alors Symfony appelle immédiatement
la méthode `getCredentials()` de `LoginFormAuhenticator`. C'est vérifiable par dump
comme ci-dessous : 
```PHP
public function getCredentials(Request $request)
{
    dd($request->request->all());
}
```
Notons qu'on peut utiliser `dd()` au lieu de `dump();die;`.
Notons que si on veut voir des données envoyées par un POST, on doit utiliser la 
propriété `request` de la classe `Request` comme montré ci-dessus.

Lorsqu'on entre dans son navigateur l'adresse `http://localhost:8000/login`, on 
voir bien que `supports()` retourne false, et donc `getCredentials()` n'est pas appelé.  

On peut alors entrer un identifiant présent en base (comme `spacebar9@example.com`) et un 
mot de passe quelconque. La méthode `supports()` retourne alors true, ce qui active notre
`dump()` dans `getCredentials`. Les noms des paramètres ("email" et "password") proviennent
des attributs `name` des balise `input` dans la vue front.  

Le vrai rôle de `getCredentials()` va être de récupérer l'identifiant utilisateur et le 
mot de passe, parmi toutes les données présentes dans la request. D'où le fait que son 
code ici va être le suivant : 
```PHP
public function getCredentials(Request $request)
{
    return [
        'email' => $request->request->get('email'),
        'password' => $request->request->get('password')
    ];
}
```

Dès que `getCredentials` a fait son job, l'Authenticator appelle `getUser()` pour
récupérer l'identifiant utilisateur (passage à cette méthode vérifiable par dd).  
Pour récupérer l'identifiant utilisateur, on va faire appel au UserRepository, 
qu'on injecte par auto-wiring dans le constructeur de `LoginFormAuthenticator`.  

Ensuite, le code de `getUser` peut alors être le suivant :
```PHP
public function getUser($credentials, UserProviderInterface $userProvider)
{
//        dd($credentials);
    return $this->userRepository->findOneBy(['email' => $credentials['email']]);
}
```

Ainsi, notre méthode `getUser()` va retourner soit l'utilisateur trouvé en base, soit
`null`. Et si cette méthode retourne `null`, tout le process d'authentification va s'arrêter
et l'utilisateur sera informé de l'erreur.  

Si la méthode `getUser()` retourne un objet Utilisateur, alors Symfony va immédiatement 
appeler `getCredentials()` ensuite.  
On peut voir qu'un `dd($user);` dans cette dernière méthode affiche directement l'objet 
utilisateur dont l'e-mail correspond à celui trouvé en base. On peut en déduire que 
la classe `UserInterface` peut correspondre à un `User` tel que défini dans nos entités.  

Pour le moment, on va juste mettre un `return true;` dans `checkCredentials`. Ca paraît 
un peu bricolé ici, mais c'est vraiment ce qu'on mettrait dans le cas d'une authentification
par token. Si on met `return false;` dans `checkCredentials`, l'authentification échoue systématiquement
et l'utilisateur est informé d'une erreur "Invalid credentials".  

Une fois que le `checkCredentials` est passé avec succès, l'Authenticateur appelle `onAuthenticationSuccess()`
(vérifiable par dump).  

