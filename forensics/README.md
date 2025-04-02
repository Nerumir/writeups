# Hackali - Forensics

Vous trouverez ici tous les writeups axés sur **l'analyse des données numériques** et **les investigations forensiques**.  
Si vous souhaitez découvrir de **nouvelles techniques d’investigation numérique** ou apprendre de **nouvelles astuces**, alors n’hésitez pas à les parcourir !

---

## Writeups disponibles

- **MuddyWater**  
  Dans ce writeup, j'ai analysé un **pcap** d'une intrusion sur un **Domain Controller**. En filtrant **NTLM** avec **Wireshark**, j'ai repéré les tentatives de **bruteforce** et isolé la **session setup réussie**. Ensuite, j'ai reconstruit le **hash NTLMv2** en corrigeant son format et utilisé **Hashcat** pour le cracker.
