# ProjectNoSQL

Il progetto **ProjectNoSQL** analizza la connettività globale degli aeroporti utilizzando tecnologie NoSQL. In particolare, usiamo i dataset open-source di [OpenFlights](https://openflights.org/data.html) (aeroporti, compagnie aeree e rotte) per:

- **Analisi statistiche** con **MongoDB** (aggregazioni) + visualizzazioni con **Pandas/Seaborn**
- **Analisi della rete dei voli** con **Neo4j** (grafo di aeroporti e rotte)

Sviluppato da **Davide Pedretti** e **Kirollos Seif**.

---

## Tecnologie usate

| Categoria | Strumenti |
|-----------|-----------|
| Linguaggio | Python 3.10+ |
| Data wrangling | Pandas, NumPy |
| Visualizzazione | Seaborn, Matplotlib |
| Database NoSQL | MongoDB (+ PyMongo) |
| Graph DB | Neo4j (+ neo4j-driver / py2neo) |
| Notebook | Jupyter |

---

## Contenuti della repo

| File/Cartella | Descrizione |
|---------------|-------------|
| `NTBK_Flights.ipynb` | Notebook principale per caricare, pulire, salvare e analizzare i dati |
| `data/` | CSV di OpenFlights (`airports.dat`, `airlines.dat`, `routes.dat`) |
| `requirements.txt` | Librerie Python necessarie |
| `imgs/` | Grafici generati dal notebook |

---

## Guida rapida al notebook `NTBK_Flights.ipynb`

1. **Setup**  
   Importa librerie e apre le connessioni a MongoDB e Neo4j.

2. **Caricamento dati**  
   ```python
   airports = pd.read_csv("data/airports.dat", header=None, names=[...])
   airlines = pd.read_csv("data/airlines.dat", header=None, names=[...])
   routes   = pd.read_csv("data/routes.dat",   header=None, names=[...])
   ```

3. **Inserimento in MongoDB**  
   ```python
   from pymongo import MongoClient
   client = MongoClient("mongodb://localhost:27017/")
   db = client.openflights
   db.airports.insert_many(airports.to_dict("records"))
   db.airlines.insert_many(airlines.to_dict("records"))
   ```

4. **Aggregazioni + grafici**  
   Query MongoDB → DataFrame → `seaborn.barplot(...)`.

5. **Costruzione grafo Neo4j**  
   Carica nodi `Airport` e relazioni `ROUTE` (`(:Airport)-[:ROUTE]->(:Airport)`).

6. **Query Cypher di esempio**  
   - Tutti i percorsi Bari → Parigi  
     ```cypher
     MATCH (a:Airport {city:'Bari'})-[:ROUTE*1..2]-(b:Airport {city:'Paris'})
     RETURN a, b
     ```
   - Solo i percorsi più brevi  
     ```cypher
     MATCH (a:Airport {city:'Bari'}), (b:Airport {city:'Paris'})
     MATCH p = shortestPath((a)-[:ROUTE*]-(b))
     RETURN p
     ```

---

## Query MongoDB di esempio

```js
// Top 10 paesi per numero di aeroporti
db.airports.aggregate([
  { $group: { _id: "$country", count: { $sum: 1 } } },
  { $sort: { count: -1 } },
  { $limit: 10 }
])

// Top 10 paesi per compagnie aeree attive
db.airlines.aggregate([
  { $group: { _id: "$country", count: { $sum: 1 } } },
  { $sort: { count: -1 } },
  { $limit: 10 }
])
```

---

## Come eseguire

1. **Clona la repo**
   ```bash
   git clone https://github.com/<user>/ProjectNoSQL.git
   cd ProjectNoSQL
   ```
2. **Installa le dipendenze**
   ```bash
   pip install -r requirements.txt
   ```
3. **Avvia i servizi**
   - MongoDB in ascolto su `mongodb://localhost:27017/`
   - Neo4j su `bolt://localhost:7687` (crea utente/password o usa i default)
4. **Esegui il notebook**
   ```bash
   jupyter notebook
   # Apri NTBK_Flights.ipynb e segui le celle
   ```

---

## Licenza

Questo progetto è distribuito con licenza **MIT**.  
I dataset sono di proprietà di **OpenFlights** (licenza Open Database / CC BY-SA).

---

## Contatti

- **Davide Pedretti** – [davide@example.com](mailto:davide@example.com)
- **Kirollos Seif** – [kirollos@example.com](mailto:kirollos@example.com)

Contributi e *pull request* sono benvenuti!
