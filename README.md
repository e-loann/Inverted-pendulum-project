Bonjour, pendant ma deuxième année de prépa aux écoles d'ingénieur, j'ai réalisé dans le cadre de ce qu'on appelle en France le "TIPE" un pendule inversé simple sur chariot. Outre le fait d'avoir réussi mon projet, je vais ici détaillé comment j'ai abordé le sujet et quels points ont également posé problème. 

### Présentation du problème : 

Prenez un manche de balai et essayez de le tenir en équilibre sur le bout des doigts, c'est relativement facile mais cela peut s'avérer plus compliqué si l'on modifie des paramètres physiques tel que la longueur de celui-ci par exemple. Et bien le pendule c'est exactement la même chose sauf que l'on va automatiser le système pour stabiliser une tige en équilibre sans l'intervention directe d'un humain.

### Matériel utilisé :

Hardware :

- Carte Arduino Mega
- 2 encodeurs rotatifs (2400CPR)
- un moteur d'essuie glace Renault (je n'ai pas la référence exacte)
- 2 poulies GT3 et une courroie, un shield moteur (DBH-12)
- 2 rails (80cm de longueur)
- une tige filetée (5 mm de diamètre).

Logiciels :

- Arduino (pour la programmation et l'acquisition des données)
- Matlab (calculs/simulations et que j'ai utilisé gratuitement en version étudiante)
- Fritzing (seulement pour l'illustration du schéma électrique)


### Conception du système : 

C'est clairement la partie qui m'a prise le plus de temps. J'ai fait le choix de créer un pendule inversé sur chariot guidée en translation sur rails et non pas avec des roues directement sur le sol pour la simple raison que je n'aurais qu'un seul et non pas deux moteurs à commander. J'ai dégoté des rails de 80cm dans l'établissement où j'étudie et sur mon temps libre, j'ai découpé et poncé deux planches de bois sur laquelle j'ai fixé les deux rails, en faisant attention à laisser un écart suffisant entre les rails pour y placer ensuite un capteur d'une part et le moteur d'autre part. 

Ensuite j'ai commandé deux moteurs à courant continu mais en les testant, je me suis vite rendu compte qu'ils n'auraient pas assez de couple pour ne serait-ce faire bouger le chariot. Le problème est qu'il faut un moteur qui soit réactif et puissant, c'est à dire, trouver le bon compromis entre vitesse de rotation élevée et fort couple et cela sans se ruiner (je dis ça car en cherchant sur le site de Maxon Motor, pour obtenir de la qualité, les prix ne sont rarement en deçà de 100$). Je suis finalement tombé par hasard sur un moteur d'essuis glace de récupération et en le testant, je me suis vite rendu compte qu'il convenait parfaitement car très réactif et énorme couple. Seul problème, pour les moteurs précédents, j'utilisais shield moteur L298N (1A max) qui ne convient pas au moteur d'essuis glace qui demande en moyenne 4A-5A pour fonctionner et peut atteindre des pics à 15A-30A lors des changements brusques de rotation par exemple. J'ai donc remplacé le L298N par le DBH-12, qui encaisse jusqu'à 30A et 12V ce qui est parfait avec mon moteur. 

Je n'ai pas parlé du chariot en lui-même car c'est plutôt simple, j'ai utilisé une planche de bois sur laquelle j'ai fixé 4 supports avec douilles à billes pour permettre une translation sans trop de frottements, et j'y ai de l'autre coté fixé le pendule. Pour ce faire, j'ai fixé un des deux support pour les encodeurs imprimés en 3D et avec une autre pièce imprimé en 3D, fixé la tige filetée à l'encodeur.

Puis, j'ai fixé le second encodeur et le moteur de façon à ce qu'avec une courroie en caoutchouc, et deux poulies installées sur l'arbre moteur et sur la tige de l'encodeur, celle-ci forme une boucle qui se ferme sur une pièce imprimée en 3D et fixée sous le chariot. 

Enfin, j'ai imprimé une énième pièce en 3D sur mesure pour maintenir le moteur en pensant à faire les fixations pour les vis à bois de telle sorte que je puisse les dévisser, décaler la pièce et le moteur puis les revisser de façon à tendre la courroie si elle venait à se détendre (cela m'a simplifié bien des problèmes).


Pour l'étude théorique, sans masse au bout de la tige, j'ai établis les deux équations de mouvement du système à l'aide de deux PFD, l'un en faisant un théorème de la résultante dynamique sur l'axe x et l'autre avec un théorème du moment dynamique sur l'axe z en isolant d'abord la {tige} seule puis l'ensemble {chariot + tige}.  


<img width="2752" height="1536" alt="png" src="https://github.com/user-attachments/assets/9d6d96fd-84c9-4f49-8371-e14ddc416fbb" />

On obtient ces équations non linéaires .


$$
\begin{cases}
(M + m)\ddot{x} + d\dot{x} + \frac{mL}{2}\ddot{\theta}\cos(\theta) - \frac{mL}{2}\dot{\theta}^2\sin(\theta) = F \\
\frac{mL^2}{3}\ddot{\theta} + \frac{mL}{2}\ddot{x}\cos(\theta) - mg\frac{L}{2}\sin(\theta) = 0
\end{cases}
$$


Où : 
- x est la position du chariot
- θ est l'angle de la tige par rapport à la verticale
- g est l'accélération de la pesanteur terrestre
- M est la masse du chariot 
- m est la masse de la tige
- d est le coefficient de frottements visqueux
- l est la longueur de la tige
     

On peut linéariser le système autour du point d'équilibre instable ($\theta \simeq 0$) et en négligeant les frottements, ce qui donne le système suivant :

$$
\begin{cases}
(M + m)\ddot{x} + \frac{mL}{2}\ddot{\theta} = F \\\\
\frac{L}{3}\ddot{\theta} + \frac{1}{2}\ddot{x} - \frac{g}{2}\theta = 0
\end{cases}
$$

### Correcteur LQR

J'ai choisi de commencer par un asservissement avec un correcteur LQR. La dynamique de ce système se caractérise par l'équation d'état classique :

$$
\dot{X} = AX + BF
$$

Où :
* **$F$** représente l'action du moteur (la force appliquée au chariot).
* **$X$** est le vecteur d'état du système, défini par :

$$
X = \begin{bmatrix} x \\\\ \dot{x} \\\\ \theta \\\\ \dot{\theta} \end{bmatrix}
$$

Cette équation représente la physique du système. Sans action du moteur (F=0), on a $\dot{X} = AX$ ou plutôt, $\dot{X} - AX = 0$ qui est une équation différentielle classique en X(t) et qui a pour solution une exponentielle divergente ce qui confirme la nature instable du système sans asservissement.


Grâce aux équations différentielles précédentes, on identifie facilement la matrice de dynamique $A$ et la matrice de commande $B$ :

$$
A = \begin{bmatrix} 
0 & 1 & 0 & 0 \\\\ 
0 & 0 & -\frac{3mg}{4M+m} & 0 \\\\ 
0 & 0 & 0 & 1 \\\\ 
0 & 0 & \frac{6(M+m)g}{L(4M+m)} & 0 
\end{bmatrix}
\quad \text{et} \quad
B = \begin{bmatrix} 
0 \\\\ 
\frac{4}{4M+m} \\\\ 
0 \\\\ 
-\frac{6}{L(4M+m)} 
\end{bmatrix}
$$

On peut faire l'application numérique mais elle n'est pas utile par la suite dans mon cas.

Il s'agit ensuite de déterminer les matrice Q et R qui minimisent la fonction de coût suivante : 

$$
J = \int_{0}^{\infty} (X^T Q X + u^T R u) dt
$$

Avec 

$$Q = \begin{bmatrix}
q_x & 0 & 0 & 0 \\
0 & q_{\dot{x}} & 0 & 0 \\
0 & 0 & q_\theta & 0 \\
0 & 0 & 0 & q_{\dot{\theta}}
\end{bmatrix}
$$

La matrice "de coût aux écarts", c'est à dire que chaque coéfficient diagonal sert à pondérer la réactivité du pendule en agissant sur ses différents paramètres (vitesse, vitesse angulaire, potition et angle). Par exemple, pour $q_θ$, plus cette valeur sera élevée, plus le système va pénaliser l'écart à la verticale et inversement. Le contrôleur n'hésitera pas à sacrifier la position du chariot (si $q_x$ est plus faible par exemple) pour rattraper la chute du pendule.

De la même manière, $R$ est une matrice 1x1, et représente de "coût des efforts", c'est à dire de l'énergie que le contrôleur va permettre au moteur de consommer.

Pour trouver ces 5 valeurs, on utilise le critère de Bryson qui est :

$$
q_i = \frac{1}{\text{Val}_{\max}^2}
$$

pour les coéfficients de la matrice $Q$, ou bien :



$$
R = \frac{1}{\text{U}_{\max}^2}
$$

pour R où Val<sub>max</sub> et U<sub>max</sub> sont les valeurs de position, d'angle, de vitesse, de vitesse de rotation ou de tension que l'on "tolère" ou possède. 

Par exemple si on veut calculer la valeur de $q_θ$, si on tolère un écart max d'environ 3° par rapport à la verticale, c'est à dire, environ 0,05 rad, on aura donc $q_θ = \frac{1}{0.05^2}$ soit $q_θ = 400$. De même, étant donné que la tension maximale que je puisse fournir au moteur avec mon alimentation est de 12V, je vais obtenir $R = \frac{1}{12^2}$ soit R = 0,007.


Sachant que notre commande moteur doit s'effectuer par feedback, c'est à dire avec rétroaction, on doit avoir une expression de la commande moteur du type :

$$
F = -KX
\quad \text{avec pour rappel} \quad 
X = \begin{bmatrix} x \\\\ \dot{x} \\\\ \theta \\\\ \dot{\theta} \end{bmatrix}
\quad \text{et où} \quad 
K = \begin{bmatrix} k1 \\\\ k2 \\\\ k3 \\\\ k4 \end{bmatrix}
$$

Note : on a vu que $\dot{X} = AX + BF$ pour le système commandé et on a $F = -KX$ donc $\dot{X} = (A-BK)X$ et par le calcul il est possible de montrer que les valeurs propres de la matrice $A-BK$ sont exactement les poles du système, et en déterminant les gains $k1$, $k2$, $k3$ et $k4$, on peut assurer la stabilité du système si la partie réelle de toutes les valeurs propres sont strictement négatives.


Pour déterminer la matrice de gain K, la théorie suggère de résoudre l'équation de Riccati, or je ne m'y suis pas attardé puisqu'il existe une fonction LQR déjà implémentée dans Matlab. J'ai donc crée un programme Matlab où j'y ai défini toutes les matrices avec la valeur de chaque grandeur physique et ai appelé cette même fonction. Le programme me renvoie alors des valeurs pour $k1$, $k2$, $k3$ et $k4$ qui vont me permettre de rendre le système stable.


### Arduino

Après avoir expliqué le fonctionnement du correcteur, je me dois de parler de la partie Arduino. Voici le schéma du montage avec lequel j'ai fait fonctionner mon pendule inversé : 

<img width="732" height="534" alt="Capture d’écran 2026-06-02 154851" src="https://github.com/user-attachments/assets/13ba65d0-6205-4007-8976-682513a074c4" />

On distingue deux parties du montage. D'abord la partie puissance à droite, composée de l'alimentation, le moteur raccordé au PWM du DVBH12. Et de l'autre coté, la partie commande, composée des deux encodeurs branchés sur les interrupt pins de la Arduino Mega et de la breaboard facilitant le montage. 

Attaquons-nous maintenant au coeur du programme de l'arduino ! 











