# Handoff – stato del lavoro

> Documento di passaggio di consegne: chi riprende il lavoro (un membro del gruppo o una nuova
> sessione di Claude) parte da qui. Aggiornarlo a ogni passo importante.

**Ultimo aggiornamento:** 2026-11-24
**Fase attuale:** brainstorming, decisione 1 (stack) **proposta, in attesa di approvazione**.
**Codice scritto:** nessuno (per regola, nessun codice prima dell'approvazione del design doc).

## Documenti di riferimento
- `docs/traccia.md`: testo completo della traccia e delle istruzioni di lavoro.
- `CLAUDE.md`: regole di lavoro per Claude (bozza, da completare dal gruppo).
- `README.md`: README iniziale (da completare man mano).

## Flusso di lavoro concordato
1. **Brainstorming** (superpowers:brainstorming), una decisione alla volta, nell'ordine:
   1. Stack ← **siamo qui**
   2. Contratti tra componenti (contratto 1 `cerca_luoghi`, contratto 2 interfaccia ↔ agente), con esempi JSON, da approvare
   3. Database vettoriale + embedding; dump vs API Wikivoyage; date relative; verifica delle citazioni nel codice
2. **Design doc** in `docs/superpowers/specs/AAAA-MM-GG-<tema>-design.md`, da approvare.
3. **Piano** (superpowers:writing-plans): blocco comune → tre filoni paralleli (uno per ruolo, con finti
   componenti) → punti di integrazione.
4. **Esecuzione** con subagent-driven-development, TDD su ogni task, worktree, code review a fine blocco,
   verification-before-completion.

## Decisioni prese
| Decisione | Stato |
|---|---|
| Repository: `https://github.com/googlecolab6-glitch/Paolo_Lab_Finale`, branch `main` | ✅ fatto |
| Lingua: italiano per comunicazione e documenti; nomi strumenti come da traccia | ✅ dalla traccia |
| Stack (sotto) | ⏳ proposta, da approvare |

## Decisione 1 – Stack (PROPOSTA, non ancora approvata)

| Scelta | Proposta | Motivo |
|---|---|---|
| Linguaggio | Python 3.12 + FastAPI + Pydantic | Pydantic = modelli tipizzati dei contratti in un unico punto condiviso |
| Ciclo agente | Scritto a mano (~100 righe), no LangGraph | Spiegabile alla lavagna: `for` fino a `MAX_ITERAZIONI`, arresto quando il modello non chiede strumenti |
| Client modello | SDK `openai` con `base_url` di LM Studio da `.env` | API compatibile richiesta, una sola dipendenza |
| MCP | SDK ufficiale `mcp` (FastMCP), server in container separato, trasporto Streamable HTTP | Confine netto ruoli 1/2, si collega con compose. stdio scartato: obbligherebbe agente e server nello stesso container |
| Servizi | `web` (UI + API conversazioni) → `agente` (HTTP, contratto 2) → `mcp` (strumenti + indice) | Un container per ruolo; agente finto attivabile da `.env`; `web` gestisce l'agente non raggiungibile |
| Frontend | HTML + JS + CSS semplici serviti da FastAPI; Markdown con `marked` + `DOMPurify` copiati nel repo | Nessuna build. Streamlit scomodo per layout a due colonne e blocco invio; React eccessivo |
| DB conversazioni | SQLite su volume Docker, modulo `sqlite3` della libreria standard | Zero container in più, due tabelle |
| Streaming | No nel blocco base (estensione finale) | La verifica citazioni avviene a risposta completa; contratto 2 resta richiesta/risposta |

**Modelli LM Studio suggeriti** (buon tool calling): Qwen3 8B (prima scelta), Qwen2.5 7B Instruct,
Llama 3.1 8B Instruct; con GPU ≥ 16 GB: Qwen3 14B o Mistral Small.

**Se il tool calling è inaffidabile:** temperatura 0–0,2; descrizioni degli strumenti brevi e precise;
argomenti validati con Pydantic e, se invalidi, rimandati al modello come errore del contratto perché si
corregga; un solo nuovo tentativo poi risposta controllata; in ultima istanza cambiare modello
(lo script di valutazione serve anche a confrontarli).

## Ricognizione già fatta: pagine Wikivoyage (API, 2026-11-24)

Dimensione del wikitesto in byte. Tutte esistono con contenuto utile salvo dove indicato.

| Pagina | Byte | Note |
|---|---|---|
| Trieste | 50 960 | |
| Gorizia | 43 247 | |
| Udine | 42 703 | |
| Forni di Sopra | 32 637 | |
| Grado (Italia) | 30 094 | «Grado» è una disambiguazione: usare «Grado (Italia)» |
| Aquileia | 29 384 | |
| Tarvisio | 22 727 | |
| Tolmezzo | 22 207 | |
| Gemona del Friuli | 20 989 | |
| Muggia | 20 370 | |
| Sappada | 19 757 | |
| Lignano Sabbiadoro | 18 984 | |
| Gradisca d'Isonzo | 18 680 | |
| Venzone | 17 805 | |
| Marano Lagunare | 13 057 | |
| Palmanova | 11 938 | |
| Spilimbergo | 9 965 | |
| Sacile | 9 568 | |
| San Daniele del Friuli | 9 318 | |
| Duino | 9 180 | «Duino-Aurisina» non esiste: usare «Duino» |
| Pordenone | 8 742 | |
| Cividale del Friuli | 8 223 | La più corta, ma sufficiente |

Riserve verificate: Monfalcone (25 485), Maniago (28 895), Cervignano del Friuli (17 440),
Villa Santina (13 281), Carnia (11 547, è una regione, non una località).
Non esistono: Codroipo, Cormons, Duino Aurisina.

## Problemi aperti
- **Push su GitHub rifiutato (403):** git su questo PC è autenticato come `GiacomoCapone`, che non ha
  permessi di scrittura sul repo di `googlecolab6-glitch`. Soluzioni: aggiungere `GiacomoCapone` come
  collaboratore del repo, oppure fare il push con le credenziali del proprietario.
  Finché non è risolto, i commit restano solo in locale.

## Prossimo passo
1. Il gruppo approva (o modifica) lo stack.
2. Claude propone i due contratti con esempi JSON di successo ed errore.
