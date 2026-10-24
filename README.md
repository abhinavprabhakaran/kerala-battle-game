<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Kerala Battlegrounds — 3D Prototype</title>
<style>
  *{box-sizing:border-box} html,body{margin:0;width:100%;height:100%;overflow:hidden;background:#101b19;font-family:system-ui,Arial;color:#fff}
  #game{display:block;width:100vw;height:100vh}
  #hud{position:fixed;inset:0;pointer-events:none;text-shadow:0 2px 4px #000}
  .panel{background:#101a1acc;border:1px solid #ffffff30;border-radius:10px;padding:10px 14px;backdrop-filter:blur(5px)}
  #title{position:absolute;left:16px;top:14px;font-weight:900;letter-spacing:1px;font-size:19px}
  #status{position:absolute;left:16px;top:62px;line-height:1.55;font-size:13px;min-width:190px}
  #help{position:absolute;left:16px;bottom:16px;font-size:12px;line-height:1.6;max-width:280px}
  #crosshair{position:absolute;left:50%;top:50%;transform:translate(-50%,-50%);font-size:24px;color:#fff}
  #message{position:absolute;left:50%;top:18%;transform:translateX(-50%);font-size:22px;font-weight:800;text-align:center}
  #center{position:absolute;left:50%;top:50%;transform:translate(-50%,-50%);text-align:center;pointer-events:auto;background:#101a1add;padding:26px;border:1px solid #ffffff55;border-radius:16px;max-width:90vw}
  #center h1{margin:0 0 8px;font-size:28px} #center p{margin:6px 0 16px;color:#d9e6df}
  button{background:#c7ff54;color:#17220e;font-weight:900;border:0;border-radius:9px;padding:12px 20px;cursor:pointer;font-size:15px}
  #zone{position:absolute;right:16px;top:16px;text-align:right;font-size:13px}
  #aimtip{position:absolute;right:16px;bottom:16px;font-size:12px}
  @media(max-width:650px){#title{font-size:15px}#status{top:54px;font-size:12px}#help{font-size:10px;max-width:230px}#center h1{font-size:22px}}
</style>
</head>
<body>
<canvas id="game"></canvas>
<div id="hud">
  <div id="title" class="panel">KERALA BATTLEGROUNDS <span style="color:#c7ff54">/ ALPHA</span></div>
  <div id="status" class="panel">❤️ Health: <b id="health">100</b><br>🔫 Ammo: <b id="ammo">30</b><br>☠️ Targets left: <b id="left">8</b><br>🏆 Eliminations: <b id="kills">0</b></div>
  <div id="zone" class="panel">SAFE ZONE<br><b id="zoneText">Closing in 60s</b></div>
  <div id="crosshair">+</div><div id="message"></div>
  <div id="help" class="panel">WASD — move · Shift — sprint<br>Mouse — look · Left click — shoot<br>R — reload · Esc — release mouse<br>Prototype only: bots are simple targets.</div>
  <div id="aimtip" class="panel">Find the targets. Stay inside the safe zone.</div>
  <div id="center">
    <h1>KERALA BATTLEGROUNDS</h1>
    <p>Drop into a tropical training zone. Eliminate all targets and stay alive.</p>
    <button id="start">ENTER BATTLEGROUND</button>
    <p style="font-size:11px;margin:14px 0 0">Desktop recommended · Three.js is loaded from a CDN</p>
  </div>
</div>
<script type="module">
import * as THREE from 'https://cdn.jsdelivr.net/npm/three@0.161.0/build/three.module.js';

const canvas=document.querySelector('#game');
const renderer=new THREE.WebGLRenderer({canvas,antialias:true});
renderer.setPixelRatio(Math.min(devicePixelRatio,2)); renderer.setSize(innerWidth,innerHeight);
renderer.shadowMap.enabled=true; renderer.shadowMap.type=THREE.PCFSoftShadowMap;
renderer.toneMapping=THREE.ACESFilmicToneMapping; renderer.toneMappingExposure=1.15;
const scene=new THREE.Scene(); scene.background=new THREE.Color(0x9ed8df); scene.fog=new THREE.Fog(0x9ed8df,65,190);
const camera=new THREE.PerspectiveCamera(70,innerWidth/innerHeight,.1,400);
scene.add(new THREE.HemisphereLight(0xdaf5ff,0x4d5833,2.0));
const sun=new THREE.DirectionalLight(0xfff1cf,3.0); sun.position.set(-35,60,25); sun.castShadow=true; sun.shadow.mapSize.set(2048,2048); scene.add(sun);

const mat=(color,roughness=1)=>new THREE.MeshStandardMaterial({color,roughness});
const grassMat=mat(0x5b9147), dirtMat=mat(0x927650), trunkMat=mat(0x60432b), leafMat=mat(0x24734b), roofMat=mat(0x9a5140), wallMat=mat(0xd8c9a2), blueMat=mat(0x2d6574), darkMat=mat(0x263b3d);
const ground=new THREE.Mesh(new THREE.PlaneGeometry(360,360),grassMat); ground.rotation.x=-Math.PI/2; ground.receiveShadow=true; scene.add(ground);
const dirt=new THREE.Mesh(new THREE.CircleGeometry(20,48),dirtMat); dirt.rotation.x=-Math.PI/2; dirt.position.y=.015; scene.add(dirt);
const road=new THREE.Mesh(new THREE.PlaneGeometry(9,250),mat(0x555b58)); road.rotation.x=-Math.PI/2; road.position.set(0,.025,-35); scene.add(road);
for(let z=-150;z<90;z+=12){const line=new THREE.Mesh(new THREE.PlaneGeometry(.16,5),mat(0xe6dfb5));line.rotation.x=-Math.PI/2;line.position.set(0,.04,z);scene.add(line);}

function addTree(x,z,scale=1){
 const g=new THREE.Group(); g.position.set(x,0,z); g.scale.setScalar(scale);
 const trunk=new THREE.Mesh(new THREE.CylinderGeometry(.28,.42,4,7),trunkMat);trunk.position.y=2;trunk.castShadow=true;g.add(trunk);
 for(let i=0;i<6;i++){const a=i*Math.PI/3;const leaf=new THREE.Mesh(new THREE.ConeGeometry(1.05,4,5),leafMat);leaf.position.set(Math.cos(a)*.8,4.3,Math.sin(a)*.8);leaf.rotation.z=-Math.cos(a)*.6;leaf.rotation.x=Math.sin(a)*.6;leaf.castShadow=true;g.add(leaf);}
 const crown=new THREE.Mesh(new THREE.SphereGeometry(1.25,8,6),leafMat);crown.position.y=5.2;g.add(crown);scene.add(g);
}
for(let i=0;i<70;i++){let x=(Math.random()-.5)*210,z=(Math.random()-.5)*220;if(Math.abs(x)<12&&z>-85&&z<20)continue;addTree(x,z,.7+Math.random()*.7);}
function house(x,z,rot=0){
 const g=new THREE.Group();g.position.set(x,0,z);g.rotation.y=rot;
 const body=new THREE.Mesh(new THREE.BoxGeometry(7,4.2,6),wallMat);body.position.y=2.1;body.castShadow=true;body.receiveShadow=true;g.add(body);
 const roof=new THREE.Mesh(new THREE.ConeGeometry(5.6,2.4,4),roofMat);roof.position.y=5.1;roof.rotation.y=Math.PI/4;roof.scale.z=.8;roof.castShadow=true;g.add(roof);
 const door=new THREE.Mesh(new THREE.BoxGeometry(1.2,2.4,.12),darkMat);door.position.set(0,1.2,3.06);g.add(door);
 const winMat=mat(0x7ab9c5,.3);for(const wx of [-2.2,2.2]){const w=new THREE.Mesh(new THREE.BoxGeometry(1.1,1,.12),winMat);w.position.set(wx,2.6,3.08);g.add(w);}
 scene.add(g);
}
house(-16,-25,.2);house(17,-40,-.35);house(-22,-55,.1);house(20,-12,.4);house(35,-68,.2);house(-38,-80,-.2);

const player=new THREE.Group();scene.add(player);
const torso=new THREE.Mesh(new THREE.CapsuleGeometry(.48,.9,4,8),blueMat);torso.position.y=1.35;torso.castShadow=true;player.add(torso);
const head=new THREE.Mesh(new THREE.SphereGeometry(.31,12,10),mat(0xb9815b));head.position.y=2.15;player.add(head);
const pack=new THREE.Mesh(new THREE.BoxGeometry(.58,.68,.28),mat(0x384d2b));pack.position.set(0,1.42,-.45);player.add(pack);
const gun=new THREE.Mesh(new THREE.BoxGeometry(.16,.16,1.05),darkMat);gun.position.set(.45,1.48,.55);player.add(gun);
player.position.set(0,0,8);

const enemyMat=mat(0x9c342b), enemyHeadMat=mat(0xd8a47c), enemyAccent=mat(0x393b35);
const enemies=[];
for(let i=0;i<8;i++){
 const g=new THREE.Group();const a=i/8*Math.PI*2,r=24+Math.random()*22;g.position.set(Math.cos(a)*r,0,-28+Math.sin(a)*r);
 const body=new THREE.Mesh(new THREE.CapsuleGeometry(.48,.9,4,8),enemyMat);body.position.y=1.35;body.castShadow=true;g.add(body);
 const h=new THREE.Mesh(new THREE.SphereGeometry(.3,10,8),enemyHeadMat);h.position.y=2.15;g.add(h);
 const helmet=new THREE.Mesh(new THREE.SphereGeometry(.33,10,6,0,Math.PI*2,0,Math.PI/2),enemyAccent);helmet.position.y=2.31;g.add(helmet);
 g.userData={hp:2,alive:true,phase:Math.random()*6};scene.add(g);enemies.push(g);
}
const zoneGeo=new THREE.RingGeometry(0.985,1,96);const zoneMat=new THREE.MeshBasicMaterial({color:0x7ee7ff,side:THREE.DoubleSide,transparent:true,opacity:.7});
const zone=new THREE.Mesh(zoneGeo,zoneMat);zone.rotation.x=-Math.PI/2;zone.position.y=.12;zone.scale.set(70,70,1);scene.add(zone);
const zoneLine=new THREE.Mesh(new THREE.TorusGeometry(70,.13,5,120),new THREE.MeshBasicMaterial({color:0x6ee7ff,transparent:true,opacity:.8}));zoneLine.rotation.x=Math.PI/2;zoneLine.position.y=.18;scene.add(zoneLine);

let started=false,locked=false,health=100,ammo=30,kills=0,elapsed=0,zoneRadius=70,zoneTimer=60,reloading=false,gameOver=false;
const keys={};let yaw=0,pitch=-.18;
const center=document.querySelector('#center'),message=document.querySelector('#message');
function updateHud(){document.querySelector('#health').textContent=Math.max(0,Math.round(health));document.querySelector('#ammo').textContent=ammo;document.querySelector('#left').textContent=enemies.filter(e=>e.userData.alive).length;document.querySelector('#kills').textContent=kills;}
document.querySelector('#start').onclick=()=>{started=true;center.style.display='none';canvas.requestPointerLock?.();};
document.addEventListener('pointerlockchange',()=>locked=document.pointerLockElement===canvas);
document.addEventListener('keydown',e=>{keys[e.code]=true;if(e.code==='KeyR')reload();});
document.addEventListener('keyup',e=>keys[e.code]=false);
document.addEventListener('mousemove',e=>{if(locked&&started){yaw-=e.movementX*.0022;pitch-=e.movementY*.0018;pitch=Math.max(-.8,Math.min(.45,pitch));}});
document.addEventListener('mousedown',e=>{if(e.button===0&&started&&locked)shoot();});
function reload(){if(reloading||ammo===30||gameOver)return;reloading=true;message.textContent='RELOADING…';setTimeout(()=>{ammo=30;reloading=false;message.textContent='';updateHud();},1000);}
const raycaster=new THREE.Raycaster();
function shoot(){
 if(ammo<=0){message.textContent='PRESS R TO RELOAD';setTimeout(()=>message.textContent='',900);return;}
 ammo--;updateHud();
 const dir=new THREE.Vector3();camera.getWorldDirection(dir);raycaster.set(camera.position,dir);raycaster.far=95;
 const targets=[];for(const e of enemies)if(e.userData.alive)targets.push(...e.children);
 const hit=raycaster.intersectObjects(targets,false)[0];
 if(hit){const e=enemies.find(en=>en.children.includes(hit.object));if(e){e.userData.hp--;if(e.userData.hp<=0){e.userData.alive=false;scene.remove(e);kills++;message.textContent='TARGET ELIMINATED';setTimeout(()=>message.textContent='',700);if(kills===8){message.textContent='VICTORY ROYALE!';finish(true);}}}}
}
function finish(win){gameOver=true;document.exitPointerLock?.();center.style.display='block';center.innerHTML=`<h1>${win?'VICTORY ROYALE!':'YOU WERE ELIMINATED'}</h1><p>Eliminations: ${kills} / 8</p><button id="again">PLAY AGAIN</button>`;document.querySelector('#again').onclick=()=>location.reload();}
function clamp(v,a,b){return Math.max(a,Math.min(b,v));}
const clock=new THREE.Clock();
function animate(){
 requestAnimationFrame(animate);const dt=Math.min(clock.getDelta(),.05);
 if(started&&!gameOver){
  elapsed+=dt;zoneTimer-=dt;
  if(zoneTimer<=0){zoneRadius=Math.max(14,zoneRadius-4*dt);zone.scale.set(zoneRadius,zoneRadius,1);zoneLine.scale.setScalar(zoneRadius/70);if(zoneRadius<=14)zoneTimer=0;else zoneTimer=0;}
  const forward=new THREE.Vector3(Math.sin(yaw),0,Math.cos(yaw));const right=new THREE.Vector3(forward.z,0,-forward.x);
  let move=new THREE.Vector3();if(keys.KeyW||keys.ArrowUp)move.add(forward);if(keys.KeyS||keys.ArrowDown)move.sub(forward);if(keys.KeyD||keys.ArrowRight)move.add(right);if(keys.KeyA||keys.ArrowLeft)move.sub(right);
  if(move.lengthSq()>0){move.normalize().multiplyScalar((keys.ShiftLeft?11:6)*dt);player.position.add(move);}
  player.position.x=clamp(player.position.x,-155,155);player.position.z=clamp(player.position.z,-155,155);player.rotation.y=yaw;
  const desired=new THREE.Vector3(player.position.x-Math.sin(yaw)*7,player.position.y+3.7+pitch*2,player.position.z-Math.cos(yaw)*7);
  camera.position.lerp(desired,1-Math.pow(.001,dt));camera.lookAt(player.position.x,player.position.y+1.5,player.position.z);
  enemies.forEach(e=>{if(!e.userData.alive)return;e.userData.phase+=dt;const d=e.position.distanceTo(player.position);if(d>12){const dir=player.position.clone().sub(e.position);dir.y=0;dir.normalize();e.position.addScaledVector(dir,dt*.85);e.rotation.y=Math.atan2(dir.x,dir.z);}else if(Math.random()<dt*.22){health-=5;updateHud();if(health<=0)finish(false);} });
  const dist=Math.hypot(player.position.x,player.position.z);if(dist>zoneRadius){health-=dt*5;updateHud();if(health<=0)finish(false);}
  document.querySelector('#zoneText').textContent=zoneRadius>14?`Radius ${Math.round(zoneRadius)}m`:'Final zone';
 }
 renderer.render(scene,camera);
}
animate();
addEventListener('resize',()=>{camera.aspect=innerWidth/innerHeight;camera.updateProjectionMatrix();renderer.setSize(innerWidth,innerHeight);});
</script>
</body>
</html>
