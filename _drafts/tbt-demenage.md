---
title: blogger.map(JekyllImporter::importBlogger)
tagline: Ou la migration de mon blog
layout: post
tags : [github, jekyll, blog]
---
{% include JB/setup %}

Ça y est The Babel Tower déménage!

Ce n'est pas simplement pour répondre à l'appel des sirènes d'une plateforme plus moderne, le mal entre moi et
blogger était plus profond, voici mes griefs:

- il faut être en ligne pour créer un billet et voir l'aperçu, pas de moyen de faire tourner le blog en local.
- le méta langage des templates est peu (ou pas?) documenté
- je trouve les templates soit d'un design un peu vieillot, soit un peu trop "uzinagazesque" dans le cas des modèles
  dynamiques
- La personnalisation est assez difficile et, tiré par la tendance flat, je voulais quelque chose de plus épuré

Bref pour que votre belle mère publie la fameuse recette de son cake, Blogger reste une proposition acceptable, mais pour
 un geek ça ne le fait pas.

La cible c'est Jekyll et les pages Github. Ce choix permet de combler la plupart de mes doléances:
exécution locale, la plateforme est documentée et j'ai les pleins pouvoirs sur le modèle. Ensuite dans les bonus, la
 gestion du référentiel est assurée par git et je peux utiliser du markdown ou du HTML au choix pour rédiger, exit
 l'éditeur WYSWY(maybe)G et le text area pour éditer le HTML et ajouter du code formatté.

Bien, le plus facile, définir l'objectif, est fait. Reste à tracer la route. Dans les choix possibles, il y a:

- oublier le passé et partir d'une feuille blanche. Et mettre au rebus mes billets naïfs et pourris de 2007? No way!
- Recréer les billets les uns après les autres à coups de copier/coller. Vu la quantité de contenu, cela aurait sûrement
été plus rentable, mais franchement moins geek.
- Écrire ou utiliser un programme chargé de migrer le blog. Yes!

Pas de bol, pas de module d'import Jekyll pour Blogger. Par contre, Blogger offre la possibilité d'exporter l'intégralité
 du contenu au format XML.

Pour le langage, Java 8 vient de sortir, voilà une bonne raison de commencer à s'affûter sur les nouvelles pratiques apportées.

La trame est simple: trouver les quelques expressions XPath afin d'extraire la matière (titre, date, corps, tags)
 pour reconstruire les posts. Au passage, les documents HTML doivent être analysés afin d'identifier les images hébergées
 par blogger pour les récupérer.
 Je ne me suis pas préoccupé pour le moment des anciens commentaires, car ils ne figurent pas
   directement dans l'export, j'ai juste accès à un lien.

Quelques points mineurs m'ont posé des problèmes:

- Certains caractères ainsi que l'encoding: des URI ont été construites à la volée à partir du contenu, du coup il a fallu
gérer au coup par coup le cas de ':', ',', des accents, etc. (même URLEncodés, l'utilisation de ces caractères générait
des erreurs, donc je les ais filtrés sans me poser de question) et je me suis évidemment pris les pieds dans le tapis sur les problématiques
 d'encodage.
- D'un point de vue global, comme je ne suis ni familier de Ruby, de Jekyll ou des templates Liquid, j'ai majoritairement
fait du copier/coller de code ou de configuration. Ce n'est pas très satisfaisant, mais comme je ne désire pas m'investir
sur cette plateforme, je m'en contenterai. De plus le contenu est généré une fois et servi statiquement, donc ce n'est pas
un sujet préoccupant.
- L'un des objectifs était de sortir de Syntax Highlighter au profit de Github Flavoured Markdown, et là il faut un peu
paramétrer Jekyll en indiquant quel moteur de rendu Markdown il faut utiliser ainsi que les options:

```yaml
markdown: redcarpet

redcarpet:
  extensions: ["no_intra_emphasis", "fenced_code_blocks", "autolink", "strikethrough", "superscript"]
```

Cela permet d'analyser le post pour placer les bonnes classes dans les balises, mais si vous n'avez pas la CSS associée,
 cela ne sert que partiellement. Je n'ai pas trouvé de CSS dédiée pour la coloration syntaxique donc j'en ai fauché une
 sur Github et j'en ai extrait les syles utiles.

Pour le design, je me suis appuyé sur mes qualités artistiques internationalement plébiscitées, c'est donc avec une
audace frisant l'insolence que j'ai utilisé un [thème Bootstrap 3](https://github.com/dbtek/jekyll-bootstrap-3). Je l'ai
quelque peu manipulé afin qu'il corresponde à l'idée que je me faisais de ce blog. J'y ai rajouté un tag cloud sur mesure
(je me suis aidé avec google et le press papier) qu'il faudra sûrement réviser à l'avenir.

Reste à insérer quelques gadgets Twitter et PlusOne, l'intégration de Disqus est prévue, il n'y a qu'à donner le nom du compte dans le
fichier de conf, et c'est parti.

Concernant mon changement d'adresse sur la page Blogger, le plus facile est de rediriger le flux RSS, Blogger le permet.
Pour les posts, les permalinks n'étant pas les mêmes, prévoir un bout de code JS générique pour rediriger les pages n'est
pas possible... Donc lors de l'analyse de l'export XML, j'ai inséré quelques lignes de
JS dans chaque billet pour assurer la redirection. A la fin, l'export est sauvé dans un nouveau fichier XML prêt à être
réimporté et lorsque une page est visitée, hop le navigateur charge la page et redirige instantanément sur la bonne page
github.io.

