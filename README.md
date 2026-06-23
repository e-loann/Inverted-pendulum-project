Bonjour, pendant ma deuxième année de prépa aux écoles d'ingénieur, j'ai réalisé dans le cadre de ce qu'on appelle en France le "TIPE" un pendule inversé simple sur chariot. Outre le fait d'avoir réussi mon projet, je vais ici détaillé comment j'ai abordé le sujet et quels points ont également posé problème. 

Note : je n'ai absolument pas la prétention de maîtriser la totalité du sujet car l'essence même du concept de pendule inversé repose sur des théories d'automatisation et d'asservissement très complexes, c'est pourquoi je ne ferai part que de ce j'ai compris et ce qui m'a permis d'aboutir à un pendule inversé fonctionnel. Il se peut également que je puisse être imprécis ou faire des erreurs.

##Présentation du problème : 

Prenez un manche de balai et essayez de le tenir en équilibre sur le bout des doigts, c'est relativement facile mais cela peut s'avérer plus compliqué si l'on modifie des paramètres physiques tel que la longueur de celui-ci par exemple. Et bien le pendule c'est exactement la même chose sauf que l'on va automatiser le système pour stabiliser une tige en équilibre sans l'intervention directe d'un humain.

##Matériel utilisé :

Hardware :

- Carte Arduino Mega
- 2 encodeurs rotatifs (2400CPR)
- un moteur d'essuie glace Renault (je n'ai pas la référence exacte)
- 2 poulies GT3 et une courroie, un shield moteur (DBH-12)
- 2 rails (80cm de longueur)
- une tige filetée (5 mm de diamètre).

Logiciels :

-Arduino (pour la programmation et l'acquisition des données)
-Matlab (calculs/simulations et que j'ai utilisé gratuitement en version étudiante)
-Fritzing (seulement pour l'illustration du schéma électrique)


##Conception du système : 

C'est clairement la partie qui m'a prise le plus de temps. J'ai fait le choix de créer un pendule inversé sur chariot guidée en translation sur rails et non pas avec des roues directement sur le sol pour la simple raison que je n'aurais qu'un seul et non pas deux moteurs à commander. J'ai dégoté des rails de 80cm dans l'établissement où j'étudie et sur mon temps libre, j'ai découpé et poncé deux planches de bois sur laquelle j'ai fixé les deux rails, en faisant attention à laisser un écart suffisant entre les rails pour y placer ensuite un capteur d'une part et le moteur d'autre part. 

Ensuite j'ai commandé deux moteurs à courant continu mais en les testant, je me suis vite rendu compte qu'ils n'auraient pas assez de couple pour ne serait-ce faire bouger le chariot. Le problème est qu'il faut un moteur qui soit réactif et puissant, c'est à dire, trouver le bon compromis entre vitesse de rotation élevée et fort couple et cela sans se ruiner (je dis ça car en cherchant sur le site de Maxon Motor, pour obtenir de la qualité, les prix ne sont rarement en deçà de 100$). Je suis finalement tombé par hasard sur un moteur d'essuis glace de récupération et en le testant, je me suis vite rendu compte qu'il convenait parfaitement car très réactif et énorme couple. Seul problème, pour les moteurs précédents, j'utilisais shield moteur L298N (1A max) qui ne convient pas au moteur d'essuis glace qui demande en moyenne 4A-5A pour fonctionner et peut atteindre des pics à 15A-30A lors des changements brusques de rotation par exemple. J'ai donc remplacé le L298N par le DBH-12, qui encaisse jusqu'à 30A et 12V ce qui est parfait avec mon moteur. 

Je n'ai pas parlé du chariot en lui-même car c'est plutôt simple, j'ai utilisé une planche de bois sur laquelle j'ai fixé 4 supports avec douilles à billes pour permettre une translation sans trop de frottements, et j'y ai de l'autre coté fixé le pendule. Pour ce faire, j'ai fixé un des deux support pour les encodeurs imprimés en 3D et avec une autre pièce imprimé en 3D, fixé la tige filetée à l'encodeur.

Puis, j'ai fixé le second encodeur et le moteur de façon à ce qu'avec une courroie en caoutchouc, et deux poulies installées sur l'arbre moteur et sur la tige de l'encodeur, celle-ci forme une boucle qui se ferme sur une pièce imprimée en 3D et fixée sous le chariot. 

Enfin, j'ai imprimé une énième pièce en 3D sur mesure pour maintenir le moteur en pensant à faire les fixations pour les vis à bois de telle sorte que je puisse les dévisser, décaler la pièce et le moteur puis les revisser de façon à tendre la courroie si elle venait à se détendre (cela m'a simplifié bien des problèmes).


##Modélisation théorique du système :

Pour l'étude théorique, j'ai établis les deux équations de mouvement du système à l'aide de deux PFD, l'un en faisant un théorème de la résultante dynamique sur l'axe x et l'autre avec un théorème du moment dynamique sur l'axe z en isolant d'abord la {tige} seule puis l'ensemble {chariot + tige}.  

### Modèle non-linéaire

$$
\begin{cases}
(M + m)\ddot{x} + d\dot{x} + \frac{mL}{2}\ddot{\theta}\cos(\theta) - \frac{mL}{2}\dot{\theta}^2\sin(\theta) = F \\
\frac{mL^2}{3}\ddot{\theta} + \frac{mL}{2}\ddot{x}\cos(\theta) - mg\frac{L}{2}\sin(\theta) = 0
\end{cases}
$$

$$
\text{linéarisation} \quad \Downarrow
$$

### Modèle linéarisé

$$
\begin{cases}
(M + m)\ddot{x} + \frac{mL}{2}\ddot{\theta} = F \\
\frac{L}{3}\ddot{\theta} + \frac{1}{2}\ddot{x} - \frac{g}{2}\theta = 0
\end{cases}
$$




