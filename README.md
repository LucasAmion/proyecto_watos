# proyecto_watos
comandos usados:
- copy atp_matches_*.csv atp_matches.csv (unir archivos csv)
- tarql matches_query.sparql archive/atp_matches.csv > atp_matches.ttl (convertir a rdf)
- fuseki-server --file ../atp_matches.ttl /atp-matches (sparql endpoint)
