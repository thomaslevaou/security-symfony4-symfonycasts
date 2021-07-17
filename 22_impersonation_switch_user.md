# Impersonation (switch_user)

Supposons qu'on se retrouve dans une situation où un utilisateur standard
remonte un bug en production, mais que je ne reproduis pas avec mon compte
admin.  

Pour se mettre à la place d'un autre utilisateur, on peut appliquer la 
fonctionnalité `switch_user`.  
Pour ce faire, on ajoute la ligne suivante dans la section
`firewalls: > main:` de `security.yaml` :
```yaml
      switch_user: true
```

On peut alors ajouter à la fin de notre URL dans le navigateur 
un `?_switch_user=spacebar1@example.com` (ou tout autre compte
dont on souhaiterait utiliser l'identité). Mais les pages
accessibles via un switch_user le sont uniquement si
l'utilisateur qui impersonifie a le rôle
`ROLE_ALLOWED_TO_SWITCH`. On l'ajoute donc à nos `ROLE_ADMIN`: 
```YAML
    role_hierarchy:
        ROLE_ADMIN: [ROLE_ADMIN_COMMENT, ROLE_ADMIN_ARTICLE, ROLE_ALLOWED_TO_SWITCH]
```

Une fois cela fait, si on va sur une adresse comme
localhost:8000/admin/comment?_switch_user=spacebar1@example.com,
on peut bien voir dans le Profiler qu'on est connecté en tant
que spacebar1@example.com même si on a toujours une erreur
403.  

Si on peut utiliser l'e-mail comme identifiant de l'utilisateur,
c'est grâce à la configuration qu'on a précisée plus tôt dans 
ce cours dans `app_user_provider` :
```YAML
# used to reload user from session & other features (e.g. switch_user)
        app_user_provider:
            entity:
                class: App\Entity\User
                property: email
```

Si on avait mis l'id dans `property` ci-dessus, il aurait fallu
entrer l'id de l'user comme valeur de `switch_user` dans l'URL.  

Pour arrêter l'impersonification, on rentrer le paramètre GET
`_exit` dans l'URL, comme par exemple :
localhost:8000/admin/comment?_switch_user=_exit  

Pour éviter d'"oublier" qu'on est en switch_user (ce qui entraînerait
un risque d'usurpation d'identité en prod par exemple), on va ajouter
un gros signal sur l'interface qu'on est en switch_user. Pour ce faire,
on va profiter dans `base.html.twig` du fait que lorsqu'on est en
switch_user, Symfony nous ajoute un rôle `ROLE_PREVIOUS_ADMIN` :
```HTML
        {% if is_granted('ROLE_PREVIOUS_ADMIN') %}
            <div class="alert alert-warning" style="margin-bottom: 0;">
                You are currently switched to this user.
                <a href="{{ path('app_homepage', {
                    '_switch_user': '_exit'
                }) }}">Exit Impersonation</a>
            </div>
        {% endif %}
```

Notons que si on passe une clé en deuxième argument de la fonction 
Twig `path()` et que la route n'a pas de wildcard avec ce nom (pas 
comme article slug quand on avait fait le clic sur le lien des articles),
Symfony ajoute l'argument en tant que paramètre de requête GET.