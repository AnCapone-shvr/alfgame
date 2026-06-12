# Operazione Liberazione — Design Document

> Stato corrente: **v4** (file `index.html`)
> Ultimo aggiornamento: 7 giugno 2026

---

## 1. Concept

Gioco mobile stealth/puzzle a tema antispecista. Visuale dall'alto, pixel art. Un gruppo ALF si infiltra in luoghi di sfruttamento animale e libera gli animali uno a uno senza farsi scoprire dai sorveglianti. Target: App Store e Play Store.

Ispirazione meccanica: Farm Jam / Parking Jam (incastri direzionali) + stealth a cono di vista. ~100 livelli totali, suddivisi in 5-6 ambientazioni da ~20 livelli ciascuna.

## 2. Struttura livelli

### Principio: poche ambientazioni, molti livelli

Ogni **ambientazione** richiede un set di asset dedicato (fondale chiuso/aperto, sprite animale, sprite sorvegliante). Ogni ambientazione ha **~20 livelli** che variano solo i parametri di gioco — stesso fondale, stessa grafica.

### Ambientazioni previste

| # | Ambientazione | Animale | Sorvegliante | Meccanica speciale |
|---|---------------|---------|--------------|-------------------|
| 1 | Allevamento | Capre | Allevatore | — (base) |
| 2 | Capannone industriale | Polli | Operaio | Velocità doppia sorvegliante |
| 3 | Stalla intensiva | Mucche | Fattore | Animali lenti, griglia più larga |
| 4 | Zoo | Misti (scimmie, elefanti) | Guardiano | Auto-liberazione: scimmia apre lucchetti |
| 5 | Laboratorio | Conigli / topi | Ricercatore | Telecamere fisse con cono oscillante |
| 6 | Circo | Elefanti / tigri | Domatore | Animali helper: elefante sfonda cancello |

### Parametri che scalano tra un livello e l'altro (stessa ambientazione)

Ogni livello è un oggetto config:

```
{
  id: "allevamento-07",
  env: "allevamento",           // quale set di asset usare
  cols: 6, rows: 6,             // dimensioni griglia (può crescere)
  density: 0.78,                // quante celle occupate
  guard: {
    speed: 0.45,                // velocità pattuglia (cresce)
    half: 26,                   // mezzo-angolo cono in gradi (cresce)
    range: 2.4,                 // raggio cono in celle (cresce)
    unpredictable: false,       // inversione random di marcia
    pauseChance: 0,             // prob. di fermarsi a guardare (0-1)
  },
  guards: 1,                    // numero di sorveglianti (1 → 2 → 3)
  cameras: [],                  // telecamere fisse [{x,y,angle,sweep}]
  clubCharges: 2,               // cariche bastone (scende nei livelli alti)
  animalBehavior: "passive",    // "passive" | "noisy" | "self-free" | "helper"
  timeLimit: 0,                 // 0 = no limite; >0 = secondi
  stars: { three: 45, two: 90 } // soglie tempo per stelle
}
```

### Progressione difficoltà per ambientazione (~20 livelli)

Livelli 1-5 (tutorial): griglia piccola, pochi animali, sorvegliante lento, cono stretto. Introduce una meccanica alla volta.

Livelli 6-10 (media): griglia 6x6, densità alta, sorvegliante più veloce, cono più largo, incastri complessi.

Livelli 11-15 (difficile): secondo sorvegliante o prima telecamera fissa, meno cariche bastone (1 sola), sorvegliante imprevedibile.

Livelli 16-20 (estremo): 2 sorveglianti + telecamere, zero bastone, animali rumorosi, time limit opzionale.

### Meccaniche animali speciali

**Rumorosi** — quando liberati, il sorvegliante si gira verso il suono per 1.5s. Rischio e opportunità tattica.

**Auto-liberazione** — certi animali (scimmie) aprono un cancello da soli. Risparmia un viaggio dell'attivista.

**Helper** — animale liberato aiuta: elefante sfonda cancello, topo disattiva telecamera per 3s.

**Noisy** — polli scappano in direzioni casuali, creando caos.

## 3. Stato attuale (v4)

File HTML singolo autocontenuto con asset PNG in base64. Implementa solo il livello allevamento, senza sistema di selezione livelli.

Fasi di gioco attuali:
1. Apri i cancelli (6 cancelli, tocca per aprire)
2. Libera le capre (tocca animale → attivista lo raggiunge → capra esce in linea retta)
3. Finale spray con slogan ALF

### Meccaniche implementate

- Sorvegliante pattuglia perimetrale con cono di vista
- Collisione attivista/sorvegliante → scoperto → ricomincia
- Celle illuminate nel cono → animali non liberabili
- Incastri direzionali
- Power-up bastone (2 cariche, stordisce 4.5s)
- Swap fondale chiuso/aperto

### Da implementare

- Sistema selezione livelli / schermata mondo
- Array configurazioni livello
- Progressione / salvataggio (localStorage)
- Stelle (1-3) per livello
- Sorveglianti multipli
- Telecamere fisse
- Sorvegliante imprevedibile
- Comportamenti animali speciali
- Schermata selezione personaggio
- Audio

## 4. Parametri tecnici (livello allevamento)

### LEVEL config attuale

```js
LEVEL = {
  cols: 6, rows: 6,
  density: 0.78,
  pen: { x: .185, y: .334, w: .63, h: .327 },
  gates: [
    { rx: .426, ry: .282, rw: .150, rh: .060, side: 'top'    },
    { rx: .423, ry: .635, rw: .151, rh: .052, side: 'bottom' },
    { rx: .167, ry: .398, rw: .034, rh: .084, side: 'left'   },
    { rx: .167, ry: .551, rw: .036, rh: .084, side: 'left'   },
    { rx: .806, ry: .363, rw: .036, rh: .082, side: 'right'  },
    { rx: .805, ry: .496, rw: .037, rh: .086, side: 'right'  },
  ],
  guard: { speed: .45, half: 26, range: 2.4 },
}
```

### Note tecniche importanti

- Pattuglia: rettangolo espanso di `cell * 0.35` rispetto al pen. Segmento TOP: `y0 - 5` (fix corridoio).
- Griglia animali: centrata nel pen, traslata -14px in alto.
- Collisione: `dist < cell * 0.88` → caught(). Ogni frame + pre-check in tryFreeAnimal. Inibito se sorvegliante stordito.

## 5. Asset

### Ambientazione 1: Allevamento (completata)

| File | Soggetto |
|------|----------|
| `ALF.png` | Attivista col passamontagna |
| `farmer1.png` | Allevatore |
| `goat1.png` | Capra |
| `allevamto1-chiuso.png` | Fondale cancelli chiusi |
| `allevamto1-aperto.png` | Fondale cancelli aperti |

L'attivista ALF resta lo stesso in tutte le ambientazioni.

Per ogni nuova ambientazione servono: 1 fondale chiuso + 1 aperto, 1 sprite animale, 1 sprite sorvegliante.

## 6. Stile visivo

- Font display: Anton / Font UI: Sora
- Palette: nero-marrone notturno, oro caldo (#d4a942), rosso (#b03020), verde (#2e7d4a)
- Cono: gradiente radiale giallo-arancio
- Glow sorvegliante: drop-shadow gialla
- Finale: spray rosso Anton con colature animate

### Slogan ALF

```
LIBERAZIONE ANIMALE ORA
NESSUNA GABBIA È GIUSTA
LIBERI TUTTI
ALF · NESSUN PADRONE
LA LORO VITA NON È MERCE
```

## 7. Architettura del codice

File singolo HTML. Prossimo refactor necessario per supportare livelli multipli:
- `LEVELS[]` — array config di tutti i livelli
- `ENVS{}` — asset per ambientazione
- `loadLevel(id)` — seleziona config + env e chiama build()
- Schermata selezione livelli
- localStorage per progressione e stelle

## 8. Cose decise — NON cambiare

- Visuale dall'alto, pixel art
- Scoperto = ricomincia (niente barra di sospetto)
- File HTML autosufficiente (per ora)
- Tono ALF militante
- ~20 livelli per ambientazione, ~100 totali
- Complessità tramite parametri, non asset nuovi per ogni livello

## 9. TODO (in ordine di priorità)

### P1 — Sistema livelli
- [ ] Refactor: estrarre config in LEVELS[] e asset in ENVS{}
- [ ] loadLevel(id) + transizione tra livelli
- [ ] Schermata selezione livello (griglia con stelline)
- [ ] Salvataggio localStorage
- [ ] 15-20 config per ambientazione "allevamento" (difficoltà crescente)

### P2 — Meccaniche nuove
- [ ] Sorvegliante imprevedibile (inversione, pause)
- [ ] Sorveglianti multipli
- [ ] Telecamere fisse oscillanti
- [ ] Animali rumorosi / auto-free / helper

### P3 — Nuove ambientazioni (servono asset)
- [ ] Capannone industriale (polli)
- [ ] Stalla intensiva (mucche)
- [ ] Zoo (misti)
- [ ] Laboratorio (conigli/topi)
- [ ] Circo (elefanti/tigri)

### P4 — Polish
- [ ] Audio
- [ ] Selezione personaggio
- [ ] Tutorial in-game
- [ ] Build nativa (Capacitor?)
- [ ] Integrazione iframe in Herbdivore

### Bug noti
- Fondale stirato su ratio diversi da 9:16
- Capra forse troppo grande, attivista da riposizionare

## 10. Convenzioni di lavoro

1. Leggere design.md prima di modifiche
2. Modifiche chirurgiche, non riscritture totali
3. Per problemi visivi, chiedere screenshot
4. Aggiornare design.md dopo modifiche significative
