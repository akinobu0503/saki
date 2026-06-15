<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>わんこチー牛育成 ～31歳、甘えん坊～</title>
<style>
  * { margin:0; padding:0; box-sizing:border-box; }
  html,body { width:100%; height:100%; overflow:hidden;
    font-family:"Hiragino Maru Gothic ProN","Rounded Mplus 1c","Segoe UI",sans-serif;
    background:#23202b; user-select:none; -webkit-user-select:none; }
  #app { position:fixed; inset:0; }
  canvas { display:block; }

  #hud { position:fixed; top:14px; left:14px; right:14px; display:flex;
    justify-content:space-between; align-items:flex-start; pointer-events:none; z-index:5; }
  .panel { background:rgba(255,255,255,.85); border:3px solid #ffb3c9; border-radius:18px;
    padding:10px 14px; box-shadow:0 6px 18px rgba(120,90,110,.35); backdrop-filter:blur(4px); }
  #stats { min-width:210px; }
  .name { font-weight:800; color:#e85a7f; font-size:16px; margin-bottom:2px; }
  .lvl { font-size:12px; color:#c77; margin-bottom:8px; }
  .bar-row { display:flex; align-items:center; gap:6px; margin:5px 0; font-size:12px; color:#774; }
  .bar-label { width:56px; font-weight:700; }
  .bar { flex:1; height:12px; background:#f3dbe3; border-radius:8px; overflow:hidden; }
  .bar>i { display:block; height:100%; width:50%; border-radius:8px; transition:width .4s ease; }
  .b-aff>i { background:linear-gradient(90deg,#ff8fb0,#ff5f8d); }
  .b-full>i { background:linear-gradient(90deg,#ffd27a,#ffaa3b); }
  .b-mood>i { background:linear-gradient(90deg,#8fe0ff,#46b6ff); }
  #clock { text-align:right; font-size:12px; color:#fff; text-shadow:0 1px 2px rgba(0,0,0,.4); }
  #clock .big { font-size:15px; font-weight:800; }

  #bubble { position:fixed; left:50%; top:15%; transform:translateX(-50%) scale(0);
    background:#fff; border:3px solid #ffb3c9; color:#d24f74; font-weight:800; font-size:18px;
    padding:12px 20px; border-radius:22px; max-width:78%; text-align:center;
    box-shadow:0 8px 22px rgba(0,0,0,.3); z-index:8;
    transition:transform .25s cubic-bezier(.18,1.5,.5,1); pointer-events:none; }
  #bubble::after { content:""; position:absolute; bottom:-14px; left:50%; transform:translateX(-50%);
    border-width:14px 12px 0; border-style:solid; border-color:#ffb3c9 transparent transparent; }
  #bubble.show { transform:translateX(-50%) scale(1); }

  #actions { position:fixed; bottom:18px; left:50%; transform:translateX(-50%); display:flex;
    gap:10px; z-index:6; flex-wrap:wrap; justify-content:center; padding:0 8px; }
  .btn { pointer-events:auto; cursor:pointer; border:none; border-radius:16px; padding:12px 16px;
    font-size:15px; font-weight:800; color:#fff;
    box-shadow:0 6px 0 rgba(0,0,0,.18),0 8px 16px rgba(0,0,0,.25);
    transition:transform .08s,box-shadow .08s; display:flex; flex-direction:column;
    align-items:center; gap:2px; min-width:78px; }
  .btn .emoji { font-size:22px; }
  .btn:active { transform:translateY(4px); box-shadow:0 2px 0 rgba(0,0,0,.18); }
  #b-pet{background:linear-gradient(160deg,#ff96b7,#ff5f8d);}
  #b-feed{background:linear-gradient(160deg,#ffc46b,#ff9a2e);}
  #b-play{background:linear-gradient(160deg,#9be37a,#4fc04f);}
  #b-gift{background:linear-gradient(160deg,#b69bff,#7b5cff);}
  #b-talk{background:linear-gradient(160deg,#7bd6ff,#3aa6ff);}

  #topright { position:fixed; top:96px; right:14px; z-index:6; display:flex; flex-direction:column; gap:8px; }
  .mini { pointer-events:auto; cursor:pointer; border:none; border-radius:12px; padding:7px 11px;
    font-size:12px; font-weight:700; color:#7a4a5e; background:rgba(255,255,255,.85);
    border:2px solid #ffc2d4; box-shadow:0 3px 8px rgba(0,0,0,.2); }
  .mini.on { background:#ffd9e6; color:#d24f74; }

  #hint { position:fixed; bottom:100px; left:50%; transform:translateX(-50%); font-size:12px;
    color:#fff; background:rgba(0,0,0,.35); padding:5px 14px; border-radius:12px; z-index:5; }

  #title { position:fixed; inset:0; background:radial-gradient(circle at 50% 30%,#3a3550,#1c1a26);
    display:flex; flex-direction:column; align-items:center; justify-content:center; z-index:20;
    text-align:center; gap:18px; }
  #title h1 { color:#ffd0e0; font-size:clamp(26px,6vw,46px); text-shadow:0 3px 10px rgba(255,120,160,.5); line-height:1.3; }
  #title p { color:#cbb6c6; font-size:15px; max-width:340px; line-height:1.6; }
  #startBtn { pointer-events:auto; cursor:pointer; background:linear-gradient(160deg,#ff96b7,#ff5f8d);
    color:#fff; border:none; border-radius:20px; padding:16px 40px; font-size:20px; font-weight:800;
    box-shadow:0 8px 0 #d84f76; transition:transform .1s,box-shadow .1s; }
  #startBtn:active { transform:translateY(6px); box-shadow:0 2px 0 #d84f76; }
  #err { position:fixed; left:10px; bottom:10px; right:10px; color:#ffd; font-size:12px;
    background:rgba(160,0,40,.85); padding:8px 12px; border-radius:8px; z-index:30; display:none; }
</style>
</head>
<body>
<div id="app"></div>

<div id="hud">
  <div class="panel" id="stats">
    <div class="name">ぴよ太 <span style="font-size:12px">(31)</span></div>
    <div class="lvl" id="lvlText">なつき度 Lv.1 ・ よそよそしい</div>
    <div class="bar-row"><span class="bar-label">好感度</span><div class="bar b-aff"><i id="barAff"></i></div></div>
    <div class="bar-row"><span class="bar-label">まんぷく</span><div class="bar b-full"><i id="barFull"></i></div></div>
    <div class="bar-row"><span class="bar-label">きげん</span><div class="bar b-mood"><i id="barMood"></i></div></div>
  </div>
  <div class="panel" id="clock">
    <div class="big" id="dayText">1日目</div>
    <div id="timeText">ひるすぎ</div>
  </div>
</div>

<div id="topright">
  <button class="mini on" id="t-ears">🐶 犬みみ</button>
  <button class="mini on" id="t-tail">しっぽ</button>
</div>

<div id="bubble"></div>
<div id="hint">ドラッグで回転 / キャラをタップでナデナデ / ホイールでズーム</div>

<div id="actions">
  <button class="btn" id="b-pet"><span class="emoji">🫳</span>なでなで</button>
  <button class="btn" id="b-feed"><span class="emoji">🍙</span>ごはん</button>
  <button class="btn" id="b-play"><span class="emoji">🎾</span>あそぶ</button>
  <button class="btn" id="b-gift"><span class="emoji">🎁</span>プレゼント</button>
  <button class="btn" id="b-talk"><span class="emoji">💬</span>はなす</button>
</div>

<div id="title">
  <h1>わんこチー牛育成<br>～31歳、甘えん坊～</h1>
  <p>よそよそしい彼を、なでて・ごはんあげて・あそんで…<br>たっぷり甘やかして、しっぽブンブンの忠犬に育てよう🐶</p>
  <button id="startBtn">はじめる</button>
</div>
<div id="err"></div>

<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
<script>
const $ = id => document.getElementById(id);
window.addEventListener("error", e=>{ const x=$("err"); x.style.display="block"; x.textContent="エラー: "+e.message; });
if(typeof THREE==="undefined"){ const x=$("err"); x.style.display="block"; x.textContent="3Dライブラリを読み込めませんでした（インターネット接続をご確認ください）"; }

let scene,camera,renderer,clock;
let chr={}; const hearts=[]; let started=false;

/* ===== ステータス ===== */
const S={ aff:8, full:60, mood:55, day:1, level:1 };
const LEVELS=[
  {min:0,name:"よそよそしい"},{min:15,name:"ちょっと気にしてる"},
  {min:35,name:"なついてきた"},{min:55,name:"甘えん坊"},
  {min:75,name:"べったり忠犬"},{min:92,name:"らぶらぶワンコ💕"},
];
const LINES={
  pet_low:["……び、びっくりした","な、なに急に…でも、やだじゃない","ふぇ…"],
  pet_mid:["えへへ、もっとなでて？","きもちぃ…","そこ、すき…","くぅん…♪"],
  pet_high:["だいすきっ!! なでなでもっとぉ","ご主人さまの手、せかいいち…","しっぽ止まんないよぉ🐶","えへへっ、はなれたくない…"],
  feed_low:["…いただきます","おいしい、です","ありがと…ございます"],
  feed_high:["あーん♪ ご主人のごはん最高ぉ","おかわりしてもいい…?","むぐむぐ…しあわせ…"],
  play_low:["え、ぼくと…?","うまくできるかな…","た、たのしい…かも"],
  play_high:["ボール投げて投げてっ!!","わーい!!もっかい!!","はぁはぁ…たのしいねっ🎾"],
  gift_low:["ぼくに…? ほんとに?","うれしい…大事にします","わ、わ、ありがとう…"],
  gift_high:["ぼくのこと考えてくれたの…?うれし涙","一生たからものにするっ","ご主人のことだいすきぃ💕"],
  talk_low:["きょうは来てくれたんだ…","あの…ぼくのこと、嫌いじゃない?","…えっと、その、こんにちは"],
  talk_mid:["きょうもいっしょにいよ?","ご主人がいると、あんしんする…","なでてくれたの、まだおぼえてる"],
  talk_high:["ずっとそばにいてくれる…?","ご主人がぼくの全部だよっ","つぎはいつ来てくれるの…?待ってるね","だいすき、だいすきっ🐶💕"],
  hungry:["おなか、すいたかも…","くぅ〜ん…(おなかの音)","ご、ごはん…ねだってもいい?"],
  lonely:["…さみしかった","もう来ないかと思った…","おそばに、いさせて?"],
  idleHigh:["ねぇ、こっち見て…?","ご主人ご主人っ","かまって〜🐶","そばにいたいな…"],
};
const pick=a=>a[Math.floor(Math.random()*a.length)];

/* ============================================================ */
function initScene(){
  scene=new THREE.Scene();
  scene.background=new THREE.Color(0x2b2738);
  scene.fog=new THREE.Fog(0x2b2738,16,34);

  camera=new THREE.PerspectiveCamera(42, innerWidth/innerHeight, 0.1, 100);

  renderer=new THREE.WebGLRenderer({antialias:true});
  renderer.setPixelRatio(Math.min(devicePixelRatio,2));
  renderer.setSize(innerWidth,innerHeight);
  renderer.shadowMap.enabled=true; renderer.shadowMap.type=THREE.PCFSoftShadowMap;
  renderer.outputEncoding=THREE.sRGBEncoding;
  $("app").appendChild(renderer.domElement);

  // lighting (soft studio)
  scene.add(new THREE.HemisphereLight(0xfff0f6,0x33304a,0.85));
  const key=new THREE.DirectionalLight(0xffffff,1.05);
  key.position.set(4,9,7); key.castShadow=true;
  key.shadow.mapSize.set(2048,2048);
  Object.assign(key.shadow.camera,{left:-8,right:8,top:9,bottom:-4});
  key.shadow.bias=-0.0004; scene.add(key);
  const fill=new THREE.DirectionalLight(0xbfe0ff,0.4); fill.position.set(-6,4,3); scene.add(fill);
  const rim=new THREE.SpotLight(0xff8fc0,1.1,30,0.6,0.6); rim.position.set(-4,7,-6); scene.add(rim);

  buildRoom();
  buildCharacter();
  setupOrbit();

  clock=new THREE.Clock();
  animate();
}

function buildRoom(){
  const floor=new THREE.Mesh(new THREE.CircleGeometry(40,64),
    new THREE.MeshStandardMaterial({color:0xe9d9e6,roughness:.95}));
  floor.rotation.x=-Math.PI/2; floor.receiveShadow=true; scene.add(floor);
  const rug=new THREE.Mesh(new THREE.CircleGeometry(3.2,64),
    new THREE.MeshStandardMaterial({color:0xffd1e0,roughness:1}));
  rug.rotation.x=-Math.PI/2; rug.position.y=0.01; rug.receiveShadow=true; scene.add(rug);
  const ring=new THREE.Mesh(new THREE.RingGeometry(2.7,3.0,64),
    new THREE.MeshStandardMaterial({color:0xff9dbf}));
  ring.rotation.x=-Math.PI/2; ring.position.y=0.012; scene.add(ring);
  const wall=new THREE.Mesh(new THREE.CylinderGeometry(20,20,18,64,1,true),
    new THREE.MeshStandardMaterial({color:0x3a3450,side:THREE.BackSide,roughness:1}));
  wall.position.y=9; scene.add(wall);
  for(let i=0;i<10;i++){
    const h=makeHeart(0xff7fae); h.scale.setScalar(.5);
    const a=Math.random()*Math.PI*2, r=11+Math.random()*4;
    h.position.set(Math.cos(a)*r, 3+Math.random()*8, Math.sin(a)*r);
    h.lookAt(0,h.position.y,0); scene.add(h);
  }
}
function makeHeart(color){
  const s=new THREE.Shape();
  s.moveTo(0,.5); s.bezierCurveTo(0,.5,-.5,1,-.5,.5);
  s.bezierCurveTo(-.5,0,0,-.2,0,-.6); s.bezierCurveTo(0,-.2,.5,0,.5,.5);
  s.bezierCurveTo(.5,1,0,.5,0,.5);
  const g=new THREE.ExtrudeGeometry(s,{depth:.18,bevelEnabled:true,bevelSize:.06,bevelThickness:.06,bevelSegments:2});
  g.center();
  return new THREE.Mesh(g,new THREE.MeshStandardMaterial({color,roughness:.5}));
}

/* ===== 半リアル人型キャラ ===== */
function buildCharacter(){
  const root=new THREE.Group(); scene.add(root); chr.root=root;
  const body=new THREE.Group(); root.add(body); chr.body=body;

  const skin=new THREE.MeshStandardMaterial({color:0xf0c6a0,roughness:.65,metalness:0});
  const skin2=new THREE.MeshStandardMaterial({color:0xe9b894,roughness:.7});
  const hair=new THREE.MeshStandardMaterial({color:0x2a2018,roughness:.85});
  const shirt=new THREE.MeshStandardMaterial({color:0x6f97c4,roughness:.8});
  const shirt2=new THREE.MeshStandardMaterial({color:0x5b82ad,roughness:.8});
  const tee=new THREE.MeshStandardMaterial({color:0xf3f1ee,roughness:.85});
  const jeans=new THREE.MeshStandardMaterial({color:0x42485e,roughness:.9});
  const shoeMat=new THREE.MeshStandardMaterial({color:0xf4f4f4,roughness:.6});
  const sole=new THREE.MeshStandardMaterial({color:0xcfcfcf,roughness:.6});
  const dark=new THREE.MeshStandardMaterial({color:0x20232a,roughness:.5,metalness:.3});

  const smooth=(g)=>g; // helper noop

  // ---- shoes & legs ----
  function leg(x){
    const grp=new THREE.Group(); grp.position.set(x,0,0); body.add(grp);
    const thigh=new THREE.Mesh(new THREE.CylinderGeometry(.19,.17,.62,16),jeans);
    thigh.position.y=.95; thigh.castShadow=true; grp.add(thigh);
    const shin=new THREE.Mesh(new THREE.CylinderGeometry(.16,.13,.6,16),jeans);
    shin.position.y=.42; shin.castShadow=true; grp.add(shin);
    const shoe=new THREE.Mesh(new THREE.SphereGeometry(.2,16,12),shoeMat);
    shoe.scale.set(1,.62,1.5); shoe.position.set(0,.11,.08); shoe.castShadow=true; grp.add(shoe);
    const sl=new THREE.Mesh(new THREE.BoxGeometry(.34,.08,.5),sole);
    sl.position.set(0,.04,.09); grp.add(sl);
    return grp;
  }
  leg(-.21); leg(.21);

  // ---- hips ----
  const hips=new THREE.Mesh(new THREE.SphereGeometry(.36,20,16),jeans);
  hips.scale.set(1.05,.7,.9); hips.position.y=1.18; hips.castShadow=true; body.add(hips);

  // ---- torso (rounded, slightly chubby) ----
  const torso=new THREE.Mesh(new THREE.SphereGeometry(.5,24,20),shirt);
  torso.scale.set(1.02,1.3,.82); torso.position.y=1.62; torso.castShadow=true; body.add(torso);
  // shirt placket
  const plk=new THREE.Mesh(new THREE.BoxGeometry(.06,.7,.04),shirt2);
  plk.position.set(0,1.6,.43); body.add(plk);
  for(let i=0;i<4;i++){ const b=new THREE.Mesh(new THREE.SphereGeometry(.022,8,8),dark);
    b.position.set(0,1.85-i*.2,.45); body.add(b); }
  // collar
  const col=new THREE.Mesh(new THREE.TorusGeometry(.17,.05,10,24,Math.PI),shirt2);
  col.rotation.x=Math.PI/2.1; col.position.set(0,1.97,.12); body.add(col);
  // shoulders
  [[-.42,1.9],[.42,1.9]].forEach(([x,y])=>{const s=new THREE.Mesh(new THREE.SphereGeometry(.18,16,14),shirt);
    s.position.set(x,y,0); s.castShadow=true; body.add(s);});

  // ---- arms (short-sleeve: forearm is skin) ----
  function arm(side){
    const grp=new THREE.Group(); grp.position.set(side*.42,1.9,0); body.add(grp);
    const up=new THREE.Mesh(new THREE.CylinderGeometry(.14,.12,.42,14),shirt);
    up.position.y=-.21; up.castShadow=true; grp.add(up);
    const fore=new THREE.Group(); fore.position.y=-.42; grp.add(fore);
    const fa=new THREE.Mesh(new THREE.CylinderGeometry(.115,.1,.42,14),skin);
    fa.position.y=-.21; fa.castShadow=true; fore.add(fa);
    const hand=new THREE.Mesh(new THREE.SphereGeometry(.12,14,12),skin);
    hand.scale.set(1,1.2,.7); hand.position.y=-.46; fore.add(hand);
    grp.rotation.z=side*.18; grp.userData.fore=fore;
    return grp;
  }
  chr.armL=arm(-1); chr.armR=arm(1);

  // ---- neck ----
  const neck=new THREE.Mesh(new THREE.CylinderGeometry(.13,.15,.2,14),skin);
  neck.position.y=2.05; body.add(neck);

  // ---- HEAD ----
  const head=new THREE.Group(); head.position.y=2.42; body.add(head); chr.head=head; chr.headBaseY=2.42;
  const skull=new THREE.Mesh(new THREE.SphereGeometry(.42,32,28),skin);
  skull.scale.set(.96,1.05,1.0); skull.castShadow=true; head.add(skull);
  // jaw/chin
  const jaw=new THREE.Mesh(new THREE.SphereGeometry(.3,24,18),skin);
  jaw.scale.set(.9,.78,.95); jaw.position.set(0,-.24,.04); head.add(jaw);
  // cheeks (slightly puffy)
  [[-.22,-.08],[.22,-.08]].forEach(([x,y])=>{const c=new THREE.Mesh(new THREE.SphereGeometry(.13,16,14),skin);
    c.position.set(x,y,.3); head.add(c);});
  // ears
  [-1,1].forEach(s=>{const e=new THREE.Mesh(new THREE.SphereGeometry(.1,14,12),skin2);
    e.scale.set(.5,1,.7); e.position.set(s*.42,-.02,.02); head.add(e);});
  // nose
  const nose=new THREE.Mesh(new THREE.SphereGeometry(.07,14,12),skin);
  nose.scale.set(.8,.9,1.1); nose.position.set(0,-.06,.42); head.add(nose);

  // eyes (sclera + iris + pupil + lids)
  function eye(x){
    const grp=new THREE.Group(); grp.position.set(x,.04,.3); head.add(grp);
    const white=new THREE.Mesh(new THREE.SphereGeometry(.085,18,16),
      new THREE.MeshStandardMaterial({color:0xfbfbff,roughness:.3}));
    white.scale.set(1,1,.6); white.position.z=.05; grp.add(white);
    const iris=new THREE.Mesh(new THREE.CircleGeometry(.045,20),
      new THREE.MeshStandardMaterial({color:0x5a3b2a,roughness:.4}));
    iris.position.set(0,0,.105); grp.add(iris);
    const pup=new THREE.Mesh(new THREE.CircleGeometry(.022,16),
      new THREE.MeshBasicMaterial({color:0x140f0c}));
    pup.position.set(0,0,.112); grp.add(pup);
    const shine=new THREE.Mesh(new THREE.CircleGeometry(.012,10),
      new THREE.MeshBasicMaterial({color:0xffffff}));
    shine.position.set(.015,.018,.12); grp.add(shine);
    // upper lid for blink
    const lid=new THREE.Mesh(new THREE.SphereGeometry(.09,18,12,0,Math.PI*2,0,Math.PI/2),skin);
    lid.scale.set(1,1,.6); lid.position.z=.05; lid.rotation.x=-.15; grp.add(lid);
    grp.userData.lid=lid;
    return grp;
  }
  chr.eyeL=eye(-.17); chr.eyeR=eye(.17);
  // eyebrows
  [-1,1].forEach(s=>{const b=new THREE.Mesh(new THREE.BoxGeometry(.14,.025,.04),hair);
    b.position.set(s*.17,.16,.38); b.rotation.z=s*.05; head.add(b);});
  // glasses
  const gf=new THREE.MeshStandardMaterial({color:0x1c1f25,metalness:.4,roughness:.35});
  const lensMat=new THREE.MeshStandardMaterial({color:0xcdeeff,transparent:true,opacity:.25,roughness:.1});
  [-.17,.17].forEach(x=>{
    const r=new THREE.Mesh(new THREE.TorusGeometry(.115,.018,12,28),gf);
    r.position.set(x,.04,.4); head.add(r);
    const l=new THREE.Mesh(new THREE.CircleGeometry(.105,22),lensMat);
    l.position.set(x,.04,.398); head.add(l);
  });
  const br=new THREE.Mesh(new THREE.BoxGeometry(.1,.02,.02),gf); br.position.set(0,.04,.41); head.add(br);
  [-1,1].forEach(s=>{const a=new THREE.Mesh(new THREE.BoxGeometry(.02,.02,.34),gf);
    a.position.set(s*.28,.05,.28); head.add(a);});
  // mouth (small, shy)
  const mouth=new THREE.Mesh(new THREE.TorusGeometry(.06,.018,8,20,Math.PI),
    new THREE.MeshStandardMaterial({color:0xb96b6e,roughness:.6}));
  mouth.rotation.z=Math.PI; mouth.position.set(0,-.26,.4); head.add(mouth); chr.mouth=mouth;

  // ---- HAIR (mushroom / center-part, layered) ----
  const hairG=new THREE.Group(); head.add(hairG); chr.hairG=hairG;
  const cap=new THREE.Mesh(new THREE.SphereGeometry(.45,28,24,0,Math.PI*2,0,Math.PI*0.58),hair);
  cap.scale.set(1.02,1.05,1.04); cap.position.y=.05; hairG.add(cap);
  // fringe (bangs) — ring of small blocks around forehead
  for(let i=0;i<11;i++){
    const a=-Math.PI*0.5 + (i/10)*Math.PI; // front arc
    const tuft=new THREE.Mesh(new THREE.ConeGeometry(.06,.2,8),hair);
    tuft.position.set(Math.sin(a)*.4, .16, Math.cos(a)*.4*0.9+0.02);
    tuft.rotation.x=Math.PI; tuft.rotation.z=Math.sin(a)*.3; hairG.add(tuft);
  }
  // sideburns + back length
  [-1,1].forEach(s=>{const sb=new THREE.Mesh(new THREE.BoxGeometry(.08,.22,.12),hair);
    sb.position.set(s*.4,-.04,.06); hairG.add(sb);});
  const back=new THREE.Mesh(new THREE.SphereGeometry(.44,24,20,0,Math.PI*2,Math.PI*0.4,Math.PI*0.35),hair);
  back.position.set(0,-.02,-.06); back.scale.set(1.02,1,1.04); hairG.add(back);

  // ---- dog ears (toggle) ----
  function dogEar(side){
    const e=new THREE.Group();
    const f=new THREE.Mesh(new THREE.SphereGeometry(.13,14,12),
      new THREE.MeshStandardMaterial({color:0x6a4a36,roughness:.9}));
    f.scale.set(.6,1.5,.4); f.position.y=-.18; e.add(f);
    const inner=new THREE.Mesh(new THREE.SphereGeometry(.07,12,10),
      new THREE.MeshStandardMaterial({color:0xe8b9b0}));
    inner.scale.set(.5,1.3,.3); inner.position.set(0,-.16,.05); e.add(inner);
    e.position.set(side*.3,.34,.04); e.rotation.z=side*.4; head.add(e); return e;
  }
  chr.earL=dogEar(-1); chr.earR=dogEar(1);

  // ---- tail (toggle) ----
  const tail=new THREE.Group(); tail.position.set(0,1.12,-.42); body.add(tail); chr.tail=tail;
  const seg=new THREE.Mesh(new THREE.ConeGeometry(.1,.6,12),
    new THREE.MeshStandardMaterial({color:0x6a4a36,roughness:.9}));
  seg.position.y=.26; seg.rotation.x=-.7; tail.add(seg);
  const tip=new THREE.Mesh(new THREE.SphereGeometry(.1,12,10),
    new THREE.MeshStandardMaterial({color:0xeaddc9})); tip.position.set(0,.52,.32); tail.add(tip);

  // ---- play ball ----
  const ball=new THREE.Mesh(new THREE.SphereGeometry(.2,18,16),
    new THREE.MeshStandardMaterial({color:0xff5f5f,roughness:.6}));
  ball.add(new THREE.Mesh(new THREE.TorusGeometry(.2,.025,8,24),
    new THREE.MeshStandardMaterial({color:0xffffff})));
  ball.castShadow=true; ball.visible=false; scene.add(ball); chr.ball=ball;

  chr.baseY=0; chr.action=null; chr.actionT=0; chr.wag=0;
}

/* ===== ドラッグ回転（自前オービット） ===== */
const orbit={ radius:5.4, theta:0, phi:1.32, target:new THREE.Vector3(0,1.5,0) };
function applyOrbit(){
  const r=orbit.radius, p=orbit.phi, t=orbit.theta;
  camera.position.set(
    orbit.target.x + r*Math.sin(p)*Math.sin(t),
    orbit.target.y + r*Math.cos(p),
    orbit.target.z + r*Math.sin(p)*Math.cos(t));
  camera.lookAt(orbit.target);
}
function setupOrbit(){
  applyOrbit();
  const el=renderer.domElement;
  let down=false,moved=false,px=0,py=0;
  const ray=new THREE.Raycaster(), m=new THREE.Vector2();
  el.addEventListener("pointerdown",e=>{down=true;moved=false;px=e.clientX;py=e.clientY;});
  el.addEventListener("pointermove",e=>{
    if(!down)return; const dx=e.clientX-px,dy=e.clientY-py;
    if(Math.abs(dx)+Math.abs(dy)>5)moved=true;
    orbit.theta-=dx*0.006; orbit.phi=Math.max(0.55,Math.min(1.5,orbit.phi-dy*0.005));
    px=e.clientX;py=e.clientY; applyOrbit();
  });
  el.addEventListener("pointerup",e=>{
    down=false;
    if(!moved && started && chr.root){
      m.x=(e.clientX/innerWidth)*2-1; m.y=-(e.clientY/innerHeight)*2+1;
      ray.setFromCamera(m,camera);
      if(ray.intersectObject(chr.root,true).length) doAction("pet");
    }
  });
  el.addEventListener("wheel",e=>{e.preventDefault();
    orbit.radius=Math.max(3,Math.min(9,orbit.radius+e.deltaY*0.003)); applyOrbit();
  },{passive:false});
}

/* ===== アニメーション ===== */
let blinkT=0, idleTalkT=6, talkCool=0, decayT=0, timeT=0, timeIdx=1;
const TIMES=["あさ","ひるすぎ","ゆうがた","よる"];
function animate(){
  requestAnimationFrame(animate);
  if(!clock)return;
  const dt=Math.min(clock.getDelta(),0.05), t=clock.elapsedTime;

  if(chr.root){
    chr.body.position.y=chr.baseY+Math.sin(t*1.8)*0.025;
    chr.head.rotation.z=Math.sin(t*1.1)*0.025;

    const wagSpeed=6+S.aff*0.12, wagAmt=0.2+chr.wag*0.6;
    chr.tail.rotation.z=Math.sin(t*wagSpeed)*wagAmt;
    chr.tail.rotation.y=Math.sin(t*wagSpeed*0.5)*0.15;
    chr.earL.rotation.x=Math.sin(t*2.2)*0.12;
    chr.earR.rotation.x=Math.sin(t*2.2+0.5)*0.12;

    blinkT-=dt; if(blinkT<=0){blinkT=2.5+Math.random()*2.5; doBlink();}

    // approach when affection high (z toward player +)
    const wantZ=(S.aff/100)*1.4;
    chr.root.position.z+=(wantZ-chr.root.position.z)*Math.min(1,dt*1.5);
    chr.root.rotation.y=Math.sin(t*0.5)*0.05;

    if(!chr.action){
      chr.armL.rotation.x=Math.sin(t*1.6)*0.07;
      chr.armR.rotation.x=Math.sin(t*1.6+1)*0.07;
    }
    updateAction(dt,t);

    if(started){
      decayT+=dt;
      if(decayT>=1){decayT=0; S.full=clamp(S.full-0.8); S.mood=clamp(S.mood-0.5);
        if(S.mood<20)S.aff=clamp(S.aff-0.3); refreshUI();}
      timeT+=dt;
      if(timeT>=18){timeT=0; timeIdx=(timeIdx+1)%4; if(timeIdx===0)S.day++;
        $("timeText").textContent=TIMES[timeIdx]; refreshUI();}
      idleTalkT-=dt; talkCool-=dt;
      if(idleTalkT<=0 && !chr.action && talkCool<=0){
        idleTalkT=7+Math.random()*6;
        if(S.full<25)say(pick(LINES.hungry));
        else if(S.mood<25)say(pick(LINES.lonely));
        else if(S.aff>40 && Math.random()<0.7)say(pick(LINES.idleHigh));
      }
    }
  }

  for(let i=hearts.length-1;i>=0;i--){const h=hearts[i];h.userData.life-=dt;
    h.position.y+=dt*1.4; h.position.x+=h.userData.vx*dt; h.rotation.z+=dt*2;
    h.scale.multiplyScalar(1+dt*0.4); h.material.opacity=Math.max(0,h.userData.life/h.userData.max);
    if(h.userData.life<=0){scene.remove(h);hearts.splice(i,1);}}

  renderer.render(scene,camera);
}
function doBlink(){[chr.eyeL,chr.eyeR].forEach(e=>{if(e)e.userData.lid.scale.set(1,2.4,.6);});
  setTimeout(()=>[chr.eyeL,chr.eyeR].forEach(e=>{if(e)e.userData.lid.scale.set(1,1,.6);}),110);}

function updateAction(dt,t){
  if(!chr.action)return; chr.actionT+=dt; const a=chr.action,p=chr.actionT;
  if(a==="pet"){
    chr.head.position.y=chr.headBaseY+Math.sin(p*10)*0.04;
    chr.head.rotation.x=-0.22+Math.sin(p*8)*0.05;
    if(p>1.6){chr.head.position.y=chr.headBaseY;chr.head.rotation.x=0;endAction();}
  }else if(a==="feed"){
    chr.armR.rotation.x=-1.7+Math.sin(p*12)*0.2; chr.head.rotation.x=0.1*Math.sin(p*12);
    if(p>1.8){chr.armR.rotation.x=0;chr.head.rotation.x=0;endAction();}
  }else if(a==="play"){
    const ball=chr.ball; ball.visible=true; const dur=1.6,k=Math.min(p/dur,1);
    ball.position.x=chr.root.position.x+Math.sin(t*3)*0.2;
    ball.position.z=chr.root.position.z+0.5+k*2.4;
    ball.position.y=0.4+Math.sin(k*Math.PI)*1.7;
    chr.baseY=Math.abs(Math.sin(p*7))*0.3*(p<dur?1:0);
    chr.armL.rotation.x=-1.2; chr.armR.rotation.x=-1.2;
    if(p>dur+0.3){ball.visible=false;chr.baseY=0;chr.armL.rotation.x=0;chr.armR.rotation.x=0;endAction();}
  }else if(a==="gift"){
    chr.armL.rotation.x=-1.5; chr.armR.rotation.x=-1.5; chr.body.rotation.y=Math.sin(p*10)*0.12;
    if(p>1.7){chr.armL.rotation.x=0;chr.armR.rotation.x=0;chr.body.rotation.y=0;endAction();}
  }
}
function endAction(){chr.action=null;chr.actionT=0;}

/* ===== ハート・吹き出し ===== */
function spawnHearts(n,color=0xff5f8d){
  for(let i=0;i<n;i++){const h=makeHeart(color); h.scale.setScalar(0.16);
    const hp=chr.head.getWorldPosition(new THREE.Vector3());
    h.position.copy(hp); h.position.x+=(Math.random()-.5)*0.6; h.position.y+=0.3;
    h.material.transparent=true; h.userData={life:1.4,max:1.4,vx:(Math.random()-.5)*0.8};
    scene.add(h); hearts.push(h);}
}
let bubbleTimer;
function say(text){const b=$("bubble"); b.textContent=text; b.classList.add("show");
  talkCool=2.2; clearTimeout(bubbleTimer); bubbleTimer=setTimeout(()=>b.classList.remove("show"),2600);}

/* ===== ロジック ===== */
function clamp(v){return Math.max(0,Math.min(100,v));}
function updateLevel(){let lv=1,name=LEVELS[0].name;
  for(let i=0;i<LEVELS.length;i++){if(S.aff>=LEVELS[i].min){lv=i+1;name=LEVELS[i].name;}}
  S.level=lv; chr.wag=(lv-1)/(LEVELS.length-1);
  $("lvlText").textContent=`なつき度 Lv.${lv} ・ ${name}`;}
function refreshUI(){$("barAff").style.width=S.aff+"%";$("barFull").style.width=S.full+"%";
  $("barMood").style.width=S.mood+"%";$("dayText").textContent=S.day+"日目";updateLevel();}
function highAff(){return S.aff>=45;}

const ACT={
  pet(){chr.action="pet";chr.actionT=0;S.aff=clamp(S.aff+5);S.mood=clamp(S.mood+8);
    spawnHearts(highAff()?5:2);say(pick(S.aff>70?LINES.pet_high:S.aff>25?LINES.pet_mid:LINES.pet_low));},
  feed(){chr.action="feed";chr.actionT=0;S.full=clamp(S.full+30);S.aff=clamp(S.aff+3);S.mood=clamp(S.mood+5);
    spawnHearts(highAff()?3:1,0xffaa3b);say(pick(highAff()?LINES.feed_high:LINES.feed_low));},
  play(){chr.action="play";chr.actionT=0;S.aff=clamp(S.aff+6);S.mood=clamp(S.mood+14);S.full=clamp(S.full-8);
    spawnHearts(highAff()?4:2,0x4fc04f);say(pick(highAff()?LINES.play_high:LINES.play_low));},
  gift(){chr.action="gift";chr.actionT=0;S.aff=clamp(S.aff+12);S.mood=clamp(S.mood+10);
    spawnHearts(8,0x7b5cff);say(pick(highAff()?LINES.gift_high:LINES.gift_low));},
  talk(){S.mood=clamp(S.mood+6);S.aff=clamp(S.aff+1);spawnHearts(1);
    say(pick(S.aff>70?LINES.talk_high:S.aff>30?LINES.talk_mid:LINES.talk_low));},
};
function doAction(name){if(chr.action&&name!=="talk")return;ACT[name]();refreshUI();}

/* ===== UI ===== */
function bindUI(){
  $("b-pet").onclick=()=>doAction("pet");
  $("b-feed").onclick=()=>doAction("feed");
  $("b-play").onclick=()=>doAction("play");
  $("b-gift").onclick=()=>doAction("gift");
  $("b-talk").onclick=()=>doAction("talk");
  $("t-ears").onclick=function(){const v=!chr.earL.visible;chr.earL.visible=v;chr.earR.visible=v;
    this.classList.toggle("on",v);};
  $("t-tail").onclick=function(){const v=!chr.tail.visible;chr.tail.visible=v;
    this.classList.toggle("on",v);};
  addEventListener("resize",()=>{camera.aspect=innerWidth/innerHeight;camera.updateProjectionMatrix();
    renderer.setSize(innerWidth,innerHeight);});
  $("startBtn").onclick=()=>{$("title").style.display="none";started=true;refreshUI();
    setTimeout(()=>say("あ…来てくれたんだ。えっと…よろしく、です"),600);
    setTimeout(()=>$("hint").style.display="none",8000);};
}

/* boot — ボタンを最優先で有効化（3D初期化が失敗してもスタートは押せる） */
function showErr(msg){ const x=$("err"); x.style.display="block"; x.textContent=msg; }

bindUI();        // 先にボタンのハンドラを登録
refreshUI();

try{
  initScene();   // 3D初期化（失敗してもボタンは生きている）
}catch(e){
  showErr("3D初期化エラー: "+e.message+" — はじめるは押せます");
}
</script>
</body>
</html>
