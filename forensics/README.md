# Hackali - Forensics

Vous trouverez ici tous les writeups axés sur **l'analyse des données numériques** et **les investigations forensiques**.  
Si vous souhaitez découvrir de **nouvelles techniques d’investigation numérique** ou apprendre de **nouvelles astuces**, alors n’hésitez pas à les parcourir !

---

## Writeups disponibles

- [MuddyWater](writeups/MuddyWater/MuddyWater.md)
  Dans ce writeup, j'ai analysé un **pcap** d'une intrusion sur un **Domain Controller**. En filtrant **NTLM** avec **Wireshark**, j'ai repéré les tentatives de **bruteforce** et isolé la **session setup réussie**. Ensuite, j'ai reconstruit le **hash NTLMv2** en corrigeant son format et utilisé **Hashcat** pour le cracker.
- [ProtoProto](writeups/ProtoProto/ProtoProto.md)
  Dans ce writeup, j'ai analysé un **pcap** contenant des communications **UDP** entre un client et un serveur utilisant un protocole personnalisé. En examinant les échanges en **hexadécimal**, j'ai reverse-engineered le protocole pour comprendre comment demander un fichier au serveur. J'ai ensuite créer un **script Python** pour reproduire la requête et récupérer le flag.
