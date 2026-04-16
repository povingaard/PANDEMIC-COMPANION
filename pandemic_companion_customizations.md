# Pandemic Legacy S2 Companion — Customizzazioni icone esposizione

Questo documento descrive tutte le modifiche apportate all'app HTML
`pandemic_tracker_new.html` ospitata su GitHub, riproducibili come prompt
per future sessioni.

---

## Obiettivo

Sostituire le icone SVG dei campi **esposizione** (teschio e cicatrice) nel
**picker** e negli **slot** con immagini PNG personalizzate, e uniformare lo
stile dei bottoni **Attivo** e **Reset** del picker agli slot stessi.

---

## Architettura della soluzione

L'app originale non viene modificata. Si crea un **file HTML wrapper** che:

1. Scarica l'HTML originale via `fetch()` da GitHub
2. Inietta uno `<script>` nel `<head>` prima che React avvii il rendering
3. Converte l'HTML modificato in un `Blob URL` e lo carica in un `<iframe>`

```
wrapper.html (4–9 KB)
  └── fetch → pandemic_tracker_new.html (967 KB, originale intatto)
        └── <script id="icon-js"> iniettato nel <head>
              ├── canvas: rimozione sfondo PNG via soglia redness
              ├── patchSvg(): sostituzione icone SVG
              ├── applyBtn(): restyling Attivo/Reset
              └── MutationObserver: sincronizzazione con i re-render React
```

---

## File sorgente delle icone

Le immagini PNG (con sfondo bianco — l'app le processa via Canvas) sono
ospitate nello stesso repository dell'app:

| Icona    | URL raw GitHub |
|----------|---------------|
| Teschio  | `https://raw.githubusercontent.com/povingaard/PANDEMIC-COMPANION/main/IMG/teschio.png` |
| Cicatrice | `https://raw.githubusercontent.com/povingaard/PANDEMIC-COMPANION/main/IMG/cicatrice.png` |

---

## SVG da sostituire (fingerprint esatti)

L'identificazione avviene per `viewBox` + `width` + figlio caratteristico.

| Elemento | viewBox | width | figlio | Ruolo |
|---|---|---|---|---|
| Slot teschio | `0 0 16 18` | `16` | `[stroke="#D03030"]` | slot selezionato |
| Slot cicatrice | `0 0 28 28` | `28` | `[stroke="#FF5050"]` o `[stroke="#FF6060"]` | slot selezionato |
| Picker teschio | `0 0 32 32` | `40` | `ellipse[cx="16"]` | bottone picker |
| Picker cicatrice | `0 0 32 32` | `40` | `polyline` | bottone picker |

Il check su `width` è essenziale per evitare false corrispondenze (es. campo foto).

---

## Bottoni Attivo e Reset (picker)

Identificati da `div[title="Attivo"]` e `div[title="Reset"]`.

| Bottone | Dimensioni | Sfondo | Bordo | Testo |
|---|---|---|---|---|
| Attivo | 60×40 px | stesso degli input di testo | `1px solid rgb(48,160,48)` | nascosto |
| Reset | 60×40 px | stesso degli input di testo | `1px solid #5588EE` | `✕` in `#5588EE` |

---

## Logica dello script iniettato

### 1. `sampleBg()` — colore di sfondo adattivo

Legge il `background-color` computato dal primo `<input type="text">` dell'app
(il campo nome) e lo usa come sfondo per tutti i box icona. Si aggiorna
dinamicamente se il tema cambia.

```js
// Fallback iniziale, poi sovrascritto a runtime
var SLOT_BG = "rgb(232,228,196)";
```

### 2. `process(url, cb)` — rimozione sfondo PNG via Canvas

Scarica il PNG da GitHub (`crossOrigin="anonymous"`), lo disegna su un
`<canvas>`, rimuove i pixel non-rossi (sfondo bianco) e ritaglia al bounding
box del contenuto opaco.

**Soglia:** pixel trasparente se `R - G < 50` **oppure** `R - B < 50`.
I pixel rossi dell'immagine hanno R≈154, G≈60, B≈60 → differenza ≈90,
ben oltre la soglia. I pixel bianchi hanno R≈G≈B → differenza ≈0.

```js
for (var i = 0; i < d.length; i += 4) {
  if ((d[i] - d[i+1]) < 50 || (d[i] - d[i+2]) < 50) d[i+3] = 0;
}
```

Il risultato viene messo in cache (`cache[url]`) per non riscaricare.

### 3. `clearPickerBg(el)` — rimozione sfondo scuro del bottone picker

Risale fino a 4 livelli nell'albero DOM da `el`, e rende trasparente qualsiasi
antenato con background colorato (il bottone picker originale ha sfondo scuro
che altrimenti si vede dietro il box 60×40).

```js
el.style.setProperty("background", "transparent", "important");
el.style.setProperty("border", "none", "important");
el.style.setProperty("box-shadow", "none", "important");
```

### 4. `patchSvg(svg)` — sostituzione icone

**Strategia chiave:** il SVG originale viene reso `opacity:0` (non
`display:none`) — rimane nel DOM con tutti i suoi event handler React
(selezione, reset, cambio stato). Il box con l'immagine è sovrapposto
con `pointer-events:none`.

**Lifecycle:** un `MutationObserver` per parent osserva la rimozione del SVG
(quando React fa reset/cambio slot) e rimuove il box in sincronia.

**Picker (40×40):**
- `clearPickerBg()` sul parent
- parent ridimensionato a `60×40px` nel layout
- box `position:absolute; inset:0` con sfondo beige
- bordo `1px solid #D03030` (teschio) o `1px solid #FF8800` (cicatrice)

**Slot:**
- box `position:absolute; inset:0` che riempie il contenitore slot
- nessun bordo aggiuntivo

### 5. `applyBtn(div)` — restyling Attivo/Reset

Applica direttamente gli stili con `!important` sul div originale.
Un secondo `MutationObserver` sull'attributo `style` re-applica se React
sovrascrive (es. all'hover o al cambio stato).

### 6. `MutationObserver` globale + scansioni ritardate

```js
// Scansioni di sicurezza dopo i render lenti di React
[50, 300, 800, 2000].forEach(function(t) { setTimeout(scanAll, t); });
```

L'observer globale intercetta ogni nuovo nodo aggiunto al DOM e lo patcha
immediatamente, senza attendere le scansioni ritardate.

---

## Codice completo del wrapper

```html
<!DOCTYPE html>
<html lang="it">
<head>
<meta charset="UTF-8"/>
<meta name="viewport" content="width=device-width,initial-scale=1"/>
<title>Pandemic S2</title>
<style>
html,body{margin:0;padding:0;width:100%;height:100%;background:#272828;overflow:hidden}
iframe{display:block;width:100%;height:100%;border:none}
#msg{position:fixed;inset:0;background:#272828;display:flex;align-items:center;
     justify-content:center;color:#E8E4C4;font-family:sans-serif;font-size:18px;z-index:9999}
</style>
</head>
<body>
<div id="msg">Caricamento…</div>
<script>
var INJ = /* JSON.stringify del blocco <script id="icon-js"> sotto */;

fetch("https://raw.githubusercontent.com/povingaard/PANDEMIC-COMPANION/refs/heads/main/pandemic_tracker_new.html")
  .then(function(r) { if (!r.ok) throw new Error("HTTP " + r.status); return r.text(); })
  .then(function(html) {
    html = html.replace("</head>", INJ + "</head>");
    var b = new Blob([html], { type: "text/html;charset=utf-8" });
    var f = document.createElement("iframe");
    f.sandbox = "allow-scripts allow-same-origin allow-forms allow-modals allow-popups allow-storage-access-by-user-activation";
    f.onload = function() { document.getElementById("msg").style.display = "none"; };
    f.src = URL.createObjectURL(b);
    document.body.appendChild(f);
  })
  .catch(function(e) { document.getElementById("msg").textContent = "Errore: " + e.message; });
</script>
</body>
</html>
```

---

## Script iniettato (da inserire come valore di `INJ`)

```html
<script id="icon-js">(function(){
  var SKULL_URL = "https://raw.githubusercontent.com/povingaard/PANDEMIC-COMPANION/main/IMG/teschio.png";
  var SCAR_URL  = "https://raw.githubusercontent.com/povingaard/PANDEMIC-COMPANION/main/IMG/cicatrice.png";
  var SLOT_BG   = "rgb(232,228,196)"; /* fallback — aggiornato da sampleBg() */
  var cache = {};
  var boxes = [];

  function sampleBg() {
    var inputs = document.querySelectorAll('input[type="text"],input:not([type])');
    for (var i = 0; i < inputs.length; i++) {
      var bg = window.getComputedStyle(inputs[i]).backgroundColor;
      if (bg && bg !== "rgba(0, 0, 0, 0)" && bg !== "transparent") {
        if (bg !== SLOT_BG) { SLOT_BG = bg; boxes.forEach(function(b){ b.style.backgroundColor = SLOT_BG; }); }
        return;
      }
    }
  }

  function process(url, cb) {
    if (cache[url]) { cb(cache[url]); return; }
    var img = new Image();
    img.crossOrigin = "anonymous";
    img.onload = function() {
      var W = img.naturalWidth, H = img.naturalHeight;
      var c = document.createElement("canvas"); c.width = W; c.height = H;
      var ctx = c.getContext("2d");
      ctx.drawImage(img, 0, 0);
      var id = ctx.getImageData(0, 0, W, H), d = id.data;
      /* Rimuovi sfondo bianco: trasparente se il pixel non è rosso */
      for (var i = 0; i < d.length; i += 4) {
        if ((d[i] - d[i+1]) < 50 || (d[i] - d[i+2]) < 50) d[i+3] = 0;
      }
      ctx.putImageData(id, 0, 0);
      /* Ritaglia al bounding box del contenuto opaco */
      id = ctx.getImageData(0, 0, W, H); d = id.data;
      var x0 = W, x1 = 0, y0 = H, y1 = 0;
      for (var y = 0; y < H; y++) for (var x = 0; x < W; x++) {
        if (d[(y * W + x) * 4 + 3] > 0) {
          if (x < x0) x0 = x; if (x > x1) x1 = x;
          if (y < y0) y0 = y; if (y > y1) y1 = y;
        }
      }
      var uri;
      if (x1 > x0 && y1 > y0) {
        var w2 = x1-x0+1, h2 = y1-y0+1;
        var c2 = document.createElement("canvas"); c2.width = w2; c2.height = h2;
        c2.getContext("2d").putImageData(ctx.getImageData(x0, y0, w2, h2), 0, 0);
        uri = c2.toDataURL("image/png");
      } else { uri = c.toDataURL("image/png"); }
      cache[url] = uri; cb(uri);
    };
    img.onerror = function() { cache[url] = url; cb(url); };
    img.src = url;
  }

  function clearPickerBg(startEl) {
    var el = startEl;
    for (var i = 0; i < 4; i++) {
      if (!el || el === document.body) break;
      var bg = window.getComputedStyle(el).backgroundColor;
      if (bg && bg !== "rgba(0, 0, 0, 0)" && bg !== "transparent") {
        el.style.setProperty("background", "transparent", "important");
        el.style.setProperty("border", "none", "important");
        el.style.setProperty("box-shadow", "none", "important");
      }
      el = el.parentNode;
    }
  }

  function classifySvg(svg) {
    var vb = svg.getAttribute("viewBox"), w = svg.getAttribute("width");
    if (!vb) return null;
    if (vb === "0 0 16 18" && w === "16" && svg.querySelector('[stroke="#D03030"]'))
      return { url: SKULL_URL, picker: false, border: "none" };
    if (vb === "0 0 28 28" && w === "28" && (svg.querySelector('[stroke="#FF5050"]') || svg.querySelector('[stroke="#FF6060"]')))
      return { url: SCAR_URL, picker: false, border: "none" };
    if (vb === "0 0 32 32" && w === "40") {
      if (svg.querySelector('ellipse[cx="16"]')) return { url: SKULL_URL, picker: true, border: "1px solid #D03030" };
      if (svg.querySelector("polyline"))          return { url: SCAR_URL,  picker: true, border: "1px solid #FF8800" };
    }
    return null;
  }

  function patchSvg(svg) {
    if (svg._p) return;
    var info = classifySvg(svg);
    if (!info) return;
    svg._p = true;
    svg.style.cssText += ";opacity:0!important;";
    var par = svg.parentNode; if (!par) return;
    var box = document.createElement("div");
    box.setAttribute("data-ib", "1");
    if (info.picker) {
      clearPickerBg(par);
      par.style.setProperty("position", "relative", "important");
      par.style.setProperty("width", "60px", "important");
      par.style.setProperty("height", "40px", "important");
      box.style.cssText = "position:absolute;inset:0;background:" + SLOT_BG + ";"
        + "border:" + info.border + ";"
        + "display:flex;align-items:center;justify-content:center;"
        + "pointer-events:none;overflow:hidden;border-radius:2px;box-sizing:border-box;";
      var imgEl = document.createElement("img");
      imgEl.style.cssText = "max-width:80%;max-height:80%;object-fit:contain;margin:auto;display:block;";
      box.appendChild(imgEl);
      process(info.url, function(uri) { imgEl.src = uri; });
    } else {
      par.style.setProperty("position", "relative", "important");
      box.style.cssText = "position:absolute;inset:0;background:" + SLOT_BG + ";"
        + "display:flex;align-items:center;justify-content:center;"
        + "pointer-events:none;overflow:hidden;";
      var imgEl2 = document.createElement("img");
      imgEl2.style.cssText = "max-width:80%;max-height:80%;object-fit:contain;margin:auto;display:block;";
      box.appendChild(imgEl2);
      process(info.url, function(uri) { imgEl2.src = uri; });
    }
    par.appendChild(box);
    boxes.push(box);
    new MutationObserver(function(ms) {
      ms.forEach(function(m) {
        m.removedNodes.forEach(function(n) {
          if (n === svg && box.parentNode) {
            box.parentNode.removeChild(box);
            boxes = boxes.filter(function(b) { return b !== box; });
          }
        });
      });
    }).observe(par, { childList: true });
  }

  function applyBtn(div) {
    var title = div.getAttribute("title");
    clearPickerBg(div);
    div.style.setProperty("width", "60px", "important");
    div.style.setProperty("height", "40px", "important");
    div.style.setProperty("background", SLOT_BG, "important");
    div.style.setProperty("border-radius", "2px", "important");
    div.style.setProperty("box-sizing", "border-box", "important");
    div.style.setProperty("display", "flex", "important");
    div.style.setProperty("align-items", "center", "important");
    div.style.setProperty("justify-content", "center", "important");
    if (title === "Reset") {
      div.style.setProperty("border", "1px solid #5588EE", "important");
      div.style.setProperty("color", "#5588EE", "important");
      div.style.setProperty("font-size", "14px", "important");
    } else {
      div.style.setProperty("border", "1px solid rgb(48,160,48)", "important");
      div.style.setProperty("color", "transparent", "important");
    }
  }

  function patchBtn(div) {
    if (div._bp) return;
    var title = div.getAttribute("title");
    if (title !== "Attivo" && title !== "Reset") return;
    div._bp = true;
    applyBtn(div);
    new MutationObserver(function() { applyBtn(div); })
      .observe(div, { attributes: true, attributeFilter: ["style"] });
  }

  function scanAll() {
    sampleBg();
    document.querySelectorAll("svg").forEach(patchSvg);
    document.querySelectorAll('div[title="Attivo"],div[title="Reset"]').forEach(patchBtn);
  }

  new MutationObserver(function(ms) {
    ms.forEach(function(m) {
      m.addedNodes.forEach(function(n) {
        if (!n || n.nodeType !== 1) return;
        var t = n.tagName && n.tagName.toLowerCase();
        if (t === "svg") patchSvg(n);
        else {
          if (t === "div") { var tt = n.getAttribute && n.getAttribute("title"); if (tt === "Attivo" || tt === "Reset") patchBtn(n); }
          if (n.querySelectorAll) {
            n.querySelectorAll("svg").forEach(patchSvg);
            n.querySelectorAll('div[title="Attivo"],div[title="Reset"]').forEach(patchBtn);
          }
        }
      });
    });
  }).observe(document.documentElement, { childList: true, subtree: true });

  [50, 300, 800, 2000].forEach(function(t) { setTimeout(scanAll, t); });
})();</script>
```

---

## Modifiche non incluse / da completare

| Feature | Stato | Note |
|---|---|---|
| Nascondere il cursore di testo nel picker sticker (mestiere/capacità/caratteristica) | ❌ Non implementato | Richiede il `className` esatto del pannello sticker per un targeting preciso. Richiedere all'utente via `document.querySelector('[class*="picker"]').className` |
| Linea spuria nel campo foto | ✅ Risolto | Aggiunto controllo `width` esatto nel `classifySvg()` — solo SVG con `width="16"` o `width="28"` vengono patchati, non altri SVG con lo stesso viewBox |

---

## Note tecniche

- **Perché Blob URL e non `srcdoc`**: `srcdoc` ha limiti di lunghezza su alcuni browser; il Blob URL non ha limiti e preserva il comportamento `localStorage`
- **Perché `opacity:0` e non `display:none`**: `display:none` rimuove l'elemento dal layout e fa perdere gli event handler React; `opacity:0` lo mantiene visibile al DOM
- **Perché `!important` su Attivo/Reset**: React sovrascrive gli stili inline ad ogni re-render; `!important` + MutationObserver sull'attributo `style` garantisce la persistenza
- **Perché la soglia `R - G < 50`**: i pixel rossi delle icone hanno R≈154, G≈60, B≈60 (differenza ≈90); i pixel bianchi di bordo hanno R≈G≈B (differenza ≈0). La soglia 50 separa nettamente le due categorie
