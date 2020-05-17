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
  - on obtient un fichier avec 3 lignes pour chaque touche. Il faut ne garder que les lignes avec le code de la touche, les lignes contenant **Misc:ScanCode** ce qui nous donne un fichier keys_light.txt

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
   -- 156 = KP_ENTER (touche entrée du pavé numérique)  
   -- 184 = RIGHTALT (AltGr)  
   -- 200 = UP  

- 4) **Décodage de ce qui a été saisi au clavier**  
```bash
xt
ssh app-systeme_ch7@challenge02.root-me.org -p 2222
yes
app-systeme-ch7
ls -l
cat ch7.c
exit
vi hihi.py
i (pour le insert dans VIM)
```
```python
#!/usr/bin/python
from sys import argv
if len(argv) < 2:
  print ("Usage:%s < to be xored > " % argv[0])
  exit(2)
def myXor(d):
  a = ''
  for i in d:
    a+= chr(157 ^ ord(i))
  return a
print(myXor(argv[1]))
```
```bash
Echap :x (pour sortir de VIM)
echo c0nGralut4t10n__$(./hihi.py $(date +%s) | sha1sum | grep -o '^[0-9a-f]\+')_hoh0 > /tmp/flag
chmod +x *.py
exit
```
Seulement il ne faut pas oublier que `(date +%s)` était le timestamp à l'heure de l'exécution du script.  
Il faut donc reprendre le fichier keys.txt, et choisir le timestamp correspondant à l'appui de la touche Entrée.  
Cela transforme donc la commande en   
`echo c0nGralut4t10n__$(./hihi.py $(echo 1510683198) | sha1sum | grep -o '^[0-9a-f]\+')_hoh0`

pour obtenir le timestamp, un petit programme en Python trouvé sur le web :
```python
#!/usr/bin/python
import struct
import time
import sys

infile_path = "./cha24decode"

"""
FORMAT represents the format used by linux kernel input event struct
See https://github.com/torvalds/linux/blob/v5.5-rc5/include/uapi/linux/input.h#L28
Stands for: long int, long int, unsigned short, unsigned short, unsigned int
"""
FORMAT = 'LI0LHHI'
EVENT_SIZE = struct.calcsize(FORMAT)

#open file in binary mode
in_file = open(infile_path, "rb")

event = in_file.read(EVENT_SIZE)

while event:
    (tv_sec, tv_usec, type, code, value) = struct.unpack(FORMAT, event)

    if type != 0 or code != 0 or value != 0:
        print("Event type %u, code %u, value %u at %d.%d" % \
            (type, code, value, tv_sec, tv_usec))
    else:
        # Events with code, type and value == 0 are "separator" events
        print("===========================================")

    event = in_file.read(EVENT_SIZE)

in_file.close()

```
