# Projet de S4-TI - détection de visages

## Algorithme LBP

Le modèle binaire local (LBP en anglais) sont des caractéristiques utilisées en vision par ordinateur pour reconnaître des textures.

### Définition d'une image numérique

Une image est définie par un nombre de pixels en hauteur et en largeur. Chaque pixel est la résulante de la synthèse additive de nuance de rouge, bleu, vert (codage RGB).
Chaque nuances de couleurs sont codées sur 8 bits donc elles peuvent prendre 256 valeurs : 0 à 255.


<img src="rgb-cymk_01.gif" width=400 align=left>

<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>


### Principe

Le principe est d'étiqueter les pixels d'une image en seuillant le voisinage de chaque pixel et considèrer le résultat comme un nombre binaire sur 8 bits.
Ce traitement est possible si on prend pas en compte les pixels des bords de l'image car ils ne possèdent pas assez de pixels à leur voisinnage.

On prend les 8 pixels dans l'ordre des aiguilles d'une montre autour du pixel choisi :
* On compare chaque pixels avoisinnants avec le pixel choisi.
<br/>
<img src="Capture1.png" width=100 align=left>
<br/>
<br/>
<br/>
<br/>
<br/>
* Si le pixel au voisinage est plus grand ou égal que le pixel choisi alors le pixel avoisinnant prend la valeur binaire "1" sinon il vaut "0".
<br/>
<img src="Capture2.png" width=100 align=left>
<br/>
<br/>
<br/>
<br/>
<br/>
* Le premier pixel comparé correspond au LSB (bit 0) et le dernier au MSB (bit 7).
<br/>
<img src="Capture3.png" width=100 align=left>
<br/>
<br/>
<br/>
<br/>
<br/>
* Le pixel choisi se voit attribué la valeur décimal des bits avoisinnants.
<br/>
<img src="Capture4.png" width=300 align=left>
<br/>
## Entraînement du classifieur cascade

Le classifieur cascade permet théoriquement de reconnaître des visages en entraînant ce classifieur de la manière suivante.

* Images négatives

Les images négatives sont des images qui ne contiennent le visage avec lequel on veut entrainer le classifieur. Il nous faut un grand nombre d'images négatives (environ 2000), pour en acquérir autant nous ne pouvons pas le faire à la main ce qui nous prendrait énormement de temps c'est pour cela qu'on va utliser un petit programme en langage Python.
Ce script a besoin pour fonctionner des modules "OpenCV" pour le traitement des images, "Numpy" pour les calculs numériques et "Urllib" pour gérer les URLs.
Le script est écrit de la manière suivante: 
<br/>

 <img src="scipt_images_négatives.png" width=900 align=left> 
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>

* Images positives

Les images positives sont des images qui contiennent le visage à reconnaître. Il en faut aussi un très grand nombre mais il faut aussi connaître les coordonnées de la postition du visage dans l'image ainsi que le nombre d'occurence du visage.
<br/>
<br/>
On ne peut pas réaliser cela à la main puisque cela serait trop long. Par chance, OpenCV propose un outil, opencv_createsample, permettant de créer des images positives à partir d’une seule image d’entrée.
<br/>
<br/>
Nous avons donc besoin d’une image contenant uniquement notre visage à reconnaître et de créer un nouveau dossier "positives" qui contiendra les images positives. Afin d'utiliser opencv_createsample, on écrit la commande de la manière suivante dans le terminal : "opencv_createsamples -img stop.png -bg negatives.txt -info positives/positives.lst -pngoutput -num 600". 
<br/>
<br/>
Cette commande nous permet de générer 600 images positives à partir des images négatives et un fichier "positives.lst" qui contient les informations relatives aux images positives.
Le fichier ressemble à ceci :
<br/>

<img src="Capture-d’écran-2019-03-11-à-19.30.36.png" width=500 align=left> 
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>

Le resultat obtenu ressemble à ceci :
<br/>
<br/>
<img src="0050_0215_0113_0176_0176.jpg" width=500 align=left>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>


* Fichier descripteur

Maintenant que nous avons nos images positives, nous devons créer un fichier binaire rassemblant toutes les images positives, afin de les utiliser pour l’apprentissage. Pour ce faire, nous allons une fois de plus utiliser open_cv_create_sample qui nous permet facilement de générer ce fichier. 
<br/>
<br/>
On écrit la commande de la manière suivante dans le terminal : "opencv_createsamples -info positives/positives.lst  -num 600 -w 24 -h 24 -vec positives.vec"
<br/>
<br/>
Les arguments pris en compte sont la liste des images positives (-info), la destination de notre fichier de sortie (-vec), le nombre d’échantillons à prendre en compte (-num) et la taille en pixel des échantillons (-w et -h).

*  Apprentissage du classificateur

Tout est prêt, nous pouvons passer à l’apprentissage du classificateur.

Nous commençons par créer un dossier data qui contiendra notre classificateur. Puis nous appelons l’outil "opencv_traincascade", qui permet d’entraîner une cascade.
<br/>
<br/>
Cet outil prend en argument le dossier qui contiendra le classificateur entraîné (-data), notre fichier descripteur postives.vec (-vec), la liste des images négatives (-bg).
<br/>
<br/>
Nous indiquons également le nombre d’images positives (-numPos) et le nombre d’images négatives (-numNeg) qui seront utilisés à chaque itération. Le nombre d’images à utiliser doit être inférieur au nombre d’images disponible. Enfin, nous indiquons le nombre de itérations à entraîner (-numStages) ainsi que la taille, en pixel des échantillons (-w -h).
<br/>
<br/>
On écrit alors la commande de la manière suivante dans le terminal : "opencv_traincascade -data data -vec positives.vec -bg negatives.txt -numPos 550 -numNeg 600-numStages 20 -w 24 -h 24"
<br/>
<br/>
Entraîner un classificateur prend un certain temps : 33 minutes dans mon cas et dépend fortement du nombre d’itérations que vous avez spécifié. Pour chaque itération, un fichier stageXX.xml est créé. Cela permet de pouvoir arrêter l’apprentissage et reprendre de le reprendre à la dernière itération connue. Lorsque l’entraînement se termine un fichier cascade.xml est créé et on obtient cet affichage dans le terminal : 
<br/>
<br/>
<img src="Entrainer_classificateur.png" width=400 align=left>

<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>






* Test du classificateur

Maintenant que l'on possède notre classificateur, il nous reste plus qu'à le tester avec le script suivant : 


<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>

### Source 

* https://fr.wikipedia.org/wiki/Motif_binaire_local
