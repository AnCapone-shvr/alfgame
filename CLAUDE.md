# CLAUDE.md — Operazione Liberazione

## Cos'è questo progetto

Gioco mobile stealth/puzzle a tema antispecista. Visuale top-down, pixel art. Un gruppo ALF si infiltra in luoghi di sfruttamento animale e libera gli animali senza farsi scoprire dai sorveglianti. Target: App Store e Play Store.

## File principali

- `index.html` — il gioco (file singolo autocontenuto, asset PNG in base64)
- `design.md` — documento tecnico con parametri, decisioni, TODO
- `assets/` — PNG sorgente degli sprite e fondali

## Come lavorare

1. Leggi `design.md` prima di qualsiasi modifica per capire lo stato attuale.
2. Modifica `index.html` con edit chirurgici. Non riscrivere l'intero file.
3. I parametri di gioco (coordinate recinto, cancelli, velocità sorvegliante) sono nell'oggetto `LEVEL` in cima al JS.
4. Gli asset PNG sono incorporati in base64 dentro `index.html`. Se l'utente fornisce nuovi PNG, convertili in base64 e sostituisci le variabili corrispondenti (`A_ALF`, `A_FARMER`, `A_GOAT`, `A_BG_CLOSED`, `A_BG_OPEN`).
5. Dopo modifiche significative, aggiorna anche `design.md`.

## Convenzioni

- Vanilla JS, niente framework. CSS variables per la palette.
- Mobile-first: board max 420px, ratio 9:16.
- `image-rendering: pixelated` su tutti gli `<img>` di sprite.
- Font: Anton (display), Sora (UI) da Google Fonts.
- L'utente (Antonio) non è uno sviluppatore. È grafico. Comunica in italiano.

## Test

Aprire `index.html` nel browser. Nessun build step necessario.
Per live reload: `npx live-server --port=8080 --entry-file=index.html`

## Cose da NON fare

- Non proporre di sostituire la meccanica "scoperto = ricomincia" con una barra di sospetto.
- Non usare struttura retorica "Non è... ma è".
- Non edulcorare i testi antispecisti: tono militante, critica ai sistemi, niente pietismo.
- Non aggiungere librerie esterne senza motivo.
