# Querying CN-MEDO using SPARQL

## Get all FJ with Components

```SQL
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX medo: <http://www.semanticweb.org/yasen/ontologies/2024/3/cn-medo#>
SELECT ?s ?p
WHERE {?s medo:includes ?p .}
ORDER BY ?s
```
