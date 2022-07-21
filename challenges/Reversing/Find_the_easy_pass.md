#Find the Easy Pass.

Une challenge concernant l'ingéniérie inversée.

La première chose à faire, travaillant avec une VM Kali, c'est d'installer de quoi pouvoir exécuter le programme sur Linux.

Le package wine et quelques autres permettent de faire cela.

```sh
sudo apt install libwine
sudo apt install wine
sudo dpkg --add-architecture i386
sudo apt install wine32:i386
echo "OUF !"
```

Dans le cas d'un problème de type :

> wine: could not load kernel32.dll, status c0000135

```sh
rm -rf ~/.wine 
winecfg
```

Bien ! Maintenant on peut exécuter le programme sur notre Kali. :)

Il s'agit d'un programme simple qui demande un mot de passe.

![ep1](https://github.com/0xbatche/HTB/blob/057f7cd112c95228328754edcd4ce672bc8e4393/challenges/Reversing/imgs/chall_find_the_easy_pass_1.PNG)

Je ne connais pas encore vraiment l'ingéniérie inversée, mais après m'être cassé les dents sur de nombreux challenges en CTF qui demandent ce type de connaissances, j'ai pu voir qu'il y a deux choses essentielles pour ce type de challenge : un désassembleur et un débugueur.

Pour désassembler, il y a énormément de solutions (avec GDB et le module python, Ida Pro, Ghidra etc). Pour ma part, je préfère Ghidra pour plusieurs raisons :

- C'est gratuit (contrairement à IDA)
- C'est "joli" (hem)
- Mes connaissances en langage assembleur étant maigres, le pseudo-code généré par le logiciel est très pratique pour essayer de s'y retrouver.

Pour débuguer, c'est idem, on peut trouver de nombreuses solutions. Pour ma part je vais utiliser ollydbg, mais un autre ferait tout aussi bien l'affaire.

Commençons par regarder ce qu'est le programme :

```sh
file EasyPass.exe
$> EasyPass.exe: PE32 executable (GUI) Intel 80386, for MS Windows
```

C'est donc un programme windows.

Désassemblons-le avec ghidra.

Le programme est manifestement obfusqué. Il est quasimment illisible, et ça peut-être décourageant. Mais chercheons tout de même à glâner des informations intéressantes.

Lorsqu'on exécute le programme, il y a la phrase "Wrong password!" qui s'affiche lorsque nous mettons un mot de passe incorrect.

On peut chercher avec ghidra ce type de phrase, en faisant "Search for > Strings" et en filtrant les résultats.

![ep2](https://github.com/0xbatche/HTB/blob/8baf7ceed976200d34c676949e2940b3ebb448f3/challenges/Reversing/imgs/chall_find_the_easy_pass_2.png)

On tombe sur notre phrase, et en cliquant dessus, la fenêtre nous permet immédiatement de voir où se trouve la référence à cette string. On remarque également qu'il y a la string "Good job. Congratulation !".

On va donc immédiatement dessus, pour savoir connaître la fonction qui invoque cette string.

Il appert qu'après tout un tas de check, la fonction compare simplement notre entrée avec le bon mot de passe.

![ep3](https://github.com/0xbatche/HTB/blob/8baf7ceed976200d34c676949e2940b3ebb448f3/challenges/Reversing/imgs/chall_find_the_easy_pass_3.PNG)

Avec Ghidra, le débugueur ayant du mal à fonctionner correctement avec le programme windows, on ne pourra manifestement pas aller plus loin (je n'ai pas eu la foi d'investiguer plus sur ce problème).

Ouvrons donc la programme avec ollydbg. Après avoir ouvert le fichier, faisons de nouveau une recherche pour  "Good job..". On tombe immédiatement sur l'instruction. Pour notre part, ce que nous souhaitons savoir, c'est ce qu'il se passe _avant_. Il suffit pour cela de mettre un point d'ancrage sur l'instruciton précédente, et de démarrer l'exécution.

![ep4](https://github.com/0xbatche/HTB/blob/8baf7ceed976200d34c676949e2940b3ebb448f3/challenges/Reversing/imgs/chall_find_the_easy_pass_4.PNG)

On regardant le panneau qui nous montre en direct ce qu'il se passe, on remarque qu'une comparaison est faite avec "fortran!".

![ep5](https://github.com/0xbatche/HTB/blob/8baf7ceed976200d34c676949e2940b3ebb448f3/challenges/Reversing/imgs/chall_find_the_easy_pass_5.PNG)

Si on essaye tout simplement ce mot de passe, on a la fenêtre avec "Good job..." qui s'ouvre. Et notre flag avec.
