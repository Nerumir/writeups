# Party Time ! - Quentin-AC

Le texte du challenge est le suivant :
> This party house is known for its 3AM outings, but you've gotta work for the location if you want to come! Enter the GPS coordinates of the location!

On nous donne un fichier HEIC (image Apple donc) représentant la façade d'une maison : [Lien de l'image](data/IMG_4048.HEIC)

En allant dans les propriétés du fichier depuis l'explorateur de fichiers (Alt + Entrée), on obtient dans l'onglet `Détails` les coordonnées GPS de la photo:

![Coordonnées GPS de la photo](data/Party%20Time!.png)

On peut donc construire le flag avec la latitude et la longitude de la forme suivante : `swampCTF{xx.xx.xx,xx.xx.xx}`.
