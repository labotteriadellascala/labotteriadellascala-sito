# CLAUDE.md — Sito vetrina La Botteria della Scala / Steel Head

Istruzioni di progetto per Claude Code. Leggile a inizio di ogni sessione.
Titolare: Andrea Canovi — Steel Head, Isola della Scala (VR), P.IVA 04775640230.
**Tutto (testi, commenti, comunicazioni) va scritto in italiano.**

---

## 1. Cos'è questo repo

Sito vetrina one-page + pagine galleria di mobili bar a forma di botte, fatti a mano su misura. Serve a raccogliere richieste di rendering 3D gratuito (lead).

- Repo: `labotteriadellascala/labotteriadellascala-sito` (**pubblico** — necessario per GitHub Pages su piano Free), branch **`main`**.
- Online su **https://www.labotteriadellascala.it** tramite **GitHub Pages** (deploy automatico a ogni push su `main`).
- DNS su Hostinger (già configurato, non toccare).

**NON toccare in questo repo / progetto:**
- Il gestionale **Botti Manager** — è un altro repo (`botti-manager`), altro sito (`gestionale.labotteriadellascala.it`). Non c'entra niente con questo lavoro.

---

## 2. Deploy e REGOLA D'ORO

**Ogni push su `main` manda il sito online SUBITO** (GitHub Pages ripubblica in 1-3 minuti). Non esiste ambiente di test.

> **Regola d'oro: non fare MAI `git push` senza l'OK esplicito di Andrea.**
> Prima mostra sempre le modifiche (diff o anteprima), spiega cosa cambia, e aspetta conferma. Solo dopo il push.

Dopo un push, per verificare online serve un **hard refresh** (Ctrl+Shift+R): cache di GitHub Pages + CDN possono mostrare la versione vecchia per qualche minuto.

Preferire **un solo commit per sessione** (batch delle modifiche), non uno per micro-cambiamento.

---

## 3. Struttura dei file

Ogni pagina è un **singolo file HTML** con CSS e JS inline (nessun build step, nessun framework).

- `index.html` — homepage one-page (hero, "cos'è", processo/Klarna, configuratore con slider, recensioni in home, garanzia, chi siamo, form contatti).
- `recensioni.html` — galleria di ~40 screenshot di messaggi clienti (masonry). Immagini `rec-01.webp` … `rec-40.webp`.
- `ispirazioni.html` — galleria di foto clienti a casa loro (masonry). Immagini `ispirazione-01.webp` … `ispirazione-34.webp`.
- `grazie.html` — pagina di conversione dopo l'invio del form (spara evento `Lead` / `generate_lead`).
- `img/` — tutte le immagini.

Struttura di un elemento galleria (identica in recensioni e ispirazioni):
```html
<figure class="mitem"><img src="img/ispirazione-NN.webp" width="W" height="H" alt="..." loading="lazy" decoding="async"></figure>
```

---

## 4. IMMAGINI — regole imparate a caro prezzo (leggere sempre)

Formato: **tutto in WebP**, TRANNE:
- `hero-social.jpg` resta **JPG** (è l'anteprima social / og:image).
- I loghi `logo-colore.png` e `logo-bianco.png` restano **PNG** (trasparenza).
- `favicon.svg` è SVG.

Regole per aggiungere/gestire foto:
1. **Ridurre sempre** le foto pesanti: lato lungo **max 1600px**, peso **< ~250KB**, WebP qualità ~82. Le foto dei telefoni sono da diversi MB: caricarle grezze rallenta il sito.
2. **Mettere sempre `width` e `height`** sull'`<img>`, con le dimensioni reali del file, per evitare il salto del layout (CLS).
3. **Nomi progressivi**: nuove ispirazioni → `ispirazione-35.webp` in poi; nuove recensioni → `rec-41.webp` in poi.
4. **Controllare i DOPPIONI prima di aggiungere.** Confronto *percettivo* (non solo file identici): una stessa foto ricompressa sembra un file diverso ma è un doppione. Confrontare i candidati sia tra loro sia contro tutte le foto già presenti; scartare i doppioni.

⚠️ **L'errore da NON rifare (ci è già successo):**
`index.html` referenzia le immagini di hero/dettaglio/prodotti in **.webp** (es. `hero-desktop.webp`, `dettaglio-*.webp`, `intera-refrigerata.webp` ecc.), **non** in `.jpg`.
**Prima di committare una modifica all'HTML, verificare che TUTTE le immagini referenziate esistano davvero nel repo.** Se si aggiunge/rinomina un `<img>`, il file corrispondente deve essere presente in `img/`, altrimenti online compaiono riquadri vuoti. Non dare per scontato che un file ci sia: controllarlo.

---

## 5. Menu e navigazione

Il menu è **ridotto e identico su tutte le pagine**, 3 voci:
1. **Recensioni** → `recensioni.html`
2. **Ispirazioni** → `ispirazioni.html`
3. **Rendering gratuito** (pulsante pieno) → il form contatti

Dettaglio link "Rendering gratuito":
- Su `index.html` → `#contatti` (scroll interno).
- Sulle pagine galleria → `index.html#contatti` (torna alla home e scrolla al form).

Sulla homepage, **sopra il form** ci sono due pulsanti che rimandano alle gallerie:
- "Vedi tutti i messaggi dei clienti →" → `recensioni.html`
- "Guarda le botti nelle case dei clienti →" → `ispirazioni.html`
(Devono stare **sopra il form**, su sfondo chiaro. NON dentro il riquadro Garanzia bordeaux: lì il testo bordeaux su sfondo bordeaux diventa invisibile.)

Se si aggiungono pagine, aggiornare il menu in **tutte e tre** le pagine per tenerlo coerente.

---

## 6. Design / identità visiva

- Palette editoriale avorio/beige/marrone. Accento **bordeaux**: `--gold:#7d2a35`, `--gold-warm:#9c3a47`. Sfondo chiaro `--plaster:#faf7f0`.
- Font **Geist**.
- Firma del brand: **una pianta in ogni scatto** (per le foto di prodotto nostre).
- Attenzione ai **contrasti di colore**: testo bordeaux va su sfondo chiaro, non su bordeaux (vedi errore §5).

---

## 7. Tracciamento, form, privacy

- Form gestito da **FormSubmit** → invia mail a `info@labotteriadellascala.it`. Ha un honeypot `azienda_extra` (campo trappola anti-bot, lasciare com'è). Redirect a `grazie.html`.
- Il lead viene scritto anche nel gestionale via **RPC Supabase `inserisci_lead_da_sito`**.
  ⚠️ **NON eseguire / non riscrivere lo SQL di questa RPC.** L'implementazione in produzione è più sofisticata di qualsiasi bozza: gestisce rate limiting, dedup, classificazione UTM. Lasciarla stare.
- **Meta Pixel** `3500501163531510` e **GA4** `G-RD4JSPQP91`: partono **solo dopo "Accetta"** sul banner cookie (consenso), revocabili da "Gestisci cookie" nel footer. Non attivarli di default.
- In footer: P.IVA sì. **CF e codice SDI NO** (scelta di privacy).
- UTM: il form cattura `fonte_utm`/`mezzo_utm`/`campagna_utm` da campi nascosti.

---

## 8. Metodo di lavoro

- Prima di consegnare/committare modifiche all'HTML: **`node --check`** su ogni blocco `<script>` inline. Se possibile, un controllo di render (immagini che caricano, niente errori JS).
- Un solo commit per sessione, messaggio chiaro in italiano.
- Trattandosi di file HTML singoli monolitici: attenzione a **collisioni di classi CSS** e a riferimenti tra blocchi. Preferire nomi di classe specifici.
- Non introdurre framework o build step: il sito deve restare a file statici serviti da GitHub Pages.

---

## 9. Questioni aperte (validazione non tecnica, non decidere da solo)

- Testo garanzia "15 giorni reso": in attesa di conferma del commercialista.
- Klarna ("rate senza interessi"): verificare che sia davvero attivabile con link di pagamento.
- Queste voci vanno **segnalate ad Andrea**, non modificate di iniziativa.

---

*Se qualcosa in queste istruzioni contrasta con una richiesta di Andrea, vince la richiesta di Andrea — ma faglielo notare prima di procedere.*
