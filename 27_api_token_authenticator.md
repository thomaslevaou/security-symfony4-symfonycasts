# API Token Authenticator

Afin de tester notre API proprement, on va utiliser Postman. Ce qui fera office
pour moi de mini-tuto pour utiliser Postman.  

Une fois Postman installé, on peut créer une nouvelle requête GET sur 
`http://localhost:8000/api/account`.  

Par convention, il vaut mieux placer le token dans un Header de la requête.  
Le mieux pour ce faire dans Postman est d'aller dans l'onglet "Authorization", 
puis sélectionner "Bearer Token" dans la liste déroulante. Dans le champ "Token"
qui s'affiche alors, on peut prendre un des token stockés en base (via un simple
`./bin/console doctrine:query:sql 'select * from api_token'`) et le coller dans
ce champ.  
Une fois cela fait, on peut voir dans l'onglet "Headers" que le token a bien été
ajouté sous la clé "Authorization" (dans les champs cachés), avec une valeur
"Bearer [nom du token]".  

La clé "Authorization" n'est là que par convention : le token pourrait être 
envoyé sous n'importe quelle clé (y compris "I-LIKE-DINOSAURS") et ça passerait
quand même. Idem pour le "Bearer" avant la valeur du token, qui est totalement
facultative. Le terme Bearer est une convention pour dire "N'importe qui qui 
possède ce token peut s'authentifier sans avoir besoin de fournir un autre
moyen d'authentification".  

On peut alors à présent remplir le contenu des méthodes de l'
`ApiTokenAuthenticator`.  
La méthode `supports` va vérifier si le token est bien
présent dans les headers sous la clé "Authorization", et si la chaîne de 
caractères "Bearer " forme bien le début de la valeur associée :
```PHP
public function supports(Request $request)
{
    return $request->headers->has('Authorization')
        && 0 === strpos($request->headers->get('Authorization'), 'Bearer ');
}
```

Dans `getCredentials()`, on va chercher à retourner le token ayant servi à 
identifier l'utilisateur (sans le "Bearer ") :
```PHP
public function getCredentials(Request $request)
{
    $authorizationHeader = $request->headers->get('Authorization');

    // skip beyond "Bearer "
    return substr($authorizationHeader, 7);
}
```

Si on place alors un `dump($credentials);die;` dans `getUser()`, on peut alors 
tester la route dans Postman : une fois qu'on a cliqué sur "Send", on peut bien
voir le retour du dump dans l'onglet "Preview" de la réponse.  

Pour coder correctement `getUser()`, on va retrouver l'ApiTokenEntity associée 
au token envoyé, puis retrouver l'User dont un des token correspond à cette 
ApiTokenEntity.  

On commence donc par préciser dans le constructeur de `ApiTokenAuthenticator` 
l'injection de nos entités :
```PHP
private $apiTokenRepo;

public function __construct(ApiTokenRepository $apiTokenRepo)
{
    $this->apiTokenRepo = $apiTokenRepo;
}
```

On code alors `getUser()` de la manière suivante :
```PHP
    public function getUser($credentials, UserProviderInterface $userProvider)
    {
//        dump($credentials);die;
        $token = $this->apiTokenRepo->findOneBy([
            'token' => $credentials
        ]);

        if (!$token) {
            return null;
        }

        return $token->getUser();
    }
```

Si on fait un dd() dans `checkCredentials()`, un appel Postman montre bien qu'on
est arrivé à cette étape.