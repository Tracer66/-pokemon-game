<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Pikachu — FIFA Pokémon Edition</title>
<style>
  :root{
    --sky:#87CEEB;
    --grass-1:#6fbf5a;
    --grass-2:#4f9b3a;
    --goal-white: #ffffff;
    --net-glow: rgba(255,240,120,0.95);
  }
  html,body{height:100%;margin:0;background:linear-gradient(var(--sky),#cfefff);font-family:Inter,system-ui,Arial}
  .stage{position:relative;width:100vw;height:100vh;overflow:hidden;display:flex;align-items:flex-end;justify-content:center;padding-bottom:20px}

/* FIELD */
  .field {
    width: min(1100px, 96vw);
    height: min(640px, 86vh);
    background: linear-gradient(180deg,var(--grass-1) 45%, var(--grass-2) 100%);
    border-radius:14px;
    box-shadow: 0 12px 40px rgba(0,0,0,0.18);
    position:relative;
    overflow:visible;
  }
  /* goals area */
  .goal {
    position:absolute;
    right: 48px;
    bottom: 64px;
    width: 220px;
    height: 130px;
    border: 8px solid var(--goal-white);
    border-radius: 6px;
    box-sizing: content-box;
    background: linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));
    overflow: visible;
  }
  .goal.net {
    pointer-events:none;
  }

  /* top stripe */
  .field::before{
    content:"";
    position:absolute;
    top:14px; left:40px; right:40px;
    height:10px;
    background: linear-gradient(90deg, rgba(255,255,255,0.12), rgba(255,255,255,0.0));
    border-radius:6px;
    opacity:.6;
  }

  /* Pikachu*/
  .pikachu {
    position:absolute;
    left:60px;
    bottom:64px;
    width:180px;
    user-select:none;
    transition: transform 220ms ease;
  }

  /* Ball */
  .ball {
    position:absolute;
    width:60px;
    bottom:128px;
    left:170px;
    transform-origin:center;
    will-change: transform;
    pointer-events:none;
  }

  /* Goalie area and image */
  .goalieWrap{
    position:absolute;
    right:78px;
    bottom:118px;
    width:140px;
    height:160px;
    display:flex;
    align-items:flex-end;
    justify-content:center;
    pointer-events:none;
  }
  .goalie {
    width:120px;
    transform-origin:center bottom;
    transition: transform 220ms ease, bottom 220ms ease;
    position:relative;
  }

  /* shadow */
  .shadow {
    position:absolute;
    left: 140px;
    bottom:48px;
    width:160px;
    height:24px;
    background: radial-gradient(ellipse at center, rgba(0,0,0,0.35), transparent 60%);
    border-radius:50%;
    filter: blur(3px);
  }

  /* scoreboard */
  .hud {
    position:absolute;
    left:18px;
    top:18px;
    background:rgba(255,255,255,0.92);
    padding:8px 12px;
    border-radius:10px;
    font-weight:700;
    box-shadow:0 6px 18px rgba(0,0,0,0.12);
  }

  /* net shake */
  .goal.shake {
    animation: netShake 420ms ease-in-out;
    box-shadow: 0 0 40px rgba(255,220,80,0.08);
  }
  @keyframes netShake {
    0% { transform: translateX(0) }
    20% { transform: translateX(-6px) }
    40% { transform: translateX(6px) }
    60% { transform: translateX(-4px) }
    80% { transform: translateX(2px) }
    100% { transform: translateX(0) }
  }

  /* Pikachu celebration cute pose */
  .pikachu.celebrate {
    transform: translateY(-28px) rotate(-6deg) scale(1.03);
    transition: transform 320ms cubic-bezier(.2,.9,.2,1);
  }

  /* lightning overlay */
  .lightning {
    position:absolute;
    right: 42px;
    bottom: 120px;
    width: 260px;
    height: 160px;
    pointer-events:none;
    mix-blend-mode: screen;
    opacity:0;
    transform-origin:center;
  }
  .lightning.show { animation: lightningPop 700ms ease forwards; }
  @keyframes lightningPop {
    0% { opacity:0; transform: scale(0.7); filter: blur(6px); }
    20% { opacity:1; transform: scale(1.08); filter: blur(0px); }
    100% { opacity:0; transform: scale(1.2); filter: blur(6px); }
  }
  .lightning svg{ width:100%; height:100%; }

  /* small HUD hint */
  .hint {
    position:absolute;
    right:18px;
    top:18px;
    color:#073;
    background:rgba(255,255,255,0.9);
    padding:8px 12px;border-radius:10px;font-weight:600;font-size:13px;
    box-shadow:0 6px 12px rgba(0,0,0,0.08);
  }

  /* power indicator */
  .power {
    position:absolute;
    left:18px;
    bottom:18px;
    background:rgba(0,0,0,0.5);
    color:white;padding:8px 12px;border-radius:10px;font-weight:700;
  }

  /* responsive */
  @media (max-width:720px){
    .field{ width:95vw; height:84vh; }
    .pikachu{ left:20px; width:140px; bottom:48px; }
    .ball{ width:48px; bottom:96px; left:120px; }
    .goal{ right:26px; bottom:46px; width:160px; height:96px; }
    .goalieWrap{ right:46px; bottom:80px; }
  }
</style>
</head>
<body>
  <div class="stage">
    <div class="field" id="field" aria-label="Pikachu soccer field — click to aim and kick">
      <div class="hud">Score: <span id="score">0</span></div>
      <div class="hint">Click/tap to aim & kick • Double-click to toggle POWER</div>
      <div class="power" id="powerTag">Power: Normal</div>

      <!-- Goal + net -->
      <div class="goal" id="goal" aria-hidden="true"></div>

      <!-- Lightning overlay -->
      <div class="lightning" id="lightning" aria-hidden="true">
        <svg viewBox="0 0 200 120" preserveAspectRatio="none">
          <defs>
            <linearGradient id="lg" x1="0" x2="1">
              <stop offset="0" stop-color="#fff7b3"/><stop offset="1" stop-color="#ffe16b"/>
            </linearGradient>
            <filter id="gblur"><feGaussianBlur stdDeviation="2"/></filter>
          </defs>
          <g filter="url(#gblur)">
            <path d="M40 10 L70 36 L52 38 L96 86 L62 86 L76 108 L44 70 L70 68 Z" fill="url(#lg)" opacity="0.98"/>
            <path d="M110 26 L140 56 L125 58 L168 106 L132 106 L145 128 L114 86 L136 84 Z" fill="url(#lg)" opacity="0.85"/>
          </g>
        </svg>
      </div>

      <!-- Goal keeper wrapper (random pokemon) -->
      <div class="goalieWrap">
        <img id="goalie" class="goalie" src="" alt="goalie" />
      </div>

      <!-- Pikachu image: use user's local if available (pikachu.png), else fallback -->
      <img src="https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/other/official-artwork/25.png"
       class="pikachu" id="pikachu">
      <!-- Ball -->
      <img id="ball" class="ball" src="https://upload.wikimedia.org/wikipedia/commons/6/6e/Football_(soccer_ball).svg" alt="ball">

      <!-- shadow -->
      <div class="shadow" aria-hidden="true"></div>
    </div>
  </div>

<script>
/* ==========================
   Game variables & assets
   ========================== */

const scoreEl = document.getElementById('score');
const field = document.getElementById('field');
const ball = document.getElementById('ball');
const pikachu = document.getElementById('pikachu');
const goalieImg = document.getElementById('goalie');
const goal = document.getElementById('goal');
const lightning = document.getElementById('lightning');
const powerTag = document.getElementById('powerTag');

let score = 0;
let busy = false;
let powerMode = false; // toggled by double click

// goalie pool (public sprite images, name, skill rating 0-1)
const goalies = [
  {name:'Snorlax', url:'https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/other/official-artwork/143.png', skill:0.78},
  {name:'Blastoise', url:'https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/other/official-artwork/9.png', skill:0.7},
  {name:'Feraligatr', url:'https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/other/official-artwork/160.png', skill:0.72},
  {name:'Greninja', url:'https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/other/official-artwork/658.png', skill:0.85},
  {name:'Lucario', url:'https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/other/official-artwork/448.png', skill:0.82},
  {name:'Infernape', url:'https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/other/official-artwork/392.png', skill:0.77},
];

// pick a random goalie to start
let currentGoalie = randomGoalie();
applyGoalie(currentGoalie);

// audio: WebAudio for short effects (cheer, block, zap)
const AudioCtx = window.AudioContext || window.webkitAudioContext;
let audioCtx = null;
function ensureAudio(){
  if(!audioCtx) audioCtx = new AudioCtx();
}
function playCheer(){
  ensureAudio();
  const ctx = audioCtx;
  const now = ctx.currentTime;
  // simple multi-tone cheer-ish chord
  const freqs = [440, 660, 880, 1100];
  freqs.forEach((f, i)=>{
    const o = ctx.createOscillator();
    const g = ctx.createGain();
    o.type = 'sine';
    o.frequency.value = f;
    g.gain.value = 0.0005;
    o.connect(g);
    g.connect(ctx.destination);
    o.start(now + 0.02 + i*0.02);
    g.gain.linearRampToValueAtTime(0.03, now + 0.05 + i*0.02);
    g.gain.linearRampToValueAtTime(0.001, now + 0.35 + i*0.02);
    o.stop(now + 0.45 + i*0.02);
  });
}
function playBlock(){
  ensureAudio();
  const ctx = audioCtx;
  const o = ctx.createOscillator();
  const g = ctx.createGain();
  o.type = 'square';
  o.frequency.value = 220;
  g.gain.value = 0.02;
  o.connect(g); g.connect(ctx.destination);
  o.start();
  g.gain.exponentialRampToValueAtTime(0.0001, ctx.currentTime + 0.22);
  o.stop(ctx.currentTime + 0.24);
}
function playZap(){
  ensureAudio();
  const ctx = audioCtx;
  const o = ctx.createOscillator();
  const g = ctx.createGain();
  o.type = 'sawtooth';
  o.frequency.setValueAtTime(1200, ctx.currentTime);
  o.frequency.exponentialRampToValueAtTime(400, ctx.currentTime + 0.25);
  g.gain.value = 0.08;
  o.connect(g); g.connect(ctx.destination);
  g.gain.exponentialRampToValueAtTime(0.0001, ctx.currentTime + 0.28);
  o.start(); o.stop(ctx.currentTime + 0.3);
}

/* ==========================
   Utility helpers
   ========================== */

function randomGoalie(){
  return goalies[Math.floor(Math.random()*goalies.length)];
}
function applyGoalie(g){
  currentGoalie = g;
  goalieImg.src = g.url;
  goalieImg.alt = g.name;
  // small scale tweak for very large sprites
  goalieImg.style.width = '120px';
  // reset goalie position
  goalieImg.style.transform = 'translateY(0)';
}

/* ==========================
   Aim & Kick logic
   ========================== */

function getCenter(el){
  const r = el.getBoundingClientRect();
  return {x:r.left + r.width/2, y:r.top + r.height/2, r};
}

function fieldClickHandler(e){
  if(busy) return;
  busy = true;
  // choose a new random goalie each play (as requested)
  applyGoalie(randomGoalie());

  // coordinates
  const clickX = e.clientX;
  const clickY = e.clientY;

  const ballRect = ball.getBoundingClientRect();
  const ballCenter = { x: ballRect.left + ballRect.width/2, y: ballRect.top + ballRect.height/2 };

  // vector to target
  const dx = clickX - ballCenter.x;
  const dy = clickY - ballCenter.y;
  const distance = Math.sqrt(dx*dx + dy*dy);
  // force/power influenced by powerMode
  const baseSpeed = powerMode ? 1.6 : 1.0;
  // flight duration inversely proportional to speed & proportional to distance (clamped)
  const duration = Math.min(1.0 + (distance/900)/baseSpeed, 1.4/baseSpeed);

  // target transform
  const tx = dx;
  const ty = dy;

  // apply pikachu jump
  pikachu.style.transform = powerMode ? 'translateY(-140px)' : 'translateY(-72px)';

  // compute if this shot aims inside goal rect (predicted landing point)
  const goalRect = goal.getBoundingClientRect();
  const predictedX = ballCenter.x + tx;
  const predictedY = ballCenter.y + ty;

  const inGoal = (predictedX > goalRect.left && predictedX < goalRect.right && predictedY > goalRect.top && predictedY < goalRect.bottom);

  // goalie reaction: delay slightly then attempt to jump
  const goalieSkill = currentGoalie.skill; // higher -> better chance
  // goalie reaction time (smaller is faster)
  const reactionBase = 300; // ms
  const reaction = reactionBase * (1 - goalieSkill) + 150; // better skill -> smaller reaction
  const goalieJumpTime = Math.min(reaction, duration*1000*0.85); // but never instant

  // Compute block chance: if inGoal, goalie still can block with chance depending on skill and powerMode
  // Make sure goalie is usually late — reduce chance slightly
  const rawBlockChance = Math.max(0, goalieSkill - (powerMode ? 0.25 : 0.08));
  const blockChance = inGoal ? rawBlockChance * 0.9 : rawBlockChance * 0.6;

  // animate the ball: use transform and transitionDuration
  ball.style.transition = `transform ${duration}s cubic-bezier(.15,.9,.25,1)`;
  ball.style.transform = `translate(${tx}px, ${ty}px) rotate(${(powerMode?1080:720)}deg)`;

  // schedule goalie attempt
  let goalieWillBlock = Math.random() < blockChance;

  // If not aiming at goal, reduce blocking chances drastically
  if(!inGoal) goalieWillBlock = Math.random() < (goalieSkill*0.18);

  // setTimeout for goalie jump (visual)
  const goalieDelay = Math.max(120, goalieJumpTime);
  setTimeout(()=>{
    goalieImg.style.transform = 'translateY(-110px)'; // jump
  }, goalieDelay);

  // determine result when ball "arrives"
  setTimeout(()=>{

    // reset pikachu to neutral
    pikachu.style.transform = '';

    // goalie landing
    setTimeout(()=>{ goalieImg.style.transform = ''; }, 400);

    // If goalie blocked, show block; else goal
    if(goalieWillBlock && inGoal){
      // block success animation
      playBlock();
      // slight ball deflection
      ball.style.transition = 'transform 0.5s ease';
      // deflect to side (random)
      const deflectX = (Math.random() > 0.5 ? -1 : 1) * 140;
      const deflectY = -40;
      ball.style.transform = `translate(${tx + deflectX}px, ${ty + deflectY}px) rotate(${360}deg)`;
      // no score
      busy = false;
    } else if(inGoal){
      // Goal!
      score++;
      scoreEl.textContent = score;
      // net shake + lightning + cheer + celebration
      goal.classList.add('shake');
      lightning.classList.add('show');
      playZap();
      // small delay then cheer
      setTimeout(()=> playCheer(), 120);
      pikachu.classList.add('celebrate');

      // remove effects after a short while
      setTimeout(()=>{
        goal.classList.remove('shake');
        lightning.classList.remove('show');
        pikachu.classList.remove('celebrate');
      }, 900);

      // ball into net final position (a bit inside)
      ball.style.transition = 'transform 0.45s ease';
      const netCenterX = (goalRect.left + goalRect.right)/2 - ballCenter.x;
      const netCenterY = (goalRect.top + goalRect.bottom)/2 - ballCenter.y;
      ball.style.transform = `translate(${netCenterX}px, ${netCenterY}px) rotate(1080deg)`;
      setTimeout(()=> {
        // reset ball after celebration
        ball.style.transition = 'transform 0.5s ease';
        ball.style.transform = `translate(0px, 0px) rotate(0deg)`;
        busy = false;
      }, 1200);
    } else {
      // Miss: ball lands where clicked (no score)
      ball.style.transition = 'transform 0.6s ease';
      // return to original slowly
      setTimeout(()=>{
        ball.style.transition = 'transform 0.6s ease';
        ball.style.transform = `translate(0px, 0px) rotate(0deg)`;
        busy = false;
      }, 700);
    }
  }, duration*1000 + 90);
}

// handle power mode toggle (double-click/tap)
let dblTimer = null;
field.addEventListener('dblclick', ()=> {
  powerMode = !powerMode;
  powerTag.textContent = powerMode ? 'Power: ON' : 'Power: Normal';
  powerTag.style.background = powerMode ? 'rgba(255,200,60,0.95)' : 'rgba(0,0,0,0.5)';
});

// mobile-friendly single-tap / click to aim
field.addEventListener('click', fieldClickHandler);

/* Accessibility: allow keyboard kicks (space bar) — shoot to center of goal */
document.addEventListener('keydown', (e)=>{
  if(e.code === 'Space') {
    // simulate click at goal center
    const g = goal.getBoundingClientRect();
    const cx = g.left + g.width/2;
    const cy = g.top + g.height/2;
    fieldClickHandler({clientX: cx, clientY: cy});
  }
});

/* reset ball instantly on window resize to keep coordinates sane */
window.addEventListener('resize', ()=>{
  ball.style.transition = 'none';
  ball.style.transform = 'translate(0px,0px) rotate(0deg)';
  setTimeout(()=> ball.style.transition = '', 50);
});

/* ==========================
   Helpful starting behavior
   ========================== */
// small idle wiggle for goalie to feel alive
setInterval(()=>{
  if(Math.random() < 0.35){
    goalieImg.style.transform = 'translateY(-16px)';
    setTimeout(()=> goalieImg.style.transform = '', 220);
  }
}, 2500);

// Inform user about goalie selected each play via console (lightweight)
function announceGoalie(g){
  console.log('Goalie selected:', g.name, 'skill:', g.skill);
}
announceGoalie(currentGoalie);

</script>
</body>
</html>
