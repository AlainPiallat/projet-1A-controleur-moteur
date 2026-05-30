### Connector

!\[Connector schematic](schematics\_puissance.pdf)

Cette partie a fait l'objet d'une étude approfondie afin de garantir l'intégrité des signaux faibles et la robustesse des flux de forte puissance. La connectique est divisée en quatre blocs distincts détaillés ci-dessous.



!\[Interface de puissance](Con1.png)

L'interface de puissance, identifiée par le bloc CON1, gère l'alimentation principale du bus continu et les sorties triphasées vers le moteur. Pour les borniers haute intensité allant de MP1 à MP5, nous avons sélectionné des connecteurs massifs REDCUBE du fabricant Würth Elektronik, portant la référence 74650174. Ces borniers utilisent une empreinte KiCad spécifique de type THR (Through-Hole Reflow) munie de quatre broches traversantes. Cette conception permet de diviser la résistance de contact tout en assurant une stabilité mécanique parfaite lors du serrage des cosses M4. Concernant le filtrage et la protection, l'alimentation positive entre par la borne MP3 et traverse immédiatement un fusible de protection F1 de type Fuse\_Small. Elle alimente ensuite le bus principal VM, lequel est lissé par un banc de trois condensateurs électrolytiques placés en parallèle de 220 microfarads et 100 volts chacun (C11, C12, C13). Ce montage offre un réservoir total de 660 microfarads permettant d'absorber efficacement les appels de courant du moteur. Les sorties de puissance vers le moteur s'effectuent via les bornes MP1 pour U\_out, MP4 pour V\_out et MP5 pour W\_out.



!\[Between board connector](Between\_board\_connector.png)

Le connecteur J3 agit comme le système nerveux central reliant l'étage de puissance à l'intelligence externe. Il s'agit d'une embase mâle à double rangée de 32 broches au pas standard de 2.54 millimètres, numérotée en mode Odd/Even sous la référence Conn\_02x16\_Odd\_Even. Cette interface regroupe de manière ordonnée l'ensemble des flux d'informations de la carte. On y retrouve les commandes de hachage PWM issues de la carte de contrôle, notées GHS1 à GHS3 et GLS1 à GLS3, ainsi que les retours différentiels VSHUNT1P/N à VSHUNT3P/N utilisés pour la mesure du courant des trois phases. J3 centralise également les signaux de surveillance analogique tels que les tensions de phases VOUT1 à VOUT3, la surveillance du bus DC noté VM et l'alerte thermique TEMP\_SENSOR. Enfin, il assure la distribution des alimentations logiques VDD et VHALL, accompagnées de multiples points de masse GND répartis stratégiquement pour garantir un bon retour de courant et limiter le bruit électromagnétique.



!\[Connector Hall](Conn\_hall.png)

Pour l'interface de retour de position du rotor, qui exploite les capteurs à effet Hall, la carte dispose d'un connecteur dédié J1 nommé CONN HAL. Cette séparation physique évite de mélanger les signaux logiques sensibles avec les flux de puissance. Nous avons opté pour un connecteur de la série JST PH, référence B5B-PH-K-S, comportant 5 broches au pas de 2.0 millimètres. Ce standard industriel intègre un verrouillage par friction garantissant une connexion fiable capable de résister aux vibrations mécaniques du moteur. Au niveau du brochage, la broche 1 est reliée à la masse GND et la broche 5 fournit l'alimentation dédiée aux capteurs via le réseau VHALL. Les trois broches centrales 2, 3 et 4 reçoivent quant à elles les signaux logiques de position HALL1, HALL2 et HALL3.

