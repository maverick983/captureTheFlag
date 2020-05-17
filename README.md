Keylogger maison
----------------

- 1) **Decodage en base64, 2 solutions**
    - avec CyberChef Online
    - avec une ligne de commande `base64 -d ch24.txt > cha24decode`


- 2) **Décodage du fichier issu du** `cat /dev/input/event0`  
Utiliser l'outil écrit en RUBY `evdump` [*Lien vers evdump*](https://www.rubydoc.info/gems/device_input/0.3.1.1)  
  - installation de RUBY `sudo apt install ruby-full`
  - installation de `device_input` grâce à **gem** : `sudo gem install device_input`
  - (si message d'erreur au lancement de la commande ci-dessous) installation de `slop`  grâce à **gem** : `sudo gem install slop`  
  - lancement de la commande de décodage : `sudo cat cha24decode |  evdump --print pretty > keys.txt`
  - on obtient un fichier avec 3 lignes pour chaque touche. Il faut ne garder que les lignes avec le code de la touche, les lignes contenant **Misc:ScanCode**

- 3) **Correspondance KeyCode / caractère**  
au moyen de la commande `dumpkeys` dans un script bash maison :

```bash
#!/bin/bash
input="./keys_light.txt"
while IFS= read -r line
do
  tab=$(echo $line | cut -f3 -d":")
  key_code=${tab[0]}
  translation=$(dumpkeys | egrep "^keycode\ *$key_code\ *=")
  echo $translation
done < "$input"
```

`sudo ./read_keys_light.sh > keys_light2.txt`  
Avec la commande `dumpkeys` on n'obtient pas toutes les correspondances, par exemple pour les keys **156**, **184** et **200**  
Une petite recherche manuelle dans keys.txt le fichier issu de **evdump** :
  - 156 = KP_ENTER
  - 184 = RIGHTALT (AltGr)
  - 200 = UP
