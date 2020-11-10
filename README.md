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
- d**’exposer les principales possibilités de diffusion des décisions au public**.

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



#### 2. Sequence Tagger (NER

Construction des IOBs

​	Split en phrases & tokens

Seqtagger

Output



#### 3. Postprocessing



### Interface de pseudonymisation



 ### Mise en production



### Projets en cours

1. Monitoring
2. mise en doute statistique
3. test unitaires
4. selection pour retrain

