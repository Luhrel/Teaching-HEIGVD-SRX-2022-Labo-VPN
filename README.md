# Teaching-HEIGVD-SRX-2021-Labo-VPN

**Ce travail de laboratoire est à faire en équipes de 2 personnes**

**Pour ce travail de laboratoire, il est votre responsabilité de chercher vous-même sur internet, le support du cours ou toute autre source (vous avez aussi le droit de communiquer avec les autres équipes), toute information relative au sujet VPN, le logiciel eve-ng, les routeur Cisco, etc que vous ne connaissez pas !**

**ATTENTION : Commencez par créer un Fork de ce répo et travaillez sur votre fork.**

Clonez le répo sur votre machine. Vous pouvez répondre aux questions en modifiant directement votre clone du README.md ou avec un fichier pdf que vous pourrez uploader sur votre fork.

**Le rendu consiste simplement à répondre à toutes les questions clairement identifiées dans le text avec la mention "Question" et à les accompagner avec des captures. Le rendu doit se faire par une "pull request". Envoyer également le hash du dernier commit et votre username GitHub par email au professeur et à l'assistant**

**N'oubliez pas de spécifier les noms des membres du groupes dans la Pull Request ainsi que dans le mail de rendu !!!**


## Echéance 

Ce travail devra être rendu au plus tard, **le 3 juin 2022, à 10h25.**


## Introduction

Dans ce travail de laboratoire, vous allez configurer des routeurs Cisco émulés, afin de mettre en œuvre une infrastructure sécurisée utilisant des tunnels IPSec.

### Les aspects abordés

-	Contrôle de fonctionnement de l’infrastructure
-	Contrôle du DHCP serveur hébergé sur le routeur
-	Gestion des routeurs en console
-	Capture Sniffer avec filtres précis sur la communication à épier
-	Activation du mode « debug » pour certaines fonctions du routeur
-	Observation des protocoles IPSec
 
 
## Matériel

Le logiciel d'émulation à utiliser c'est eve-ng (vous l'avez déjà employé). Vous trouverez ici un [guide très condensé](files/Manuel_EVE-NG.pdf) pour l'utilisation et l'installation de eve-ng.

Vous pouvez faire fonctionner ce labo sur vos propres machines à condition de copier la VM eve-ng. A présent, la manière la plus simple d'utiliser eve-ng est de l'installer sur Windows (mais, il est possible de le faire fonctionner sur Mac OS et sur Linux...). **Si vous avez toujours la VM eve-ng que vous avez utilisée dans un cours précédant, cela devrait fonctionner aussi et vous n'avez donc pas besoin de récupérer une nouvelle version.**

**Récupération de la VM pré-configurée** (vous ne pouvez pas utiliser la versión qui se trouve sur le site de eve-ng) : comme indiqué dans le [manuel](files/Manuel_EVE-NG.pdf) vous la trouverez sur [ce lien switch drive](https://drive.switch.ch/index.php/s/4KtTNwzxbF94P1d).

Il est conseillé de passer la VM en mode "Bridge" si vous avez des problèmes. Le mode NAT **devrait** aussi fonctionner.

Les user-password en mode terminal sont : "root" | "eve"

Les user-password en mode navigateur sont : "admin" | "eve"

Ensuite, terminez la configuration de la VM, connectez vous et récupérez l'adresse ip de la machine virtuelle.

Utilisez un navigateur internet (hors VM) et tapez l'adresse IP de la VM.


## Fichiers nécessaires 

Tout ce qu'il vous faut c'est un [fichier de projet eve-ng](files/eve-ng_Labo_VPN_SRX.zip), que vous pourrez importer directement dans votre environnement de travail.


## Mise en place

Voici la topologie qui sera simulée. Elle comprend deux routeurs interconnectés par l'Internet. Les deux réseaux LAN utilisent les services du tunnel IPSec établi entre les deux routeurs pour communiquer.

Les "machines" du LAN1 (connecté au ISP1) sont simulées avec l'interface loopback du routeur. Les "machines" du LAN2 sont représentées par un seul ordinateur.  

![Topologie du réseau](images/topologie.png)

Voici le projet eve-ng utilisé pour implémenter la topologie. Le réseau Internet (nuage) est simulé par un routeur. 

![Topologie eve-ng](images/topologie-eve-ng.png)


## Manipulations

- Commencer par importer le projet dans eve-ng.
- Prenez un peu de temps pour vous familiariser avec la topologie présentée dans ce guide et comparez-la au projet eve-ng. Identifiez les éléments, les interconnexions et les adresses IP.
- À tout moment, il vous est possible de sauvegarder la configuration dans la mémoire de vos routeurs :
	- Au Shell privilégié (symbole #) entrer la commande suivante pour sauvegarder la configuration actuelle dans la mémoire nvram du routeur : ```wr```
	- Vous **devez** faire des sauvegardes de la configuration (exporter) dans un fichier - c.f. [document guide eve-ng](files/Manuel_EVE-NG.pdf), section 3.2 et 3.3.


### Vérification de la configuration de base des routeurs
Objectifs:

Vérifier que le projet a été importé correctement. Pour cela, nous allons contrôler certains paramètres :

- Etat des interfaces (`show interface`)
- Connectivité (`ping`, `show arp`)
- Contrôle du DHCP serveur hébergé sur R2


### A faire...

- Contrôlez l’état de toutes vos interfaces dans les deux routeurs et le routeur qui simule l'Internet - Pour contrôler l’état de vos interfaces (dans R1, par exmeple) les commandes suivantes sont utiles :

```
R1# show ip interface brief
R1# show interface <interface-name>
R1# show ip interface <interface-name>
```

Un « status » différent de `up` indique très souvent que l’interface n’est pas active.

Un « protocol » différent de `up` indique la plupart du temps que l’interface n’est pas connectée correctement (en tout cas pour Ethernet).

**Question 1: Avez-vous rencontré des problèmes ? Si oui, qu’avez-vous fait pour les résoudre ?**

---

**Réponse :**  

Étonnamment, non. 

---


- Contrôlez que votre serveur DHCP sur R2 est fonctionnel - Contrôlez que le serveur DHCP préconfiguré pour vous sur R2 a bien distribué une adresse IP à votre station « VPC ».


Les commandes utiles sont les suivantes :

```
R2# show ip dhcp pool 
R2# show ip dhcp binding
```

Côté station (VPC) vous pouvez valider les paramètres reçus avec la commande `show ip`. Si votre station n’a pas reçu d’adresse IP, utilisez la commande `ip dhcp`.

- Contrôlez la connectivité sur toutes les interfaces à l’aide de la commande ping.

Pour contrôler la connectivité les commandes suivantes sont utiles :

```
R2# ping ip-address
R2# show arp (utile si un firewall est actif)
```

Pour votre topologie il est utile de contrôler la connectivité entre :

- R1 vers ISP1 (193.100.100.254)
- R2 vers ISP2 (193.200.200.254)
- R2 (193.200.200.1) vers RX1 (193.100.100.1) via Internet
- R2 (172.17.1.1) et votre poste « VPC »

**Question 2: Tous vos pings ont-ils passé ? Si non, est-ce normal ? Dans ce cas, trouvez la source du problème et corrigez-la.**

---

**Réponse :**  

Oui, ils ont tous passé (une fois la commande `ip dhcp` faite sur le VPC).

---

- Activation de « debug » et analyse des messages ping.

Maintenant que vous êtes familier avec les commandes « show » nous allons travailler avec les commandes de « debug ». A titre de référence, vous allez capturer les messages envoyés lors d’un ping entre votre « poste utilisateur » et un routeur. Trouvez ci-dessous la commande de « debug » à activer.

Activer les messages relatif aux paquets ICMP émis par les routeurs (repérer dans ces messages les type de paquets ICMP émis - < ICMP: echo xxx sent …>)

```
R2# debug ip icmp
```
Pour déclencher et pratiquer les captures vous allez « pinger » votre routeur R1 avec son IP=193.100.100.1 depuis votre « VPC ». Durant cette opération vous tenterez d’obtenir en simultané les informations suivantes :

-	Une trace sniffer (Wireshark) à la sortie du routeur R2 vers Internet. Si vous ne savez pas utiliser Wireshark avec eve-ng, référez-vous au document explicatif eve-ng. Le filtre de **capture** (attention, c'est un filtre de **capture** et pas un filtre d'affichage) suivant peut vous aider avec votre capture : `ip host 193.100.100.1`. 
-	Les messages de R1 avec `debug ip icmp`.


**Question 3: Montrez vous captures**

---

**Screenshots :**  
Debug des requêtes ICMP sur R1:
![Screenshot de R1](images/Q3_R1.png)

Capture des paquets ICMP entre R2 et Internet:
![Screenshot de Wireshark](images/Q3_Wireshark.png)

---

## Configuration VPN LAN 2 LAN

**Il est votre responsabilité de chercher vous-même sur internet toute information relative à la configuration que vous ne comprenez pas ! La documentation CISCO en ligne est extrêmement complète et le temps pour rendre le labo est plus que suffisant !**

Nous allons établir un VPN IKE/IPsec entre le réseau de votre « loopback 1 » sur R1 (172.16.1.0/24) et le réseau de votre « VPC » R2 (172.17.1.0/24). La terminologie Cisco est assez « particulière » ; elle est listée ici, avec les étapes de configuration, qui seront les suivantes :

- Configuration des « proposals » IKE sur les deux routeurs (policy)
- Configuration des clefs « preshared » pour l’authentification IKE (key)
- Activation des « keepalive » IKE
- Configuration du mode de chiffrement IPsec
- Configuration du trafic à chiffrer (access list)
- Activation du chiffrement (crypto map)


### Configuration IKE

Sur le routeur R1 nous activons un « proposal » IKE. Il s’agit de la configuration utilisée pour la phase 1 du protocole IKE. Le « proposal » utilise les éléments suivants :

| Element          | Value                                                                                                        |
|------------------|----------------------------------------------------------------------------------------------------------------------|
| Encryption       | AES 256 bits    
| Signature        | Basée sur SHA-1                                                                                                      |
| Authentification | Preshared Key                                                                                                        |
| Diffie-Hellman   | avec des nombres premiers sur 1536 bits                                                                              |
| Renouvellement   | des SA de la Phase I toutes les 30 minutes                                                                           |
| Keepalive        | toutes les 30 secondes avec 3 « retry »                                                                              |
| Preshared-Key    | pour l’IP du distant avec le texte « cisco-1 », Notez que dans la réalité nous utiliserions un texte plus compliqué. |


Les commandes de configurations sur R1 ressembleront à ce qui suit :

```
crypto isakmp policy 20
  encr aes 256
  authentication pre-share
  hash sha
  group 5
  lifetime 1800
crypto isakmp key cisco-1 address 193.200.200.1 no-xauth
crypto isakmp keepalive 30 3
```

Sur le routeur R2 nous activons un « proposal » IKE supplémentaire comme suit :

```
crypto isakmp policy 10
  encr 3des
  authentication pre-share
  hash md5
  group 2
  lifetime 1800
crypto isakmp policy 20
  encr aes 256
  authentication pre-share
  hash sha
  group 5
  lifetime 1800
crypto isakmp key cisco-1 address 193.100.100.1 no-xauth
crypto isakmp keepalive 30 3
```

Vous pouvez consulter l’état de votre configuration IKE avec les commandes suivantes. Faites part de vos remarques :

**Question 4: Utilisez la commande `show crypto isakmp policy` et faites part de vos remarques :**

---

**Réponse :**  

```
Global IKE policy
Protection suite of priority 10
        encryption algorithm:   Three key triple DES
        hash algorithm:         Message Digest 5
        authentication method:  Pre-Shared Key
        Diffie-Hellman group:   #2 (1024 bit)
        lifetime:               1800 seconds, no volume limit
Protection suite of priority 20
        encryption algorithm:   AES - Advanced Encryption Standard (256 bit keys).
        hash algorithm:         Secure Hash Standard
        authentication method:  Pre-Shared Key
        Diffie-Hellman group:   #5 (1536 bit)
        lifetime:               1800 seconds, no volume limit
```
On peut voir qu'on a bien deux policies définies dans notre configuration, chacune avec une priorité différente. La configuration avec la priorité la plus haute sera utilisée, si elle ne peut pas être mise en oeuvre on va passer à celle avec la priorité plus basse. On peut aussi voir que la configuration avec la priorité de 10 utilise des algorithmes moins sécurisés, tels que 3DES et MD5. Cette configuation utilise également une longeur de clé de DH plus petite (1024 bits vs 1536 bits).

Même le groupe #5 pour la configuration plus avancée n'est pas suffisant selon les standards actuels, on devrait avoir minimum le groupe #14 (2048 bits) ou #19 (256 bits EC).


---


**Question 5: Utilisez la commande `show crypto isakmp key` et faites part de vos remarques :**

---

**Réponse :**  

```
Keyring      Hostname/Address                            Preshared Key

default      193.100.100.1                               cisco-1
```
La clé n'est absolument pas assez robuste pour une utilisation en réalité. Il faudrait une clé plus longue et plus compliquée. Si cette clé est cassée, des attaquants pourraient avoir accès aux communications chiffrées dans le tunnel VPN.

---

## Configuration IPsec

Nous allons maintenant configurer IPsec de manière identique sur les deux routeurs. Pour IPsec nous allons utiliser les paramètres suivants :

| Paramètre      | Valeur                                  |
|----------------|-----------------------------------------|
| IPsec avec IKE | IPsec utilisera IKE pour générer ses SA |
| Encryption     | AES 192 bits                            |
| Signature      | Basée sur SHA-1                         |
| Proxy ID R1    | 172.16.1.0/24                           |
| Proxy ID R2    | 172.17.1.0/24                           |

Changement de SA toutes les 5 minutes ou tous les 2.6MB

Si inactifs les SA devront être effacés après 15 minutes

Les commandes de configurations sur R1 ressembleront à ce qui suit :

```
crypto ipsec security-association lifetime kilobytes 2560
crypto ipsec security-association lifetime seconds 300
crypto ipsec transform-set STRONG esp-aes 192 esp-sha-hmac 
  ip access-list extended TO-CRYPT
  permit ip 172.16.1.0 0.0.0.255 172.17.1.0 0.0.0.255
crypto map MY-CRYPTO 10 ipsec-isakmp 
  set peer 193.200.200.1
  set security-association idle-time 900
  set transform-set STRONG 
  match address TO-CRYPT
```

Les commandes de configurations sur R2 ressembleront à ce qui suit :

```
crypto ipsec security-association lifetime kilobytes 2560
crypto ipsec security-association lifetime seconds 300
crypto ipsec transform-set STRONG esp-aes 192 esp-sha-hmac 
  mode tunnel
  ip access-list extended TO-CRYPT
  permit ip 172.17.1.0 0.0.0.255 172.16.1.0 0.0.0.255
crypto map MY-CRYPTO 10 ipsec-isakmp 
  set peer 193.100.100.1
  set security-association idle-time 900
  set transform-set STRONG 
  match address TO-CRYPT
```

Vous pouvez contrôler votre configuration IPsec avec les commandes suivantes :

```
show crypto ipsec security-association
show crypto ipsec transform-set
show access-list TO-CRYPT
show crypto map
```

## Activation IPsec & test

Pour activer cette configuration IKE & IPsec il faut appliquer le « crypto map » sur l’interface de sortie du trafic où vous voulez que l’encryption prenne place. 

Sur R1 il s’agit, selon le schéma, de l’interface « Ethernet0/0 » et la configuration sera :

```
interface Ethernet0/0
  crypto map MY-CRYPTO
```

Sur R2 il s’agit, selon le schéma, de l’interface « Ethernet0/0 » et la configuration sera :

```
interface Ethernet0/0
  crypto map MY-CRYPTO
```


Après avoir entré cette commande, normalement le routeur vous indique que IKE (ISAKMP) est activé. Vous pouvez contrôler que votre « crypto map » est bien appliquée sur une interface avec la commande `show crypto map`.

Pour tester si votre VPN est correctement configuré vous pouvez maintenant lancer un « ping » sur la « loopback 1 » de votre routeur RX1 (172.16.1.1) depuis votre poste utilisateur (172.17.1.100). De manière à recevoir toutes les notifications possibles pour des paquets ICMP envoyés à un routeur comme RX1 vous pouvez activer un « debug » pour cela. La commande serait :

```
debug ip icmp
```

Pensez à démarrer votre sniffer sur la sortie du routeur R2 vers internet avant de démarrer votre ping, collectez aussi les éventuels messages à la console des différents routeurs. 

**Question 6: Ensuite faites part de vos remarques dans votre rapport. :**

---

**Réponse :**  

Messages de debug ICMP sur R1:

![Screenshot R1](images/Q6_R1.png)

Capture de Wireshark:

![Capture Wireshark](images/Q6_Wireshark.png)

Ping depuis VPC:

![Screenshot VPC](images/Q6_VPC.png)

On remarque que sur les 5 ping envoyés, seuls 4 sont arrivés au loopback. Ceci peut être du à l'établissement de la connection sécurisée entre R1 et R2. On remarque également sur la capture Wireshark que les paquets ICMP ne sont plus visibles et ils sont sous la forme de paquets ESP (Encapulating Security Payload), on ne peut plus voir le trafic entre R2 et R1 de manière claire. On a également des paquets qui servent à l'échange des clés et à l'établissement de la connection sécurisée avant l'envoi de paquets ICMP.

---

**Question 7: Reportez dans votre rapport une petite explication concernant les différents « timers » utilisés par IKE et IPsec dans cet exercice (recherche Web). :**

---

**Réponse :**  

Pour IKE, il y a deux timers différents :
- `lifetime`: Temps avant que la clef utilisée pour le chiffrement soit remplacée (8h par défaut).
- `keepalive` : Temps entre chaque message DPD (Dead Peer Detection) (mode `periodic`) ou nombre de secondes pendant lesquelles le trafic n'est pas reçu du pair avant que les messages de relance DPD ne soient envoyés s'il y a du trafic de données (IPSec) à envoyer (mode `on-demand`)

Pour IPSec, 
- `idle-time` : Temps avant de supprimer automatiquement les SAs (Security Association) lorsqu'une connexion est inactive.
- `lifetime` : Temps avant que la SA expire (1h par défaut).


Sources :
- [Meraki - IPsec VPN Lifetimes ](https://documentation.meraki.com/MX/Site-to-site_VPN/IPsec_VPN_Lifetimes)
- [Cisco - Understand IPsec IKEv1 Protocol](https://www.cisco.com/c/en/us/support/docs/security-vpn/ipsec-negotiation-ike-protocols/217432-understand-ipsec-ikev1-protocol.html#anc12)
- [Cisco - Overview of Keepalive Mechanisms on Cisco IOS](https://www.cisco.com/c/en/us/support/docs/content-networking/keepalives/118390-technote-keepalive-00.html)
- [Cisco - `crypto isakmp keepalive` command](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/security/a1/sec-a1-cr-book/sec-cr-c4.html#wp2512063629)
- [Netgate - Phase 1 Settings](https://docs.netgate.com/pfsense/en/latest/vpn/ipsec/configure-p1.html)
- [Netgate - Phase 2 Settings](https://docs.netgate.com/pfsense/en/latest/vpn/ipsec/configure-p2.html)
- [Cisco - `crypto ipsec security-association lifetime ` command](https://www.cisco.com/c/en/us/td/docs/ios/12_2/security/command/reference/srfipsec.html#wp1017619)
- [Cisco - IPsec Security Association Idle Timers](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/sec_conn_dplane/configuration/zZ-Archive/IPsec_Security_Association_Idle_Timers.html)

---


# Synthèse d’IPsec

En vous appuyant sur les notions vues en cours et vos observations en laboratoire, essayez de répondre aux questions. À chaque fois, expliquez comment vous avez fait pour déterminer la réponse exacte (capture, config, théorie, ou autre).


**Question 8: Déterminez quel(s) type(s) de protocole VPN a (ont) été mis en œuvre (IKE, ESP, AH, ou autre).**

---

**Réponse :**  

Les protocoles utilisés sont:
* IKE pour l'échange de clés:  on peut le voir dans la configuration de *RX1* et *RX2* au moment de configurer les proposal IKE. On peut aussi le voir dans les captures de paquets, dans la phase 1 le protocole **ISAKMP** est utilisé et des proposals **IKE** sont envoyés par les routeurs pour se mettre d'accord sur une policy d'échange de clés.
* ISAKMP est utilisé comme protocole sous-jacent à **IKE** pour établir le *SA* (*Security Association*)
* ESP pour l'encapsulation des paquets: au moment de configurer **IPSec** dans les routeurs on a défini l'utilisation d'**ESP** avec *AES* et *HMAC*. On peut également le voir dans la capture de Wireshark (cf. Question 6), le protocole détecté est bien **ESP**.
---


**Question 9: Expliquez si c’est un mode tunnel ou transport.**

---

**Réponse :**

C'est un mode tunnel qui est utilisé. Premièrement, dans la configuration des routeurs, c'est défini par défaut (avec le `transform-set STRONG`). Deuxièmement, car dans la capture Wireshark suivante, les headers des paquets sont également encapsulé et il n'est donc pas possible de connaitre les adresses IP source et destination d'origine, car tout le paquet IP a été encapsulé par le routeur. C'est donc l'adresse des routeurs qui est affichée.

![tunnel proof](images/Q9_tunnel_proof.png)

Configuration Cisco:
```ios
crypto ipsec transform-set STRONG esp-aes 192 esp-sha-hmac 
 mode tunnel
 ```
---


**Question 10: Expliquez quelles sont les parties du paquet qui sont chiffrées. Donnez l’algorithme cryptographique correspondant.**

---

**Réponse :**

Tout le paquet IP est chiffré, header comme contenu. On ne sait donc pas du tout ce qu'il contient ou à qui il est réellement addressé. On peut voir dans le deuxième paquet capturé pendant la phase 1 (**IKE**) que le proposal, choisi par les routeurs, utilise le chiffrement *AES-CBC* :

![proof](images/Q10_proof.png)

---


**Question 11: Expliquez quelles sont les parties du paquet qui sont authentifiées. Donnez l’algorithme cryptographique correspondant.**

---

**Réponse :**

Comme on utilise ESP et pas AH, il n'y a pas de partie authentifiée du paquet. Néanmoins, comme on est en mode tunnel, tout le paquet est encapsulé, ce qui inclus les headers internes. Il y a donc une protection de la part d'ESP pour le paquet IP **interne** mais rien pour le paquet IP **externe**. Une certaine forme d'authenticité est fournie par l'encapsulation.

L'algorithme utilisé pour l'authenticité est *HMAC-SHA1*, comme configuré dans les routeurs (`transform-set STRONG esp-sha-hmac`).
On peut également le déduire, lors de l'échange des proposals entre routeurs :

![proof](images/Q11_proof.png)

---


**Question 12: Expliquez quelles sont les parties du paquet qui sont protégées en intégrité. Donnez l’algorithme cryptographique correspondant.**

---

**Réponse :**

Toute la partie IP encapsulée par ESP est protégée en intégrité par le protocole ESP. Comme l'algorithme utilisé est *HMAC-SHA1* pour l'authenticité, celui protège également l'intégrité du message (car c'est un *MAC*).

---
