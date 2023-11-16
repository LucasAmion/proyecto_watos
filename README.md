# proyecto_watos
## comandos usados:
- unir archivos csv: awk 'NR==1||FNR>1' atp_matches_*.csv > atp_matches.csv
- convertir a rdf: tarql --dedup 2000000 matches_query.sparql ../archive/atp_matches.csv > ../atp_matches.ttl
- iniciar sparql endpoint: java -jar fuseki-server.jar

## queries:
Jugadores Chilenos con más visctorias:
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX ex:  <http://ex.org/a#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX data:  <http://ex.org/>

SELECT ?winner_name (COUNT(DISTINCT ?match) as ?wins) 
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
