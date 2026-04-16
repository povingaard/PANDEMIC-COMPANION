# Pandemic Legacy S2 Companion — Modifiche all'app

## Struttura

L'app originale (`pandemic_tracker_new.html`, ~967 KB su GitHub) non viene
modificata. Si crea un file wrapper che la carica, inietta uno script nel
`<head>` e la renderizza in un iframe.

```
wrapper.html  →  fetch(app originale)  →  injectionScript nel <head>  →  Blob URL  →  <iframe>
```

---

## File wrapper

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
var INJ = /* JSON.stringify dello script sotto, con </ escaped come <\/ */;

fetch("https://raw.githubusercontent.com/povingaard/PANDEMIC-COMPANION/refs/heads/main/pandemic_tracker_new.html")
  .then(function(r){ if(!r.ok) throw new Error("HTTP "+r.status); return r.text(); })
  .then(function(html){
    html = html.replace("</head>", INJ + "</head>");
    var b = new Blob([html], {type:"text/html;charset=utf-8"});
    var f = document.createElement("iframe");
    f.sandbox = "allow-scripts allow-same-origin allow-forms allow-modals allow-popups allow-storage-access-by-user-activation";
    f.onload = function(){ document.getElementById("msg").style.display="none"; };
    f.src = URL.createObjectURL(b);
    document.body.appendChild(f);
  })
  .catch(function(e){ document.getElementById("msg").textContent="Errore: "+e.message; });
</script>
</body>
</html>
```

---

## Script iniettato

Il valore di `INJ` è questo blocco serializzato con `JSON.stringify()`,
con tutte le occorrenze di `</` rimpiazzate da `<\/`.

```html
<script id="icon-js">(function(){

  /* ── Costanti ──────────────────────────────────────────── */

  var SKULL_URL = "https://raw.githubusercontent.com/povingaard/PANDEMIC-COMPANION/main/IMG/teschio.png";
  var SCAR_URL  = "https://raw.githubusercontent.com/povingaard/PANDEMIC-COMPANION/main/IMG/cicatrice.png";
  var SLOT_BG   = "rgb(232,228,196)"; /* aggiornato dinamicamente da sampleBg() */
  var cache = {};   /* URI processate, per non riscaricare */
  var boxes = [];   /* riferimenti ai box overlay, per aggiornare SLOT_BG */


  /* ── sampleBg ──────────────────────────────────────────────
     Legge il background-color del primo <input type="text">
     (campo nome) e lo usa come sfondo per tutti i box icona.
     Aggiorna i box già creati se il colore cambia.           */

  function sampleBg(){
    var inputs = document.querySelectorAll('input[type="text"],input:not([type])');
    for(var i=0; i<inputs.length; i++){
      var bg = window.getComputedStyle(inputs[i]).backgroundColor;
      if(bg && bg!=="rgba(0, 0, 0, 0)" && bg!=="transparent"){
        if(bg !== SLOT_BG){
          SLOT_BG = bg;
          boxes.forEach(function(b){ b.style.backgroundColor = SLOT_BG; });
        }
        return;
      }
    }
  }


  /* ── process ───────────────────────────────────────────────
     Scarica il PNG da GitHub, rimuove lo sfondo bianco via
     Canvas API, ritaglia al bounding box del contenuto rosso.

     Soglia: pixel trasparente se (R-G < 50) OR (R-B < 50).
     Le icone hanno R≈154 G≈60 B≈60 → differenza ≈90 (passa).
     Lo sfondo bianco ha R≈G≈B → differenza ≈0 (rimosso).    */

  function process(url, cb){
    if(cache[url]){ cb(cache[url]); return; }
    var img = new Image();
    img.crossOrigin = "anonymous";
    img.onload = function(){
      var W = img.naturalWidth, H = img.naturalHeight;
      var c = document.createElement("canvas"); c.width=W; c.height=H;
      var ctx = c.getContext("2d");
      ctx.drawImage(img, 0, 0);
      var id = ctx.getImageData(0,0,W,H), d = id.data;
      for(var i=0; i<d.length; i+=4){
        if((d[i]-d[i+1])<50 || (d[i]-d[i+2])<50) d[i+3]=0;
      }
      ctx.putImageData(id, 0, 0);
      /* Bounding box */
      id = ctx.getImageData(0,0,W,H); d = id.data;
      var x0=W, x1=0, y0=H, y1=0;
      for(var y=0; y<H; y++) for(var x=0; x<W; x++){
        if(d[(y*W+x)*4+3]>0){
          if(x<x0)x0=x; if(x>x1)x1=x; if(y<y0)y0=y; if(y>y1)y1=y;
        }
      }
      var uri;
      if(x1>x0 && y1>y0){
        var w2=x1-x0+1, h2=y1-y0+1;
        var c2 = document.createElement("canvas"); c2.width=w2; c2.height=h2;
        c2.getContext("2d").putImageData(ctx.getImageData(x0,y0,w2,h2), 0, 0);
        uri = c2.toDataURL("image/png");
      } else { uri = c.toDataURL("image/png"); }
      cache[url] = uri; cb(uri);
    };
    img.onerror = function(){ cache[url]=url; cb(url); };
    img.src = url;
  }


  /* ── clearPickerBg ─────────────────────────────────────────
     Risale fino a 4 livelli nell'albero DOM e rende trasparente
     ogni antenato con background colorato (il bottone picker
     originale ha sfondo scuro visibile dietro il box 60×40).  */

  function clearPickerBg(startEl){
    var el = startEl;
    for(var i=0; i<4; i++){
      if(!el || el===document.body) break;
      var bg = window.getComputedStyle(el).backgroundColor;
      if(bg && bg!=="rgba(0, 0, 0, 0)" && bg!=="transparent"){
        el.style.setProperty("background","transparent","important");
        el.style.setProperty("border","none","important");
        el.style.setProperty("box-shadow","none","important");
      }
      el = el.parentNode;
    }
  }


  /* ── classifySvg ───────────────────────────────────────────
     Identifica i 4 SVG da sostituire per viewBox + width + figlio.
     Il check su width evita false corrispondenze (es. campo foto).

     Slot teschio:    viewBox="0 0 16 18"  width="16"  [stroke="#D03030"]
     Slot cicatrice:  viewBox="0 0 28 28"  width="28"  [stroke="#FF5050"|"#FF6060"]
     Picker teschio:  viewBox="0 0 32 32"  width="40"  ellipse[cx="16"]
     Picker cicatrice:viewBox="0 0 32 32"  width="40"  polyline              */

  function classifySvg(svg){
    var vb = svg.getAttribute("viewBox"), w = svg.getAttribute("width");
    if(!vb) return null;
    if(vb==="0 0 16 18" && w==="16" && svg.querySelector('[stroke="#D03030"]'))
      return {url:SKULL_URL, picker:false, border:"none"};
    if(vb==="0 0 28 28" && w==="28" && (svg.querySelector('[stroke="#FF5050"]')||svg.querySelector('[stroke="#FF6060"]')))
      return {url:SCAR_URL, picker:false, border:"none"};
    if(vb==="0 0 32 32" && w==="40"){
      if(svg.querySelector('ellipse[cx="16"]')) return {url:SKULL_URL, picker:true, border:"1px solid #D03030"};
      if(svg.querySelector("polyline"))          return {url:SCAR_URL,  picker:true, border:"1px solid #FF8800"};
    }
    return null;
  }


  /* ── patchSvg ──────────────────────────────────────────────
     Sostituisce un SVG icona con il box immagine.

     SVG → opacity:0 (non display:none: mantiene eventi React e layout).
     Box → position:absolute sul parent, pointer-events:none.

     Picker:  parent ridimensionato a 60×40px,
              clearPickerBg() per togliere lo sfondo scuro del bottone,
              bordo colorato sulla box (rosso/arancione).
     Slot:    box inset:0, riempie il contenitore slot.

     Lifecycle: MutationObserver sul parent rimuove il box quando
     React rimuove il SVG (reset/cambio stato).                */

  function patchSvg(svg){
    if(svg._p) return;
    var info = classifySvg(svg);
    if(!info) return;
    svg._p = true;
    svg.style.cssText += ";opacity:0!important;";
    var par = svg.parentNode; if(!par) return;
    var box = document.createElement("div");
    box.setAttribute("data-ib","1");

    if(info.picker){
      clearPickerBg(par);
      par.style.setProperty("position","relative","important");
      par.style.setProperty("width","60px","important");
      par.style.setProperty("height","40px","important");
      box.style.cssText = "position:absolute;inset:0;background:"+SLOT_BG+";"
        +"border:"+info.border+";"
        +"display:flex;align-items:center;justify-content:center;"
        +"pointer-events:none;overflow:hidden;border-radius:2px;box-sizing:border-box;";
      var imgEl = document.createElement("img");
      imgEl.style.cssText = "max-width:80%;max-height:80%;object-fit:contain;margin:auto;display:block;";
      box.appendChild(imgEl);
      process(info.url, function(uri){ imgEl.src=uri; });
    } else {
      par.style.setProperty("position","relative","important");
      box.style.cssText = "position:absolute;inset:0;background:"+SLOT_BG+";"
        +"display:flex;align-items:center;justify-content:center;"
        +"pointer-events:none;overflow:hidden;";
      var imgEl2 = document.createElement("img");
      imgEl2.style.cssText = "max-width:80%;max-height:80%;object-fit:contain;margin:auto;display:block;";
      box.appendChild(imgEl2);
      process(info.url, function(uri){ imgEl2.src=uri; });
    }
    par.appendChild(box);
    boxes.push(box);
    new MutationObserver(function(ms){
      ms.forEach(function(m){
        m.removedNodes.forEach(function(n){
          if(n===svg && box.parentNode){
            box.parentNode.removeChild(box);
            boxes = boxes.filter(function(b){ return b!==box; });
          }
        });
      });
    }).observe(par, {childList:true});
  }


  /* ── applyBtn / patchBtn ───────────────────────────────────
     Restyling dei bottoni Attivo e Reset del picker.
     Identificati da div[title="Attivo"] e div[title="Reset"].

     Attivo: 60×40, sfondo SLOT_BG, bordo verde 1px, testo nascosto.
     Reset:  60×40, sfondo SLOT_BG, bordo blu 1px, ✕ in blu.

     clearPickerBg() rimuove lo sfondo scuro del bottone originale.
     MutationObserver sull'attributo style re-applica se React
     sovrascrive gli stili (hover, cambio stato).               */

  function applyBtn(div){
    var title = div.getAttribute("title");
    clearPickerBg(div);
    div.style.setProperty("width","60px","important");
    div.style.setProperty("height","40px","important");
    div.style.setProperty("background",SLOT_BG,"important");
    div.style.setProperty("border-radius","2px","important");
    div.style.setProperty("box-sizing","border-box","important");
    div.style.setProperty("display","flex","important");
    div.style.setProperty("align-items","center","important");
    div.style.setProperty("justify-content","center","important");
    if(title==="Reset"){
      div.style.setProperty("border","1px solid #5588EE","important");
      div.style.setProperty("color","#5588EE","important");
      div.style.setProperty("font-size","14px","important");
    } else {
      div.style.setProperty("border","1px solid rgb(48,160,48)","important");
      div.style.setProperty("color","transparent","important");
    }
  }

  function patchBtn(div){
    if(div._bp) return;
    var title = div.getAttribute("title");
    if(title!=="Attivo" && title!=="Reset") return;
    div._bp = true;
    applyBtn(div);
    new MutationObserver(function(){ applyBtn(div); })
      .observe(div, {attributes:true, attributeFilter:["style"]});
  }


  /* ── scanAll + observers ───────────────────────────────────
     scanAll() viene chiamata subito e a 50/300/800/2000ms per
     coprire i render lenti di React.
     L'observer globale patcha ogni nuovo nodo appena aggiunto. */

  function scanAll(){
    sampleBg();
    document.querySelectorAll("svg").forEach(patchSvg);
    document.querySelectorAll('div[title="Attivo"],div[title="Reset"]').forEach(patchBtn);
  }

  new MutationObserver(function(ms){
    ms.forEach(function(m){
      m.addedNodes.forEach(function(n){
        if(!n || n.nodeType!==1) return;
        var t = n.tagName && n.tagName.toLowerCase();
        if(t==="svg") patchSvg(n);
        else {
          if(t==="div"){ var tt=n.getAttribute&&n.getAttribute("title"); if(tt==="Attivo"||tt==="Reset") patchBtn(n); }
          if(n.querySelectorAll){
            n.querySelectorAll("svg").forEach(patchSvg);
            n.querySelectorAll('div[title="Attivo"],div[title="Reset"]').forEach(patchBtn);
          }
        }
      });
    });
  }).observe(document.documentElement, {childList:true, subtree:true});

  [50,300,800,2000].forEach(function(t){ setTimeout(scanAll,t); });

})();</script>
```

---

## Serializzazione di INJ

```js
// In Node.js o nella console del browser:
var INJ = JSON.stringify(scriptString).replace(/<\//g, "<\\/");
```

`scriptString` è il contenuto del blocco `<script id="icon-js">…</script>`
sopra, inclusi i tag di apertura e chiusura.
