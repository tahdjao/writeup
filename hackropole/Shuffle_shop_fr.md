---
date: 2024-01-04
MOC:
  - "[[MOC_CONNAISSANCE]]"
tags:
  - /connaissance/crypto
  - /connaissance/web
  - /challenge/hackropole
descriptions:
link:
---
### Ressources
>https://en.wikipedia.org/wiki/Advanced_Encryption_Standard


Shuffle Shop est un chall proposé à FCSC 2020 en catégories Web/crypto.
Ce Chall va nous permettre de voir les bases du chiffrement [AES ECB](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard)
# Recon
Tout d'abord analisons le site qui nous est proposé.
![](/images/1.png)
La première page du site semble juste être le sas d'entrée. Rien à signaler pour le code source non plus. Il nous est simplement dit que peu importe les actions que nous réaliserons plus tard, revenir à cette page les annulera.

Entrons.
![](/images/2.png)Cette page semble être un magasin de potions. Avec :

>1- Un tableau de potions avec les différents prix de chacune.
  2- Notre portefeuille -> 5000 FCSC-coins
  3- Notre panier actuel

Il y a deux boutons :

>4- Un pour shuffle le magasin et changer les produits
>5- Un pour confirmer l'achat du panier

Quand nous appuyons sur le Confirm cart avec le Panier vide nous avons une Alert erreur.
![](/images/4.png)

- Le but du chall semble donc de reussir à acheter la Flag potion. Le problème est que cette potion coûte 10 000 FCSC-coins.
![](/images/3.png)
### Source code

Il y a plusieurs fonctions js intéressantes dans le code source de cette page
- initialisation des variables. Chaque options a donc un "idx" spécifique qui la défini.
```js
var potions = JSON.parse('[{"idx":0,"name":"Health potion","price":10,"color":"#ff0000"},{"idx":1,"name":"Stamina potion","price":10,"color":"#33cc33"},{"idx":2,"name":"Strengh potion","price":15,"color":"#996633"},{"idx":3,"name":"Clerverness potion","price":15,"color":"#99ffcc"},{"idx":9,"name":"Flag potion","price":100000,"color":"#ffff00"},{"idx":4,"name":"Breath potion","price":25,"color":"#00ffcc"},{"idx":5,"name":"Invisible potion","price":30,"color":"#ffffcc"},{"idx":6,"name":"Fly potion","price":59,"color":"#ff99ff"},{"idx":7,"name":"Love potion","price":99,"color":"#ff0066"},{"idx":8,"name":"Fire-protection potion","price":67,"color":"#ff6600"}]');
	var cart_enc = "";
	var p1 = 0, p2 = 1, p3 = 2;
```

- Fonction pour afficher les magasins
Cette fonction affiche trois potions dans le magasin
```js
function displayPotions()
	{
		document.getElementById('potion_name_1').innerHTML = potions[p1]['name'];
		document.getElementById('potion_name_1').style = "background-color:" + potions[p1]['color'];
		document.getElementById('potion_price_1').innerHTML = potions[p1]['price'];
		document.getElementById('potion_buy_1').onclick = function(){addItem(potions[p1]['idx']);};
		document.getElementById('potion_name_2').innerHTML = potions[p2]['name'];
		document.getElementById('potion_name_2').style = "background-color:" + potions[p2]['color'];
		document.getElementById('potion_price_2').innerHTML = potions[p2]['price'];
		document.getElementById('potion_buy_2').onclick = function(){addItem(potions[p2]['idx']);};
		document.getElementById('potion_name_3').innerHTML = potions[p3]['name'];
		document.getElementById('potion_name_3').style = "background-color:" + potions[p3]['color'];
		document.getElementById('potion_price_3').innerHTML = potions[p3]['price'];
		document.getElementById('potion_buy_3').onclick = function(){addItem(potions[p3]['idx']);};
	}
```

- Fonction addItem
qui permet d'ajouter une potion au panier.
Cette fonction nous bloque pour ajouter notre Flag potions au panier
```js
function addItem(idx)
	{
		var xhr_addItem = new XMLHttpRequest();
		xhr_addItem.open("GET", "cart.php?add&idx="+idx, true);
		xhr_addItem.onload = function()
		{
			if(this.readyState === XMLHttpRequest.DONE && this.status === 200)
			{
				if(this.responseText === "not enough FCSC-coins")
					alert("Le montant de votre panier ne peut pas dépasser la quantité de FCSC-coins que vous possédez !");
				else if(this.responseText === "potion added")
					refreshCart();
			}
		}
		xhr_addItem.send();
	}
```


- Fonction refreshCart
qui va mettre à jour les potions du panier.
la requête `cart.php?refresh` return le json en clair
```text
{"total":0,"Health potion":0,"Stamina potion":0,"Strengh potion":0,"Clerverness potion":0,"Flag potion":0,"Breath potion":0,"Invisible potion":0,"Fly potion":0,"Love potion":0,"Fire-protection potion":0}
```

la requête `cart.php?refreshEnc`return le json chiffré et le stoque dans la variale cart_enc
```text
d6753c1c1c0fea6dc5acb87141896dc2d04f5ef584029a718b779a05967bae0b58a831353216845122eb26a951ac29eee049440c06e50aec529fb14bdbbe81d4e0af636931ea0d18f030c8a9453679a17fca8a0ff4f5cf1c34812e251a99f5ad3c7e45712b14444d470cfe46c6f9b27a53d2da2e395063a67047ecc49dec5aba568c3df55964d7d3bab704e701831c3429e83a4493fd102cd9b49d2ada8b68d1a04cb37d89749b3254e1933360286b8fa0ca8a3aa1e227b75d2a0e3baa91aac35cc148da97c5cdd3cde4a35494765949
```
```js
function refreshCart()
	{
		var xhr_refreshCart = new XMLHttpRequest();
		xhr_refreshCart.open("GET", "cart.php?refresh", true);
		xhr_refreshCart.onload = function()
		{
			if (this.readyState === XMLHttpRequest.DONE && this.status === 200)
				document.getElementById('cart').innerHTML = this.responseText;
		}
		var xhr_refreshCartEnc = new XMLHttpRequest();
		xhr_refreshCartEnc.open("GET", "cart.php?refreshEnc", true);
		xhr_refreshCartEnc.onload = function()
		{
			if (this.readyState === XMLHttpRequest.DONE && this.status === 200)
				cart_enc = this.responseText;
		}
		xhr_refreshCart.send();
		xhr_refreshCartEnc.send();
	}

```

- Fonction ValidateCart
qui va valider le panier actuel avec la variable `cart_enc`.
```js
function validateCart()
	{
		refreshCart();
		var xhr_validateCart = new XMLHttpRequest();
		xhr_validateCart.open("GET", "cart.php?validate&cart="+cart_enc, true);
		xhr_validateCart.onload = function()
		{
			if (this.readyState == XMLHttpRequest.DONE && this.status === 200)
				alert(this.responseText);
		}
		xhr_validateCart.send();
	}
```



Le but maintenant semble clair modifier la valeur de `cart_enc` par du json_encoded avec le paramètre de Flag potion a 1.


### Ffuf
Faisons quand meme un petit Ffuf pour être sur de ne pas passer à coté d'une page.
```
>>> ffuf -w `fzf-wordlists` -u "http://localhost:8000/FUZZ"
________________________________________________

.htpasswd               [Status: 403, Size: 276, Words: 20, Lines: 10]
.htaccess               [Status: 403, Size: 276, Words: 20, Lines: 10]
server-status           [Status: 403, Size: 276, Words: 20, Lines: 10]
```
Celui-ci donne rien intéressant.

Revenons donc au Json_encoded.

### Encodage AES ECB

- Encodage avec un panier vide est toujours le même. On peut donc supposer que la clef est une constante et ne change pas.

| AES always uses a block size of 128 bits (16 bytes) [source](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard)

Sachant que c'est de l'hexa on peut découper le json_encoded par bloc de 32 caractères, soit 16 bytes en hexa.

```python
>>>len('ed01c983b07a4706eb938131297f4a56d04f5ef584029a718b779a05967bae0b58a831353216845122eb26a951ac29eee049440c06e50aec529fb14bdbbe81d4cd1df04eb05e53626848b88121bb35747fca8a0ff4f5cf1c34812e251a99f5ad3c7e45712b14444d470cfe46c6f9b27a53d2da2e395063a67047ecc49dec5aba568c3df55964d7d3bab704e701831c3429e83a4493fd102cd9b49d2ada8b68d1a04cb37d89749b3254e1933360286b8fa0ca8a3aa1e227b75d2a0e3baa91aac35cc148da97c5cdd3cde4a35494765949')
416 # en hexa donc

>>> 416/32
13.0
```
Il y a donc 13 blocs de 16 bytes.
Dans le json ça donne des blocs de 16 caractères car l'ascii est encodé sur 1 bytes. Voici les Blocs :
```python
>>>for i in range (13) :
...     print('{"total":15,"Health potion":0,"Stamina potion":0,"Strengh potion":0,"Clerverness potion":1,"Flag potion":0,"Breath potion":0,"Invisible potion":0,"Fly potion":0,"Love potion":0,"Fire-protection potion":0}'[16*i:][:16])
...
{"total":15,"Hea
lth potion":0,"S
tamina potion":0
,"Strengh potion
":0,"Clerverness
 potion":0,"Flag <---  | # les Deux boc se ressemble on pourrait les switch
 potion":0,"Brea<---
th potion":0,"In
visible potion":
0,"Fly potion":0
,"Love potion":0
,"Fire-protectio
n potion":0}
```

Ligne 9 et 10 les blocs se ressemble il suffirait donc de remplacer la ligne 10 (`potion":0,"Brea`) par la ligne 9 (`potion":1,"Flag`). Le nom importe peu vu que le programme utilise des ids.
Je vais donc acheter une potion Clerverness et coller le bloc a 1 dans Flag potion
```python
#!/bin/python3

# --- DEFINITION ---

def poc():
    # cart JSon
    json = '{"total":15,"Health potion":0,"Stamina potion":0,"Strengh potion":0,"Clerverness potion":1,"Flag potion":0,"Breath potion":0,"Invisible potion":0,"Fly potion":0,"Love potion":0,"Fire-protection potion":0}'
    # JSon ecoded
    jsonencode = "d6753c1c1c0fea6dc5acb87141896dc2d04f5ef584029a718b779a05967bae0b58a831353216845122eb26a951ac29eee049440c06e50aec529fb14bdbbe81d4e0af636931ea0d18f030c8a9453679a17fca8a0ff4f5cf1c34812e251a99f5ad3c7e45712b14444d470cfe46c6f9b27a53d2da2e395063a67047ecc49dec5aba568c3df55964d7d3bab704e701831c3429e83a4493fd102cd9b49d2ada8b68d1a04cb37d89749b3254e1933360286b8fa0ca8a3aa1e227b75d2a0e3baa91aac35cc148da97c5cdd3cde4a35494765949"

    # Knowing that the json is encoded using 32 bits Block
    # print the json decoded blocks
    print("\njson blocks =")
    for i in range(int(len(jsonencode)/32)):
        print(json[16*i:16*(i+1)])

    print("\n#------RESULT--------#")
    print()
    json_result = json[16*0:16*6]+json[16*5:16*6]+json[16*7:]
    print(f"json = {json_result}")
    print()
    json_encode_result = jsonencode[32*0:32*6] + \
        jsonencode[32*5:32*6]+jsonencode[32*7:]
    print(f"encoded json = {json_encode_result}")


# --- UTILISATION ---
if __name__ == "__main__":
    poc()

```

ce code retourne

```python

json blocks =
{"total":15,"Hea
lth potion":0,"S
tamina potion":0
,"Strengh potion
":0,"Clerverness
 potion":1,"Flag
 potion":0,"Brea
th potion":0,"In
visible potion":
0,"Fly potion":0
,"Love potion":0
,"Fire-protectio
n potion":0}

#------RESULT--------#

json = {"total":15,"Health potion":0,"Stamina potion":0,"Strengh potion":0,"Clerverness potion":1,"Flag potion":1,"Flagth potion":0,"Invisible potion":0,"Fly potion":0,"Love potion":0,"Fire-protection potion":0}

encoded json = d6753c1c1c0fea6dc5acb87141896dc2d04f5ef584029a718b779a05967bae0b58a831353216845122eb26a951ac29eee049440c06e50aec529fb14bdbbe81d4e0af636931ea0d18f030c8a9453679a17fca8a0ff4f5cf1c34812e251a99f5ad7fca8a0ff4f5cf1c34812e251a99f5ad53d2da2e395063a67047ecc49dec5aba568c3df55964d7d3bab704e701831c3429e83a4493fd102cd9b49d2ada8b68d1a04cb37d89749b3254e1933360286b8fa0ca8a3aa1e227b75d2a0e3baa91aac35cc148da97c5cdd3cde4a35494765949
```


Nous avons plus qu'à écraser la valeur de cart_enc avec notre nouveau json_encoded
![](/images/5.png)

![](/images/6.png)
# Flag

```
FCSC{c9582322dfa2997384b7d38b73b5a80a69374d5f3f616c431e98823172f5c7df}
```

