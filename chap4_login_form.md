# The Login Form

00h40 - 1h10

On rappelle que pour développer un formulaire de connexion, on a besoin du
formulaire en front et de la logique PHP derrière. 
Ici, on va créer ce formulaire en commençant par le contrôleur :
`./bin/console make:controller`, et on va l'appeler `SecurityController`.

On se retrouve alors avec un nouveau contrôleur `SecurityController`, avec le code
suivant qu'on enrichit un peu en regardant la doc de Symfony sur les formulaires
de connexion :
```PHP
class SecurityController extends AbstractController
{    
    /**
     * @Route("/login", name="app_login ")
     */
    public function login(AuthenticationUtils $authenticationUtils): Response
    {
        // get the login error if there is one
        $error = $authenticationUtils->getLastAuthenticationError();
        // last username entered by the user
        $lastUsername = $authenticationUtils->getLastUsername();

        return $this->render('security/index.html.twig', [
            'last_username' => $lastUsername,
            'error' => $error
        ]);
    }
}
```

Et son code associé dans `security/login.html.twig`.  
Notons que si un formulaire n'a pas de path dans son attribut "action" de 
la balise "form", il renvoie par défaut au même contrôleur que celui qui 
l'a appelé (SecurityController ici donc).  
On a déplacé le fichier `tutorial/login.css` dans `public.css` pour ajouter 
un peu de style visuel au formulaire : 
```HTML
{% extends 'base.html.twig' %}

{% block title %}Login!{% endblock %}

{% block stylesheets %}
    {{ parent() }}
    <link rel="stylesheet" href="{{ asset('css/login.css') }}">
{% endblock %}

{% block body %}
    <form class="form-signin" method="post">
        {% if error %}
            <div class="alert alert-danger">{{ error.messageKey|trans(error.messageData, 'security') }}</div>
        {% endif %}
        <h1 class="h3 mb-3 font-weight-normal">Please sign in</h1>
        <label for="inputEmail" class="sr-only">Email address</label>
        <input type="email" name="email" id="inputEmail" class="form-control" placeholder="Email address" required autofocus>
        <label for="inputPassword" class="sr-only">Password</label>
        <input type="password" name="password" id="inputPassword" class="form-control" placeholder="Password" required>
        <div class="checkbox mb-3">
            <label>
                <input type="checkbox" value="remember-me"> Remember me
            </label>
        </div>
        <button class="btn btn-lg btn-primary btn-block" type="submit">
            Sign in
        </button>
    </form>
{% endblock %}
```

Notons la surcharge du block stylesheets pour ajouter le bloc concerné.

On a aussi remplacé le menu en haut à droite par un unique bouton login, pour
le moment : 
```HTML
<ul class="navbar-nav ml-auto">
    <li class="nav-item">
        <a style="color: #fff;" class="nav-link" href="{{ path('app_login') }}">Login</a>
    </li>
    {#<li class="nav-item dropdown" style="margin-right: 75px;">
        <a class="nav-link dropdown-toggle" href="http://example.com" id="navbarDropdownMenuLink" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false">
          <img class="nav-profile-img rounded-circle" src="{{ asset('images/astronaut-profile.png') }}">
        </a>
        <div class="dropdown-menu" aria-labelledby="navbarDropdownMenuLink">
            <a class="dropdown-item" href="#">Profile</a>
            <a class="dropdown-item" href="#">Create Post</a>
            <a class="dropdown-item" href="#">Logout</a>
        </div>
    </li>#}
</ul>
```