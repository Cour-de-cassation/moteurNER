# Moteur de pseudoanonymisation de la Cour de Cassation

1. [TOC]

##  Pourquoi Open data de la justice ?



   « ***La justice doit être accessible et la Cour de cassation s’engage à relever le défi en utilisant les potentialités des technologies  appliquées au droit*** » - *Chantal Arnes, première présidente de la Cour de cassation, pour La Semaine juridique - édition générale (30.03.20)*



### Qu'es Open Data de la justice?

La diffusion en **Open data**, littéralement  «  **données  ouvertes**  »,  de  l’ensemble des décisions de justice, est prévue par la loi n° 2016-1321 pour une République numérique  du  7  octobre  2016.  Chaque  année ce  sont  près  de  3,9  millions  de  décisions de  justice  qui  sont  rendues  par  les  juridictions  françaises. 



### Open Data de la justice et la Cour de Cassation

Le rôle de la Cour de cassation est central dans la diffusion de décisions en Open Data. 

«*Il est admis, depuis les conclusions de la mission Cadiet, que la **Cour de cassation a vocation à se voir confier, dans le prolongement naturel de ses missions présentes, la responsabilité exclusive de la diffusion en open data des décisions de justice judiciaires** qui sera faite en application de l’article L. 111-13 du Code de l’organisation judiciaire.* » 



La Cour de cassation, conformément aux recommandations de la mission Cadiet, assurera le pilotage et la responsabilité :

**•** de la collecte automatisée de la jurisprudence de l’ensemble des juridictions de son périmètre juridictionnel ; 

**•** de son traitement, en particulier la pseudonymisation ; 

**•** de sa diffusion.



### Et concrètement...

La mise en  œuvre  normative et technique  de cet open data dans la perspective devrait comporter les éléments suivant:

- de  renforcer  les  techniques  existantes  dites  de «pseudonymisation» des  décisions,  afin  d’**assurer  la protection de la vie privée des personnes, qui est garantie par la loi pour une République numérique**;
- d’instituer une régulation des algorithmes qui exploitent les données issues des décisions, afin d’assurer une transparence sur les méthodologies mises en œuvre;
- de définir les principes directeurs de l’architecture nouvelle de l’open data, en confiant la gestion des bases à la  Cour  de  cassation  et  au  Conseil  d’Etat,  ainsi  que  la  mission  essentielle  de «pseudonymisation» des décisions collectées auprès des juridictions;
- **d’exposer les principales possibilités de diffusion des décisions au public**.

![objectifopendata](https://raw.githubusercontent.com/Cour-de-cassation/moteurNER/main/img/objectifopendata.png)

A ce jour, 180 000 décisions par an sont collectées dans les bases de  données « Jurinet » et « Jurica » tenues par la Cour de cassation.  Sur ces 180 000 décisions, moins de 15 000 étaient diffusées en *open data* et sur le site Légifrance. 

Le logiciel d'anonymisation reposant sur un moteur de règles Luxide, qui fonctionnait de janvier 2018 au septembre 2019, dont le taux d'erreur rapprochait 5% demandait un temps important de la correction manuelle. De plus, les évolutions de ce logiciel étaient chronophages et coûteuses. Il n'était pas adapté à faire face à augmentation et diversification du flux. 

### Programme EIG

La Cour de cassation a fait acte de candidature dans le cadre du programme « Entrepreneurs d’intérêt général » (EIG) en proposant de **développer des   techniques   d’apprentissage  automatique  destinées  à identifier les données à pseudonymiser**   (retirer   les données  directement  identifiantes)  dans  les décisions de justice avant de les rendre accessibles en ligne et réutilisables.

![](https://raw.githubusercontent.com/Cour-de-cassation/moteurNER/main/img/logo-marianne-eig-site.png)

 

### Savoir plus...

Pour apprendre plus sur le projet EIG 3 "Open Justice": [le site officiel]( https://entrepreneur-interet-general.etalab.gouv.fr/defis/2019/openjustice.html) 

Sur l'open data à la Cour de Cassation: [sur le site de la Cour de Cassation](https://www.courdecassation.fr/institution_1/intelligence_artificielle_7985/open_data_intelligence_artificielle_7821/open_data_decisions_justice_9787/)



## Le moteur de pseudoanonymisation



### Les briques du moteur NLP:



#### 1. Language Model

![](https://raw.githubusercontent.com/Cour-de-cassation/moteurNER/main/img/jurica_jurinet.png)

Language Model (le modèle du langage) permet d'obtenir des vecteurs multidimensionnels de mots (embeddings) qui décrivent dans un langage mathématique le mot (ou token plus précisément dans son contexte).

Il existe de nombreuses méthodes d'obtention des embeddings, word2vec, GloVe, BERT, ELMo ...

Dans notre modèle en production, nous utilisons une combinaison des embeddings Fasttex (une variante de word2vec) [ref] et Flair (characted-based embeddings) [ref].

Nous sommes également en train de tester d'autres combinaisons d'embeddings dont BPEembeddings et CamemBERT

Il est possible d'utiliser des vecteurs déjà entrainés, par exemple sur Wikipédia. Néanmoins pour avoir une meilleure représentation de mots dans leur contexte, nous avons entrainé notre propre language models sur le stock de décisions de la Cour de Cassation (base jurinet) et des Cours d'Appels (base jurica) - **environ XXX décisions intègres**.

Les texte de décisions a été extrait et formaté pour permettre l'entrainement du langage.



#### Fasttext

Pour le LM fasttext, nous avons utilisé les paramètres décrites dans l'article [Learning Word Vectors for 157 Languages](https://arxiv.org/abs/1802.06893)

```python
from fastText import train_unsupervised

# fasttext_model = train_unsupervised(input='your/tokenized/txt/file.txt', model='cbow', lr=0.5,  neg=100, epoch=25)
# To reduce vocabulary size which was huge (more than 1M words), we fix minCount to 100
# We augment windows size to 8 and dim to 300
fasttet_model = train_unsupervised(input='your/tokenized/txt/file.txt', model='cbow', lr=0.5,  neg=100, epoch=25, ws=8, dim=300, minCount=100)
fasttext_model.save_model('fasttext_model.bin')
```

 #### Flair embeddings 

Les flairs embeddings ont été entrainé suivant les [instructions officielles](https://github.com/flairNLP/flair/blob/master/resources/docs/TUTORIAL_9_TRAINING_LM_EMBEDDINGS.md). Les vecteurs ont été fine-tunés à partir de vecteurs: "fr-forward" et "fr-backward".

Les paramètres utilisés pour entrainer les flair embeddings:

**[...]**



#### 2. Sequence Tagger (NER)



#### Données annotées 

Contrairement à l'entrainement du language model, qui ne demande pas de données étiquetées, l'apprentissage d'un modèle de reconnaissance des entités nommées (Named Entity Recognition, NER) ne peut pas fonctionner sans les exemples (une quantité importante de données étiquetés/labelisées/annotées). 

Grâce au travail manuel des agents, à la Cour de Cassation, il existait un nombre important de données annotées. 

Exemple d'une phase annotée:



*<tag: prenom start:0 end:4> Jean <tag: nom start:5 end:12 > Du Bois né le <tag:date_de_naissance start:18 end:33> 31 janvier 2000 a fondé une entreprise d'informatique <tag: personne_morale start: end:> INFOWORLD.*

#### Construction des fichiers au format IOB

Afin de traduire les données annotées en format le plus commun pour l'apprentissage NER, il fallait diviser le texte en phrases et en tokens.

Nous avons utilisé une version modifié de `spacy.tokenizer()` de la version spacy 2.1.6. Nous  sommes en train de tester le tokenizer et sentence splitter de la nouvelle version 3.0.

Un fichier IOB de la phrase de l'exemple précédent ressemblerait à 

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

Où la première colonne correspond au token, deuxième au label (tag), troisième indique si le token est suivi par une espace et quatrième indique id de document.

Les document annotées transformés en fichiers iob, ont été revu par les data scientist afin de constituer un jeu de données "**GOLD**", un corpus de 183145 train + 24944 dev + 21721 test sentences.

#### Sequence Tagger

Nous avons utilisé le framework flair [ref] pour entrainer le modèle BILSTM + CRF. Les vecteurs de mots discuté au dessus ont été utilisée en entrée du modèle:

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

Le modèle entrainé est ensuite utilisé à prédire les entités pour les nouveaux documents. 

Le code est rendu exploitable sous la forme d'une librairie. 

Une autre librairie python sert d'une api REST conçu avec Flask.  

Grâce à l'api, le texte d'une décision stockée dans les bases internes est lu, séparé en phrases et tokenisé, les entités détectées par le modèle et ensuite les positions de ces entités dans le textes sont stockés dans un format json.


```json
{
    "entities": [
        {
            "text": "FROUIN",
            "start": 844,
            "end": 850,
            "label": "professionnelnom",
            "source": "NER model"
        },
        {
            "text": "AG2R Réunica  prévoyance",
            "start": 1227,
            "end": 1251,
            "label": "personnemorale",
            "source": "NER model"
        },
        {
            "text": "Slove",
            "start": 2453,
            "end": 2458,
            "label": "professionnelnom",
            "source": "NER model"
        }]
}        
```

#### 4. Postprocessing

En plus de la prédiction du modèle deep learning, nous appliquons plusieurs corrections *déterministes* qui permettent de corriger les fautes communs, omission du modèle. Si un nom où prénom est détectée plus de 2 fois dans décision, toutes les occurrences de ces entités seront annotés. 

La catégorie "physicomorale" qui englobe les personnes morales dont le nom contient le nom de famille d'une partie, elle est constituée par une fonction déterministe en vérifiant si dans le noms de personnes morales détectés par l'algorithme un nom d'une personne physique est présent. 

Les numéros normalisées comme nr de carte bancaires qui apparaissent très rarement dans les décisions dont également mieux détecté par une règle regex.  



##### Rapport d'anonymisation

Avec des règles déterministes, certains doutes sont levés et communiqué sous la forme des messages qui seront visible pour les annotateurs dans l'interface. Nous appelons l'ensemble de ces message 'rapport d'anonymisation' dans notre jargon.

Exemple d'un doute ce serait un nom de longueur de 1 à 2 lettres, M. Cédric O serait signalé comme un doute dans le rapport. Egalement si deux entreprises de noms "Pino" et "Pinot" sont détectées dans le même décision, on se posera la question s'il s'agit de la même entreprise. 



### Interface de pseudonymisation

Actuellement l'équipe d'annotateurs de la Cours de Cassation dispose d'une interface qui reçoit les documents de la base de documents, d'autre part elle reçoit les positions des entités  et elle permet d'afficher les données labelisées. Les annotateurs peuvent corriger les annotations: en ajouter, supprimer ou modifier. Ils ont aussi un aperçu du document pseudonymisé (où les entités voulues sont remplacées par un texte de remplacement). Cette interface a des fonctionnalités limitées pour les annotateurs, amis aussi ne permet pas une remontée d'information directe vers le moteur de pseudonymisation, ou plutôt son module de suivi de performances (lire plus bas). L'alimentation de la base d'entrainement/test du modèle n'est pas automatisé.

Pour ceci un appel à projet EIG4 a été lancé. Une nouvelle équipe des entrepreneurs intérêt général travaille sur la conception d'une nouvelle interface. Le projet est open source et devrait permettre à d'autres institutions et organisations s'en inspirer pour leur interfaces: [Le repo github du projet Label ](). Cette nouvel interface serait également un outil de gestion et fermera le cercle de remontée d'information vers le moteur de pseudonymisation. Elle permettra également une meilleure gestion d'annotations, contrôle de qualité et une meilleure remontée des erreurs vers les équipes techniques.



### Projets en cours

Actuellement le flux de documents est automatisé dans une seule direction: les documents arrivant dans la base sont pseudonymisés par le moteur par batches toutes les semaines, ensuite ils sont transmis vers interface d'annotation et finalement remontés dans la base documentaire et exportés vers la DILA (ref). Le modèle est actuellement évalué par les annotateurs en comptabilisant le nombre de décisions traités par jour et le nombre de sous-anonymisations (de données sensibles qui n'ont pas été pseudonymisées). Une remontée automatique de corrections manuels et son évaluation par équipe technique n'est pas, à ce jour, automatisée.

##### Suivi de performances

Un calcul de performances du modèle de nouveaux documents annotés en prenant en compte différents critères est une information clef que nous allons calculer. Un nouveau module de suivi de performances permettra de calculer la performance du modèle et la valeur ajouté du postprocessing. Ceci aidera a indiquer les points faibles du modèle.

##### Contrôle de qualité

Les phrases les plus difficiles, les erreurs de modèle vont enrichir la base de **tests unitaires** qui assurent un contrôle de qualité du modèle. Il est important que les erreurs connus et communs ne soit pas reproduits par le modèle. 



##### Sélection de documents pour améliorer le modèle

##### Mise en doute statistique

entropy et companie

##### 

 

##### Alertes





![](https://raw.githubusercontent.com/Cour-de-cassation/moteurNER/main/img/moteur_pseudo.png)

![](https://raw.githubusercontent.com/Cour-de-cassation/moteurNER/main/img/monitoring.png)

1. Monitoring
2. mise en doute statistique
3. test unitaires
4. sélection pour retrain

