Voici l'énoncé du challenge :
> I’ve been looking for well-rated ice-skating rinks, I wonder if there is one out there with decent reviews?
>
> The flag format is swampCTF{...}. You will not need to wrap it yourself.

On nous fournit également une image noire, de laquelle on ne peut pas tirer grand-chose à première vue. L'analyse des détails révèle cependant un titre chiffré en hexadécimal :

![Détails et titre en hexadécimal](../data/On%20Thin%20Ice%20hex.png)

Ce titre déchiffré donne `Ӧкмысӧд воськов. Мездлун.`, que Google Traduction traduit comme du Komi sgnifiant "Étape huit. Liberté" :

![Google Traduction](../data/On%20Thin%20Ice%20Translate.png)

Une recherche Google permet de comprendre qu'il s'agit d'une étape [d'une mission de Call of Duty: Black Ops](https://callofduty.fandom.com/wiki/Vorkuta_(level)#Walkthrough). Cette mission se déroule dans la ville de Vorkuta, dans la république des Komis.

Une recherche Maps nous permet de trouver cette ville, ainsi que sa patinoire, et [un commentaire](https://maps.app.goo.gl/LBj1bdenR5RjRoMBA) comportant le flag de la forme `swampCTF{xxx}`.
