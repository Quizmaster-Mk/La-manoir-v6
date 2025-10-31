<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>La Malédiction du Manoir — Version Longue</title>
<style>
  @import url('https://fonts.googleapis.com/css2?family=Creepster&display=swap');

  :root{
    --accent:#ff3c00;
    --bg:#050506;
    --panel:rgba(0,0,0,0.78);
  }

  html,body{height:100%;margin:0;background:radial-gradient(circle at 25% 10%, #030304 8%, #0b0b0e 60%); color:#eee; font-family:'Creepster',cursive}
  #bg{position:fixed; inset:0; z-index:-4; background:url('https://i.ibb.co/ZTDCxmr/haunted-mansion-night.jpg') center/cover no-repeat; filter:brightness(0.22) contrast(1.02)}
  #fog{position:fixed; inset:0; z-index:-3; pointer-events:none; mix-blend-mode:screen; opacity:0.9}
  #flash{position:fixed; inset:0; z-index:9999; pointer-events:none; background:#fff; opacity:0; transition:opacity .08s linear}

  header{display:flex; justify-content:space-between; align-items:center; padding:12px 16px}
  .title{font-size:clamp(20px,4.5vw,34px); color:var(--accent); text-shadow:0 0 12px rgba(255,60,0,0.6)}
  .controls{display:flex; gap:8px}
  .btn{background:transparent; border:2px solid rgba(255,255,255,0.06); color:#fff; padding:8px 12px; border-radius:10px; cursor:pointer; font-family:inherit}
  .btn.active{background:var(--accent); color:#000; border-color:transparent}

  .hud{max-width:1100px;margin:12px auto; display:flex; gap:12px; align-items:center; justify-content:center; flex-wrap:wrap}
  .bar{background:#141414; border-radius:10px; padding:8px 10px; border:1px solid rgba(255,255,255,0.03); min-width:160px}
  .label{font-size:12px; color:rgba(255,255,255,0.7)}
  .meter{height:14px; background:#111; border-radius:8px; overflow:hidden; margin-top:6px; border:1px solid rgba(255,255,255,0.02)}
  .fill{height:100%; width:100%; transition:width .32s, background .32s}

  main{max-width:1000px;margin:14px auto 60px;padding:18px}
  #story{background:var(--panel); border:3px solid rgba(255,60,0,0.12); border-radius:12px; padding:20px; font-size:1.05rem; line-height:1.6; box-shadow:0 12px 40px rgba(0,0,0,.6); transition:transform .22s}
  .choices{display:flex; flex-direction:column; gap:12px; margin-top:14px}
  button.choice{background:#121214; color:var(--accent); border:2px solid var(--accent); padding:12px; border-radius:10px; cursor:pointer; font-size:1rem}
  button.choice:hover{background:var(--accent); color:#111; transform:scale(1.03)}

  .muted{color:rgba(255,255,255,0.7); font-size:13px}
  .small{font-size:12px;color:rgba(255,255,255,0.6)}

  /* ritual ui */
  #ritualBox{display:none;margin-top:14px;background:rgba(255,255,255,0.02); padding:12px;border-radius:8px}
  #ritualInput{font-size:1.1rem;padding:10px;width:60%;max-width:280px;border-radius:8px;border:none;text-align:center;background:#000;color:#fff}

  .shake{animation:shake .45s}
  @keyframes shake{0%{transform:translateX(0)}25%{transform:translateX(-12px)}50%{transform:translateX(10px)}75%{transform:translateX(-8px)}100%{transform:translateX(0)}}
  .low-sanity{filter:hue-rotate(-12deg) saturate(.6) blur(.4px)}
  .pulse{animation:pulse 1s infinite}
  @keyframes pulse{0%{box-shadow:0 0 6px rgba(255,0,0,0.18)}50%{box-shadow:0 0 18px rgba(255,0,0,0.44)}100%{box-shadow:0 0 6px rgba(255,0,0,0.18)}}

  .memoriesBox{position:fixed;left:10px;bottom:10px;background:rgba(0,0,0,0.6);padding:10px;border-radius:8px;border:1px solid rgba(255,255,255,0.04);font-size:13px}
  .memoriesBox .title{color:var(--accent);font-size:13px;margin-bottom:6px}

  footer{text-align:center;margin-top:12px;color:rgba(255,255,255,0.45);font-size:13px}

  @media(max-width:640px){main{padding:12px} button.choice{font-size:.95rem;padding:10px}}
</style>
</head>
<body>

<div id="bg"></div>
<canvas id="fog"></canvas>
<div id="flash"></div>

<header>
  <div class="title">🏚️ La Malédiction du Manoir — Long</div>
  <div class="controls">
    <button id="soundBtn" class="btn active">🔊 Son</button>
    <button id="fastBtn" class="btn">⚡ Fast</button>
    <button id="inspectBtn" class="btn">🔎 Inspect</button>
    <button id="memToggle" class="btn">👁️ Souvenirs</button>
  </div>
</header>

<div class="hud">
  <div class="bar" title="Santé mentale">
    <div class="label">Santé mentale</div>
    <div class="meter"><div id="sanityFill" class="fill" style="width:100%; background:linear-gradient(90deg,#00ff99,#0099ff)"></div></div>
    <div class="small" id="sanityText">100 / 100</div>
  </div>

  <div class="bar" title="Curiosité">
    <div class="label">Curiosité</div>
    <div class="meter"><div id="curiosityFill" class="fill" style="width:0%; background:linear-gradient(90deg,#66ccff,#00ddff)"></div></div>
    <div class="small" id="curiosityText">0</div>
  </div>

  <div class="bar" title="Corruption">
    <div class="label">Corruption</div>
    <div class="meter"><div id="corruptionFill" class="fill" style="width:0%; background:linear-gradient(90deg,#ffcc00,#ff4d4d)"></div></div>
    <div class="small" id="corruptionText">0</div>
  </div>
</div>

<main>
  <div id="story" role="main" aria-live="polite"></div>
</main>

<div class="memoriesBox" id="memoriesBox" style="display:none">
  <div class="title">Souvenirs & progression</div>
  <div id="memSummary" class="small">Aucun souvenir.</div>
  <div style="margin-top:8px"><button id="clearMem" class="btn">🗑️ Effacer souvenirs</button></div>
</div>

<footer>Tu peux revenir autant de fois que tu veux. Le manoir se souvient.</footer>

<!-- audio -->
<audio id="amb" src="https://cdn.pixabay.com/download/audio/2022/10/28/audio_5e6f6e7f2c.mp3?filename=horror-ambient-113870.mp3" loop></audio>
<audio id="thunder" src="https://actions.google.com/sounds/v1/weather/thunder_strike.ogg"></audio>
<audio id="shock" src="https://actions.google.com/sounds/v1/foley/metal_drop.ogg"></audio>
<audio id="typeok" src="https://actions.google.com/sounds/v1/cartoon/clang_and_wobble.ogg"></audio>
<audio id="typebad" src="https://actions.google.com/sounds/v1/cartoon/wood_plank_flicks.ogg"></audio>

<script>
/* =========================
   État & mémoire persistante
   ========================= */
let sanity = 100;
let curiosity = 0;
let corruption = 0;
let fastMode = false;
let soundOn = true;
let inspectMode = false;

// Persistent memory in localStorage
const STORAGE_KEY = 'manoir_long_memory_v1';
let memory = JSON.parse(localStorage.getItem(STORAGE_KEY)) || {
  runs:0,
  good:0,
  bad:0,
  maxSanity:100,
  totalCorruption:0,
  discoveredTrueName: false,
  trueName: null,
  souvenirs: []
};

/* DOM refs */
const storyEl = document.getElementById('story');
const sanityFill = document.getElementById('sanityFill');
const sanityText = document.getElementById('sanityText');
const curiosityFill = document.getElementById('curiosityFill');
const curiosityText = document.getElementById('curiosityText');
const corruptionFill = document.getElementById('corruptionFill');
const corruptionText = document.getElementById('corruptionText');

const amb = document.getElementById('amb');
const thunder = document.getElementById('thunder');
const shock = document.getElementById('shock');
const typeok = document.getElementById('typeok');
const typebad = document.getElementById('typebad');
const flash = document.getElementById('flash');
const fogCanvas = document.getElementById('fog');

const soundBtn = document.getElementById('soundBtn');
const fastBtn = document.getElementById('fastBtn');
const inspectBtn = document.getElementById('inspectBtn');
const memToggle = document.getElementById('memToggle');
const memoriesBox = document.getElementById('memoriesBox');
const memSummary = document.getElementById('memSummary');
const clearMem = document.getElementById('clearMem');

/* =========================
   Scenes & extended narrative
   ========================= */

/* We'll include many scenes; choices can carry deltas (curiosity/corruption/sanity)
   Scenes may set flags like ritual:true, mirror:true, discoverTrueName:true
*/

const scenes = {
  start: {
    text: () => getIntroText(),
    sanity: 0,
    fog: 0.18,
    choices: [
      { text: "Pousser la porte et entrer", next: "hall", curiosity: 2 },
      { text: "Faire demi-tour", next: "outside" }
    ]
  },

  outside: {
    text: "Tu pars en courant. La pluie efface tes pas. Peut-être reviendras-tu un autre jour.",
    sanity: +5,
    endText: "Tu as survécu. Mais certains rêves ne s'effacent pas."
  },

  hall: {
    text: "Le hall te reçoit avec des portraits qui semblent respirer. Une chandelle vacille, un courant d'air susurre un nom incompréhensible.",
    sanity: -5, fog:0.25,
    choices:[
      { text: "Suivre l'ombre", next: "corridor", curiosity:1 },
      { text: "Explorer la bibliothèque", next: "library", curiosity:3 },
      { text: "Monter l'escalier", next: "landing", curiosity:2 }
    ]
  },

  corridor: {
    text: "Un couloir étroit, tapis usé, une porte entrouverte laisse échapper une mélodie lointaine.",
    sanity: -8, fog:0.3,
    choices:[
      { text: "Entrer dans la chambre verrouillée", next: "lockedRoom", curiosity:3 },
      { text: "Poursuivre l'ombre", next: "mirrorRoom", curiosity:2 }
    ]
  },

  lockedRoom: {
    text: "La porte cède. Une petite chambre d'enfant, poupées silencieuses et une berceuse à moitié cassée. Un griffonnage au mur: des lettres effacées.",
    sanity: -10, curiosity:+4,
    choices:[
      { text: "Inspecter les poupées", next: "doll", curiosity:2 },
      { text: "Lire le griffonnage", next: "childName", curiosity:3 }
    ]
  },

  doll: {
    text: "Une poupée conserve une clef miniature. Dans le couvercle, une toute petite plaque avec un nom réduit: « ...EREL »",
    sanity:-6, curiosity:+2,
    choices:[
      { text: "Garder la clé", next: "keepKey", curiosity:+1 },
      { text: "La laisser", next: "lockedRoom" }
    ]
  },

  childName: {
    text: "Tu lis les lettres: peut-être le début du vrai nom du manoir. Tu sens une résonance. (Tu pourrais recoller les fragments plus tard.)",
    curiosity:+4,
    choices:[
      { text: "Noter le fragment", next: "lockedRoom" },
      { text: "Ignorer et partir", next: "hall" }
    ]
  },

  keepKey: {
    text: "Tu glisses la clé dans ta poche. Elle semble vibrer. Tu sens qu'elle pourrait ouvrir plus que des serrures.",
    curiosity:+2, corruption:+1,
    choices:[
      { text: "Remonter", next: "landing" },
      { text: "Explorer plus bas (sous-sol)", next: "trapdoor" }
    ]
  },

  landing: {
    text: "À l'étage, les portraits te regardent différemment. Il y a une porte vers le jardin de statues.",
    sanity:-4,
    choices:[
      { text: "Aller au jardin", next: "statues", curiosity:2 },
      { text: "Descendre à la cuisine", next: "kitchen" }
    ]
  },

  statues: {
    text: "Le jardin des statues est figé dans l'obscurité. Certaines t'observent même lorsque tu détournes le regard.",
    sanity:-10, fog:0.35,
    choices:[
      { text: "Approcher la statue centrale", next: "statueCenter", curiosity:3 },
      { text: "Fuir le jardin", next: "landing", sanity:+2 }
    ]
  },

  statueCenter: {
    text: "La statue tient une inscription: 'NOM VÉRITABLE: *XEREL'*. Le souffle du vent révèle d'autres lettres dans la mousse.",
    curiosity:+6,
    // If curiosity high and player has any prior fragment, reveal True Name opportunity
    choices:[
      { text: "Rassembler les fragments", next: "discoverTrueName" },
      { text: "Cacher et partir", next: "landing" }
    ]
  },

  discoverTrueName: {
    text: "En assemblant souvenirs et fragments (poupée, griffonnage, inscription), tu entends le vrai nom murmuré: « XEREL » — le manoir frissonne.",
    curiosity:+0,
    // mark discovery
    onEnter: ()=> { memory.discoveredTrueName = true; memory.trueName = "XEREL"; saveMemory(); addMemoryNote('Découvert: vrai nom XEREL'); },
    choices:[
      { text: "Prononcer le nom à voix basse", next: "sayTrueName", sanity:-6, corruption:+5 },
      { text: "Garder le secret", next: "landing" }
    ]
  },

  sayTrueName: {
    text: "Le nom résonne. Un chant ancien s'élève. L'atmosphère change: soit tu brises quelque chose, soit tu t'enchaînes.",
    sanity:-14, fog:0.6, flash:true,
    choices:[
      { text: "Souffler la lumière (libérer)", next: "chapel", curiosity:+2 },
      { text: "Boire du pouvoir (prendre)", next: "crypt", corruption:+10 }
    ]
  },

  mirrorRoom: {
    text: "La salle du miroir ancien. Le verre n'est pas tout à fait droit — il reflète des possibles.",
    sanity:-12, corruption:+4,
    choices:[
      { text: "Regarder longuement", next: "mirrorGaze", corruption:+3 },
      { text: "Briser le miroir", next: "mirrorBreak", corruption:+12 }
    ]
  },

  mirrorGaze: {
    text: "Ton reflet sourit d'une façon qui n'est pas la tienne. Tu sens ton esprit se fissurer.",
    sanity:-18, corruption:+8,
    endText: "Tu es aspiré. Le miroir garde une âme de plus."
  },

  mirrorBreak: {
    text: "Les éclats chantent. Tu vois des images d'autres vies. Une force t'a choisi.",
    sanity:-20, corruption:+18,
    endText: "Les fragments du monde se recomposent... sans toi."
  },

  library: {
    text: "Un grimoire reposait sur un pupitre. Les pages parlent d'un rituel et d'un mot-clef.",
    sanity:-6, curiosity:+2,
    choices:[
      { text: "Suivre les instructions (rituel)", next: "crypt", ritual:true },
      { text: "Emporter le grimoire (étudier plus tard)", next: "takeBook", curiosity:+3 }
    ]
  },

  takeBook: {
    text: "Tu caches le grimoire dans ta veste. Son odeur colle à ta peau. Parfois il chuchote des indices furtifs.",
    curiosity:+2, corruption:+2,
    choices:[
      { text: "Aller au sous-sol", next: "trapdoor" },
      { text: "Monter à l'étage", next: "landing" }
    ]
  },

  trapdoor: {
    text: "Le tapis cache une trappe. Un air froid suinte: quelque chose attend en bas.",
    sanity:-8, choices:[
      { text: "Descendre", next: "cellar", curiosity:+2 },
      { text: "Fermer et partir", next: "hall", sanity:+1 }
    ]
  },

  cellar: {
    text: "Le sous-sol sent la terre et le temps. Une dalle porte des symboles. Au fond, une chapelle oubliée.",
    sanity:-10, curiosity:+2, corruption:+2,
    choices:[
      { text: "Entrer dans la chapelle", next: "chapel" },
      { text: "Remonter à l'étage", next: "hall", sanity:+1 }
    ]
  },

  chapel: {
    text: "La chapelle est en ruine, mais une lumière persistante y filtre. Cierges éteints, mais le silence y guérit.",
    sanity:+12, fog:0.1,
    choices:[
      { text: "Réciter une prière silencieuse (purifier)", next: "purify", sanity:+10 },
      { text: "Utiliser le rituel pour toi", next: "crypt", ritual:true }
    ]
  },

  purify: {
    text: "Tu chantes doucement. Une brise purifie l'air, et les murs semblent respirer à nouveau.",
    sanity:+18, corruption:-10,
    endText: "La lumière gagne. Tu as brisé une partie de la malédiction. Pour aujourd'hui, le manoir s'apaise."
  },

  crypt: {
    text: "La crypte du gardien: une dalle, des inscriptions, et un cercle ancien. C'est ici que se joue le destin.",
    sanity:-14, fog:0.7,
    choices:[
      { text: "Accomplir le rituel (taper LUMEN)", next: "ritual", ritual:true },
      { text: "Refuser et fuir", next: "escape", sanity:+6 }
    ]
  },

  ritual: {
    // The ritual scene will trigger the mini-game typing "LUMEN"
    text: "Tu prépares le rituel. Pour tenter de libérer, tu dois taper le mot mystique *LUMEN* avant l'échéance.",
    sanity:-8, ritual:true
  },

  ritualSuccess: {
    text: "Les cierges s'élèvent, la pierre exhale une vapeur claire. Les chaînes des âmes se brisent.",
    sanity:+20, curiosity:+6, corruption:-18, endText: "Les esprits sont libérés. Le manoir se souvient de toi comme d'un libérateur."
  },

  ritualFail: {
    text: "Les mots se déchirent; l'obscurité se précipite. Quelque chose te touche et sourit.",
    sanity:-28, corruption:+25, endText: "Le rituel a échoué. Tu as donné au manoir ce dont il avait faim."
  },

  escape: {
    text: "Tu trouves une issue et t'éloignes en courant. Le manoir te regarde partir.",
    sanity:+6, endText: "Tu es libre, mais marqué pour toujours."
  },

  cryptBoss: {
    text: "Une présence ancienne se dresse: le Gardien. Il propose un marché — pouvoir contre âme.",
    sanity:-18, corruption:+10,
    choices:[
      { text: "Accepter le marché", next: "acceptBoss", corruption:+30 },
      { text: "Combattre (sacrifier curiosité)", next: "fightBoss", curiosity:-6 }
    ]
  },

  acceptBoss: {
    text: "Tu acceptes. Une force te traverse; tes mains ne te semblent plus appartenières.",
    sanity:-30, corruption:+40, endText: "Tu deviens le nouvel émissaire du manoir."
  },

  fightBoss: {
    text: "Tu luttes. Ce fut violent. Tu t'en sors mais transformé.",
    sanity:-20, curiosity:-6, endText: "Tu as survécu. Le manoir te respectera — et te craindra."
  },

  insanity: {
    text: "La peur t'a dévoré. Le monde vacille et la nuit devient éternelle.",
    sanity:-100, endText: "Tu as sombré dans la folie. Le manoir a gagné un murmure."
  }
};

/* =========================
   Helpers: memory & HUD
   ========================= */

function saveMemory(){
  localStorage.setItem(STORAGE_KEY, JSON.stringify(memory));
  renderMemorySummary();
}

function addMemoryNote(note){
  const ts = new Date().toISOString();
  memory.souvenirs.push({ note, ts });
  saveMemory();
}

function renderMemorySummary(){
  const s = memory;
  let text = `Parties: ${s.runs} · Bonnes: ${s.good} · Mauvaises: ${s.bad} · Corruption totale: ${s.totalCorruption} · Découvert: ${s.discoveredTrueName ? s.trueName : '—'}`;
  memSummary.innerText = text;
}

/* HUD update */
function updateHUD(){
  sanity = Math.max(0, Math.min(100, Math.round(sanity)));
  curiosity = Math.max(0, Math.min(100, Math.round(curiosity)));
  corruption = Math.max(0, Math.min(100, Math.round(corruption)));

  sanityFill.style.width = sanity + '%';
  sanityText.innerText = `${sanity} / 100`;
  curiosityFill.style.width = curiosity + '%';
  curiosityText.innerText = curiosity;
  corruptionFill.style.width = corruption + '%';
  corruptionText.innerText = corruption;

  if(sanity > 60) sanityFill.style.background = 'linear-gradient(90deg,#00ff99,#0099ff)';
  else if(sanity > 30) sanityFill.style.background = 'linear-gradient(90deg,#ffcc00,#ff6600)';
  else sanityFill.style.background = 'linear-gradient(90deg,#ff0000,#660000)';

  // ambience volume
  if(soundOn){
    const vol = Math.max(0.04, Math.min(0.45, 0.08 + (100 - sanity)/220));
    amb.volume = vol;
    if(amb.paused) amb.play().catch(()=>{/* autoplay blocked */});
  } else {
    amb.pause();
  }

  // visual cue
  if(sanity <= 25) storyEl.classList.add('low-sanity'); else storyEl.classList.remove('low-sanity');
  if(sanity <= 12) storyEl.classList.add('pulse'); else storyEl.classList.remove('pulse');

  // if sanity zero => insanity
  if(sanity <= 0) showEnd('insanity');
}

/* =========================
   Fog animation & effects
   ========================= */

const fctx = fogCanvas.getContext('2d');
let FWIDTH, FHEIGHT, fogParts = [];
function resizeFog(){ fogCanvas.width = window.innerWidth; fogCanvas.height = window.innerHeight; FWIDTH=fogCanvas.width; FHEIGHT=fogCanvas.height; }
window.addEventListener('resize', resizeFog);
function spawnFog(intensity=0.25){
  fogParts = [];
  const count = Math.round(20 + intensity*140);
  for(let i=0;i<count;i++){
    fogParts.push({x:Math.random()*FWIDTH, y:Math.random()*FHEIGHT, r:60+Math.random()*160, vx:(Math.random()-0.5)*0.35, vy:(Math.random()-0.5)*0.35, a:0.04+Math.random()*0.28, hue:190-Math.random()*40});
  }
}
function drawFog(){
  fctx.clearRect(0,0,FWIDTH,FHEIGHT);
  fogParts.forEach(p=>{
    const g = fctx.createRadialGradient(p.x,p.y,p.r*0.1,p.x,p.y,p.r);
    g.addColorStop(0, `hsla(${p.hue},80%,90%,${p.a*0.09})`);
    g.addColorStop(1, `hsla(${p.hue},80%,40%,0)`);
    fctx.fillStyle = g;
    fctx.fillRect(p.x-p.r,p.y-p.r,p.r*2,p.r*2);
    p.x += p.vx; p.y += p.vy;
    if(p.x < -p.r) p.x = FWIDTH + p.r;
    if(p.x > FWIDTH + p.r) p.x = -p.r;
    if(p.y < -p.r) p.y = FHEIGHT + p.r;
    if(p.y > FHEIGHT + p.r) p.y = -p.r;
  });
  requestAnimationFrame(drawFog);
}
resizeFog(); spawnFog(0.25); drawFog();

function triggerFlash(){
  flash.style.opacity = '1';
  thunder.currentTime = 0;
  if(soundOn) thunder.play();
  setTimeout(()=> flash.style.opacity = '0', 120);
}
function triggerShake(){
  storyEl.classList.add('shake');
  shock.currentTime = 0;
  if(soundOn) shock.play();
  setTimeout(()=> storyEl.classList.remove('shake'), 460);
}

/* =========================
   Scene renderer & navigation
   ========================= */

let currentScene = 'start';
function getIntroText(){
  if(memory.runs === 0) return "La pluie fouette la façade. Devant toi, un manoir au nom perdu attend. Entrer ? Il t'appellera peut-être...";
  if(memory.discoveredTrueName) return `Le manoir te reconnaît. Son vrai nom ( ${memory.trueName} ) résonne dans les pierres.`;
  if(memory.totalCorruption > 300) return "Les nuits auprès du manoir ont laissé des traces: un frisson familier te parcourt.";
  if(memory.good >= 2) return "Le manoir semble plus clément — ou peut-être te teste-t-il. Tu reviens, déterminé.";
  return "Tu reviens. Chaque retour t'apprend quelque chose. Le manoir a changé, et toi aussi.";
}

function applySceneDeltas(scene){
  if(!scene) return;
  if(typeof scene.sanity === 'number') sanity += scene.sanity;
  if(typeof scene.curiosity === 'number') curiosity += scene.curiosity;
  if(typeof scene.corruption === 'number') corruption += scene.corruption;
}

function showScene(name, opts = {}){
  const scene = scenes[name];
  if(!scene) return;
  currentScene = name;

  // apply base deltas
  if(typeof scene.sanity === 'number') sanity += scene.sanity;
  if(typeof scene.curiosity === 'number') curiosity += scene.curiosity;
  if(typeof scene.corruption === 'number') corruption += scene.corruption;

  updateHUD();

  // visual flags
  if(scene.flash) triggerFlash();
  if(scene.shake) triggerShake();
  spawnFog(scene.fog || 0.25);

  // handle onEnter function if present
  if(scene.onEnter) scene.onEnter();

  // build html
  let text = typeof scene.text === 'function' ? scene.text() : scene.text;
  let html = `<p>${text}</p>`;

  if(scene.choices){
    html += `<div class="choices">`;
    scene.choices.forEach(choice=>{
      // add data attributes for deltas
      const san = (choice.sanity ? ` data-sanity="${choice.sanity}"` : '');
      const cur = (choice.curiosity ? ` data-curiosity="${choice.curiosity}"` : '');
      const cor = (choice.corruption ? ` data-corruption="${choice.corruption}"` : '');
      html += `<button class="choice" ${san}${cur}${cor} data-next="${choice.next}" onclick="choiceClick(event)">${choice.text}</button>`;
    });
    html += `</div>`;
    storyEl.innerHTML = html;

    if(fastMode){
      setTimeout(()=> {
        const first = storyEl.querySelector('.choice');
        if(first) first.click();
      }, 600);
    }

  } else if(scene.endText){
    // terminal scene
    html += `<div style="margin-top:14px" class="muted">${scene.endText}</div>`;
    html += `<div class="choices" style="margin-top:12px"><button class="choice" onclick="finishRun('${name}')">Fin</button></div>`;
    storyEl.innerHTML = html;
  } else if(scene.ritual){
    // show ritual mini-game
    html += `<div id="ritualBox"><div class="muted">Tape le mot <b>LUMEN</b> dans la boîte ci-dessous avant la fin du temps imparti.</div>
             <div style="margin-top:10px"><input id="ritualInput" placeholder="Tape ici..." maxlength="20" autocapitalize="off" autocomplete="off"></div>
             <div class="muted small" style="margin-top:8px">Temps: <span id="ritualTimer">7</span>s</div></div>`;
    storyEl.innerHTML = html;
    startRitualMiniGame();
  } else {
    storyEl.innerHTML = html;
  }

  updateHUD();
}

/* handle choice click */
function choiceClick(e){
  const btn = e.currentTarget;
  const next = btn.getAttribute('data-next');
  const s = parseInt(btn.getAttribute('data-sanity') || '0', 10);
  const c = parseInt(btn.getAttribute('data-curiosity') || '0', 10);
  const corr = parseInt(btn.getAttribute('data-corruption') || '0', 10);
  if(s) sanity += s;
  if(c) curiosity += c;
  if(corr) corruption += corr;
  updateHUD();

  // special: if next is 'discoverTrueName' we mark memory
  if(next === 'discoverTrueName'){
    memory.discoveredTrueName = true;
    memory.trueName = memory.trueName || 'XEREL';
    addMemoryNote('Découverte du vrai nom: ' + memory.trueName);
  }

  // ritual handling
  if(next === 'ritual'){
    showScene('ritual');
    return;
  }

  // show next
  showScene(next);
}

/* finish run: update memory and present end */
function finishRun(sceneName){
  // classify run as good or bad based on corruption and scene
  memory.runs++;
  memory.totalCorruption += corruption;
  memory.maxSanity = Math.max(memory.maxSanity, sanity);

  // determine good vs bad
  const goodEndings = ['purify','ritualSuccess','ritualSuccess','escape','purify','chapel','ritualSuccess'];
  const isGood = (sanity > 30 && corruption < 40 && (goodEndings.includes(sceneName) || sceneName === 'ritualSuccess' || sceneName === 'purify'));
  if(isGood){
    memory.good++;
    addMemoryNote('Fin positive obtenue');
  } else {
    memory.bad++;
    addMemoryNote('Fin sombre obtenue');
  }

  saveMemory();

  // show a final cinematic text
  let finalText = '';
  if(memory.discoveredTrueName && sceneName === 'ritualSuccess'){
    finalText = `En ayant prononcé le vrai nom (${memory.trueName}), tu as accompli un acte rare: la libération totale. Le manoir s'effondre en paix. FIN SECRÈTE.`;
    addMemoryNote('Fin secrète: vraie libération');
  } else if(isGood){
    finalText = 'Tu as quitté le manoir, la lumière au coeur. Le monde est moins lourd ce soir.';
  } else if(sanity <= 0){
    finalText = 'Ta raison a succombé. Le manoir a gagné une nouvelle voix.';
  } else {
    finalText = 'Le manoir se referme sur ce qui reste de toi. Une cicatrice parmi d\'autres.';
  }

  // present cinematic
  storyEl.innerHTML = `<p style="font-size:1.1rem">${finalText}</p>
    <div style="margin-top:12px" class="muted">Souvenirs acquis: ${memory.souvenirs.length}</div>
    <div class="choices" style="margin-top:14px">
      <button class="choice" onclick="restartFromEnd()">🔁 Rejouer (revenir au début)</button>
      <button class="choice" onclick="exportMemory()">📋 Exporter souvenirs</button>
    </div>`;

  // save memory already done
}

/* restart helpers */
function restartFromEnd(){
  sanity = 100; curiosity = 0; corruption = 0;
  updateHUD(); spawnFog(0.25); showScene('start');
}

/* =========================
   Ritual mini-game: typing "LUMEN"
   ========================= */

let ritualActive = false;
let ritualTimer = null;

function startRitualMiniGame(){
  ritualActive = true;
  const timeLimit = 7000; // ms
  const input = document.getElementById('ritualInput');
  const timerSpan = document.getElementById('ritualTimer');
  input.value = '';
  input.focus();

  let remaining = Math.ceil(timeLimit / 1000);
  timerSpan.innerText = remaining;

  ritualTimer = setInterval(()=>{
    remaining--;
    if(remaining <= 0){
      clearInterval(ritualTimer);
      if(ritualActive) ritualFail();
    } else {
      timerSpan.innerText = remaining;
    }
  }, 1000);

  const deadline = Date.now() + timeLimit;
  const checker = setInterval(()=>{
    if(!ritualActive){ clearInterval(checker); return; }
    if(Date.now() >= deadline){
      clearInterval(checker);
      if(ritualActive) ritualFail();
    }
  }, 150);

  const onInput = (e) => {
    const v = input.value.trim().toLowerCase();
    if(v === 'lumen'){
      ritualActive = false;
      clearInterval(ritualTimer);
      input.removeEventListener('input', onInput);
      typeok.currentTime = 0; if(soundOn) typeok.play();
      ritualSuccess();
    }
  };
  input.addEventListener('input', onInput);

  input.addEventListener('keydown', (ev)=>{
    if(ev.key === 'Enter'){
      if(input.value.trim().toLowerCase() === 'lumen'){
        ritualActive = false;
        clearInterval(ritualTimer);
        input.removeEventListener('input', onInput);
        typeok.currentTime = 0; if(soundOn) typeok.play();
        ritualSuccess();
      } else {
        typebad.currentTime = 0; if(soundOn) typebad.play();
      }
    }
  });

  // small flash + fog intensify
  triggerFlash();
  spawnFog(0.9);
}

function ritualSuccess(){
  ritualActive = false;
  spawnFog(0.12);
  sanity += 20; curiosity += 6; corruption = Math.max(0, corruption - 18);
  updateHUD();
  addMemoryNote('Rituel réussi');
  showScene('ritualSuccess');
}

function ritualFail(){
  ritualActive = false;
  spawnFog(0.85);
  sanity -= 28; corruption += 25;
  updateHUD();
  addMemoryNote('Rituel raté');
  showScene('ritualFail');
}

/* =========================
   Memory UI & utilities
   ========================= */

function saveMemory(){
  localStorage.setItem(STORAGE_KEY, JSON.stringify(memory));
}

function exportMemory(){
  const txt = JSON.stringify(memory, null, 2);
  navigator.clipboard.writeText(txt).then(()=> alert('Souvenirs copiés dans le presse-papiers.'));
}

function toggleMemories(){
  memoriesBox.style.display = (memoriesBox.style.display === 'none' ? 'block' : 'none');
  renderMemorySummary();
}

clearMem.addEventListener('click', ()=>{
  if(confirm('Effacer toutes les données locales (souvenirs) ?')){
    localStorage.removeItem(STORAGE_KEY);
    memory = { runs:0, good:0, bad:0, maxSanity:100, totalCorruption:0, discoveredTrueName:false, trueName:null, souvenirs:[] };
    renderMemorySummary();
    alert('Souvenirs effacés.');
  }
});

memToggle.addEventListener('click', toggleMemories);

/* render summary */
function renderMemorySummary(){
  memSummary.innerText = `Parties: ${memory.runs} · Bonnes: ${memory.good} · Mauvaises: ${memory.bad} · Corruption: ${memory.totalCorruption} · TrueName: ${memory.discoveredTrueName ? memory.trueName : '—'}`;
}

/* =========================
   Controls binding & init
   ========================= */

soundBtn.addEventListener('click', ()=>{
  soundOn = !soundOn;
  soundBtn.classList.toggle('active', soundOn);
  soundBtn.innerText = soundOn ? '🔊 Son' : '🔇 Muet';
  updateHUD();
});

fastBtn.addEventListener('click', ()=>{
  fastMode = !fastMode;
  fastBtn.classList.toggle('active', fastMode);
  fastBtn.innerText = fastMode ? '⚡ Fast (ON)' : '⚡ Fast';
});

inspectBtn.addEventListener('click', ()=>{
  inspectMode = !inspectMode;
  inspectBtn.classList.toggle('active', inspectMode);
  if(inspectMode){
    alert(`DEBUG — curiosity: ${curiosity}, corruption: ${corruption}, sanity: ${sanity}`);
  }
});

/* keyboard: Enter triggers first choice; background click triggers thunder flash for fun */
window.addEventListener('keydown', (e) => {
  if(e.key === 'Enter' && !ritualActive){
    const first = storyEl.querySelector('.choice');
    if(first) first.click();
  }
});

document.getElementById('bg').addEventListener('click', ()=> { triggerFlash(); });

/* initial */
updateHUD();
renderMemorySummary();
spawnFog(0.25);
showScene('start');
if(soundOn){ amb.volume = 0.12; amb.play().catch(()=>{/* autoplay may be blocked */}); }

</script>
</body>
</html>
