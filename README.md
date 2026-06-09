# FIFA World Cup Champions Ontology

## Domain & Motivation

The domain is the **FIFA Men's World Cup** — specifically, the history of champion nations from 1930 to 2022.  
Football is the world's most widely followed sport and its results are well-documented, making it a natural fit for a knowledge-graph exercise: the entities are crisp, the relationships are meaningful, and the data is rich enough to write non-trivial SPARQL queries without becoming unwieldy.

---

## Entity Types & Relationships

```
:Continent (class)

:NationalTeam (class)
    ├── :EuropeanNation   rdfs:subClassOf :NationalTeam
    └── :SouthAmericanNation  rdfs:subClassOf :NationalTeam

:Tournament (class)
```

### Key properties

| Property | Domain | Range | Notes |
|---|---|---|---|
| `:heldInYear` | Tournament | xsd:gYear | Year of the edition |
| `:hostedBy` | Tournament | Literal | Host nation(s) |
| `:champion` | Tournament | NationalTeam | Winning team |
| `:runnerUp` | Tournament | NationalTeam | Finalist |
| `:finalScore` | Tournament | Literal | Score / penalty result |
| `:confederation` | NationalTeam | Literal | UEFA / CONMEBOL |
| `:titlesWon` | NationalTeam | xsd:integer | Aggregate titles |
| `:hostedEdition` | NationalTeam | Tournament | **OPTIONAL** — see below |

### Instances

**South American Nations** (`:SouthAmericanNation`): Brazil, Argentina, Uruguay  
**European Nations** (`:EuropeanNation`): Germany, Italy, France, England, Spain, Netherlands  
**Tournaments**: WC1930, WC1950, WC1966, WC1970, WC1974, WC1978, WC1994, WC1998, WC2006, WC2010, WC2014, WC2022

---

## Non-Obvious Modeling Decision

**Why model `NationalTeam` subclasses by confederation rather than continent?**  
The most natural split in football governance is *confederation* (UEFA, CONMEBOL, etc.), not geography.  Using `EuropeanNation rdfs:subClassOf NationalTeam` keeps the hierarchy football-meaningful: a query for "all European champions" is a real analytical question, whereas a flat list of teams with a `confederation` literal would require a `FILTER` and would not support transitive reasoning. The subclass tree enables the `rdfs:subClassOf*` path operator used in Query 1, which is the point of the exercise.

---

## SKOS Labels

### Where they appear

| Entity | `skos:prefLabel` | `skos:altLabel` | Language |
|---|---|---|---|
| `:Brazil` | "Brazil" | "Seleção Brasileira" | Portuguese |
| `:Argentina` | "Argentina" | "La Albiceleste" | Spanish |
| `:Uruguay` | "Uruguay" | "La Celeste" | Spanish |
| `:Germany` | "Germany" | "Die Mannschaft" | German |
| `:Italy` | "Italy" | "Gli Azzurri" | Italian |
| `:France` | "France" | "Les Bleus" | French |
| `:England` | "England" | "Three Lions" | English |
| `:Spain` | "Spain" | "La Roja" | Spanish |
| `:Netherlands` | "Netherlands" | "Oranje" | Dutch |
| `:Tournament` | "Tournament" | "World Cup Edition" | English |
| `:NationalTeam` | "National Football Team" | "International Side" | English |

### What they enable

Fans, journalists, and multilingual databases frequently use **nicknames** ("Die Mannschaft", "La Albiceleste") rather than formal country names. Storing both labels on the same resource — instead of creating separate resources — lets a SPARQL query match by *either* label without duplicating triples or joining across two nodes. Query 2 demonstrates this: it retrieves a champion regardless of whether the user searches by official name or popular nickname.

---

## OPTIONAL Property

`:hostedEdition` links a national team to a tournament their country hosted. This property is **intentionally absent** from `:Spain` (and any team that never hosted while being a champion) because Spain has not hosted a World Cup. This models the real-world condition that hosting is an additional, non-universal attribute of a team — not every champion has also been a host. A SPARQL query using `OPTIONAL { ?team :hostedEdition ?wc }` will return `UNDEF` for Spain while returning the correct tournament for Brazil, Argentina, Germany, etc.

---

## Triple Count

The ontology contains **179 triples** (verified via rdflib parse), including class declarations, property definitions, and all instance data — well above the 30-triple minimum while remaining focused on the domain.
