# Weak RSA

Un challenge qui aura été assez long à faire malgré la simplicité de la résolution.

En effet, je ne connaissais pour ainsi dire pas grand chose de la cryptographie, et c'était donc l'occasion de plonger les mains dans le cambouis.

Si on souhaite réaliser rapidement le challenge, il suffit de taper "RSACtftool" sur google et d'exécuter le script python.

Pour ma part, j'ai souhaité voir comment on pouvait faire en écrivant à la main un script pour résoudre ce challenge.

Ce fut beaucoup plus long que prévu. Ici un condensé de ce à quoi je suis arrivé.

On a donc un fichier flag.enc 

```sh
�_�vc[��~�kZ�1�Ĩ�4�I�9V��^G���(�+3Lu"�T$���F0�VP�־j@������|j▒�������{¾�,�����YE������Xx��,��c�N&Hl2�Ӎ��[o�� 
```

Et la clé publique :

```
-----BEGIN PUBLIC KEY-----
MIIBHzANBgkqhkiG9w0BAQEFAAOCAQwAMIIBBwKBgQMwO3kPsUnaNAbUlaubn7ip
4pNEXjvUOxjvLwUhtybr6Ng4undLtSQPCPf7ygoUKh1KYeqXMpTmhKjRos3xioTy
23CZuOl3WIsLiRKSVYyqBc9d8rxjNMXuUIOiNO38ealcR4p44zfHI66INPuKmTG3
RQP/6p5hv1PYcWmErEeDewKBgGEXxgRIsTlFGrW2C2JXoSvakMCWD60eAH0W2PpD
qlqqOFD8JA5UFK0roQkOjhLWSVu8c6DLpWJQQlXHPqP702qIg/gx2o0bm4EzrCEJ
4gYo6Ax+U7q6TOWhQpiBHnC0ojE8kUoqMhfALpUaruTJ6zmj8IA1e1M6bMqVF8sr
lb/N
-----END PUBLIC KEY-----
```

L'idée, c'est à partir de cette clé publique de trouver la clé privée et donc in fine de décoder le message au sein du fichier.

La difficulté pour cracker le chiffrement RSA, c'est de factoriser N, qui est le produit de p * q, deux nombres premiers.

J'ai appris par la suite (une fois le flag chopé) qu'on pouvait trouver "facilement" la solution grâce à une attaque de Wiener.

Pour ma part je ne sais pas quelle attaque c'est, mais grâce à un mélange de ce que j'ai appris sur [cryptohack](https://cryptohack.org/) et à force d'essais, j'ai pu réussir à trouver le flag.

Alors la première chose qu'on souhaite trouver, ce sont p et q. Car une fois qu'on connaît ces deux nombres, on peut calculer la clé privée k telle que :

```
k = e ^ -1 (mod phi)
```

Ou e sera l'exposant, et phi la fonction indicatrice d'Euler tel que phi :

```
phi = (p - 1) * (q - 1)
```

Pour trouver p et q, j'ai tuilisé une méthode connue sous le nom de "google it" : à savoir en allant sur [factordb](http://factordb.com/index.php).

En effet, il y a déjà plein de nombres dont on connaît p et q (sinon on peut essayer de les calculer, mais c'est trèèèès long, sauf avec l'attaque de Wiener peut-être, en partant du fait que p et q sont proches et e faible).

```python
#!/usr/bin/python

import binascii
from Crypto.PublicKey import RSA
from Crypto.Util.number import inverse

file = open('./key.pub')
key = RSA.importKey(file.read())
print('\nn défini tel que n = p * q, deux nombres premiers :\n')
print(key.n)
print('\ne défini tel que 1 < e < phi(n) et gcd - plus grand commun diviseur - gcd(e, phi(n)) = 1\n')
print(key.e)
print('\nEn faisant un tour sur factordb, on peut trouver directement P et Q :\n')

p = 20423438101489158688419303567277343858734758547418158024698288475832952556286241362315755217906372987360487170945062468605428809604025093949866146482515539
q = 28064707897434668850640509471577294090270496538072109622258544167653888581330848582140666982973481448008792075646342219560082338772652988896389532152684857
```

Ensuite, nous devons donc calculer la clé privée, qui est l'inverse modulaire de e modulo la fonction indicatrice d'euler, (le _totient_ en anglais) de n ( == p * q) :

```python
def totient(p, q):
    phi = (p - 1) * (q - 1)
    return phi

def priv_key(e, p, q):
    return pow(e, -1, totient(p, q))
```

Et donc décrypter le message, ce qui nous donnera un looooooooooooooong nombre :
```
def decrypt(msg, pk, n):
    return pow(msg, pk, n)
```

Il nous reste donc plus qu'à) mélanger tout ça et décoder notre message :

```python
enc_msg=open('flag.enc', 'rb').read()
dec = decrypt(int.from_bytes(enc_msg, "big"), pk, int(p) * int(q))
dec = dec.to_bytes((dec.bit_length() + 7) // 8 , "big").decode('utf-8', errors='ignore')
print("The decrypted message is : ",dec)
```

Ce qu'il se passe, c'est que grâce à la clé privée, nous pouvons donc retrouver notre "message", mais sous forme d'un nombre. Ce nombre ne veut rien en soi, donc on va devoir l'imprimer sous forme de char, qui font 7bits, et décoder ce tableau pour qu'il soit lisible.

L'option "big" est relative au [boutisme](https://fr.wikipedia.org/wiki/Boutisme) (drôle de mot), ou l'_endianess_ en anglais, ie la façon dont les octets sont rangés. Là c'est brute-force, mais on n'a que deux choix, donc ça va.

In fine, on récupère le message :

```
!ϲo@gX{Dn(Dx".,-)!6WQ+L~J9{_SDYCߚ▒4LU
                                                                 p/AHTB{s1mpl3_Wi3n3rs_4tt4ck}
```

Code complet :

```python
#!/usr/bin/python

import binascii
from Crypto.PublicKey import RSA
from Crypto.Util.number import inverse

file = open('./key.pub')
key = RSA.importKey(file.read())
print('\nn défini tel que n = p * q, deux nombres premiers :\n')
print(key.n)
print('\ne défini tel que 1 < e < phi(n) et gcd - plus grand commun diviseur - gcd(e, phi(n)) = 1\n')
print(key.e)
print('\nEn faisant un tour sur factordb, on peut trouver directement P et Q :\n')

p = 20423438101489158688419303567277343858734758547418158024698288475832952556286241362315755217906372987360487170945062468605428809604025093949866146482515539
q = 28064707897434668850640509471577294090270496538072109622258544167653888581330848582140666982973481448008792075646342219560082338772652988896389532152684857

def totient(p, q):
    phi = (p - 1) * (q - 1)
    return phi

def priv_key(e, p, q):
    return pow(e, -1, totient(p, q))

def decrypt(msg, pk, n):
    return pow(msg, pk, n)

print("Le totient est :", totient(int(p), int(q)))
pk =  priv_key(key.e, p, q)
# ou pk = inverse(e, totient(p, q)) avec la librairie crypto.

print("L'inverse du modulo et donc la clé privée est :", pk)

enc_msg=open('flag.enc', 'rb').read()
dec = decrypt(int.from_bytes(enc_msg, "big"), pk, int(p) * int(q))
dec = dec.to_bytes((dec.bit_length() + 7) // 8 , "big").decode('utf-8', errors='ignore')
print("The decrypted message is : ", dec) 
```

## Conclusion

Malgré le tag facile de ce challenge, j'ai appris pas mal de choses sur les bases du chiffrement RSA. C'était très intéressant de faire soi-même le script afin de comprendre de façon plus fine qu'un outil pré-fait comment on peut cracker ce type de chiffrement.

