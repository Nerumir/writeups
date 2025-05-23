# Hackali - SwampCTF 2025

Vous trouverez ici tous les writeups du SwampCTF 2025.  

---

## Writeups disponibles

- [MuddyWater](writeups/MuddyWater/MuddyWater.md) :star::star: : Dans ce writeup, j'ai analysé un **pcap** d'une intrusion sur un **Domain Controller**. En filtrant **NTLM** avec **Wireshark**, j'ai repéré les tentatives de **bruteforce** et isolé la **session setup réussie**. Ensuite, j'ai reconstruit le **hash NTLMv2** en corrigeant son format et utilisé **Hashcat** pour le cracker.
- [ProtoProto](writeups/ProtoProto/ProtoProto.md) :star::star: : Dans ce writeup, j'ai analysé un **pcap** contenant des communications **UDP** entre un client et un serveur utilisant un protocole personnalisé. En examinant les échanges en **hexadécimal**, j'ai reverse-engineered le protocole pour comprendre comment demander un fichier au serveur. J'ai ensuite créer un **script Python** pour reproduire la requête et récupérer le flag.
- [Preferencial Treatment](writeups/PreferentialTreatment/PreferentialTreatment.md) :star: : Dans ce writeup, j'ai exploré un fichier .pcap provenant d'une instance Windows Server, dont le mot de passe avait été oublié. Grâce à Wireshark, j'ai filtré le trafic pour isoler les échanges NTLMSSP et suivi un flux TCP qui m'a mené à un champ caché, le cpassword, dissimulé dans un fichier XML de GPP. En utilisant gpp-decrypt, j'ai pu décrypter ce mot de passe et finalement révéler le flag.
- [Party Time!](./writeups/PartyTime/Party%20Time!.md) :star: : extraction de données publiques d'une photo 
- [Party Time! Level 2](./writeups/PartyTime2/Party%20Time!%20Level%202.md) :star: : recherche de commentaires Maps
- [On Thin Ice](./writeups/OnThinIce/On%20Thin%20Ice.md) :star::star: : extraction de données publiques d'une photo  et recherche de commentaires Maps
- [Intercepted Communications](writeups/Interceptedcommunications/InterceptedCommunications.md) :star::star: : Challenge qui consiste à exploiter une faille dans un chiffrement OTP mal sécurisé.
