# Análisis dataset partidos profesionales de tennis
## Comandos usados:
- unir archivos csv: awk 'NR==1||FNR>1' atp_matches_*.csv > atp_matches.csv
- convertir a rdf: tarql --dedup 2000000 matches_query.sparql archive/atp_matches.csv > atp_matches.ttl
- iniciar sparql endpoint: java -jar fuseki-server.jar

## Queries:
```sparql 1.1
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX ex:  <http://ex.org/a#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX data:  <http://ex.org/>

# Jugadores Chilenos con más victorias:

SELECT DISTINCT ?winner_name (COUNT(DISTINCT ?match) AS ?wins) 
FROM NAMED data:atp_matches
FROM NAMED data:countries
WHERE {
  GRAPH data:countries {
    ?country ex:ioc ?ioc ;
             ex:name ?country_name .
  } .
  GRAPH data:atp_matches {
    ?match ex:winner ?winner .
    ?winner ex:name ?winner_name ;
            ex:country ?ioc .
  } .
}
GROUP BY ?winner_name ?country_name
HAVING(?country_name='Chile') 
ORDER BY DESC(?wins)
LIMIT 10

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
    SELECT ?player_height  (COUNT(DISTINCT ?player) AS ?num_players) (COUNT(DISTINCT ?match) AS ?wins) 
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

```
