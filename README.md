# protocole-DHCP
Approfondissement du protocole DHCP

### Qu'est-ce que le protocole DHCP?
```
DHCP signifie Dynamic Host Configuration Protocol. C'est un protocole de la couche Application qui utilise UDP et IP qui permet à
un ordinateur qui se connecte sur un réseau d'obtenir dynamiquement sa configuration
(principalement, sa configuration réseau). Vous n'avez qu'à spécifier à l'ordinateur de se
trouver une adresse IP tout seul par DHCP. Le but principal étant la simplification de
l'administration d'un réseau.
```

Le protocole DHCP sert principalement à distribuer des adresses IP sur un réseau, mais il a été conçu au départ comme extension au protocole BOOTP (Bootstrap Protocol) qui est utilisé par exemple lorsque l'on installe une machine à travers un réseau (BOOTP est utilisé en étroite collaboration avec un serveur TFTP sur lequel le client va trouver les fichiers à charger et à copier sur le disque dur).<br>
Un serveur DHCP peut renvoyer des paramètres BOOTP ou de configuration propres à un hôte donné.

![image](https://user-images.githubusercontent.com/83721477/167026043-ccf9a24a-a6a0-4f8f-95df-550eb6223760.png)


## Fonctionnement du protocole DHCP

* Il faut un serveur DHCP avec une adresse IP fixe.
* Le mécanisme de base de la communication est BOOTP (avec trame UDP). Quand une machine est démarrée, elle n'a aucune information sur sa configuration réseau, et surtout, l'utilisateur ne doit rien faire de particulier pour trouver une adresse IP. 
* On envoi un broadcast pour trouver et dialoguer avec le serveur DHCP (avec d'autres informations comme le type de requête, les ports de connexion...) sur le réseau local.
* Lorsque le serveur DHCP recevra le paquet de broadcast, il renverra un autre paquet de broadcast (n'oubliez pas que le client n'a pas forcement son adresse IP et que donc il n'est pas joignable directement) contenant toutes les informations requises pour le client.

### Listes des paquets DHCP
* DHCPDISCOVER (pour localiser les serveurs DHCP disponibles)
* * DHCPOFFER (réponse du serveur à un paquet DHCPDISCOVER, qui contient les premiers paramètres)
* DHCPREQUEST (requête diverse du client pour par exemple prolonger son bail)
* DHCPACK (réponse du serveur qui contient des paramètres et l'adresse IP du client)
* DHCPNAK (réponse du serveur pour signaler au client que son bail est échu ou si le client annonce une mauvaise configuration réseau)
* DHCPDECLINE (le client annonce au serveur que l'adresse est déjà utilisée)
* DHCPRELEASE (le client libère son adresse IP)
* DHCPINFORM (le client demande des paramètres locaux, il a déjà son adresse IP)

![image](https://user-images.githubusercontent.com/83721477/167024987-fadbca7e-179a-46f2-abe4-9881ca21ea44.png)

### 1. Demande de bail IP par le client
Initialisation du client avec une version limitée de TCP/IP
Le client diffuse  une demande d'adresse IP (message DhcpDiscover) avec :
* source 0.0.0.0
* destination 255.255.255.255

### 2. Offre de bail par un serveur
Tous les serveurs reçoivent la demande (sur le port UDP 67). S'ils sont configurés pour répondre (message DhcpOffer), ils diffusent une offre avec les informations suivantes :
* L'adresse MAC du client
* Une adresse IP
* Un masque de sous-réseau
* Une durée de bail
* Son adresse IP (pour la phase 3)

Un client DHCP attend une offre pendant une seconde. En cas de non réponse il rediffuse sa demande quatre fois (à des intervalle de 9, 13 et 16 secondes puis un intervalle aléatoire entre 0 et 1000 millisecondes). Après ces quatre tentatives, il renouvelle sa demande toutes les 5 minutes.

### 3. Sélection d'un bail par le client
* Le client sélectionne une offre (en général la première)
* Le client annonce par diffusion qu'il a accepté une offre (message DhcpRequest).
* Son message comporte l'identification du serveur sélectionné. Ce dernier sait que son offre a été retenue.
* Tous les autres retirent leur offre et l'adresse redevient disponible.

### 4. Accusé de réception par le serveur
* Le serveur sélectionné accuse réception au client (message DhcpAck).
* Son message contient éventuellement d'autres informations (serveur DNS, Passerelle, etc.)
* Le client stocke la configuration qui lui a été attribuée.

## Note
Au démarrage:
* le client DHCP tente de renouveler son bail avec les paramètres qu'il possède déjà en s'adressant au serveur DHCP à l'origine du bail (message DhcpRequest).
* Si le bail est renouvelé le client continue avec un nouveau bail et éventuellement de nouveaux paramètres (message DhcpAck).
* Si la demande n'aboutit pas, il continue à utiliser les paramètres en vigueur.
