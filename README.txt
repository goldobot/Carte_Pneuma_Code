
########## INFOS CARTE PNEUMA ##########

### Actionneurs et protocoles associés
---
Actionneurs/capteurs gérés par la nucléo:
	Canons (pour propulsion des cerises):
		ESC Canon 1 : D-Shot
		ESC Canon 2 : D-Shot
		ESC Canon 3 : D-Shot
	Électrovannes (pour actionner piston de la pince gâteaux n°x OU pour purger le réservoir):
		Électrovanne 1 (n°1) : TOR
		Électrovanne 2 (n°2) : TOR
		Électrovanne 3 (n°3) : TOR
		Électrovanne 4 (purge) : TOR
	ESC Turbine (pour aspiration cerises) : D-Shot
	ESC Compresseur (pour réservoir) : D-Shot
	Capteur de pression (pression du réservoir) : I2C ou analog input
	Capteur de température (pour Compresseur réservoir) : I2C ou analog input


### Fonctionnalités des actionneurs
---
Cahier des charges des actions à réaliser :
	Canons (pour propulsion des cerises):
		Démarrer/Arrêter les moteurs canon (pas tous à la même vitesse, à voir en fonction de la balistique) lorsqu’on veut éjecter des cerises.
		Sélectionner au moins 3 niveaux de vitesse, différent pour chaque moteur
	Électrovannes 1 à 3: activer en TOR les EV lorsqu’on choppe les gâteaux avec les barillet.
	Électrovanne 4: purger le réservoir en fin de match/lorsque l'arret d'urgence est enclenché. optionnelement la purge servira à souffler les cerises devant le robot
	ESC Turbine : Activer lorqu’on veut aspirer des cerises.
	ESC Compresseur :
		Activer en fonction de la pression du capteur de pression du réservoir pour garder une pression stable dans le réservoir.
		Retourner à la Rpi la pression courante du réservoir entre 0.0 et 10.0 bar par incrément de 0.1 (soit moins de 128 valeurs nécessaire) si on veut se limiter à 7bits)
	Capteur de pression : lire en continu les valeurs de pression du réservoir
	Capteur de température : lire en continu et inhiber le compresseur si température critique (l'erreur doit être remonté à la Rpi).


### Protocole de com Raspi <--> Nucleo
---
Information binaire minimales requises, structure "mots" 8bits en écriture (W): [XXZZZZZZ]
- [II] : instruction:
	> [00]: reset [00000000] ou arrêt compresseur seul [00ZZZZZZ]
	> [01]: canon
	> [10]: compresseur
	> [11]: Turbine+EV
- [ZZZZZZ] : valeurs, voir ci-dessous:

W/R Canons, 3x2 bits : [GGDDHH]
	> [00] : arrêt
	> [01] ; [10] et [11] : vitesse 1 ; 2 et 3
W/R Compresseur/reservoir, 6 bits : [PPPPPP]
	> [000000] : arret du maintient en pression et purge
	> [PPPPPP] : mise en pression du compresseur à la valeur consigne (=[PPPPPP] x 0,1bar) qui ne dépassera jamais 6,3 bar (grace à la soupape de sécurité)
R 	Temperature (optionnel si les trains de mots ne sont pas possible, sinon elle fait suite au R du compresseur), 6 bits :[TTTTTT]
	> [TTTTTT] : température 0-160° (=[TTTTTT] x 2,5°C)
W/R	Electrovane, 4x1 bits : [ABCE]
	> [0] : arrêt, pas de pression/pince ouverte
	> [1] : active, pression/pince fermée
W/R	Turbine, 1x2 bits : [FF]
	> [00] : arrêt, pas de pression/pince ouverte
	> [01] ; [10] et [11] : vitesse 1 ; 2 et 3
	
	

### Informations Pinout
---
- PA2 et PA15 ⇒ UART_TX/RX car hard wired sur la carte nucléo (voir schematics F303K8)