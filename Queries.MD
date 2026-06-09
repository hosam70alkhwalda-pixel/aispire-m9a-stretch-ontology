# SPARQL Sample Queries

All queries assume the ontology is loaded into a SPARQL endpoint (e.g. Apache Jena Fuseki) at  
`http://localhost:3030/worldcup/sparql`.

Prefix block used by all queries:

```sparql
PREFIX :     <http://example.org/worldcup#>
PREFIX rdf:  <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX xsd:  <http://www.w3.org/2001/XMLSchema#>
```

---

## Query 1 — Subclass-Aware: All European World Cup Champions

Uses `rdfs:subClassOf*` to capture both direct instances of `:EuropeanNation` **and** any deeper
subclass that may be added in the future, without needing to enumerate subclasses explicitly.

```sparql
PREFIX :     <http://example.org/worldcup#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd:  <http://www.w3.org/2001/XMLSchema#>

SELECT ?teamName ?year ?score
WHERE {
  # Subclass-aware: matches EuropeanNation and any subclass of it
  ?subClass rdfs:subClassOf* :EuropeanNation .
  ?team     a ?subClass ;
            ^:champion ?wc .          # ?wc has ?team as its champion

  ?wc :heldInYear ?year ;
      :finalScore  ?score .

  BIND(STR(?team) AS ?teamName)
}
ORDER BY ?year
```

**Expected results include:** England (1966), Germany (1974, 1990, 2014), Italy (1934, 1938, 1982, 2006), France (1998, 2018), Spain (2010).

---

## Query 2 — SKOS-Disambiguated: Find a Team by Official Name OR Nickname

Returns any team whose `skos:prefLabel` **or** `skos:altLabel` matches a search string.  
This matters when a user queries with "Die Mannschaft" (the German nickname) rather than "Germany".

```sparql
PREFIX :     <http://example.org/worldcup#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX xsd:  <http://www.w3.org/2001/XMLSchema#>

SELECT ?team ?label ?titles
WHERE {
  ?team a ?type .
  ?type rdfs:subClassOf* :NationalTeam .

  # Match by prefLabel OR altLabel — SKOS disambiguation
  { ?team skos:prefLabel ?label . }
  UNION
  { ?team skos:altLabel  ?label . }

  ?team :titlesWon ?titles .

  # Change the FILTER value to any label variant, e.g. "Die Mannschaft"@de
  FILTER( ?label = "Die Mannschaft"@de
       || ?label = "Germany"@en )
}
```

**Expected result:** Returns the `:Germany` resource with `titlesWon = 4`, matched via either label.

---

## Query 3 — Domain-Specific: Champions That Also Hosted Their Winning Tournament

This is an interesting football-history question: which countries won the World Cup *on home soil*?  
The query joins `:champion` with `:hostedEdition` and checks that the hosted tournament is the
same tournament the team won — exploiting the OPTIONAL property and the graph structure together.

```sparql
PREFIX :     <http://example.org/worldcup#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX xsd:  <http://www.w3.org/2001/XMLSchema#>

SELECT ?teamLabel ?year ?score
WHERE {
  ?wc   :champion       ?team ;
        :heldInYear     ?year ;
        :finalScore     ?score .

  # :hostedEdition is the OPTIONAL property — only present on host nations
  ?team :hostedEdition  ?wc .          # same tournament: team hosted AND won it

  ?team skos:prefLabel  ?teamLabel .
  FILTER(LANG(?teamLabel) = "en")
}
ORDER BY ?year
```

**Expected results:**
| teamLabel | year | score |
|-----------|------|-------|
| Uruguay   | 1930 | 4–2   |
| Italy     | 1934 | 2–1   |
| England   | 1966 | 4–2   |
| Germany   | 1974 | 2–1   |
| Argentina | 1978 | 3–1   |
| France    | 1998 | 3–0   |

These are the six confirmed home-soil winners in World Cup history.  
Spain (2010 champion) has **no** `:hostedEdition` triple, so it is correctly excluded.