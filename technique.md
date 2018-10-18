# Défi Prévisecours par \<EIG2018 x SDIS91>

Le défi initial s'est déroulé du 15 janvier au 14 novembre 2018 au sein du Ministère de l'Intérieur en coopération avec le SDIS 91 et Etalab dans le cadre de la 2e promotion des Entrepreneurs d'Intérêt Général.

But: aider les sapeurs pompiers à mieux prévoir les nombre de solicitations opérationnelles afin de pouvoir mieux ajuster leurs ressources internes.

Date d'écriture initiale: octobre 2018.

Cette documentation a plus une vocation d'entrée en matière pour le passage de témoin vers la future équipe technique reprenant le projet. Les points de pérénnisation et de road maps sont non exhaustifs.

## Equipes

- Entrepreneurs d'Intérêt Général / EIG
	- Guillaume Lancrenon <guillaume.lancrenon@interieur.gouv.fr>
	- Tiphaine Phe-Neau <tiphaine.phe-neau@interieur.gouv.fr>
- Sapeurs Pompiers de l'Essonne / SDIS91
	- Fabrice Baret 
	- Muriel Flottès
- MGMSIC @ Ministère de l'Intérieur / MI
	- Fabien Antoine 
	- Daniel Ansellem

	
## Schéma technique

Le défi Prévisecours se divise en trois parties principales:

1. Aggrégation de données (open data et données partenaires)
	- Référents EIG: Tiphaine Phe-Neau et Guillaume Lancrenon
2. Modèles de prédiction des solicitations opérationnnelles
	- Référents EIG: Tiphaine Phe-Neau 
3. Application cartographique de déliv
	- Référents EIG: Guillaume Lancrenon


Comme le projet partait de 0, nous avons du travailler à l'aggrégation des données (1) afin d'avoir de quoi travailler et nourrir nos modèles de prédictions (2). En parallèle de la partie (2), nous avons conçu une application cartographique permettant de restituer visuellement les prédictions aux Sapeurs Pompiers de l'Essonne. Tous ces travaux ont été menés conjointement par l'équipe EIG et le SDIS 91. 

	
## Architecture technique

- Dessin de l'architecture technique avec détails des technos utilisées.

![Schéma technique](https://github.com/previsecours/previsecours-documentation/blob/master/img/Pr%C3%A9visecours%20-%20Architecture%20technique.png)

[Source: Google drawing pour modifications](https://docs.google.com/drawings/d/19m2pBj67aUdZEH8puVMv7UNx6OB7ofpAmTmhfFUWSe0/edit?usp=sharing)



## 1. Aggrégation des données (opérations, opendata et données partenaires)

Nous avons commencé par analyser les rapports d'activité du SDIS91 pour mieux comprendre la nature de leurs interventions (solicitations opérationnelles), ensuite nous avons très rapidement pu récupérer les données des interventions de 2010 à 2017 et nous familiariser avec.

En ce qui concerne les données non opérationnnelles connexes: comme nous partions de rien (donc nous avions le choix de tout) et ne connaissions pas le coeur de métier, nous avions quelques a priori pour la récolte des données mais nous sommes allés voir le métier (les sapeurs pompiers en casernes pour écouter leurs intuitions). Au final, nous avons les données suivantes:

- Sources de données, type avec fréquence de mise à jour possible et portée géographique
	- [Données calendaires](https://algo.previsecours.fr/projects/DATAPREPCALENDRIER/flow/) (jours féries, fêtes religieuses, vacances, soldes, phases de la lune…) - portée nationale via génération et fichier plat en one shot;
	- [Données de pollution de l’air](https://algo.previsecours.fr/projects/DATAPREPOPENDATAPOLLUTION/) (NO2, O3, PM10) - portée départementale via fichier plat mensuel;
	- [Données allergènes type pollen](https://algo.previsecours.fr/projects/DATAPREPOPENDATAPOLLEN/) (graminées, cumul de taxons, etc.) - portée portée départementale via fichier plat mensuel;
	- [Données météo](https://algo.previsecours.fr/projects/DATAPREPCALENDRIER/flow/) (humidité, précipitation, pression atmosphérique, température, vent, durée moyenne du jour) - portée nationale via API;
	- [Données épidémiques](https://algo.previsecours.fr/projects/OPENDATARSEAUSENTINELLE/flow/) (diarrhée, grippe, varicelle) - portée nationale via fichier plat hebdomadaire;
	- [Données sur les incivilités](https://algo.previsecours.fr/projects/DATASSMSI/) (certains types d’infractions) - portée départementale via fichier plat en one shot;
	- Données sur la population ([1](https://algo.previsecours.fr/projects/DATAPREPOPENDATAMENAGE/), [2](https://algo.previsecours.fr/projects/DATAPREPOPENDATAFORMATION/), [3](https://algo.previsecours.fr/projects/DATAPREPOPENDATACAF/), [4](https://algo.previsecours.fr/projects/DATAPREPOPENDATAIDH2/)) (tranches d’âge, constitution des foyers, scolarisation, diplôme, profession, idh2 – indice de développement humain, couverture RSA, etc.) - portée nationale pour (1, 2, 3) annuels et départementale pour (4) via fichier plat one shot;
	- [Données sur la commune](https://algo.previsecours.fr/projects/DATAPREPOPENDATAFINESS/) (présence d’établissements sociaux, de soins, etc.) - portée nationale via fichier plat annuels ou pluri-annuels;
	- [Données sur le transport](https://algo.previsecours.fr/projects/DATAPREPOPENDATATRANSPORTS/) (gares, trafic cumulé, terminus de ligne) - portée départementale via fichier plat pluri-annuel.

Chaque source de données est un projet DSS spécifique à part et les facteurs de chaque source de données sont aggrégés dans une meta table dans le projet [Featuring](https://algo.previsecours.fr/projects/PRVISECOURSFEATURING/).

Il y a un projet DSS de monitoring de l'état des bases et des mises à jour: [ici](https://algo.previsecours.fr/projects/DATAPREPOPENDATASUPERVISION/datasets/supervision_opendata/explore/).

### "Open-Open data" alias `o2-data` @ Projet Featuring

L'une des choses les plus intéressantes de cette partie 1 est le travail de consolidation de toute la donnée aggrégées en 2 metatables: une première concernant les descripteurs des communes relativement statique à l'année (ex: pyramide des âges), une deuxième décrivant des facteurs concernant les communes mais variant beaucoup plus régulièrement (ex: météo).

Il est à noter que ce travail a été chronophage (au moins 4 mois sur les 10 du défi EIG) et demande un investissement pour le maintien à jour de ces données open data. Cependant, la majorité des gens n'ayant pas mis les mains dedans pensent que c'est trivial: NON, ça ne l'est pas (preuve: il y des business models entier basés dessus). 

Cette table est capitalisable à hauteur du MI voir même de l'Administration de façon générale. Actuellement ce sont deux tables qui peuvent être jointes à d'autres données au besoin (comme c'est le cas avec les données d'interventions du SDIS 91). Avec un léger travail par dessus (1 semaine grand max pour quelqu'un de relativement compétent), on peut construire une API par dessus pour réquêter les données à la volée et n'obtenir que les données qui nous intéressent.

- Dictionnaires des données (disponible en annexe, il y a actuellement 195 facteurs)
	- o2-data communes
	- o2-data variability

### Pérénnisation

Pour péréniser cette partie du projet (qui peut et doit idéalement être sorti du scope de Prévisecours pour remonter à un niveau supérieur), nous voyons plusieurs possibilités:

-  Une équipe de reprise avec 1/5e ETP data engineer pour le maintien et idéalement l'extension donc potentiellement un autre profil data hunter.
-  Achat des données préformatées auprès de broker de données ou emploi d'un data hunter
-  Ouverture en open source pour réutilisation et amélioration par une communauté
-  Idéalement, cette partie serait détachée de Prévisecours et hostée par une instance de type Etalab pour toute l'Administration voir une instance commune à tout le MI avec plein de données supplémentaires et Prévisecours taperait dedans en fonction de ses besoins.

### Road map

- Etendre à de nouvelles sources déjà disponibles ou par data hunter
- Automatiser la mise à jour des sources existantes

## 2. IA: modèles de prédictions

A partir des features récoltées et retravaillées, nous avons produit des modèles de prédiction pour les cas suivants:

- Type de sinistre
	- Secours à personnes (SUAP)
	- Incendies urbains
	- Incendies en milieux naturels	
	- Accidents
- Maille et portée temporelle
	- Semaine sur l'année calendaire
- Maille géographique
	- Commune
	- Zone de couverture (zdc)

Chaque modèle de prédiction se trouve dans un projet DSS à part par type de sinistre (un projet pour [SUAP / semaine](https://algo.previsecours.fr/projects/SUAPSEMAINEV2/flow/) va contenir les prédictions par [communes](https://algo.previsecours.fr/projects/SUAPSEMAINEV2/datasets/sdis91_com_suap_s/explore/) et [zdc](https://algo.previsecours.fr/projects/SUAPSEMAINEV2/datasets/sdis91_zdc_suap_s/explore/)). 

Ces modèles utilisent les données provenant du projet Featuring à la bonne maille temporelle (2 tables) et les données provenant du projet solicitations opérationnelles. Les opérations sont globalement self-contained dans les scripts python/sql ou les recettes DSS. Il faudra simplement vérifier que les modèles ne produisent pas d'erreurs mais il y a des checks automatiques d'erreurs sur les tables de predictions qui ont été mis en place dans l'automatisation des calculs.

Globalement, la meilleure façon de rentrer dans un projet et de suivre les [scripts d'automatisation](https://algo.previsecours.fr/projects/SUAPSEMAINEV2/scenarios/SUAP___Semaine___Run__/steps) mis en place (scénarios). Ils permettent de voir quel est l'ordonnancement des étapes principales et leur suite logique.

Nous avons autant que possible tenté de mettre en place des dashboards de performances sous Tableau et de mettre en place un système d'alerting en cas de dérive des modèles.

Les modèles ne sont pas encore re-entrainés automatiquement car nous n'avions pas à l'heure où j'écris ces lignes une livraison automatique et régulière des données d'intervention du SDIS91.


### Pérénnisation

-  L'un des points les plus importants est de mettre en place un point de retour régulier avec les SDIS et particulièrement le SDIS91 qui est pilote sur le projet. Ces points sont utiles pour comprendre les dérives: est ce la première occurence d'un phénomène exceptionnel ou y a t il une réelle dérive des modèles de prédiction?
-  Travail sur une optimisation des modèles avec les nouveaux évènements.
-  Travail de feature engineering additionel pour de nouveaux facteurs en entrée.

### Road map

-  Travail à la mise en place des projets pour les nouveaux SDIS: récolte des intuitions pour récupération open data et accompagnement métier.
- Modèles de prédictions à la maille temporelle différentes: jour. 

## 3. Application cartographique

TBD pour Maitre Guillim.

### Pérénnisation

-  Formation à l'utilisation de l'outil Prévisecours.


### Road map
- Point 1