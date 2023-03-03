---
marp: true
theme: gaia
markdown.marp.enableHtml: true
paginate: true
---

<style>

section {
  background-color: #fefefe;
  color: #333;
  font-size: 28px;
}

img[alt~="center"] {
  display: block;
  margin: 0 auto;
}
blockquote {
  background: #ffedcc;
  border-left: 10px solid #d1bf9d;
  margin: 1.5em 10px;
  padding: 0.5em 10px;
}
blockquote:before{
  content: unset;
}
blockquote:after{
  content: unset;
}

/* https://github.com/marp-team/marp/discussions/62 */
div.twocols {
  margin-top: 35px;
  column-count: 2;
}
div.twocols p:first-child,
div.twocols h1:first-child,
div.twocols h2:first-child,
div.twocols ul:first-child,
div.twocols ul li:first-child,
div.twocols ul li p:first-child {
  margin-top: 0 !important;
}
div.twocols p.break {
  break-before: column;
  margin-top: 0;
}
</style>

<!-- _class: lead -->

# IPI - JVA320

## Servlets, Spring MVC et Thymeleaf

&nbsp;

#### Marc Dutoo

---

## L'enseignant

- Marc Dutoo, architecte
  - 20 ans d'Open Source, Java, distribué, innovation
  - et plus récemment Data, Python...
  - https://www.linkedin.com/in/marcdutoo
  - https://www.slideshare.net/mdutoo
  - https://github.com/mdutoo
- Mes crédos pédagogiques :
  - pour tirer les bénéfices d'un outil, méthode ou approche, il faut comprendre comment cela marche ! et souvent d'où ça vient
  - des exemples concrets éclairants, tirés du monde réel
  - Pratiquez, pratiquez, pratiquez !

---

## Le cours

[
TODO évaluation ; td, tp ; et méthodologies ?
]::

Objectifs et compétences visés :

(**en gras**, les travaux pratiques en séance et évalués)

- Serveur : Servlets, **Spring MVC**
  - **@Controller, @RequestMapping (@Get/Post...Mapping), @RequestParam, @PathVariable, databinding d'entité**
  - BONUS : BindingResult
- Frontend : **Spring Thymeleaf**
  - **th:text et ${}, URLs avec @{...}, messages, #temporals, th:if/switch, th:each**
  - BONUS : erreurs, th:fragment

& dates, évaluation et retours

---

<!-- _class: lead -->

# Servlets

---

## Architecture d'internet - origines et TCP/IP

Internet est le réseau des réseaux mondial maillé réunissant des ordinateurs communiquant par des protocoles standards. Ses origines remontent à ARPAnet lors de la guerre froide, laquelle a été la première cause de son caractère redondant.

Internet est en pratique la pile **TCP/IP** ("TCP sur IP"), qui consiste fondamentalement en 2 couches :
- la couche Transport (protocole TCP),
- bâtie au-dessus de la couche Réseau (protocole IP).

Tout ce qui est au-dessus (et se sert donc de TCP) est de la couche Application,
Tout ce qui est en-dessous (sur quoi est bati IP) est la couche d'accès réseau.

---

## Architecture d'internet - la pile OSI

Une version détaillée bien connnue est la pile des 7 couches inspirées du modèle OSI :
- 7. application : HTTP, HTTPS et à peu près tout le reste : SNMP
- 6. Présentation : HTML, CSS ; JSON, XML ; mais aussi MQTT, SSL/TLS, Mime...
- 5. Session : RPC...
- 4. Transport - TCP (Paquets, connexion garantissant un niveau de service), aussi UDP (sans)
- 3. Réseau - IP (Trames, "best effort")
- 2. Liaison - adresses Mac
- 1. Physique (bit, en numérique ou analogique)

Là aussi, chaque couche est basée sur la précédente.

---

## Architecture du web - le standard HTTP

- HTTP est le protocole standard du web. Il permet notamment aux navigateurs web de récupérer des pages web.
- Plus largement, HTTP permet des interactions client-serveur, par le biais d'opérations (**method**)
  - similaires à celles de CRUD : GET, POST, PUT, DELETE
  - et PATCH (PUT avec fusion), HEAD (GET sans body retourné), OPTIONS...

- La force de HTTP est sa capacité de passage à l'échelle. Son secret en est la possibilité de **cacher des données côté client**.
  - Notamment, une fois qu'un navigateur a récupéré une page web, si l'utilisateur revient dessus plus tard, il ne va la récupérer à nouveau que s'il y a une chance qu'elle ait changé. Par exemple, de nombreux sites indiquent à HTTP que leurs pages on une date d'expiration dans une heure.

---

## HTTP - garanties des methods, contraintes REST

Le cache côté client est possible notamment grâce aux garanties des methods HTTP : *safe* (ne change pas la ressource côté serveur) et *idempotent* (amène toujours le même résultat).

| method | Equivalent CRUD | safe | idempotent | request body | response body |
|--------|-----------------|------|------------|--------------|---------------|
| GET    | Read            | *    | *          |              | *             |
| POST   | Create          |      |            | *            | *             |
| PUT    | Update          |      | *          | *            |               |
| DELETE | Delete          |      | *          |              | *             |


[
Le protocole HTTP permet de gérer une **resource** (donnée atomique) hébergée par le serveur, par le biais d'opérations (**method**) similaires à celles de CRUD :
https://www.tablesgenerator.com/markdown_tables#
https://blog.octo.com/should-i-put-or-should-i-patch/
La force de HTTP est son passage à l'échelle, permis par le cache côté client, autorisé - à condition d'utiliser les bonnes method HTTP - par leurs garanties :
safe : ne change pas la ressource côté serveur
idempotent : aboutit toujours au même résultat
]::

Les APIs REST (REpresentational State Transfer ou transfert de représentation de la donnée) utilisent les methods HTTP à la manière de CRUD pour manipuler une donnée atomique hébergée par le serveur (**resource**), en s'efforçant (pas toujours...) de respecter ces garanties et d'autres contraintes (URIs pérennes, sans état...).

[
TODO TODO TODO !!!
exo et screenshot debug HTTP GET page web et post
(ou dans Spring MVC)
WeTransfer et / ou httpbin.org et autres stackoverflow ET debug (Spring MVC) mapping
]::

---

## Architecture d'une application web - client et serveur

L'architecture d'une application web est composée de :
- le serveur HTTP
  - avec potentiellement en façade des serveurs proxy, de pages web statiques, de cache, d'équilibrage de charge
  - et derrière des librairies voire serveurs de services métier, persistence, bases de données...
- le client web (le navigateur) faisant tourner les ressources web
  - HTML, CSS et le code côté client : JavaScript

On appelle "frontend" la partie visible par l'utilisateur, et donc par extension le code en général non seulement des ressources web, mais aussi de la partie du serveur qui interagit avec elles. Cette dernière repose sur les services codés internes au côté serveur, appelés "backend".

---

## Application web - pages web dynamiques

[incluant ces données spécifiques]::

Développer le côté serveur d'une application web permet de ne pas se contenter de servir par HTTP des ressources web statiques (disponibles à l'identique sur le système de fichier du serveur), mais bien de les adapter de manière "dynamique" selon les données actuelles de l'application, et ce :
- soit en générant du HTML, typiquement par un **rendu côté serveur** de pages web "templates" (modèles) comprenant un langage de présentation web coté serveur
- soit en générant du JSON (ou autre format lisible de données), qui va être inclu dans la page web par le biais de son modèle en mémoire (le DOM) par un **rendu côté client web** effectué par une librairie JavaScript (telle React.js ou Angular, qui ont leur propre langage de présentation web côté client)

Chacun a son avantage. Le deuxième est plus puissant pour des interactions utilisateurs nombreuses, mais aussi plus lourd et complexe, et mérite un cours séparé. Aussi le présent cours se focalise-t-il sur le premier.


---

## Le standard J2EE Servlet

Java a **inventé** la notion d'application web avec les **standards J2EE**, et tout d'abord les servlets en 1996.

Aujourd'hui, une application web Java sera plutôt développée à l'aide de frameworks de plus haut niveau. Mais TOUS ces frameworks sont eux-mêmes toujours développés **au-dessus des servlets**, et en simplifient les notions. Donc il reste utile de savoir comment ils marchent et leurs principes.

[
Le **standard J2EE** par lequel les interactions web (HTTP) se retrouvent gérables en Java est les servlets.
...
Mais la plupart du temps, une application web Java sera plutôt développée à l'aide d'un framework de plus haut niveau qui lui-même se base sur des servlets.
]::

Les servlets d'une application J2EE sont hébergés par son **serveur d'application** J2EE (Tomcat...).

[
Il est alors possible de développer les traitements de la partie serveur des interactions web
soit en écrivant directement un servlet, étendant HttpServlet et surchargeant ses méthodes doGet/Post() au besoin,
soit dans un framework web Java plus haut niveau lui-même basé sur les servlets.
  ]::

Développer une application web en servlets revient à écrire un ou plusieurs **nouveau servlet qui étend HttpServlet, à surcharger ses méthodes doGet/Post()** au besoin et à y implémenter les traitements souhaités selon l'appel HTTP (demande de page web, création de données...).

---

## La classe HttpServlet

```java
public abstract class HttpServlet extends GenericServlet {

protected void doGet(HttpServletRequest req, HttpServletResponse resp) {} // à surcharger
protected void doPost(HttpServletRequest req, HttpServletResponse resp) {}
protected void doPut(HttpServletRequest req, HttpServletResponse resp) {}
protected void doDelete(HttpServletRequest req, HttpServletResponse resp) {}

protected void doHead(HttpServletRequest req, HttpServletResponse resp) {}
protected void doOptions(HttpServletRequest req, HttpServletResponse resp) {}
protected void doTrace(HttpServletRequest req, HttpServletResponse resp) {}
public void service(...) {...} // redirige vers les précédentes qui sont à surcharger
  
...
}
```

La classe **HttpServletRequest** permet d'accéder à toutes les informations de la requête HTTP envoyée au servlet : URL dont ses paramètres, header, body...
La classe **HttpServletResponse** permet de modifier toutes les données de la réponse HTTP qui sera retournée par le servlet au client web : header, body...

[
void init(ServletConfig config) {} // à surcharger
et HttpServletResponse fournissent tous les paramètres permettant respectivement d'accéder 
NB. doPatch() à ajouter en surchargeant service()
]::

---

## Un servlet simple qui affiche Hello World

<div class="twocols">

```java
package ex;
import...
public class HelloWorldServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req,
                         HttpServletResponse resp)
            throws ServletException, IOException {
        resp.setContentType("text/html");
        PrintWriter pw = resp.getWriter();
        pw.println("<html><body>");
        pw.println("<h1>Hello World !</h1>");
        pw.println("</body></html>");
        pw.close();
    }
}
```

<p class="break"></p>

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<web-app version="3.0"
xmlns="http://java.sun.com/xml/ns/javaee"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd">

    <servlet>
        <servlet-name>HelloWorldServlet</servlet-name>
        <servlet-class>ex.HelloWorldServlet</servlet-class>
    </servlet>

    <servlet-mapping>
        <servlet-name>HelloWorldServlet</servlet-name>
        <url-pattern>/helloworld</url-pattern>
    </servlet-mapping>
</web-app>
```

</div>
http://localhost:8080/helloworld => Hello World !

<!--
<html>
<body>
<h1>Hello World !</h1>
</body>
</html>
-->

(ici, par simplification, **le HTML est écrit** directement par le code Java. En pratique, ce sera plutôt par un **langage dédié à la présentation** : JSP ou templates Thymeleaf)

[TODO !!! OU image liant class, (name), url]::

---

## Descripteur de déploiement web.xml

Pour que le serveur d'application **déploie** ce servlet, il faut lui fournir :
- son code (sa .class et toutes ses dépendances : fichier(s) .jar ou .war)
- son **descripteur** de déploiement, le fichier web.xml, avec dedans au moins le **mapping** (correspondance requête vers code) qu'il doit appliquer :
  - **quelle classe** instancier
  - et servir sur **quelle URL**.

Une fois fourni au serveur d'application et celui-ci lancé, le code développé sera appelé en navigant à l'URL http://localhost:8080/helloworld (le serveur d'application étant configuré par défaut pour répondre sur le port 8080)

[TODO image liant class, (name), url]::

---

## Cycle de vie du servlet

Le cycle de vie du servlet 

- initialisation (le serveur d'application se lance...) : appel de sa méthode init()
- fonctionnement nominal : à chaque appel HTTP mappé vers le servlet, appel de ses méthodes doGet(), doPost() etc. selon la method HTTP
- destruction (le serveur d'application s'arrête...) : appel de sa méthode destroy()

Par exemple, on peut écrire dans init() et destroy() du code de connexion et déconnexion à une base de donnnées à utiliser dans l'application.

---

## Configuration avancée web.xml

<div class="twocols">

- **init-param** : passage de configuration spécifique au servlet
- **filter** : code qui sera appelé avant des servlets sur la même URL, typiquement pour d'abord et de manière commune déléguer l'authentification de l'utilisateur à un annuaire d'entreprise (Active Directory, LDAP)...
- **listener** : code appelé lors des événements du cycle de vie de tous les servlets, typiquement pour mutualiser l'initialisation de l'accès à une base de données commune...

<p class="break"></p>

```xml
<filter>
    <filter-name>AuthenticationFilter</filter-name>
    <filter-class>ex.AuthFilter</filter-class>
    <init-param>
        <param-name>password</param-name>
        <param-value>mypassword</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>AuthenticationFilter</filter-name>
    <url-pattern>/</url-pattern>
</filter-mapping>
<listener>
    <listener-class>ex.DBListener</listener-class>
</listener>
<servlet>
    <servlet-name>HelloWorldServlet</servlet-name>
    <servlet-class>ex.HelloWorldServlet</servlet-class>
    <init-param>
        <param-name>prefix</param-name>
        <param-value>Hello</param-value>
    </init-param>
</servlet>
```

</div>

---

## Afficher ce qui est passé en requête et en initialisation

```java
private String prefix;
@Override
void init(ServletConfig config) {
    prefix = getServletConfig().getInitParameter("prefix"); // on récupère la valeur configurée dans web.xml
}
@Override
protected void doGet(HttpServletRequest req,
                     HttpServletResponse resp)
        throws ServletException, IOException {
    String name = req.getParameter("name"); // on récupère le paramètre passé en URL
    
    resp.setContentType("text/html");
    PrintWriter pw = resp.getWriter();
    pw.println("<html>");
    pw.println("<body>");
    pw.println("<h1>" + prefix + " " + name + " !</h1>");
    pw.println("</body");
    pw.println("</html>");
    pw.close();
}
```

http://localhost:8080/helloworld?name=World => Hello World !

---

## Traiter ce qui est entré par l'utilisateur

Permettre à un utilisateur d'entrer des données à envoyer au serveur se fait en HTML par un formulaire, un élément "form" avec les paramètres de l'appel HTTP qu'il fera :
- l'URL qu'il ciblera
- la method HTTP qu'il emploiera, en HTML en général POST

<div class="twocols">

```html
<!-- index.html -->
<!DOCTYPE html>
<html>
<head><title>Hello World</title></head>
<body>
<form method="post"
      action="http://localhost:8080/helloworld">
  Nom : <input type="text" name="nom" value="">
  <input type="submit" value="OK">
</form>
</body></html>
```

<p class="break"></p>

Cela affiche :

<!-- index.html -->
<!DOCTYPE html>
<html>
<head><title>Hello World</title></head>
<body>
<form method="post"
    action="http://localhost:8080/helloworld">
  Nom : <input type="text" name="nom" value="">
  <input type="submit" value="OK">
</form>
</body></html>

Cliquer sur le bouton déclenchera un appel HTTP POST du navigateur vers le serveur, qui sera mappé par le serveur d'application vers la méthode doPost() du servlet développé précédemment.

</div>

---

## Traiter ce qui est entré - côté servlet

Pour que le code de ce servlet puisse accéder alors à la valeur entrée, il faut encore simplement mettre ce code dans la méthode doPost() (et non plus doGet()) :

```java
...
@Override
protected void doPost(HttpServletRequest req,
                     HttpServletResponse resp)
        throws ServletException, IOException {
    String name = req.getParameter("name"); // on récupère le paramètre passé (en body de POST)
    
    ...
}
```

---

## Approche MVC : sortir le HTML du servlet avec JSP

Bien sûr, dans la réalité, on n'écrit pas le HTML sous forme de String dans le code Java du serveur comme vu ! car
- c'est très peu lisible (pas de coloration syntaxique) et productif (copier-coller...)
- le HTML est "fragile" lors des évolutions du code du serveur
- inversement, le HTML est très vite plus gros et évolue bien plus vite que le code serveur, donc les évolution du HTML fragilisent le code du serveur.

La solution est d'adopter l'approche MVC, qui sépare
- Model : entités métier,
- View, écrites avec un langage dédié à la présentation : JSP, Thymeleaf
- et Controller : code Java et framework Servlet, ou Spring MVC Controller...

---

## Introduction aux pages JSP

Les pages JSP permettent d'écrire du HTML qui soit dynamique, en lui permettant de contenir des bouts de Java exécutés côté serveur avant retour de la requête HTTP.

```jsp
<!-- index.jsp -->
<%@ page language="java" contentType="text/html;
    charset=US-ASCII" pageEncoding="US-ASCII"%>
<html>
<body>
<h1>Hello <%= request.getParameter("nom") %> !</h1>
</body>
</html>
```

http://localhost:8080/?nom=World => Hello World !

<!--
<html>
<body>
<h1>Hello World !</h1>
</body>
</html>
-->

Ce standard J2EE apparaît en 1999 en réaction à MS ASP et PHP. Mais aujourd'hui d'autres frameworks lui sont préférés, aussi ce cours n'ira pas plus loin.

---

## MVC avec servlets et JSP

Il est possible d'appliquer MVC aux servlets. Leur code d'écriture HTML est déplacé dans une JSP (ou plusieurs) qui joue le rôle de la Vue, et remplacé dans le servlet (qui est le Controller) par une indirection de traitement (*forward*) vers celle-ci :

<div class="twocols">

```java
// HelloWorldServlet.java
@Override
protected void doGet(HttpServletRequest req,
        HttpServletResponse resp) {
    String nom = req.getAttribute("nom");
    req.setAttribute("message", "Hello " + nom + " !");
    
    RequestDispatcher dispatcher = getServletContext()
        .getRequestDispatcher("/index.jsp");
    dispatcher.forward(req, resp);
}
```

<p class="break"></p>

```jsp
<!-- index.jsp -->
<%@ page language="java" contentType="text/html;
    charset=US-ASCII" pageEncoding="US-ASCII"%>
<html>
<body>
<h1><%=message%></h1>
</body>
</html>
```

=> Hello World !

<!--
<html>
<body>
<h1>Hello World !</h1>
</body>
</html>
-->

</div>

Mais... ce code de servlet reste lourd, et dans la JSP on peut toujours exécuter n'importe quel code Java ! *(une JSP est en fait transformée en servlet avant exécution)*

[
pour la petite histoire, toute JSP est en fait transformée en servlet par le serveur d'application avant d'être exécutée
]::


---

## Conclusion - un grand ancien avec des limitations

Un servlet permet d'écrire du code Java (doGet/Post()...) qui traite et répond à des appels HTTP. Mais...
- avec un modèle d'appel HTTP qui est générique et non "métier" (HttpServletRequest/Response),
- une configuration de mapping (requête HTTP vers code) peu pratique (XML) et qui reste trop grossière (par préfixe d'URL : si 2 requêtes partagent le même chemin d'URL, elles doivent être traitées par la même méthode de servlet)
- un langage de présentation (JSP) pas très adapté à MVC, comme on va le voir.

---

<!-- _class: lead -->

# Spring MVC

---

## Spring MVC

Spring MVC est le framework permettant de développer des applications web fourni par le framework Spring et Sprint Boot, sa version "tout compris" avec :
- les configurations par défaut utiles 80% du temps,
- serveur d'application Tomcat embarqué, jar exécutable...

Il adopte le paradigme MVC (séparation des éléments Model, View, Controller), dispose d'un mapping de requêtes puissant et d'utilitaires facilitant de très nombreux cas d'usage.

A noter qu'il est également utilisable pour développer de pures APIs web et se pose alors en alternative au standard J2EE JAX-RS.

---

## Les contrôleurs Spring

Une classe est déclarée comme un contrôleur Spring en utilisant l'annotation ```@Controller``` pour des pages HTML dynamiques (pour une API REST, ce serait l'annotation ```@RestController```).

```java
@Controller // pour des pages HTML dynamiques, ou sinon @RestController pour une API REST
@RequestMapping(value = "/secu") // ou @RequestMapping("/secu")
public class HelloWorldController {

//...

}
```

C'est une bonne pratique de définir sur un **Controller** un chemin d'URL de base avec l'annotation ```@RequestMapping```. Ce chemin est le préfixe commun de toutes les URLs qu'il gérera. Par exemple, ce controlleur gère toutes les requêtes d'URL commençant par ```/secu``` .

---

## Mapping de requête

Décorer une **méthode** du Controller d'une annotation ```@RequestMapping``` permet d'y rediriger les requêtes HTTP correspondant aux chemins et paramètres spécifiés dans l'annotation. On peut alors écrire dans cette méthode le code de traitement de ces requêtes HTTP.

```java
@RequestMapping(
method = RequestMethod.GET, // méthode HTTP : GET, POST, PUT, DELETE... ou @GetMapping, @PostMapping...
consumes = "application/json", // type MIME des données envoyées avec la requête : JSON, XML, form (multipart)
produces = "application/json", // type MIME des données renvoyées dans la réponse : HTML, JSON...
value = "/patients" // chemin du mapping d'URL (préfixé de l'éventuel chemin présent au niveau du controller)
)
public String getPatients() {
```

[
NB. Il n'y a pas forcément de données dans une requête (GET) ou dans une réponse (DELETE)
]::

Comme dans toute annotation Java, si value est le seul paramètre fourni, pas besoin de le nommer. Et value peut être multiple, utile par exemple pour la page d'accueil :

```java
@RequestMapping(value={"", "/", "welcome"}) // ou @RequestMapping({"", "/", "welcome"})
```

---

## Accéder aux paramètres de query HTTP

Les paramètres de query HTTP (query params) sont les paramètres qui sont passés dans l'url.

Par exemple, dans http://localhost:8080?page=2&size=10 : il y a deux paramètres, page valant à 2 et size valant 10. Pour y accéder, on utilise l'annotation ```@RequestParam``` ains :

```java
@GetMapping("/patients") // ou @RequestMapping(method = RequestMethod.GET, value = "/patients")
public String patients(
    @RequestParam("page") Integer page,
    @RequestParam("size") Integer size) {
...
}
```

Cette approche est adaptée pour les quelques paramètres d'une opération simple. S'il y a de nombreux paramètres, mieux vaut tous les modéliser dans un objet métier.

[
TODO TODO TODO exo re debug ou/et println (?)
]::

---

## Accéder aux paramètres de chemin d'URL

Une syntaxe fréquente dans une application web consiste à intégrer des paramètres directement dans l'URL comme par exemple http://localhost:8080/secu/patients/1 . Pour y accéder, on utilise l'annotation ```@PathVariable``` ainsi :

```java
@GetMapping("/patients/{id}")
// ou @RequestMapping(method = RequestMethod.GET, value = "/patients/{id}")
public String getPatient(@PathVariable(value = "id") String id) {
...
}
```

Cette notation est très fréquente dans les applications et APIs REST, où une URL est directement une référence vers une donnée (représentée par une Resource HTTP), dans laquelle les paramètres de chemin sont des bouts d'identifiants.

---

## Accéder au corps de la requête

Pour envoyer des données plus complexes, par exemple à la création ou la modification d'un objet, il est possible d'utiliser le corps de la requête HTTP (champ *data*). Il suffit pour cela de modéliser ces données sous la forme d'une classe entité, et de la mettre en argument de la méthode.

```java
@PostMapping("/patients")
// ou @RequestMapping(method = RequestMethod.POST, value = "/patients")
public String createPatient (Patient patient) {
...
}
```

Les données serialisées doivent logiquement correspondre à cette entité. Sinon, les données seront récupérables sous la forme d'une ```Map```.

NB. dans le cas d'une API REST, on envoie dans ce champ des données structuées en JSON ou en XML, et on peut rajouter l'annotation ```@RequestBody``` à l'argument.

[
=> TODO databinding (donc form th: ?! OU? @RequestMapping) et @Valid et (affichage) errors
& reste MVC : ModelMap, return "view"
]::

---

<!-- _class: lead -->

# Thymeleaf

---

## Thymeleaf - Agenda

- Présentation et principes
- Injection de contenu et attributs
- Formulaires
- Objets et expressions
- Conditions et itérations
- Fragments

---

## Présentation de Thymeleaf

Le plus utilisé des moteurs de templates web en Java, du fait de son utilisation au sein du framework Spring. Il permet le développement de pages HTML dynamiques suivant l'approche MVC, **jouant le rôle de la Vue à côté de Controllers Spring MVC**.

![height:100px center](binaries/thymeleaf_logo_white.png)

NB. les alternatives pour faire du web en Java sont (outre JSP, standard obsolète) :
- un autre moteur de rendu / templating, tel Freemarker
- une architecture web de type AJAX, avec un serveur d'API REST en Spring MVC ou J2EE JAX-RS et une IHM HTML utilisant React.js, Angular...
- NB. le standard J2EE comprend un framework MVC mais orienté composant : JSF, hélas plus très utilisé

---

## Exemple Spring MVC + Thymeleaf

<div class="twocols">

```java
// HomeController.java :
@Controller
public class HomeController {
    
    @GetMapping(value = "/")
    public String home(final ModelMap model) {
        model.put("message", "Hello World !");
        return "index";
    }
}
```

<p class="break"></p>

```html
<!-- index.html -->
<html lang="en" xmlns:th="http://www.thymeleaf.org">
  <head>
      <title>Hello World Thymeleaf</title>
  </head>
  <body>
  <h1><span th:text="${message}"></span></h1>
  </body>
</html>
```

http://localhost:8080/ => Hello World !

<!--
<html lang="en" xmlns:th="http://www.thymeleaf.org">
  <head>
      <title>Hello World Thymeleaf</title>
  </head>
  <body>
  <h1><span>Hello World !</span></h1>
  </body>
</html>
-->

</div>

- le **mapping** de requête de Spring MVC est réutilisé (mais avec ```@Controller```)
- la **Vue** à appliquer est indiquée en retour de méthode (ici "index", par défaut suffixée par ".html")
- les objets à passer à la vue doivent être mis dans une ```ModelMap``` à déclarer en argument

---

## RAPPEL - MVC (motif de conception), avec Thymeleaf

Inclure le code métier dans la vue est la cause d'une application fragile :
```html
<div>Nombre de patients : <span th:text="${patientService.countPatient()}"></span></div>
```

Le motif de conception MVC désigne la bonne pratique de séparer Modèle, Vue et Contrôleur, notamment dans une architecture web :
```html
<!-- patients.html (la Vue : la couche de présentation, en HTML+Thymeleaf) -->
<div>Nombre de patients : <span th:text="${patientCount}"></span></div>
```

```java
// PatientController.java (le Contrôleur : le code "métier", en Java Spring MVC)
@GetMapping("/patients")
public String patients(ModelMap model) {
    // on remplit le Modèle :
    model.put("patientCount", patientService.countPatient());
}
```

---

## Afficher des variables dans le HTML

Mettre des objets (entités métier...) dans le ```ModelMap``` permet de les rendre disponible en tant que variables lors du rendu du template Thymeleaf vers le HTML qui sera renvoyé au client web.

Il y a 3 manières de faire afficher par Thymeleaf des variables dans la page HTML :
- au sein d'une balise HTML
- inline (sans balise)
- HTML

---

## Afficher des variables - par attribut au sein d'une balise

<div class="twocols">

```java
// HomeController.java :
@Controller
public class HomeController {
    @GetMapping(value = "/")
    public String home(final ModelMap model) {
        model.put("greeting", "Hello");
        model.put("nom", "World");
        return "index";
    }
}
```

<p class="break"></p>

```html
<!-- index.html -->
<html lang="en" xmlns:th="http://www.thymeleaf.org">
  <body>
  <h1>
      <span th:text="${greeting}">Greetings</span> 
      <span data-th-text="${nom}">John</span> !
  </h1>
  </body>
</html>
```

http://localhost:8080/ => Hello World !

<!--
<html lang="en" xmlns:th="http://www.thymeleaf.org">
  <head>
      <title>Hello World Thymeleaf</title>
  </head>
  <body>
  <h1><span>Hello World !</span></h1>
  </body>
</html>
-->

</div>

- Thymeleaf remplace le contenu de la balise par le rendu d'un attribut
- l'attribut ```th:text``` est la pratique usuelle Thymeleaf. Il faut déclarer ce préfixe en en-tête pour avoir du XHTML.
- l'attribut ```data-th-text``` est une version qui suit davantage les bonnes pratiques HTML (pas de conflits éventuels dans le futur)

[
afficher la variable par attribut au sein d'une balise HTML (souvent span, div, p...)
data- : https://stackoverflow.com/questions/31562007/why-do-we-need-prefix-data-for-custom-data-attributes-in-html5
    It guarantees there will not be any clashes with extensions to HTML in future editions.
    They provide a clear indication of which attributes are custom attributes, and which ones are standardised attributes.
    More convenient DOM API for accessing these attributes from scripts. (mais ce n'est pas un argument si on utilise un framework qui cache le DOM ex.React, d'ailleurs en général le choix ou non de cette syntaxe est à faire selon l'avis du framework utilisé)
]::

---

## Avantage : développement graphique statique

- cette méthode de passer la variable ou expression à rendre en attribut est l'**avantage de Thymeleaf** : permettre un HTML aussi lisible que possible même sans le rendu.
- Cela facilite le développement graphique (charte, mise en page) sous forme de pages statiques dans un premier temps (avant tout Controller), y compris par des designers **sans connaissances Thymeleaf** (*"Natural Templating"*).
- En effet, en partant de pages web complètes mais encore statiques, il est facile de les rentre dynamiques. Il suffit d'y rajouter des attributs ```th:text``` (et si nécessaire pour cela des balises non impactantes telle <span>), avec une valeur pouvant être littérale dans un premier temps :

```html
<h1>Nom : Jean</h1> <!-- => -->
```

```html
<h1>Nom : <span th:text="'Jean'">Jean</span></h1>
```

[
TODO TODO TODO exo tdtp 26 => th:text
]::

---

## Afficher des variables - HTML


<div class="twocols">

```java
// HomeController.java :
@Controller
public class HomeController {
    
    @GetMapping(value = "/")
    public String home(final ModelMap model) {
        model.put("htmlMessage", "<h1>Hello World !</h1>");
        return "index";
    }
}
```

<p class="break"></p>

```html
<!-- index.html -->
<html lang="en" xmlns:th="http://www.thymeleaf.org">
  <head>
      <title>Hello World Thymeleaf</title>
  </head>
  <body>
  <span th:utext="${htmlMessage}"></span>
  </body>
</html>
```

http://localhost:8080/ => Hello World !

<!--
<html lang="en" xmlns:th="http://www.thymeleaf.org">
  <head>
      <title>Hello World Thymeleaf</title>
  </head>
  <body>
  <h1><span>Hello World !</span></h1>
  </body>
</html>
-->

</div>

- le HTML passé est injecté sous la balise tel quel, sans échappement contrairement à la précédente méthode
- à utiliser quand des données disponibles côté serveur sont du HTML : stocké (car rentré par un input de type ```richtext``` ou équivalent...) ou généré (par un service externe à l'application, ou depuis une syntaxe Markdown...) côté serveur

---

## Afficher des variables - inline (sans balise)

<div class="twocols">

```java
// HomeController.java :
@Controller
public class HomeController {
    
    @GetMapping(value = "/")
    public String home(final ModelMap model) {
        model.put("nom", "World");
        model.put("htmlMessage", "<h1>Hello Me !</h1>");
        return "index";
    }
}
```

<p class="break"></p>

```html
<!-- index.html -->
<html lang="en" xmlns:th="http://www.thymeleaf.org">
  <head>
      <title>Hello World Thymeleaf</title>
  </head>
  <body>
  <h1>Hello [[${nom}]] !</h1>
  [[${htmlMessage}]]
  </body>
</html>
```

http://localhost:8080/ => Hello World !

<!--
<html lang="en" xmlns:th="http://www.thymeleaf.org">
  <head>
      <title>Hello World Thymeleaf</title>
  </head>
  <body>
  <h1>Hello World !</h1>
  <h1>Hello Me !</h1>
  </body>
</html>
-->

</div>

- syntaxe : ```[[...]] ```déclenche le rendu de son contenu par Thymeleaf avec échappement (comme ```th:text```), ```[(...)]``` le fait sans échappement (comme ```th:utext```)
- utile quand rajouter une balise HTML pose problème (casserait la mise en page...)
- à éviter sinon, car moins lisible sans rendu et Natural Templating pas possible.

---

## Attributs HTML et URLs

<div class="twocols">

```java
@Controller
public class PatientController {
    @GetMapping(value = "/patients/1")
    public String getPatient(final ModelMap model) {
        model.put("id", 1);
        model.put("nom", "Jean");
        model.put("ex", sexe == "M" ? "John" : "Anna");
        return "patients";
    }
}
```

<p class="break"></p>

```html
<html lang="en" xmlns:th="http://www.thymeleaf.org">
  <body>
    <input th:value="${id}" /><!-- => value="1" -->
    <input th:attr="value=${nom},placeholder=${ex}" />
    <!-- => value="Jean" placeholder="John" -->
    <a th:href="@{/patients(id=${id})}">permalien 1</a>
    <!-- => href="/patients?id=1" -->
    <a th:href="@{/patients/{id}}">permalien 2</a>
    <!-- => href="/patients/1" -->
  </form>
  </body>
</html>
```

</div>

- un attribut est injecté en le préfixant par ```th:``` (ou par ```th:attr``` si les attributs ne sont pas connus à l'avance)
- une URL est injectée avec la syntaxe : ```@{prefix/{pathParamValue}(queryParam1=${value1},queryParam2=${value2}}```. Attention, si les valeurs sont plutôt des expressions Thymeleaf, il faut activer leur preprocessing en encadrant celles-ci (ou tout) par ```__...__``` ou ```|...|```

[
@{} PAS ASSEZ pour path param, voir FAQ
th:attr moins bien, que pour dynamiques
]::

---

## Formulaires

L'injection d'attributs et URL permettent de développer des formulaires HTML ainsi.

<div class="twocols">

```java
@Controller
public class PatientController {
    @Autowired
    PatientService patientService;
    // même getPatient()
    @PostMapping(value = "/patients/1")
    public String savePatient(Patient patient) {
        // patient a été rempli des données du POST
        patientService.save(patient);
        return "redirect:/patients/" + patient.getId();
        // redirige vers un HTTP GET /patients/1
    }
}
```

<p class="break"></p>

```html
<html lang="en" xmlns:th="http://www.thymeleaf.org">
  <body>
  <form id="save" method="post"
        action="@{/patients/{id}}">
    <!-- => action="/patients/1" -->
    <input name="id" th:value="${id}" />
    <input name="nom" th:value="${nom}" />
    <input type="Submit" value="Enregistrer" />
  </form>
  </body>
</html>
```

</div>

- les données envoyées en POST sont automatiquement désérialisées dans l'entité en argument (**databinding**), au lieu d'un argument générique ModelMap
- noter en fin de traitement de POST la redirection vers un GET

[
avec comme clé l'attribut "name" de l'input
]::

---


## Formulaires - avec entité aussi en GET

Il est plus pratique d'utiliser le modèle de l'entité également dans le GET, réécrivons-le :

<div class="twocols">

```java
@Controller
public class PatientController {
  @Autowired
  PatientService patientService;
  @GetMapping(value = "/patients/{id}")
  public String getPatient(final ModelMap model,
        @PathVariable Long id) {
    Patient patient = patientService.getPatient(id);
    model.put("patient", patient);
    return "patients";
  }
  // même savePatient()
}
```

<p class="break"></p>

```html
<html lang="en" xmlns:th="http://www.thymeleaf.org">
  <body>
  <form id="save" method="post"
        action="@{/patients/{patient.id}}">
    <!-- => action="/patients/1" -->
    <input name="id" th:value="${patient.id}" />
    <input name="nom" th:value="${patient.nom}" />
    <input type="Submit" value="Enregistrer" />
  </form>
  </body>
</html>
```

</div>

---

## Facilités th:object et th:field

[
Une autre syntaxe est possible pour les formulaires, plus spécifique, incluant du databinding facilitant notamment l'affichage des erreurs de lecture des données envoyées (parsing). La syntaxe th:object="obj">...<span th:text="*{obj.anAttr}"> est utilisable indépendamment.
@ModelAttribute
]::

<!--
- La syntaxe ```<... th:object="obj">...<... th:text="*{unChampDObj}">``` (notez ```*```) déclare puis réutilise une variable locale. Elle est utilisable en dehors des formulaires.
-->

- Déclarer une *variable locale* avec la syntaxe ```<... th:object="monObj">``` permet en-dessous de référencer ses champs avec ```<... th:text="*{unChampDeMonObj}">``` (notez ```*```). Elle est utilisable en dehors des formulaires.
- Dans les formulaires, utiliser en-dessous en plus ```<input th:field="*{unChampDeMonObj}"``` remplit les 3 champs name, id et value.

```html
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<body>
<form id="save" method="post"
      th:object="${patient}" action="@{/patients/{id}}">
  <!-- => action="/patients/1" -->
  <input name="id" id="id" th:value="*{id}" /><!--sans th:field-->
  <input th:field="*{nom}" /><!--=> name="id" id="id" value="1"-->
  <input type="Submit" value="Enregistrer" />
</form>
</body>
</html>
```

---

## Accéder aux erreurs côté client et serveur

- les erreurs de désérialisation s'accédent côté serveur en ajoutant un argument ```BindingResult```. Voire les erreurs d'annotations de validation Hibernate avec ```@Validation Patient patient``` (en ajoutant la dépendance à ```spring-boot-validation``` dans pom.xml et ```@NotBlank(message="Requis !") private String nom;``` dans Patient)
- et côté client web pour affichage par ```#fields.errors('monChamp')``` (ou ```...(*)```) si ```th:field``` a été utilisé, et sinon en passant explictement le ```BindingResult``` à la vue

<div class="twocols">

```java
@Controller
public class PatientController {
  // ...
  @PostMapping(value = "/patients/1")
  public String savePatient(Patient patient,
        BindingResult bindingResult) {
    if (bindingResult.hasErrors()) {
        System.out.println("problème de désérialisation");
    }
    patientService.save(patient);
    ...
```

<p class="break"></p>

```html
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<body>
<form id="save" method="post"
      action="@{/patients/{id}}">
  <input name="id" id="id" th:value="${patient.id}" />
  <input name="nom" id="nom" th:value="${patient.nom}" />
  <ul th:each="err : ${#fields.errors('nom')}"><!-- ou ('*') -->
    <li th:text="${err}"/></li>
  </ul>
  <input type="Submit" value="Enregistrer" />
</form>
</body>
</html>
```

</div>

[
th:field : ="*{nom}" (noter le "*" !) generates the name, id, and value of the field it's on, requiert th:object, https://stackoverflow.com/questions/69388623/invalid-generation-of-input-fields-id-and-name-attribute-in-thymeleaf-with-js
plus dur pour dynamique (th:field="*{__${myVar}__}") mais sinon TODO Q ignore bean binding (i.e. mettre value à la main) https://stackoverflow.com/questions/25027801/how-to-set-thymeleaf-thfield-value-from-other-variable
https://www.baeldung.com/thymeleaf-in-spring-mvc
]::
<!--
7. Handling User Input
   We can handle form input using the th:action=”@{url}” and th:object=”${object}” attributes. We use th:action to provide the form action URL and th:object to specify an object to which the submitted form data will be bound.
   Individual fields are mapped using the th:field=”*{name}” attribute, where the name is the matching property of the object.
+ Afficher les erreurs de validation : <ul><li th:each="err : ${#fields.errors('id')}" th:text="${err}" />...</ul>
https://www.baeldung.com/spring-thymeleaf-error-messages
-->

---

## Objets et expressions

Thymeleaf couvre la plupart des besoins usuels au cours du développement de templates, en mettant à disposition lors de leur écriture de nombreux utilitaires et en offrant une syntaxe d'expression riche et puissante :

- Les objets du contexte web 
- Les objets utilitaires (messages, formatage...)
- Expressions simples et litérales 
- Opérations, comparaisons et condition ternaires

---

## Accès aux objets du contexte web (Servlet !)

On peut accéder aux objets du contexte web sous-jacent : HttpServletRequest par ```#request```, HttpSession par ```#session``` et ServletContext par ```#servletContext```.
```param```, ```session``` et ```application``` sont des raccourcis pour accéder à leurs données. De nombreux autres existent : ```#ctx```, ```#vars```, ```#locale```, ```#response```...

<div class="twocols">

```java
@Controller
public class HomeController {
  @Autowired
  private ServletContext context;
  @GetMapping(value = "/")
  public String home(final ModelMap m,
    HttpSession s, HttpServletRequest r) {
    context.setAttribute("greet", "Hello");
    // ou
    r.getServletContext().setAttribute("greeting", "Hello");
    s.setAttribute("adjectif", "Cool");
    m.put("suffix", "!");
    return "index";
  }
}
```

<p class="break"></p>

http://localhost:8080/?nom=World

```html
<span th:text="${#servletContext.getServerInfo()}">Tomcat</span>
<span th:text="${#servletContext.getAttribute('greet')}">
  Hello</span>
<span th:text="${application.greet}">Hello</span>

<span th:text="${#session.getAttribute('adjectif')}">Cool</span>
<span th:text="${session.adjectif}">Cool</span>
<span th:text="${#session.id}">
  BA7CD710FE9610F1CEF5174DFB3C04AE</span>

<span th:text="${#request.getParameter('nom')}">World</span>
<span th:text="${param.msg}">World</span>
<span th:text="${param.msg[0]}">World</span>

<span th:text="${#request.getAttribute('suffix')}">!</span>
<span th:text="${#request.getRequestURI()}">/</span>
```

</div>

[
TODO TODO TODO exo car facile ??? si pas tdtp
]::

---

## Objets utilitaires pratiques (messages, formattage...)

[
=>
#calendars, #objects, #bools, #arrays
TODO TODO TODO exo messages, voire localisation (ET NON i18n) ; voire #temporals.format() https://www.baeldung.com/spring-boot-internationalization
]::
<!--
<p th:text="#{home.welcome}">Welcome to our grocery store!</p>
/WEB-INF/templates/home(_en/fr(_FR)).properties
-->

```#messages```(localisation/i18n), ```#temporals``` (date/heure), ```#strings```, ```#numbers```, ```#lists```...

<div class="twocols">

```properties
# application[(_en/fr_FR)].properties
msg.hi=Hello
msg.hi2=Greetings
```

```java
@Controller
public class HomeController {
  @GetMapping(value = "/")
  public String home(final ModelMap m) {
    m.put("now", LocalDate.now());
    m.put("codePostal", 07700);
    m.put("montant", 2500);
    m.put("reduction", 0.2034);
    m.put("liste", Arrays.asList(9, 7, 5));
    return "index";
  }
}
```

<p class="break"></p>

http://localhost:8080/?nom=World

```html
<span th:text="${#messages.msg('msg.hi')}">Hello</span>
<span th:text="${#messages.msg('msg.hi2', 'KO')}">Hello</span>

<span th:text="${#temporals.format(now, 'dd/MM/YYYY')}">
  21/12/2022</span>

<span
  th:text="${#strings.defaultString(notDefined,'-')}">-</span>
<span th:text="${#strings.containsIgnoreCase(param.nom,'rL')}">
  true</span>
<span th:text="${#strings.toUpperCase(param.nom)}">WORLD</span>

<span th:text="${#numbers.formatInteger(codePostal, 5)}">
  07700</span>
<span th:utext="${#numbers.formatCurrency(montant)}">
  2 500,00 €</span>
<span th:utext="${#numbers.formatPercent(reduction, 1, 0)}">
  20 %</span>

<span th:utext="${#lists.size(liste)}">3</span>
<span th:utext="${#lists.sort(liste)}">[5, 7, 9]</span>
```

</div>

[
TODO TODO TODO exo car facile ??? si pas tdtp
]::

---

## Expressions simples et litérales

On peut utiliser du texte, des nombres, des booléens ou même null dans les expressions Thymeleaf. Voici quelques exemples.

http://localhost:8080/?nom=World
=>

```html
<span th:text="'Hello ' + ${param.nom}">Hello World</span>
<span th:text="|Hello ${param.nom}|">Hello World</span>
<span th:text="__Hello ${param.nom}__">Hello World</span>
<span th:text="10 * 2">20</span>
<span th:text="false == true">false</span>
<span th:text="${param.ko} == null">true</span>
```

Notez l'activation du preprocessing en encadrant tout ou partie des expressions par ```__...__``` ou ```|...|```. C'est utile pour bâtir des URLs (dans th:href/action...) plus lisiblement qu'en concaténant avec des ```+```.

---

## Opérations, comparaisons et conditions ternaires

Il est possible, à l'intérieur d'une expression, de réaliser des opérations arithmétiques (+, -, *, /, %), des comparaisons ou des conditions ternaires :

<div class="twocols">

```java
@Controller
public class HomeController {
  @GetMapping(value = "/")
  public String home(final ModelMap m) {
    m.put("prix", 100.0);
    m.put("tva", 20);
    return "index";
  }
}
```

<p class="break"></p>

```html
<span th:text="${prix} * (100- ${tva})/100">120</span>
<span th:text="${prix} % 3">1</span>
<span th:text="${prix} - 10">90</span>

<span th:text="${tva} &gt; 10">true</span>
<span th:text="${prix} &lt; 150">true</span>

<span th:text="${tva} == null ? 'HT'">HT</span>
<span th:text="${tva} == 20 ? 'TTC' : 'HT'">TTC</span>

<span th:text="${prix} ?: 'manquant'">manquant</span>
<span th:text="${prix} ?: _">manquant</span>
```

</div>

Notez la possibilité d'*expressions par défaut* utilisant l'opérateur Elvis (```?:```), un peu à la manière des monades.

---

## Mise en page dynamique - conditions et itérations

Jusqu'à présent, nous avons comment afficher avec Thymeleaf uniquement des choses à l'intérieur d'une balise HTML.

Nous allons à présent voir comment il est possible d'avoir une mise en page (**layout**) dynamique, en contrôlant l'affichage de blocs ou fragments de (balises) HTML, et notamment :
- afficher un fragment de HTML en fonction d'une **condition**
- afficher un fragment de HTML plusieurs fois (**itération**).

---

## Affichage conditionnel : th:if/unless, th:switch/case

L'affichage conditionnel de HTML est utile pour désactiver l'affichage de portions complètes de page HTML (droits insuffisants...), ou pour le faire varier (selon la catégorie de données, ou le profil utilisateur...).

[
=> ça pour ce qui change l'IHM, et réserver les expressions ex. pour le formatage
]::

<div class="twocols">

```java
@Controller
public class ProduitController {
  @GetMapping(value = "/produits/1")
  public String getProduit(final ModelMap m) {
    m.put("id", 4);
    m.put("type", "book");
    m.put("tags", Arrays.asList("policier", "SF"));
    return "index";
  }
}
```

<p class="break"></p>

```html
<div th:switch="${id} != null">
  <a href="/update" th:case="true">Enregistrer</a>
  <a href="/create" th:case="*">Créer</a>
</div>
<a href="/details"
   th:if="${not #lists.isEmpty(tags)}">Détail...</a>
```

=>

```html
<div>
  <a href="/update">Enregistrer</a>
</div>
<a href="/details">Détail...</a>
```

</div>

---

## Itérations : th:each

```th:each``` permet de répéter un fragment de HTML pour tout élément d'une liste (java.util.List/Iterable/Enumeration/Map ou tableau primitif) :

<div class="twocols">

```java
@Controller
public class ProduitController {
  @GetMapping(value = "/produits/1")
  public String getProduit(final ModelMap m) {
    m.put("id", 4);
    m.put("type", "book");
    m.put("tags", Arrays.asList("policier", "SF"));
    return "index";
  }
}
```

<p class="break"></p>

```html
<ul th:each="tag : ${tags}">
  <li><a th:text="${tag}"
    th:href="@{/tags(tag=${tag})}"></a></li>
</ul>
```

=>

```html
<ul>
  <li><a href="/tags?tag=policier">policier</a></li>
  <li><a href="/tags?tag=SF">SF</a></li>
</ul>
```

---

## Mise en page dynamique - les fragments Thymeleaf

Dans Thymeleaf, la notion de **fragment** permet de réutiliser des portions d'HTML dans une application, par exemple : un header, un footer, une barre de menu...

Cela se fait en définissant des fragments (blocs) de HTML puis en y faisant référence pour les inclure dans un template.

- Définition et inclusion
- Les fragments paramétrés
- Fonctionnalités avancées

---

## Définition et inclusion de fragments

On définit un bloc de HTML en tant que fragment Thymeleaf, en déclarant son nom dans l'attribut ```th:fragment``` de sa balise racine. On peut alors l'inclure dans un autre template avec l'attribut ```th:insert/replace="~{templateDuFragment :: monFragment}"```

<div class="twocols">

```html
<!-- common.html contenant les fragments partagés -->
<html xmlns:th="http://www.thymeleaf.org">
  <body>
    <div class="nav-div" th:fragment="menu">
      <ul class="nav">
        <li class="nav-item">
          <a class="nav-link" href="#">Accueil</a>
        </li>
        ...
      </ul>
    </div>
    <div class="footer" th:fragment="footer">
      Contact : Jean Dupont
    </div>
  </body>
</html>
```

<p class="break"></p>

```html
<!-- ma_page.html -->
<html xmlns:th="http://www.thymeleaf.org">
  <body>
    <div class="container"th:insert="~{common :: menu}"></div>
    ...
    <div class="container" th:replace="~{common :: footer}"></div>
  </body>
</html>
<!-- => -->
<html>
<body>
  <div class="container">
    <div class="nav-div">
      <ul class="nav">...</ul>
    </div>
  </div>
  ...
  <div class="footer">Contact : Jean Dupont</div>
</body>
</html>
```
</div>

---

## Fragments paramétrés

Définir des paramètres dans ```th:fragment``` permet de rendre le fragment davantage dynamique :

[
Il est possible de rendre dynamique le contenu d'un fragment, en définissant des paramètres sur le fragment.
Lorsqu'on insère un fragment paramétré, il suffit de spécifier les paramètres.
]::

<div class="twocols">

```java
@Controller
public class HomeController {
  @GetMapping(value = "/")
  public String home(final ModelMap m) {
    m.put("elts", Arrays.asList("Accueil", "Patients"));
    return "index";
  }
}
```

```html
<!-- common.html -->
<html xmlns:th="http://www.thymeleaf.org">
<body>
  <div th:fragment="menu(title, elements)">
    <h3 th:text="${title}"></h3>
    <ul th:each="element : ${elements}" class="nav">
      <li class="nav-item">${element}</li>
    </ul>
  </div>
</body>
</html>
```

<p class="break"></p>

```html
<!-- ma_page.html -->
<html xmlns:th="http://www.thymeleaf.org">
  <body>
    <div th:insert="~{common :: menu('Menu', ${elts})}"></div>
  </body>
</html>
<!-- => -->
<html>
<body>
  <div>
    <h3>Menu</h3>
    <div class="nav-div">
      <ul class="nav">
        <li class="nav-item">Accueil</li>
        <li class="nav-item">Patients</li>
      </ul>
    </div>
  </div>
</body>
</html>
```

</div>

---

## Définition et inclusion de fragments - détails

- ```th:insert``` rajoute en-dessous de la balise, alors que ```th:replace``` remplace la balise
- entourer la référence à un fragment par ```~{...}``` en fait une **expression de fragment** que Thymeleaf va évaluer comme telle, et qui ouvre de nombreuses possibilités supplémentaires. Exemples :
  - ```~{common :: menu}``` (dans ce cas, uniquement ```common :: menu``` marcherait aussi)
  - ```~{ common :: menu(~{::h3},~{common::nav}) }``` où h3 est la balise fournie sous la balise d'inclusion et common::nav un fragment défini par ailleurs.
- en étape de design, les fragments aussi peuvent être développés en Natural Templating, étant définis au sein de pages entières

---

## Fonctionnalités supplémentaires

Les fragments ont d'autres fonctionnalités encore plus puissantes pour faciliter la réutilisation :
- insertion conditionnelle de fragments
- "Flexible Layouts" (i.e. passage de **blocs de HTML en paramètre de fragment**)
- Suppression de fragments 
  - ex. de données d'exemple (mock) au rendu i.e. en dehors de l'étape de design en Natural Templating
  - peu utile, possible avec ex. affichage conditionnel ou flexible layouts...
- Héritage de fragments (une page HTML entière comme fragment paramétré)

Si besoin, la documentation détaillée en est disponible à https://www.thymeleaf.org/doc/tutorials/3.1/usingthymeleaf.html#template-layout