# Deny Access in the Controller

Après avoir vu en quoi consistait l'access_control, on va à présent voir 
la deuxième manière de restreindre des accès en Symfony.  
En effet l'access_control est très pratique pour restreindre plusieurs pages d'un coup.  
Mais parfois, on a besoin de restreindre uniquement certaines parties de contrôleur
(genre juste des méthodes), ou juste un contrôleur précis pour lequel faire une regexp
juste pour ce contrôleur ne ferait pas très propre.  

Si on commente les deux lignes dans la partie `access_control` de `security.yaml`, 
on peut toujours restreindre l'accès à la méthode `index()` de
`CommentAdminController` en ajoutant la ligne suivante en début de fonction :
```PHP
/**
 * @Route("/admin/comment", name="comment_admin")
 */
public function index(CommentRepository $repository, Request $request, PaginatorInterface $paginator)
{
    $this->denyAccessUnlessGranted('ROLE_ADMIN');
   ....
```

Mais on peut même faire encore mieux avec des annotations, comme sur Trëmma : 
```PHP
    /**
     * @Route("/admin/comment", name="comment_admin")
     * @IsGranted("ROLE_ADMIN")
     */
    public function index(CommentRepository $repository, Request $request, PaginatorInterface $paginator)
    {
        $q = $request->query->get('q');
    ...
```
Cette annotation peut être prise en charge via le bundle
`SensioFrameworkExtraBundle` qui a été installé lorsqu'on a fait un
`composer require annotations` il y a longtemps, pour rappel.  

On peut aussi déplacer cette annotation tout en haut du contrôleur :
```PHP
/**
 * @IsGranted("ROLE_ADMIN")
 */
class CommentAdminController extends Controller
{
    /**
     * @Route("/admin/comment", name="comment_admin")
     */
    public function index(CommentRepository $repository, Request $request, PaginatorInterface $paginator)
    {
        $q = $request->query->get('q');
...
```

Ce qui aura le même résultat sur toutes les méthodes de la classe.