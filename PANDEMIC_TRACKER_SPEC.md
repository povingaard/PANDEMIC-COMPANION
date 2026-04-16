# Pandemic Legacy Season 2 — Companion App
## Specifiche tecniche e funzionali complete

> Documento di riferimento per ricostruire l'app da zero.  
> Stato attuale: file HTML auto-contenuto (~964KB), React 18 compilato con Vite.

---

## 1. Architettura generale

### Stack tecnologico
- **React 18** (bundle compilato con Vite, IIFE self-executing)
- **Single HTML file** — tutto inline: JS, CSS, immagini (WebP base64), font
- **Stato persistente** — `localStorage`, chiavi con prefisso `pl_`
- **Font** — Oswald (Google Fonts) + Impact/Arial Narrow come fallback

### Struttura dell'app
```
App
├── Tab: PERSONAGGI    → CrewRegistry
├── Tab: RETE DI CITTÀ → CityRegistry
├── Tab: MAZZO GIOCATORI → PlayerDeck (+ InfectionDeck)
└── Tab: DASHBOARD     → (navigazione globale)
```

### Tab bar
- Posizione: top sticky, sfondo `#272828`
- Bottoni: border `1.5px solid #7A7660`, border-radius top only
- Tab attivo: bg `#272828` (card), opacity 1; inattivo opacity 0.6
- Font: Oswald, 13px, weight 700, letter-spacing 0.1em

---

## 2. Design System

### Palette colori principali
| Token | Hex | Uso |
|---|---|---|
| `--color-bg` | `#1A1A18` | sfondo pagina |
| `--color-card` | `#272828` | sfondo schede/card |
| `--color-border` | `#7A7660` | bordi generici |
| `--color-border-light` | `#5A5840` | bordi secondari |
| `--color-label` | `#D4C89A` | testo etichette/titoli |
| `--color-text` | `#1B2A4A` | testo nei campi (navy) |
| `--field-bg` | `#E8D9BE` | sfondo campi scheda personaggio |

### Sticker mestiere/capacità
| Proprietà | Valore |
|---|---|
| Sfondo | `#D4C4A0` |
| Bordo (edit overlay) | nessuno (stesso colore sfondo) |
| Testo etichetta | `#7A6830` |
| Edit overlay border | `4px solid #6A9AB0` |

### Sticker cicatrice
| Proprietà | Valore |
|---|---|
| Sfondo | `#e9b770` |
| Bordo (edit overlay) | nessuno (stesso colore sfondo) |
| Testo etichetta | `#8A5010` |
| Edit overlay border | `4px solid #6A9AB0` |

### Picker / dialogs
| Proprietà | Valore |
|---|---|
| Sfondo | `transparent` |
| Bordo | `4px solid #6A9AB0` |
| Titolo | `#2A4A6A`, 16px, weight 700, letter-spacing 0.1em |

### Pulsanti azione
| Tipo | BG | Bordo | Testo |
|---|---|---|---|
| OK / MODIFICA | `#1A3A4A` | `#A8CED8` | `#A8CED8` |
| RITORNA / ↩ | `#2A2810` | `#D4C89A` | `#D4C89A` |
| ELIMINA | `#3A0808` | `#FF6060` | `#FF6060` |

### Worn border overlay
- Border-radius 20px su tutti i lati
- Mask SVG con `feTurbulence` per effetto consumato
- Colore: `#7A7660`
- Linea orizzontale inferiore: `height: 12px`, stesso colore

---

## 3. Scheda Personaggio (CharForm)

### Dimensioni card
- **Totale**: 1100 × 720px
- **Layout frontale** (div assoluto): 1100 × 675px
- `zIndex: 100` per la card, `zIndex: 200` per il layout assoluto
- `border-radius: 20px`
- `background: #272828`

### Header
- Altezza: 45px (grid-template-rows: 45px 1fr)
- Padding: `11px 20px 17px`
- Titolo: "SCHEDA PERSONAGGIO", 26px, weight 700, `#D4C89A`, letter-spacing 0.1em
- Icona utente: SVG 26px, `marginTop: 2px`, fill currentColor
- Gap tra icona e testo: 8px

#### Pulsanti header (destra)
- `FLIP ▶` / `◀ FRONTE`: toggle lato scheda
- `💾 SALVA`: salva personaggio
- `↩`: annulla/ritorna
- `🗑` (rosso): elimina personaggio (condizionale)
- Stile: `1.5px solid #7A7660`, border-radius 6, 13px

### Layout frontale — coordinate assolute (px)

#### Colonna sinistra (left: 22.9, width: 320.9)
| Campo | top | height | Note |
|---|---|---|---|
| NOME | 11.3 | 45.4 | `borderBottom: 2.5px solid #272828` |
| FOTO | 56.7 | 306.2 | area foto personaggio |
| IMG CITTÀ | 362.9 | 170.0 | bg `rgba(210,185,100,0.45)`, immagine city_light centrata |
| DIMORA | 532.9 | 45.4 | `borderBottom + borderRight: 2.5px solid #272828`, width: 252.1 |
| ETÀ | 532.9 | 45.4 | `borderBottom: 2.5px`, left: 275.0, width: 68.8 |
| LUOGO DELLA MORTE | 578.3 | 45.4 | select città |

#### Colonna destra (left: 389.6, width: 687.5)
| Campo | top | height | Note |
|---|---|---|---|
| Skill 0 | 8.8 | 100.7 | `borderTop + borderBottom: 2.5px solid #272828` |
| Skill 1 | 109.5 | 100.7 | idem |
| Skill 2 | 210.2 | 100.7 | idem |
| Skill 3 | 310.9 | 100.7 | idem |
| Skill 4 | 411.6 | 100.7 | idem |
| Label ESPOSIZIONE | 522.9 | 34.1 | `#272828` bg, testo `#808080` |
| Slot esposizione | 557.0 | 44.0 | 7 slot |
| Numeri slot | 604 | — | 1-7, testo `#D4C89A` |

#### Sfondo campi
```css
background: #E8D9BE;
background-image:
  radial-gradient(ellipse at 20% 60%, rgba(100,60,10,0.07) 0%, transparent 60%),
  radial-gradient(ellipse at 80% 30%, rgba(80,45,5,0.05)  0%, transparent 50%),
  radial-gradient(ellipse at 55% 85%, rgba(110,70,15,0.04) 0%, transparent 40%);
```

#### Etichette campi
- `position: absolute, top: 1, right: 3`
- `fontSize: 14, fontWeight: 700, color: #808080`
- Visibili su: NOME, FOTO, DIMORA, ETÀ, LUOGO DELLA MORTE
- Nascoste sui campi skill

---

## 4. Sistema Sticker (SkillSticker)

### Tipi sticker
| Tipo | Campi disponibili | Label | BG | Testo etichetta |
|---|---|---|---|---|
| `mestiere` | solo campo 0 | MESTIERE | `#D4C4A0` | `#7A6830` |
| `capacita` | campi 1-4 | CAPACITÀ | `#D4C4A0` | `#7A6830` |
| `cicatrice` | campi 1-4 | CICATRICE | `#e9b770` | `#8A5010` |

### Sticker (stato normale)
- `position: absolute, top:5, bottom:5, left:5, right:5`
- `borderRadius: 6, zIndex: 51`
- Nessun bordo
- Testo: `fontSize: 15, color: #1B2A4A` — **grassetto fino ai `:`**, resto opacity 0.7
- Etichetta tipo: `position: absolute, bottom: 6, right: 8`, `fontSize: 13, fontWeight: 700, letterSpacing: 0.12em`
- `cursor: pointer` — click apre sticker menu

### Picker tipo (nessun sticker selezionato)
- Si apre cliccando sul campo skill vuoto
- `position: absolute` nel layout assoluto
- Coordinate picker (top = field_top + 2.5 border + 5 inset):
  | Campo | top picker | height |
  |---|---|---|
  | 0 | 16.3 | 85.7 |
  | 1 | 117.0 | 85.7 |
  | 2 | 217.7 | 85.7 |
  | 3 | 318.4 | 85.7 |
  | 4 | 419.1 | 85.7 |
- `left: 394.6, width: 677.5`
- BG: `transparent`, bordo: `4px solid #6A9AB0`
- Titolo: "INSERISCI CARATTERISTICA", `paddingLeft:8, paddingRight:50`
- Pulsante X: `30×30px`, `1.5px solid #4A7A9A`, `color: #2A4A6A`, `fontSize: 18`
- Pulsanti tipo: `height: 30, padding: 0 6px`, BG = sfondo sticker, bordo = colore testo etichetta
  - MESTIERE: `bg #D4C4A0, border #A89860, color #7A6830`  
  - CAPACITÀ: `bg #D4C4A0, border #A89860, color #7A6830`
  - CICATRICE: `bg #e9b770, border #DBA250, color #8A5010`
- Click su tipo → salva tipo + auto-entra in edit

### Sticker menu (click su sticker esistente)
- `position: absolute, inset: 0, zIndex: 500`
- BG: `transparent`, bordo: `1.5px solid #6A9AB0`
- 3 pulsanti `40×40px`:
  - ✏️ MODIFICA: `bg #1A3A4A, border #A8CED8, color #A8CED8`
  - 🗑 ELIMINA: `bg #3A0808, border #FF6060, color #FF6060, fontSize 22`
  - ↩ RITORNA: `bg #2A2810, border #D4C89A, color #D4C89A, fontSize 22`

### Conferma eliminazione
- Testo "ELIMINARE LO STICKER?": `fontSize: 15, fontWeight: 700, color: #1B2A4A`
- Pulsante ELIMINA: `bg #3A0808, border #FF6060, padding: 8px 14px` (adattivo)
- Pulsante ↩: `40×40px`, stesso stile RITORNA

### Edit overlay
- `position: absolute, inset: 5px, zIndex: 63`
- BG = sfondo sticker tipo (mestiere/capacità: `#D4C4A0`, cicatrice: `#e9b770`)
- Bordo: `4px solid #6A9AB0` (stesso del picker)
- Layout row: contenteditable (`flex:1`) + colonna pulsanti destra
- Colonna pulsanti: `justifyContent: space-evenly, alignSelf: stretch`
- Pulsante OK (`30×30`): `bg #1A3A4A, border #A8CED8, color #A8CED8, fontSize 12, fontWeight 700`
- Pulsante ↩ (`30×30`): `bg #2A2810, border #D4C89A, color #D4C89A, fontSize 14`
- Limite 3 righe: se `scrollHeight > clientHeight` → revert al draft precedente
- Contenteditable uncontrolled (ref callback per inizializzazione + posizione cursore)

---

## 5. Slot Esposizione

### Layout
- 7 slot in riga, `justify-content: space-evenly`
- Ogni slot: `60×40px`
- Wrapper: `left: 389.6, top: 557.0, width: 687.5, height: 44.0, zIndex: 2`
- Numeri 1-7: `top: 604`, stessa larghezza, centrati

### Colori slot
| Stato | BG | Note |
|---|---|---|
| Vuoto | `#8A9090` | |
| Attivo | `transparent` | |
| Cicatrice | `#7A3800` | |
| Teschio | `#200808` | |
| Bordo sempre | `2px solid #808080` | |

### Logica slot
- Slot non cliccabile se precedente non compilato (`isLocked`)
- `pointer-events: none` su slot bloccato (stessa grafica degli sbloccati)
- Slot con teschio: tutti gli altri slot con valore diverso da teschio diventano non cliccabili
- Click slot → apre dropdown tendina sopra lo slot

### Tendina dropdown
- `position: absolute, bottom: calc(100% + 6px), left: 50%, transform: translateX(-50%)`
- `width: 60px` (stessa larghezza dello slot)
- `background: #272828, border: 1px solid #1E4A5A, borderRadius: 8, padding: 6px, gap: 4px`
- `box-shadow: 0 4px 16px #000B`
- 4 opzioni `44×44px`:
  1. Reset (✕): `transparent, border 2px solid #7A7660, color #D4C89A, fontSize 17`
  2. Attivo: `bg #1A5C1A (o #1A5C1A33 se non attivo), border 2px solid #30A030`
  3. Cicatrice: `bg variabile, border 4px solid #CC7820` + icona zigzag SVG rosso
  4. Teschio: `bg variabile, border 4px solid #D03030` + icona teschio SVG rosso

### Icona cicatrice (SVG 40×40)
```svg
<polyline points="4,16 9,10 14,22 19,10 24,22 28,16"
  stroke="#FF6060" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"/>
```
viewBox: `0 0 32 32`, fill: none

---

## 6. Sistema Focus Lock (overlay di blocco)

Quando `picker | stickerMenu | editingSkill` è attivo:

| Overlay | Tipo | z-index | Condizione | onClick |
|---|---|---|---|---|
| Dim esterno | `position:fixed, inset:0` | 39 | picker \| menu \| edit | nessuno |
| Full-card | `position:absolute, inset:0` (dentro card) | 99 | picker \| menu \| edit | nessuno |
| Intra-card | `position:absolute, inset:0` (dentro abs layout) | 55 | picker \| edit | nessuno |

**Note z-index:**
- Card: z 100 (nel root)
- Abs layout: z 200 (dentro card) — batte full-card overlay z 99
- Sticker: z 51 (dentro abs layout)
- Intra-card overlay: z 55 — copre sticker (z 51) ma non picker (z 600) né edit (z 63)
- StickerMenu: z 500 (dentro abs layout) — batte intra-card overlay z 55
- Picker: z 600 (dentro abs layout)
- Edit overlay: z 63 (dentro field) — batte intra-card z 55

**Conseguenza:** quando picker o edit è aperto, slot esposizione (z 2) e altri campi sono bloccati. Il stickerMenu (z 500) è cliccabile anche con intra-card attivo.

---

## 7. Immagini

### Immagini embedded (WebP base64)
| Nome | Dimensione | Uso | Note |
|---|---|---|---|
| `city_dark` | ~10KB | CSS background della card | 269×245px, `cardBgImg` |
| `city_light` | ~21KB | Campo IMG CITTÀ | 150×136px, `opacity: 0.7` |
| `city_icon` | ~6KB | CompassIcon (dashboard) | 80×72px, 38px render |

---

## 8. Retro Scheda Personaggio

Accessibile tramite pulsante "RETRO ▶" nell'header. Struttura da implementare — attualmente placeholder. Il toggle `showBack` controlla quale lato è visibile.

---

## 9. CrewRegistry (Gestione Personaggi)

### Struttura
- Lista personaggi con selezione attiva
- Tabella PIG (Players In Game): filtra personaggi con nome non vuoto
- Click riga → seleziona personaggio corrente
- Slot mazzo calcolati da numero personaggi nominati

### Operazioni
- Aggiungere nuovo personaggio (`mkChar()`)
- Eliminare personaggio (se campi vuoti)
- Salvare modifiche
- Navigare tra personaggi

---

## 10. PlayerDeck e InfectionDeck

### PlayerDeck (~69KB)
- Gestione mazzo giocatori
- Distribuzione carte per fase
- Calcolo probabilità epidemia
- `cardsPerPhase`, `hypergeomAtLeastOne` — funzioni matematiche per probabilità

### InfectionDeck (~31KB)
- Mazzo infezione
- `ProbTable`, `ProbBar` — visualizzazione probabilità
- Tabelle di probabilità contagio per città

---

## 11. CityRegistry (~33KB)

### Componenti
- `CityEditRow` — riga di modifica città
- `CityTable` — tabella città
- `CityRow` — singola riga
- `CityRegistry` — container principale

### CitySelect
- Componente select riusabile con lista città
- Usato anche nel campo LUOGO DELLA MORTE del CharForm
- Props: `value, onChange, lineCol, textCol, S`

---

## 12. Stato Globale (localStorage)

| Chiave | Contenuto |
|---|---|
| `pl_characters` | array personaggi (mkChar objects) |
| `pl_players` | array giocatori con colori |
| `pl_assignments` | assegnazioni personaggio-giocatore |
| `pl_cities` | array città con stato |
| `pl_game` | `{started, epidemics, contLevel}` |

### Struttura `mkChar()`
```js
{
  id:           string,       // timestamp + random
  name:         string,
  age:          string,
  dimora:       string,
  death:        string,       // città della morte
  portrait:     null | base64,
  preferito:    boolean,
  backRows:     {},           // retro scheda
  skills:       string[5],    // testo skill 0-4
  skillTypes:   (null|string)[5], // 'mestiere'|'capacita'|'cicatrice'|null
  stickerTexts: string[5],    // testo negli sticker
  exposure:     string[7],    // ''|'active'|'scar'|'skull'
}
```

---

## 13. NavBar

- Componente navigazione globale
- Usa `.navbar-container` e `.nav-icon` CSS class
- ~2KB

---

## 14. File HTML — caratteristiche tecniche

### Dimensioni attuali
- **Totale**: ~964KB
- React runtime: 758KB (79%)
- Codice app: ~192KB
- HTML+CSS: ~13KB
- Immagini embedded (WebP): ~37KB

### CSS classi definite
Attive: `.char-editing-overlay`, `.city-action-btn`, `.discard-pill`, `.nav-icon`, `.navbar-container`
Inutilizzate (legacy): `.active-tab`, `.chevron-aggiungi`, `.closed`, `.nav-tab`, `.open`, `.pill-hover`

### Note bundle
- `process.env.NODE_ENV`: 0 occorrenze (build production corretta)
- `/* @__PURE__ */`: 1118 occorrenze (React createElement ottimizzato)
- `abilita` nel tInfo: 5 occorrenze (tipo legacy, mai esposto nell'UI)
- `nameError`, `confirmDeleteSticker`: state variables definite ma mai usate

---

## 15. Flusso di lavoro sviluppo

### Modifiche all'HTML
1. Aprire il file in nuova chat Claude
2. Condividere link GitHub raw: `https://raw.githubusercontent.com/povingaard/PANDEMIC-COMPANION/refs/heads/main/pandemic_tracker_new.html`
3. Claude scarica il file, applica modifiche, restituisce l'HTML aggiornato
4. Caricare su GitHub la nuova versione

### Repository GitHub
- URL: `https://github.com/povingaard/PANDEMIC-COMPANION`
- Visibilità: pubblica
- File principale: `pandemic_tracker_new.html`
- Immagini: `/IMG/` (city_icon.png, teschio.png, ecc.)

### Progetto sorgente React (opzionale)
Disponibile come `pandemic-tracker-src.zip` con:
- `npm install && npm run dev` per sviluppo locale
- `?preview=charform` per isolare CharForm
- Struttura: `src/components/`, `src/data/constants.js`, `src/hooks/useStorage.js`
