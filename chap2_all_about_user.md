# All About the User class

Dans la classe `Entity/User`, on peut voir qu'en plus de l'id et de l'e-mail créés, 
cette entité dispose aussi d'un attribut `roles`.  

En fait la classe User ressemble pas mal à une entité classique, sauf que cette
entité hérite l'interface `UserInterface`.  

En fait cette interface implique juste de faire hériter la classe de certaines méthodes,
qui sont les suivantes :
- `getUsername()` : c'est un mauvais nom car les utilisateurs n'ont pas forcément
un username, mais c'est la méthode qui doit juste retourner l'"identifiant visuel"
(comme on disait dans le chapitre précédent) de l'utilisateur. Elle sert généralement
à afficher l'identifiant de l'utilisateur sur le Profiler uniquement;
- `getRoles()` : on en reparlera plus tard, c'est pour les permissions;
- `getPassword()`, `getSalt()`, `eraseCredentials()` seront utiles quand on fera
une gestion des mots de passe dans notre appli Symfony, ce qui n'est pas le cas
pour le moment.  
  
Donc seules `getUsername()` et `getRoles()` sont utiles pour le moment.  

Notons aussi qu'avec la création d'User, le fichier `config/packages/security.yaml`
a également été modifié. En effet, la clé `app_user_provider` a été modifiée.  
En effet chaque classe User (et en général, un projet Symfony n'aura besoin que
d'une classe User) a besoin d'un **user provider** associé. Mais pour l'instant 
ce n'est pas important d'en savoir plus, on verra ça plus tard.


