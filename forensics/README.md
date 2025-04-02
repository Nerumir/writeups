# Hackali - Forensics

Vous trouverez ici tous les writeups axés sur **l'analyse des données numériques** et **les investigations forensiques**.  
Si vous souhaitez découvrir de **nouvelles techniques d’investigation numérique** ou apprendre de **nouvelles astuces**, alors n’hésitez pas à les parcourir !

---

## Writeups disponibles

- [MuddyWater](writeups/MuddyWater/MuddyWater.md)
  Dans ce writeup, j'ai analysé un **pcap** d'une intrusion sur un **Domain Controller**. En filtrant **NTLM** avec **Wireshark**, j'ai repéré les tentatives de **bruteforce** et isolé la **session setup réussie**. Ensuite, j'ai reconstruit le **hash NTLMv2** en corrigeant son format et utilisé **Hashcat** pour le cracker.
- [ProtoProto](writeups/ProtoProto/ProtoProto.md)
  Dans ce writeup, j'ai analysé un **pcap** contenant des communications **UDP** entre un client et un serveur utilisant un protocole personnalisé. En examinant les échanges en **hexadécimal**, j'ai reverse-engineered le protocole pour comprendre comment demander un fichier au serveur. J'ai ensuite créer un **script Python** pour reproduire la requête et récupérer le flag.
- [Preferencial Treatment](writeups/PreferenciaTreatment/PreferencialTreatment.md)
   Dans ce writeup, j'ai exploré un fichier .pcap provenant d'une instance Windows Server, dont le mot de passe avait été oublié. Grâce à Wireshark, j'ai filtré le trafic pour isoler les échanges NTLMSSP et suivi un flux TCP qui m'a mené à un champ caché, le cpassword, dissimulé dans un fichier XML de GPP. En utilisant gpp-decrypt, j'ai pu décrypter ce mot de passe et finalement révéler le flag.
