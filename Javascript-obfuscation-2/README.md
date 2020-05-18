**Javascript Obfuscation 2**
================
Enoncé
----------------
Demande de mot passe


Résolution
----------------
Regarder les sources avec Google Chrome  
On joue avec **decodeURI** une première fois, puis  
Utilisation de la fonction **unescape** sur le résultat.  
Tout ça grâce à la console JS du navigateur, au final la fonction String.fromCharCode affiche le mot de passe.
