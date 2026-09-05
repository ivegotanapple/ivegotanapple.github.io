# TP 1 Reseaux

## Introduction
Dans ce TP de 2h nous allons monter un cluster de machines en utilisant le systeme [proxmox VE](https://proxmox.com/), système d'exploitation (abrégé OS dans le reste du TP) basé sur debian. Proxmox permet de gérer des machines virtuelles et des conteneurs dans un cluster de machines. Par rapport au reseau local notre cluster se comportera comme un sous reseau classique. Nous détaillerons les particularités de proxmox le moment venu.

Pour ce TP vous avez à votre disposition le matériel suivant :
- 3 mini-pc (PC) et 1 cable ethernet pour chaque
- 1 machine 'reseau' (ROUTER) qui possède 5 ports ethernet: 3 en RJ45 et 2 en SPF <détailler>
- 1 switch et son cable SPF pour le uplink
- 1 petit écran et son cable hdmi 
- 1 clavier et souris à dongle
- 1 alimentation par appareil (6 en tout)
- 1 clé USB contenant l'installer de proxmox

Le routeur est désigné dans notre configuration pour 
L'écran et le clavier doivent vous servir à consulter l'état des mini-pc une fois démarrés. Dans une configuration réelle, sauf cas particulier, on configurerais le service ssh pour pouvoir accéder aux machines sans devoir brancher/débrancher sans arrêt des périphériques sur les machines.

> [!CAUTION]
> Il va sans dire que l'on compte bien retrouver l'ensemble du matériel prêté à la fin du TP sinon quoi vous vous exposez à des sanctions

> [!NOTE]
> La durée donnée à coté du titre de chaque  exercice est donné à titre informatif

## Exercice 1 (30 minutes)

### A) dessiner reseau av. convention de nommage (10-15min)

### B) Installer proxmox sur le routeur (15 minutes)
Vous devez d'abord insérer la clé USB dans la machine routeur, brancher les cables (alimentation, hdmi) puis allumer la machine. À partir de la 2 possibilités existent, soit : 
1. La machine démarre directement sur la clé. Pas de problème vous pouvez continuer l'installation. 
2. La machine démarre sur l'OS précédemment installé (a priori proxmox). Nous souhaitons réinstaller systématiquement l'OS pour ce TP. Auquel cas il vous faudra éteindre la machine puis accéder à l'UEFI[^1] pour démarrer en partant de la clé. Pour accéder à l'UEFI, il faut, juste après le redémarrage de la machine, appuyer sur une touche 'spéciale' du clavier pour y accéder. En général il s'agit d'Échap ou une combinaison de touche Fonction. Dans notre cas il s'agit de la touche Suppr.

> [!NOTE]
> On note qu'avant l'UEFI il existait un microprogramme appelé BIOS qui accomplissait les même tâches. Les deux termes sont parfois utilisés de manière interchangeable.

3. Sur l'écran de démarrage choississez "Install Proxmox with Terminal UI" (la fenêtre d'installation graphique ne rentre pas dans le petit écran)
4. Acceptez les conditions d'utilisation, choississez la région et la disposition clavier (!). On vous demande ensuite de choisir un mot de passe pour l'utilisateur root du système. Dans ce TP nous irons à l'encontre de toute recommandation en matière de cybersécurité en vous recommandant de choisir un mot de passe très simple et identique pour chaque machine. (Votre enseignant n'aimerais pas à avoir réinitialiser votre mot de passe). Pour le mail vous pouvez mettre n'importe quoi au format mail (`a@b.c`). À notez que si vous montez un jour votre propre serveur vous devriez mettre un mail où vous êtes joignable...
5. Entrez le nom de domaine et l'IP. Pour des raison de praticité nous avons déjà choisi les noms de domaines des machines qui seront :
- `router.home` pour la machine 'ROUTER'
- `PC1.home`, `PC2.home` ou `PC3.home` pour les mini-pc (voir l'étiquette dessus)
- Dans `IP address` renseignez l'IP que vous avez choisi dans la premiere partie. <Comme nous avons crée un sous réseau vous devez aussi changer l'adresse de la passerelle et du serveur DNS.>
6. Passez à l'écran suivant et selectionnez 'Install'. Il faudra retirer la clé entre le moment ou vous selectionnez Install et celui où la machine redémarre, sinon vous redémarrerez sur l'installer proxmox au lieu du système !
7. Attendez que le système démarre. Une fois sur l'écran de login. (Écran disponible ici seulement en console, on dit aussi CLI[^2]). Au dessus vous devriez voir un lien du type : `https://<VOTRE_IP>:8006`. Il s'agit de l'adresse de l'interface web de proxmox. Nous allons continuer les manipulations dessus.

## Exercice 2 (10 minutes)

Comme vu précédement, nous allons créer un cluster de machines. Toutes les machines que nous allons relier au cluster feront parti du même sous réseau. Pour faciliter la gestion du réseau global (WAN[^3]) au niveau du routeur, nous allons créer un *pont*, ou *bridge*[^4].  

Connectez vous à l'interface web de proxmox sur le routeur avec l'identifiant `root` et le mot de passe que vous avez fourni à l'installation. Une fois que c'est fait, allez dans l'onglet **Datacenter → router → System → Network**. Cliquez sur **Create → Linux Bridge**. On vous demande de remplir un formulaire : Dans le champ `Name` mettez `vmbr1` (`vmbrX` étant la convention de nommage des bridges). Dans le champ `Bridge ports` mettez l'interface à laquelle le switch sera relié. On choisira ici une des deux interfaces SFP (nommées nic3 ou nic4).

Cliquez sur **Create**, puis sur **Apply Configuration** en haut de la fenêtre du network. Votre pont est crée !

Pour assurer que les machines du sous réseau puissent acceder à internet nous allons configurer l'IP forwarding sur la machine proxmox. Cliquez sur le noeud `router` puis dans l'onglet `Shell` du panneau qui s'est ouvert.

Nous vous proposons deux méthodes pour obtenir l'IP forwarding, l'une est temporaire et l'autre permanente. Les voici:
1. Temporaire: Entrez la commande `sudo sysctl -w net.ipv4.ip_forward=1`
2. Permanent: Éditez le fichier `/etc/sysctl.conf` (avec nano ou vi par exemple) pour y ajouter la ligne `net.ipv4.ip_forward=1`

## Exercice 3 (30 minutes)

On passe à la création du cluster proprement dit.

- relier chaque PC au sous reseau puis cluster (10min par PC)
	- installer proxmox
		- brancher ethernet sur routeur
		- shell router ip link set if up
		- verifier avec ip a
		- add adresse statique
		- debrancher/rebrancher PC sur switch
		- sur PC lancer tcpdump -i itf icmp (possibilité ecoute tcp/udp etc)
		- ping le PC avec l'ip attribuée et observer la reception des pings
	- join cluster
		- cliquer sur datacenter > cluster > join information 
		- cliquer sur copy information (pas besoin de ctrl+c/v)
		- se connecter au node en web 
			- brancher cable
			- ip link set / ip addr add de chaque coté
			- mettre 2 ip diff !!!!
		- datacenter > cluster > join cluster > coller le contenu du presse papier
		- attendre le message "task done", sinon verifier que vos ip sont bien config (l'ip copiée n'est pas dans le reseau du node mais proxmox devrais chercher dans le reseau disponible et trouver quand meme)
		- repeter pour les 3 autres machines (1 eleve par manip)
## Glossaire

[^1]:UEFI
: *Unified Extensible Firmware Interface* : le logiciel de bas niveau qui fait le lien entre les composants matériels d'un ordinateur et son système d'exploitation lors du démarrage

[^2]:CLI
: *Command Line Interface* : l'outil textuel qui permet de contrôler un ordinateur ou un logiciel en tapant des commandes au clavier, au lieu de cliquer avec une souris. C'est un héritage des tout premiers ordinateurs qui n'avaient souvent pas d'écran mais une imprimante.

[^3]:WAN
: *Wide Area Network* : le réseau informatique qui relie des ordinateurs et des réseaux locaux (LAN) sur une grande zone géographique, comme un pays ou le monde entier.

[^4]:Pont réseau
: Un *pont réseau* (ou *network bridge*) est un équipement ou une fonction logicielle de la couche 2 (liaison de données) du modèle OSI qui relie deux segments de réseau local pour les faire communiquer comme un seul et même réseau.