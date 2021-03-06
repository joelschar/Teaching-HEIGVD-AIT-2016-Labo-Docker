# Labo 04, Docker

Yann Lederrey et Joel Schär

URL vers le repo : https://github.com/joelschar/Teaching-HEIGVD-AIT-2016-Labo-Docker



[Introduction](#introduction)

0. [Identify issues and install the tools](#task-0)
1. [Add a process supervisor to run several processes](#task-1)
2. [Add a tool to manage membership in the web server cluster](#task-2)
3. [React to membership changes](#task-3)
4. [Use a template engine to easily generate configuration files](#task-4)
5. [Generate a new load balancer configuration when membership changes](#task-5)
6. [Make the load balancer automatically reload the new configuration](#task-6)

[Difficultés](#difficulties)

[Conclusion](#conclusion)

## <a name="introduction"></a> Introduction

Ce laboratoire à pour but d’approfondir notre expérience avec Docker et de voir quel sont les possibilités qu'offre HAProxy comme solution viable dans un environnement de production.
Nous allons mettre en pratique des connaissance théorique de haute disponibilité que nous avons vu en cours sur la base d'une infrastructure de container docker et de HAProxy.

## <a name="task-0"></a> Task 0: Identify issues and install the tools

- **[M1]** :  Dans la solution implémentée au chapitre précédent nous devions faire toutes les manipulations de configuration et de reload du serveur proxy à la main. Cela implique donc une maintenance lourde et un interruption du service ce qui n'est pas souhaitable dans un environnement de production.

- **[M2]** :  Pour ajouter un nouveau nœud `webapp` disponible pour le serveur proxy  il faudra :

1. Ajouter le serveur comme nouveau nœud dans le fichier de configuration du ha proxy `ha/config/haproxy.cfg` 
   ![1546763589456](img/1546710081192.png)
2. Configurer le script pour que la valeur `<s3>` soit remplacée par l'adresse ip du container dans le fichier de configuration du proxy : `ha/scripts/run.sh`
   ![1546763631016](img/1546763631016.png)
3. Modifier le script `provision.sh` de vagrant pour que le container S3 soit créé sur la base de l'image webapp:
   ![1546710384875](img/1546710384875.png)
4. Linker le nouveau container au server proxy : 
   ![1546710472863](img/1546710472863.png)
5. Démarrer vagrant pour créer les containers ou lancer le script de reprovisonning.

- **[M3]** : Dans cette solution il est nécessaire d'ajouter manuellement les noeuds dans la configuration de HAproxy et relancer manuellement le serveur. 
  Pour rendre cela plus dynamique il faudrait qu'un scripte s'occupe de créer le nouveau container, ajoute celui-ci dans les fichiers de configuration et relance automatiquement le script de provisionning.
- **[M4]** : HaProxy offre une `Runtime API` qui permet de faire l'ajout ou la suppression de noeud dynamiquement. Pour cela on va créer des templates de serveurs qui pourront être attribué aux nouveau containers.
  Voir https://www.haproxy.com/blog/dynamic-scaling-for-microservices-with-runtime-api/ pour plus de détails concernant la solution.
- **[M5]** : En consultant les scripts `run.sh` du HaProxy et de la WebApp, on remarque que le script du proxy démarre `rsyslogd` et copie un fichier de configuration `rsyslogd.cfg` contrairement aux webapps. Il serait possible de configurer également les containers WebApp pour qu'ils utilisent `rsyslog` et configurer celui-ci pour que les logs soient redirigés vers un serveur centralisé.
  Il serait également possible d'utiliser `syslog logging driver` de docker qui permet de récupérer les logs des containers directement et de les centralisés.
- **[M6]** : Quand on ajouter un serveur au système il faut ajouter celui-ci dans le fichier comme nouveau noeud et relancer le script pour que les adresses soient remplacées. Pour rendre cela plus dynamique il serait possible d'utiliser le système de `Runtime API` implémenté par HAProxy.

On illustre ici par la même occasion que l'ajout d'un troisième noeud à bien fonctionné.

1. ![1546765641526](img/1546765641526.png)
2. URL vers le repo : https://github.com/joelschar/Teaching-HEIGVD-AIT-2016-Labo-Docker

## <a name="task-1"></a> Task 1: Add a process supervisor to run several processes

1. ![1546769814453](img/1546769814453.png)

2. Aucune difficulté n'a été rencontrée dans cette étape. Tout c'est déroulé pour le mieux et nous avons une structure qui tient la route en fin de tache.

   Au niveau du fonctionnent, comme expliqué dans cet article ( https://docs.docker.com/config/containers/multi-service_container/ ), docker est conçu pour ne faire tourner qu'un seul processus principale. Il est possible d'avoir des sous processus, mais c'est le processus principale qui régit l'instance du container. 
   Pour gérer plus facilement de multiple processus il est conseillé de démarrer un petit processus "init" qui sera le processus principale et les autres processus tournerons comme processus secondaires dans le container. Si un des processus cesse de fonctionner, cela n'influence pas l’existence du container.
   Dans notre cas nous allons ajouter S6 comme gestionnaire de process. On aura donc le processus init qui tourne en tant que processus principale et S6 qui gère le fonctionnement des processus applications qui tourne dans le container et nous permet ainsi de gérer plusieurs processus qui tournent en parallèle.


## <a name="task-2"></a> Task 2: Add a tool to manage membership in the web server cluster

1. voir `logs/task2`

2. On crée ici un cluster qui tourne autour de "ha" en indiquant de joindre ce nœud précis. En effet cela assure que tous les membre du cluster dépendent du même nœud, mais cela implique aussi que celui-ci soit déjà créé au moment de l'arrivée des autres nœuds. Ce que l'on voudrait c'est que tous les nœuds puissent joindre le cluster indépendamment les uns des autres.

3. `Serf` fonctionne selon une communication bidirectionnelle entre les noeuds d'un même cluster. Les noeuds vont donc rejoindre un même cluster et vont s'informer régulièrement de leur état et de l'état des autres éléments du cluster. De nouvelle machines peuvent rejoindre ou quitter le cluster.

   Il exsite d'autre solution qui permettent de résoudre cette problématique comme par exemple `consul` qui offre un certain nombre de fonctionnalité haut niveau supplémentaire qui ne sont pas implémentées par `serf` et fonctionne de manière centralisée. Cela peut avoir des avantages et des inconvénient, mais en l’occurrence, cela permet de gérer les services plus finement.
   Il en existe d'autre comme Zookeeper, Eureka ou etcd.


## <a name="task-3"></a> Task 3: React to membership changes

1. voir `logs/task3`

2. idem


### <a name="task-4"></a> Task 4: Use a template engine to easily generate configuration files

1. Il serait possible d'installer l'application directement depuis le container et ensuite de faire un docker commit pour appliquer la modification sur l'image docker. De cette manière il n'est pas nécessaire de rebuild l'image, mais la modification est tout de même persistée.

   Dans le Dockerfile, chaque commande représente une nouvelle couche dans l'image docker. Créer une nouvelle couche signifie une image plus volumineuse. Combiner plusieurs commandes permet donc de rassembler les éléments sur une même coucher. Cela permet de réduire le volume de l'image.

   Pour réduire encore la taille d'une il image il est possible avec des outils comme `docker-squash` de réduire la taille d'une image en écrasant toutes les couches pour en garder qu'une seule. Cela réduit radicalement l'espace utilisé par l'image, mais le procédé n'est pas compatible avec toutes les commandes comme (PORT, ENV, ..).

2. Pour réduire le nombre de couche il serait possible par exemple de regrouper les fichiers copiés dans un seul répertoire et de le copier en une seule fois ou bien d'organiser l'ordre des couches pour pouvoir réutiliser une partie dans une autre image qui reprend une partie des éléments. Pour faire cela on va créer une image de référence et on va baser les images plus spécifique sur cette dernière en l'indiquant sur la première ligne avec la commande `FROM`.

3. voir `logs/task4`

4. Les fichiers générés sont sous la forme de donnée JSON et on remarque que beaucoup de valeurs sont null. Il serait donc judicieux de faire une sélection des données qui nous intéressent et d'ignorer le reste. Pour faire cela Docker met à disposition un paramètre `--format`à la commande `inspect`qui permet de choisir les données que l'on veut extraire. (https://docs.docker.com/engine/reference/commandline/inspect/)

### <a name="task-5"></a> Task 5: Generate a new load balancer configuration when membership changes

1. voir `logs/task5/haproxy.cfg_1`
   voir `logs/task5/haproxy.cfg_2`
   voir `logs/task5/haproxy.cfg_3`

   voir `logs/task5/docker_ps`

   voir `logs/task5/inspect_ha`
   voir `logs/task5/inspect_s1` 
   voir `logs/task5/inspect_s2`

2. voir `logs/task5/nodes`

3. voir `logs/task5/nodes_after_s1_stopped`
   voir `logs/task5/haproxy.cfg_after_s1_stopped`
   voir `logs/task5/docker_ps_after_s1_stopped`

4. a

### <a name="task-6"></a> Task 6: Make the load balancer automatically reload the new configuration

1. avec 2 noeuds :
   ![1546811247072](img/1546811247072.png)

   On peut ensuite démarrer des neuds avec la commande `docker run -d --network heig --name s1 softengheigvd/webapp`

   avec 3 noeuds :
   ![1546811422620](img/1546811422620.png)


   avec 5 neuds :

   ![1546811483899](img/1546811483899.png)


   En stoppant le neud 2 et 3:

   ![1546811586061](img/1546811586061.png)

2. La solution finale est très réactive et fonctionne très bien. On voit bien que les noeuds sont ajouter au système et qu'ils sont fonctionnels.
   Une lacune qui résides dans cette solution est qu'elle ne permet l'ajout dynamique de noeud backend uniquement. Pour amélorier cette solution il serait judicieux d'appliquer la même structure pour le HaProxy. 
   Il serait également intéressant d’analyser la question du `zero downtime` pour offrir à l'utilisateur la meilleure expérience possible.

3. Pour une live démo n'hésitez pas à prendre contact avec nous. Nous serons ravis de vous faire une démonstration.

### <a name="difficulties"></a> Difficultés

Le laboratoire étant très claire et bien détaillé avec une marche à suivre précise il nous à été très facile de suivre et de comprendre chaque étape. Si quelque chose ne fonctionnait pas du premier coup c'était plutôt du à une petite erreur de syntaxe ou une erreur dans le suivit des étapes.

### <a name="conclusion"></a> Conclusion

Ce laboratoire nous à permis selon un file conducteur très bien conduit de découvrir les possibilités de haute disponibilité qu'il est possible d'obtenir avec le serveur proxy HAProxy dans une infrastructure Docker. Nous avons vu comment automatiser toutes les étapes et rendre le système réactifs à l'ajout ou la suppression de noeuds dans le système.