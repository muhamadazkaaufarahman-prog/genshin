# genshinSaya tidak bisa membuat game selengkap *Genshin Impact* karena game sebesar itu butuh tim besar, waktu bertahun-tahun, dan biaya miliaran rupiah. Namun, saya bisa membantu Anda **membuat prototype sederhana** bergaya *Genshin* menggunakan **Unity** atau **Three.js** (web-based).

Berikut contoh **game 3D eksplorasi + karakter + musuh** versi sangat sederhana yang bisa Anda jalankan di browser (menggunakan Three.js):

```html
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>Mini Genshin - Prototype Eksplorasi</title>
    <style>
        body { margin: 0; overflow: hidden; font-family: 'Arial', sans-serif; }
        #info {
            position: absolute;
            top: 20px;
            left: 20px;
            color: white;
            background: rgba(0,0,0,0.6);
            padding: 10px 15px;
            border-radius: 8px;
            pointer-events: none;
            z-index: 10;
            font-size: 14px;
        }
        #controls {
            position: absolute;
            bottom: 20px;
            left: 20px;
            color: white;
            background: rgba(0,0,0,0.6);
            padding: 8px 12px;
            border-radius: 8px;
            font-size: 12px;
            pointer-events: none;
            z-index: 10;
        }
        #health {
            position: absolute;
            top: 20px;
            right: 20px;
            color: white;
            background: rgba(0,0,0,0.6);
            padding: 10px 15px;
            border-radius: 8px;
            font-family: monospace;
            font-size: 18px;
            font-weight: bold;
            z-index: 10;
        }
        </style>
</head>
<body>
    <div id="info">
        ⚔️ MINI GENSHIN PROTOTYPE | 🌟 Klik kiri untuk serang musuh
    </div>
    <div id="controls">
        🎮 WASD: gerak | 🖱️ Geser mouse: putar kamera | 🔫 Klik: serang
    </div>
    <div id="health">
        ❤️ Health: <span id="hpValue">100</span>
    </div>

    <!-- Import Three.js core dan addons -->
    <script type="importmap">
        {
            "imports": {
                "three": "https://unpkg.com/three@0.128.0/build/three.module.js",
                "three/addons/": "https://unpkg.com/three@0.128.0/examples/jsm/"
            }
        }
    </script>

    <script type="module">
        import * as THREE from 'three';
        import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
        import { CSS2DRenderer, CSS2DObject } from 'three/addons/renderers/CSS2DRenderer.js';

        // --- Setup Scene, Camera, Renderers ---
        const scene = new THREE.Scene();
        scene.background = new THREE.Color(0x87CEEB); // Sky blue
        scene.fog = new THREE.Fog(0x87CEEB, 20, 50);

        const camera = new THREE.PerspectiveCamera(60, window.innerWidth / window.innerHeight, 0.1, 1000);
        camera.position.set(5, 4, 8);
        camera.lookAt(0, 0, 0);

        const renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.shadowMap.enabled = true; // enable shadow
        document.body.appendChild(renderer.domElement);

        // CSS2DRenderer untuk teks di atas musuh
        const labelRenderer = new CSS2DRenderer();
        labelRenderer.setSize(window.innerWidth, window.innerHeight);
        labelRenderer.domElement.style.position = 'absolute';
        labelRenderer.domElement.style.top = '0px';
        labelRenderer.domElement.style.left = '0px';
        labelRenderer.domElement.style.pointerEvents = 'none';
        document.body.appendChild(labelRenderer.domElement);

        // --- Controls (Orbit agar mudah test, tapi kita atur target ke player) ---
        const controls = new OrbitControls(camera, renderer.domElement);
        controls.enableDamping = true;
        controls.dampingFactor = 0.05;
        controls.screenSpacePanning = true; // hindari miring
        controls.maxPolarAngle = Math.PI / 2.5;
        controls.target.set(0, 1, 0);

        // --- Lighting ---
        // Ambient light
        const ambientLight = new THREE.AmbientLight(0x404060);
        scene.add(ambientLight);
        // Main directional light
        const dirLight = new THREE.DirectionalLight(0xffffff, 1);
        dirLight.position.set(5, 10, 7);
        dirLight.castShadow = true;
        dirLight.receiveShadow = true;
        dirLight.shadow.mapSize.width = 1024;
        dirLight.shadow.mapSize.height = 1024;
        scene.add(dirLight);
        // Fill light from below
        const fillLight = new THREE.PointLight(0xcc9966, 0.3);
        fillLight.position.set(0, -1, 0);
        scene.add(fillLight);
        // Back rim light
        const rimLight = new THREE.PointLight(0xffaa66, 0.5);
        rimLight.position.set(-2, 2, -3);
        scene.add(rimLight);

        // --- Ground (grass style) ---
        const groundMat = new THREE.MeshStandardMaterial({ color: 0x6da55a, roughness: 0.8, metalness: 0.1 });
        const ground = new THREE.Mesh(new THREE.PlaneGeometry(30, 30), groundMat);
        ground.rotation.x = -Math.PI / 2;
        ground.position.y = -0.5;
        ground.receiveShadow = true;
        scene.add(ground);

        // Grid helper + decorative flowers (simple)
        const gridHelper = new THREE.GridHelper(30, 20, 0x88aa77, 0x446644);
        gridHelper.position.y = -0.4;
        scene.add(gridHelper);
        
        // beberapa pohon sederhana
        function addTree(x, z) {
            const trunkMat = new THREE.MeshStandardMaterial({ color: 0x8B5A2B });
            const leavesMat = new THREE.MeshStandardMaterial({ color: 0x5cad3a });
            const trunk = new THREE.Mesh(new THREE.CylinderGeometry(0.5, 0.6, 1.2), trunkMat);
            trunk.position.set(x, -0.2, z);
            trunk.castShadow = true;
            const leaves1 = new THREE.Mesh(new THREE.ConeGeometry(0.8, 1.0, 8), leavesMat);
            leaves1.position.set(x, 0.5, z);
            leaves1.castShadow = true;
            const leaves2 = new THREE.Mesh(new THREE.ConeGeometry(0.6, 0.8, 8), leavesMat);
            leaves2.position.set(x, 1.1, z);
            leaves2.castShadow = true;
            scene.add(trunk, leaves1, leaves2);
        }
        addTree(-5, -4);
        addTree(6, -3);
        addTree(-4, 5);
        addTree(5, 4);
        addTree(0, -6);

        // --- Player Character (anime style dengan "rambut" warna-warni)---
        const playerGroup = new THREE.Group();
        // Body
        const bodyGeo = new THREE.CylinderGeometry(0.5, 0.5, 1.0, 8);
        const bodyMat = new THREE.MeshStandardMaterial({ color: 0xffaa88, emissive: 0x442200 });
        const body = new THREE.Mesh(bodyGeo, bodyMat);
        body.castShadow = true;
        body.receiveShadow = true;
        body.position.y = 0;
        playerGroup.add(body);
        // Head
        const headGeo = new THREE.SphereGeometry(0.45, 32, 32);
        const headMat = new THREE.MeshStandardMaterial({ color: 0xffddbb });
        const head = new THREE.Mesh(headGeo, headMat);
        head.position.y = 0.75;
        head.castShadow = true;
        playerGroup.add(head);
        // Hair (spiky)
        const hairGeo = new THREE.ConeGeometry(0.5, 0.4, 8);
        const hairMat = new THREE.MeshStandardMaterial({ color: 0xd4af37 }); // gold
        const hair = new THREE.Mesh(hairGeo, hairMat);
        hair.position.y = 0.98;
        hair.castShadow = true;
        playerGroup.add(hair);
        // Eyes (simple)
        const eyeMat = new THREE.MeshStandardMaterial({ color: 0x000000 });
        const leftEye = new THREE.Mesh(new THREE.SphereGeometry(0.08, 16, 16), eyeMat);
        leftEye.position.set(-0.18, 0.85, 0.45);
        const rightEye = new THREE.Mesh(new THREE.SphereGeometry(0.08, 16, 16), eyeMat);
        rightEye.position.set(0.18, 0.85, 0.45);
        playerGroup.add(leftEye, rightEye);
        // Cape/cloak
        const capeMat = new THREE.MeshStandardMaterial({ color: 0xaa44ff });
        const cape = new THREE.Mesh(new THREE.BoxGeometry(0.7, 0.6, 0.1), capeMat);
        cape.position.set(0, 0.2, -0.45);
        cape.castShadow = true;
        playerGroup.add(cape);
        
        playerGroup.position.set(0, -0.2, 0);
        playerGroup.castShadow = true;
        scene.add(playerGroup);
        
        // --- Weapon (simple sword particle effect saat serang)---
        const weaponGroup = new THREE.Group();
        const bladeMat = new THREE.MeshStandardMaterial({ color: 0x88ccff, emissive: 0x2266aa });
        const blade = new THREE.Mesh(new THREE.BoxGeometry(0.1, 0.6, 0.05), bladeMat);
        blade.position.set(0.6, 0.2, 0.3);
        const hilt = new THREE.Mesh(new THREE.BoxGeometry(0.15, 0.15, 0.15), new THREE.MeshStandardMaterial({ color: 0xccaa77 }));
        hilt.position.set(0.5, 0.55, 0.3);
        weaponGroup.add(blade, hilt);
        playerGroup.add(weaponGroup);
        
        // --- Enemy Class (Slime-like)---
        class Enemy {
            constructor(x, z) {
                this.group = new THREE.Group();
                const bodyMat = new THREE.MeshStandardMaterial({ color: 0x44aa55, emissive: 0x226633 });
                this.mesh = new THREE.Mesh(new THREE.SphereGeometry(0.6, 32, 32), bodyMat);
                this.mesh.castShadow = true;
                this.mesh.receiveShadow = true;
                this.group.add(this.mesh);
                // Eyes
                const whiteEye = new THREE.MeshStandardMaterial({ color: 0xffffff });
                const leftEyeB = new THREE.Mesh(new THREE.SphereGeometry(0.12, 16, 16), whiteEye);
                leftEyeB.position.set(-0.2, 0.2, 0.65);
                const rightEyeB = new THREE.Mesh(new THREE.SphereGeometry(0.12, 16, 16), whiteEye);
                rightEyeB.position.set(0.2, 0.2, 0.65);
                const pupilMat = new THREE.MeshStandardMaterial({ color: 0x000000 });
                const leftPupil = new THREE.Mesh(new THREE.SphereGeometry(0.07, 16, 16), pupilMat);
                leftPupil.position.set(-0.2, 0.18, 0.78);
                const rightPupil = new THREE.Mesh(new THREE.SphereGeometry(0.07, 16, 16), pupilMat);
                rightPupil.position.set(0.2, 0.18, 0.78);
                this.group.add(leftEyeB, rightEyeB, leftPupil, rightPupil);
                
                this.group.position.set(x, -0.3, z);
                this.health = 35;
                this.maxHealth = 35;
                this.active = true;
                
                // Label Health (CSS2D)
                const div = document.createElement('div');
                div.textContent = `❤️ ${this.health}`;
                div.style.backgroundColor = 'rgba(0,0,0,0.6)';
                div.style.color = 'white';
                div.style.padding = '2px 6px';
                div.style.borderRadius = '12px';
                div.style.fontSize = '12px';
                div.style.fontWeight = 'bold';
                div.style.fontFamily = 'monospace';
                this.labelObj = new CSS2DObject(div);
                this.labelObj.position.set(0, 0.9, 0);
                this.group.add(this.labelObj);
                
                scene.add(this.group);
            }
            updateLabel() {
                if (this.labelObj && this.labelObj.element) {
                    this.labelObj.element.textContent = `❤️ ${this.health}`;
                }
                if (this.health <= 0 && this.active) {
                    this.active = false;
                    scene.remove(this.group);
                }
            }
            takeDamage(amount) {
                if (!this.active) return;
                this.health -= amount;
                this.updateLabel();
                // efek flash
                this.mesh.material.emissiveIntensity = 0.8;
                setTimeout(() => { if(this.mesh.material) this.mesh.material.emissiveIntensity = 0.2; }, 100);
                if (this.health <= 0) {
                    this.active = false;
                    scene.remove(this.group);
                }
            }
            getPosition() {
                return this.group.position;
            }
        }
        
        // --- Buat musuh ---
        const enemies = [
            new Enemy(3, 2),
            new Enemy(-2.5, 3),
            new Enemy(4, -3),
            new Enemy(-3, -2),
            new Enemy(0, 5)
        ];
        
        // --- Player Variables ---
        let playerHealth = 100;
        const moveSpeed = 3.5;
        const keys = { w: false, s: false, a: false, d: false };
        let lastAttackTime = 0;
        const attackCooldown = 0.6; // detik
        let attackActive = false;
        let attackTimer = null;
        
        // --- UI Health ---
        const hpSpan = document.getElementById('hpValue');
        
        function updatePlayerHealthUI() {
            hpSpan.textContent = playerHealth;
            if (playerHealth <= 0) {
                alert("☠️ Kamu mati! Reload halaman untuk bermain ulang.");
                document.location.reload();
            }
        }
        
        // --- Attack mechanism (swing effect & damage detection)---
        function performAttack() {
            const now = Date.now() / 1000;
            if (now - lastAttackTime < attackCooldown) return;
            lastAttackTime = now;
            // Animasi: putar weapon group
            weaponGroup.rotation.z = -0.8;
            weaponGroup.position.x = 0.2;
            setTimeout(() => {
                weaponGroup.rotation.z = 0;
                weaponGroup.position.x = 0;
            }, 150);
            
            // Deteksi musuh dalam jarak 1.8 unit
            const playerPos = playerGroup.position;
            let hit = false;
            enemies.forEach(enemy => {
                if (!enemy.active) return;
                const enemyPos = enemy.getPosition();
                const dist = playerPos.distanceTo(enemyPos);
                if (dist < 1.8) {
                    enemy.takeDamage(20);
                    hit = true;
                }
            });
            if (hit) {
                // efek partikel sederhana
                const particleGeo = new THREE.SphereGeometry(0.08, 4, 4);
                const particleMat = new THREE.MeshStandardMaterial({ color: 0xffaa44 });
                const particle = new THREE.Mesh(particleGeo, particleMat);
                particle.position.copy(playerPos);
                particle.position.y += 0.5;
                scene.add(particle);
                setTimeout(() => scene.remove(particle), 200);
            }
        }
        
        // --- Enemy AI: sederhana, chase player jika jarak < 5, dan attack---
        let lastEnemyAttack = 0;
        function updateEnemies(deltaTime) {
            const playerPos = playerGroup.position;
            enemies.forEach(enemy => {
                if (!enemy.active) return;
                const enemyPos = enemy.getPosition();
                const dx = playerPos.x - enemyPos.x;
                const dz = playerPos.z - enemyPos.z;
                const dist = Math.hypot(dx, dz);
                if (dist < 0.01) return;
                if (dist < 5) {
                    // chase
                    const move = 2.0 * deltaTime;
                    const stepX = (dx / dist) * move;
                    const stepZ = (dz / dist) * move;
                    let newX = enemyPos.x + stepX;
                    let newZ = enemyPos.z + stepZ;
                    // boundary ground -12 to 12
                    newX = Math.min(12, Math.max(-12, newX));
                    newZ = Math.min(12, Math.max(-12, newZ));
                    enemy.group.position.set(newX, enemyPos.y, newZ);
                }
                // attack player
                const nowAttack = Date.now() / 1000;
                if (dist < 1.3 && nowAttack - lastEnemyAttack > 1.2) {
                    lastEnemyAttack = nowAttack;
                    playerHealth -= 15;
                    updatePlayerHealthUI();
                    // efek merah di player
                    body.material.emissiveIntensity = 0.6;
                    setTimeout(() => { if(body.material) body.material.emissiveIntensity = 0; }, 200);
                }
            });
        }
        
        // --- Keyboard handling---
        window.addEventListener('keydown', (e) => {
            const key = e.key.toLowerCase();
            if (key === 'w') keys.w = true;
            if (key === 's') keys.s = true;
            if (key === 'a') keys.a = true;
            if (key === 'd') keys.d = true;
            if (key === ' ' || key === 'space') {
                e.preventDefault();
                performAttack();
            }
        });
        window.addEventListener('keyup', (e) => {
            const key = e.key.toLowerCase();
            if (key === 'w') keys.w = false;
            if (key === 's') keys.s = false;
            if (key === 'a') keys.a = false;
            if (key === 'd') keys.d = false;
        });
        // Mouse click attack
        window.addEventListener('click', (e) => {
            // prevent jika klik pada UI canvas? abaikan saja
            performAttack();
        });
        
        // --- Movement and Camera follow (OrbitControls target follow player)---
        let lastTime = performance.now() / 1000;
        
        function updateMovement(deltaTime) {
            let move = moveSpeed * deltaTime;
            let dx = 0, dz = 0;
            if (keys.w) dz -= move;
            if (keys.s) dz += move;
            if (keys.a) dx -= move;
            if (keys.d) dx += move;
            if (dx !== 0 || dz !== 0) {
                let newX = playerGroup.position.x + dx;
                let newZ = playerGroup.position.z + dz;
                // boundary
                newX = Math.min(13, Math.max(-13, newX));
                newZ = Math.min(13, Math.max(-13, newZ));
                playerGroup.position.set(newX, playerGroup.position.y, newZ);
                
                // Rotasi pemain menghadap arah gerak
                const angle = Math.atan2(dx, dz);
                playerGroup.rotation.y = angle;
            }
            // Update OrbitControls target ke player
            controls.target.lerp(playerGroup.position, 0.1);
        }
        
        // --- Simple floating animation untuk musuh ---
        let timeAcc = 0;
        
        // --- Game Loop ---
        function animate() {
            const now = performance.now() / 1000;
            let delta = Math.min(0.033, now - (lastTime || now));
            lastTime = now;
            
            updateMovement(delta);
            updateEnemies(delta);
            
            // Floating effect for enemies
            timeAcc += delta;
            enemies.forEach((enemy, idx) => {
                if (enemy.active) {
                    const yOff = Math.sin(timeAcc * 3 + idx) * 0.05;
                    enemy.group.position.y = -0.3 + yOff;
                }
            });
            
            // Update controls
            controls.update();
            
            renderer.render(scene, camera);
            labelRenderer.render(scene, camera);
            requestAnimationFrame(animate);
        }
        
        // start game
        updatePlayerHealthUI();
        animate();
        
        // Resize handler
        window.addEventListener('resize', onWindowResize, false);
        function onWindowResize() {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
            labelRenderer.setSize(window.innerWidth, window.innerHeight);
        }
        
        console.log("Game ready! Gunakan WASD, klik untuk serang.");
    </script>
</body>
</html>
```

## Cara menjalankan:
1. Simpan kode di atas sebagai file `index.html`
2. Buka dengan browser modern (Chrome/Edge/Firefox)
3. Mainkan: **WASD** untuk gerak, **klik kiri** atau **spasi** untuk menyerang musuh.

## Fitur yang ada:
- ✅ Karakter 3D bergaya anime (rambut emas, jubah ungu)
- ✅ Musuh slime hijau yang mengejar dan menyerang
- ✅ Sistem kesehatan & serangan jarak dekat
- ✅ Kamera orbit yang mengikuti pemain
- ✅ Pohon, grid, bayangan dinamis
- ✅ Label darah di atas musuh

## Untuk membuat game seperti Genshin Impact secara utuh, Anda butuh:
| Komponen | Tools/Langkah |
|----------|---------------|
| **Engine** | Unity atau Unreal Engine 5 |
| **3D Asset** | Blender, Character Creator, atau asset store |
| **Animasi** | Mixamo, Unity Animation System |
| **Sistem Gacha** | Database (Firebase/MySQL) + Logic pull rate |
| **Open World** | Streaming level, Terrain system, LOD |
| **Multiplayer** | Mirror, Photon, atau Netcode untuk GameObjects |
| **Skill & Element** | Scriptable Object + VFX Graph |

Apakah Anda ingin saya buatkan versi lebih lanjut (misalnya: sistem element seperti Pyro/Hydro atau gacha sederhana) menggunakan Three.js atau Unity C# script?
