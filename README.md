#from pathlib import Path

html = """<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<title>Mini Battle Royale Mobile - Vida</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
<style>
body { margin:0; overflow:hidden; background:black; touch-action:none; }
#ui {
  position:absolute; top:10px; left:10px;
  color:white; font-family:Arial;
  background:rgba(0,0,0,0.6);
  padding:10px; border-radius:8px;
  z-index:2;
}
#health {
  margin-top:5px;
}
#shoot {
  position:absolute; bottom:30px; right:30px;
  width:70px; height:70px;
  background:red; border-radius:50%;
  opacity:0.7;
}
</style>
</head>
<body>

<div id="ui">
  ❤️ Vida: <span id="health">100</span>
</div>
<div id="shoot"></div>

<script src="https://cdn.jsdelivr.net/npm/three@0.152.2/build/three.min.js"></script>

<script>
const scene = new THREE.Scene();
scene.background = new THREE.Color(0x87ceeb);

const camera = new THREE.PerspectiveCamera(75, innerWidth/innerHeight, 0.1, 1000);
const renderer = new THREE.WebGLRenderer({antialias:true});
renderer.setSize(innerWidth, innerHeight);
document.body.appendChild(renderer.domElement);

// Luces
scene.add(new THREE.AmbientLight(0xffffff,0.6));
const sun = new THREE.DirectionalLight(0xffffff,1);
sun.position.set(20,30,10);
scene.add(sun);

// Terreno con relieve
const groundGeo = new THREE.PlaneGeometry(300,300,50,50);
const pos = groundGeo.attributes.position;
for(let i=0;i<pos.count;i++){
  pos.setZ(i, Math.random()*2);
}
groundGeo.computeVertexNormals();

const ground = new THREE.Mesh(
  groundGeo,
  new THREE.MeshStandardMaterial({color:0x2ecc71})
);
ground.rotation.x = -Math.PI/2;
scene.add(ground);

// Jugador
const player = new THREE.Group();
const body = new THREE.Mesh(
  new THREE.CapsuleGeometry(0.5,1.2,4,8),
  new THREE.MeshStandardMaterial({color:0x3498db})
);
player.add(body);
player.position.set(0,1.5,0);
scene.add(player);

let playerHealth = 100;
const healthUI = document.getElementById("health");

// Enemigos
const enemies = [];
for(let i=0;i<5;i++){
  const enemy = new THREE.Mesh(
    new THREE.CapsuleGeometry(0.5,1.2,4,8),
    new THREE.MeshStandardMaterial({color:0xe67e22})
  );
  enemy.position.set(
    (Math.random()-0.5)*120,
    1.5,
    (Math.random()-0.5)*120
  );
  enemy.health = 50;
  enemies.push(enemy);
  scene.add(enemy);
}

// Balas
const bullets = [];
function shoot(from, dir, color, owner){
  const b = new THREE.Mesh(
    new THREE.SphereGeometry(0.15),
    new THREE.MeshStandardMaterial({color})
  );
  b.position.copy(from);
  b.velocity = dir.clone().multiplyScalar(0.6);
  b.owner = owner;
  bullets.push(b);
  scene.add(b);
}

// Botón disparar
document.getElementById("shoot").addEventListener("touchstart", ()=>{
  const dir = new THREE.Vector3(0,0,-1).applyQuaternion(camera.quaternion);
  shoot(player.position, dir, 0xff0000, "player");
});

// Movimiento táctil
let touchX = 0;
document.addEventListener("touchstart", e => touchX = e.touches[0].clientX);
document.addEventListener("touchmove", e => {
  const dx = e.touches[0].clientX - touchX;
  player.position.x += dx * 0.002;
  touchX = e.touches[0].clientX;
});

// Cámara
camera.position.set(0,4,7);

function animate(){
  requestAnimationFrame(animate);

  // IA enemigos
  enemies.forEach((en, index)=>{
    en.lookAt(player.position);
    en.position.lerp(player.position, 0.0004);

    if(Math.random()<0.01){
      const dir = new THREE.Vector3();
      dir.subVectors(player.position,en.position).normalize();
      shoot(en.position, dir, 0xffff00, "enemy");
    }
  });

  // Balas y colisiones
  bullets.forEach((b, i)=>{
    b.position.add(b.velocity);

    // Golpe a jugador
    if(b.owner === "enemy" && b.position.distanceTo(player.position) < 1){
      playerHealth -= 5;
      healthUI.textContent = playerHealth;
      scene.remove(b);
      bullets.splice(i,1);
    }

    // Golpe a enemigos
    enemies.forEach((en, ei)=>{
      if(b.owner === "player" && b.position.distanceTo(en.position) < 1){
        en.health -= 10;
        scene.remove(b);
        bullets.splice(i,1);
        if(en.health <= 0){
          scene.remove(en);
          enemies.splice(ei,1);
        }
      }
    });
  });

  // Cámara seguimiento
  camera.position.lerp(
    new THREE.Vector3(player.position.x, player.position.y+4, player.position.z+7),
    0.1
  );
  camera.lookAt(player.position);

  renderer.render(scene,camera);
}
animate();
</script>
</body>
</html>
"""

path = Path("/mnt/data/MiniBattleMobile_Vida.html")
path.write_text(html, encoding="utf-8")

path
