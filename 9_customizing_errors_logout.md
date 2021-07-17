# Customizing Errors & Logout

Si on souhaite customiser les messages d'erreurs de connexion, on peut soit :
- ajouter un nouveau type d'exception appelé `CustomUserMessageAuthenticationException`
: c'est ce qu'on fera lorsque le tuto parlera des connexions par API;
- modifier les traductions : c'est ce qu'on va faire maintenant.  

Lorsqu'on voit une erreur de connexion sur la page, on peut aussi cliquer
sur l'icône de traduction dans le profiler. Elle permet de voir les traductions 
utilisées sur la page: les id, la preview, le nombre de fois où la traduction 
est utilisée, etc.  

Pour customiser notre traduction, on va créer un fichier `security.en.yaml` dans 
le répertoire `translations/`. On va le remplir avec une nouvelle traduction :
```yaml
"Invalid credentials.": "Oh no ! Invalid credentials."
```
(Je n'arrive toujours pas à avoir l'id "Username could not be found", du coup j'ai 
changé le message "Invalid Credentials").  
Notons que sur certaines versions de Symfony, le cache doit être vidé manuellement
(`./bin/console cache:clear`) avant de voir la mise en application des nouveaux
changements de traductions.  

On cherche à présent à déconnecter un utilisateur connecté.  
Pour ce faire, on va commencer par créer une nouvelle fonction `logout()` dans 
notre `SecurityController` : 
```PHP
/**
 * @Route("/logout", name="app_logout")
 */
public function logout ()
{
}
```
La fonction n'est pas vide par inattention: en fait, on doit créer la route et son nom, mais 
on n'a pas besoin de remplir le contenu de la fonction `logout()`.  

En effet, il suffit d'ajouter le contenu ci-dessous dans `security.yaml`, dans `main:`
après la partie sur `guard:` :
```yaml
logout:
  path: app_logout
```

Et c'est bon : lorsqu'on va sur `localhost:8000/logout`, on est redirigé sur la page d'accueil
mais le profiler indique bien qu'on est en session anonyme, même si on était connecté via 
un compte avant. Des options existent pour customiser la déconnexion (par exemple, changer
 l'url de déconnexion), et sont disponibles si besoin dans la section "référence" de Symfony.