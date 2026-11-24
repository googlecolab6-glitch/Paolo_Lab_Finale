# Assistente di viaggio per il Friuli Venezia Giulia

Progetto finale ITS – **Traccia B**. Prototipo di assistente conversazionale che aiuta un turista a
organizzare una visita o un fine settimana in Friuli Venezia Giulia: luoghi da visitare, come arrivare,
dove mangiare, tenendo conto delle previsioni meteo e delle distanze tra località.

> **Stato:** in progettazione. Le sezioni marcate *(da completare)* verranno compilate man mano che il
> lavoro procede. Lo stato aggiornato è in [`docs/HANDOFF.md`](docs/HANDOFF.md).

Esempio di richiesta che il sistema deve gestire end-to-end:

> «Sabato prevedo di visitare Cividale del Friuli e una seconda località nelle vicinanze: quali soluzioni
> sono consigliate in caso di pioggia?»

## Come funziona (in breve)

- Un **modello linguistico locale** (LM Studio, API compatibile OpenAI) guida un **agente** che alterna
  chiamate al modello ed esecuzione di strumenti, con un numero massimo di iterazioni configurabile.
- Gli strumenti sono esposti da un **server MCP**:
  - `cerca_luoghi`: ricerca semantica (RAG) sulle pagine Wikivoyage, con filtri per località e sezione;
  - `meteo`: previsioni per località e data (Open-Meteo);
  - `distanza`: distanza **in linea d'aria** tra due località, calcolata dal codice;
  - `salva_itinerario`: *(estensione facoltativa)* salva un itinerario dopo conferma dell'utente.
- Ogni informazione tratta dai documenti è accompagnata dalla **fonte** (località, sezione, URL), e il
  codice verifica che le fonti mostrate provengano davvero dai risultati di `cerca_luoghi`.
- Un'**interfaccia web** mostra l'elenco delle conversazioni a sinistra e la conversazione aperta a destra;
  conversazioni e messaggi sono salvati in un database su volume Docker.

## Architettura *(proposta, in attesa di approvazione)*

```
 browser ──▶ web (interfaccia + API conversazioni + SQLite)
                │  contratto 2
                ▼
             agente (ciclo modello ↔ strumenti) ──▶ LM Studio (sull'host)
                │  MCP (Streamable HTTP)
                ▼
             mcp (cerca_luoghi, meteo, distanza) ──▶ indice vettoriale, Open-Meteo
```

## Organizzazione del gruppo

| Ruolo | Responsabilità |
|---|---|
| 1. Agente e strumenti | Ciclo dell'agente, istruzioni di sistema, server MCP, strumenti `meteo` e `distanza` |
| 2. Documenti e retrieval | Acquisizione da Wikivoyage, pulizia, chunking, metadati, embedding, indice, `cerca_luoghi` |
| 3. Applicazione | Interfaccia web, database delle conversazioni, integrazione con l'agente, Docker Compose |

## Prerequisiti *(da completare)*

- Docker con Docker Compose.
- [LM Studio](https://lmstudio.ai/) in esecuzione sull'host con il server locale attivo e un modello che
  supporta il tool calling (es. Qwen3 8B). È l'unico prerequisito esterno.

## Configurazione *(da completare)*

Copiare `.env.example` in `.env` e adattare i valori (indirizzo di LM Studio, nome del modello,
numero massimo di iterazioni dell'agente, ...).

## Ingestion dei documenti *(da completare)*

Comando separato che scarica (o rilegge dalla copia salvata, anche senza rete) le pagine Wikivoyage,
le ripulisce, le divide in chunk per sezione e le indicizza.

## Avvio *(da completare)*

```
docker compose up
```

## Test e valutazione *(da completare)*

- Test automatici senza rete né LM Studio.
- Script di valutazione end-to-end contro LM Studio reale; i risultati sono nel rapporto di valutazione.

## Località coperte

Pagine di Wikivoyage in italiano verificate tramite API il 2026-11-24 (esistono e hanno contenuto utile):

1. Aquileia
2. Cividale del Friuli
3. Duino
4. Forni di Sopra
5. Gemona del Friuli
6. Gorizia
7. Gradisca d'Isonzo
8. Grado (pagina «Grado (Italia)»)
9. Lignano Sabbiadoro
10. Marano Lagunare
11. Muggia
12. Palmanova
13. Pordenone
14. Sacile
15. San Daniele del Friuli
16. Sappada
17. Spilimbergo
18. Tarvisio
19. Tolmezzo
20. Trieste
21. Udine
22. Venzone

## Contratti tra i componenti *(da completare)*

Descritti in `docs/contratti.md` (con esempi JSON di successo ed errore) e riportati qui una volta approvati:

- **Contratto 1:** risultati di `cerca_luoghi` (documenti e retrieval → agente), con formato di errore unico
  per tutti gli strumenti.
- **Contratto 2:** messaggi tra interfaccia e agente.

## Fonti e licenza dei contenuti

I testi delle località provengono da **[Wikivoyage in italiano](https://it.wikivoyage.org/)** e sono
distribuiti con licenza **[Creative Commons Attribuzione – Condividi allo stesso modo (CC BY-SA)](https://creativecommons.org/licenses/by-sa/4.0/deed.it)**.
Ogni risposta che usa questi contenuti cita la pagina e la sezione di provenienza con il link.
Previsioni meteo e geocoding: [Open-Meteo](https://open-meteo.com/).
