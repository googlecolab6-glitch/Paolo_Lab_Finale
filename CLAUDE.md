# CLAUDE.md – Assistente di viaggio FVG (Progetto finale ITS, Traccia B)

> Bozza iniziale: da completare/modificare a cura del gruppo.

## Contesto
- Prototipo per il progetto finale ITS, gruppo di 3 persone, consegna entro il **16 dicembre** via Git.
- Assistente che aiuta un turista a organizzare una visita in Friuli Venezia Giulia usando
  RAG su Wikivoyage (it), meteo e distanze (Open-Meteo), con modello locale su LM Studio.
- Il codice va spiegato alla lavagna da ogni membro: **leggibile, poco "magico", poche dipendenze**,
  commenti dove le scelte non sono ovvie. Sempre la soluzione più semplice che soddisfa i requisiti.

## Lingua
- Comunicazione con il gruppo, README, design doc, piano e rapporto di valutazione: **in italiano**.
- Nomi degli strumenti MCP esattamente: `cerca_luoghi`, `meteo`, `distanza`, `salva_itinerario`.

## Ruoli e confini del repository
1. **Agente e strumenti**: ciclo dell'agente, prompt di sistema, server MCP, strumenti `meteo` e `distanza`.
2. **Documenti e retrieval**: ingestion Wikivoyage, pulizia, chunking, metadati, embedding, indice, `cerca_luoghi`.
3. **Applicazione**: interfaccia web, database conversazioni, integrazione con l'agente, Docker Compose.

Ogni ruolo lavora nella propria cartella; il codice condiviso è solo quello dei contratti.

## Contratti tra componenti
- Contratto 1: risultati di `cerca_luoghi` (+ formato di errore unico per tutti gli strumenti).
- Contratto 2: messaggi tra interfaccia e agente.
- Documentati in `docs/contratti.md`, definiti nel codice in **un unico punto condiviso**, con
  finti componenti e test di contratto.
- **Un contratto cambia solo con una modifica esplicita**, segnalata al gruppo, che aggiorna
  insieme documento, definizione condivisa, esempi e test.

## Regole non negoziabili
- Ciclo agente con condizione di arresto esplicita e **numero massimo di iterazioni configurabile**;
  al limite risponde in modo controllato.
- **Citazioni verificate nel codice**: le fonti mostrate si costruiscono dai risultati reali di
  `cerca_luoghi` nel turno (tramite id del chunk); quelle non riscontrabili vengono scartate/segnalate.
- Calcoli deterministici (distanze, date relative come «sabato»/«domani», fuso Europe/Rome) fatti dal codice,
  mai dal modello.
- Errori e risultati vuoti degli strumenti gestiti esplicitamente; niente invenzioni.
- Tutto parte con `docker compose up`; LM Studio sull'host è l'unico prerequisito esterno
  (raggiungibile dai container, es. `host.docker.internal`, configurabile via `.env`).
- Fonte Wikivoyage citata (licenza CC BY-SA) in interfaccia e README.

## Flusso di lavoro
- Processo superpowers: brainstorming → design doc approvato → piano → esecuzione
  (subagent-driven-development) con TDD su ogni task, worktree per isolare il lavoro,
  systematic-debugging per i bug, code review a fine blocco, verification-before-completion.
- **Nessun codice prima dell'approvazione del design doc.**
- I test automatici non dipendono da rete né da LM Studio (mock/fake per Open-Meteo e modello).
- Lo script di valutazione end-to-end gira invece contro LM Studio reale.

## Comandi
<!-- Da compilare quando lo stack è definito (test, ingestion, avvio). -->
