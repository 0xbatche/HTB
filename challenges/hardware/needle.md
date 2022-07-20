Nouveau challenge Hardware.

Cette fois-ci nous avons un fichier "firmware.bin" . Je n'ai aucune idée de ce que c'est et il appert que ce type de fichier peut-être tout et n'importe quoi.

J'essaye de l'ouvrir avec autopsy (<3), mais vu que je ne sais pas ce que c'est, ça ne donne rien de probant.

Je vais donc sur la VM Kali et décide de faire ce qu'on fait de base (et ce que j'aurais dû faire dès le départ) lorsqu'on ne connaît pas un fichier : strings et file.

Strings ne donne que du garbage. La commande file quant à elle nous donne :

```sh
file firmware.bin
firmware.bin: Linux kernel ARM boot executable zImage (big-endian)
```

Intéressant... Mais je n'ai toujours aucune idée de ce que c'est ! Après quelques recherches sur les internets, je tombe sur ce blog :

https://cjhackerz.net/posts/writeup-first-ever-real-like-simulated-iot-security-challenge/

Il présente un challenge CTF (peu ou proue le format des challenges HTB), mais un challenge qui me semble... bien compliqué pour un easy challenge !

Cependant, un outil très intéressant est utilisé pour lire et extraire de la data humainement appréhendable : binwalk.

```sh
binwalk firmware.bin
DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             Linux kernel ARM boot executable zImage (big-endian)
14419         0x3853          xz compressed data
14640         0x3930          xz compressed data
538952        0x83948         Squashfs filesystem, little endian, version 4.0, compression:xz, size: 2068458 bytes, 995 inodes, blocksize: 262144 bytes, created: 2021-03-11 03:18:10
```

Ensuite on peut extraire ce qui semble être une capture d'un système linux :

```sh
binwalk -Me firmware.bin
```

Ca nous donne eh bien des centaines de fichiers !!

![img](https://github.com/0xbatche/HTB/blob/0101d3c548183a5f057fff5f6f5e0c4766b4d7a9/challenges/hardware/chall2-1.png)

J'essaye donc de voir ce qui se trouve dedans, comme des txt ou des scripts.

```sh
find . -type f -name "*.txt" | xargs cat && find . -type f -name "*.sh" | xargs cat
```
On a effectivement deux fichiers .txt qui n'ont rien de probants et plusieurs scripts sh. Mais je ne vois toujours pas quoi faire de tout ça. Puis à force de tourner en rond,
je vais sur le forum, et je vois... qu'il faut se connecter à une machine (> _ >) . Ah.

Alors connectons-nous avec nc. On a tout simplement une demande de login et de mot de passe.

Attachons nous donc à chercher quelque chose en rapport avec le login.

```sh
find . -type f -name "*.sh" | xargs cat | grep login
```

On a seulement deux occurrences dans tout le système :

```sh
        if [ -f "/usr/sbin/login" ]; then
                telnetd -l "/usr/sbin/login" -u Device_Admin:$sign      -i $lf &
                                *procd*|*ash*|*init*|*watchdog*|*ssh*|*dropbear*|*telnet*|*login*|*hostapd*|*wpa_supplicant*|*nas*|*relayd*) : ;;
        if [ -f "/usr/sbin/login" ]; then
                telnetd -l "/usr/sbin/login" -u Device_Admin:$sign      -i $lf &
[ "$(uci get system.@system[0].ttylogin)" == 1 ] || exec /bin/ash --login
exec /bin/login
[ "$(uci get system.@system[0].ttylogin)" == 1 ] || exec /bin/ash --login
exec /bin/login
                                *procd*|*ash*|*init*|*watchdog*|*ssh*|*dropbear*|*telnet*|*login*|*hostapd*|*wpa_supplicant*|*nas*|*relayd*) : ;;
```

Ce que nous disent ces commandes, c'est qu'on se connecte via telnet avec un user Device_Admin (on a donc apriori un user à essayer) avec le mot de passe $sign.
Si on regarde le script en question :
![script](https://github.com/0xbatche/HTB/blob/0101d3c548183a5f057fff5f6f5e0c4766b4d7a9/challenges/hardware/chall2-2.png)

On se rend compte qu'il s'agit d'un simple cat dans etc/config/sign. Youpi ! Sauf que dans le dossier etc il n'y a rien.

Cependant, en faisant une simple recherche :

```sh
find . -type f -name "sign" 
```

On tombe sur deux match :

```sh
./_firmware.bin.extracted/squashfs-root/etc/config/sign
./_firmware.bin.extracted/sign
```
On lit les deux fichiers, et on a ce qui semble être un mot de passe.

On l'essaye et bingo ! - on peut se connecter et lire le flag.

Conclusion :

Bien qu'il soit un challenge very easy, il a pu me permettre de découvrir de façon très simple le domaine du hack de firmware. Cette branche de la sécurité, qu'i m'était inconnue, semble très vaste et intéressante à explorer. On verra avec la suite des challenges !


