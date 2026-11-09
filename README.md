# tp-docker

## Exercice 1 — Comprendre le cycle de vie des processus dans un conteneur
1. Quel est le rôle du processus principal dans un conteneur ?
> Il est le processus parent de tous les sous-processus du conteneur, son arrêt implique l'arrêt

2. Quelle différence existe-t-il entre `docker stop` et `docker kill` ?
> `docker stop` équivaut à `docker kill --signal=TERM` (arrêt *doux*) et cas d'echec `docker kill --signal=KILL` (arrêt *brutal*)

3. Pourquoi un conteneur s'arrête-t-il lorsque son processus principal se termine ?
> car il est le coordianteur et la porte d'entrée vers les sous-processus du conteneur

4. Quelle différence existe-t-il entre `SIGTERM` et `SIGKILL` ?
> `SIGTERM` => arrêt *doux*, "*propre*"
>
> `SIGKILL` => arrêt *brutal*, risque de perte de donnée

5. Pourquoi est-il important qu'une application gère correctement les signaux d'arrêt ?
> cela permet de gérer correctement le comportement attendu de l'appli : maitriser l'arrêt en prenant en charge la sauvegarde d'un éventuel fichier, terminer de traiter une requête, etc...

---
## Exercice 2 — Comprendre les couches d'une image et le système de fichiers

1. Quelle différence existe-t-il entre les couches d'une image et la couche inscriptible d'un conteneur ?
> la couche d'une image est une couche `ReadOnly` qui représente la configuration initiale d'une image. la couche inscriptible d'un conteneur vient par dessus cette première pour en modifier le comportement du conteneur en run.

2. À quoi sert le mécanisme *copy-on-write* ?
> IL permet d'appliquer des mofication sur des élements de la couche image en les copiant tout d'abord puis les modifiant sur la couche inscriptible.

3. Pourquoi les modifications effectuées dans un conteneur ne modifient-elles pas directement l'image d'origine ?
> Toute modification se fait sur la couche inscriptible afin de permettre la reproductibilité.

4. À quoi sert la commande `docker diff` ?
> il permet de connaitre la difference entre la couche image et la couche inscriptible. à l'image de git diff, il précise les ajouts/modifications/suppressions apportés à la couche inscriptible à l'image.

5. Pourquoi les modifications apportées à la couche inscriptible sont-elles perdues lorsqu'on supprime le conteneur ?
> la couche inscriptible définit le conteneur: une image à laquelle on apporte des modifications via la couche inscriptible -> supprimer le conteneur revient à supprimer cette couche.
---

## Exercice 3 — Construire une image avec plusieurs étapes (Multi-stage build)
1. Pourquoi utiliser plusieurs instructions `FROM` ?
> pour changer d'environnement mi-parcours. ici, on passe d'un env de dev go on l'on compile le code à un env vide avec rien que le binaire executable, l'image finale ne garde que l'environnement scratch -> gain d'espace de stockage.

2. Quelle différence existe-t-il entre l'étape `builder` et l'étape finale ?
> dans builder, on compile le code go en binaire dans /app puis on ne garde que cet executable que l'on configure en execution automatique.

3. Pourquoi l'image finale ne contient-elle pas le compilateur Go ?
> `FROM scratch` ecrase l'ancien environnement contenant le compilateur

4. Quels sont les avantages d'une image finale minimale ?
> gain d'espace de stockage
>
> pas de contenu superflu
>
> pas d'info critique laissée
>
> pas d'alteration du conteneur

5. Quelles précautions faut-il prendre avant d'utiliser une image `scratch` en production ?
> que le binaire executé est lancé dès le démarrage du conteneur
>
> que le conteneur soit correctement configuré pour fonctionner sur l'OS de la machine hôte.
>
> qu'il n'y ait pas besoin d'interagir avec le conteneur après son démarrage.


---

## Exercice 4 — Renforcer la sécurité d'un conteneur
1. Pourquoi est-il préférable d'exécuter une application avec un utilisateur non privilégié ?
> car l'application, dans le cas d'une attaque, se verrait donné les clés du royaume (l'accès à la machine hôte)

2. Que sont les capacités Linux ?
> des droits d'admisnistration propre à l'admin (`root`) du système, comme changer le propiétaire d'un fichier, tuer des processus, etc...

3. Quelle différence existe-t-il entre `--read-only` et `--tmpfs` ?
> `--read-only` fige l'arborescence de fichier du conteneur.
>
> `--tmpfs` donne l'accès une arborescence de fichier en RAM

4. Pourquoi un conteneur peut-il avoir besoin de zones inscriptibles malgré un système de fichiers racine en lecture seule ?
> pour du déboguage (logs)
>
> pour gèrer des fichiers en entrée/sortie et les garder après supression du conteneur (volumes)

5. Ces options suffisent-elles à garantir une isolation parfaite ? Justifiez votre réponse.
> les conteneur utilise le même noyeau que la machine hôte -> si une faille au niveau de ce dernier est exploitée , il est possible de compromettre la machine hôte.


---

## Exercice 5 — Diagnostiquer un conteneur qui se termine anormalement
1. Quelle différence existe-t-il entre un conteneur arrêté et un conteneur supprimé ?
> un conteneur arrếté peut être remis en marche en reprenant sa couche insriptible par dessus la couche image via `docker start`.
>
> un conteneur supprimé implique la suppression de sa couche inscriptible et des données qui s'y trouvaient. Le relancer, c'est créer un nouveau conteneur depuis la même image (`docker run`).

2. Que signifie un code de sortie différent de zéro ?
> Cela signifie qu'une erreur est survenue (ou que l'on a configuré une sortie custom).

3. Quelle différence existe-t-il entre les sorties standard et d'erreur ?
> les sorties standards et d'erreur sont deux flux différents facilement identifiables dans le système. ce qui permet une analyse de l'exection plus efficace.

4. Pourquoi les journaux sont-ils utiles lorsqu'un conteneur se termine trop rapidement ?
> Ils permettent le déboguage en facilitant la recherche du point d'echec (le moment de l'execution où l'erreur survient)

5. Pourquoi faut-il examiner le code de sortie au lieu de se fier uniquement au message d'erreur ?
> car il se peut que le conteneur n'ait pas pu loggé l'erreur qui a induit son arrêt. et le code peut être customisé de sorte à reconnaitre rapidement le type d'erreur.


---

## Exercice 6 — Configurer la rotation des journaux

1. Pourquoi des journaux trop volumineux peuvent-ils poser problème ?
> 1. ils finissent par prendre un espace non négligeable.
> 2. il devient difficile d'y chercher une info précise.
> 3. les logs peuvent finir par surcharger la machine hôte.

2. À quoi sert un pilote de journaux Docker ?
> c'est le méchanisme qui gère les logs docker (taille max, rotation, libellé, etc...). Par défaut c'est `json-file logging driver` qui prend cette tâche.

3. Quelle différence existe-t-il entre la consultation des journaux avec `docker logs` et leur stockage sur le long terme ?
> Les journaux `docker logs` subissent une rotation impliquant leur disparition définitive au bout d'un certain temps. Ces logs disparaissent avec le conteneur suite à un `docker rm`.
>
> Le stockage sur le long terme revient à archiver des journaux afin qu'ils perdurent en dehors du cycle de vie des logs docker. les rendant consumtables après la supression du conteneur (pour un  audit par exemple)

4. Pourquoi la rotation des journaux est-elle importante sur un serveur ?
> elle permet de limiter taille max des logs d'un conteneur ("max-size"="1m" et max-file="2" => 2Mo)

5. Pourquoi faut-il choisir une stratégie de journalisation adaptée à l'environnement de production ?
> afin de pouvoir se retrouver rapidement et efficacement dans les logs:
>   se débarasser du superflu. (ex: des logs outdated qui ne sont plus pertinents)
>   maintenir une empreinte disque raisonnable.
---

## Exercice 7 — Explorer les processus et les modifications d'un conteneur en fonctionnement

1. Quelle différence existe-t-il entre `docker exec` et `docker run` ?
> `docker run` est utilisé pour créer un conteneur à partir d'une image.
>
> `docker exec` permet de lancer un nouveau processus sur un conteneur en cours d'execution.

2. À quoi sert `docker top` ?
> il permet de lister les processus en cours d'execution sur un conteneur.

3. Pourquoi `docker diff` peut-il être utile pendant un diagnostic ?
> les changements apportés à l'image initiale peuvent être un bon indicateur du déroulé d'execution du conteneur.

4. Quelles informations fournit `docker stats` ?
> elle fournit la consommation matériel des conteneurs (CPU, RAM, transferts des supports de stockages, nbr de processus enfants) à l'image du gestionnaire de tâche.

5. Pourquoi faut-il éviter de modifier manuellement un conteneur en production sans documenter les changements ?
> car le chagement n'est pas facilement reversible
>
> aucun log n'est enregistré pour ces changements
>
> ces changements disparaissent à la supression du conteneur (`docker rm`) => il vaut mieux les appliquer à un DockerFile


## Conclution
à travers ce tp, nous avons pu voir qu'avec une bonne configuration, il est parfaitement possible de créer des conteneurs:
    . dont on maitrise l'execution (debug, log et gestion des processus)
    . optimisés (de taille minimale)
    . qui n'affecte pas la machine hôte (isolation, sécurité et gestion mémoire)
    . facilement reproductible (DockerFile)