# Projet de S4-TI - détection de visages

Dans le cadre de notre projet d'étude, nous allons réaliser un module de reconnaissance de visage pour systèmes embarqués.

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
<img src="Capture1.PNG" width=100 align=left>
<br/>
<br/>
<br/>
<br/>
<br/>
* Si le pixel au voisinage est plus grand ou égal que le pixel choisi alors le pixel avoisinnant prend la valeur binaire "1" sinon il vaut "0".
<br/>
<img src="Capture2.PNG" width=100 align=left>
<br/>
<br/>
<br/>
<br/>
<br/>
* Le premier pixel comparé correspond au LSB (bit 0) et le dernier au MSB (bit 7).
<br/>
<img src="Capture3.PNG" width=100 align=left>
<br/>
<br/>
<br/>
<br/>
<br/>
* Le pixel choisi se voit attribué la valeur décimale des bits avoisinnants.
<br/>
<img src="Capture4.PNG" width=300 align=left>
<br/>

### Mise en pratique

Nous allons maintenant transformer une image avec l'algorithme LBP. Pour ce faire nous allons utiliser une photo de Barack Obama disponible ci-dessous et un script python disponible sur GitHub.
     
<img src="Poster-sized_portrait_of_Barack_Obama.jpg" width=300 align=left/>
<br/>



Après traitement, on obtient le résultat suivant : 

<img src="obama_countours.jpg" width=300 align=left/>


## Entraînement du classifieur cascade avec une seule image du visage à détecter

Le classifieur cascade permet théoriquement de reconnaître des visages en entraînant ce classifieur de la manière suivante.

* Images négatives

Les images négatives sont des images qui ne contiennent le visage avec lequel on veut entrainer le classifieur. Il nous faut un grand nombre d'images négatives (environ 2000), pour en acquérir autant nous ne pouvons pas le faire à la main ce qui nous prendrait énormement de temps c'est pour cela qu'on va utliser un petit programme en langage Python.
Ce script a besoin pour fonctionner des modules "OpenCV" pour le traitement des images, "Numpy" pour les calculs numériques et "Urllib" pour gérer les URLs.
Le script est écrit de la manière suivante: 
<br/>

 <img src="scipt_images_négatives.PNG" width=1100 align=left> 
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
<img src="dossier_data.jpg" width=500 align=left/>
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
<img src="Entrainer_classificateur.png" width=500 align=left>

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
<br/>


* Test du classificateur 

Maintenant que l'on possède notre classificateur, il nous reste plus qu'à le tester avec le script suivant : 
<br/>
<img src="éxecution_classifier.png" width=800 align=left/>
<br/>
On utilise comme visage à détecter ce de Barack Obama avec l'image suivante :
<img src="image.jpg" width=400 align=left/>
<br/>
On obtient le résultat suivant :
<br/>
<img src="visage_image.jpg" width=400 align=left/>
<br/>
Comme on peut le voir le classifier a réussi à détecter le visage mais il y a eu 4 autres détections parisites.


<br/>

## Entraînement du classifieur cascade avec plusieurs images du visage à détecter

* Rassembler les images 

Afin de pouvoir entraîner un nouveau classificateur, nous avons besoin d’un grand nombre d’images positives et d’un nombre encore plus élevé d'images négatives.
<br/>
<br/>
Dans notre cas, on a 10 photos du visage à reconnaître et 200 images négatives qui ont été générées précèdemment.
<br/>
<br/>
Maintenant qu'on a ces images, il faut créer un fichier contenant la liste des images négatives avec la commande suivante : "find ./negatives/ -name '*.jpg' > negatives.txt"
<br/>
<br/>
<img src="images_obama.jpg" width=500 align=left/>
<br/>
Puis il faut aussi créer un fichier contenant la liste des images du visage à reconnaître avec la commande suivante : "find ./positives/ -name '*.jpg' > positives.txt" 
<br/>
<br/>

* Générer les images positives

Pour ce faire, nous pouvons appeler la fonction opencv_createsamples pour chaque image positive. Cela nous permet de générer de nouvelles images puis de générer les fichiers descripteurs. Cependant appeler cette fonction à la main sera très long.

Pour faciliter cette opération, nous pouvons appeler un nouveau script : "createsamples.pl". Ce script est disponible sur GitHub (https://github.com/mrnugget/opencv-haar-classifier-training). Cet outil permet, pour chaque image du visage à reconnaître, de générer automatiquement les images positives en appelant plusieurs fois l’outil opencv_createsamples. De plus les fichiers descripteurs sont également générés lors de l’appel de cette fonction.
<br/>
<br/>
On utilise la fonction de la manière suivante : perl createsamples.pl positives.txt negatives.txt samples 1000 "opencv_createsamples -maxxangle 1.1 -maxyangle 1.1 maxzangle 0 -maxidev 20 -w 20 -h 20"
<br/>
<br/>
Nous devons donc fournir lors de l’appel de l’outil un certain nombre d’informations. Tout d’abord, nous spécifions les listes des images positives et négatives. Puis, nous indiquons le chemin vers le dossier où seront stockés les fichiers descripteurs (samples). Enfin nous fxons le nombre d’images positives que nous voulons générer. Dans cet exemple, j’ai décidé de générer 1000 images, soit 100 nouvelles images par image initiale.

* Rassembler les fichiers descripteurs

L’appel du script createsamples.pl, nous permet de générer facilement les fichiers descripteurs pour chaque image positive initiale. L’ensemble de ces fichiers descripteurs ont été stockés dans le dossier samples.
<br/>
<br/>
<img src="samples.jpg" width=500 align=left/>
<br/>
Nous devons maintenant ces fichiers dans un seul fichier descripteur afin de pouvoir entrainer le classificateur. Pour ce faire, nous allons appeler le programme mergevec.py qui permet de regrouper tous les fichiers descripteurs. Ce programme est disponible sur GitHub (https://github.com/wulfebw/mergevec).
<br/>
<br/>
On éxécute le programme en tapant la commance suivante : "python3 mergevec.py -v samples -o out.vec"
<br/>
<br/>
Ce programme prend en option le dossier contenant les descripteurs (-v), et le fichier à générer (-o).

* Entrainer le classificateur

Maintenant que nous avons notre fichier descripteur, nous devons entraîner le classificateur.
<br/>
Mais avant cela, créons un dossier qui contiendra notre classificateur et les différents fichiers générés à chaque étape de l’entraînement. Il s'agit du dossier "data" 
<br/>
<img src="dossier_data.jpg" width=500 align=left/>

Maintenant, comme avec l'entraînement du classifieur cascade avec une image du visage à détecter nous allons utliser l'éxécutable opencv_traincascade de la manière suivante afin d'entrainer le classifier : 
<br/>
<br/>
"opencv_traincascade -data data -vec out.vec -bg negatives.txt -numPos 780 -numNeg 200 -numStages 20 -w 20 -h 20"
<br/>
<br/>
Ce commande prend du temps à s'éxécuter comme avec l'entraînement du classifieur cascade avec une image du visage à détecter.
<br/>
<br/>
Cet outil prend en argument le dossier qui contiendra le classificateur entraîné (-data), notre fichier descripteur postives.vec (-vec), la liste des images négatives (-bg).

Nous indiquons également le nombre d’images positives (-numPos) et le nombre d’images négatives (-numNeg) qui seront utilisés à chaque itération. Le nombre d’images à utiliser doit être inférieur au nombre d’images disponible. Enfin, nous indiquons le nombre de itérations à entraîner (-numStages) ainsi que la taille, en pixel des échantillons (-w -h).

* Test du classificateur

Une fois l'entrînement fini, il suffit d'utiliser le même script qui a permis l'entrainement du classifieur cascade avec une image du visage à détecter mais il faut seulement changer le fichier xml. 
<br/>
Voici le résulat avec une photo de Barack Obama
<br/>
<br/>
<img src="visage_obamaface.png" width=200 align=left/>
<br/>

Comme on peut le voir le classifier n'a pas réussi à détecter efficacement le visage mais il y a eu 3 autres détections parisites.
On en déduit les limites du classifier : tout d'abord la détection d'un visage est très peu précise et ensuite il n'arrive pas à détecter un visage en particulier.
## Nouvelle solution : Reconizer



