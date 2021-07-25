# Manual Authentication / Registration

On cherche à présent à créer un formulaire de création de compte. Une fois le 
compte créé, celui-ci sera directement authentifié.  

On commence donc par créer sa route associée dans `SecurityController` :
```PHP
/**
 * @Route("/register", name="app_register")
 */
public function register()
{
    $this->render('security/register.html.twig');
}
```

On écrit ensuite le code du Template Twig (qui sera refait dans le cours sur les
formulaires, donc on se permet de retirer la protection CSRF pour le moment), qui
reste assez simple et fortement inspiré du formulaire de login :
```HTML
{% extends 'base.html.twig' %}

{% block title %}Register!{% endblock %}

{% block stylesheets %}
    {{ parent() }}
    <link rel="stylesheet" href="{{ asset('css/login.css') }}">
{% endblock %}

{% block body %}
    <form class="form-signin" method="post">
        {# todo - replace with a Symfony form #}
        <h1 class="h3 mb-3 font-weight-normal">Register</h1>
        <label for="inputEmail" class="sr-only">Email address</label>
        <input type="email" name="email" id="inputEmail" class="form-control" placeholder="Email address" required autofocus>
        <label for="inputPassword" class="sr-only">Password</label>
        <input type="password" name="password" id="inputPassword" class="form-control" placeholder="Password" required>
        <div class="checkbox mb-3">
            <label>
                <input type="checkbox" name="_term"> Agree to terms I for sure read
            </label>
        </div>
        <button class="btn btn-lg btn-primary btn-block" type="submit">
            Register
        </button>
    </form>
{% endblock %}
```

On a aussi ajouté un lien vers ce formulaire dans `base.html.twig`.  

Le formulaire de création de compte est dans son fonctionnement similaire
à un formulaire quelconque, il n'a pas besoin de traitement par un 
Authenticator comme en a besoin le formulaire de connexion. On va 
donc directement récupérer ses données dans le `SecurityController`
et les insérer en base, ce qui donne le code suivant (sans vérification
des données du formulaire, car on le fera plus tard avec les outils Symfony) :
```PHP
    public function register(Request $request, UserPasswordEncoderInterface $passwordEncoder)
    {
        // TODO - use Symfony forms & validation
        if ($request->isMethod('POST')) {
            $user = new User();
            $user->setEmail($request->request->get('email'));
            $user->setFirstname('Mystery');
            $user->setPassword($passwordEncoder->encodePassword(
                $user,
                $request->request->get('password')
            ));

            $em = $this->getDoctrine()->getManager();
            $em->persist($user);
            $em->flush();

            return $this->redirectToRoute('app_account');
        }
        return $this->render('security/register.html.twig');
    }
```

Du coup ce formulaire marche bien, mais demande à l'utilisateur de se connecter 
avec ses identifiants juste après la création du compte. De plus, après 
l'authentification on souhaite ici que l'utilisateur soit redirigé sur la page
qu'il avait laissée avant de s'authentifier.  

Le mieux ici est de directement utiliser `LoginFormAuthenticator`, dont on 
va faire appel via un `GuardAuthenticatorHandler` comme ci-dessous: 
```PHP
public function register(Request $request, UserPasswordEncoderInterface $passwordEncoder, GuardAuthenticatorHandler $guardHandler, LoginFormAuthenticator $formAuthenticator)
    {
        // TODO - use Symfony forms & validation
        if ($request->isMethod('POST')) {
     ...
//            return $this->redirectToRoute('app_account');
            return $guardHandler->authenticateUserAndHandleSuccess(
                $user,
                $request,
                $formAuthenticator,
                'main'
            );
        }
        return $this->render('security/register.html.twig');
    }
```

La méthode `authenticateUserAndHandleSuccess()` prend l'utilisateur,
la requête, le LoginFormAuthenticator et le nom du firewall (ici main)
pour authentifier directement l'utilisateur et appeler `onAuthenticationSuccess()`
directement après la requête. C'est un peu un raccourci pour appeler directement
`LoginFormAuthenticator` même si l'envoi des données de création de compte
n'est initialement pas un cas géré par `supports` dans le `LoginFormAuthenticator`.  

On rappelle que comme vu dans le chapitre 14, le `TargetPathTrait` permet de
récupérer l'avant-dernier lien visité avant l'appel au LoginFormAuthenticator,
ce qui est tout à fait valide dans notre contexte de création de compte ici.

