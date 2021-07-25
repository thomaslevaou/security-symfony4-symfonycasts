# Adding a Custom Voter

On va créer ici le Voteur custom mentionné dans le chapitre précédent.  
Pour ce faire, on va passer par la commande `php bin/console make:voter`.  
On va l'appeler `ArticleVoter`. En général, on va avoir un voteur par objet
qui en demande un pour déterminer son accès.  

Le voteur est alors disponible dans `src/Security/Voter/ArticleVoter.php`.  
Sa fonction `supports()` sera appelée à chaque appel de `isGranted()` 
pour savoir s'il est concerné par l'appel de cette dernière méthode
ou non. De ce fait, on va modifier légèrement son code pour qu'il
soit le suivant :
```PHP
protected function supports($attribute, $subject): bool
{
    return in_array($attribute, ['MANAGE'])
        && $subject instanceof \App\Entity\Article;
}
```
Si `supports()` retourne false, alors le voteur s'abstient.  
Si cette méthode retourne true, alors Symfony appelle 
la méthode `voteOnAttribute()`. Le voteur votera 
en fonction du résultat de cette méthode: l'accès
sera autorisé en cas de true, refusé en cas de false.  

Ainsi, après avoir ajouté le service Security au constructeur pour pouvoir
accéder à `isGranted()` même en dehors d'un contrôleur, notre code 
de `voteOnAttribute()` va être le suivant :
```PHP
protected function voteOnAttribute($attribute, $subject, TokenInterface $token): bool
{
    /** @var Article $subject */
    $user = $token->getUser();
    // if the user is anonymous, do not grant access
    if (!$user instanceof UserInterface) {
        return false;
    }

    // ... (check conditions and return true to grant permission) ...
    switch ($attribute) {
        case 'MANAGE':
            // this is the author
            if ($subject->getAuthor() == $user) {
                return true;
            }

            if ($this->security->isGranted('ROLE_ADMIN_ARTICLE')) {
                return true;
            }

            return false;
    }

    return false;
}
```

Ce qui résout notre problème d'accès sans avoir à écrire la logique directement
dans le contrôleur.  

Dans le contrôleur, on va optimiser le code précédent en une ligne: 
`$this->denyAccessUnlessGranted('MANAGE', $article);` remplace le if 
précédemment à cette place.  
Et on peut même faire encore mieux en remplaçant l'appel à cette 
fonction par l'annotation telle que décrite ci-dessous dans ce contexte :
```PHP
    /**
     * @Route("/admin/article/{id}/edit")
     * @IsGranted("MANAGE", subject="article")
     */
    public function edit(Article $article)
    {
        dd($article);
    }
```

Mais ça marche parce qu'on a utilisé le bundle qui permet de passer
l'article en argument de l'annotation et que le "subject" ici est un 
argument du contrôleur. Quand ce n'est pas le cas, la fonction
`denyAccessUnlessGranted` fait très bien le taff.

Je suppose qu'en Twig ça doit être pareil avec la méthode `is_granted()`,
à tester à l'occasion.

