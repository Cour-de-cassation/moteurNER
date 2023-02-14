# Moteur de pseudonymisation de la Cour de Cassation

## Pourquoi l'Open data des décisions de justice ?

   « ***La justice doit être accessible et la Cour de cassation s’engage à relever le défi en utilisant les potentialités des technologies  appliquées au droit*** » - *Chantal Arnes, première présidente de la Cour de cassation, pour La Semaine juridique - édition générale (30.03.20)*

### Qu'est l'ouverture des décisions de justice ?

La diffusion en **Open data**, littéralement  «  **données  ouvertes**  »,  de  l’ensemble des décisions de justice, est prévue par la loi n° 2016-1321 pour une République numérique  du  7  octobre  2016.  Chaque  année ce  sont  près  de  3,9  millions  de  décisions de  justice  qui  sont  rendues  par  les  juridictions  françaises.

### Open Data et la Cour de Cassation

Le rôle de la Cour de cassation est central dans la diffusion de décisions en Open Data.

«*Il est admis, depuis les conclusions de la mission Cadiet, que la **Cour de cassation a vocation à se voir confier, dans le prolongement naturel de ses missions présentes, la responsabilité exclusive de la diffusion en open data des décisions de justice judiciaires** qui sera faite en application de l’article L. 111-13 du Code de l’organisation judiciaire.* »

La Cour de cassation, conformément aux recommandations de la mission Cadiet, assurera le pilotage et la responsabilité :

**•** de la collecte automatisée de la jurisprudence de l’ensemble des juridictions de son périmètre juridictionnel ;

**•** de son traitement, en particulier la pseudonymisation ;

**•** de sa diffusion.

### Et concrètement...

La mise en  œuvre  normative et technique de cet open data dans la perspective devrait comporter les éléments suivant:

- de  renforcer  les  techniques  existantes  dites  de «pseudonymisation» des  décisions,  afin  d’**assurer  la protection de la vie privée des personnes, qui est garantie par la loi pour une République numérique**;
- d’instituer une régulation des algorithmes qui exploitent les données issues des décisions, afin d’assurer une transparence sur les méthodologies mises en œuvre;
- de définir les principes directeurs de l’architecture nouvelle de l’open data, en confiant la gestion des bases à la  Cour  de  cassation  et  au  Conseil  d’Etat,  ainsi  que  la  mission  essentielle  de «pseudonymisation» des décisions collectées auprès des juridictions;
- **d’exposer les principales possibilités de diffusion des décisions au public**.

![objectifopendata](https://raw.githubusercontent.com/Cour-de-cassation/moteurNER/main/img/objectifopendata.png)

A ce jour, 180 000 décisions par an sont collectées dans les bases de  données « Jurinet » et « Jurica » tenues par la Cour de cassation.  Sur ces 180 000 décisions, moins de 15 000 étaient diffusées en *open data* et sur le site Légifrance.

Le logiciel d'anonymisation reposant sur un moteur de règles Luxid, qui fonctionnait de janvier 2018 à décembre 2019, dont le taux d'erreur s'approchait de 5% demandait un temps important de correction manuelle. De plus, les évolutions de ce logiciel étaient chronophages et coûteuses. Il n'était pas adapté pour faire face à l'augmentation et et la diversification du flux.

### Programme EIG

La Cour de cassation a proposé sa candidature dans le cadre du programme « Entrepreneurs d’intérêt général » (EIG) en souhaitant **développer des   techniques   d’apprentissage  automatique  destinées  à identifier les données à pseudonymiser**   (retirer   les données  directement  identifiantes)  dans  les décisions de justice avant de les rendre accessibles en ligne et réutilisables.

![](https://raw.githubusercontent.com/Cour-de-cassation/moteurNER/main/img/logo-marianne-eig-site.png)

### En savoir plus...

Pour en apprendre plus sur le projet EIG 3 "Open Justice": [le site officiel]( https://entrepreneur-interet-general.etalab.gouv.fr/defis/2019/openjustice.html) 

Sur l'open data à la Cour de Cassation: [sur le site de la Cour de Cassation](https://www.courdecassation.fr/institution_1/intelligence_artificielle_7985/open_data_intelligence_artificielle_7821/open_data_decisions_justice_9787/)

## Le moteur de pseudonymisation

### Les briques du moteur NLP:

#### 1. Language Model

![](https://raw.githubusercontent.com/Cour-de-cassation/moteurNER/main/img/jurica_jurinet.png)

Language Model (le modèle du langage) permet d'obtenir des vecteurs multidimensionnels de mots (embeddings) qui décrivent dans un langage mathématique le mot (ou token plus précisément dans son contexte).

Il existe de nombreuses méthodes d'obtention des embeddings, word2vec, GloVe, BERT, ELMo ...

Dans notre modèle en production, nous utilisons une combinaison de [Byte Pair Embeddings](https://aclanthology.org/L18-1473/) et de [Flair Embeddings](https://www.aclweb.org/anthology/C18-1139/).

Il est possible d'utiliser des vecteurs déjà entrainés, par exemple sur Wikipédia. Néanmoins pour avoir une meilleure représentation de mots dans leur contexte, nous avons entrainé notre propre language models sur le stock de décisions de la Cour de Cassation (base jurinet) et des Cours d'Appels (base jurica) - **environ 2 millions décisions intègres**.

Les texte de décisions a été extrait et formaté pour permettre l'entrainement du langage.

#### Byte Pair Embeddings

Les Byte Pair Embeddings ont été entrainés à l'aide du répertoire de code [piecelearn](https://github.com/stephantul/piecelearn) disponible sur  GitHub.

#### Flair embeddings

Les flair embeddings ont été entrainés suivant les [instructions officielles](https://github.com/flairNLP/flair/blob/master/resources/docs/TUTORIAL_9_TRAINING_LM_EMBEDDINGS.md). Les vecteurs ont été fine-tunés à partir de vecteurs: "fr-forward" et "fr-backward".


#### 2. Sequence Tagger (NER)

#### Données annotées

Contrairement à l'entrainement du modèle de langage, qui ne demande pas de données étiquetées, l'apprentissage d'un modèle de reconnaissance d'entités nommées (Named Entity Recognition, NER) ne peut pas fonctionner sans exemples (une quantité importante de données étiquetés/labelisées/annotées).

Grâce au travail manuel des agents à la Cour de Cassation, un nombre important de données annotées est disponible.

Exemple d'une phase annotée:

*<tag: prenom start:0 end:4> Jean <tag: nom start:5 end:12 > Du Bois né le <tag:date_de_naissance start:18 end:33> 31 janvier 2000 a fondé une entreprise d'informatique <tag: personne_morale start: end:> INFOWORLD.*

#### Construction des fichiers au format IOB

Afin de traduire les données annotées en format le plus commun pour l'apprentissage NER, il fallait diviser le texte en phrases et en tokens. Pour ce faire, nous avons utilisé une version modifiée du tokenizer français de [spaCy](https://spacy.io/) (v.2.1.6).

Un fichier IOB de la phrase de l'exemple précédent ressemblerait à :

```
Jean B-prenom + arret_id
Du B-nom + arret_id
Bois I-nom + arret_id
né O + arret_id
le O + arret_id
31 B-date_de_naissance + arret_id
janvier I-date_de_naissance + arret_id
2000 I-date_de_naissance + arret_id
a O + arret_id
fondé O + arret_id
une O + arret_id
entreprise O + arret_id
INFOWORLD B-personne_morale + arret_id
.  O - arret_id
```

Où la première colonne correspond au token, la deuxième au label (tag), la troisième indique si le token est suivi par une espace et la quatrième indique l'identifiant du document.

Les document annotées transformés en fichiers iob, ont été revu par les data scientist afin de constituer un jeu de données "**GOLD**", un corpus de 183145 (train) + 24944 (dev) + (21721) test phrases.

#### Sequence Tagger

Nous avons utilisé le framework flair [ref] pour entrainer le modèle BILSTM + CRF. Les vecteurs de mots discutés ci-dessus ont été utilisés en entrée du modèle:

```
Model: "SequenceTagger(
  (embeddings): StackedEmbeddings(
    (list_embedding_0): WordEmbeddings('fasttext_model.gensim')
    (list_embedding_1): FlairEmbeddings(
      (lm): LanguageModel(
        (drop): Dropout(p=0.5, inplace=False)
        (encoder): Embedding(275, 100)
        (rnn): LSTM(100, 1024)
        (decoder): Linear(in_features=1024, out_features=275, bias=True)
      )
    )
    (list_embedding_2): FlairEmbeddings(
      (lm): LanguageModel(
        (drop): Dropout(p=0.5, inplace=False)
        (encoder): Embedding(275, 100)
        (rnn): LSTM(100, 1024)
        (decoder): Linear(in_features=1024, out_features=275, bias=True)
      )
    )
  )
  (word_dropout): WordDropout(p=0.05)
  (locked_dropout): LockedDropout(p=0.5)
  (embedding2nn): Linear(in_features=2148, out_features=2148, bias=True)
  (rnn): LSTM(2148, 256, num_layers=2, batch_first=True, dropout=0.5, bidirectional=True)
  (linear): Linear(in_features=512, out_features=45, bias=True)
  (beta): 1.0
  (weights): None
  (weight_tensor) None
)
```

Avec les paramètres  les plus importants du modèle séquence tagger:

```
Parameters:
learning_rate: "0.1"
mini_batch_size: "32"
patience: "2"
anneal_factor: "0.5"
max_epochs: "100"
shuffle: "True"
train_with_dev: "False"
batch_growth_annealing: "False"
```

#### Résultat

Les résultats d'entrainement sur le jeu de données test est actuellement (pour les entités les plus fréquentes):

```bash
######################
personnephysiquenom precision: 0.9711 - recall: 0.9777 - accuracy: 0.9500 - f1-score: 0.9744
personnephysiqueprenom precision: 0.9326 - recall: 0.9547 - accuracy: 0.8931 - f1-score: 0.9435
#######################
personnemorale precision: 0.8630 - recall: 0.9286 - accuracy: 0.8093 - f1-score: 0.8946
#######################
adresse precision: 0.9110 - recall: 0.8238 - accuracy: 0.7624 - f1-score: 0.8652
#######################
datedenaissance precision: 0.9571 - recall: 0.9630 - accuracy: 0.9231 - f1-score: 0.9600
#######################
professionnelnom precision: 0.9706 - recall: 0.9739 - accuracy: 0.9460 - f1-score: 0.9722
professionnelprenom precision: 0.9800 - recall: 0.9608 - accuracy: 0.9424 - f1-score: 0.9703
#######################
```

`MICRO_AVG: acc 0.8882 - f1-score 0.9408`

#### 3. Prédiction & mise en production

Le modèle entrainé est ensuite utilisé pour prédire les entités présentes dans de nouvelles décisions de justice.

Le code est rendu exploitable sous la forme d'une librairie.

Une autre librairie python sert d'une api REST conçu avec Flask.  

Grâce à l'API, le texte d'une décision stockée dans les bases internes est lu, séparé en phrases et tokenisé, les entités détectées par le modèle et ensuite les positions de ces entités dans le textes sont stockés dans un format json.

```json
{
    "entities": [
        {
            "text": "FROUIN",
            "start": 844,
            "end": 850,
            "label": "professionnelMagistrat",
            "source": "NER model"
        },
        {
            "text": "AG2R Réunica  prévoyance",
            "start": 1227,
            "end": 1251,
            "label": "personneMorale",
            "source": "NER model"
        },
        {
            "text": "Slove",
            "start": 2453,
            "end": 2458,
            "label": "professionnelAvocat",
            "source": "NER model"
        }]
}
```

#### 4. Postprocessing

En plus de la prédiction du modèle deep learning, nous appliquons plusieurs corrections *déterministes* qui permettent de corriger les fautes communes et les omissions du modèle. Par exemple, si un nom ou prénom est détecté plus de deux fois dans une décision, toutes les occurrences de cette entité seront annotées.

La catégorie "physicomorale" qui englobe les personnes morales dont le nom contient le nom de famille d'une partie, elle est constituée par une fonction déterministe en vérifiant si dans le noms de personnes morales détectés par l'algorithme un nom d'une personne physique est présent.

Les numéros normalisées comme ceux de carte bancaires ou de sécurité sociale qui apparaissent très rarement dans les décisions sont également mieux détectés par des expressions régulières.

En tout, 8 méthodes déterministes sont appliquées sur chaque décision pour rendre la détection d'entités nommées plus robuste.

##### Rapport d'anonymisation

Avec des règles déterministes, certains doutes sont levés et communiqués sous la forme des messages qui seront visible par les annotateurs dans l'interface de pseudonymistion. Nous appelons l'ensemble de ces message le 'rapport d'anonymisation'.

Par exemple, si un nom de famille ne contient qu'une ou deux lettres, il sera signalé comme un doute dans le rapport. Egalement si deux prénoms "Thibaut" et "Thibault" sont détectés dans le même décision, on demandera s'il s'agit de la même personne ou non.

### Interface de pseudonymisation

Actuellement l'équipe d'annotateurs de la Cour de Cassation dispose d'une interface leur permettant de vérifier les entités détectées par le moteur de pseudonymisation (machine learning + post-traitement). Les annotateurs peuvent corriger les annotations : en ajouter, supprimer ou modifier. Ils ont aussi un aperçu du document pseudonymisé (où les entités sont remplacées par un texte de remplacement). Cette interface a des fonctionnalités limitées pour les annotateurs, mais aussi ne permet pas une remontée d'information directe vers le moteur de pseudonymisation, ou plutôt son module de suivi de performances (lire plus bas). L'alimentation de la base d'entrainement/test du modèle n'est pas automatisée.

Afin de concrétiser la mise en oeuvre de l'Open Data, la Cour de Cassation a de nouveau répondu à l'appel à projet Entrepreneur d'Intérêt Général (EIG4). Une nouvelle équipe, composée de deux développeurs et d'un designer, travaille sur la conception d'une nouvelle interface de pseudonymisation. Avec le développement de celle-ci, la Cour aura pleinement la main sur l'ensemble de la pseudonymisation et ne sera plus dépendante de préstataires externes. Le projet est open source et devrait permettre à d'autres institutions et organisations s'en inspirer pour leur interfaces: [Le projet Label](https://github.com/Cour-de-cassation/label). Cette nouvel interface sera également un outil de gestion et fermera le cercle de remontée d'information vers le moteur de pseudonymisation. Elle permettra également une meilleure gestion d'annotations, contrôle de qualité et une meilleure remontée des erreurs vers les équipes techniques.

### Projets en cours

Actuellement le flux de documents est automatisé dans une seule direction : les documents arrivant dans la base sont pseudonymisés par le moteur toutes les semaines, ensuite ils sont transmis vers l'interface d'annotation et finalement remontés dans la base documentaire et exportés vers la DILA (ref). Le modèle est actuellement évalué par les annotateurs en comptabilisant le nombre de décisions traités par jour et le nombre de sous-anonymisations (de données sensibles qui n'ont pas été pseudonymisées). Une remontée automatique des corrections manuels et son évaluation par l'équipe technique n'est pas, à ce jour, automatisée.

#### Suivi de performances

Un calcul de performances du modèle sur de nouveaux documents annotés en prenant en compte différents critères est une information clé que nous allons calculer. Un nouveau module de suivi des performances permettra de calculer la performance du modèle et la valeur ajoutée du post-traitement. Ceci aidera à indiquer les points faibles du modèle.

#### Contrôle de qualité

Les phrases les plus complexes dans lesquelles le modèle *deep learning* vont venir enrichir la base de **tests unitaires** qui assurent un contrôle de qualité du modèle. Il est important que les erreurs connues et communes ne soient pas reproduites par les nouvelles version du modèle, qu'une augmentation sur un axe ne soit pas une détérioration sur un autre.

A terme un suivi automatisé permettra la détection des erreurs et ajout de phrases au panel de tests unitaires.  Il est possible d'ajouter également les variations de ces phrases, par une augmentation du langage (text augmentation) ce qui testera si le modèle n'a pas appris un exemple par cœur sans généraliser.

#### Sélection des phrases pour améliorer le modèle

Actuellement, la base d'entrainement est enrichie régulièrement par des nouveaux documents annotés. Il se peut qu'avec cette technique le modèle arrive un jour à un plafond de verre où plus de donnée ne se traduit pas en amélioration de performances.  Pour ceci une sélection des contenus qui impactent significativement les performances du modèle peut être plus intéressant. Avec moins de données mais mieux sélectionnés, on métrise aussi le poids du modèle qui est un facteur important en production.  

A la base de détection des erreurs, il possible d'ajouter dans l'entrainement que les phrases en erreur remplissant certaines critères et faire évaluer les performances du modèle.

#### Mise en doute statistique: prédire ses propres erreurs

Il serait intéressant de savoir à quel point on peut se fier à une prédiction. Ceci est possible grâce à une indice de confiance ou certitude qui se calcule pour chaque entité. Il n'existe pas un façon bien établi de cette confiance. Elle peut être estimé à partir de distribution de probabilités, entropie ou marge maximale. En soit, les valeur de confiance ne peuvent pas se traduire en pourcentage ou une confiance pour le document dans sa entièreté. Pour les entités composées de plusieurs tokens, le calcul n'est pas trivial non plus. 

Une traduction de ces valeurs mathématiques vers une vérité du terrain permettra dans le futur de lever des doutes pour les documents et entités dont les prédictions sont moins fiables que les autres. Nous envisageons de nous inspirer de techniques d'online learning et entrainer un classifier qui aidera à définir le seuil de déclenchement des alertes de confiance. 

A terme ces alertes pourraient être intégrés dans l'interface de suivi et peut-être même dans l'interface d'utilsateur.



![](https://raw.githubusercontent.com/Cour-de-cassation/moteurNER/main/img/moteur_pseudo.svg)

![](https://raw.githubusercontent.com/Cour-de-cassation/moteurNER/main/img/monitoring.svg)

