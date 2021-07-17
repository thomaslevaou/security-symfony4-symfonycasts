# Fetching the User In a Service

On a vu comment récupérer un User dans un contrôleur ou en Twig. Maintenant, on va
voir comment en récupérer un depuis un Service.

Par exemple, dans notre `MarkdownHelper`, au lieu d'afficher en log juste le message
"They are talking about bacon again", on va vouloir afficher le nom de l'utilisateur
actuellement connecté. Pour ce faire, on va faire appel au Service `Security`.  

Ce Service contient deux méthodes `getUser()` et `isGranted()` qu'on peut 
utiliser. On peut donc ajouter le Service `Security` au constructeur de 
`MarkdownHelper`, et ajouter le user via le tableau appelé "context" disponible en
deuxième argument des méthodes de logger telles que `debug()`, `info()`, ou
`alert()` :
```PHP
if (stripos($source, 'bacon') !== false) {
    $this->logger->info('They are talking about bacon again!', [
        'user' => $this->security->getUser()
    ]);
}
```
Dans les logs debug du Profiler, le User associé devient alors visible 
via le lien "Show context".