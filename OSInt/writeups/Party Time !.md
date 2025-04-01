Le texte du challenge est le suivant :\
`This party house is known for its 3AM outings, but you've gotta work for the location if you want to come! Enter the GPS coordinates of the location!`

On nous donne un fichier HEIC (image Apple donc) représentant la façade d'une maison : [Lien de l'image](https://github.com/Nerumir/writeups/blob/9706837ebbe1ab822a354c292e7667347384d7c3/OSInt/data/IMG_4048.HEIC)

En allant dans les propriétés du fichier depuis l'explorateur de fichiers (Alt + Entrée), on obtient dans l'onglet `Détails` les coordonnées GPS de la photo:\
![Coordonnées GPS de la photo](https://github.com/user-attachments/assets/442a08fe-efe8-40aa-9cd7-8ebbc74d6ad9)

On peut donc construire le flag : `swampCTF{29.39.10,82.19.59}`.
