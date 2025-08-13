<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1"/>
  <title>Sylvaen Translator (Client‑side)</title>
  <style>
    body{font-family:system-ui,Segoe UI,Arial,sans-serif;max-width:900px;margin:40px auto;padding:0 16px}
    textarea{width:100%;min-height:120px}
    .row{display:flex;gap:12px;flex-wrap:wrap}
    .row>*{flex:1}
    label{display:block;font-weight:600;margin:10px 0 6px}
    select,button,input[type="file"]{padding:8px}
    pre{background:#f6f7f8;padding:12px;border-radius:8px;overflow:auto}
    .pill{display:inline-block;padding:2px 8px;border-radius:999px;background:#eef}
    .muted{color:#666}
  </style>
</head>
<body>
  <h1>💫 Sylvaen Translator</h1>
  <p class="muted">Client‑side, rule‑based demo. Replace the inline dictionary with your full 1400‑word JSON.</p>

  <div class="row">
    <div>
      <label>Source language</label>
      <select id="srcLang">
        <option value="en">English → Sylvaen</option>
        <option value="fr">French → Sylvaen</option>
      </select>
    </div>
    <div>
      <label>Sentence type</label>
      <select id="sentType">
        <option value="statement">Statement (OVS)</option>
        <option value="question">Question (OV?S)</option>
      </select>
    </div>
    <div>
      <label>Verb Tense/Aspect/Mood</label>
      <select id="tam">
        <!-- From your Sylvaen table -->
        <option value="pres_ind">Present (Indicative)</option>
        <option value="past_ind">Past (Indicative)</option>
        <option value="fut_ind">Future (Indicative)</option>
        <option value="pres_perf">Present Perfect</option>
        <option value="past_perf">Past Perfect</option>
        <option value="fut_perf">Future Perfect</option>
        <option value="pres_prog">Present Continuous</option>
        <option value="past_prog">Past Continuous</option>
        <option value="fut_prog">Future Continuous</option>
        <option value="pres_cond">Present Conditional</option>
        <option value="past_cond">Past Conditional</option>
        <option value="fut_cond">Future Conditional</option>
        <option value="pres_imp">Imperative</option>
      </select>
    </div>
  </div>

  <label for="in">Input (simple SVO works best)</label>
  <textarea id="in" placeholder="e.g., The hunter sees the deer."></textarea>

  <div class="row">
    <div>
      <button id="run">Translate →</button>
    </div>
    <div>
      <label class="pill">Optional: Load dictionary JSON</label>
      <input id="file" type="file" accept="application/json" />
    </div>
  </div>

  <label>Output (Sylvaen)</label>
  <textarea id="out" readonly></textarea>

  <details>
    <summary>Show demo dictionary schema</summary>
    <pre id="schema"></pre>
  </details>

<script>
// === 1) Sylvaen TAM helpers (conjugation words are separate particles) ===
// Your table: (blank present-indicative), Past=eg, Future=ir; Perfect: ega/ona/ira;
// Continuous: egi/oni/iri; Conditional: egu/onu/iru; Imperative: ege/one/ire
const TAM = {
  pres_ind:  "",         // present indicative = no particle
  past_ind:  "eg",
  fut_ind:   "ir",

  pres_perf:"ona",
  past_perf:"ega",
  fut_perf: "ira",

  pres_prog:"oni",
  past_prog:"egi",
  fut_prog: "iri",

  pres_cond:"onu",
  past_cond:"egu",
  fut_cond: "iru",

  pres_imp: "one" // Imperative defaulting to present flavor
};

// === 2) Minimal POS tags we’ll rely on ===
// Add/expand as needed in your full lexicon.
const POS = {
  NOUN: "noun", VERB: "verb", ADJ:"adj", ADV:"adv",
  DET:"det", PRON:"pron", PREP:"prep", NUM:"num"
};

// === 3) Demo in-memory dictionary (replace with your full JSON) ===
// Schema: { en, fr, syl, pos, lemma?, notes? }
// Tip: use lemmas for verbs so we can map inflected forms back to the lemma key.
let DICT = [
  // Determiners / simple function words
  {en:"the", fr:"le", syl:"aelth", pos:POS.DET},
  {en:"a", fr:"un", syl:"aelth", pos:POS.DET}, // if your grammar uses same article
  {en:"to", fr:"à", syl:"a", pos:POS.PREP},

  // Nouns (examples)
  {en:"hunter", fr:"chasseur", syl:"chaelorn", pos:POS.NOUN},
  {en:"deer", fr:"cerf", syl:"tholael", pos:POS.NOUN},
  {en:"forest", fr:"forêt", syl:"thol", pos:POS.NOUN},
  {en:"soul", fr:"âme", syl:"elyr", pos:POS.NOUN},

  // Verbs
  {en:"see", fr:"voir", syl:"lek", pos:POS.VERB, lemma:"see"},
  {en:"hunt", fr:"chasser", syl:"chael", pos:POS.VERB, lemma:"hunt"},
  {en:"be", fr:"être", syl:"raen", pos:POS.VERB, lemma:"be"},

  // Adjectives / adverbs
  {en:"good", fr:"bon", syl:"vey", pos:POS.ADJ},
  {en:"calm", fr:"calme", syl:"thal", pos:POS.ADJ}
];

// Show schema for convenience
const DEMO_SCHEMA = {
  en:"english headword",
  fr:"french headword",
  syl:"sylvaen form",
  pos:"noun|verb|adj|adv|det|prep|pron|num",
  lemma:"(for verbs) base form to map inflections",
  notes:"Root Sylvaen / Original Sylvaen / French origin \"word\" / etc."
};
document.getElementById('schema').textContent = JSON.stringify({
  example_entries: DICT.slice(0,6),
  require_fields: Object.keys(DEMO_SCHEMA),
  tam_particles: TAM
}, null, 2);

// === 4) Utilities ===
const isUpper = s => s && s[0] === s[0].toUpperCase();
function normalizeToken(t){
  return t.toLowerCase().replace(/[.,!?;:()"]/g,"");
}
function splitSentences(text){
  // Basic splitter. For real use, improve for quotes/ellipsis.
  return text.match(/[^.!?]+[.!?]?/g)?.map(s=>s.trim()).filter(Boolean) || [];
}
function tokenize(sent){
  // Keep punctuation as tokens (for later restoration)
  const tokens = sent.match(/[\w’'-]+|[.,!?;:()"]/g) || [];
  return tokens;
}

// Dictionary lookup with bilingual support
function lookup(token, lang="en"){
  const norm = normalizeToken(token);
  // Try exact by source language
  let row = DICT.find(d => (d[lang]||"").toLowerCase() === norm);
  if(row) return row;
  // Try lemma fallback for verbs (very naive lemmatization)
  // English heuristics: strip -s, -ed, -ing
  if(lang==="en"){
    const tryForms = [norm.replace(/(ing|ed|es|s)$/,""), norm];
    for(const f of tryForms){
      row = DICT.find(d => d.pos===POS.VERB && (d.lemma||d.en) === f);
      if(row) return row;
    }
  }
  // French minimal: strip plural 's'
  if(lang==="fr"){
    const f2 = norm.replace(/s$/,"");
    row = DICT.find(d => (d[lang]||"").toLowerCase() === f2);
    if(row) return row;
  }
  return null;
}

// Very simple chunker to guess SVO: [DET? NOUN]+ VERB [DET? NOUN]+
function naiveSVO(tokens, lang="en"){
  // Identify verb first by dictionary POS.
  let vIdx = -1;
  for(let i=0;i<tokens.length;i++){
    const row = lookup(tokens[i], lang);
    if(row?.pos===POS.VERB){ vIdx = i; break; }
  }
  if(vIdx<0) return {subject:tokens, verb:null, object:[], pre:[], post:[]};

  const pre = tokens.slice(0, vIdx);
  const verb = tokens[vIdx];
  const post = tokens.slice(vIdx+1);

  // Heuristic: subject = leading NP in pre, object = trailing NP in post
  const subject = pre;      // naive: everything before verb
  const object  = post;     // naive: everything after verb

  return {subject, verb, object, pre:[], post:[]};
}

function applyTAM(verbSyl, tamKey){
  const part = TAM[tamKey] || "";
  return part ? `${part} ${verbSyl}` : verbSyl;
}

// Lowercase/uppercase restoration for first token only (demo)
function restoreCase(tokens, ref){
  if(!tokens.length) return tokens;
  if(isUpper(ref?.[0])) tokens[0] = tokens[0].charAt(0).toUpperCase()+tokens[0].slice(1);
  return tokens;
}

// Translate word-by-word with POS guidance
function translateTokens(tokens, lang="en", tamKey="pres_ind"){
  const out = [];
  for(const t of tokens){
    const bare = normalizeToken(t);
    const punct = /[.,!?;:()"]/.test(t) ? t : null;
    if(punct){ out.push(punct); continue; }

    const row = lookup(t, lang);
    if(!row){ out.push(t); continue; } // passthrough

    if(row.pos===POS.VERB){
      const v = applyTAM(row.syl, tamKey);
      out.push(v);
    }else{
      out.push(row.syl);
    }
  }
  return out;
}

// Reorder SVO -> OVS (statement) or OVS? (question)
function reorderToSylvaen(tokens, lang, tamKey, type){
  const {subject, verb, object} = naiveSVO(tokens, lang);

  // Map each chunk to Sylvaen
  const s_subj = translateTokens(subject, lang, tamKey);
  const s_verb = verb ? translateTokens([verb], lang, tamKey) : [];
  const s_obj  = translateTokens(object, lang, tamKey);

  // Basic punctuation handling
  const endsQ = type==="question" ? "?" : "";
  const endsPunct = tokens[tokens.length-1].match(/[.!?]$/) ? "" : (type==="question" ? "?" : ".");

  // Assemble: O V S (Sylvaen typical: O-S-V, but we keep particles closest to verb)
  let assembled = [];
  if(s_obj.length) assembled = assembled.concat(s_obj);
  if(s_subj.length) assembled = assembled.concat(s_subj);
  if(s_verb.length) assembled = assembled.concat(s_verb);

  // Cleanup multiple spaces & join
  let text = assembled.join(" ");

  // Terminal punctuation
  text = text.replace(/\s+([.,!?;:])/g,"$1"); // tidy punctuation spaces
  text = text.trim() + (endsQ || endsPunct);

  return text;
}

// === 5) Wire up UI ===
document.getElementById('run').addEventListener('click', () => {
  const srcLang = document.getElementById('srcLang').value; // en|fr
  const tamKey  = document.getElementById('tam').value;
  const type    = document.getElementById('sentType').value;
  const input   = document.getElementById('in').value;

  const sents = splitSentences(input);
  const outSents = sents.map(sent=>{
    const toks = tokenize(sent);
    return reorderToSylvaen(toks, srcLang, tamKey, type);
  });
  document.getElementById('out').value = outSents.join(" ");
});

// Allow loading your full dictionary JSON
document.getElementById('file').addEventListener('change', (e)=>{
  const f = e.target.files?.[0];
  if(!f) return;
  const reader = new FileReader();
  reader.onload = () => {
    try{
      const json = JSON.parse(reader.result);
      if(Array.isArray(json)) {
        DICT = json;
        alert(`Loaded ${json.length} dictionary entries.`);
      } else {
        alert('JSON must be an array of entries.');
      }
    }catch(err){
      alert('Invalid JSON: '+err.message);
    }
  };
  reader.readAsText(f);
});

</script>
</body>
</html>
