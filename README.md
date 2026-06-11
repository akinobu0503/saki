<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
<title>ヘソクエスト 〜へそ探検RPG〜</title>
<style>
  * { margin:0; padding:0; box-sizing:border-box; user-select:none; -webkit-user-select:none; }
  body {
    font-family: 'Hiragino Maru Gothic ProN', 'BIZ UDGothic', 'Yu Gothic', sans-serif;
    background: linear-gradient(180deg,#1a1c3a 0%,#2d2f5e 60%,#3e4178 100%);
    height:100vh; overflow:hidden; color:#fff;
    display:flex; align-items:center; justify-content:center;
  }
  #game {
    width:min(480px,100vw); height:min(800px,100vh);
    position:relative; overflow:hidden;
    background: linear-gradient(180deg,#7ec8e3 0%,#bfe9f5 55%,#8fd17e 56%,#6fbf63 100%);
    border-radius:12px; box-shadow:0 10px 40px rgba(0,0,0,.5);
    cursor:none; touch-action:manipulation;
  }
  .cloud { position:absolute; opacity:.85; animation: drift linear infinite; font-size:40px; }
  @keyframes drift { from{transform:translateX(-80px)} to{transform:translateX(560px)} }

  /* finger cursor */
  #finger {
    position:absolute; font-size:46px; pointer-events:none; z-index:50;
    transform: translate(-12px,-6px) rotate(-30deg);
    transition: transform .07s; filter: drop-shadow(2px 3px 2px rgba(0,0,0,.3));
  }
  #finger.poke { transform: translate(-12px,-6px) rotate(-30deg) scale(.78) translateY(8px); }

  /* HUD */
  #hud {
    position:absolute; top:0; left:0; right:0; padding:10px 14px; z-index:20;
    background: linear-gradient(180deg,rgba(0,0,30,.55),transparent);
  }
  .bar-wrap { background:rgba(0,0,0,.45); border-radius:10px; height:16px; overflow:hidden; border:2px solid rgba(255,255,255,.6); }
  .bar { height:100%; border-radius:8px; transition: width .25s; }
  #playerBar { background: linear-gradient(90deg,#4cd964,#a8f0a0); }
  #enemyBar  { background: linear-gradient(90deg,#ff5e57,#ffb199); }
  .label { font-size:12px; font-weight:bold; text-shadow:1px 1px 2px #000; margin-bottom:2px; display:flex; justify-content:space-between; }
  #enemyHud { margin-top:8px; }

  #stage { position:absolute; inset:0; }
  #enemyBox {
    position:absolute; left:50%; top:52%; transform:translate(-50%,-50%);
    width:340px; height:380px; z-index:5;
  }
  #enemySvg { width:100%; height:100%; overflow:visible; }
  .wobble { animation: wobble 2.6s ease-in-out infinite; transform-origin:170px 360px; }
  @keyframes wobble { 0%,100%{transform:rotate(-1.5deg)} 50%{transform:rotate(1.5deg)} }
  .hurt { animation: hurtAnim .45s; }
  @keyframes hurtAnim { 0%{transform:translateX(0)} 20%{transform:translateX(-14px)} 40%{transform:translateX(12px)} 60%{transform:translateX(-8px)} 80%{transform:translateX(6px)} 100%{transform:translateX(0)} }
  .enemyAttack { animation: lungeAnim .5s; }
  @keyframes lungeAnim { 0%{transform:translate(-50%,-50%) scale(1)} 40%{transform:translate(-50%,-38%) scale(1.18)} 100%{transform:translate(-50%,-50%) scale(1)} }
  .die { animation: dieAnim 1s forwards; }
  @keyframes dieAnim { to{ transform:translate(-50%,10%) rotate(14deg); opacity:0; } }
  .appear { animation: appearAnim .6s; }
  @keyframes appearAnim { from{ transform:translate(-50%,-50%) scale(0); } to{ transform:translate(-50%,-50%) scale(1);} }

  /* floating text */
  .float {
    position:absolute; z-index:40; font-weight:900; pointer-events:none;
    animation: floatUp 1s forwards; text-shadow:2px 2px 0 rgba(0,0,0,.35); white-space:nowrap;
  }
  @keyframes floatUp { 0%{transform:translateY(0) scale(.6); opacity:0} 15%{opacity:1; transform:translateY(-8px) scale(1.15)} 100%{transform:translateY(-70px) scale(1); opacity:0} }

  #msg {
    position:absolute; bottom:14px; left:12px; right:12px; z-index:20;
    background:rgba(0,0,30,.75); border:3px solid #fff; border-radius:12px;
    padding:10px 14px; font-size:15px; line-height:1.5; min-height:64px; font-weight:bold;
  }
  #lvlTag { position:absolute; top:8px; right:14px; font-size:13px; font-weight:bold; text-shadow:1px 1px 2px #000; z-index:25; }

  /* overlay screens */
  .screen {
    position:absolute; inset:0; z-index:100; display:flex; flex-direction:column;
    align-items:center; justify-content:center; text-align:center; gap:18px;
    background:rgba(15,16,45,.92); cursor:default; padding:24px;
  }
  .screen h1 { font-size:34px; text-shadow:0 0 18px #ffd45e; }
  .screen p { font-size:15px; line-height:1.9; }
  .btn {
    font-family:inherit; font-size:20px; font-weight:bold; padding:14px 38px;
    border-radius:999px; border:4px solid #fff; cursor:pointer;
    background:linear-gradient(180deg,#ffb347,#ff7b54); color:#fff;
    box-shadow:0 6px 0 #c0492f; transition:.1s;
  }
  .btn:active { transform:translateY(4px); box-shadow:0 2px 0 #c0492f; }
  .hidden { display:none !important; }
  .bigheso { font-size:80px; animation: pulse 1.4s infinite; }
  @keyframes pulse { 0%,100%{transform:scale(1)} 50%{transform:scale(1.12)} }
</style>
</head>
<body>
<div id="game">
  <div class="cloud" style="top:11%; animation-duration:34s;">☁️</div>
  <div class="cloud" style="top:20%; animation-duration:48s; animation-delay:-20s;">☁️</div>

  <div id="hud" class="hidden">
    <div class="label"><span id="playerName">ゆうしゃ Lv.1</span><span id="playerHpText"></span></div>
    <div class="bar-wrap"><div id="playerBar" class="bar" style="width:100%"></div></div>
    <div id="enemyHud">
      <div class="label"><span id="enemyName"></span><span id="enemyHpText"></span></div>
      <div class="bar-wrap"><div id="enemyBar" class="bar" style="width:100%"></div></div>
    </div>
  </div>

  <div id="stage">
    <div id="enemyBox"></div>
  </div>

  <div id="msg" class="hidden"></div>
  <div id="finger" class="hidden">👆</div>

  <!-- Title screen -->
  <div id="title" class="screen">
    <div class="bigheso">🌀</div>
    <h1>ヘソクエスト</h1>
    <p>〜へそ探検RPG〜<br><br>
    おなかを出したモンスターたちの<br><b>「へそ」に指を入れて</b>倒せ!<br><br>
    🌀 へそをタップ → ダメージ!<br>
    🙅 ガード中につつくと反撃される!<br>
    ✨ 金色のへそは大チャンス!</p>
    <button class="btn" onclick="startGame()">ぼうけんに でる</button>
  </div>

  <!-- Result screen -->
  <div id="result" class="screen hidden">
    <div id="resultEmoji" class="bigheso"></div>
    <h1 id="resultTitle"></h1>
    <p id="resultText"></p>
    <button class="btn" onclick="startGame()">もういちど</button>
  </div>
</div>

<script>
'use strict';
const $ = id => document.getElementById(id);
const game = $('game'), enemyBox = $('enemyBox'), finger = $('finger'), msg = $('msg');

/* ---------- enemy data ---------- */
const ENEMIES = [
  { name:'へそスライム', hp:30, atk:6,  color:'#6fd3f7', dark:'#3aa8d8', belly:'#d9f6ff',
    guardRate:.18, navelSpeed:2600, intro:'ぷるぷる… へそスライムが あらわれた!', type:'slime' },
  { name:'おなかグマ', hp:55, atk:10, color:'#b07b4f', dark:'#8a5a33', belly:'#e8c79a',
    guardRate:.30, navelSpeed:2100, intro:'のっしのっし! おなかグマが あらわれた!', type:'bear' },
  { name:'はらだしオーガ', hp:80, atk:15, color:'#7ac96f', dark:'#4f9a46', belly:'#c9eab8',
    guardRate:.42, navelSpeed:1600, intro:'ガオォ! はらだしオーガが あらわれた!', type:'oni' },
  { name:'まおう・デベソス', hp:120, atk:20, color:'#9b6fd3', dark:'#6f44a8', belly:'#e3d0f7',
    guardRate:.5, navelSpeed:1200, intro:'フハハハ! まおうデベソスが あらわれた!\nわがヘソは うちゅうの しんえんだ!', type:'boss' },
];

/* ---------- state ---------- */
let S = null, timers = [];
function clearTimers(){ timers.forEach(t=>clearTimeout(t)); timers=[]; }
function later(fn,ms){ const t=setTimeout(fn,ms); timers.push(t); return t; }

function startGame(){
  clearTimers();
  S = { lv:1, hp:60, maxHp:60, atkMin:8, atkMax:14, idx:0, enemy:null, eHp:0,
        guarding:false, golden:false, busy:false, over:false };
  $('title').classList.add('hidden');
  $('result').classList.add('hidden');
  $('hud').classList.remove('hidden');
  msg.classList.remove('hidden');
  finger.classList.remove('hidden');
  updateHud();
  spawnEnemy();
}

/* ---------- SVG monster ---------- */
function monsterSvg(e){
  const horns = (e.type==='oni'||e.type==='boss') ?
    `<path d="M110 70 L95 25 L135 55 Z" fill="#ffe9a8" stroke="#caa84e" stroke-width="4"/>
     <path d="M230 70 L245 25 L205 55 Z" fill="#ffe9a8" stroke="#caa84e" stroke-width="4"/>` : '';
  const ears = e.type==='bear' ?
    `<circle cx="105" cy="62" r="26" fill="${e.color}" stroke="${e.dark}" stroke-width="5"/>
     <circle cx="235" cy="62" r="26" fill="${e.color}" stroke="${e.dark}" stroke-width="5"/>` : '';
  const crown = e.type==='boss' ?
    `<path d="M135 28 L145 6 L160 24 L170 0 L180 24 L195 6 L205 28 Z" fill="#ffd45e" stroke="#caa84e" stroke-width="3"/>` : '';
  const slime = e.type==='slime';
  return `
  <svg id="enemySvg" viewBox="0 0 340 380" xmlns="http://www.w3.org/2000/svg">
    <g id="mBody" class="wobble">
      <ellipse cx="170" cy="368" rx="120" ry="14" fill="rgba(0,0,0,.18)"/>
      ${ears}${horns}${crown}
      <!-- body -->
      ${slime
        ? `<path d="M50 330 Q40 140 170 100 Q300 140 290 330 Q230 358 170 356 Q110 358 50 330 Z" fill="${e.color}" stroke="${e.dark}" stroke-width="6"/>`
        : `<ellipse cx="170" cy="115" rx="78" ry="68" fill="${e.color}" stroke="${e.dark}" stroke-width="6"/>
           <ellipse cx="170" cy="255" rx="105" ry="105" fill="${e.color}" stroke="${e.dark}" stroke-width="6"/>
           <ellipse cx="62" cy="225" rx="24" ry="46" fill="${e.color}" stroke="${e.dark}" stroke-width="6" transform="rotate(18 62 225)"/>
           <ellipse cx="278" cy="225" rx="24" ry="46" fill="${e.color}" stroke="${e.dark}" stroke-width="6" transform="rotate(-18 278 225)"/>
           <ellipse cx="120" cy="352" rx="32" ry="18" fill="${e.dark}"/>
           <ellipse cx="220" cy="352" rx="32" ry="18" fill="${e.dark}"/>`}
      <!-- belly -->
      <ellipse id="belly" cx="170" cy="262" rx="78" ry="74" fill="${e.belly}" stroke="${e.dark}" stroke-width="4" opacity="0.97"/>
      <!-- face -->
      <g id="face">
        <circle cx="140" cy="${slime?170:108}" r="9" fill="#222"/>
        <circle cx="200" cy="${slime?170:108}" r="9" fill="#222"/>
        <circle cx="143" cy="${slime?167:105}" r="3" fill="#fff"/>
        <circle cx="203" cy="${slime?167:105}" r="3" fill="#fff"/>
        <path id="mouth" d="M150 ${slime?196:134} Q170 ${slime?210:148} 190 ${slime?196:134}" stroke="#222" stroke-width="5" fill="none" stroke-linecap="round"/>
        <ellipse cx="118" cy="${slime?190:128}" rx="11" ry="7" fill="rgba(255,120,120,.45)"/>
        <ellipse cx="222" cy="${slime?190:128}" rx="11" ry="7" fill="rgba(255,120,120,.45)"/>
      </g>
      <!-- navel -->
      <g id="navel" style="cursor:none">
        <circle id="navelGlow" cx="0" cy="0" r="24" fill="rgba(255,215,90,.55)" opacity="0"/>
        <ellipse cx="0" cy="0" rx="13" ry="15" fill="#8a5a33"/>
        <ellipse cx="0" cy="-2" rx="8" ry="9" fill="#5d3a1d"/>
        <path d="M-4 -8 Q0 -12 5 -7" stroke="rgba(255,255,255,.5)" stroke-width="2.5" fill="none"/>
      </g>
      <!-- guard arms -->
      <g id="guard" opacity="0" style="pointer-events:none">
        <ellipse cx="170" cy="262" rx="86" ry="80" fill="${e.color}" stroke="${e.dark}" stroke-width="6"/>
        <text x="170" y="276" font-size="40" text-anchor="middle">🙅</text>
        <text x="170" y="225" font-size="20" text-anchor="middle" fill="#fff" font-weight="bold" stroke="${e.dark}" stroke-width="0.5">ガード!</text>
      </g>
    </g>
  </svg>`;
}

/* ---------- spawn / navel movement ---------- */
function spawnEnemy(){
  const e = ENEMIES[S.idx];
  S.enemy = e; S.eHp = e.hp; S.guarding=false; S.golden=false; S.busy=false;
  enemyBox.innerHTML = monsterSvg(e);
  enemyBox.classList.remove('die');
  enemyBox.classList.add('appear');
  later(()=>enemyBox.classList.remove('appear'),700);
  $('enemyName').textContent = e.name;
  updateHud();
  say(e.intro);
  moveNavel(true);
  scheduleNavel();
  scheduleEnemyAction();
}

function navelEl(){ return enemyBox.querySelector('#navel'); }

function moveNavel(instant){
  const n = navelEl(); if(!n) return;
  // random position within belly ellipse (cx170 cy262 rx78 ry74), margin
  const a = Math.random()*Math.PI*2, r = Math.sqrt(Math.random())*0.62;
  const x = 170 + Math.cos(a)*78*r, y = 262 + Math.sin(a)*74*r;
  n.style.transition = instant ? 'none' : 'transform 0.9s ease-in-out';
  n.setAttribute('transform','');
  n.style.transform = `translate(${x}px,${y}px)`;
}

function scheduleNavel(){
  if(S.over) return;
  later(()=>{ if(!S.over && !S.busy){ moveNavel(false); } scheduleNavel(); }, S.enemy.navelSpeed);
}

/* ---------- enemy actions: guard / golden / attack ---------- */
function scheduleEnemyAction(){
  if(S.over) return;
  later(()=>{
    if(S.over || S.busy){ scheduleEnemyAction(); return; }
    const roll = Math.random();
    if(roll < S.enemy.guardRate) doGuard();
    else if(roll < S.enemy.guardRate + 0.15) doGolden();
    else doEnemyAttack();
    scheduleEnemyAction();
  }, 2300 + Math.random()*1700);
}

function doGuard(){
  if(S.guarding||S.golden) return;
  S.guarding = true;
  const g = enemyBox.querySelector('#guard'); if(g){ g.setAttribute('opacity','1'); g.style.pointerEvents='auto'; }
  say(`${S.enemy.name}は おなかを ガードした! いまは つつくな!`);
  later(()=>{ S.guarding=false; const g2=enemyBox.querySelector('#guard'); if(g2){ g2.setAttribute('opacity','0'); g2.style.pointerEvents='none'; } }, 1400+Math.random()*800);
}

function doGolden(){
  if(S.guarding||S.golden) return;
  S.golden = true;
  const glow = enemyBox.querySelector('#navelGlow'); if(glow) glow.setAttribute('opacity','1');
  say('✨ へそが きんいろに かがやいている! いまだ!!');
  later(()=>{ S.golden=false; const g=enemyBox.querySelector('#navelGlow'); if(g) g.setAttribute('opacity','0'); }, 1600);
}

function doEnemyAttack(){
  if(S.over) return;
  const dmg = Math.floor(S.enemy.atk * (0.8 + Math.random()*0.4));
  enemyBox.classList.add('enemyAttack');
  later(()=>enemyBox.classList.remove('enemyAttack'),500);
  S.hp = Math.max(0, S.hp - dmg);
  floatText(`-${dmg}`, '50%', '86%', '#ff5e57', 26);
  say(`${S.enemy.name}の こうげき! ゆうしゃに ${dmg} のダメージ!`);
  updateHud();
  if(S.hp <= 0) lose();
}

/* ---------- player poke ---------- */
game.addEventListener('pointerdown', ev=>{
  if(!S || S.over) return;
  finger.classList.add('poke');
  setTimeout(()=>finger.classList.remove('poke'),120);

  const target = ev.target.closest ? ev.target : null;
  const inNavel = target && target.closest('#navel');
  const inGuard = target && target.closest('#guard');
  const inBelly = target && (target.closest('#belly') || target.id==='belly');

  if(S.busy) return;

  if(S.guarding && (inGuard || inBelly || inNavel)){
    // poked a guarding enemy → counter
    const dmg = Math.floor(S.enemy.atk * 1.3);
    S.hp = Math.max(0,S.hp-dmg);
    floatText('はんげき!', ev.clientX, ev.clientY, '#ff5e57', 22, true);
    say(`ガードちゅうに つついた! はんげきで ${dmg} のダメージ!`);
    updateHud();
    if(S.hp<=0) lose();
    return;
  }
  if(inNavel){
    pokeNavel(ev);
  } else if(inBelly){
    floatText('ぷに…', ev.clientX, ev.clientY, '#fff', 18, true);
    say('おなかは ぷにぷにだ。でも へそじゃない…!');
  }
});

function pokeNavel(ev){
  const crit = Math.random()<0.15;
  let dmg = S.atkMin + Math.floor(Math.random()*(S.atkMax-S.atkMin+1));
  let label = `-${dmg}`;
  if(S.golden){ dmg*=3; label=`✨-${dmg}✨`; }
  else if(crit){ dmg=Math.floor(dmg*1.8); label=`かいしん! -${dmg}`; }
  S.eHp = Math.max(0, S.eHp - dmg);

  const body = enemyBox.querySelector('#mBody');
  if(body){ body.classList.remove('hurt'); void body.offsetWidth; body.classList.add('hurt'); }
  const mouth = enemyBox.querySelector('#mouth');
  if(mouth){ mouth.setAttribute('d', mouth.getAttribute('d').replace(/Q(\d+) (\d+)/, (m,a,b)=>`Q${a} ${+b-26}`)); setTimeout(()=>{ try{mouth.setAttribute('d', mouth.getAttribute('d').replace(/Q(\d+) (\d+)/,(m,a,b)=>`Q${a} ${+b+26}`));}catch(e){} },400); }

  floatText(label, ev.clientX, ev.clientY, S.golden?'#ffd45e':(crit?'#ffd45e':'#fff'), S.golden?30:24, true);
  const lines = ['ずぼっ! へそに ゆびが はいった!','ぐりぐり! ' + S.enemy.name + 'は くすぐったがっている!','ぷにっ! かんぺきな へそアタック!','むにゅっ! ' + S.enemy.name + 'は ひるんだ!'];
  say(S.golden ? '✨ おうごんのへそに クリティカルヒット!!' : lines[Math.floor(Math.random()*lines.length)]);
  updateHud();
  moveNavel(false);
  if(S.eHp<=0) beatEnemy();
}

/* ---------- progression ---------- */
function beatEnemy(){
  S.busy = true; S.over = false;
  clearTimers();
  enemyBox.classList.add('die');
  say(`${S.enemy.name}を たおした!`);
  later(()=>{
    S.idx++;
    if(S.idx >= ENEMIES.length){ win(); return; }
    // level up
    S.lv++; S.maxHp+=15; S.hp=S.maxHp; S.atkMin+=4; S.atkMax+=5;
    floatText(`LEVEL UP! Lv.${S.lv}`, '50%', '40%', '#ffd45e', 30);
    say(`レベルが ${S.lv}に あがった! HPがぜんかいふくした!`);
    updateHud();
    later(spawnEnemy, 1400);
  }, 1300);
}

function win(){
  S.over=true; clearTimers();
  $('hud').classList.add('hidden'); msg.classList.add('hidden'); finger.classList.add('hidden');
  $('resultEmoji').textContent='🏆';
  $('resultTitle').textContent='せかいに へいわが もどった!';
  $('resultText').innerHTML=`まおうデベソスの へそto……もとい、やぼうは くだかれた。<br>きみこそ しんの <b>へそマスター</b> だ!<br><br>とうたつレベル: Lv.${S.lv}`;
  $('result').classList.remove('hidden');
}

function lose(){
  S.over=true; clearTimers();
  $('hud').classList.add('hidden'); msg.classList.add('hidden'); finger.classList.add('hidden');
  $('resultEmoji').textContent='😵';
  $('resultTitle').textContent='ゆうしゃは ちからつきた…';
  $('resultText').innerHTML=`${S.enemy.name}は つよかった…<br>ガードちゅうは つつかず、✨きんいろのへそ✨を ねらおう!`;
  $('result').classList.remove('hidden');
}

/* ---------- UI helpers ---------- */
function updateHud(){
  $('playerName').textContent = `ゆうしゃ Lv.${S.lv}`;
  $('playerHpText').textContent = `${S.hp}/${S.maxHp}`;
  $('playerBar').style.width = (100*S.hp/S.maxHp)+'%';
  if(S.enemy){
    $('enemyHpText').textContent = `${S.eHp}/${S.enemy.hp}`;
    $('enemyBar').style.width = (100*S.eHp/S.enemy.hp)+'%';
  }
}
function say(t){ msg.textContent = t; }

function floatText(text, x, y, color, size, clientCoords){
  const el = document.createElement('div');
  el.className='float'; el.textContent=text;
  el.style.color=color; el.style.fontSize=size+'px';
  const rect = game.getBoundingClientRect();
  if(clientCoords){ el.style.left=(x-rect.left-20)+'px'; el.style.top=(y-rect.top-30)+'px'; }
  else { el.style.left=x; el.style.top=y; el.style.transform='translateX(-50%)'; }
  game.appendChild(el);
  setTimeout(()=>el.remove(),1000);
}

/* finger follows pointer */
game.addEventListener('pointermove', ev=>{
  const rect = game.getBoundingClientRect();
  finger.style.left=(ev.clientX-rect.left)+'px';
  finger.style.top=(ev.clientY-rect.top)+'px';
});
game.addEventListener('pointerdown', ev=>{
  const rect = game.getBoundingClientRect();
  finger.style.left=(ev.clientX-rect.left)+'px';
  finger.style.top=(ev.clientY-rect.top)+'px';
});
</script>
</body>
</html>
