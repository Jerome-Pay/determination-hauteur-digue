# determination-hauteur-digue
Projet de Master 2 - Détermination de la hauteur d'une digue pour une centrale nucléaire

## Introduction

Les centrales nucléaires jouent un rôle essentiel dans la production d’électricité en fournissant
une énergie stable et à faible émission de CO<sub>2</sub>, c’est la première source de production et
d’électricité en France [[1]](https://www.edf.fr/groupe-edf/comprendre/production/nucleaire/nucleaire-en-chiffres).  

Il est nécessaire d’installer les centrales nucléaires à proximité de
points d’eau douce car une grande quantité d’eau est nécessaire pour le refroidissement.  
Notre étude portera sur un site industriel, en l’occurence une centrale nucléaire, situé en bordure
d’une rivière avec une digue de protection dont la hauteur optimale est à déterminer.  
La
proximité avec la rivière peut provoquer des inondations du site si la cote atteinte pendant un
épisode de crue dépasse la hauteur de la digue de protection. Les dommages matériels sur le
site peuvent entraîner des coûts très lourds ou un ralentissement de la production voire un arrêt
temporaire.  

Par exemple, en 1999, la centrale nucléaire du Blayais a subi une inondation forte à cause d’une
grande marée et des vents violents. Les digues de protection, initialement construites à une
hauteur de 5,2 mètres au-dessus du niveau général de la France (NGF) n’étaient pas suffisantes.  
Pendant l’événement, le niveau de l’eau a atteint jusqu’à 5,3 mètres NGF, dépassant ainsi la
hauteur des digues et entraînant l’inondation partielle de la centrale. Des équipements et
matériels électriques ont été abîmés nécessitant des réparations et des remplacements. Pendant
quelques jours, 2 réacteurs ont été placé dans l’état "arrêt normal des générateurs de vapeurs"
(AN/GV) ralentissant fortement la production [[2]](https://www.irsn.fr/actualites/linondation-centrale-blayais-decembre-1999)  

Ce projet a pour objectif de déterminer la hauteur de la digue idéale pour minimiser les risques
d’inondations avec ou sans contraintes économiques.  

Nous avons à disposition des plans décrivant la morphologie de la rivière au niveau de notre
site industriel.  

![fiabilite_plan1_site](https://github.com/user-attachments/assets/5b23d58c-b968-41b0-8b2d-cf4246fb6de8)
![fiabilite_plan2_site](https://github.com/user-attachments/assets/8d3cd856-289d-4b77-a28c-d7f87ee779d2)

Avec :  

• $Zd$ et $Zc$ désignent respectivement la cote de la digue et la cote maximale de la crue (en
m NGF)  
• $H$ désigne la hauteur d’eau maximal (en m)  
• $Q$ désigne le débit maximal du cours d’eau (en m3/s)  
• $L$ désigne la longueur du tronçon de cours d’eau étudié (en m)  
• $B$ désigne la largeur du cours d’eau (en m)  
• $Zm$ et $Zv$ désignent la cote du fond du cours d’eau (en m NGF) respectivement en amont
et en aval du tronçon  
• $hd$ désigne la hauteur de la digue (en m)  
• $Zb$ désigne la cote de la berge (en m)  
Rechercher une hauteur de digue adéquate correspond à déterminer un risque de surverse
acceptable pouvant comprendre des considérations financières. Pour le risque de surverse seul
avec une digue, on se fixera un seuil qui n’est dépassé qu’avec probabilité 10<sup>−4</sup>
[[2]](https://www.lufi.uni-hannover.de/fileadmin/lufi/publications/63_littoral_2002_279.pdf).
Dans la suite nous utiliserons l’équation suivante pour modéliser la hauteur de surverse :
$S = Zv + H − Zb − hd$

## Méthodes mises en oeuvre

### Première méthode

La première méthode consiste à utiliser les données historiques.  
Nous disposons de données sur 149 années, de 1849 à 1997. Ce sont les débits maxima annuels
de crue en (m3/s) associés à leur hauteur d’eau (en m). Il manque 26 valeurs de hauteur d’eau
dans notre jeu de données.  

Nous procédons alors à une estimation de la loi de H. On teste l’adéquation des données à une loi de Gumbel, de Fréchet, de Weibull, log-Normal et de
Pearson III. Des lois des extrêmes.
Les lois de Pearson et de Weibull sont les plus adéquates, nous estimons les paramètres par maximum de vraisemblance.
On prendra un quantile à 0.9999 pour ne prendre que très peu de risque.  
Pour prendre en compte l'incertitude liée à $Zv$ nous utiliserons des techniques de Boostrap pour avoir des intervalles de confiances du quantile.  

Une autre idée est de modéliser la hauteur de la digue avec une régression sur les données de débit.

Nous voulons estimer H à partir des données historiques mais nous ne connaissons pas la loi
de cette variable. Cependant, nous disposons de la loi du débit Q(Gumbell) et nous
supposons qu’il existe une forte relation entre le débit et la hauteur d’eau. Nous avons pensé
à une régression linéaire avec une transformation de la variable débit. Une transformation très
pertinente est de lui appliquer la fonction racine carré. Nous avons ainsi une corrélation linéaire
de 0.98 entre la hauteur et la racine carré du débit d’eau. Ainsi :
$H = \beta_{0} + \beta{1}\sqrt{Q} + \epsilon$
Le modèle de régression linéaire avec nos données historiques sans prendre en compte les valeurs
manquantes est très performant et le coefficient R2 vaut 0.96 donc 96% de la variance est
expliquée. Le modèle complet est plus pertinent que le modèle qui ne prend en compte que
l’intercept car nous rejetons l’hypothèse que β1 = 0 avec une p-value presque nulle
Nous prendrons des quantiles élevés.
Nos effectuerons également une estimation avec Monte-Carlo.

### Deuxème méthode

Dans cette approche, nous disposons d’une formule pour H, la "hauteur d’écoulement maximale
de la crue".

$H = \left( \frac{Q}{K_s \sqrt{\frac{Z_m - Z_v}{L}} B} \right)^{\frac{3}{5}}$


où Ks correspond au coefficient de Strickler (coefficient "moyen de frottement sur la rivière).
On rappelle :  

• $L$ désigne la longueur du tronçon de cours d’eau étudié (en m), déterministe égal à 5000m.  
• $B$ désigne la largeur du cours d’eau (en m), déterministe égal à 300m.  
• $Q$ désigne le débit maximal du cours d’eau (en m$^3$/s), de loi $Gumbel(1013,558)$  
• $Zm$ désigne la cote du fond du cours d’eau (en m NGF) en amont du tronçon, aléatoire
de loi $\mathcal{T}ri(54, 56, 55)$.  
• $Zv$ désigne la cote du fond du cours d’eau (en m NGF) en aval du tronçon, aléatoire de
loi $\mathcal{T}ri(49, 51, 50)$.  
• $Ks$ aléatoire de loi $\mathcal{N}(30, 7.5^2)$.  

Pour approcher le quantile à 99,99%, on décide de simuler un échantillon de 10<sup>6</sup>
(toujours
selon la règle du 10<sup>k+2</sup> pour l’estimation d’un quantile à 10<sup>−k</sup>
) surverses à l’aide du modèle
hydraulique où $Zm$, $Zv$, $Ks$ et $Q$ sont aléatoires

### Troisième méthode

Dans cette partie, on considère des contraintes financières : on cherche à déterminer une hauteur
de digue dont le coût en espérance est le plus petit sur une période donnée de 30 ans.

On distingue deux types de coûts :  
• Les coûts d’investissements/construction et de maintenance.  
• Les coûts en cas d’inondation : dommages à la digue et au site (dégâts et perte de
production).  
On se ramènera à la fin à un coût annualisé calculé comme suit :  

$cout_final_annualise=\frac{(cout_investissement+cout_maintenance+cout_dommages)}{30}$

On reprend sensiblement la même manière de procéder que pour le modèle hydraulique seul, à
la différence qu’au lieu de calculer une unique hauteur d’eau à chaque itération, on en calcule
30. Ensuite, pour une grille de hauteurs de digue, on calcule les surverses associées pour les 30
ans. Enfin, on détermine les coûts annualisés pour chaque hauteur de digue.

## Résultats

On décide de ne conserver que les hauteurs déterminées à l’aide de techniques de bootstrap ou
de Monte-Carlo : ces méthodes nous ont semblé apporter la meilleure gestion du risque et en
particulier de la propagation de l’incertitude.  
Elles ont également l’avantage d’avoir pu être
appliquées à toutes nos approches permettant une comparaison plus facile entre les hauteurs
de digues retenues.

![fiabilite_tableau_recap](https://github.com/user-attachments/assets/77b393b4-72db-4419-90f2-8d66261df626)

Si le modèle hydraulique semblait plus sévère, l’ajout de la composante économique a ramené le
choix de la hauteur de la digue dans des hauteurs semblables à celles retenues avec l’approche
historique. Ainsi, on retient certes une hauteur de digue plus petite (et donc plus exposée au
risque) qu’avec le modèle hydraulique seul mais on a par ailleurs des arguments historiques qui
nous rassurent dans notre choix. La hauteur de digue obtenue avec la dernière approche est
donc très satisfaisante.
