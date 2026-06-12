# Operazione Liberazione — Design Document

> Stato corrente: **v4** (file `operazione-liberazione-v4.html`)
> Ultimo aggiornamento: 5 giugno 2026

---

## 1. Concept

Gioco mobile stealth/puzzle a tema antispecista. Visuale dall'alto. Un gruppo ALF si infiltra in luoghi di sfruttamento animale (allevamento, capannone, zoo, laboratorio, circo, negozio esotici) e libera gli animali uno a uno senza farsi vedere dai sorveglianti.

Ispirazione meccanica: Farm Jam / Parking Jam (incastri direzionali) + stealth a cono di vista. Identità grafica: pixel art top-down, palette terrose, atmosfera notturna. Target finale: App Store e Play Store.

## 2. Stato attuale (v4)

Il gioco esiste come HTML singolo autocontenuto, con asset PNG incorporati in base64. Implementa **solo il livello allevamento**.

Fasi di gioco:
1. **Apri i cancelli** — 6 cancelli sul recinto, vanno toccati uno a uno per aprire i lucchetti.
2. **Libera le capre** — toccando una capra, l'attivista la raggiunge e la sblocca; la capra esce in linea retta nella sua direzione, attraversando il cancello assegnato (= il più vicino).

Quando tutte le capre sono libere → finale spray con slogan ALF a rotazione casuale.

### Meccaniche chiave

- **Sorvegliante** che pattuglia il perimetro del recinto in senso orario, con cono di vista giallo davanti a sé.
- **Cono di vista**: se una capra è nel cono (anche solo durante il tragitto verso il cancello) → SCOPERTO → ricomincia.
- **Collisione attivista/sorvegliante**: se durante uno spostamento l'attivista finisce addosso al sorvegliante → SCOPERTO → ricomincia.
- **Incastri**: una capra è bloccata se nel suo percorso c'è un'altra capra non ancora liberata.
- **Power-up bastone** (🏏, 2 cariche): armi il bastone e tocchi il sorvegliante per stordirlo 4.5s (perde il cono di vista, non rileva collisioni). Visivamente: stelline 💫 e sprite desaturato.

## 3. Parametri tecnici

### LEVEL config (allevamento)

```js
LEVEL = {
  cols: 6, rows: 6,
  density: 0.78,                          // % di celle occupate da animali
  pen: { x: .185, y: .334, w: .63, h: .327 },  // recinto in coord. relative al board
  gates: [
    { rx: .426, ry: .282, rw: .150, rh: .060, side: 'top'    },
    { rx: .423, ry: .635, rw: .151, rh: .052, side: 'bottom' },
    { rx: .167, ry: .398, rw: .034, rh: .084, side: 'left'   },
    { rx: .167, ry: .551, rw: .036, rh: .084, side: 'left'   },
    { rx: .806, ry: .363, rw: .036, rh: .082, side: 'right'  },
    { rx: .805, ry: .496, rw: .037, rh: .086, side: 'right'  },
  ],
  guard: { speed: .45, half: 26, range: 2.4 },  // velocità, mezzo-angolo cono in gradi, raggio cono in celle
}
```

### Geometria pattuglia

Il sorvegliante segue un rettangolo perimetrale espanso di `RO = cell * 0.35` rispetto al `pen`. Funzione `perimAt(s)` con `s ∈ [0,1)` restituisce `{x, y, nx, ny}` sul perimetro.

**Correzione applicata (importante):** sul segmento superiore la y viene corretta a `y0 - 5` per posizionare il sorvegliante nel corridoio sopra le gabbie, non sopra le gabbie stesse.

### Griglia animali

La griglia delle capre è centrata nel `pen` ma **traslata in alto di 14px** rispetto al centro geometrico, per allinearla visivamente alla parte dipinta del recinto.

### Soglia di collisione attivista/sorvegliante

`dist < cell * 0.88` → caught(). Controllo eseguito:
1. Ogni frame nel `loop()`, con la posizione tracciata in `S.alfPos`.
2. Anche prima del movimento in `tryFreeAnimal()`, per intercettare il caso in cui il target sia già sopra al sorvegliante.

Il controllo è inibito quando `ctrl.stun > 0` (sorvegliante stordito).

## 4. Asset

Cinque PNG, pixel art top-down, sfondo trasparente o pieno:

| File | Soggetto |
|------|----------|
| `ALF.png` | Attivista col passamontagna |
| `farmer1.png` | Allevatore (sorvegliante del livello allevamento) |
| `goat1.png` | Capra (animale del livello) |
| `allevamto1-chiuso.png` | Fondale recinto con cancelli chiusi |
| `allevamto1-aperto.png` | Fondale recinto con cancelli aperti |

Sono incorporati nell'HTML come `data:image/png;base64,...`.

Il fondale fa lo swap automatico chiuso → aperto quando `gatesOpen === gates.length`.

## 5. Stile visivo

- **Font display**: Anton (slogan, HUD, titoli)
- **Font UI**: Sora
- **Palette**: nero/marrone notturno (#0c0f0b, #1a1f18), oro caldo (#d4a942), rosso allerta (#b03020), verde ok (#2e7d4a)
- **Cono di vista**: gradiente radiale giallo-arancio
- **Glow torcia**: drop-shadow gialla sul corpo dell'allevatore (è lui stesso la sorgente luminosa, non c'è una torcia separata)
- **Finale spray**: testo rosso font Anton con bordi morbidi, colature animate `@keyframes drip`, puntini schizzati

### Slogan ALF (rotazione casuale al finale)

```
LIBERAZIONE ANIMALE ORA
NESSUNA GABBIA È GIUSTA
LIBERI TUTTI
ALF · NESSUN PADRONE
LA LORO VITA NON È MERCE
```

## 6. Architettura del codice

File singolo HTML. Sezioni JS in ordine:

1. **Costanti**: `SLOGANS`, `LEVEL`, riferimenti DOM
2. **Stato globale**: `S` (state object), flags (`locked`, `phase`, `gameOver`, ecc.)
3. **Helpers**: `mk()`, `setPos()`, `setCone()`, `cellX/Y/ctrX/Y`
4. **`perimAt(s)`**: calcola posizione lungo il perimetro [contiene fix +5px sul top]
5. **`genAnimals()`**: distribuisce capre e direzioni
6. **`build()`**: monta tutto il livello (board, celle, cancelli, attivista, animali, cono, sorvegliante)
7. **`loop(now)`**: game loop con requestAnimationFrame [contiene collision check attivista/sorvegliante]
8. **`markCone()`**: marca celle dentro il cono di vista
9. **Logica**: `tryOpenGate`, `tryFreeAnimal`, `pathOf`, `isBlocked`, `bumpAnim`, `exitAnimal`
10. **`caught()`**: flash rosso + shake + restart
11. **`endWin()`**: finale spray con slogan
12. **Power-up**: `clubArm`, `clubBonk`
13. **FX**: `puff`, `toast`
14. **UI**: `startGame`, resize handler

## 7. Cose decise e da NON cambiare senza motivo

- Visuale dall'alto, niente parallasse o pseudo-3D
- Stealth con cono di vista, non barra di sospetto (l'avevo proposto, Antonio l'ha rifiutato: scoperto = ricomincia)
- Pixel art (gli asset PNG esistenti vanno preservati)
- File singolo HTML autosufficiente come build di sviluppo (porting in app nativa dopo)
- Tono ALF militante, niente edulcorazione

## 8. Open issues / TODO

### Da fare prossimamente

- [ ] Verificare proporzioni reali dei PNG nel gioco (capra forse troppo grande, attivista a riposo da riposizionare)
- [ ] Animali successivi: maiali, mucche, polli (richiedono PNG dedicati)
- [ ] Livello 2: **capannone industriale** (operaio veloce, polli, gabbie strette)
- [ ] Livello 3: **zoo** (auto-liberazione: scimmia apre lucchetti, ultimo animale colpisce il guardiano)
- [ ] Livello 4: **laboratorio** (medico imprevedibile + telecamera con cono che oscilla)
- [ ] Schermata selezione personaggio (vari membri ALF)
- [ ] Audio: musica ambient + effetti
- [ ] Salvataggio progresso (localStorage)
- [ ] Build verso app nativa (Capacitor o React Native?)

### Bug noti

- Il fondale PNG è dimensionato come sfondo del board (`width:100%; height:100%`), quindi può stiracchiarsi su schermi con proporzioni diverse da 9:16. Da verificare con dispositivi reali.

## 9. Convenzioni di lavoro

Quando si riprende il lavoro in una nuova chat:

1. Leggere questo design.md prima di proporre modifiche
2. Leggere l'HTML più recente caricato nel Project
3. Verificare se l'utente ha caricato nuovi asset PNG (asset prevalgono su SVG segnaposto)
4. Non riscrivere parti che funzionano: usare modifiche chirurgiche (str_replace) quando possibile
5. Quando l'utente segnala un problema visivo, chiedere uno screenshot prima di tentare aggiustamenti pixel-perfect alla cieca
