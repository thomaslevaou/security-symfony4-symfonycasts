# CSRF Protection

Chaque page qui demande une action (par exemple sauvegarder un truc, ou se connecter)
doit être protégée par un **CSRF token** (jeton CSRF).  
Quand on utilise un formulaire géré par Symfony, la protection CSRF est incluse.  
Mais notre formulaire de connexion n'est pas géré par Symfony, donc on va ajouter
nous-mêmes la protection CSRF.  

On rappelle que la protection CSRF permet d'éviter que n'importe qui envoie
des données via ce formulaire. Un token est généré côté client à chaque affichage
de page tout en étant stocké côté serveur également. Lorsque le client envoie 
les données du formulaire, on compare le token envoyé côté client et celui côté 
serveur pour vérifier la bonne compatibilité entre les deux.

Pour ce faire on va procéder de la manière suivante :
1. On ajoute un `input type="hidden"` dans `login.html.twig`, entre le champ "mot
  de passe" et la case à cocher "Remember Me" : 
```HTML
<input type="hidden" name="_csrf_token" value="{{ csrf_token('authenticate') }}">
```

La chaîne de caractères en paramètre de `csrf_token` sert de nom identifiant ce token 
(ça pourrait être n'importe quelle chaîne de caractères).

2. Dans la méthode `getCredentials()` du `LoginFormAuthenticator`, on va ajouter une clé `'csrf_token'` dans `$credentials`: 
```PHP
public function getCredentials(Request $request)
    {
        $credentials = [
            'email' => $request->request->get('email'),
            'password' => $request->request->get('password'),
            'csrf_token' => $request->request->get('_csrf_token'),
        ];

        $request->getSession()->set(
            Security::LAST_USERNAME,
            $credentials['email']
        );

        return $credentials;
    }
```

3. Dans la méthode `getUser()`, on va vérifier ce token. Il vaut mieux le faire avant l'appel de vérification à 
l'utilisateur. Et pour vérifier le token, on va faire appel au Service `CsrfTokenManagerInterface`.  
Après avoir ajouté ce service en 3ème paramètre du constructeur de l'authenticator, on modifie le code
de `getUser()` de la manière suivante :
```PHP
public function getUser($credentials, UserProviderInterface $userProvider)
{
    $token = new CsrfToken('authenticate', $credentials['csrf_token']);
    if (!$this->csrfTokenManager->isTokenValid($token)) {
        throw new InvalidCsrfTokenException();
    }
    return $this->userRepository->findOneBy(['email' => $credentials['email']]);
}
```   


Du coup voilà si on supprime ce champ caché côté client dans le navigateur, il devient impossible de se connecter.  


CSRF veut dire Cross-Site Request Forgery, et un exemple de conséquence de son non-utilsiation est décrit [dans cette
discussion StackOverflow](https://stackoverflow.com/questions/5207160/what-is-a-csrf-token-what-is-its-importance-and-how-does-it-work/33829607#33829607)

Le but de cette protection est d'éviter que si un utilisateur utilise deux sites
en même temps sur un même navigateur, qu'un des sites utilise l'autre site ouvert
pour envoyer des données en usurpant l'identité de l'utilisateur. Le csrf token 
ne peut pas être deviné en amont par un site malintentionné, alors que d'autres données
GET peuvent être devinées en amont et utilisées dans le navigateur de la cible avec
ses cookies. 

L'ajout d'une protection CSRF est recommandée pour tous les formulaires.