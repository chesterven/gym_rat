
<html lang="es">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<meta name="theme-color" content="#0E1113">
<meta name="mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
<meta name="format-detection" content="telephone=no">
<title>Gym Tracker</title>
<link rel="icon" href="data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 100 100'><rect width='100' height='100' rx='22' fill='%230E1113'/><text y='72' x='50' font-size='58' text-anchor='middle' fill='%23C6FF3D' font-family='sans-serif' font-weight='bold'>G</text></svg>">
<style>
/* ============================================================
   Identidad: sala de pesas. Casi negro, un solo acento lima,
   números pesados y tabulares. Sin fuentes externas: la app
   debe verse igual sin conexión.

   Reglas de layout que evitan solapamientos:
   --bottomh  = altura REAL medida de la barra inferior
                (nav + temporizador + botón finalizar).
   --kb       = altura del teclado en pantalla (visualViewport).
   Todo lo flotante se ancla a --bottomh, nunca a píxeles fijos.
   ============================================================ */
:root{
  --accent:#C6FF3D; --on-accent:#101214;
  --bg:#0E1113; --card:#181B1F; --card-2:#20252A;
  --fg:#EDF1F2; --muted:#8B9499; --line:#2A3036;
  --danger:#FF6B6B; --radius:18px;
  --safe-b: env(safe-area-inset-bottom, 0px);
  --safe-l: env(safe-area-inset-left, 0px);
  --safe-r: env(safe-area-inset-right, 0px);
  --chip-bg:#242A2F; --chip-fg:#8B9499;
  --accent-soft:rgba(198,255,61,.16); --accent-soft2:rgba(198,255,61,.22);
  --accent-line:rgba(198,255,61,.55); --accent-line2:rgba(198,255,61,.34);
  --accent-timer:#1B2318; --danger-soft:rgba(255,107,107,.14); --danger-line:rgba(255,107,107,.42);
  --bottomh: 118px;   /* ambas las recalcula measureChrome() */
  --topbarh: 58px;
  --kb: 0px;
  /* Área realmente visible. En iOS, vh ignora el teclado: por eso los
     sheets se medían más altos que la pantalla y su parte superior
     (con el campo enfocado) quedaba fuera de vista. */
  --vvh: 100vh;
  --vvtop: 0px;
  color-scheme: dark;
}
:root[data-theme="light"]{
  --accent:#4C6B00; --on-accent:#FFFFFF;
  --bg:#F3F5F0; --card:#FFFFFF; --card-2:#EDEFEA;
  --fg:#151815; --muted:#66706B; --line:#DFE3DB;
  --chip-bg:#E7EAE2; --chip-fg:#5C665F;
  --accent-soft:rgba(76,107,0,.13); --accent-soft2:rgba(76,107,0,.20);
  --accent-line:rgba(76,107,0,.55); --accent-line2:rgba(76,107,0,.32);
  --accent-timer:#F0F4E6; --danger-soft:rgba(197,48,48,.12); --danger-line:rgba(197,48,48,.38);
  color-scheme: light;
}
*{box-sizing:border-box;-webkit-tap-highlight-color:transparent}
html,body{margin:0;padding:0;min-height:100%}
body{
  background:var(--bg); color:var(--fg);
  font-family:-apple-system,BlinkMacSystemFont,"Segoe UI",Roboto,"Helvetica Neue",Arial,sans-serif;
  font-size:15px; line-height:1.45;
  overscroll-behavior-y:none; touch-action:manipulation;
  -webkit-font-smoothing:antialiased; text-rendering:optimizeLegibility;
  padding-left:var(--safe-l); padding-right:var(--safe-r);
}
#app{max-width:560px;margin:0 auto;position:relative}
button,input,select,textarea{font-family:inherit;color:inherit}
button{background:none;border:0;cursor:pointer;touch-action:manipulation;
  -webkit-user-select:none;user-select:none;-webkit-touch-callout:none}
h1,h2,h3{margin:0;letter-spacing:-.6px;overflow-wrap:anywhere}
a{color:var(--accent)}

/* ---------- estructura ---------- */
.screen{padding:8px 16px calc(var(--bottomh) + 28px)}
.topbar{display:flex;align-items:center;gap:10px;padding:12px 16px 6px;position:sticky;top:0;z-index:20;
  background:var(--bg);min-height:56px}
.topbar h1{font-size:23px;font-weight:800;overflow:hidden;text-overflow:ellipsis;white-space:nowrap}
.topbar .sub{font-size:11px;color:var(--muted);font-weight:500;letter-spacing:0;
  overflow:hidden;text-overflow:ellipsis;white-space:nowrap}
.iconbtn{width:42px;height:42px;flex:none;border-radius:12px;display:grid;place-items:center;
  color:var(--fg);font-size:18px}
.iconbtn:active{background:var(--card-2)}
.iconbtn.sm{width:36px;height:36px;font-size:15px}
.eyebrow{font-size:11px;font-weight:800;letter-spacing:1.4px;text-transform:uppercase;color:var(--muted);margin:26px 4px 10px}
.eyebrow.accent{color:var(--accent)}

/* ---------- tarjetas ---------- */
.card{background:var(--card);border-radius:var(--radius);padding:16px;border:1.5px solid transparent}
.card+.card{margin-top:10px}
.card.hl{border-color:var(--accent-line2)}
.card.solid{border-color:var(--accent)}
.row{display:flex;align-items:center;gap:10px;min-width:0}
.grow{flex:1;min-width:0}
.ell{overflow:hidden;text-overflow:ellipsis;white-space:nowrap}
.grid2{display:grid;grid-template-columns:1fr 1fr;gap:10px}
.stat{background:var(--card);border-radius:var(--radius);padding:14px 16px;min-width:0}
.stat .v{font-size:22px;font-weight:800;letter-spacing:-.8px;font-variant-numeric:tabular-nums;
  overflow:hidden;text-overflow:ellipsis;white-space:nowrap}
.stat .v.accent{color:var(--accent)}
.stat .l{font-size:11.5px;color:var(--muted);margin-top:1px;overflow:hidden;text-overflow:ellipsis;white-space:nowrap}
.muted{color:var(--muted)}
.small{font-size:12.5px}
.tiny{font-size:11.5px}
.b7{font-weight:700}
.b8{font-weight:800}
.nums{font-variant-numeric:tabular-nums}
.chip{display:inline-flex;align-items:center;padding:5px 10px;border-radius:9px;font-size:11.5px;font-weight:700;
  background:var(--chip-bg);color:var(--chip-fg);white-space:nowrap}
.chip.on{background:var(--accent-soft);color:var(--accent)}
.wrap{display:flex;flex-wrap:wrap;gap:6px}
.sep{height:1px;background:var(--line);margin:10px 0}

/* ---------- botones ---------- */
.btn{display:flex;align-items:center;justify-content:center;gap:8px;width:100%;min-height:54px;border-radius:15px;
  font-weight:800;letter-spacing:.5px;font-size:15px;background:var(--accent);color:var(--on-accent);padding:0 16px;
  transition:transform .08s ease}
.btn:active{transform:scale(.98)}
.btn.ghost{background:transparent;border:1.5px solid var(--line);color:var(--fg)}
.btn.grey{background:var(--card-2);color:var(--fg)}
.btn.danger{background:transparent;color:var(--danger);border:1.5px solid var(--danger-line)}
.btn[disabled]{opacity:.5;pointer-events:none}
.pill{min-height:40px;padding:0 14px;border-radius:11px;background:var(--card-2);color:var(--fg);
  font-size:13px;font-weight:700;white-space:nowrap;display:inline-flex;align-items:center;flex:none}
.pill.on{background:var(--accent);color:var(--on-accent)}
.hscroll{display:flex;align-items:center;gap:8px;overflow-x:auto;padding:2px 16px;margin:0 -16px;
  scrollbar-width:none;-webkit-overflow-scrolling:touch}
.hscroll::-webkit-scrollbar{display:none}
.hscroll.flush{margin:0;padding:0}

/* ---------- formularios (16px evita el zoom automático de iOS) ---------- */
label.f{display:block;margin-bottom:12px;min-width:0}
label.f>span{display:block;font-size:11px;font-weight:800;letter-spacing:1.1px;text-transform:uppercase;
  color:var(--muted);margin-bottom:5px}
input.t,select.t,textarea.t{width:100%;font-size:16px;background:var(--card-2);border:0;border-radius:12px;
  padding:13px 14px;outline:none;-webkit-appearance:none;appearance:none}
select.t{background-image:linear-gradient(45deg,transparent 50%,var(--muted) 50%),
  linear-gradient(135deg,var(--muted) 50%,transparent 50%);
  background-position:calc(100% - 18px) 21px, calc(100% - 13px) 21px;
  background-size:5px 5px,5px 5px;background-repeat:no-repeat;padding-right:38px}
input.t:focus,select.t:focus,textarea.t:focus{box-shadow:0 0 0 2px var(--accent)}
textarea.t{resize:vertical;min-height:76px;line-height:1.4}

/* ---------- stepper (una mano) ---------- */
.stepper{min-width:0}
.stepper .cap{font-size:10px;font-weight:800;letter-spacing:1.1px;text-transform:uppercase;color:var(--muted);
  margin-bottom:5px;overflow:hidden;text-overflow:ellipsis;white-space:nowrap}
.stepper .body{display:flex;align-items:center;gap:4px;min-width:0}
.stepper .rnd{width:46px;height:46px;flex:none;border-radius:50%;background:var(--card-2);font-size:22px;
  font-weight:600;display:grid;place-items:center;color:var(--fg)}
.stepper .rnd:active{background:var(--accent);color:var(--on-accent)}
.stepper input{flex:1;min-width:0;width:100%;text-align:center;background:transparent;border:0;outline:none;
  font-size:25px;font-weight:800;letter-spacing:-1px;font-variant-numeric:tabular-nums;padding:6px 0;
  -webkit-appearance:none;appearance:none}
.stepper .unit{font-size:11px;color:var(--muted);font-weight:700;flex:none}
@media (max-width:359px){
  .stepper .rnd{width:42px;height:42px}
  .stepper input{font-size:22px}
  .grid2{gap:8px}
}

/* ---------- series ---------- */
.setrow{display:flex;align-items:center;gap:12px;background:var(--card);border-radius:14px;padding:12px 14px;
  border:1.5px solid transparent;min-width:0}
.setrow.pending{opacity:.5}
.setrow.done{border-color:var(--accent-line2)}
.badge{width:32px;height:32px;flex:none;border-radius:10px;display:grid;place-items:center;
  font-weight:800;font-size:13px;background:var(--card-2)}
.badge.on{background:var(--accent);color:var(--on-accent)}
.badge.ok{background:var(--accent-soft);color:var(--accent)}

/* ---------- barra inferior ---------- */
.bottom{position:fixed;left:0;right:0;bottom:0;z-index:40;background:var(--bg);
  padding-bottom:var(--safe-b);max-width:560px;margin:0 auto}
.nav{display:flex;background:var(--card);border-top:1px solid var(--line)}
.nav button.on .ic{transform:translateY(-1px)}
.nav button{flex:1;min-width:0;padding:8px 2px 9px;display:grid;justify-items:center;gap:2px;
  color:var(--muted);font-size:10px;font-weight:700;min-height:52px}
.nav button.on{color:var(--accent)}
.nav .ic{font-size:19px;line-height:1.1}
.docked{padding:8px 16px 6px;display:grid;gap:8px}
.docked:empty{display:none}

/* ---------- temporizador ---------- */
.timer{background:var(--accent-timer);border:1.5px solid var(--accent-line);
  border-radius:16px;padding:9px 12px}
.timer .clock{font-size:31px;font-weight:900;letter-spacing:-1.5px;color:var(--accent);
  font-variant-numeric:tabular-nums;line-height:1.15}
.timer .bar{height:5px;background:var(--accent-soft2);border-radius:3px;overflow:hidden;margin-top:6px}
.timer .bar i{display:block;height:100%;background:var(--accent);width:0;transition:width .25s linear}
.tact{display:grid;justify-items:center;gap:1px;padding:4px 6px;font-size:10px;color:var(--muted);
  font-weight:700;min-width:44px;border-radius:10px}
.tact:active{background:var(--accent-soft)}
.tact .ic{font-size:17px;color:var(--fg);line-height:1.2}
@keyframes flash{0%,100%{opacity:1}50%{opacity:.35}}
.timer.over{animation:flash .7s ease-in-out 3}

/* ---------- flotantes anclados a la barra real ---------- */
.fabwrap{position:fixed;left:0;right:0;bottom:calc(var(--bottomh) + 10px);z-index:35;
  max-width:560px;margin:0 auto;padding:0 16px;display:flex;justify-content:flex-end;pointer-events:none}
.fab{pointer-events:auto;background:var(--accent);color:var(--on-accent);border-radius:16px;
  min-height:52px;padding:0 20px;font-weight:800;font-size:14px;display:inline-flex;align-items:center;gap:7px;
  box-shadow:0 6px 22px rgba(0,0,0,.34)}
.fab:active{transform:scale(.97)}
.toast{position:fixed;left:16px;right:16px;bottom:calc(var(--bottomh) + 10px);z-index:80;
  background:var(--accent);color:var(--on-accent);border-radius:14px;padding:13px 16px;font-weight:700;
  max-width:528px;margin:0 auto;box-shadow:0 8px 24px rgba(0,0,0,.35);animation:up .2s ease-out}
.toast.plain{background:var(--card-2);color:var(--fg)}
/* Con el teclado abierto no compiten por el mismo espacio. */
body.kb .bottom,body.kb .fabwrap{display:none}
/* overflow:hidden no basta en iOS: el documento sigue desplazándose para
   revelar el campo enfocado y arrastra los elementos fijos fuera de vista. */
body.locked{position:fixed;left:0;right:0;width:100%;overflow:hidden}

/* ---------- sheets y diálogos ---------- */
/* El scrim se ancla al viewport VISIBLE (alto y desplazamiento reales),
   no al de la página: así la hoja siempre cabe entre el borde superior
   y el teclado, en lugar de desbordarse hacia arriba. */
.scrim{position:fixed;left:0;right:0;top:var(--vvtop);height:var(--vvh);background:rgba(0,0,0,.6);
  z-index:60;display:flex;align-items:flex-end;justify-content:center}
.scrim.center{align-items:center;padding:20px}
.sheet{background:var(--bg);width:100%;max-width:560px;border-radius:22px 22px 0 0;max-height:100%;
  overflow-y:auto;-webkit-overflow-scrolling:touch;overscroll-behavior:contain;
  padding:14px 16px calc(24px + var(--safe-b));animation:up .22s ease-out}
.sheet.dialog{border-radius:20px;max-width:400px;padding:20px;animation:none;max-height:100%}
@keyframes up{from{transform:translateY(18px);opacity:.5}to{transform:none;opacity:1}}
.grab{width:38px;height:4px;border-radius:2px;background:var(--line);margin:0 auto 14px;flex:none}
.sheet h3{font-size:19px;font-weight:800;margin-bottom:14px}
.opt{display:flex;align-items:center;gap:12px;padding:15px 4px;border-bottom:1px solid var(--line);
  width:100%;text-align:left;min-height:54px;font-size:15px}
.opt:last-child{border-bottom:0}
.opt.danger{color:var(--danger)}

/* ---------- misc ---------- */
.empty{text-align:center;padding:46px 24px;color:var(--muted)}
.empty .ic{font-size:40px;opacity:.5}
.empty h3{font-size:17px;color:var(--fg);margin:12px 0 6px}
.banner{background:var(--danger-soft);border:1px solid var(--danger-line);color:var(--fg);border-radius:12px;
  padding:11px 13px;font-size:12.5px;line-height:1.45;margin-bottom:12px}
svg.chart{width:100%;display:block;overflow:visible}
@media (prefers-reduced-motion:reduce){*{animation:none!important;transition:none!important}}
</style>
</head>
<body>
<div id="app"><div id="view"></div></div>
<div class="bottom"><div id="dock" class="docked"></div><nav class="nav" id="nav"></nav></div>
<div id="layer"></div>
<script>
"use strict";
/* ================================================================
   CAPA 1 — ALMACENAMIENTO LOCAL
   Mismo modelo relacional que la versión Flutter: cada "tabla" es
   un array de filas con id propio. Se serializa completo en
   localStorage tras cada escritura, así nada vive solo en RAM.
   ================================================================ */
const LS_KEY = 'gym_tracker_v1';
const TABLES = ['exercises','routines','routine_days','routine_exercises',
                'workouts','workout_exercises','workout_sets',
                'body_measurements','personal_records'];

let store = null;
let persistent = true;   // false => el navegador bloqueó el almacenamiento

const db = () => store.data;
const now = () => Date.now();

function emptyStore(){
  const data = {schema_version:1, seq:{},
    settings:{theme:'dark', sound:true, vibration:true, defaultRest:120, unit:'lb'}};
  TABLES.forEach(t => data[t] = []);
  return {app:'gym_tracker', schema_version:1, exported_at:new Date().toISOString(), data};
}

function nextId(table){
  const seq = db().seq;
  seq[table] = (seq[table] || 0) + 1;
  return seq[table];
}
function insert(table, row){
  row.id = nextId(table);
  db()[table].push(row);
  return row;
}
function byId(table, id){ return db()[table].find(r => r.id === id) || null; }
function remove(table, pred){
  const keep = db()[table].filter(r => !pred(r));
  db()[table] = keep;
}

function save(){
  if(!persistent) return;
  try{
    store.exported_at = new Date().toISOString();
    localStorage.setItem(LS_KEY, JSON.stringify(store));
  }catch(e){
    persistent = false;
    console.warn('Almacenamiento no disponible:', e);
  }
}

function load(){
  try{
    const raw = localStorage.getItem(LS_KEY);
    if(raw){
      const parsed = JSON.parse(raw);
      if(parsed && parsed.data){
        store = parsed;
        TABLES.forEach(t => { if(!Array.isArray(db()[t])) db()[t] = []; });
        if(!db().settings) db().settings = {theme:'dark',sound:true,vibration:true,defaultRest:120,unit:'lb'};
        // Instalaciones anteriores a las unidades escribieron sus números
        // pensando en kilos: se respetan tal cual.
        if(!db().settings.unit) db().settings.unit = 'kg';
        if(!db().seq) db().seq = {};
        return;
      }
    }
  }catch(e){
    persistent = false;
    console.warn('No se pudo leer el almacenamiento:', e);
  }
  store = emptyStore();
  seed();
  save();
  // Si save() falló, el aviso de "modo temporal" se muestra en el dashboard.
  if(persistent){
    try{ localStorage.getItem(LS_KEY); }catch(e){ persistent = false; }
  }
}

/* ================================================================
   Biblioteca base de ejercicios + rutina de ejemplo
   ================================================================ */
const MUSCLES = ['Pecho','Espalda','Hombros','Piernas','Bíceps','Tríceps','Abdomen','Cardio','Otro'];
const EQUIPMENT = ['Barra','Mancuernas','Máquina','Polea','Peso corporal','Libre'];
const EXTYPES = ['Compuesto','Aislamiento','Cardio','Movilidad'];

const SEED_EXERCISES = [
  ['Press banca','Pecho','Barra','Compuesto','Escápulas retraídas, baja al esternón y empuja sin rebotar.'],
  ['Press inclinado','Pecho','Barra','Compuesto',''],
  ['Press declinado','Pecho','Barra','Compuesto',''],
  ['Press con mancuernas','Pecho','Mancuernas','Compuesto',''],
  ['Press inclinado con mancuernas','Pecho','Mancuernas','Compuesto',''],
  ['Aperturas con mancuernas','Pecho','Mancuernas','Aislamiento',''],
  ['Fondos','Pecho','Peso corporal','Compuesto',''],
  ['Dominadas','Espalda','Peso corporal','Compuesto',''],
  ['Jalón al pecho','Espalda','Polea','Compuesto',''],
  ['Remo con barra','Espalda','Barra','Compuesto',''],
  ['Remo con mancuerna','Espalda','Mancuernas','Compuesto',''],
  ['Remo en máquina','Espalda','Máquina','Compuesto',''],
  ['Peso muerto','Espalda','Barra','Compuesto',''],
  ['Press militar','Hombros','Barra','Compuesto',''],
  ['Press con mancuernas hombro','Hombros','Mancuernas','Compuesto',''],
  ['Elevaciones laterales','Hombros','Mancuernas','Aislamiento',''],
  ['Elevaciones frontales','Hombros','Mancuernas','Aislamiento',''],
  ['Face pulls','Hombros','Polea','Aislamiento',''],
  ['Sentadilla','Piernas','Barra','Compuesto',''],
  ['Prensa','Piernas','Máquina','Compuesto',''],
  ['Peso muerto rumano','Piernas','Barra','Compuesto',''],
  ['Extensión de cuádriceps','Piernas','Máquina','Aislamiento',''],
  ['Curl femoral','Piernas','Máquina','Aislamiento',''],
  ['Hip thrust','Piernas','Barra','Compuesto',''],
  ['Pantorrillas','Piernas','Máquina','Aislamiento',''],
  ['Curl con barra','Bíceps','Barra','Aislamiento',''],
  ['Curl con mancuernas','Bíceps','Mancuernas','Aislamiento',''],
  ['Curl martillo','Bíceps','Mancuernas','Aislamiento',''],
  ['Extensión de tríceps en polea','Tríceps','Polea','Aislamiento',''],
  ['Press francés','Tríceps','Barra','Aislamiento',''],
  ['Fondos en paralelas','Tríceps','Peso corporal','Compuesto',''],
  ['Plancha','Abdomen','Peso corporal','Aislamiento',''],
  ['Elevación de piernas','Abdomen','Peso corporal','Aislamiento',''],
  ['Crunch en polea','Abdomen','Polea','Aislamiento','']
];

const LB2KG = lb => Math.round((lb / 2.2046226218) * 1000) / 1000;

function seed(){
  const idOf = {};
  SEED_EXERCISES.forEach(([name,group,equipment,type,instructions]) => {
    const ex = insert('exercises',{name,muscle_group:group,equipment,type,
      description:'',instructions,image:null,is_custom:false});
    idOf[name] = ex.id;
  });

  const routine = insert('routines',{name:'Rutina Hipertrofia',
    notes:'Rutina de ejemplo. Edítala o crea la tuya.',is_active:true,
    created_at:now(),updated_at:now()});

  // Pesos escritos en libras y guardados en kilos.
  const plan = [
    [1,'Pecho + Tríceps',false,[['Press banca',4,8,135,120,8],['Press inclinado con mancuernas',3,10,50,90,8],
      ['Aperturas con mancuernas',3,12,25,60,8],['Extensión de tríceps en polea',3,12,50,60,8]]],
    [2,'Espalda + Bíceps',false,[['Dominadas',4,8,0,120,8],['Remo con barra',4,8,115,120,8],
      ['Jalón al pecho',3,12,100,90,8],['Curl con barra',3,12,45,60,8]]],
    [3,'Descanso',true,[]],
    [4,'Piernas',false,[['Sentadilla',4,8,185,180,8],['Prensa',4,10,270,120,8],
      ['Peso muerto rumano',3,10,135,120,8],['Curl femoral',3,12,80,60,8],['Pantorrillas',4,15,90,45,8]]],
    [5,'Hombros + Abdomen',false,[['Press militar',4,8,95,120,8],['Elevaciones laterales',4,12,20,60,8],
      ['Face pulls',3,15,45,60,8]]],
    [6,'Descanso',true,[]],
    [7,'Descanso',true,[]]
  ];

  plan.forEach(([weekday,title,rest,items],di) => {
    const day = insert('routine_days',{routine_id:routine.id,weekday,title,is_rest:rest,order_index:di});
    items.forEach(([exName,sets,reps,weight,restSec,rpe],i) => {
      insert('routine_exercises',{routine_day_id:day.id,exercise_id:idOf[exName],order_index:i,
        target_sets:sets,target_reps:reps,target_reps_max:null,target_weight:LB2KG(weight),
        rest_seconds:restSec,target_rpe:rpe,notes:''});
    });
  });
}

/* ================================================================
   CAPA 2 — REPOSITORIOS
   Las vistas nunca tocan los arrays directamente; pasan por aquí.
   Sustituir esto por llamadas a una API es lo único que haría falta
   para sincronizar en la nube más adelante.
   ================================================================ */
const ExerciseRepo = {
  all(search='', group='Todos'){
    const q = search.trim().toLowerCase();
    return db().exercises
      .filter(e => (!q || e.name.toLowerCase().includes(q)) &&
                   (group === 'Todos' || e.muscle_group === group))
      .sort((a,b) => a.muscle_group.localeCompare(b.muscle_group) || a.name.localeCompare(b.name));
  },
  get(id){ return byId('exercises', id); },
  create(data){ const e = insert('exercises',{...data,is_custom:true}); save(); return e; },
  update(id, data){ Object.assign(byId('exercises',id), data); save(); },
  remove(id){
    remove('routine_exercises', re => re.exercise_id === id);
    remove('personal_records', pr => pr.exercise_id === id);
    remove('exercises', e => e.id === id);
    save();
  }
};

const RoutineRepo = {
  all(){ return [...db().routines].sort((a,b) => (b.is_active?1:0)-(a.is_active?1:0) || a.name.localeCompare(b.name)); },
  get(id){ return byId('routines', id); },
  active(){ return db().routines.find(r => r.is_active) || null; },
  create(name){
    const routines = db().routines;
    const r = insert('routines',{name,notes:'',is_active:routines.length===0,created_at:now(),updated_at:now()});
    for(let w=1; w<=7; w++){
      insert('routine_days',{routine_id:r.id,weekday:w,
        title: w<=5 ? 'Día '+w : 'Descanso', is_rest: w>5, order_index:w-1});
    }
    save(); return r;
  },
  update(id, data){ Object.assign(byId('routines',id), data, {updated_at:now()}); save(); },
  setActive(id){ db().routines.forEach(r => r.is_active = (r.id === id)); save(); },
  remove(id){
    const days = db().routine_days.filter(d => d.routine_id === id).map(d => d.id);
    remove('routine_exercises', re => days.includes(re.routine_day_id));
    remove('routine_days', d => d.routine_id === id);
    remove('routines', r => r.id === id);
    // Los entrenamientos ya guardados NO se tocan: solo pierden el vínculo.
    db().workouts.forEach(w => { if(w.routine_id === id){ w.routine_id = null; w.routine_day_id = null; } });
    save();
  },
  duplicate(id){
    const src = byId('routines', id);
    const copy = insert('routines',{name:src.name+' (copia)',notes:src.notes,is_active:false,
      created_at:now(),updated_at:now()});
    this.days(id).forEach(d => {
      const nd = insert('routine_days',{routine_id:copy.id,weekday:d.weekday,title:d.title,
        is_rest:d.is_rest,order_index:d.order_index});
      this.dayExercises(d.id).forEach(re => {
        const {id:_, routine_day_id:__, exercise_name:___, ...rest} = re;
        insert('routine_exercises',{...rest, routine_day_id:nd.id});
      });
    });
    save(); return copy;
  },
  days(routineId){
    return db().routine_days.filter(d => d.routine_id === routineId)
      .sort((a,b) => a.order_index - b.order_index || a.weekday - b.weekday);
  },
  day(id){ return byId('routine_days', id); },
  dayForWeekday(weekday){
    const r = this.active();
    if(!r) return null;
    return db().routine_days.find(d => d.routine_id === r.id && d.weekday === weekday) || null;
  },
  addDay(routineId){
    const days = this.days(routineId);
    const last = days.length ? days[days.length-1].weekday : 0;
    const d = insert('routine_days',{routine_id:routineId, weekday:(last % 7)+1,
      title:'Nuevo día', is_rest:false, order_index:days.length});
    save(); return d;
  },
  updateDay(id, data){ Object.assign(byId('routine_days',id), data); save(); },
  removeDay(id){
    remove('routine_exercises', re => re.routine_day_id === id);
    remove('routine_days', d => d.id === id);
    save();
  },
  moveDay(routineId, id, dir){
    const days = this.days(routineId);
    const i = days.findIndex(d => d.id === id);
    const j = i + dir;
    if(i < 0 || j < 0 || j >= days.length) return;
    [days[i].order_index, days[j].order_index] = [days[j].order_index, days[i].order_index];
    save();
  },
  copyDay(fromId, toId, move=false){
    if(move) remove('routine_exercises', re => re.routine_day_id === toId);
    this.dayExercises(fromId).forEach(re => {
      const {id:_, routine_day_id:__, exercise_name:___, ...rest} = re;
      insert('routine_exercises',{...rest, routine_day_id:toId});
    });
    if(move) remove('routine_exercises', re => re.routine_day_id === fromId);
    save();
  },
  dayExercises(dayId){
    return db().routine_exercises.filter(re => re.routine_day_id === dayId)
      .sort((a,b) => a.order_index - b.order_index)
      .map(re => ({...re, exercise_name: (byId('exercises', re.exercise_id) || {name:'(eliminado)'}).name}));
  },
  addExercise(dayId, exerciseId){
    const count = db().routine_exercises.filter(re => re.routine_day_id === dayId).length;
    const re = insert('routine_exercises',{routine_day_id:dayId,exercise_id:exerciseId,order_index:count,
      target_sets:4,target_reps:8,target_reps_max:null,target_weight:0,
      rest_seconds:db().settings.defaultRest,target_rpe:8,notes:''});
    save(); return re;
  },
  updateExercise(id, data){ Object.assign(byId('routine_exercises',id), data); save(); },
  removeExercise(id){ remove('routine_exercises', re => re.id === id); save(); },
  moveExercise(dayId, id, dir){
    const list = this.dayExercises(dayId);
    const i = list.findIndex(e => e.id === id);
    const j = i + dir;
    if(i < 0 || j < 0 || j >= list.length) return;
    const a = byId('routine_exercises', list[i].id), b = byId('routine_exercises', list[j].id);
    [a.order_index, b.order_index] = [b.order_index, a.order_index];
    save();
  }
};

/* ---------------- Entrenamientos ---------------- */
const epley = (w,r) => (w>0 && r>0) ? (r===1 ? w : w*(1 + r/30)) : 0;

const WorkoutRepo = {
  inProgress(){ return db().workouts.find(w => w.status === 'in_progress') || null; },
  get(id){ return byId('workouts', id); },

  /* Al iniciar se COPIAN los ejercicios de la rutina (nombre incluido).
     Desde aquí la sesión es independiente: editar la rutina después
     jamás cambia lo que quedó registrado. */
  startFromDay(routineId, dayId, title){
    const w = insert('workouts',{routine_id:routineId, routine_day_id:dayId, title,
      started_at:now(), finished_at:null, duration_seconds:0, total_volume:0,
      status:'in_progress', notes:''});
    RoutineRepo.dayExercises(dayId).forEach((re,i) => {
      this._addExercise(w.id, re.exercise_id, re.exercise_name, i,
        re.target_sets, re.target_reps, re.target_weight, re.rest_seconds, re.notes);
    });
    save(); return w;
  },
  startEmpty(title='Entrenamiento libre'){
    const w = insert('workouts',{routine_id:null, routine_day_id:null, title,
      started_at:now(), finished_at:null, duration_seconds:0, total_volume:0,
      status:'in_progress', notes:''});
    save(); return w;
  },
  _addExercise(workoutId, exerciseId, name, order, sets, reps, weight, rest, notes){
    const we = insert('workout_exercises',{workout_id:workoutId, exercise_id:exerciseId,
      exercise_name:name, order_index:order, target_sets:sets, target_reps:reps,
      target_weight:weight, rest_seconds:rest, notes:notes||''});
    for(let s=1; s<=sets; s++){
      insert('workout_sets',{workout_exercise_id:we.id, workout_id:workoutId, exercise_id:exerciseId,
        set_number:s, weight, reps, rpe:null, is_completed:false, completed_at:null});
    }
    return we;
  },
  addExercise(workoutId, exerciseId){
    const ex = ExerciseRepo.get(exerciseId);
    const order = db().workout_exercises.filter(e => e.workout_id === workoutId).length;
    const we = this._addExercise(workoutId, exerciseId, ex.name, order, 3, 10, 0, db().settings.defaultRest, '');
    save(); return we;
  },
  removeExercise(weId){
    remove('workout_sets', s => s.workout_exercise_id === weId);
    remove('workout_exercises', e => e.id === weId);
    save();
  },
  groups(workoutId){
    return db().workout_exercises.filter(e => e.workout_id === workoutId)
      .sort((a,b) => a.order_index - b.order_index)
      .map(e => ({exercise:e,
        sets: db().workout_sets.filter(s => s.workout_exercise_id === e.id)
                .sort((a,b) => a.set_number - b.set_number)}));
  },
  addSet(weId){
    const we = byId('workout_exercises', weId);
    const sets = db().workout_sets.filter(s => s.workout_exercise_id === weId)
                   .sort((a,b) => a.set_number - b.set_number);
    const last = sets[sets.length-1];
    const s = insert('workout_sets',{workout_exercise_id:weId, workout_id:we.workout_id,
      exercise_id:we.exercise_id, set_number:(last ? last.set_number : 0)+1,
      weight: last ? last.weight : we.target_weight, reps: last ? last.reps : we.target_reps,
      rpe:null, is_completed:false, completed_at:null});
    save(); return s;
  },
  removeSet(setId){ remove('workout_sets', s => s.id === setId); save(); },
  updateSet(setId, data){ Object.assign(byId('workout_sets', setId), data); save(); },

  /* Guarda la serie al instante y devuelve los récords nuevos. */
  completeSet(setId, weight, reps, rpe){
    const s = byId('workout_sets', setId);
    Object.assign(s, {weight, reps, rpe, is_completed:true, completed_at:now()});
    const records = this._checkRecords(s);
    save();
    return records;
  },
  uncompleteSet(setId){
    Object.assign(byId('workout_sets', setId), {is_completed:false, completed_at:null});
    save();
  },
  _checkRecords(s){
    if(!s.reps) return [];
    const candidates = [
      ['max_weight', s.weight],
      ['max_reps', s.reps],
      ['max_volume', s.weight * s.reps],
      ['max_1rm', epley(s.weight, s.reps)]
    ];
    const out = [];
    candidates.forEach(([type, value]) => {
      if(!(value > 0)) return;
      const prev = db().personal_records.find(p => p.exercise_id === s.exercise_id && p.record_type === type);
      if(prev && value <= prev.value) return;
      const row = {exercise_id:s.exercise_id, record_type:type, value, weight:s.weight,
        reps:s.reps, workout_id:s.workout_id, achieved_at:now()};
      if(prev) Object.assign(prev, row); else insert('personal_records', row);
      // El primer registro de un ejercicio no se anuncia como récord.
      if(prev) out.push({...row});
    });
    return out;
  },
  finish(workoutId){
    const w = byId('workouts', workoutId);
    // Las series no realizadas no ensucian el historial.
    remove('workout_sets', s => s.workout_id === workoutId && !s.is_completed);
    const usedWe = new Set(db().workout_sets.filter(s => s.workout_id === workoutId)
                     .map(s => s.workout_exercise_id));
    remove('workout_exercises', e => e.workout_id === workoutId && !usedWe.has(e.id));
    const volume = db().workout_sets.filter(s => s.workout_id === workoutId)
                     .reduce((a,s) => a + s.weight*s.reps, 0);
    Object.assign(w, {status:'completed', finished_at:now(),
      duration_seconds: Math.round((now() - w.started_at)/1000), total_volume: volume});
    save(); return w;
  },
  cancel(workoutId){
    remove('workout_sets', s => s.workout_id === workoutId);
    remove('workout_exercises', e => e.workout_id === workoutId);
    remove('personal_records', p => p.workout_id === workoutId);
    remove('workouts', w => w.id === workoutId);
    save();
  },
  history(){
    return db().workouts.filter(w => w.status === 'completed')
      .sort((a,b) => b.started_at - a.started_at);
  },
  removeWorkout(id){ this.cancel(id); },
  recordsInWorkout(id){ return db().personal_records.filter(p => p.workout_id === id).length; },
  recentRecords(limit=5){
    return [...db().personal_records].sort((a,b) => b.achieved_at - a.achieved_at)
      .slice(0, limit)
      .map(p => ({...p, exercise_name:(byId('exercises',p.exercise_id)||{name:'?'}).name}));
  },
  recordsFor(exerciseId){ return db().personal_records.filter(p => p.exercise_id === exerciseId); },

  /* Sesiones completadas de un ejercicio, agrupadas por entrenamiento. */
  exerciseSessions(exerciseId){
    const map = new Map();
    db().workout_sets.filter(s => s.exercise_id === exerciseId && s.is_completed).forEach(s => {
      const w = byId('workouts', s.workout_id);
      if(!w || w.status !== 'completed') return;
      if(!map.has(w.id)) map.set(w.id, {workout_id:w.id, date:w.started_at, title:w.title, sets:[]});
      map.get(w.id).sets.push(s);
    });
    return [...map.values()].sort((a,b) => b.date - a.date).map(s => {
      s.sets.sort((a,b) => a.set_number - b.set_number);
      s.bestWeight = Math.max(...s.sets.map(x => x.weight), 0);
      s.bestReps = Math.max(...s.sets.map(x => x.reps), 0);
      s.volume = s.sets.reduce((a,x) => a + x.weight*x.reps, 0);
      return s;
    });
  }
};

/* ---------------- Progreso ---------------- */
const startOfDay = d => { const x = new Date(d); x.setHours(0,0,0,0); return x; };
function startOfWeek(d){
  const x = startOfDay(d);
  x.setDate(x.getDate() - ((x.getDay() + 6) % 7));   // lunes
  return x;
}

const ProgressRepo = {
  statsBetween(from, to){
    const ws = db().workouts.filter(w => w.status === 'completed' && w.started_at >= from && w.started_at < to);
    const ids = new Set(ws.map(w => w.id));
    const sets = db().workout_sets.filter(s => ids.has(s.workout_id) && s.is_completed);
    return {
      workouts: ws.length,
      volume: ws.reduce((a,w) => a + w.total_volume, 0),
      sets: sets.length,
      seconds: ws.reduce((a,w) => a + w.duration_seconds, 0),
      maxWeight: sets.reduce((a,s) => Math.max(a, s.weight), 0),
      maxReps: sets.reduce((a,s) => Math.max(a, s.reps), 0)
    };
  },
  week(){ const s = startOfWeek(new Date()); return this.statsBetween(+s, +new Date(+s + 7*864e5)); },
  month(){ const n = new Date(); return this.statsBetween(+new Date(n.getFullYear(),n.getMonth(),1),
                                                          +new Date(n.getFullYear(),n.getMonth()+1,1)); },
  allTime(){ return this.statsBetween(0, now() + 864e5); },
  weeklyVolume(weeks=8){
    const first = +startOfWeek(new Date()) - (weeks-1)*7*864e5;
    const buckets = Array.from({length:weeks}, (_,i) => ({date:new Date(first + i*7*864e5), value:0}));
    db().workouts.filter(w => w.status === 'completed' && w.started_at >= first).forEach(w => {
      const idx = Math.floor((+startOfWeek(new Date(w.started_at)) - first) / (7*864e5));
      if(idx >= 0 && idx < weeks) buckets[idx].value += w.total_volume;
    });
    return buckets;
  },
  streak(){
    const days = [...new Set(db().workouts.filter(w => w.status === 'completed')
                    .map(w => +startOfDay(new Date(w.started_at))))].sort((a,b) => b-a);
    if(!days.length) return 0;
    const today = +startOfDay(new Date());
    if(days[0] !== today && days[0] !== today - 864e5) return 0;
    let streak = 0, cursor = days[0];
    for(const d of days){
      if(d === cursor){ streak++; cursor -= 864e5; }
      else if(d < cursor) break;
    }
    return streak;
  },
  exercisePoints(exerciseId){
    return WorkoutRepo.exerciseSessions(exerciseId)
      .slice(0, 20).reverse()
      .map(s => ({date:new Date(s.date), maxWeight:s.bestWeight, maxReps:s.bestReps, volume:s.volume}));
  }
};

const MeasureRepo = {
  all(){ return [...db().body_measurements].sort((a,b) => b.measured_at - a.measured_at); },
  create(data){ insert('body_measurements', data); save(); },
  remove(id){ remove('body_measurements', m => m.id === id); save(); }
};

/* ---------------- Respaldo JSON ----------------
   Mismo formato que la versión Flutter: los respaldos son
   intercambiables entre ambas. */
const BackupRepo = {
  build(){
    return {app:'gym_tracker', schema_version:1, exported_at:new Date().toISOString(),
            data: JSON.parse(JSON.stringify(db()))};
  },
  filename(){
    const d = new Date();
    const p = n => String(n).padStart(2,'0');
    return `gym_backup_${d.getFullYear()}-${p(d.getMonth()+1)}-${p(d.getDate())}.json`;
  },
  import(text){
    const raw = JSON.parse(text);
    if(!raw || raw.app !== 'gym_tracker' || !raw.data)
      throw new Error('El archivo no es un respaldo de Gym Tracker.');
    const data = raw.data;
    TABLES.forEach(t => { if(!Array.isArray(data[t])) data[t] = []; });
    if(!data.settings) data.settings = {theme:'dark',sound:true,vibration:true,defaultRest:120};
    // Los respaldos de la versión Flutter guardan kilos y no traen unidad.
    if(!data.settings.unit) data.settings.unit = 'kg';
    // Los respaldos de la versión Flutter no traen "seq": lo reconstruimos.
    data.seq = data.seq || {};
    TABLES.forEach(t => {
      data.seq[t] = data[t].reduce((a,r) => Math.max(a, r.id || 0), data.seq[t] || 0);
      // Normaliza booleanos guardados como 0/1 por SQLite.
      data[t].forEach(r => {
        ['is_active','is_rest','is_completed','is_custom','is_warmup'].forEach(k => {
          if(k in r) r[k] = !!r[k];
        });
      });
    });
    store = {app:'gym_tracker', schema_version:1, exported_at:raw.exported_at, data};
    save();
    return TABLES.reduce((a,t) => a + data[t].length, 0);
  }
};

/* ================================================================
   CAPA 3 — UTILIDADES DE PRESENTACIÓN
   ================================================================ */
const WEEKDAYS = ['Lunes','Martes','Miércoles','Jueves','Viernes','Sábado','Domingo'];
const MONTHS = ['enero','febrero','marzo','abril','mayo','junio','julio','agosto',
                'septiembre','octubre','noviembre','diciembre'];

const esc = s => String(s ?? '').replace(/[&<>"']/g, c =>
  ({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[c]));
const num = v => {
  const n = Number(v) || 0;
  return n % 1 === 0 ? String(n) : n.toFixed(1);
};
/* ---------------- Unidades ----------------
   Todo se ALMACENA siempre en kilos y centímetros. La unidad elegida
   es solo una capa de presentación: se convierte al mostrar y al leer
   un campo. Así cambiar de libras a kilos nunca altera el historial
   ni descuadra los volúmenes acumulados, y los respaldos siguen
   siendo compatibles con la versión Flutter. */
const LB_PER_KG = 2.2046226218;
const CM_PER_IN = 2.54;

const unitW = () => (db().settings.unit === 'kg' ? 'kg' : 'lb');
const unitL = () => (unitW() === 'kg' ? 'cm' : 'in');

const kgToUnit = v => unitW() === 'lb' ? (Number(v)||0) * LB_PER_KG : (Number(v)||0);
const unitToKg = v => unitW() === 'lb' ? (Number(v)||0) / LB_PER_KG : (Number(v)||0);
const cmToUnit = v => unitL() === 'in' ? (Number(v)||0) / CM_PER_IN : (Number(v)||0);
const unitToCm = v => unitL() === 'in' ? (Number(v)||0) * CM_PER_IN : (Number(v)||0);

/* Número de peso ya convertido, sin etiqueta (para "60 × 8"). */
const wnum = v => num(Math.round(kgToUnit(v) * 10) / 10);
/* Peso con etiqueta y separador de miles (para volúmenes). */
const wt = v => {
  const n = Math.round(kgToUnit(v) * 10) / 10;
  return (n % 1 === 0 ? n.toLocaleString('en-US') : n.toLocaleString('en-US',{maximumFractionDigits:1}))
    + ' ' + unitW();
};
/* Incremento natural del stepper según la unidad. */
const wStep = () => unitW() === 'lb' ? 5 : 2.5;
const wMax = () => unitW() === 'lb' ? 2200 : 1000;
const fmtDate = ms => { const d = new Date(ms); return `${d.getDate()} ${MONTHS[d.getMonth()]} ${d.getFullYear()}`; };
const fmtShort = ms => { const d = new Date(ms), p = n => String(n).padStart(2,'0');
  return `${p(d.getDate())}/${p(d.getMonth()+1)}`; };
const weekdayName = w => WEEKDAYS[(w-1+7)%7];
const todayWeekday = () => ((new Date().getDay() + 6) % 7) + 1;   // 1 = lunes

function fmtDur(sec){
  sec = Math.max(0, Math.round(sec));
  const h = Math.floor(sec/3600), m = Math.floor((sec%3600)/60);
  if(h) return `${h}h ${String(m).padStart(2,'0')}min`;
  if(m) return `${m}min`;
  return `${sec}s`;
}
const clock = sec => { sec = Math.max(0, Math.round(sec));
  return `${String(Math.floor(sec/60)).padStart(2,'0')}:${String(sec%60).padStart(2,'0')}`; };

const PR_LABEL = {max_weight:'Peso máximo', max_reps:'Más repeticiones',
                  max_volume:'Mayor volumen en una serie', max_1rm:'Mejor 1RM estimado'};

/* ================================================================
   ESTADO DE NAVEGACIÓN
   ================================================================ */
const state = {
  tab: 'home',
  stack: [],            // pila de sub-pantallas
  exSearch: '', exGroup: 'Todos',
  workoutTab: 0,        // ejercicio visible en modo entrenamiento
  metric: 'weight'
};

const view = () => state.stack.length ? state.stack[state.stack.length-1] : {name:state.tab};
function go(name, params={}){ state.stack.push({name, ...params}); render(); window.scrollTo(0,0); }
function back(){ state.stack.pop(); render(); }
function setTab(tab){ state.tab = tab; state.stack = []; render(); window.scrollTo(0,0); }

const $ = sel => document.querySelector(sel);

/* ================================================================
   TEMPORIZADOR DE DESCANSO
   Se guarda el instante final, no un contador: bloquear la pantalla
   o recargar la página no lo desfasa.
   ================================================================ */
const timer = {
  endsAt:null, total:0, paused:false, pausedLeft:0,
  start(seconds){
    if(!(seconds > 0)) return;
    this.total = seconds; this.paused = false; this.pausedLeft = 0;
    this.endsAt = now() + seconds*1000;
    this.persist(); renderDock();
  },
  add(seconds){
    if(!this.running()) return;
    this.total += seconds;
    if(this.paused) this.pausedLeft += seconds; else this.endsAt += seconds*1000;
    this.persist(); renderDock();
  },
  toggle(){
    if(!this.running()) return;
    if(this.paused){ this.endsAt = now() + this.pausedLeft*1000; this.paused = false; }
    else { this.pausedLeft = this.left(); this.paused = true; }
    this.persist(); renderDock();
  },
  skip(){ this.endsAt = null; this.paused = false; this.total = 0; this.persist(); renderDock(); },
  running(){ return this.endsAt !== null; },
  left(){ return this.paused ? this.pausedLeft : Math.max(0, Math.ceil((this.endsAt - now())/1000)); },
  persist(){
    try{ localStorage.setItem('gym_timer', JSON.stringify({
      endsAt:this.endsAt, total:this.total, paused:this.paused, pausedLeft:this.pausedLeft})); }catch(e){}
  },
  restore(){
    try{
      const t = JSON.parse(localStorage.getItem('gym_timer') || 'null');
      if(t && t.endsAt && (t.paused || t.endsAt > now())) Object.assign(this, t);
    }catch(e){}
  },
  ring(){
    const s = db().settings;
    if(s.vibration && navigator.vibrate) navigator.vibrate([180,90,180]);
    if(s.sound) beep();
  }
};

/* Pitido sintetizado con WebAudio: sin archivos, funciona sin conexión. */
let audioCtx = null;
function beep(){
  try{
    audioCtx = audioCtx || new (window.AudioContext || window.webkitAudioContext)();
    if(audioCtx.state === 'suspended') audioCtx.resume();
    [0, 0.22].forEach(delay => {
      const o = audioCtx.createOscillator(), g = audioCtx.createGain();
      const t = audioCtx.currentTime + delay;
      o.type = 'sine'; o.frequency.setValueAtTime(880, t);
      g.gain.setValueAtTime(0.0001, t);
      g.gain.exponentialRampToValueAtTime(0.32, t + 0.02);
      g.gain.exponentialRampToValueAtTime(0.0001, t + 0.18);
      o.connect(g); g.connect(audioCtx.destination);
      o.start(t); o.stop(t + 0.2);
    });
  }catch(e){}
}

/* Un solo reloj global actualiza los nodos vivos sin volver a
   dibujar la pantalla (así no se pierde el foco de los inputs). */
let timerFired = false;
setInterval(() => {
  if(timer.running()){
    const left = timer.left();
    const el = $('#tclock');
    if(el){
      el.textContent = clock(left);
      const bar = $('#tbar'); if(bar) bar.style.width = (timer.total ? (1 - left/timer.total)*100 : 0) + '%';
    }
    if(left <= 0 && !timer.paused){
      if(!timerFired){ timerFired = true; timer.ring(); toast('⏱️ Descanso terminado'); }
      timer.skip();
    }
  } else timerFired = false;

  const el = $('#elapsed');
  if(el){
    const w = WorkoutRepo.inProgress();
    if(w) el.textContent = fmtDur((now() - w.started_at)/1000);
  }
}, 400);

/* ================================================================
   SHEETS, DIÁLOGOS Y AVISOS
   ================================================================ */
/* scrollIntoView con opciones no existe en Safari antiguo; nunca debe
   tumbar la app por un detalle cosmético. */
function scrollIntoViewSafe(el, opts){
  if(!el || typeof el.scrollIntoView !== 'function') return;
  try{ el.scrollIntoView(opts); }catch(e){ try{ el.scrollIntoView(); }catch(e2){} }
}

function closeLayer(){
  // Cerrar el teclado antes de desmontar evita que iOS deje la página
  // desplazada en una posición imposible.
  const el = document.activeElement;
  if(el && typeof el.blur === 'function') el.blur();
  $('#layer').innerHTML = '';
  lockScroll(false);
}

/* Con un sheet abierto el fondo se congela. En iOS no basta con
   overflow:hidden: hay que fijar el body y devolverlo a su sitio al cerrar,
   porque si no el navegador desplaza el documento para revelar el campo
   enfocado y se lleva por delante todo lo posicionado en fixed. */
let lockedY = 0;
function lockScroll(on){
  const b = document.body;
  if(on){
    if(b.classList.contains('locked')) return;
    lockedY = window.scrollY || window.pageYOffset || 0;
    b.style.top = (-lockedY) + 'px';
    b.classList.add('locked');
  } else {
    if(!b.classList.contains('locked')) return;
    b.classList.remove('locked');
    b.style.top = '';
    window.scrollTo(0, lockedY);
  }
}

function sheet(html, opts={}){
  $('#layer').innerHTML =
    `<div class="scrim" data-scrim="1"><div class="sheet">
       <div class="grab"></div>${html}</div></div>`;
  lockScroll(true);
  if(opts.focus) setTimeout(() => {
    const f = $('#layer ' + opts.focus);
    if(f){ f.focus(); scrollIntoViewSafe(f, {block:'center'}); }
  }, 120);
}

function dialog(html){
  $('#layer').innerHTML =
    `<div class="scrim center" data-scrim="1"><div class="sheet dialog">${html}</div></div>`;
  lockScroll(true);
}

function confirmDialog(title, message, confirmLabel='Confirmar'){
  return new Promise(resolve => {
    dialog(`<h3>${esc(title)}</h3><p class="muted small" style="margin:0 0 18px">${esc(message)}</p>
      <div class="row" style="gap:10px">
        <button class="btn ghost" id="dlg-no">Volver</button>
        <button class="btn" id="dlg-yes">${esc(confirmLabel)}</button></div>`);
    $('#dlg-no').onclick = () => { closeLayer(); resolve(false); };
    $('#dlg-yes').onclick = () => { closeLayer(); resolve(true); };
  });
}

function promptDialog(title, value='', opts={}){
  return new Promise(resolve => {
    dialog(`<h3>${esc(title)}</h3>
      <input class="t" id="dlg-in" name="campo" value="${esc(value)}" placeholder="${esc(opts.hint||'')}"
        autocomplete="off" autocorrect="off" spellcheck="false" enterkeyhint="done"
        ${opts.numeric ? 'inputmode="decimal"' : ''} style="margin-bottom:16px">
      <div class="row" style="gap:10px">
        <button class="btn ghost" id="dlg-no">Cancelar</button>
        <button class="btn" id="dlg-yes">Guardar</button></div>`);
    const input = $('#dlg-in');
    input.focus(); input.select();
    const done = v => { closeLayer(); resolve(v); };
    $('#dlg-no').onclick = () => done(null);
    $('#dlg-yes').onclick = () => done(input.value);
    input.onkeydown = e => { if(e.key === 'Enter') done(input.value); };
  });
}

let toastTimer = null;
function toast(html, plain=false){
  const el = document.createElement('div');
  el.className = 'toast' + (plain ? ' plain' : '');
  el.innerHTML = html;
  const old = $('.toast'); if(old) old.remove();
  document.body.appendChild(el);
  clearTimeout(toastTimer);
  toastTimer = setTimeout(() => el.remove(), 3200);
}

/* Selector de ejercicio reutilizable (rutinas y sesión activa). */
function pickExercise(onPick){
  const rowsFor = q => {
    const list = ExerciseRepo.all(q || '');
    return list.length
      ? list.map(e => `<button class="opt" data-pick="${e.id}">
          <div class="grow"><div class="b7 ell">${esc(e.name)}</div>
          <div class="tiny muted ell">${esc(e.muscle_group)} · ${esc(e.equipment)}</div></div></button>`).join('')
      : '<p class="muted small" style="padding:14px 4px">Sin resultados. Puedes crear uno nuevo en Más → Ejercicios.</p>';
  };
  sheet(`<h3>Elegir ejercicio</h3>
    <input class="t" id="pick-search" name="buscar" placeholder="Buscar…" autocomplete="off"
      autocorrect="off" spellcheck="false" enterkeyhint="search" style="margin-bottom:12px">
    <div id="pick-list">${rowsFor('')}</div>`);
  const search = $('#pick-search');
  search.oninput = () => { $('#pick-list').innerHTML = rowsFor(search.value); };
  $('#pick-list').onclick = ev => {
    const btn = ev.target.closest('[data-pick]');
    if(!btn) return;
    closeLayer();
    onPick(Number(btn.dataset.pick));
  };
}

/* ================================================================
   COMPONENTES
   ================================================================ */
function stepper(id, label, value, o={}){
  const {step=1, unit='', min=0, max=99999, decimals=false} = o;
  return `<div class="stepper">
    <div class="cap">${esc(label)}</div>
    <div class="body">
      <button class="rnd" data-act="step" data-t="${id}" data-d="${-step}" data-min="${min}" data-max="${max}">−</button>
      <input id="${id}" inputmode="${decimals?'decimal':'numeric'}" value="${num(value)}"
             data-min="${min}" data-max="${max}" autocomplete="off" autocorrect="off"
             spellcheck="false" enterkeyhint="done">
      ${unit ? `<span class="unit">${esc(unit)}</span>` : ''}
      <button class="rnd" data-act="step" data-t="${id}" data-d="${step}" data-min="${min}" data-max="${max}">+</button>
    </div></div>`;
}
const readNum = id => { const el = document.getElementById(id);
  return el ? (parseFloat(String(el.value).replace(',','.')) || 0) : 0; };

function statTile(label, value, accent=false){
  return `<div class="stat"><div class="v${accent?' accent':''}">${value}</div><div class="l">${esc(label)}</div></div>`;
}
function empty(icon, title, message, action=''){
  return `<div class="empty"><div class="ic">${icon}</div><h3>${esc(title)}</h3>
    <p class="small">${esc(message)}</p>${action}</div>`;
}

/* Gráficas dibujadas a mano en SVG: sin librerías ni servicios. */
function lineChart(points, unit=''){
  if(points.length < 2)
    return `<p class="muted small" style="text-align:center;padding:26px 0">
      Necesitas al menos 2 registros para ver la evolución.</p>`;
  const W = 320, H = 150, L = 38, B = 20;
  const vals = points.map(p => p.value);
  let min = Math.min(...vals), max = Math.max(...vals);
  if(max === min){ max += Math.abs(max)*0.1 + 1; min -= Math.abs(min)*0.1 + 1; }
  const x = i => L + (W - L - 6) * (i/(points.length-1));
  const y = v => 6 + (H - B - 12) * (1 - (v-min)/(max-min));
  const d = points.map((p,i) => `${i?'L':'M'}${x(i).toFixed(1)},${y(p.value).toFixed(1)}`).join(' ');
  const area = `${d} L${x(points.length-1).toFixed(1)},${H-B} L${L},${H-B} Z`;
  const grid = [0,1,2,3].map(i => {
    const gy = 6 + (H-B-12)*i/3, v = max - (max-min)*i/3;
    return `<line x1="${L}" y1="${gy}" x2="${W-6}" y2="${gy}" stroke="var(--line)" stroke-width="1"/>
            <text x="0" y="${gy+3.5}" font-size="9" fill="var(--muted)">${num(v)}</text>`;
  }).join('');
  return `<svg class="chart" viewBox="0 0 ${W} ${H+12}" preserveAspectRatio="none" style="height:170px">
    <defs><linearGradient id="lg" x1="0" y1="0" x2="0" y2="1">
      <stop offset="0%" stop-color="var(--accent)" stop-opacity=".30"/>
      <stop offset="100%" stop-color="var(--accent)" stop-opacity="0"/></linearGradient></defs>
    ${grid}<path d="${area}" fill="url(#lg)"/>
    <path d="${d}" fill="none" stroke="var(--accent)" stroke-width="2.2" stroke-linejoin="round" stroke-linecap="round"/>
    ${points.map((p,i) => `<circle cx="${x(i).toFixed(1)}" cy="${y(p.value).toFixed(1)}" r="2.8" fill="var(--accent)"/>`).join('')}
    <text x="${L}" y="${H+8}" font-size="9" fill="var(--muted)">${esc(points[0].label)}</text>
    <text x="${W-6}" y="${H+8}" font-size="9" fill="var(--muted)" text-anchor="end">${esc(points[points.length-1].label)}</text>
  </svg>`;
}

function barChart(points){
  if(!points.length) return '';
  const max = Math.max(...points.map(p => p.value), 1);
  return `<div class="row" style="align-items:flex-end;gap:5px;height:140px">${
    points.map(p => `<div class="grow" style="display:grid;justify-items:center;gap:6px;height:100%">
      <div style="flex:1;width:100%;display:flex;align-items:flex-end">
        <div style="width:100%;height:${Math.max(2,(p.value/max)*100)}%;border-radius:6px 6px 0 0;
          background:${p.value ? 'var(--accent)' : 'var(--line)'}"></div></div>
      <div class="tiny muted">${esc(p.label)}</div></div>`).join('')}</div>`;
}

/* ================================================================
   PANTALLA: INICIO
   ================================================================ */
function screenHome(){
  const routine = RoutineRepo.active();
  const day = RoutineRepo.dayForWeekday(todayWeekday());
  const items = (day && !day.is_rest) ? RoutineRepo.dayExercises(day.id) : [];
  const week = ProgressRepo.week();
  const last = WorkoutRepo.history()[0];
  const active = WorkoutRepo.inProgress();
  const records = WorkoutRepo.recentRecords(3);

  let today;
  if(!routine){
    today = `<div class="card"><h2 style="font-size:21px">Sin rutina activa</h2>
      <p class="muted small" style="margin:6px 0 16px">Crea una rutina para ver aquí tu entrenamiento del día.</p>
      <button class="btn" data-act="tab" data-v="routines">CREAR RUTINA</button></div>`;
  } else if(!day || day.is_rest){
    today = `<div class="card"><div class="eyebrow" style="margin:0 0 6px">${weekdayName(todayWeekday())}</div>
      <h2 style="font-size:22px">Día de descanso 😴</h2>
      <p class="muted small" style="margin:6px 0 16px">Puedes entrenar igualmente con una sesión libre.</p>
      <button class="btn ghost" data-act="start-free">ENTRENAMIENTO LIBRE</button></div>`;
  } else {
    today = `<div class="card hl">
      <div class="eyebrow accent" style="margin:0 0 4px">${weekdayName(day.weekday)}</div>
      <h2 style="font-size:26px;font-weight:800">${esc(day.title)}</h2>
      <div style="margin:16px 0 4px">${items.map(e => `<div class="row" style="margin-bottom:9px">
        <div class="grow b7 ell" style="font-weight:600">${esc(e.exercise_name)}</div>
        <div class="muted b7 nums">${e.target_sets} × ${e.target_reps_max && e.target_reps_max !== e.target_reps
          ? e.target_reps+'-'+e.target_reps_max : e.target_reps}</div></div>`).join('')
        || '<p class="muted small">Este día aún no tiene ejercicios.</p>'}</div>
      ${items.length ? `<button class="btn" data-act="start-today" style="margin-top:12px">▶  INICIAR ENTRENAMIENTO</button>` : ''}
    </div>`;
  }

  return `
  <div class="topbar"><div class="grow"><h1>Hoy</h1></div>
    <div class="sub">${fmtDate(now())}</div></div>
  <div class="screen">
    ${persistent ? '' : `<div class="banner"><b>Modo temporal.</b> Este navegador bloqueó el
      almacenamiento local, así que los datos se perderán al cerrar. Descarga el archivo HTML
      y ábrelo directamente desde tu teléfono o computadora para que se guarden.</div>`}
    ${active ? `<div class="card solid" style="margin-bottom:14px">
      <div class="row"><div class="grow b7">⚡ Tienes un entrenamiento en progreso.</div></div>
      <div class="tiny muted" style="margin:4px 0 14px">${esc(active.title)} · iniciado ${fmtDate(active.started_at)}</div>
      <div class="row" style="gap:10px">
        <button class="btn" data-act="tab" data-v="train">CONTINUAR</button>
        <button class="btn ghost" style="width:auto;padding:0 18px" data-act="cancel-workout">Cancelar</button>
      </div></div>` : ''}
    ${today}
    <div class="eyebrow">Esta semana</div>
    <div class="grid2">
      ${statTile('Entrenamientos', week.workouts, true)}
      ${statTile('Volumen semanal', wt(week.volume))}
      ${statTile('Racha actual', ProgressRepo.streak() === 1 ? '1 día' : ProgressRepo.streak()+' días')}
      ${statTile('Tiempo entrenado', fmtDur(week.seconds))}
    </div>
    ${last ? `<div class="eyebrow">Último entrenamiento</div>
      <div class="card" data-act="open-workout" data-v="${last.id}"><div class="row">
        <div class="grow"><div class="b7 ell">${esc(last.title)}</div>
          <div class="tiny muted">${fmtDate(last.started_at)} · ${fmtDur(last.duration_seconds)}</div></div>
        <div class="b8 nums" style="color:var(--accent)">${wt(last.total_volume)}</div></div></div>` : ''}
    ${records.length ? `<div class="eyebrow">Últimos récords</div>${
      records.map(r => `<div class="card"><div class="row">
        <div>🏆</div><div class="grow"><div class="b7 ell">${esc(r.exercise_name)}</div>
        <div class="tiny muted">${PR_LABEL[r.record_type]}</div></div>
        <div class="b8 nums" style="color:var(--accent)">${wnum(r.weight)} ${unitW()} × ${r.reps}</div></div></div>`).join('')}` : ''}
  </div>`;
}

/* ================================================================
   PANTALLA: ENTRENAR
   ================================================================ */
function screenTrain(){
  const active = WorkoutRepo.inProgress();
  if(active) return screenActiveWorkout(active);

  const routine = RoutineRepo.active();
  const day = RoutineRepo.dayForWeekday(todayWeekday());
  const items = (day && !day.is_rest) ? RoutineRepo.dayExercises(day.id) : [];
  const canStart = !!(routine && day && !day.is_rest && items.length);

  return `<div class="topbar"><h1>Entrenar</h1></div>
  <div class="screen">
    <div class="card${canStart ? ' hl' : ''}">
      <div class="eyebrow" style="margin:0 0 4px">${weekdayName(todayWeekday())}</div>
      <h2 style="font-size:23px">${canStart ? esc(day.title) : 'Sin entrenamiento programado'}</h2>
      <div style="margin:16px 0 6px">${items.map(e => `<div style="margin-bottom:11px">
        <div class="b7" style="font-weight:600">${esc(e.exercise_name)}</div>
        <div class="tiny muted nums">${e.target_sets} × ${e.target_reps}${
          e.target_weight ? ' · '+wt(e.target_weight) : ''} · ${e.rest_seconds}s descanso</div></div>`).join('')}</div>
      ${canStart
        ? `<button class="btn" data-act="start-today">▶  INICIAR ENTRENAMIENTO</button>`
        : `<button class="btn ghost" data-act="tab" data-v="routines">IR A RUTINAS</button>`}
    </div>
    <div style="height:12px"></div>
    <button class="btn ghost" data-act="start-free">＋  ENTRENAMIENTO LIBRE</button>
  </div>`;
}

function screenActiveWorkout(w){
  const groups = WorkoutRepo.groups(w.id);
  if(state.workoutTab >= groups.length) state.workoutTab = Math.max(0, groups.length-1);
  const done = groups.reduce((a,g) => a + g.sets.filter(s => s.is_completed).length, 0);
  const total = groups.reduce((a,g) => a + g.sets.length, 0);
  const volume = groups.reduce((a,g) => a + g.sets.filter(s => s.is_completed)
                    .reduce((b,s) => b + s.weight*s.reps, 0), 0);

  const header = `<div class="topbar">
    <div class="grow"><h1 style="font-size:18px">${esc(w.title)}</h1>
      <div class="sub nums"><span id="elapsed">${fmtDur((now()-w.started_at)/1000)}</span> ·
        ${done}/${total} series · ${wt(volume)}</div></div>
    <button class="iconbtn" data-act="add-ex-workout">＋</button>
    <button class="iconbtn" data-act="workout-menu">⋯</button></div>`;

  if(!groups.length)
    return header + `<div class="screen">${empty('🏋️','Sesión vacía',
      'Agrega el primer ejercicio para empezar a registrar series.',
      '<button class="btn" data-act="add-ex-workout" style="margin-top:18px">AGREGAR EJERCICIO</button>')}</div>`;

  const tabs = `<div class="hscroll" style="padding:2px 16px 8px;position:sticky;top:var(--topbarh);z-index:19;background:var(--bg)">${
    groups.map((g,i) => {
      const gd = g.sets.filter(s => s.is_completed).length;
      return `<button class="pill${i===state.workoutTab?' on':''}" data-act="wtab" data-v="${i}">
        ${gd === g.sets.length && g.sets.length ? '✓ ' : ''}${esc(g.exercise.exercise_name)}
        <span style="opacity:.7;margin-left:5px">${gd}/${g.sets.length}</span></button>`;
    }).join('')}</div>`;

  const g = groups[state.workoutTab];
  const we = g.exercise;
  const nextIdx = g.sets.findIndex(s => !s.is_completed);

  const sets = g.sets.map((s,i) => {
    if(s.is_completed)
      return `<div class="setrow done" data-act="set-menu" data-v="${s.id}" style="margin-bottom:10px">
        <div class="badge ok">${s.set_number}</div>
        <div class="grow b7 nums" style="font-size:17px">${wnum(s.weight)} ${unitW()} × ${s.reps}${
          s.rpe ? `<span class="muted small" style="font-weight:500"> · RPE ${num(s.rpe)}</span>` : ''}</div>
        <div style="color:var(--accent)">✓</div></div>`;
    if(i !== nextIdx)
      return `<div class="setrow pending" style="margin-bottom:10px">
        <div class="badge">${s.set_number}</div>
        <div class="grow nums b7">${wnum(s.weight)} ${unitW()} × ${s.reps}</div>
        <button class="iconbtn" data-act="del-set" data-v="${s.id}">✕</button></div>`;
    return `<div class="card solid" id="active-set" style="margin-bottom:10px">
      <div class="row" style="margin-bottom:10px">
        <div class="badge on">${s.set_number}</div>
        <div class="grow b8" style="font-size:12px;letter-spacing:1.2px">SERIE ${s.set_number}</div>
        <button class="iconbtn" data-act="del-set" data-v="${s.id}">🗑</button></div>
      <div class="grid2">
        ${stepper('in-w','Peso', kgToUnit(s.weight), {step:wStep(), unit:unitW(), decimals:true, max:wMax()})}
        ${stepper('in-r','Reps', s.reps, {step:1, min:1, max:200})}
      </div>
      <div style="height:14px"></div>
      ${stepper('in-rpe','RPE (opcional)', s.rpe || 8, {step:0.5, min:1, max:10, decimals:true})}
      <div style="height:16px"></div>
      <button class="btn" data-act="complete-set" data-v="${s.id}" data-rest="${we.rest_seconds}">COMPLETAR SERIE</button>
    </div>`;
  }).join('');

  return header + tabs + `<div class="screen" style="padding-top:10px">
    <h2 style="font-size:26px;font-weight:800;letter-spacing:-1px;line-height:1.15">${esc(we.exercise_name)}</h2>
    <div class="wrap" style="margin:8px 0 18px">
      <span class="chip on">Objetivo: ${we.target_sets} × ${we.target_reps}</span>
      ${we.target_weight ? `<span class="chip">${wt(we.target_weight)}</span>` : ''}
      <span class="chip">${we.rest_seconds}s descanso</span>
    </div>
    ${sets}
    <button class="btn ghost" data-act="add-set" data-v="${we.id}">＋  AGREGAR SERIE</button>
  </div>`;
}

/* ================================================================
   PANTALLA: RUTINAS
   ================================================================ */
function screenRoutines(){
  const list = RoutineRepo.all();
  return `<div class="topbar"><h1>Rutinas</h1></div>
  <div class="screen">${
    list.length ? list.map(r => `<div class="card${r.is_active?' hl':''}">
      <div class="row">
        <div class="grow" data-act="open-routine" data-v="${r.id}">
          <div class="b7" style="font-size:17px">${esc(r.name)}</div>
          ${r.notes ? `<div class="tiny muted" style="margin-top:3px">${esc(r.notes)}</div>` : ''}
          ${r.is_active ? '<div style="margin-top:8px"><span class="chip on">ACTIVA</span></div>' : ''}
        </div>
        <button class="iconbtn" data-act="routine-menu" data-v="${r.id}">⋯</button>
      </div></div>`).join('')
    : empty('📅','Aún no tienes rutinas','Crea tu primera rutina y organiza la semana a tu gusto.')}
  </div>
  <div class="fabwrap"><button class="fab" data-act="new-routine">＋ Nueva rutina</button></div>`;
}

function screenRoutine(routineId){
  const r = RoutineRepo.get(routineId);
  if(!r) return screenRoutines();
  const days = RoutineRepo.days(routineId);
  return `<div class="topbar">
    <button class="iconbtn" data-act="back">←</button>
    <div class="grow"><h1 style="font-size:20px">${esc(r.name)}</h1></div>
    ${r.is_active ? '' : `<button class="pill" data-act="activate-routine" data-v="${r.id}">ACTIVAR</button>`}
  </div>
  <div class="screen">${days.map(d => {
    const items = d.is_rest ? [] : RoutineRepo.dayExercises(d.id);
    return `<div class="card">
      <div class="row">
        <div class="grow" ${d.is_rest ? '' : `data-act="open-day" data-v="${d.id}"`}>
          <div class="eyebrow" style="margin:0">${weekdayName(d.weekday)}</div>
          <div class="b7" style="font-size:17px;margin-top:2px">${esc(d.title)}</div>
        </div>
        <button class="iconbtn" data-act="day-menu" data-v="${d.id}">⋯</button>
      </div>
      ${d.is_rest ? '<div style="margin-top:8px"><span class="chip">DESCANSO</span></div>' :
        (items.length ? `<div style="margin-top:10px">${items.map(e => `<div class="row" style="margin-bottom:6px">
          <div class="grow small ell">${esc(e.exercise_name)}</div>
          <div class="tiny muted b7 nums">${e.target_sets} × ${e.target_reps}</div></div>`).join('')}</div>`
        : '<div class="tiny muted" style="margin-top:8px">Sin ejercicios — toca para configurar</div>')}
    </div>`;
  }).join('')}
    <p class="tiny muted" style="text-align:center;margin-top:18px">
      Editar esta rutina nunca modifica los entrenamientos ya guardados.</p>
  </div>
  <div class="fabwrap"><button class="fab" data-act="add-day" data-v="${routineId}">＋ Agregar día</button></div>`;
}

function screenDay(dayId){
  const d = RoutineRepo.day(dayId);
  if(!d) return screenRoutines();
  const items = RoutineRepo.dayExercises(dayId);
  return `<div class="topbar">
    <button class="iconbtn" data-act="back">←</button>
    <div class="grow"><h1 style="font-size:20px">${esc(d.title)}</h1>
      <div class="sub">${weekdayName(d.weekday)}</div></div></div>
  <div class="screen">${
    items.length ? items.map((e,i) => `<div class="card">
      <div class="row">
        <div class="grow b7 ell" style="font-size:16px" data-act="config-ex" data-v="${e.id}">${esc(e.exercise_name)}</div>
        <button class="iconbtn sm" data-act="move-ex" data-v="${e.id}" data-d="-1" data-day="${dayId}" ${i===0?'style="opacity:.25"':''}>↑</button>
        <button class="iconbtn sm" data-act="move-ex" data-v="${e.id}" data-d="1" data-day="${dayId}" ${i===items.length-1?'style="opacity:.25"':''}>↓</button>
        <button class="iconbtn sm" data-act="del-routine-ex" data-v="${e.id}">🗑</button>
      </div>
      <div class="wrap" style="margin-top:10px" data-act="config-ex" data-v="${e.id}">
        <span class="chip on">${e.target_sets} series</span>
        <span class="chip">${e.target_reps_max && e.target_reps_max !== e.target_reps
          ? e.target_reps+'-'+e.target_reps_max : e.target_reps} reps</span>
        ${e.target_weight ? `<span class="chip">${wt(e.target_weight)}</span>` : ''}
        <span class="chip">${e.rest_seconds}s descanso</span>
        ${e.target_rpe ? `<span class="chip">RPE ${num(e.target_rpe)}</span>` : ''}
      </div>
      ${e.notes ? `<div class="tiny muted" style="margin-top:9px">${esc(e.notes)}</div>` : ''}
    </div>`).join('')
    : empty('🏋️','Día sin ejercicios','Agrega ejercicios y define series, repeticiones y peso.')}
  </div>
  <div class="fabwrap"><button class="fab" data-act="add-day-ex" data-v="${dayId}">＋ Ejercicio</button></div>`;
}

/* ================================================================
   PANTALLA: PROGRESO
   ================================================================ */
function screenProgress(){
  const w = ProgressRepo.week(), m = ProgressRepo.month(), a = ProgressRepo.allTime();
  const vol = ProgressRepo.weeklyVolume(8);
  const records = WorkoutRepo.recentRecords(10);
  return `<div class="topbar"><h1>Progreso</h1></div>
  <div class="screen">
    <div class="eyebrow" style="margin-top:6px">Volumen por semana</div>
    <div class="card">${barChart(vol.map(p => ({label:fmtShort(+p.date), value:p.value})))}</div>
    <div class="eyebrow">Esta semana</div>
    <div class="grid2">
      ${statTile('Volumen', wt(w.volume), true)}${statTile('Entrenamientos', w.workouts)}
      ${statTile('Series', w.sets)}${statTile('Tiempo', fmtDur(w.seconds))}
    </div>
    <div class="eyebrow">Este mes</div>
    <div class="grid2">
      ${statTile('Volumen mensual', wt(m.volume))}${statTile('Entrenamientos', m.workouts)}
    </div>
    <div class="eyebrow">Histórico</div>
    <div class="grid2">
      ${statTile('Entrenamientos', a.workouts)}${statTile('Series realizadas', a.sets)}
      ${statTile('Peso máximo', wt(a.maxWeight))}${statTile('Reps máximas', a.maxReps)}
      ${statTile('Volumen total', wt(a.volume))}${statTile('Tiempo entrenado', fmtDur(a.seconds))}
    </div>
    ${records.length ? `<div class="eyebrow">Récords personales</div>${records.map(r => `
      <div class="card"><div class="row"><div>🏆</div>
        <div class="grow"><div class="b7">${esc(r.exercise_name)}</div>
        <div class="tiny muted">${PR_LABEL[r.record_type]} · ${fmtDate(r.achieved_at)}</div></div>
        <div class="b8 nums" style="color:var(--accent)">${
          r.record_type === 'max_1rm' ? wnum(r.value)+' '+unitW() : wnum(r.weight)+' '+unitW()+' × '+r.reps}</div>
      </div></div>`).join('')}` : ''}
    <p class="tiny muted" style="margin-top:18px">El 1RM es una estimación (fórmula de Epley)
      usada solo como referencia de progreso. No es un valor médico ni exacto.</p>
  </div>`;
}

/* ================================================================
   PANTALLA: MÁS + secciones
   ================================================================ */
function screenMore(){
  const opts = [
    ['📋','Ejercicios','Biblioteca y ejercicios personalizados','exercises'],
    ['🕓','Historial','Todos tus entrenamientos guardados','history'],
    ['📏','Medidas','Peso corporal y medidas','measures'],
    ['⚙️','Configuración','Tema, temporizador y respaldos','settings']
  ];
  return `<div class="topbar"><h1>Más</h1></div>
  <div class="screen">${opts.map(([ic,t,s,v]) => `<div class="card" data-act="open" data-v="${v}">
    <div class="row"><div style="font-size:20px">${ic}</div>
      <div class="grow"><div class="b7">${t}</div><div class="tiny muted">${s}</div></div>
      <div class="muted">›</div></div></div>`).join('')}</div>`;
}

function screenExercises(){
  const list = ExerciseRepo.all(state.exSearch, state.exGroup);
  return `<div class="topbar"><button class="iconbtn" data-act="back">←</button><h1>Ejercicios</h1></div>
  <div style="padding:0 16px 10px">
    <input class="t" id="ex-search" name="buscar-ejercicio" placeholder="Buscar ejercicio"
      value="${esc(state.exSearch)}" autocomplete="off" autocorrect="off" spellcheck="false"
      enterkeyhint="search"></div>
  <div class="hscroll" style="padding-bottom:10px">${
    ['Todos',...MUSCLES].map(g => `<button class="pill${state.exGroup===g?' on':''}"
      data-act="ex-group" data-v="${esc(g)}">${g}</button>`).join('')}</div>
  <div class="screen" style="padding-top:4px">${
    list.length ? list.map(e => `<div class="card" data-act="open-exercise" data-v="${e.id}">
      <div class="row"><div class="grow">
        <div class="b7">${esc(e.name)}</div>
        <div class="wrap" style="margin-top:6px">
          <span class="chip">${esc(e.muscle_group)}</span><span class="chip">${esc(e.equipment)}</span>
          ${e.is_custom ? '<span class="chip on">Personalizado</span>' : ''}</div>
      </div><div class="muted">›</div></div></div>`).join('')
    : empty('🔍','Sin resultados','Puedes crear un ejercicio personalizado con el botón +.')}
  </div>
  <div class="fabwrap"><button class="fab" data-act="new-exercise">＋ Nuevo</button></div>`;
}

function screenExercise(id){
  const e = ExerciseRepo.get(id);
  if(!e) return screenExercises();
  const sessions = WorkoutRepo.exerciseSessions(id);
  const points = ProgressRepo.exercisePoints(id);
  const records = WorkoutRepo.recordsFor(id);
  const metric = state.metric;
  const chartPoints = points.map(p => ({label:fmtShort(+p.date),
    value: metric === 'weight' ? kgToUnit(p.maxWeight)
         : metric === 'reps'   ? p.maxReps
         : kgToUnit(p.volume)}));

  return `<div class="topbar"><button class="iconbtn" data-act="back">←</button>
    <div class="grow"><h1 style="font-size:20px">${esc(e.name)}</h1></div>
    ${e.is_custom ? `<button class="iconbtn" data-act="edit-exercise" data-v="${id}">✎</button>` : ''}</div>
  <div class="screen">
    <div class="wrap"><span class="chip on">${esc(e.muscle_group)}</span>
      <span class="chip">${esc(e.equipment)}</span><span class="chip">${esc(e.type)}</span></div>
    ${e.instructions ? `<div class="card" style="margin-top:14px"><div class="small muted">${esc(e.instructions)}</div></div>` : ''}
    ${records.length ? `<div class="eyebrow">Récords personales</div>${records.map(r => `
      <div class="card"><div class="row"><div class="grow small">${PR_LABEL[r.record_type]}</div>
      <div class="b8 nums" style="color:var(--accent)">${
        r.record_type === 'max_1rm' ? wnum(r.value)+' '+unitW() : wnum(r.weight)+' '+unitW()+' × '+r.reps}</div></div></div>`).join('')}` : ''}
    <div class="eyebrow">Evolución</div>
    <div class="hscroll flush" style="padding-bottom:12px">
      ${[['weight','Peso'],['reps','Reps'],['volume','Volumen']].map(([k,l]) =>
        `<button class="pill${metric===k?' on':''}" data-act="metric" data-v="${k}">${l}</button>`).join('')}</div>
    <div class="card">${lineChart(chartPoints)}</div>
    <div class="eyebrow">Últimos entrenamientos</div>
    ${sessions.length ? sessions.map(s => `<div class="card">
      <div class="row"><div class="grow b7 small ell">${fmtDate(s.date)}</div>
        <div class="tiny muted nums">${wt(s.volume)} · 1RM ~${wnum(epley(s.bestWeight,s.bestReps))} ${unitW()}</div></div>
      <div class="wrap" style="margin-top:9px">${s.sets.map(x =>
        `<span class="chip">${wnum(x.weight)} × ${x.reps}</span>`).join('')}</div></div>`).join('')
    : empty('🕓','Sin registros todavía','Cuando entrenes este ejercicio aparecerá aquí.')}
  </div>`;
}

function screenHistory(){
  const list = WorkoutRepo.history();
  return `<div class="topbar"><button class="iconbtn" data-act="back">←</button><h1>Historial</h1></div>
  <div class="screen">${
    list.length ? list.map(w => `<div class="card" data-act="open-workout" data-v="${w.id}">
      <div class="row"><div class="grow">
        <div class="eyebrow" style="margin:0">${fmtDate(w.started_at)}</div>
        <div class="b7" style="font-size:17px;margin-top:3px">${esc(w.title)}</div>
        <div class="wrap" style="margin-top:8px">
          <span class="chip">${fmtDur(w.duration_seconds)}</span>
          <span class="chip on">${wt(w.total_volume)}</span></div>
      </div><div class="muted">›</div></div></div>`).join('')
    : empty('🕓','Todavía no hay entrenamientos','Cuando finalices tu primera sesión aparecerá aquí.')}
  </div>`;
}

function screenWorkout(id){
  const w = WorkoutRepo.get(id);
  if(!w) return screenHistory();
  const groups = WorkoutRepo.groups(id);
  return `<div class="topbar"><button class="iconbtn" data-act="back">←</button>
    <div class="grow"><h1 style="font-size:20px">${esc(w.title)}</h1>
      <div class="sub">${fmtDate(w.started_at)}</div></div>
    <button class="iconbtn" data-act="del-workout" data-v="${id}">🗑</button></div>
  <div class="screen">
    <div class="grid2">${statTile('Duración', fmtDur(w.duration_seconds))}
      ${statTile('Volumen', wt(w.total_volume), true)}</div>
    <div class="eyebrow">Lo que hiciste</div>
    ${groups.map(g => `<div class="card">
      <div class="b7" style="font-size:16px">${esc(g.exercise.exercise_name)}</div>
      <div style="margin-top:10px">${g.sets.map(s => `<div class="row" style="margin-bottom:6px">
        <div class="tiny muted" style="width:22px">${s.set_number}</div>
        <div class="grow b7 nums">${wnum(s.weight)} × ${s.reps}</div>
        ${s.rpe ? `<div class="tiny muted">RPE ${num(s.rpe)}</div>` : ''}</div>`).join('')}</div>
    </div>`).join('')}
    <p class="tiny muted" style="margin-top:16px">Este registro es inmutable: refleja exactamente
      lo realizado ese día, aunque la rutina haya cambiado después.</p>
  </div>`;
}

function screenMeasures(){
  const list = MeasureRepo.all();
  const points = list.filter(m => m.body_weight).reverse()
    .map(m => ({label:fmtShort(m.measured_at), value:kgToUnit(m.body_weight)}));
  // [clave, etiqueta, unidad, conversión]
  const FIELDS = [['body_weight','Peso',unitW(),kgToUnit],['body_fat','Grasa','%',v=>v],
    ['chest','Pecho',unitL(),cmToUnit],['waist','Cintura',unitL(),cmToUnit],
    ['arm','Brazo',unitL(),cmToUnit],['thigh','Muslo',unitL(),cmToUnit],
    ['calf','Pantorrilla',unitL(),cmToUnit]];
  return `<div class="topbar"><button class="iconbtn" data-act="back">←</button><h1>Medidas</h1></div>
  <div class="screen">${
    list.length ? `<div class="eyebrow" style="margin-top:6px">Peso corporal</div>
      <div class="card">${lineChart(points)}</div>
      <div class="eyebrow">Registros</div>
      ${list.map(m => `<div class="card">
        <div class="row"><div class="grow b7 small ell">${fmtDate(m.measured_at)}</div>
          <button class="iconbtn" data-act="del-measure" data-v="${m.id}">🗑</button></div>
        <div class="wrap" style="margin-top:6px">${FIELDS.filter(([k]) => m[k] != null && m[k] !== '')
          .map(([k,l,u,conv]) => `<span class="chip">${l} ${num(Math.round(conv(m[k])*10)/10)} ${u}</span>`).join('')}</div>
      </div>`).join('')}`
    : empty('📏','Sin medidas registradas','Registra peso corporal y medidas para ver tu evolución.')}
  </div>
  <div class="fabwrap"><button class="fab" data-act="new-measure">＋ Registrar</button></div>`;
}

function screenSettings(){
  const s = db().settings;
  return `<div class="topbar"><button class="iconbtn" data-act="back">←</button><h1>Configuración</h1></div>
  <div class="screen">
    <div class="eyebrow" style="margin-top:6px">Apariencia</div>
    <div class="card"><div class="hscroll flush">
      ${[['dark','Oscuro'],['light','Claro'],['system','Sistema']].map(([k,l]) =>
        `<button class="pill${s.theme===k?' on':''}" data-act="theme" data-v="${k}">${l}</button>`).join('')}
    </div></div>
    <div class="eyebrow">Unidades</div>
    <div class="card">
      <div class="hscroll flush" style="margin-bottom:10px">
        ${[['lb','Libras (lb)'],['kg','Kilos (kg)']].map(([k,l]) =>
          `<button class="pill${unitW()===k?' on':''}" data-act="unit" data-v="${k}">${l}</button>`).join('')}
      </div>
      <p class="small muted" style="margin:0">Cambia cómo se muestran y se escriben los pesos.
        Internamente todo se guarda igual, así que puedes alternar cuando quieras sin alterar tu
        historial ni tus récords. Las medidas corporales acompañan a la elección:
        ${unitW()==='lb' ? 'libras y pulgadas' : 'kilos y centímetros'}.</p>
    </div>
    <div class="eyebrow">Temporizador</div>
    <div class="card">
      <div class="row" style="padding:6px 0"><div class="grow">Sonido al terminar</div>
        <button class="pill${s.sound?' on':''}" data-act="toggle" data-v="sound">${s.sound?'Sí':'No'}</button></div>
      <div class="sep"></div>
      <div class="row" style="padding:6px 0"><div class="grow">Vibración al terminar</div>
        <button class="pill${s.vibration?' on':''}" data-act="toggle" data-v="vibration">${s.vibration?'Sí':'No'}</button></div>
      <div class="sep"></div>
      <div class="row" style="padding:6px 0"><div class="grow">Descanso por defecto</div>
        <button class="pill" data-act="default-rest">${s.defaultRest}s</button></div>
    </div>
    <div class="eyebrow">Respaldo local</div>
    <div class="card">
      <p class="small muted" style="margin:0 0 14px">No hay servidor: tus datos viven solo en este
        dispositivo. Exporta un JSON para no perderlos si cambias de teléfono. El formato es el mismo
        que el de la versión Flutter, así que los respaldos son intercambiables.</p>
      <button class="btn" data-act="export">⬆  EXPORTAR MIS DATOS</button>
      <div style="height:10px"></div>
      <button class="btn ghost" data-act="import">⬇  IMPORTAR RESPALDO</button>
      <input type="file" id="import-file" accept=".json,application/json" style="display:none">
    </div>
    <div class="eyebrow">Privacidad</div>
    <div class="card"><p class="small muted" style="margin:0">
      Esta aplicación funciona 100% sin conexión. No envía información a servidores externos,
      no usa analytics, no muestra publicidad y no requiere cuenta ni correo electrónico.
      Todo el código está en este único archivo.</p></div>
    <div class="eyebrow">Zona peligrosa</div>
    <button class="btn danger" data-act="wipe">BORRAR TODOS LOS DATOS</button>
    <p class="tiny muted" style="text-align:center;margin-top:20px">Gym Tracker · versión web local</p>
  </div>`;
}

/* ================================================================
   RENDER
   ================================================================ */
const NAV = [['home','🏠','Inicio'],['train','🏋️','Entrenar'],['routines','📅','Rutinas'],
             ['progress','📈','Progreso'],['more','⋯','Más']];

function render(){
  const v = view();
  const screens = {
    home:screenHome, train:screenTrain, routines:screenRoutines, progress:screenProgress,
    more:screenMore, exercises:screenExercises, history:screenHistory,
    measures:screenMeasures, settings:screenSettings
  };
  let html;
  if(v.name === 'routine') html = screenRoutine(v.id);
  else if(v.name === 'day') html = screenDay(v.id);
  else if(v.name === 'exercise') html = screenExercise(v.id);
  else if(v.name === 'workout') html = screenWorkout(v.id);
  else html = (screens[v.name] || screenHome)();

  const keepY = window.scrollY;
  $('#view').innerHTML = html;
  if(keepY) window.scrollTo(0, keepY);
  $('#nav').innerHTML = NAV.map(([k,ic,l]) =>
    `<button class="${state.tab===k?'on':''}" data-act="tab" data-v="${k}">
      <span class="ic">${ic}</span><span>${l}</span></button>`).join('');
  renderDock();
}

function renderDock(){
  const active = WorkoutRepo.inProgress();
  const inWorkout = active && state.tab === 'train' && !state.stack.length;
  const parts = [];

  if(timer.running()){
    const left = timer.left();
    parts.push(`<div class="timer${left<=0?' over':''}">
      <div class="row">
        <div class="clock" id="tclock">${clock(left)}</div>
        <div class="grow"></div>
        <button class="tact" data-act="t-add"><span class="ic">＋</span>30s</button>
        <button class="tact" data-act="t-toggle"><span class="ic">${timer.paused?'▶':'⏸'}</span>${timer.paused?'Seguir':'Pausar'}</button>
        <button class="tact" data-act="t-skip"><span class="ic">⏭</span>Saltar</button>
      </div>
      <div class="bar"><i id="tbar" style="width:${timer.total?((1-left/timer.total)*100):0}%"></i></div>
    </div>`);
  } else if(inWorkout){
    parts.push(`<div class="hscroll" style="margin:0;padding:0 0 2px">
      <span class="chip" style="align-self:center">Descanso</span>
      ${[30,60,90,120,180].map(s => `<button class="pill" data-act="t-preset" data-v="${s}">${s}s</button>`).join('')}
      <button class="pill" data-act="t-custom">Otro</button></div>`);
  }
  if(inWorkout) parts.push(`<button class="btn grey" data-act="finish-workout">FINALIZAR ENTRENAMIENTO</button>`);

  $('#dock').innerHTML = parts.join('');
  measureChrome();
}

/* La barra inferior cambia de altura según haya temporizador y botón de
   finalizar. Se mide en vez de asumirla, y todo lo flotante (FAB, toast,
   relleno inferior de la pantalla) se ancla a esa medida real: así nada
   queda debajo de otra cosa en ninguna combinación. */
function measureChrome(){
  const bar = document.querySelector('.bottom');
  if(!bar) return;
  const h = document.body.classList.contains('kb') ? 0 : bar.offsetHeight;
  document.documentElement.style.setProperty('--bottomh', Math.round(h) + 'px');
  const top = document.querySelector('.topbar');
  if(top) document.documentElement.style.setProperty('--topbarh', Math.round(top.offsetHeight) + 'px');
}
window.addEventListener('resize', measureChrome);
window.addEventListener('orientationchange', () => setTimeout(measureChrome, 250));
if(window.ResizeObserver){
  new ResizeObserver(measureChrome).observe(document.querySelector('.bottom'));
}

/* Teclado en pantalla: en Android empuja los elementos fijos y en iOS los
   tapa. Medimos el viewport visual, escondemos la barra inferior mientras
   se escribe y levantamos los sheets por encima del teclado. */
if(window.visualViewport){
  const vv = window.visualViewport;
  const onViewport = () => {
    const root = document.documentElement.style;
    const kb = Math.max(0, window.innerHeight - vv.height - vv.offsetTop);
    // Alto y desplazamiento del área visible: con estos dos valores los
    // sheets se dibujan exactamente donde el usuario está mirando.
    root.setProperty('--vvh', Math.round(vv.height) + 'px');
    root.setProperty('--vvtop', Math.round(vv.offsetTop) + 'px');
    root.setProperty('--kb', Math.round(kb) + 'px');
    document.body.classList.toggle('kb', kb > 120);
    measureChrome();
  };
  vv.addEventListener('resize', onViewport);
  vv.addEventListener('scroll', onViewport);
  window.addEventListener('orientationchange', () => setTimeout(onViewport, 300));
  onViewport();

  /* Al enfocar un campo, dejarlo a la vista DENTRO del sheet. El teclado
     tarda en aparecer, así que se reintenta un par de veces. */
  document.addEventListener('focusin', ev => {
    const field = ev.target;
    if(!field || !/^(INPUT|TEXTAREA|SELECT)$/.test(field.tagName)) return;
    [120, 350].forEach(delay => setTimeout(() => {
      if(document.activeElement === field) scrollIntoViewSafe(field, {block:'center'});
    }, delay));
  });
} else {
  // Navegadores antiguos: sin visualViewport nos apoyamos en el foco.
  const syncHeight = () =>
    document.documentElement.style.setProperty('--vvh', window.innerHeight + 'px');
  syncHeight();
  window.addEventListener('resize', syncHeight);
  const isField = el => el && /^(INPUT|TEXTAREA|SELECT)$/.test(el.tagName);
  document.addEventListener('focusin', e => {
    if(isField(e.target)){
      document.body.classList.add('kb'); measureChrome();
      setTimeout(() => scrollIntoViewSafe(e.target, {block:'center'}), 250);
    }
  });
  document.addEventListener('focusout', () => {
    setTimeout(() => {
      if(!isField(document.activeElement)){ document.body.classList.remove('kb'); measureChrome(); }
    }, 80);
  });
}

/* ================================================================
   ACCIONES
   ================================================================ */
document.addEventListener('click', async ev => {
  if(ev.target.closest('[data-scrim]') && !ev.target.closest('.sheet')){ closeLayer(); return; }
  const el = ev.target.closest('[data-act]');
  if(!el) return;
  const act = el.dataset.act, v = el.dataset.v;

  switch(act){

  /* --- navegación --- */
  case 'tab': setTab(v); break;
  case 'back': back(); break;
  case 'open': go(v); break;
  case 'open-routine': go('routine',{id:Number(v)}); break;
  case 'open-day': go('day',{id:Number(v)}); break;
  case 'open-exercise': go('exercise',{id:Number(v)}); break;
  case 'open-workout': go('workout',{id:Number(v)}); break;
  case 'metric': state.metric = v; render(); break;
  case 'ex-group': state.exGroup = v; render(); break;

  /* --- stepper --- */
  case 'step': {
    const input = document.getElementById(el.dataset.t);
    if(!input) break;
    const min = parseFloat(input.dataset.min), max = parseFloat(input.dataset.max);
    const next = Math.min(max, Math.max(min,
      (parseFloat(String(input.value).replace(',','.')) || 0) + parseFloat(el.dataset.d)));
    input.value = num(Math.round(next*100)/100);
    if(navigator.vibrate) navigator.vibrate(8);
    break;
  }

  /* --- inicio de sesión --- */
  case 'start-today': {
    const r = RoutineRepo.active(), d = RoutineRepo.dayForWeekday(todayWeekday());
    if(!r || !d) break;
    WorkoutRepo.startFromDay(r.id, d.id, d.title);
    state.workoutTab = 0; keepAwake();
    setTab('train');
    break;
  }
  case 'start-free':
    WorkoutRepo.startEmpty();
    state.workoutTab = 0; keepAwake();
    setTab('train');
    break;
  case 'cancel-workout': {
    const w = WorkoutRepo.inProgress();
    if(w && await confirmDialog('¿Cancelar entrenamiento?',
        'Se borrarán las series de esta sesión. Tu historial anterior no se toca.','Cancelar sesión')){
      WorkoutRepo.cancel(w.id); timer.skip(); render();
    }
    break;
  }

  /* --- sesión activa --- */
  case 'wtab': state.workoutTab = Number(v); render(); break;
  case 'complete-set': {
    const records = WorkoutRepo.completeSet(Number(v), unitToKg(readNum('in-w')),
      Math.round(readNum('in-r')), readNum('in-rpe'));
    const rest = Number(el.dataset.rest) || db().settings.defaultRest;
    timer.start(rest);
    if(navigator.vibrate) navigator.vibrate(25);
    if(records.length)
      toast(`🏆 <b>NUEVO RÉCORD</b><br><span style="font-weight:500">${
        records.map(r => PR_LABEL[r.record_type]).join(' · ')}</span>`);
    render();
    setTimeout(() => {
      scrollIntoViewSafe(document.getElementById('active-set'),
        {block:'center', behavior:'smooth'});
    }, 60);
    break;
  }
  case 'add-set': WorkoutRepo.addSet(Number(v)); render(); break;
  case 'del-set': WorkoutRepo.removeSet(Number(v)); render(); break;
  case 'set-menu': {
    const id = Number(v);
    sheet(`<h3>Serie registrada</h3>
      <button class="opt" data-act="undo-set" data-v="${id}">↩️ Marcar como no realizada</button>
      <button class="opt danger" data-act="del-set-x" data-v="${id}">🗑 Eliminar serie</button>`);
    break;
  }
  case 'undo-set': closeLayer(); WorkoutRepo.uncompleteSet(Number(v)); render(); break;
  case 'del-set-x': closeLayer(); WorkoutRepo.removeSet(Number(v)); render(); break;
  case 'add-ex-workout': {
    const w = WorkoutRepo.inProgress();
    if(w) pickExercise(exId => {
      WorkoutRepo.addExercise(w.id, exId);
      state.workoutTab = WorkoutRepo.groups(w.id).length - 1;
      render();
    });
    break;
  }
  case 'workout-menu': {
    const groups = WorkoutRepo.groups(WorkoutRepo.inProgress().id);
    const g = groups[state.workoutTab];
    sheet(`<h3>Entrenamiento</h3>
      ${g ? `<button class="opt" data-act="del-workout-ex" data-v="${g.exercise.id}">
        🗑 Quitar «${esc(g.exercise.exercise_name)}» de la sesión</button>` : ''}
      <button class="opt danger" data-act="cancel-workout">✕ Cancelar entrenamiento</button>`);
    break;
  }
  case 'del-workout-ex':
    closeLayer(); WorkoutRepo.removeExercise(Number(v));
    state.workoutTab = 0; render();
    break;
  case 'finish-workout': {
    const w = WorkoutRepo.inProgress();
    if(!w) break;
    const anyDone = db().workout_sets.some(s => s.workout_id === w.id && s.is_completed);
    if(!anyDone){
      if(await confirmDialog('No registraste series',
        'Si finalizas ahora no se guardará nada. ¿Prefieres cancelar la sesión?','Cancelar sesión')){
        WorkoutRepo.cancel(w.id); timer.skip(); render();
      }
      break;
    }
    const groups = WorkoutRepo.groups(w.id);
    const prs = WorkoutRepo.recordsInWorkout(w.id);
    const done = WorkoutRepo.finish(w.id);
    const sets = db().workout_sets.filter(s => s.workout_id === w.id).length;
    timer.skip(); releaseWake(); render();
    dialog(`<h3 style="font-size:24px">¡Entrenamiento completado! 💪</h3>
      <p class="small muted" style="margin:-8px 0 16px">${esc(done.title)} · ${fmtDate(done.started_at)}</p>
      <div class="grid2">${statTile('Duración', fmtDur(done.duration_seconds), true)}
        ${statTile('Ejercicios', groups.length)}${statTile('Series', sets)}
        ${statTile('Volumen', wt(done.total_volume))}</div>
      ${prs ? `<div class="card solid" style="margin-top:10px"><div class="row"><div>🏆</div>
        <div class="grow b7 small">${prs === 1 ? '1 récord personal' : prs+' récords personales'} en esta sesión</div>
        </div></div>` : ''}
      <div style="height:16px"></div>
      <button class="btn" data-act="see-detail" data-v="${done.id}">VER DETALLES</button>
      <div style="height:10px"></div>
      <button class="btn ghost" data-act="close-layer">Volver al inicio</button>`);
    break;
  }
  case 'see-detail': closeLayer(); setTab('more'); go('workout',{id:Number(v)}); break;
  case 'close-layer': closeLayer(); setTab('home'); break;

  /* --- temporizador --- */
  case 't-add': timer.add(30); break;
  case 't-toggle': timer.toggle(); break;
  case 't-skip': timer.skip(); break;
  case 't-preset': timer.start(Number(v)); break;
  case 't-custom': {
    const r = await promptDialog('Descanso personalizado (segundos)', String(db().settings.defaultRest), {numeric:true});
    const s = parseInt(r, 10);
    if(s > 0) timer.start(s);
    break;
  }

  /* --- rutinas --- */
  case 'new-routine': {
    const name = await promptDialog('Nueva rutina','',{hint:'Ej. Rutina Hipertrofia'});
    if(name && name.trim()){ const r = RoutineRepo.create(name.trim()); render(); go('routine',{id:r.id}); }
    break;
  }
  case 'routine-menu': {
    const r = RoutineRepo.get(Number(v));
    sheet(`<h3>${esc(r.name)}</h3>
      ${r.is_active ? '' : `<button class="opt" data-act="activate-routine" data-v="${r.id}">✓ Activar</button>`}
      <button class="opt" data-act="rename-routine" data-v="${r.id}">✎ Cambiar nombre</button>
      <button class="opt" data-act="dup-routine" data-v="${r.id}">⧉ Duplicar</button>
      <button class="opt danger" data-act="del-routine" data-v="${r.id}">🗑 Eliminar</button>`);
    break;
  }
  case 'activate-routine': closeLayer(); RoutineRepo.setActive(Number(v)); render(); break;
  case 'dup-routine': closeLayer(); RoutineRepo.duplicate(Number(v)); render(); break;
  case 'rename-routine': {
    closeLayer();
    const r = RoutineRepo.get(Number(v));
    const name = await promptDialog('Cambiar nombre', r.name);
    if(name && name.trim()){ RoutineRepo.update(r.id,{name:name.trim()}); render(); }
    break;
  }
  case 'del-routine': {
    closeLayer();
    const r = RoutineRepo.get(Number(v));
    if(await confirmDialog(`¿Eliminar "${r.name}"?`,
      'Tus entrenamientos ya guardados NO se borran ni se modifican.','Eliminar')){
      RoutineRepo.remove(r.id);
      if(view().name === 'routine') back(); else render();
    }
    break;
  }
  case 'add-day': RoutineRepo.addDay(Number(v)); render(); break;
  case 'day-menu': {
    const d = RoutineRepo.day(Number(v));
    sheet(`<h3>${esc(d.title)} · ${weekdayName(d.weekday)}</h3>
      <button class="opt" data-act="rename-day" data-v="${d.id}">✎ Cambiar nombre</button>
      <button class="opt" data-act="weekday-day" data-v="${d.id}">📅 Cambiar día de la semana</button>
      <button class="opt" data-act="rest-day" data-v="${d.id}">😴 ${d.is_rest?'Marcar como entrenamiento':'Marcar como descanso'}</button>
      <button class="opt" data-act="order-day" data-v="${d.id}" data-d="-1">↑ Subir</button>
      <button class="opt" data-act="order-day" data-v="${d.id}" data-d="1">↓ Bajar</button>
      <button class="opt" data-act="copy-day" data-v="${d.id}" data-move="0">⧉ Copiar a otro día</button>
      <button class="opt" data-act="copy-day" data-v="${d.id}" data-move="1">➜ Mover a otro día</button>
      <button class="opt danger" data-act="del-day" data-v="${d.id}">🗑 Eliminar día</button>`);
    break;
  }
  case 'rename-day': {
    closeLayer();
    const d = RoutineRepo.day(Number(v));
    const t = await promptDialog('Nombre del día', d.title);
    if(t && t.trim()){ RoutineRepo.updateDay(d.id,{title:t.trim()}); render(); }
    break;
  }
  case 'rest-day': {
    closeLayer();
    const d = RoutineRepo.day(Number(v));
    RoutineRepo.updateDay(d.id,{is_rest:!d.is_rest, title: d.is_rest ? d.title : 'Descanso'});
    render();
    break;
  }
  case 'weekday-day': {
    const d = RoutineRepo.day(Number(v));
    sheet(`<h3>Día de la semana</h3>${WEEKDAYS.map((n,i) =>
      `<button class="opt" data-act="set-weekday" data-v="${d.id}" data-w="${i+1}">
        <div class="grow">${n}</div>${d.weekday===i+1?'✓':''}</button>`).join('')}`);
    break;
  }
  case 'set-weekday': closeLayer();
    RoutineRepo.updateDay(Number(v),{weekday:Number(el.dataset.w)}); render(); break;
  case 'order-day': {
    closeLayer();
    const d = RoutineRepo.day(Number(v));
    RoutineRepo.moveDay(d.routine_id, d.id, Number(el.dataset.d)); render();
    break;
  }
  case 'copy-day': {
    const d = RoutineRepo.day(Number(v));
    const move = el.dataset.move === '1';
    const others = RoutineRepo.days(d.routine_id).filter(x => x.id !== d.id);
    sheet(`<h3>${move?'Mover':'Copiar'} a…</h3>${others.map(o =>
      `<button class="opt" data-act="do-copy-day" data-v="${d.id}" data-to="${o.id}" data-move="${move?1:0}">
        ${weekdayName(o.weekday)} — ${esc(o.title)}</button>`).join('')}`);
    break;
  }
  case 'do-copy-day': closeLayer();
    RoutineRepo.copyDay(Number(v), Number(el.dataset.to), el.dataset.move === '1'); render(); break;
  case 'del-day': {
    closeLayer();
    if(await confirmDialog('¿Eliminar día?','Se elimina de la rutina. El historial no cambia.','Eliminar')){
      RoutineRepo.removeDay(Number(v)); render();
    }
    break;
  }
  case 'add-day-ex': {
    const dayId = Number(v);
    pickExercise(exId => { RoutineRepo.addExercise(dayId, exId); render(); });
    break;
  }
  case 'del-routine-ex': RoutineRepo.removeExercise(Number(v)); render(); break;
  case 'move-ex': RoutineRepo.moveExercise(Number(el.dataset.day), Number(v), Number(el.dataset.d)); render(); break;
  case 'config-ex': configExerciseSheet(Number(v)); break;
  case 'save-config-ex': {
    RoutineRepo.updateExercise(Number(v), {
      target_sets: Math.round(readNum('cfg-sets')),
      target_reps: Math.round(readNum('cfg-reps')),
      target_reps_max: Math.round(readNum('cfg-repsmax')) || null,
      target_weight: unitToKg(readNum('cfg-weight')),
      rest_seconds: Math.round(readNum('cfg-rest')),
      target_rpe: readNum('cfg-rpe'),
      notes: $('#cfg-notes').value.trim()
    });
    closeLayer(); render();
    break;
  }

  /* --- ejercicios --- */
  case 'new-exercise': exerciseFormSheet(null); break;
  case 'edit-exercise': exerciseFormSheet(Number(v)); break;
  case 'save-exercise': {
    const data = {
      name: $('#exf-name').value.trim(),
      muscle_group: $('#exf-group').value,
      equipment: $('#exf-equip').value,
      type: $('#exf-type').value,
      description: $('#exf-desc').value.trim(),
      instructions: $('#exf-inst').value.trim()
    };
    if(!data.name){ toast('Escribe un nombre para el ejercicio.', true); break; }
    if(v) ExerciseRepo.update(Number(v), data); else ExerciseRepo.create(data);
    closeLayer(); render();
    break;
  }

  /* --- historial y medidas --- */
  case 'del-workout': {
    if(await confirmDialog('¿Eliminar entrenamiento?','Esta acción no se puede deshacer.','Eliminar')){
      WorkoutRepo.removeWorkout(Number(v)); back();
    }
    break;
  }
  case 'new-measure': measureSheet(); break;
  case 'save-measure': {
    const val = id => { const el = document.getElementById(id);
      const n = parseFloat(String(el.value).replace(',','.')); return isNaN(n) ? null : n; };
    MeasureRepo.create({
      measured_at: +new Date($('#m-date').value || new Date()),
      body_weight: val('m-weight') === null ? null : unitToKg(val('m-weight')),
      body_fat: val('m-fat'),
      chest: val('m-chest') === null ? null : unitToCm(val('m-chest')),
      waist: val('m-waist') === null ? null : unitToCm(val('m-waist')),
      arm: val('m-arm') === null ? null : unitToCm(val('m-arm')),
      thigh: val('m-thigh') === null ? null : unitToCm(val('m-thigh')),
      calf: val('m-calf') === null ? null : unitToCm(val('m-calf')), notes:''
    });
    closeLayer(); render();
    break;
  }
  case 'del-measure': MeasureRepo.remove(Number(v)); render(); break;

  /* --- configuración --- */
  case 'theme': db().settings.theme = v; save(); applyTheme(); render(); break;
  case 'unit':
    db().settings.unit = v; save(); render();
    toast(v === 'lb' ? 'Pesos en libras' : 'Pesos en kilos', true);
    break;
  case 'toggle': db().settings[v] = !db().settings[v]; save(); render(); break;
  case 'default-rest': {
    const r = await promptDialog('Descanso por defecto (segundos)', String(db().settings.defaultRest), {numeric:true});
    const s = parseInt(r, 10);
    if(s > 0){ db().settings.defaultRest = s; save(); render(); }
    break;
  }
  case 'export': {
    const blob = new Blob([JSON.stringify(BackupRepo.build(), null, 2)], {type:'application/json'});
    const a = document.createElement('a');
    a.href = URL.createObjectURL(blob);
    a.download = BackupRepo.filename();
    document.body.appendChild(a); a.click(); a.remove();
    setTimeout(() => URL.revokeObjectURL(a.href), 4000);
    toast('Respaldo descargado 📁', true);
    break;
  }
  case 'import': {
    if(!await confirmDialog('Importar respaldo',
      'Se reemplazarán TODOS los datos actuales por los del archivo.','Continuar')) break;
    $('#import-file').click();
    break;
  }
  case 'wipe': {
    if(await confirmDialog('¿Borrar todo?',
      'Se eliminarán rutinas, entrenamientos, récords y medidas de este dispositivo.','Borrar todo')){
      try{ localStorage.removeItem(LS_KEY); }catch(e){}
      store = emptyStore(); seed(); save();
      state.stack = []; setTab('home');
      toast('Datos borrados. Se restauró la biblioteca inicial.', true);
    }
    break;
  }
  }
});

/* --- inputs que necesitan escucha propia --- */
document.addEventListener('input', ev => {
  if(ev.target.id === 'ex-search'){
    state.exSearch = ev.target.value;
    const list = ExerciseRepo.all(state.exSearch, state.exGroup);
    const screen = $('.screen');
    if(screen) screen.innerHTML = list.length
      ? list.map(e => `<div class="card" data-act="open-exercise" data-v="${e.id}">
          <div class="row"><div class="grow"><div class="b7">${esc(e.name)}</div>
          <div class="wrap" style="margin-top:6px"><span class="chip">${esc(e.muscle_group)}</span>
          <span class="chip">${esc(e.equipment)}</span>
          ${e.is_custom ? '<span class="chip on">Personalizado</span>' : ''}</div></div>
          <div class="muted">›</div></div></div>`).join('')
      : empty('🔍','Sin resultados','Puedes crear un ejercicio personalizado con el botón +.');
  }
});

document.addEventListener('change', ev => {
  if(ev.target.id !== 'import-file') return;
  const file = ev.target.files[0];
  if(!file) return;
  const reader = new FileReader();
  reader.onload = () => {
    try{
      const rows = BackupRepo.import(String(reader.result));
      applyTheme(); state.stack = []; setTab('home');
      toast(`Respaldo restaurado (${rows} registros).`, true);
    }catch(err){
      toast('Archivo inválido: ' + esc(err.message), true);
    }
  };
  reader.readAsText(file);
  ev.target.value = '';
});

/* ================================================================
   SHEETS DE FORMULARIO
   ================================================================ */
function configExerciseSheet(id){
  const e = RoutineRepo.dayExercises(RoutineRepo.day(view().id).id).find(x => x.id === id);
  if(!e) return;
  sheet(`<h3>${esc(e.exercise_name)}</h3>
    <div class="grid2">
      ${stepper('cfg-sets','Series', e.target_sets, {min:1, max:20})}
      ${stepper('cfg-reps','Repeticiones', e.target_reps, {min:1, max:100})}
    </div><div style="height:14px"></div>
    <div class="grid2">
      ${stepper('cfg-repsmax','Rango máx.', e.target_reps_max || e.target_reps, {min:0, max:100})}
      ${stepper('cfg-weight','Peso', kgToUnit(e.target_weight), {step:wStep(), unit:unitW(), decimals:true, max:wMax()})}
    </div><div style="height:14px"></div>
    <div class="grid2">
      ${stepper('cfg-rest','Descanso', e.rest_seconds, {step:15, min:0, max:600, unit:'s'})}
      ${stepper('cfg-rpe','RPE', e.target_rpe || 8, {step:0.5, min:0, max:10, decimals:true})}
    </div>
    <div style="height:16px"></div>
    <label class="f"><span>Notas</span>
      <textarea class="t" id="cfg-notes" placeholder="Opcional">${esc(e.notes||'')}</textarea></label>
    <button class="btn" data-act="save-config-ex" data-v="${e.id}">GUARDAR</button>`);
}

function exerciseFormSheet(id){
  const e = id ? ExerciseRepo.get(id) : null;
  const sel = (name, options, value) => `<select class="t" id="${name}">${
    options.map(o => `<option${o===value?' selected':''}>${o}</option>`).join('')}</select>`;
  sheet(`<h3>${e ? 'Editar ejercicio' : 'Nuevo ejercicio'}</h3>
    <label class="f"><span>Nombre</span>
      <input class="t" id="exf-name" name="exercise-name" value="${esc(e?.name || '')}"
        placeholder="Ej. Press Hammer máquina" autocomplete="off" autocorrect="off"
        autocapitalize="sentences" spellcheck="false" enterkeyhint="done"></label>
    <label class="f"><span>Grupo muscular</span>${sel('exf-group', MUSCLES, e?.muscle_group)}</label>
    <label class="f"><span>Equipo</span>${sel('exf-equip', EQUIPMENT, e?.equipment)}</label>
    <label class="f"><span>Tipo</span>${sel('exf-type', EXTYPES, e?.type)}</label>
    <label class="f"><span>Descripción</span>
      <input class="t" id="exf-desc" value="${esc(e?.description || '')}" autocomplete="off"></label>
    <label class="f"><span>Instrucciones</span>
      <textarea class="t" id="exf-inst">${esc(e?.instructions || '')}</textarea></label>
    <button class="btn" data-act="save-exercise" ${e?`data-v="${e.id}"`:''}>GUARDAR</button>`,
    {focus:'#exf-name'});
}

function measureSheet(){
  const today = new Date().toISOString().slice(0,10);
  const L = unitL();
  const field = (id,label,unit) => `<label class="f"><span>${label} (${unit})</span>
    <input class="t" id="${id}" inputmode="decimal"></label>`;
  sheet(`<h3>Nueva medición</h3>
    <label class="f"><span>Fecha</span><input class="t" type="date" id="m-date" value="${today}"></label>
    <div class="grid2">
      ${field('m-weight','Peso corporal',unitW())}${field('m-fat','Grasa corporal','%')}
      ${field('m-chest','Pecho',L)}${field('m-waist','Cintura',L)}
      ${field('m-arm','Brazo',L)}${field('m-thigh','Muslo',L)}
    </div>
    ${field('m-calf','Pantorrilla',L)}
    <button class="btn" data-act="save-measure">GUARDAR</button>`);
}

/* ================================================================
   TEMA, PANTALLA ACTIVA E INICIO
   ================================================================ */
function prefersDark(){
  try{ return !!(window.matchMedia && window.matchMedia('(prefers-color-scheme: dark)').matches); }
  catch(e){ return true; }
}
function applyTheme(){
  const t = db().settings.theme || 'dark';
  const dark = t === 'dark' || (t === 'system' && prefersDark());
  document.documentElement.dataset.theme = dark ? 'dark' : 'light';
  const meta = document.querySelector('meta[name="theme-color"]');
  if(meta) meta.content = dark ? '#0E1113' : '#F3F5F0';
}
/* Safari antiguo solo tiene addListener, y algunos WebViews no traen
   matchMedia: el tema del sistema es opcional, la app nunca debe caerse. */
try{
  const mq = window.matchMedia('(prefers-color-scheme: dark)');
  const onScheme = () => { if(db().settings.theme === 'system'){ applyTheme(); render(); } };
  if(mq.addEventListener) mq.addEventListener('change', onScheme);
  else if(mq.addListener) mq.addListener(onScheme);
}catch(e){}

/* Evita que la pantalla se apague en mitad de la serie (si el navegador lo permite). */
let wakeLock = null;
async function keepAwake(){
  try{ if('wakeLock' in navigator) wakeLock = await navigator.wakeLock.request('screen'); }catch(e){}
}
function releaseWake(){ try{ wakeLock && wakeLock.release(); wakeLock = null; }catch(e){} }
document.addEventListener('visibilitychange', () => {
  if(document.visibilityState === 'visible'){
    if(WorkoutRepo.inProgress()) keepAwake();
    renderDock();   // el temporizador se recalcula solo con la hora real
  }
});

window.addEventListener('beforeunload', e => {
  if(WorkoutRepo.inProgress()){ e.preventDefault(); e.returnValue = ''; }
});

/* Arranque */
load();
timer.restore();
applyTheme();
if(WorkoutRepo.inProgress()) state.tab = 'home';
render();
</script></body></html>
