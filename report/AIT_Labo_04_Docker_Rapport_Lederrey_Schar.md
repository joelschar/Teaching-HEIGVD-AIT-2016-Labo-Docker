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

### <a name="introduction"></a> Introduction

### <a name="task-0"></a> Task 0: Identify issues and install the tools

- **[M1]** :  Dans la solution implémentée au chapitre précédent nous devions faire toutes les manipulations de configuration et de reload du serveur proxy à la main. Cela implique donc une maintenance lourde et un interruption du service ce qui n'est pas souhaitable dans un environnement de production.

- **[M2]** :  Pour ajouter un nouveau noeud `webapp` disponible pour le  il faudra :

1. Ajouter le serveur comme nouveau noeud dans le fichier de configuration du ha proxy `ha/config/haproxy.cfg` 
   ![1546763589456](/home/joel/Switchdrive/HEIG/S-5/AIT/Labos/labo-04-Docker/report/img/1546710081192.png)
2. Configurer le script pour que la valeur `<s3>` soit remplacée par l'adresse ip du container dans le fichier de configuration du proxy : `ha/scripts/run.sh`
   ![1546763631016](/home/joel/Switchdrive/HEIG/S-5/AIT/Labos/labo-04-Docker/report/img/1546763631016.png)
3. Modifier le script `provision.sh` de vagrant pour que le container S3 soit créé sur la base de l'image webapp:
   ![1546710384875](/home/joel/Switchdrive/HEIG/S-5/AIT/Labos/labo-04-Docker/report/img/1546710384875.png)
4. Linker le nouveau container au server proxy : 
   ![1546710472863](/home/joel/Switchdrive/HEIG/S-5/AIT/Labos/labo-04-Docker/report/img/1546710472863.png)
5. Démarrer vagrant pour créer les containers ou lancer le script de reprovisonning.

- **[M3]** : Dans cette solution il est nécessaire d'ajouter manuellement les noeuds dans la configuration de HAproxy et relancer manuellement le serveur. 
  Pour rendre cela plus dynamique il faudrait qu'un scripte s'occupe de créer le nouveau container, ajoute celui-ci dans les fichiers de configuration et relance automatiquement le script de provisionning.
- **[M4]** : HaProxy offre une `Runtime API` qui permet de faire l'ajout ou la suppression de noeud dynamiquement. Pour cela on va créer des templates de serveurs qui pourront être attribué aux nouveau containers.
  Voir https://www.haproxy.com/blog/dynamic-scaling-for-microservices-with-runtime-api/ pour plus de détails concernant la solution.
- **[M5]** : En consultant les scripts `run.sh` du HaProxy et de la WebApp, on remarque que le script du proxy démarre `rsyslogd` et copie un fichier de configuration `rsyslogd.cfg` contrairement aux webapps. Il serait possible de configurer également les containers WebApp pour qu'ils utilisent `rsyslog` et configurer celui-ci pour que les logs soient redirigés vers un serveur centralisé.
  Il serait également possible d'utiliser `syslog logging driver` de docker qui permet de récupérer les logs des containers directement et de les centralisés.
- **[M6]** : Quand on ajouter un serveur au système il faut ajouter celui-ci dans le fichier comme nouveau noeud et relancer le script pour que les adresses soient remplacées. Pour rendre cela plus dynamique il serait possible d'utiliser le système de `Runtime API` implémenté par HAProxy.

On illustre ici par la même occasion que l'ajout d'un troisième noeud à bien fonctionné.

1. ![1546765641526](/home/joel/Switchdrive/HEIG/S-5/AIT/Labos/labo-04-Docker/report/img/1546765641526.png)
2. URL vers le repo : https://github.com/joelschar/Teaching-HEIGVD-AIT-2016-Labo-Docker

### <a name="task-1"></a> Task 1: Add a process supervisor to run several processes

1. ![1546769814453](/home/joel/Switchdrive/HEIG/S-5/AIT/Labos/labo-04-Docker/report/img/1546769814453.png)
2. Comme expliqué dans cet article ( https://docs.docker.com/config/containers/multi-service_container/ ), docker est consu pour ne faire tourner qu'un seul processus principale. Il est possible de faire des sous processus, mais ce sera le processus principale qui régit l'instance du container. 
   Pour gérer plus facilement de multiple processus il est conseillé de démarrer un petit processus "init" qui sera le processus principale et les autres processus tournerons comme processus secondaires dans le container. Si un des processus cesse de fonctionner, cela n'influence pas l’existence du container.
   Dans notre cas nous allons utiliser S6 comme gestionnaire de process. On aura donc le processus init qui tourne en tant que processus principale et S6 qui gère le fonctionnement des processus applications qui tourne dans le container. Cela nous offre une couche d’administration supplémentaire.

### <a name="task-2"></a> Task 2: Add a tool to manage membership in the web server cluster



### <a name="task-3"></a> Task 3: React to membership changes

### <a name="task-4"></a> Task 4: Use a template engine to easily generate configuration files

### <a name="task-5"></a> Task 5: Generate a new load balancer configuration when membership changes

### <a name="task-6"></a> Task 6: Make the load balancer automatically reload the new configuration

### <a name="difficulties"></a> Difficultés

### <a name="conclusion"></a> Conclusion

### 