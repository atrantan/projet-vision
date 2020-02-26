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

Les images positives sont des images qui contiennent le visage à reconnaître. Il en faut aussi un très grand nombre mais il faut aussi connaître les coordonnées de la postition du visage dans l'image ainsi que le nombre d'occurence du visage. On ne peut pas réaliser cela à la main puisque cela serait trop long. Par chance, OpenCV propose un outil, opencv_createsample, permettant de créer des images positives à partir d’une seule image d’entrée. Nous avons donc besoin d’une image contenant uniquement notre visage à reconnaître et de créer un nouveau dossier "positives" qui contiendra les images positives. Afin d'utiliser opencv_createsample on écrit la commande suivante dans le terminal "opencv_createsamples -img stop.png -bg negatives.txt -info positives/positives.lst -pngoutput -num 600". Cette commande nous permet de générer 600 images positives à partir des images négatives et un fichier "positives.lst" qui contiennent les informations relatives aux images positives.


































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
