# Partie 1 - Gestion de données simples
### INDEX AND UPDATE API
- (Question) Après avoir étudié la structure d’un document, ajouter un nouveau compte dont l’ID est 10001

PUT bank/_create/10001
{
  "account_number": 10001,
  "balance": 1,
  "firstname": "Lilian",
  "lastname": "Laure",
  "age": 22,
  "gender": "M",
  "address": "1 rue de rue",
  "city": "TKT",
  "state": "VAR",
  "employer": "non",
  "email": "mail@mail.mail"
}

- (Question) Mettre à jour le compte précédent en modifiant l’adresse

POST bank/_update/10001
{
  "doc": {
    "address": "2 rue de rue",
    "city": "TKTKT"
  }
}

### DELETE API
- (Question) Supprimer le compte précédemment créé

DELETE bank/_doc/10001

- (Question) Supprimer tous les comptes de la ville de Nicholson (Utiliser _delete_by_query)

POST bank/_delete_by_query
{
  "query": {
    "term": { "city.keyword": "Nicholson" }
  }
}

### GET API
- (Question) Obtenir le compte dont l’ID est 2

GET bank/_doc/2

- (Question) Idem mais récupérer uniquement le source (champs _source)

GET bank/_source/2

- (Question) Idem mais en ne sélectionnant que le nom et le prénom (firstname, lastname)

GET bank/_source/2?_source=firstname,lastname

### SEARCH API

- (Question) Retrouver tous les comptes

GET bank/_search
{
  "query": { "match_all": {} }
}

- (Question) Retrouver tous les comptes dont la ville (champs city) est Belvoir

GET bank/_search
{
  "query": {
    "term": { "city.keyword": "Belvoir" }
  }
}

- (Question) Retrouver tous les comptes dont la ville (champs city) est Belvoir ET l’employeur Xurban

GET bank/_search
{
  "query": {
    "bool": {
      "must": [
        { "term": { "city.keyword": "Belvoir" } },
        { "term": { "employer.keyword": "Xurban" } }
      ]
    }
  }
}

### Visualisation des données avec OpenSearch Dashboards

- (Question) Métrique affichant la somme des soldes de tous les comptes. Sauver le graphique.
  
  ![Somme des soldes](img_rendu/sum_balances.png)

- (Question) Graphique barre affichant la moyenne des soldes (champ balance) selon l’état (champ state). Sauver le graphique.
  
  ![Moyenne des soldes par état](img_rendu/balance_per_state.png)

- (Question) Nuage de mots représentant les villes dans lesquelles il y a le plus de comptes. Sauver le graphique.
  
  ![Nuage de mots des villes](img_rendu/word_cloud_city.png)

- (Question) Réaliser un tableau de bord regroupant les 3 visuels précédents.

  ![Tableau de bord avec les visuels](img_rendu/dashboard.png)

- (Question) Exécuter une requête dans la barre de recherche situé au dessus et remarquez que les visuels se mettent automatiquement à jour.

  ![Tableau de bord pour la ville de Belvoir](img_rendu/dashboard_after.png)

# Partie 2 - Gestion de documents textuels

### SEARCH API

- (Question) Rechercher les documents contenant le terme KING dans les champs text_entry OU playname. Accordez deux fois plus d’importance aux documents qui contiennent le terme dans le champ play_name (astuce : KING^2).

GET shakespeare/_search?q=play_name:KING^2 OR text_entry:KING

- (Question) Rechercher les documents où l’orateur (champ speaker) CAESAR parle de Brutus (champ text_entry)

GET shakespeare/_search?q=speaker:CAESAR AND text_entry:Brutus

- (Question) Rechercher les documents où l’orateur(champ speaker) CAESAR ne parle PAS de Brutus (champ text_entry)

GET shakespeare/_search?q=speaker:CAESAR AND NOT text_entry:Brutus

- (Question) Rechercher les documents répondant à la requête `caesar brutus calpurnia`

GET shakespeare/_search?q=caesar brutus calpurnia

- (Question) Selon vous, pourquoi le quatrième document, qui contient tous les termes, n’est pas en première position ?

La requête est traitée comme un `caesar OR brutus OR calpurnia`, et donc l'affichage dépends des paramètres de la recherche, donc même si il y a les 3, ils sera interprété différament

- (Question) Modifier la requête pour que seul le quatrième document réponde.

GET shakespeare/_search?q=caesar AND brutus AND calpurnia

- (Question) Rechercher les documents répondant à la requête cesar (la faute est volontaire)

GET shakespeare/_search?q=cesar

- (Question) Pourquoi aucun document ne répond ?

Aucun doc ne réponds car il y a une faute

- (Question) Essayez maintenant avec la requête cesar~ 

GET shakespeare/_search?q=cesar~

- (Question) En déduire le rôle de l’opérateur ~

La recherche est étendue pour toruver des termes similaires.

### AGGREGATION API

- (Question) Trouver le nombre total de pièces (champ play_name)

POST shakespeare/_search?q=*
{
  "size": 0,
  "aggs": {
    "nb_pieces": {
      "cardinality": { "field": "play_name.keyword" }
    }
  }
}

- (Question) En une requête, calculer le nombre de lignes (champ line_id) pour chaque pièce (champ play_name)

POST shakespeare/_search
{
  "size": 0,
  "aggs": {
    "par_piece": {
      "terms": {
        "field": "play_name.keyword"
      },
      "aggs": {
        "nb_lignes": { 
          "value_count": { "field": "line_id" } 
        }
      }
    }
  }
}

# Partie 3 - Recherches sémantiques

group_id : m2rppZoBYKbjyQMtZGfK
task_id : n2rqpZoBYKbjyQMtQGdJ
model_id : o2rqpZoBYKbjyQMtTWfj

- (Question) Rechercher "Usage of artificial intelligence in medicine". Afficher les champs headline, short_description, date et link.

GET news/_search?size=10
{
  "query": {
    "neural": {
      "text_embedding": {
        "query_text": "Usage of artificial intelligence in medicine",
        "model_id": "o2rqpZoBYKbjyQMtTWfj",
        "k": 10
      }
    }
  },
  "fields": ["liste des champs à afficher"],
  "_source": false
}

- (Question) Rechercher les articles similaires à celui-ci https://www.huffpost.com/entry/15-comedy-documentaries-worth-watching-on-netflix-photos_n_1619966. Copier / coller le début de l'article dans la requête.

GET news/_search?size=10
{
  "query": {
    "neural": {
      "text_embedding": {
        "query_text": "Netflix, to its credit, has a phenomenal archive of documentaries available to watch whenever you want. Even better, they have some really interesting comedy-related documentaries available to watch whenever you want. These movies take a poignant look at such comedy legends as Bill Hicks, Conan O'Brien, Woody Allen, Joan Rivers and more and make for great summer movie marathon-ing.With that in mind, and because going to the beach is totally overrated, we put together a list of 15 of the best comedy documentaries available on Netflix Instant. So, what are you waiting for? Plant yourself on the couch, crank up that A.C. and get to watching!",
        "model_id": "o2rqpZoBYKbjyQMtTWfj",
        "k": 10
      }
    }
  },
  "fields": ["liste des champs à afficher"],
  "_source": false
}

# Partie 4 - Pour aller plus loin : Logstach

- (Question) Développez un tableau de bord sur ces données. Ce tableau devra comporter une carte géographique des hôtels classés en fonction de leur nombre d’étoiles.

  ![Liste des hôtels en Île-de-Francd](img_rendu/hotels_map.png)
