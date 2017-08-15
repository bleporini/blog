---
title: Et si Java était mort et qu'on ne le sache pas encore
layout: post
tags : [ Java ]
---
{% include JB/setup %}

Vous devez vous en douter, j'ai évidemment longuement hésité avant de publier ce billet, que dis-je, avant même de réfléchir à donner vie à ces pensées hérétiques!

Je vais commencer par la fin. J'étais dans une [conférence](https://www.youtube.com/watch?v=wfWYm0MYj_8) qui présentait ce que *pourrait* donner le pattern matching dans Java 10. Le concept tire un certain nombre de sujets comme les *Value Types*, les tuples (enfin!) et la destructuration des objets. Soit. A la fin de la session, l'orateur explique que soit c'est implanté en 2018, soit cela ne sera pas dans Java 10. **Ouch!** Ça ferait un peu long quand même: on a pas encore de certitudes sur la disponibilité de Java 9, ça sera peut-être pas embarqué dans Java 10, dont on a encore moins d'information mais peut-être dans Java 11. Si on rajoute les retards, le temps d'adoption des nouvelles versions ainsi que la prise en charge de la fonctionnalité dans les diverses bibliothèques courrament usitées, je me suis dit que les enfants de mes enfants verront peut-être le pattern matching dans une application Java à la fin de leur carrière. 

Hashtag ça bitche dur.

Et puis ensuite, j'ai naïvement fait l'analogie avec Scala d'abord, car c'est mon langage préféré, mais également et surtout à des nouveaux venus dans lesquels je ne me suis pas spécialement lancé comme Swift et Kotlin. Les trois intègrent le pattern matching. Ma première reflexion fut que c'était abuser, que ces trois langages et spécialement les deux derniers, n'avaient pas eu besoin de tout ce temps là. Evidemment la quasi absence d'héritage explique sûrement en grande partie cette différence. Mais pire, j'ai pensé que si on poussait le curseur à fond, cela revenait quasiment à dire qu'implanter le pattern matching en Java pourrait prendre autant de temps que redévelopper un langage! 

Là j'avais l'impression de divaguer et soit j'avais trop bu de bières, soit il était nécessaire d'aller en chercher une autre.  

Mais quand même, du coup je me suis intéressé à ce qui allait venir dans Java 9 à part les modules et j'ai regardé une [présentation dédiée à ce sujet](https://fr.slideshare.net/jmdoudoux/java-9-modulo-les-modules-devoxx-fr-2017). J'ai trouvé le bilan un peu maigre au niveau des fonctionnalités du langage et du JDK: une amélioration du `try-with-resource`, un peu plus d'inférence de type et quelques API usuelles étendues, je vous fais grâce d'API comme [StackWalker](http://download.java.net/java/jdk9/docs/api/java/lang/StackWalker.html) qu'on utilisera une fois par an. J'ai bien conscience que beaucoup de choses se passent dans la JVM mais pour un développeur cela rend la version beaucoup moins excitante que des évolutions dans le langage. Et puis les modules, ça va résoudre quel problème dans ma vie de dev? Ok on va pouvoir faire des applis que se lancent beaucoup plus vite blablablabla... Mais à la fin quand je vais lancer mon projet Spring et qu'il va charger la moitié des jar de Maven central au démarrage, j'ai l'impression que ça va me faire une belle jambe d'avoir réussi à éviter de monter les classes de Swing ou AWT par transitivité... D'autant que j'ai à peu près compris le principe des modules mais je ne suis pas persuadé d'en avoir encore mesuré l'impact sur l'existant.

Bref, j'étais lancé et j'ai commencé à troller tout seul dans ma tête sur Java 8. Bien sûr l'arrivée des lambda était super cool et cela allait changer la physionomie du code, mais plusieurs années plus tard, j'ai un peu la dent dure concernant l'API des `Streams`. Bien sûr, typer un pipeline de modifications d'un jeu de données a du sens. Mais quand même

```java
maListe
  .map(i -> i+1);

```         

C'est quand même moins relou que 

```java
maListe
  .stream()
  .map(i -> i+1)
  .collect(Collectors.toList());

```

L'API `Stream` a sa place, mais pouvoir mapper directement sans passer par les `Stream` aurait permis d'alléger le code. L'intention était légitime mais au final le passage obligatoire assombrit l'apport des lambdas dans Java 8 et il ne manque plus qu'une `StreamFactory` pour que le tableau soit complet! Ces lambdas qui ont mis tant de temps à arriver notamment à cause de la dépendance à Invoke Dynamic... C'est bizarre mais Scala, Clojure, Groovy et j'en passe s'en sont sortis sans. Je pourrais également déverser mon fiel sur le mariage hideux des lambdas et de nos chères exceptions vérifiées, des larmes de sang versées lorsque j'ai découvert les méthodes `Stream#reduce` pour coder un `fold`, mais j'aurais l'air de tirer sur une ambulance... 

J'évolue majoritairement dans l'informatique de gestion en "entreprise" et le rythme innovations dans Java ressemble à la gestion de projets  dans les DSI avec tous leurs processus délirants et dans lesquels on ne sait pas quantifier le temps autrement qu'en trimestres. Java avance de la même façon, dans des tunnels qui peuvent durer de deux à cinq années entre chaque version majeure, seulement une sur deux ou trois apportant vraiment des nouveautés marquantes, du bon gros cycle en V à l'heure du *continuous delivery*, des annonces de retard tardives comme dans les projets...   

Pendant ce temps là les services métiers cherchent depuis quelques années à faire réaliser leurs besoins *hors DSI* pour engager des itérations plus courtes et espérer un *time to market* plus rapide. Et pendant ce temps là, pour continuer sur l'analogie, Kotlin, par exemple, gagne du terrain en couvrant le manque de Java 8 sur Android (ok là Oracle n'y est pour rien) et devient officiellement supporté pour la nouvelle version de Spring...

Et si tout ça n'était qu'un ensemble de *smells* qui ne font que démontrer que le modèle actuel est en train de pérécliter et qu'il est urgent de trouver un moyen de livrer plus rapidement pour éviter de sombrer définitivement vers le statut de *legacy*? Le démarrage raté de Java 7 suite à l'incompatibilité avec Lucene a sûrement laissé des traces et il faut toujours assurer la sacro sainte compatibilité ascendante. Cependant est-il utopique d'imaginer d'un côté un Java *"canal historique"* qui ne bousculerait pas trop les anciens et qui intégrerait les fonctionnalités jugées matures; et de l'autre un canal plus innovant qui ne nécessiterait pas forcément d'attendre des années pour mettre à disposition des *nouveautés* déjà présentes depuis longtemps dans d'autres technologies? Ou pourquoi pas carrément un reboot? Un Java 3? Exit les *checked exceptions*, l'effacement de type à la compilation, `java.(util|sql).Date`, JUL, *I believe I can fly, I believe I can touch the sky...*

J'ai bien compris qu'une lambda en Java n'était pas une classe anonyme et qu'un *value type* sera bien plus efficace qu'un *POJO* mais quand même, ces dépendances, que j'oserais qualifier d'évolutions internes, plombent bien les avancées du langage. Entre Java 5 et Java 8, dix ans se sont écoulés (les versions 6 et 7 n'apportent à mon goût que des changements mineurs dans notre code), j'ai envie de voir arriver de vraies nouveautés plus souvent qu'une fois par décennie et j'ai de plus en plus de mal à comprendre que ça prenne autant de temps sur la plateforme Java...

  

