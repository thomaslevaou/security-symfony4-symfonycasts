# Serializer & API Endpoint

On souhaite à présent que les utilisateurs puissent se connecter par 
Endpoint API. Ce qui implique de faire de notre côté un endpoint API
propre.  

Pour ce faire, on crée une nouvelle fonction dans notre `AccountController`,
appelée `accountApi()` qui va retourner les informations de l'utilisateur 
connecté, au format JSON. Son code va être le suivant :
```PHP
/**
 * @Route("/api/account", name="api_account")
 */
public function accountApi()
{
    $user = $this->getUser();
    return $this->json($user);
}
```

Si on l'exécute dans le navigateur (en s'étant préalablement connecté)
en entrant `http://localhost:8000/api/account`, on se retrouve avec un
JSON vide. En effet, comme le Service Serializer n'est pas installé,
la fonction `json()` ne fait qu'appliquer un `JsonResponse()`, lui-même
ne faisant en gros qu'un `json_encode()` ici. Le problème c'est que 
sur un objet PHP, la fonction `json_encode()` ne convertit en JSON que les 
propriétés publiques, là où toutes les propriétés de notre entité User
sont privées.  

Pour remédier à ce problème, on va installer le Service Serializer, et ce
via la commande `composer require serializer`, ce qui nous permet d'avoir
instantanément le retour json en rechargeant la page
`http://localhost:8000/api/account`.  

Cependant, histoire d'éviter de passer toutes les propriétés privées 
du User au Serializer mais passer uniquement les propriétés qu'on veut,
on va associer les propriétés qui nous intéressent dans le User à 
l'annotation `@Groups("main")`:
```PHP
    /**
     * @ORM\Column(type="string", length=180, unique=true)
     * @Groups("main")
     */
    private $email;

...
    /**
     * @ORM\Column(type="string", length=255)
     * @Groups("main")
     */
    private $firstName;

...
    /**
     * @ORM\Column(type="string", length=255, nullable=true)
     * @Groups("main")
     */
    private $twitterUsername;
```

Il suffit alors de modifier légèrement l'appel à `$this->json()` pour 
n'avoir que les propriétés qu'on souhaite dans l'appel API :
```PHP
return $this->json($user, 200, [], [
    'groups' => ['main']
]);
```

