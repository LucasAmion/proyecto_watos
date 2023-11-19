# Análisis dataset partidos profesionales de tennis
## Comandos usados:
- unir archivos csv: awk 'NR==1||FNR>1' atp_matches_*.csv > atp_matches.csv
- convertir a rdf: tarql --dedup 2000000 matches_query.sparql archive/atp_matches.csv > atp_matches.ttl
- iniciar sparql endpoint: java -jar fuseki-server.jar

## Queries:
```sparql
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX ex:  <http://ex.org/a#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX data:  <http://ex.org/>

# Porcentaje de victorias según mano dominante:

SELECT ?player_hand ?num_players (?wins/(?wins+?losses)*100 AS ?win_percentage)
FROM data:atp_matches # or data:wta_matches
WHERE {
  {
    SELECT ?player_hand (COUNT(DISTINCT ?match) AS ?wins) 
    WHERE {
      ?match rdf:type ex:Match ;
             ex:winner ?player .
      ?player rdf:type ex:Player ;
              ex:hand ?player_hand .
    }
    GROUP BY ?player_hand
  }
  {
    SELECT ?player_hand (COUNT(DISTINCT ?match) AS ?losses) 
    WHERE {
      ?match rdf:type ex:Match ;
             ex:loser ?player .
      ?player rdf:type ex:Player ;
              ex:hand ?player_hand .
    }
    GROUP BY ?player_hand
  }
  {
    SELECT ?player_hand (COUNT(DISTINCT ?player) AS ?num_players) 
    WHERE {
      ?player rdf:type ex:Player ;
              ex:hand ?player_hand .
    }
    GROUP BY ?player_hand
  }
}
ORDER BY ?player_hand

# Porcentaje de victorias según altura:

SELECT ?player_height ?num_players (?wins/(?wins+?losses)*100 AS ?win_percentage)
FROM data:atp_matches # or data:wta_matches
WHERE {
  {
    SELECT ?player_height (COUNT(DISTINCT ?match) AS ?wins) 
    WHERE {
      ?match rdf:type ex:Match ;
             ex:winner ?player .
      ?player rdf:type ex:Player ;
              ex:height ?height .
      BIND (ROUND(FLOOR(?height/5)*5) AS ?player_height)
    }
    GROUP BY ?player_height
  }
  {
    SELECT ?player_height (COUNT(DISTINCT ?match) AS ?losses) 
    WHERE {
      ?match rdf:type ex:Match ;
             ex:loser ?player .
      ?player rdf:type ex:Player ;
              ex:height ?height .
      BIND (ROUND(FLOOR(?height/5)*5) AS ?player_height) 
    }
    GROUP BY ?player_height
  }
  {
    SELECT ?player_height (COUNT(DISTINCT ?player) AS ?num_players) 
    WHERE {
      ?player rdf:type ex:Player ;
              ex:height ?height .
      BIND (ROUND(FLOOR(?height/5)*5) AS ?player_height)
    }
    GROUP BY ?player_height
  }
}
ORDER BY ?player_height

# Porcentaje de victorias según edad:

SELECT ?player_age (?wins+?losses AS ?matches_played) (?wins/?matches_played*100 AS ?win_percentage)
FROM data:atp_matches # or data:wta_matches
WHERE {
  {
    SELECT ?player_age (COUNT(DISTINCT ?match) AS ?wins) 
    WHERE {
      ?match rdf:type ex:Match ;
             ex:winner_age ?age .
      BIND (ROUND(FLOOR(?age/5)*5) AS ?player_age) 
    }
    GROUP BY ?player_age
  }
  {
    SELECT ?player_age (COUNT(DISTINCT ?match) AS ?losses) 
    WHERE {
      ?match rdf:type ex:Match ;
           ex:loser_age ?age .
      BIND (ROUND(FLOOR(?age/5)*5) AS ?player_age) 
    }
    GROUP BY ?player_age
  }
}
ORDER BY ?player_age

10 jugadores con más victorias:

SELECT ?player_name (COUNT(DISTINCT ?match) AS ?wins) 
FROM data:atp_matches # or data:wta_matches
WHERE {
  ?match rdf:type ex:Match ;
         ex:winner ?player .
  ?player rdf:type ex:Player ;
          ex:name ?player_name .
}
GROUP BY ?player_name 
ORDER BY DESC(?wins)
LIMIT 10

Seguimiento anual 3 mejores jugadores:

SELECT ?player_name ?year (COUNT(DISTINCT ?match) AS ?wins) 
FROM data:atp_matches # or data:wta_matches
WHERE {
  ?match rdf:type ex:Match ;
         ex:tourney ?tourney ;
         ex:winner ?player .
  ?tourney rdf:type ex:Tournament ;
           ex:tourney_date ?date.
  ?player rdf:type ex:Player ;
          ex:name ?player_name .
  BIND (YEAR(?date) AS ?year)
}
GROUP BY ?player_name ?year
HAVING (?player_name = 'Roger Federer' || # Serena Williams
        ?player_name = 'Rafael Nadal' || # Maria Sharapova
        ?player_name = 'Novak Djokovic') # Venus Williams
ORDER BY ?year DESC(?wins)

```
