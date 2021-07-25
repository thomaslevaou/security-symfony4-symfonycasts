# Article Admin & Low-Level Access Controls

Depuis les devs du chapitre précédent, la page d'accueil est buguée. En effet
un objet User ne peut pas être converti en string comme ça.  
Dans `base.html.twig`, on appelle l'auteur de l'article via un `article.author`.  
On pourrait mettre `article.author.firstName` à la place pour résoudre le souci,
mais pour changer ici on va faire appel à une fonction `__toString()` dans notre
entité `User` qui ne fera que retourner le prénom de l'utilisateur:
```PHP
public function __toString()
{
    return $this->getFirstName();
}
```

Ce qui débloque notre page d'accueil.  

On souhaite à présent créer un bouton d'édition sur nos articles, mais qui ne sera
accessible que pour l'auteur de l'article, pas pour un autre utilisateur du site.  

Pour ce faire, on commence par créer une méthode de base dans notre
`ArticleAdminController` :
```PHP
/**
 * @Route("/admin/article/{id}/edit")
 */
public function edit(Article $article)
{
    dd($article);
}
```

Comme `Article` est une entité, le bundle `SensioFrameworkExtraBundle` 
peut directement retrouver l'article associé à l'id dans l'URL.  

Avec le code ci-dessus, on peut rentrer l'URL  ̀http://localhost:8000/admin/article/10/edit`,
et après connexion avec un admin, on peut voir directement le contenu de l'article
d'identifiant 10.

Maintenant on va vouloir changer le droit d'accès à cette page: la page d'édition
de l'article sera accessible si on est administrateur OU si on est simple utilisateur
mais auteur de l'article sélectionné.  

Pour commencer, on déplace le `@IsGranted("ROLE_ADMIN_ARTICLE")` au-dessus de la 
méthode `new()`, de sorte à avoir le champ libre pour notre méthode `edit()`.  

Une solution pour se sortir de cette situation est de spécifier
la logique dans la méthode `edit()` :
```PHP
public function edit(Article $article)
{
    if ($article->getAuthor() != $this->getUser() && !$this->isGranted('ROLE_ADMIN_ARTICLE')) {
        throw $this->createAccessDeniedException('No access !');
    }
    dd($article);
}
```

Ca dépanne, mais pour éviter d'avoir une logique copié-collée dans tous les sens
dans l'appli (en php et en twig) pour gérer les accès (et donc galère à
maintenir sur la durée), on va utiliser un 
système de **Voters**, qu'on va voir dans le chapitre suivant.
