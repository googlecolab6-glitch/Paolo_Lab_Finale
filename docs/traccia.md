# Traccia del progetto (testo originale fornito dal gruppo)

> Copia di riferimento della traccia e delle istruzioni di lavoro, così chi riprende il lavoro
> (persona o sessione di Claude) ha il testo completo nel repository.

# Progetto finale ITS – Traccia B: Assistente di viaggio per il Friuli Venezia Giulia

Usa il flusso superpowers dall'inizio alla fine: parti con superpowers:brainstorming, poi superpowers:writing-plans, poi l'esecuzione (preferisco superpowers:subagent-driven-development) con superpowers:test-driven-development su ogni task, superpowers:using-git-worktrees per isolare il lavoro, superpowers:systematic-debugging per ogni bug, superpowers:requesting-code-review alla fine di ogni blocco e superpowers:verification-before-completion prima di dichiarare qualsiasi cosa finita. Non scrivere codice prima che io abbia approvato il design doc.

Comunica con me in italiano. README, design doc, piano e rapporto di valutazione vanno scritti in italiano. I nomi degli strumenti MCP devono essere esattamente quelli della traccia (cerca_luoghi, meteo, distanza, salva_itinerario).

## Contesto

È il progetto finale di un corso ITS, svolto da un gruppo di 3 persone, consegna entro il 16 dicembre tramite repository Git. È un PROTOTIPO: circoscritto e funzionante, non un prodotto. Verrà presentato alla lavagna e ogni membro del gruppo deve saper spiegare ogni parte del sistema, quindi il codice deve essere leggibile, poco "magico", con poche dipendenze e commenti dove le scelte non sono ovvie. Preferisci sempre la soluzione più semplice che soddisfa i requisiti.

Il lavoro è diviso in tre ruoli, e la struttura del repository deve rispecchiarli con confini netti:
1. Agente e strumenti: grafo/ciclo dell'agente, istruzioni di sistema, server MCP, strumenti meteo e distanza.
2. Documenti e retrieval: acquisizione da Wikivoyage, pulizia, chunking, metadati, embedding, indicizzazione, strumento cerca_luoghi.
3. Applicazione: interfaccia web, database delle conversazioni, integrazione con l'agente, Docker Compose.

Vogliamo lavorare in parallelo, ognuno sul proprio componente: per questo i contratti tra componenti si fissano per primi (vedi la sezione "Contratti tra componenti").

## Obiettivo dell'agente (traccia B)

Supportare un turista nell'organizzare una visita o un fine settimana in FVG:
- indicazioni su luoghi da visitare, come arrivare, dove mangiare;
- tenere conto delle previsioni meteo e delle distanze tra località.

Richiesta di esempio da far funzionare end-to-end: «Sabato prevedo di visitare Cividale del Friuli e una seconda località nelle vicinanze: quali soluzioni sono consigliate in caso di pioggia?»

## Requisiti comuni (obbligatori)

1. Modello locale: LM Studio, endpoint compatibile con l'API OpenAI. Dai container va raggiunto l'host (es. host.docker.internal), configurabile via .env. Il modello deve supportare il tool calling: proponimi modelli adatti e cosa fare se il tool calling del modello è inaffidabile.
2. Agente: ciclo che alterna chiamate al modello ed esecuzione degli strumenti, con condizione di arresto esplicita e numero massimo di iterazioni esplicito e configurabile; al raggiungimento del limite l'agente risponde in modo controllato, non si blocca.
3. Strumenti MCP: esposti da almeno un server MCP. Per ogni strumento: nome, descrizione, parametri tipizzati, formato del risultato e condizioni di errore, tutti documentati.
4. RAG: cerca_luoghi è uno strumento dell'agente. Documenti divisi in chunk con metadati, indicizzati in un database vettoriale, interrogabili con filtri sui metadati.
5. Citazione fonti: ogni informazione tratta dai documenti indica la fonte (pagina/località e sezione, con URL). Le fonti citate DEVONO essere presenti nei risultati restituiti da cerca_luoghi in quel turno: voglio una verifica nel codice, non solo un'istruzione al modello (es. le fonti mostrate all'utente si costruiscono dai risultati reali degli strumenti, e le citazioni non riscontrabili vengono scartate o segnalate).
6. Controlli: l'agente dichiara l'assenza di evidenze invece di inventare; gestisce esplicitamente errori e risultati vuoti degli strumenti; i calcoli deterministici sono fatti dal codice, non dal modello.
7. Interfaccia web conforme alla sezione "Interfaccia web" sotto.
8. Riproducibilità: tutto (interfaccia, agente, server MCP, database vettoriale, database conversazioni) si avvia con `docker compose up` su una macchina senza configurazioni preesistenti, seguendo il README. LM Studio gira sull'host ed è l'unico prerequisito esterno.

Fuori perimetro: autenticazione, utenti e profili, autorizzazioni, pubblicazione su infrastrutture pubbliche.

## Corpus documentale

- Wikivoyage in italiano, pagine di località del Friuli Venezia Giulia, licenza CC BY-SA: la fonte va citata (anche nell'interfaccia e nel README).
- Due vie possibili: dump completo (dumps.wikimedia.org/itwikivoyage/latest, file itwikivoyage-latest-pages-articles.xml.bz2, circa 51 MB) da filtrare, oppure API MediaWiki (it.wikivoyage.org/w/api.php) scaricando solo le pagine della regione. Proponimi quale e perché.
- Almeno 20 località, elencate nel README. Verifica tramite API che ogni pagina esista davvero e abbia contenuto utile prima di sceglierla (candidati da controllare: Trieste, Udine, Pordenone, Gorizia, Cividale del Friuli, Aquileia, Grado, Lignano Sabbiadoro, Palmanova, Spilimbergo, San Daniele del Friuli, Venzone, Gemona del Friuli, Tarvisio, Sappada, Duino-Aurisina, Muggia, Sacile, Tolmezzo, Gradisca d'Isonzo, Marano Lagunare, Forni di Sopra).
- Metadati minimi per chunk: località, sezione della pagina («Da vedere», «Come arrivare», «Dove mangiare», ecc.), URL della pagina. La ricerca deve consentire il filtro per località e per sezione.
- Il chunking deve rispettare la struttura a sezioni del wikitesto; il markup wiki (template, link, tabelle) va ripulito in modo che il testo resti leggibile.
- Per la riproducibilità, salva una copia dei dati grezzi scaricati con la data di download, così l'ingestion può essere rieseguita anche senza rete. La procedura di ingestion è un comando separato e documentato nel README.

## Servizi esterni

Open-Meteo, gratuito, senza chiave: previsioni (api.open-meteo.com/v1/forecast) e geocoding (geocoding-api.open-meteo.com/v1/search). Il geocoding va vincolato all'Italia e, dove possibile, al FVG, per evitare omonimi in altre regioni.

## Strumenti minimi

cerca_luoghi
- Parametri: query (testo); localita e sezione (facoltativi, filtri); k (numero risultati).
- Risultato: elenco di passaggi con testo, località, sezione, URL e punteggio.
- Errori da gestire: nessun risultato; località non presente nel corpus (risposta esplicita, non un elenco vuoto silenzioso).

meteo
- Parametri: localita, data.
- Risultato per la data indicata: temperatura minima e massima, probabilità di precipitazione, descrizione sintetica (da weather code a testo in italiano).
- Errori da gestire: località non trovata dal geocoding; data passata o oltre l'orizzonte di previsione; servizio non raggiungibile o in timeout (con timeout esplicito nel codice).

distanza
- Parametri: da, a (nomi di località).
- Risultato: distanza in linea d'aria in km, calcolata dal codice dalle coordinate (es. formula dell'emisenoverso). Il risultato deve dichiararsi "in linea d'aria", mai come percorso stradale.
- Errori da gestire: località non individuata.

Estensione facoltativa (da valutare in brainstorming, solo se il resto è finito): salva_itinerario (titolo, tappe, data), che salva l'itinerario nel database dell'applicazione solo dopo conferma esplicita dell'utente.

Attenzione alle date relative: richieste come «sabato» o «domani» non devono essere risolte con aritmetica fatta dal modello. Proponimi come renderle deterministiche (data e giorno della settimana correnti nel prompt di sistema, risoluzione nel codice dello strumento meteo, o altro), con fuso Europe/Rome.

## Controlli specifici della traccia

- Se il servizio meteo non è disponibile, l'assistente lo segnala e prosegue con le informazioni disponibili.
- Orari, prezzi e recapiti si riportano solo se presenti nei passaggi recuperati, con l'avvertenza che la fonte può non essere aggiornata.
- Le località non comprese nel corpus sono dichiarate come non coperte.
- Richieste estranee all'ambito (non turismo in FVG) ricevono un rifiuto motivato, senza risposta nel merito.
- Richieste ambigue ricevono una richiesta di chiarimento.

## Interfaccia web (tecnologia a scelta, proponimi le opzioni)

Una pagina divisa in due aree: a sinistra l'elenco delle conversazioni, a destra la conversazione selezionata.
1. Pulsante «Nuova conversazione» che avvia una conversazione vuota.
2. Elenco conversazioni dalla più recente, ognuna con un titolo (es. inizio del primo messaggio); selezionandone una si vede tutto il contenuto; quella aperta è evidenziata.
3. Messaggi utente distinguibili da quelli dell'agente (allineamento o colore), in ordine cronologico.
4. Risposte dell'agente in Markdown (elenchi, grassetto, tabelle). Durante l'elaborazione c'è un indicatore di attesa e non si possono inviare altri messaggi.
5. Fonti riportate sotto la risposta che le cita: documento, sezione e link.
6. (Facoltativo, ma lo vorrei) pannello espandibile «Strumenti utilizzati» con argomenti, esito e durata di ogni chiamata.
7. Campo di testo con pulsante Invia; il tasto Invio invia il messaggio.

Persistenza e comportamento:
- Conversazioni e messaggi salvati in un database su volume Docker, disponibili dopo il riavvio dei container.
- Alla ripresa di una conversazione, l'agente riceve la cronologia della sola conversazione selezionata.
- Se il modello o un servizio non risponde, l'interfaccia mostra un messaggio di errore nella conversazione e resta utilizzabile. Con LM Studio spento il sistema non deve bloccarsi.

Estensioni facoltative, solo a fine lavoro: streaming della risposta; rinomina ed eliminazione conversazioni.

## Contratti tra componenti: da fissare per primi

Nel brainstorming, subito dopo la scelta dello stack, la prima cosa da definire e farmi approvare sono i due contratti tra componenti, prima di ogni altra decisione di dettaglio.

Contratto 1, risultati di cerca_luoghi (Documenti e retrieval → Agente):
- formato di ogni passaggio: id del chunk, testo, località, sezione, URL, punteggio (con il significato del punteggio: più alto = migliore o il contrario);
- formato dei casi non riusciti: nessun risultato, località non presente nel corpus, errore interno. Proponimi un formato di errore unico, da usare anche per meteo e distanza, così l'agente li gestisce tutti allo stesso modo.

Contratto 2, messaggi tra interfaccia e agente (Applicazione ↔ Agente):
- richiesta: id della conversazione, messaggio dell'utente, come arriva la cronologia;
- risposta: testo in Markdown, elenco delle fonti (documento, sezione, URL), elenco degli strumenti usati (nome, argomenti, esito, durata), eventuale errore;
- se scegliamo lo streaming, anche il formato degli eventi.

Per ogni contratto voglio:
- un documento nel repository (es. docs/contratti.md) con la descrizione dei campi ed esempi JSON reali, sia di successo sia di errore, da riportare anche nel README come richiede la traccia;
- la definizione nel codice in un unico punto condiviso (es. modelli tipizzati), che tutti i componenti importano invece di ridefinirla;
- dei finti componenti basati sugli esempi (un cerca_luoghi finto che restituisce dati di esempio, un agente finto che risponde con una risposta d'esempio), così ognuno può sviluppare e testare il proprio pezzo senza aspettare gli altri;
- test di contratto che verificano che ogni componente reale produca e accetti esattamente quel formato.

Una volta approvati, i contratti cambiano solo con una modifica esplicita, che mi segnali e che aggiorna documento, definizione condivisa, esempi e test insieme. La verifica delle citazioni (requisito comune 5) si appoggia sull'id del chunk definito nel contratto 1.

## Test

TDD su tutto il codice deterministico. I test automatici non devono dipendere da rete né da LM Studio: usa mock/fake per Open-Meteo e per il modello. Copertura minima che mi aspetto:
- distanza: casi noti (es. Trieste–Udine con tolleranza), stessa località = 0, località sconosciuta.
- meteo: parsing della risposta, data passata, data oltre orizzonte, località non trovata, timeout ed errore HTTP, traduzione dei weather code.
- ingestion: pulizia del wikitesto, divisione in sezioni, metadati corretti su ogni chunk.
- cerca_luoghi: filtri per località e sezione, località fuori corpus, nessun risultato, formato del risultato conforme al contratto 1.
- agente: con un modello finto che segue uno script, verifica la sequenza di chiamate agli strumenti, il limite di iterazioni, la gestione di errori degli strumenti e la verifica delle citazioni.
- backend: API delle conversazioni, conformità al contratto 2, isolamento tra conversazioni, persistenza, gestione del modello non raggiungibile.

In più serve uno script di valutazione end-to-end, eseguito contro LM Studio reale, che registri per ogni caso richiesta, strumenti invocati, risposta ed esito, con almeno un caso per ciascuna tipologia:
1. quesito sui documenti → risposta corretta con fonte presente nei risultati della ricerca;
2. quesito sul servizio esterno → scelta dello strumento corretto con argomenti corretti;
3. quesito combinato → più strumenti in sequenza e sintesi (usa l'esempio di Cividale);
4. quesito senza evidenze nei documenti → dichiarazione di assenza;
5. errore o risultato vuoto di uno strumento → segnalazione dell'errore senza invenzioni;
6. richiesta ambigua → richiesta di chiarimento;
7. richiesta fuori ambito → rifiuto motivato.
I risultati alimentano il rapporto di valutazione.

## Consegna

Il repository deve contenere:
- codice di tutti i componenti e docker-compose.yml;
- script di ingestion;
- README con prerequisiti, configurazione (.env.example), comandi di ingestion e avvio, elenco delle località, attribuzione CC BY-SA, contratti tra i componenti;
- rapporto di valutazione sintetico con la tabella dei casi di prova e i limiti noti, utile anche per la presentazione.

## Cosa voglio dal brainstorming

Il materiale sopra copre già il "cosa": non serve chiedermelo di nuovo. Concentrati sul "come" e sulle decisioni aperte, una alla volta, con la tua raccomandazione motivata per ciascuna, tenendo conto che deve essere spiegabile alla lavagna da studenti.

Ordine delle decisioni:
1. Stack: linguaggio e framework; ciclo dell'agente scritto a mano oppure LangGraph (o simili); come l'agente parla al server MCP (trasporto, processo separato o no); tecnologia del frontend; database delle conversazioni; streaming sì o no.
2. Subito dopo, i due contratti tra componenti (sezione dedicata sopra), con esempi, da farmi approvare prima di proseguire.
3. Poi il resto: database vettoriale e modello di embedding (locale nel container o via LM Studio); dump vs API per Wikivoyage; gestione delle date relative; come garantire nel codice che le citazioni provengano dai risultati.

Poi scrivi il design doc e, dopo la mia approvazione, il piano. Il piano deve avere task piccoli e testabili, organizzati così: un primo blocco comune (contratti, struttura del repository, docker-compose di base); poi tre filoni paralleli, uno per ruolo, sviluppabili su branch o worktree separati usando i finti componenti; infine i punti di integrazione in cui si sostituiscono i finti componenti con quelli reali. L'obiettivo è che il sistema integrato funzioni il prima possibile, anche in forma minima, e poi cresca.
