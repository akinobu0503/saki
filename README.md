<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
<title>へそリズム 〜へその達人〜</title>
<style>
  :root{
    --bg1:#2a1430; --bg2:#140a1c; --ink:#fff3e6; --accent:#ffcf4d;
    --don:#ff5a6e; --ka:#4db8ff; --good:#5fe089; --lane:#3a2348;
  }
  *{box-sizing:border-box;-webkit-tap-highlight-color:transparent;user-select:none;}
  html,body{margin:0;height:100%;}
  body{background:radial-gradient(circle at 50% 0%,var(--bg1),var(--bg2) 70%);
    color:var(--ink);font-family:"Hiragino Maru Gothic ProN","Yu Gothic",sans-serif;
    display:flex;align-items:center;justify-content:center;min-height:100vh;padding:8px;
    overflow:hidden;touch-action:none;}
  #app{position:relative;width:100%;max-width:600px;background:#241334;border:3px solid #4a2d5e;border-radius:20px;
    overflow:hidden;box-shadow:0 18px 50px rgba(0,0,0,.5);}
  header{display:flex;justify-content:space-between;align-items:center;padding:10px 16px;}
  h1{margin:0;font-size:20px;color:var(--accent);}
  .scorebox{text-align:right;font-size:12px;line-height:1.4;}
  .scorebox b{font-size:20px;color:var(--accent);}
  /* lyric */
  #lyric{height:46px;display:flex;align-items:center;justify-content:center;
    font-size:26px;font-weight:bold;color:#ffe9b0;letter-spacing:2px;
    text-shadow:0 2px 8px rgba(255,180,80,.4);transition:transform .15s,opacity .3s;}
  /* lane */
  #lane{position:relative;height:140px;background:linear-gradient(#2c1a3c,#3a2348);
    border-top:3px solid #4a2d5e;border-bottom:3px solid #4a2d5e;overflow:hidden;}
  #judge{position:absolute;top:50%;left:78px;width:64px;height:64px;margin-top:-32px;margin-left:-32px;
    border:4px solid rgba(255,255,255,.45);border-radius:50%;box-sizing:border-box;}
  #judge.flash{animation:jflash .18s;}
  @keyframes jflash{0%{transform:scale(1.25);border-color:#fff;}100%{transform:scale(1);}}
  .note{position:absolute;top:50%;width:48px;height:48px;margin-top:-24px;margin-left:-24px;
    border-radius:50%;display:flex;align-items:center;justify-content:center;font-size:22px;
    border:3px solid rgba(255,255,255,.6);will-change:left;}
  .note.don{background:radial-gradient(circle at 35% 30%,#ff96a4,var(--don));}
  .note.ka{background:radial-gradient(circle at 35% 30%,#a6dcff,var(--ka));}
  .note.big{transform:scale(1.18);}
  .hero{position:absolute;top:50%;left:18px;margin-top:-26px;font-size:42px;
    transition:transform .08s;}
  .hero.bop{transform:translateY(-6px) scale(1.1);}
  .judgetext{position:absolute;left:78px;top:8px;transform:translateX(-50%);
    font-weight:bold;font-size:18px;pointer-events:none;opacity:0;}
  .judgetext.show{animation:jt .5s;}
  @keyframes jt{0%{opacity:1;transform:translate(-50%,0)}100%{opacity:0;transform:translate(-50%,-22px)}}
  /* drum buttons */
  #drums{display:grid;grid-template-columns:1fr 1fr;gap:10px;padding:14px 16px;}
  .drum{height:74px;border-radius:16px;border:none;font-size:17px;font-weight:bold;color:#fff;
    cursor:pointer;display:flex;flex-direction:column;align-items:center;justify-content:center;gap:2px;}
  .drum small{font-weight:normal;opacity:.85;font-size:12px;}
  .drum.don{background:linear-gradient(#ff5a6e,#d83a52);box-shadow:0 5px 0 #a92a3e;}
  .drum.ka{background:linear-gradient(#4db8ff,#2f8fe0);box-shadow:0 5px 0 #1d6cb0;}
  .drum:active{transform:translateY(3px);box-shadow:0 2px 0 rgba(0,0,0,.4);}
  .combo{text-align:center;height:30px;font-size:18px;font-weight:bold;color:var(--accent);}
  .combo b{font-size:24px;}
  /* overlays */
  .overlay{position:absolute;inset:0;background:rgba(20,10,28,.94);z-index:20;
    display:flex;flex-direction:column;align-items:center;justify-content:center;gap:16px;
    padding:26px;text-align:center;}
  .overlay .e{font-size:70px;}
  .overlay p{margin:0;font-size:14px;line-height:1.7;color:#d8c6e6;}
  button.big{font-family:inherit;background:var(--accent);color:#3a2150;font-weight:bold;
    font-size:18px;padding:13px 30px;border:none;border-radius:14px;cursor:pointer;
    box-shadow:0 5px 0 #c89a2e;}
  button.big:active{transform:translateY(3px);box-shadow:0 2px 0 #c89a2e;}
  #count{font-size:90px;font-weight:bold;color:var(--accent);}
  .res-row{display:flex;gap:18px;font-size:15px;}
  .res-row div b{display:block;font-size:26px;}
  .rank{font-size:40px;font-weight:bold;color:var(--accent);}
  .tip{font-size:12px;color:#a890b8;}
</style>
</head>
<body>
<div id="app">
  <header>
    <h1>🥁 へそリズム</h1>
    <div class="scorebox">スコア<br><b id="score">0</b></div>
  </header>
  <div id="lyric">♪ へその歌 ♪</div>
  <div id="lane">
    <div class="hero" id="hero">🧍</div>
    <div id="judge"></div>
    <div class="judgetext" id="jtext"></div>
    <!-- notes injected -->
  </div>
  <div class="combo" id="combo"></div>
  <div id="drums">
    <button class="drum don" onpointerdown="Game.hit('don',event)">ドン<small>赤・中央（Fキー）</small></button>
    <button class="drum ka" onpointerdown="Game.hit('ka',event)">カッ<small>青・ふち（Jキー）</small></button>
  </div>

  <div class="overlay" id="startov">
    <div class="e">🥁🫛</div>
    <h1 style="color:var(--accent);font-size:26px">へそリズム 〜へその達人〜</h1>
    <p>流れてくる音符が <b>へそ(判定の輪)</b> に重なった<br>しゅんかんに たたこう！<br>
      <span style="color:var(--don)">●ドン</span>＝赤ボタン/Fキー
      <span style="color:var(--ka)">●カッ</span>＝青ボタン/Jキー</p>
    <button class="big" onclick="Game.begin()">▶ えんそう スタート</button>
    <div class="tip">『へその歌』が流れます。音量はほどほどに🔊</div>
  </div>
  <div class="overlay" id="countov" style="display:none"><div id="count">3</div></div>
  <div class="overlay" id="resultov" style="display:none">
    <div class="rank" id="rank"></div>
    <div class="e" id="resemo">🎉</div>
    <div class="res-row">
      <div>良<b id="r-great" style="color:var(--good)">0</b></div>
      <div>可<b id="r-good" style="color:var(--accent)">0</b></div>
      <div>不可<b id="r-miss" style="color:var(--don)">0</b></div>
      <div>最大<br>コンボ<b id="r-combo">0</b></div>
    </div>
    <p>スコア <b id="r-score" style="font-size:22px;color:var(--accent)">0</b></p>
    <div id="password" style="background:#3a2348;border:2px dashed var(--accent);border-radius:12px;
      padding:12px 18px;font-size:16px;color:#fff;">🔑 クリアパスワード： <b style="font-size:26px;letter-spacing:6px;color:var(--accent)">1234</b></div>
    <button class="big" onclick="location.reload()">🔁 もう一度</button>
  </div>
</div>

<script>
const $=id=>document.getElementById(id);

/* ===== Audio ===== */
let AC=null;
function audioInit(){ if(!AC){AC=new (window.AudioContext||window.webkitAudioContext)();} if(AC.state==='suspended')AC.resume(); }
const NOTE={C5:523.25,D5:587.33,E5:659.25,F5:698.46,G5:783.99,A5:880.0,B5:987.77,C6:1046.5,
            G4:392.0,A4:440.0,E4:329.63,C4:261.63};

function tone(freq,t,dur,type='triangle',vol=0.16){
  const o=AC.createOscillator(),g=AC.createGain();
  o.type=type;o.frequency.setValueAtTime(freq,t);
  g.gain.setValueAtTime(0,t);
  g.gain.linearRampToValueAtTime(vol,t+0.02);
  g.gain.exponentialRampToValueAtTime(0.001,t+dur);
  o.connect(g).connect(AC.destination);o.start(t);o.stop(t+dur+0.05);
}
function bass(freq,t,dur){
  const o=AC.createOscillator(),g=AC.createGain();
  o.type='sine';o.frequency.setValueAtTime(freq,t);
  g.gain.setValueAtTime(0.0001,t);g.gain.linearRampToValueAtTime(0.12,t+0.02);
  g.gain.exponentialRampToValueAtTime(0.001,t+dur);
  o.connect(g).connect(AC.destination);o.start(t);o.stop(t+dur+0.05);
}
function kick(t){
  const o=AC.createOscillator(),g=AC.createGain();
  o.type='sine';o.frequency.setValueAtTime(150,t);o.frequency.exponentialRampToValueAtTime(50,t+0.12);
  g.gain.setValueAtTime(0.22,t);g.gain.exponentialRampToValueAtTime(0.001,t+0.16);
  o.connect(g).connect(AC.destination);o.start(t);o.stop(t+0.2);
}
function donSfx(){ if(!AC)return; const t=AC.currentTime;
  const o=AC.createOscillator(),g=AC.createGain();
  o.type='sine';o.frequency.setValueAtTime(220,t);o.frequency.exponentialRampToValueAtTime(80,t+0.12);
  g.gain.setValueAtTime(0.3,t);g.gain.exponentialRampToValueAtTime(0.001,t+0.15);
  o.connect(g).connect(AC.destination);o.start(t);o.stop(t+0.18);}
function kaSfx(){ if(!AC)return; const t=AC.currentTime;
  const o=AC.createOscillator(),g=AC.createGain();
  o.type='triangle';o.frequency.setValueAtTime(900,t);o.frequency.exponentialRampToValueAtTime(500,t+0.08);
  g.gain.setValueAtTime(0.22,t);g.gain.exponentialRampToValueAtTime(0.001,t+0.1);
  o.connect(g).connect(AC.destination);o.start(t);o.stop(t+0.12);}

/* ===== Song: 『へその歌』 =====
   each event: [beat, noteName|null, lyric, durBeats]  */
const BPM=128, BEAT=60/BPM, LEAD=4;   // 4 beats lead-in
function buildSong(){
  const m=[];   // melody+lyric events
  // helper to push a phrase from compact tokens
  // token: [pitch, dur, lyric]
  let b=0;
  const seq=[
    // ねえねえ みんなの おへそ
    ['E5',1,'ね'],['E5',1,'え'],['G5',1,'み'],['G5',1,'ん'],['A5',1,'な'],['G5',1,'の'],['E5',2,'おへそ'],
    // まんなか ぽっこり かわいいな
    ['D5',1,'ま'],['D5',1,'ん'],['E5',1,'な'],['G5',1,'か'],['E5',1,'ぽっ'],['D5',1,'こり'],['C5',2,'かわいい'],
    // ごまが たまったら とっちゃおう
    ['E5',1,'ご'],['E5',1,'ま'],['G5',1,'た'],['A5',1,'まっ'],['C6',1,'たら'],['A5',1,'とっ'],['G5',2,'ちゃおう'],
    // へそ へそ せかいいち
    ['G5',1,'へ'],['A5',1,'そ'],['G5',1,'へ'],['E5',1,'そ'],['G5',1,'せ'],['A5',1,'かい'],['C6',2,'いち'],
    // ぽんぽん おなか げんきのしるし
    ['C6',1,'ぽん'],['A5',1,'ぽん'],['G5',1,'お'],['E5',1,'なか'],['G5',1,'げん'],['A5',1,'きの'],['G5',2,'しるし'],
    // みんなの おへそ せかいいち！
    ['E5',1,'み'],['G5',1,'ん'],['A5',1,'な'],['C6',1,'の'],['A5',1,'お'],['G5',1,'へそ'],['C6',2,'いち！'],
  ];
  for(const [p,d,w] of seq){ m.push({beat:b, note:NOTE[p], lyric:w, dur:d}); b+=d; }
  const totalBeats=b;
  // hit notes: one per syllable, color by pitch
  const hits=m.map(e=>({beat:e.beat, type:(e.note>=NOTE.A5?'ka':'don'),
                        big:(e.dur>=2), judged:false}));
  return {melody:m, hits, totalBeats};
}

/* ===== Game ===== */
const Game={
  song:null, T0:0, raf:0, running:false,
  score:0, combo:0, maxCombo:0, great:0, good:0, miss:0,
  mi:0, bi:0, li:0, noteEls:[],

  begin(){
    audioInit();
    this.song=buildSong();
    $('startov').style.display='none';
    this.countdown(3);
  },
  countdown(n){
    const ov=$('countov'); ov.style.display='flex'; $('count').textContent=n;
    if(AC){ tone(NOTE.C5, AC.currentTime, 0.1,'square',0.12); }
    if(n>1){ setTimeout(()=>this.countdown(n-1),700); }
    else { setTimeout(()=>{ ov.style.display='none'; this.startPlay(); },700); }
  },

  startPlay(){
    this.running=true;
    this.T0=AC.currentTime + LEAD*BEAT;   // first hit lands at T0
    // schedule lane (note elements created lazily as they approach)
    this.lane=$('lane');
    this.laneW=this.lane.clientWidth;
    this.judgeX=78;
    this.SPEED=(this.laneW-this.judgeX)/ (LEAD*BEAT) * 1.0; // px per second (so spawn at right edge ~LEAD beats early)
    this.loop();
  },

  // audio time of a beat
  atime(beat){ return this.T0 + beat*BEAT; },

  loop(){
    if(!this.running) return;
    const now=AC.currentTime;
    const look=now+0.18;
    // schedule melody + bass
    while(this.mi<this.song.melody.length && this.atime(this.song.melody[this.mi].beat) < look){
      const e=this.song.melody[this.mi];
      const t=this.atime(e.beat);
      tone(e.note, t, e.dur*BEAT*0.92, 'triangle', 0.15);
      // soft harmony a third below occasionally
      if(this.mi%2===0) tone(e.note/1.26, t, e.dur*BEAT*0.5, 'sine', 0.05);
      this.mi++;
    }
    // kick on every beat + bass each 2 beats
    while(this.atime(this.bi) < look){
      const t=this.atime(this.bi);
      kick(t);
      if(this.bi%2===0){ const bn=[NOTE.C4,NOTE.G4,NOTE.A4,NOTE.E4][(this.bi/2)%4]; bass(bn,t,BEAT*1.6); }
      this.bi++;
      if(this.bi>this.song.totalBeats+2) break;
    }

    // spawn note elements ~LEAD beats before hit
    for(const h of this.song.hits){
      if(h.el||h.judged) continue;
      const dt=this.atime(h.beat)-now;
      if(dt < LEAD*BEAT + 0.3){
        const el=document.createElement('div');
        el.className='note '+h.type+(h.big?' big':'');
        el.textContent=h.type==='don'?'●':'○';
        this.lane.appendChild(el); h.el=el;
      }
    }
    // move notes
    for(const h of this.song.hits){
      if(!h.el) continue;
      const dt=this.atime(h.beat)-now;            // >0 means upcoming
      const x=this.judgeX + dt*this.SPEED;
      h.el.style.left=x+'px';
      if(!h.judged && dt < -0.16){                // passed: miss
        h.judged=true; this.registerMiss();
        h.el.style.transition='opacity .2s'; h.el.style.opacity=0;
        setTimeout(()=>h.el&&h.el.remove(),200);
      }
    }
    // update lyric (show lyric whose note time is closest just passed)
    while(this.li<this.song.melody.length && this.atime(this.song.melody[this.li].beat) <= now+0.02){
      const w=this.song.melody[this.li].lyric;
      const L=$('lyric'); L.textContent=w; L.style.transform='scale(1.18)';
      setTimeout(()=>L.style.transform='scale(1)',120);
      $('hero').classList.add('bop'); setTimeout(()=>$('hero').classList.remove('bop'),120);
      this.li++;
    }

    // end condition
    if(now > this.atime(this.song.totalBeats) + 1.2){ this.finish(); return; }
    this.raf=requestAnimationFrame(()=>this.loop());
  },

  hit(type,ev){ if(ev)ev.preventDefault();
    if(!this.running) return;
    type==='don'?donSfx():kaSfx();
    $('judge').classList.remove('flash'); void $('judge').offsetWidth; $('judge').classList.add('flash');
    const now=AC.currentTime;
    // find nearest unjudged note of same type within window
    let best=null,bestAbs=1;
    for(const h of this.song.hits){
      if(h.judged||h.type!==type) continue;
      const dt=Math.abs(this.atime(h.beat)-now);
      if(dt<bestAbs){bestAbs=dt;best=h;}
    }
    if(best && bestAbs<=0.13){
      best.judged=true;
      let label,gain;
      if(bestAbs<=0.055){label='良！';gain=300;this.great++;}
      else {label='可';gain=120;this.good++;}
      this.combo++; this.maxCombo=Math.max(this.maxCombo,this.combo);
      this.score+=gain+this.combo*2;
      this.showJudge(label, label==='良！'?'var(--good)':'var(--accent)');
      best.el && (best.el.style.transition='transform .15s,opacity .15s',
                  best.el.style.transform='scale(1.6)',best.el.style.opacity=0);
      setTimeout(()=>best.el&&best.el.remove(),160);
      this.refresh();
    }
    // mistapped (no note) -> no penalty, just sound
  },

  registerMiss(){ this.miss++; this.combo=0; this.showJudge('不可','var(--don)'); this.refresh(); },

  showJudge(t,c){ const j=$('jtext'); j.textContent=t; j.style.color=c;
    j.classList.remove('show'); void j.offsetWidth; j.classList.add('show'); },

  refresh(){ $('score').textContent=this.score;
    $('combo').innerHTML=this.combo>1?`<b>${this.combo}</b> コンボ`:''; },

  finish(){
    this.running=false; cancelAnimationFrame(this.raf);
    const total=this.great+this.good+this.miss;
    const acc=total? (this.great+this.good*0.5)/total : 0;
    let rank,emo,msg;
    if(this.miss===0 && acc>0.9){rank='へその達人！';emo='👑';}
    else if(acc>=0.85){rank='スッキリ名人';emo='🌟';}
    else if(acc>=0.6){rank='へそ見習い';emo='🫛';}
    else {rank='もう一回！';emo='💪';}
    $('rank').textContent=rank; $('resemo').textContent=emo;
    $('r-great').textContent=this.great; $('r-good').textContent=this.good;
    $('r-miss').textContent=this.miss; $('r-combo').textContent=this.maxCombo;
    $('r-score').textContent=this.score;
    $('lyric').textContent='♪ おしまい ♪';
    $('resultov').style.display='flex';
  }
};

/* keyboard */
window.addEventListener('keydown',e=>{
  if(e.repeat) return;
  const k=e.key.toLowerCase();
  if(k==='f'){Game.hit('don');e.preventDefault();}
  else if(k==='j'){Game.hit('ka');e.preventDefault();}
});
</script>
</body>
</html>
