# Preferential Treatment - dtss

Nous avons un fichier `.pcap` et une consigne 
> "We have an old Windows Server 2008 instance that we lost the password for. Can you see if you can find one in this packet capture?"

**J'ouvre** le fichier `.pcap` avec **Wireshark**, on voit des échanges sur différents protocoles.

![echanges](data/échanges.png)

> [!NOTE]
> En consultant à nouveau la consigne, j'essaie de repérer sur quel **protocole d'authentification** les échanges ont eu lieu. 
> Je filtre sur Wireshark avec **ntlmssp** (NT LAN Manager Security Support Provider), qui est le protocole d'authentification utilisé principalement dans les environnements Windows.

![Filter NTLMSSP](data/filter.png)

--> Je trouve trois paquets ; le troisième paquet est la requête de **Session Setup AndX**, qui est la réponse au challenge et permet de s'authentifier. Pour en savoir plus sur NTLM, consultez la [Présentation du protocole](https://www.crowdstrike.com/fr-fr/cybersecurity-101/identity-protection/windows-ntlm/)

Je fais un clic droit sur le paquet concerné et sélectionne Follow > TCP Stream.

![XML Stream](data/stream.png)

En explorant cette requête je trouves un formulaire **xml** qui renseigne un "cpassword"

```yml
cpassword="dAw7VQvfj9rs53A8t4PudTVf85Ca5cmC1Xjx6TpI/cS8WD4D8DXbKiWIZslihdJw3Rf+ijboX7FgLW7pF0K6x7dfhQ8gxLq34ENGjN8eTOI="
```

Ce **"cpassword"** est un mot de passe chiffré en Base64 qui a été stocké dans un fichier XML de GPP, j'utilises gpp-decrypt sur kali qui permet de décoder ce type de mot de passe.

![Flag](data/flag.png)


Je trouve le flag : **swampCTF{4v3r463_-------_--------}**
