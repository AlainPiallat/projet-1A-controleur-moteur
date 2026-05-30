# Rapport Technique : Étage de Puissance et Systèmes d'Acquisition 

## 1\. Étage de Puissance (Power Stage)

L'onduleur repose sur une architecture à 6 MOSFETs Canal N (répartis en 2 transistors par phase) formant 3 demi-ponts indépendants. Cette structure permet de piloter un moteur Brushless DC (BLDC) en alternant l'activation des transistors pour générer un système de tensions triphasées.

* **MOSFETs High-Side :** Connectés directement au bus d'alimentation principal de la batterie ($V\_M = 50\\text{ V}$).
* **MOSFETs Low-Side :** Connectés en série entre la phase du moteur et les résistances de Shunt pour le retour d'information du courant vers la masse ($GND$).

### Caractéristiques Thermiques du Silicium

Les transistors choisis sont des composants robustes de référence **STB100N6F7**. Selon la documentation technique constructeur, leur résistance interne à l'état passant ($R\_{DS(on)}$) évolue significativement avec la température de jonction ($T\_j$) :

* **À $25^\\circ\\text{C}$ :** $R\_{DS(on)} \\approx 5.6\\text{ m}\\Omega$ (valeur nominale).
* **À $125^\\circ\\text{C}$ :** $R\_{DS(on)} \\approx 8\\text{ m}\\Omega$.

## 2\. Commande de Grille et Prévention du Shoot-Through

Le **shoot-through** (ou cross-conduction) est un court-circuit destructeur provoqué par l'activation simultanée du MOSFET High-Side et du Low-Side d'un même bras. Pour l'éviter, l'architecture met en œuvre deux barrières de protection.

### A. Contrôle Dynamique de la Grille (Asymétrie Charge/Décharge)

Chaque transistor possède un réseau de commande de grille composé d'une résistance en parallèle avec une diode Schottky (marquées `D1`, `D2`, etc.).

* **À l'allumage (Turn-on) :** La diode Schottky est bloquée. Le courant fourni par le driver est contraint de traverser la résistance de grille ($R\_g$). Cela ralentit légèrement la charge de la capacité d'entrée du MOSFET ($C\_{iss}$) et adoucit la transition pour limiter les perturbations électromagnétiques (EMI).
* **À l'extinction (Turn-off) :** La diode Schottky devient passante. Le courant de décharge de la grille contourne la résistance et s'évacue instantanément à travers la diode vers le driver.
### B. Dimensionnement des Résistances de Grille

Le driver intégré au STSPIN32G4 possède un étage de sortie alimenté sous une tension interne $V\_{CC} = 12\\text{ V}$, capable de délivrer ou d'absorber un courant maximal de $I\_{max} = 1\\text{ A}$.

La fiche technique du STB100N6F7 indique qu'une charge totale de grille $Q\_g = 30\\text{ nC}$ est nécessaire pour saturer complètement le canal sous $12\\text{ V}$. Au moment précis de l'amorçage, la capacité de grille est déchargée et se comporte temporairement comme un court-circuit. Le courant de crête est alors limité uniquement par la loi d'Ohm :

$$\\text{R}*{\\text{totale}} = \\frac{V*{CC}}{I\_{max}} = \\frac{12\\text{ V}}{1\\text{ A}} = 12,\\Omega$$

Un temps de commutation optimal théorique s'obtient par la relation :

$$t = \\frac{Q\_g}{I\_{max}} = \\frac{30\\text{ nC}}{1\\text{ A}} = 30\\text{ ns}$$

## 3\. Mécanisme d'Alimentation Flottante (Bootstrap)

Pour maintenir conducteur un MOSFET High-Side (Canal N), la tension appliquée sur sa Grille doit rester supérieure d'au moins $10\\text{ V}$ à celle de sa Source. Or, lorsque ce MOSFET s'active, sa Source (le nœud de phase `U`, `V` ou `W`) monte immédiatement au potentiel de la batterie ($50\\text{ V}$). La grille doit donc atteindre $50\\text{ V} + 12\\text{ V} = 62\\text{ V}$, une tension supérieure au potentiel maximum disponible sur la carte.

Le système utilise l'astuce du condensateur de bootstrap (`C1` de $1,\\mu\\text{F} / 25\\text{ V}$) couplé à une diode interne du STSPIN32G4 agissant comme une pompe à charge :

1. **Phase de Recharge (Low-Side ON) :** Le MOSFET du bas est activé. La phase du moteur (`DUT1` / `U`) est tirée vers la masse ($0\\text{ V}$). Le condensateur de bootstrap `C1` se retrouve connecté entre le rail $12\\text{ V}$ interne et le $0\\text{ V}$. Il se charge instantanément à $12\\text{ V}$.
2. **Phase d'Élévation (High-Side ON) :** Le Low-Side s'éteint et le driver commande l'ouverture du High-Side. Au fur et à mesure que le transistor conduit, le potentiel de la phase monte à $50\\text{ V}$. Le condensateur `C1`, connecté au nœud de phase, voit sa borne négative s'élever à $50\\text{ V}$ ; par effet capacitif, sa borne positive (`BOOT1`) est propulsée à $50\\text{ V} + 12\\text{ V} = 62\\text{ V}$.
3. **Maintien :** Le driver interne commute la ligne `BOOT1` sur la broche de commande de grille **`GHS1`**, appliquant les $62\\text{ V}$ sur la Grille de `Q1`. On conserve ainsi une tension $V\_{GS} = 12\\text{ V}$, maintenant le transistor passant.

## 4\. Circuits d'Acquisition et Capteurs (Analog Sensors)

### A. Condensateurs de Découplage de Bus

Des condensateurs céramiques multicouches (MLCC) de $220\\text{ nF} / 100\\text{ V}$ (`C2`, `C7`, `C10`) sont positionnés au plus près de chaque demi-pont. Leur rôle est d'absorber les pics de tension transitoires induits par les commutations à haute fréquence.

### B. Mesure de Courant par Shunts Basse (Low-Side Shunts)

La mesure du courant global de phase s'effectue au niveau des MOSFETs Low-Side à l'aide de résistances Shunt de très faible valeur ($R\_4 = 1\\text{ m}\\Omega$, calibrée pour dissiper $15\\text{ W}$).

À un courant maximal de $50\\text{ A}$, la chute de tension mesurable aux bornes du shunt est de :

$$V\_{shunt} = R \\cdot I = 0.001,\\Omega \\times 50\\text{ A} = 50\\text{ mV}$$

Cette tension étant extrêmement faible pour être lue directement de manière précise par le convertisseur analogique-numérique (ADC) du microcontrôleur, le signal électrique différentiel est acheminé vers les amplificateurs de détection de courant internes du STSPIN32G4 afin d'exploiter toute la dynamique de l'ADC.

### C. Mesure de la Tension des Phases

Pour mesurer la tension des phases (qui varie de $0\\text{ V}$ à $50\\text{ V}$), un pont diviseur résistif applique un ratio d'atténuation fixe de 0,0526 :

$$V\_{ADC} = 50\\text{ V} \\times 0.0526 = 2.63\\text{ V}$$

La valeur obtenue de $2.63\\text{ V}$ reste inférieure au plafond d'alimentation logique de $3.3\\text{ V}$ du microcontrôleur, évitant ainsi la saturation de la mesure ou la destruction de l'entrée. Des diodes Schottky externes de clamping protègent en outre ces entrées fragiles contre d'éventuels pics de tension.

### D. Protection Thermique par CTN/NTC

La surveillance de la température à proximité des transistors de puissance est confiée à une thermistance à coefficient de température négatif (NTC) de $10\\text{ k}\\Omega$. Elle est insérée dans un pont diviseur avec une résistance fixe de $4.7\\text{ k}\\Omega$ afin de linéariser sa courbe de réponse exponentielle.

Le calcul de la tension de sortie s'établit selon la formule :

$$V\_{out} = \\frac{V\_{CC} \\times R\_{NTC}}{R\_{fixe} + R\_{NTC}}$$

* **À $25^\\circ\\text{C}$ ($R\_{NTC} = 10\\text{ k}\\Omega$) :** $V\_{out} = \\frac{3.3\\text{ V} \\times 10\\text{ k}\\Omega}{4.7\\text{ k}\\Omega + 10\\text{ k}\\Omega} \\approx 2.24\\text{ V}$
* **À $80^\\circ\\text{C}$ ($R\_{NTC} \\approx 1.2\\text{ k}\\Omega$) :** $V\_{out} = \\frac{3.3\\text{ V} \\times 1.2\\text{ k}\\Omega}{4.7\\text{ k}\\Omega + 1.2\\text{ k}\\Omega} \\approx 0.67\\text{ V}$

Un condensateur de filtrage C5 de $33\\text{ nF}$ est placé en parallèle pour former un filtre passe-bas destiné à éliminer le bruit électrique haute fréquence généré par le découpage PWM du moteur.
## PCB

Pour le PCB il a été choisit de le faire en 2 couches pour en simplifier la conception étant donné que la taille de celui ci dépend principalement de la taille des pistes de puissance

### Répartition des couches

* Couche 1 (Top) : cette couche contient tout les composants à l'exception du connecteur avec la carte de puissance qui doit être situé sur la couche du dessous) ainsi que la majorité des pistes de signal et est dédiée a la masse.
* Couche 2 (Bottom) : cette piste sert principalement aux pistes de puissance, d'autres pistes qui ne rentraient pas sur la couche 1 on été mises en minimisant leurs longueurs pour garder une masse au plus dégagée. Le connecteur pour la carte de contrôle se trouve sur cette couche pour permettre l'installation de radiateur sur les composants de puissance de la couche 1

Le déplacement des courants importants entre les deux couches se font grâce a plusieurs dizaines de vias qui après s'être remplis d'étain forment une connexion assez épaisse pour supporter le courant.

### Largeur des pistes

* Les pistes de puissance : cette catégorie regroupe toutes les liaison entre l'alimentation et le moteur, elles doivent supporter des intensités importantes c'est pour cela que leur largeur est de l'ordre du centimètre et qu'elles sont dénudées pour y souder un fil ou une découpe de cuivre par dessus ce qui augmente l'épaisseur de conduction pour permettre un passe de courant important
* Les pistes de signal/faible puissance : cette catégorie regroupe toutes les autres pistes, leur épaisseur a été choisie de 0.5 mm sauf près du connecteur avec la carte de contrôle qui ne permet pas cette largeur, dans ce cas elles sont de 0.3mm

### isolation

&#x20;une isolation minimal de 0.25mm a été imposé entre toutes les pistes et les pads. Aucun composant de puissance ne possède de frein thermique pour maximiser l'épaisseur de la connexion.

### Positionnement des composants

Les composants principaux, c'est à dire les composants de puissance on été positionné de sorte que les pistes soient le plus droites possibles pour faciliter le soudage des fil ou découpe de cuivre par dessus.

Tout les autres composants on été positionné de sorte que les composants de puissance puissent être refroidis par un radiateur.

Nous avons pris soin de positionner la thermistance proche des composants de puissance pour mesurer au mieux la température de ceux-ci.

Les trous de fixation et le connecteurs ont été positionné pour permettre compatibilité avec la carte de contrôle.
