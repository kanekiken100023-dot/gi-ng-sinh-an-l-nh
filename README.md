<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Grand Luxury Tree - Adaptive Hand Gestures</title>
    <style>
        body { margin: 0; overflow: hidden; background-color: #000000; font-family: 'Times New Roman', serif; }
        #canvas-container { width: 100vw; height: 100vh; position: absolute; top: 0; left: 0; z-index: 1; }
        
        /* UI Overlay */
        #ui-layer {
            position: absolute; top: 0; left: 0; width: 100%; height: 100%;
            z-index: 10; pointer-events: none;
            display: flex; flex-direction: column; 
            align-items: center;
            padding-top: 40px;
            box-sizing: border-box;
            transition: opacity 0.5s ease;
        }
        
        .ui-hidden {
            opacity: 0 !important;
            pointer-events: none !important;
        }

        /* Loading */
        #loader {
            position: absolute; top: 0; left: 0; width: 100%; height: 100%;
            background: #000; z-index: 100;
            display: flex; flex-direction: column; align-items: center; justify-content: center;
            transition: opacity 0.8s ease-out;
        }
        .loader-text {
            color: #d4af37; font-size: 14px; letter-spacing: 4px; margin-top: 20px;
            text-transform: uppercase; font-weight: 100;
        }
        .spinner {
            width: 40px; height: 40px; border: 1px solid rgba(212, 175, 55, 0.2); 
            border-top: 1px solid #d4af37; border-radius: 50%; 
            animation: spin 1s linear infinite;
        }
        @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }

        /* Typography */
        h1 { 
            color: #fceea7; font-size: 56px; margin: 0; font-weight: 400; 
            letter-spacing: 6px; 
            text-shadow: 0 0 50px rgba(252, 238, 167, 0.6); 
            background: linear-gradient(to bottom, #fff, #eebb66);
            -webkit-background-clip: text; -webkit-text-fill-color: transparent;
            font-family: 'Cinzel', 'Times New Roman', serif;
            opacity: 0.9;
            transition: opacity 0.5s ease;
        }

        /* Controls */
        .controls-wrapper {
            position: absolute; 
            top: 30px;          
            right: 30px;        
            pointer-events: auto;
            display: flex;
            flex-direction: column;
            align-items: flex-end;
            gap: 10px;
            z-index: 20;
            transition: opacity 0.5s ease;
        }

        .btn-group {
            display: flex;
            gap: 10px;
        }

        .upload-btn {
            background: rgba(20, 20, 20, 0.6); 
            border: 1px solid rgba(212, 175, 55, 0.4); 
            color: #d4af37; 
            padding: 10px 20px; 
            cursor: pointer; 
            text-transform: uppercase; 
            letter-spacing: 2px; 
            font-size: 10px;
            transition: all 0.4s;
            display: flex;
            align-items: center;
            justify-content: center;
            backdrop-filter: blur(5px);
            min-width: 120px;
        }
        .upload-btn:hover { 
            background: #d4af37; 
            color: #000; 
            box-shadow: 0 0 20px rgba(212, 175, 55, 0.5);
        }
        
        .hint-text {
            color: rgba(212, 175, 55, 0.5);
            font-size: 9px;
            letter-spacing: 1px;
            text-transform: uppercase;
            text-align: right;
            margin-top: 5px;
        }

        input[type="file"] { display: none; }

        /* Webcam feedback */
        #webcam-wrapper {
            position: absolute; 
            bottom: 30px;       
            left: 30px;         
            width: 280px;       
            height: 210px;
            border: 1px solid rgba(212, 175, 55, 0.5); 
            box-shadow: 0 0 20px rgba(0,0,0,0.9);
            border-radius: 4px;
            overflow: hidden; 
            opacity: 1;         
            pointer-events: none;
            z-index: 50;
            background: #000;
            transition: opacity 0.5s ease; 
        }
        
        #webcam {
            width: 100%;
            height: 100%;
            object-fit: cover;
            transform: scaleX(-1); 
        }

        /* Debug info */
        #debug-info {
            position: absolute;
            bottom: 5px;
            left: 5px;
            color: rgba(212, 175, 55, 0.8);
            font-size: 10px;
            font-family: monospace;
            background: rgba(0,0,0,0.5);
            padding: 2px 5px;
            pointer-events: none;
        }

        /* Heart text */
        #heart-text {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            color: #ff69b4;
            font-size: 80px;
            font-family: 'Cinzel', serif;
            opacity: 0;
            pointer-events: none;
            z-index: 100;
            text-shadow: 0 0 30px #ff69b4;
            transition: opacity 0.8s ease;
        }

        /* Settings Panel */
        #toggle-controls {
            position: absolute;
            top: 30px;
            left: 30px;
            background: rgba(20,20,20,0.6);
            color: #d4af37;
            padding: 8px 12px;
            border: 1px solid rgba(212,175,55,0.4);
            border-radius: 4px;
            cursor: pointer;
            z-index: 31;
            font-size: 10px;
            text-transform: uppercase;
            letter-spacing: 1px;
        }

        #color-controls, #text-controls {
            position: absolute;
            left: 30px;
            top: 80px;
            background: rgba(0,0,0,0.7);
            padding: 15px;
            border-radius: 8px;
            border: 1px solid rgba(212,175,55,0.4);
            backdrop-filter: blur(10px);
            z-index: 30;
            pointer-events: auto;
            display: none;
            flex-direction: column;
            gap: 12px;
            color: #d4af37;
            font-size: 12px;
            min-width: 220px;
        }

        #text-controls label {
            display: flex;
            flex-direction: column;
            gap: 5px;
        }

        #text-controls input[type="text"] {
            background: rgba(20,20,20,0.8);
            border: 1px solid rgba(212,175,55,0.4);
            color: #d4af37;
            padding: 6px;
            font-family: 'Cinzel', serif;
        }

        #color-controls label {
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        #color-controls input[type="color"] {
            width: 40px;
            height: 30px;
            border: none;
            cursor: pointer;
        }
    </style>
    
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Cinzel:wght@400;700&display=swap');
    </style>

    <script type="importmap">
        {
            "imports": {
                "three": "https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js",
                "three/addons/": "https://cdn.jsdelivr.net/npm/three@0.160.0/examples/jsm/",
                "@mediapipe/tasks-vision": "https://cdn.jsdelivr.net/npm/@mediapipe/tasks-vision@0.10.3/+esm"
            }
        }
    </script>
</head>
<body>

    <div id="loader">
        <div class="spinner"></div>
        <div class="loader-text">Loading Memories</div>
    </div>

    <div id="canvas-container"></div>

    <div id="ui-layer">
        <h1>Last ember</h1>
        
        <div class="controls-wrapper">
            <div class="btn-group">
                <label class="upload-btn">
                    Select Folder
                    <input type="file" id="folder-input" webkitdirectory directory multiple>
                </label>
                
                <label class="upload-btn">
                    Select Files
                    <input type="file" id="file-input" multiple accept="image/*">
                </label>
            </div>
            <div class="hint-text">Use "Select Folder" to load all photos at once</div>
            <div class="hint-text" style="opacity: 0.7; font-size: 8px;">Or put photos in "./images/" (e.g., 1.jpg, 2.png, 3.webp)</div>
        </div>
    </div>

    <div id="webcam-wrapper">
        <video id="webcam" autoplay playsinline></video>
        <div id="debug-info">Initializing...</div>
        <canvas id="webcam-preview" style="display:none;"></canvas>
    </div>

    <div id="heart-text">With Love ❤️</div>

    <div id="toggle-controls">Settings</div>

    <div id="color-controls">
        <strong>Theme Colors</strong>
        <label>Gold <input type="color" id="color-gold" value="#ffd966"></label>
        <label>Green <input type="color" id="color-green" value="#03180a"></label>
        <label>Red <input type="color" id="color-red" value="#990000"></label>
        <label>Heart Text <input type="color" id="color-heart" value="#ff69b4"></label>
        <label>Bloom Strength <input type="range" id="bloom-strength" min="0" max="2" step="0.1" value="0.45"></label>
    </div>

    <div id="text-controls">
        <strong>Customize Texts</strong>
        <label>Title <input type="text" id="text-title" value="Lastember"></label>
        <label>Heart Message <input type="text" id="text-heart" value="With Love ❤️"></label>
    </div>

    <script type="module">
        import * as THREE from 'three';
        import { RoomEnvironment } from 'three/addons/environments/RoomEnvironment.js'; 
        import { EffectComposer } from 'three/addons/postprocessing/EffectComposer.js';
        import { RenderPass } from 'three/addons/postprocessing/RenderPass.js';
        import { UnrealBloomPass } from 'three/addons/postprocessing/UnrealBloomPass.js';
        import { FilesetResolver, HandLandmarker } from '@mediapipe/tasks-vision';

        const CONFIG = {
            colors: {
                bg: 0x000000, 
                champagneGold: 0xffd966, 
                deepGreen: 0x03180a,     
                accentRed: 0x990000,     
            },
            particles: {
                count: 1000,     
                dustCount: 1500, 
                treeHeight: 24,  
                treeRadius: 8    
            },
            camera: { z: 50 },
            
            preload: {
                autoScanLocal: true,
                scanCount: 200,
                extensions: ['jpg', 'jpeg', 'png', 'webp', 'bmp'],
                images: [
                    'https://images.unsplash.com/photo-1543589077-47d81606c1bf?q=80&w=600', 
                    'https://images.unsplash.com/photo-1576919228236-a097c32a5cd4?q=80&w=600',
                    'https://images.unsplash.com/photo-1512389142860-9c449e58a543?q=80&w=600', 
                    'https://images.unsplash.com/photo-1482638588057-dce9509db949?q=80&w=600'
                ]
            }
        };

        const STATE = {
            mode: 'TREE', 
            focusTarget: null,
            hand: { detected: false, x: 0, y: 0 },
            rotation: { x: 0, y: 0 } 
        };

        let scene, camera, renderer, composer;
        let mainGroup; 
        let clock = new THREE.Clock();
        let particleSystem = []; 
        let photoMeshGroup = new THREE.Group();
        let handLandmarker, video;
        let caneTexture; 
        const debugInfo = document.getElementById('debug-info');
        const heartText = document.getElementById('heart-text');
        const titleElement = document.querySelector('#ui-layer h1');

        async function init() {
            initThree();
            setupEnvironment(); 
            setupLights();
            createTextures();
            createParticles(); 
            createDust();     
            loadPredefinedImages();
            setupPostProcessing();
            setupEvents();
            setupControls();
            await initMediaPipe();
            
            const loader = document.getElementById('loader');
            loader.style.opacity = 0;
            setTimeout(() => loader.remove(), 800);

            animate();
        }

        function loadPredefinedImages() {
            const loader = new THREE.TextureLoader();
            
            CONFIG.preload.images.forEach(url => {
                loader.load(url, 
                    (t) => { t.colorSpace = THREE.SRGBColorSpace; addPhotoToScene(t); },
                    undefined,
                    () => {}
                );
            });

            if (CONFIG.preload.autoScanLocal) {
                for (let i = 1; i <= CONFIG.preload.scanCount; i++) {
                    CONFIG.preload.extensions.forEach(ext => {
                        const path = `./images/${i}.${ext}`;
                        loader.load(path, 
                            (t) => { 
                                t.colorSpace = THREE.SRGBColorSpace; 
                                addPhotoToScene(t); 
                            },
                            undefined,
                            () => {}
                        );
                    });
                }
            }
        }

        function initThree() {
            const container = document.getElementById('canvas-container');
            scene = new THREE.Scene();
            scene.background = new THREE.Color(CONFIG.colors.bg);
            scene.fog = new THREE.FogExp2(CONFIG.colors.bg, 0.01); 

            camera = new THREE.PerspectiveCamera(42, window.innerWidth / window.innerHeight, 0.1, 1000);
            camera.position.set(0, 2, CONFIG.camera.z); 

            renderer = new THREE.WebGLRenderer({ antialias: true, alpha: false, powerPreference: "high-performance" });
            renderer.setSize(window.innerWidth, window.innerHeight);
            renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
            renderer.toneMapping = THREE.ReinhardToneMapping; 
            renderer.toneMappingExposure = 2.2; 
            container.appendChild(renderer.domElement);

            mainGroup = new THREE.Group();
            scene.add(mainGroup);
        }

        function setupEnvironment() {
            const pmremGenerator = new THREE.PMREMGenerator(renderer);
            scene.environment = pmremGenerator.fromScene(new RoomEnvironment(), 0.04).texture;
        }

        function setupLights() {
            const ambient = new THREE.AmbientLight(0xffffff, 0.6);
            scene.add(ambient);

            const innerLight = new THREE.PointLight(0xffaa00, 2, 20);
            innerLight.position.set(0, 5, 0);
            mainGroup.add(innerLight);

            const spotGold = new THREE.SpotLight(0xffcc66, 1200);
            spotGold.position.set(30, 40, 40);
            spotGold.angle = 0.5;
            spotGold.penumbra = 0.5;
            scene.add(spotGold);

            const spotBlue = new THREE.SpotLight(0x6688ff, 600);
            spotBlue.position.set(-30, 20, -30);
            scene.add(spotBlue);
            
            const fill = new THREE.DirectionalLight(0xffeebb, 0.8);
            fill.position.set(0, 0, 50);
            scene.add(fill);
        }

        function setupPostProcessing() {
            const renderScene = new RenderPass(scene, camera);
            const bloomPass = new UnrealBloomPass(new THREE.Vector2(window.innerWidth, window.innerHeight), 1.5, 0.4, 0.85);
            bloomPass.threshold = 0.7; 
            bloomPass.strength = 0.45; 
            bloomPass.radius = 0.4;

            composer = new EffectComposer(renderer);
            composer.addPass(renderScene);
            composer.addPass(bloomPass);
        }

        function createTextures() {
            const canvas = document.createElement('canvas');
            canvas.width = 128; canvas.height = 128;
            const ctx = canvas.getContext('2d');
            ctx.fillStyle = '#ffffff';
            ctx.fillRect(0,0,128,128);
            ctx.fillStyle = '#880000'; 
            ctx.beginPath();
            for(let i=-128; i<256; i+=32) {
                ctx.moveTo(i, 0); ctx.lineTo(i+32, 128); ctx.lineTo(i+16, 128); ctx.lineTo(i-16, 0);
            }
            ctx.fill();
            caneTexture = new THREE.CanvasTexture(canvas);
            caneTexture.wrapS = THREE.RepeatWrapping;
            caneTexture.wrapT = THREE.RepeatWrapping;
            caneTexture.repeat.set(3, 3);
        }

        class Particle {
            constructor(mesh, type, isDust = false) {
                this.mesh = mesh;
                this.type = type;
                this.isDust = isDust;
                
                this.posTree = new THREE.Vector3();
                this.posScatter = new THREE.Vector3();
                this.baseScale = mesh.scale.x; 

                const speedMult = (type === 'PHOTO') ? 0.3 : 2.0;

                this.spinSpeed = new THREE.Vector3(
                    (Math.random() - 0.5) * speedMult,
                    (Math.random() - 0.5) * speedMult,
                    (Math.random() - 0.5) * speedMult
                );

                this.calculatePositions();
            }

            calculatePositions() {
                if (this.type === 'PHOTO') {
                    this.posTree.set(0, 0, 0); 
                    const rScatter = 8 + Math.random()*12;
                    const theta = Math.random() * Math.PI * 2;
                    const phi = Math.acos(2 * Math.random() - 1);
                    this.posScatter.set(
                        rScatter * Math.sin(phi) * Math.cos(theta),
                        rScatter * Math.sin(phi) * Math.sin(theta),
                        rScatter * Math.cos(phi)
                    );
                    return;
                }

                const h = CONFIG.particles.treeHeight;
                const halfH = h / 2;
                let t = Math.random(); 
                t = Math.pow(t, 0.8); 
                const y = (t * h) - halfH;
                
                let rMax = CONFIG.particles.treeRadius * (1.0 - t); 
                if (rMax < 0.5) rMax = 0.5;

                const angle = t * 50 * Math.PI + Math.random() * Math.PI; 
                const r = rMax * (0.8 + Math.random() * 0.4); 
                this.posTree.set(Math.cos(angle) * r, y, Math.sin(angle) * r);

                let rScatter = this.isDust ? (12 + Math.random()*20) : (8 + Math.random()*12);
                const theta = Math.random() * Math.PI * 2;
                const phi = Math.acos(2 * Math.random() - 1);
                this.posScatter.set(
                    rScatter * Math.sin(phi) * Math.cos(theta),
                    rScatter * Math.sin(phi) * Math.sin(theta),
                    rScatter * Math.cos(phi)
                );
            }

            update(dt, mode, focusTargetMesh) {
                let target = this.posTree;
                
                if (mode === 'SCATTER' || mode === 'HEART') {
                    target = this.posScatter;
                } else if (mode === 'FOCUS') {
                    if (this.mesh === focusTargetMesh) {
                        const desiredWorldPos = new THREE.Vector3(0, 2, 35);
                        const invMatrix = new THREE.Matrix4().copy(mainGroup.matrixWorld).invert();
                        target = desiredWorldPos.applyMatrix4(invMatrix);
                    } else {
                        target = this.posScatter;
                    }
                }

                const lerpSpeed = (mode === 'FOCUS' && this.mesh === focusTargetMesh) ? 5.0 : (mode === 'HEART' ? 8.0 : 2.0); 
                this.mesh.position.lerp(target, lerpSpeed * dt);

                if (mode === 'SCATTER' || mode === 'HEART') {
                    this.mesh.rotation.x += this.spinSpeed.x * dt * (mode === 'HEART' ? 2 : 1);
                    this.mesh.rotation.y += this.spinSpeed.y * dt * (mode === 'HEART' ? 2 : 1);
                    this.mesh.rotation.z += this.spinSpeed.z * dt * (mode === 'HEART' ? 2 : 1); 
                } else if (mode === 'TREE') {
                    if (this.type === 'PHOTO') {
                        this.mesh.lookAt(0, this.mesh.position.y, 0);
                        this.mesh.rotateY(Math.PI);
                    } else {
                        this.mesh.rotation.x = THREE.MathUtils.lerp(this.mesh.rotation.x, 0, dt);
                        this.mesh.rotation.z = THREE.MathUtils.lerp(this.mesh.rotation.z, 0, dt);
                        this.mesh.rotation.y += 0.5 * dt; 
                    }
                }
                
                if (mode === 'FOCUS' && this.mesh === focusTargetMesh) {
                    this.mesh.lookAt(camera.position); 
                }

                let s = this.baseScale;
                if (this.isDust) {
                    s = this.baseScale * (0.8 + 0.4 * Math.sin(clock.elapsedTime * 4 + this.mesh.id));
                    if (mode === 'TREE') s = 0; 
                } else if (mode === 'HEART' && this.type === 'PHOTO') {
                    s = this.baseScale * 3.5;
                } else if (mode === 'SCATTER' && this.type === 'PHOTO') {
                    s = this.baseScale * 2.5; 
                } else if (mode === 'FOCUS') {
                    if (this.mesh === focusTargetMesh) s = 4.5; 
                    else s = this.baseScale * 0.8; 
                }
                
                this.mesh.scale.lerp(new THREE.Vector3(s,s,s), (mode === 'HEART' ? 6 : 4)*dt);
            }
        }

        function updatePhotoLayout() {
            const photos = particleSystem.filter(p => p.type === 'PHOTO');
            const count = photos.length;
            if (count === 0) return;

            const h = CONFIG.particles.treeHeight * 0.9;
            const bottomY = -h/2;
            const stepY = h / count;
            const loops = 3;

            photos.forEach((p, i) => {
                const y = bottomY + stepY * i + stepY/2;
                const fullH = CONFIG.particles.treeHeight;
                const normalizedH = (y + fullH/2) / fullH; 

                let rMax = CONFIG.particles.treeRadius * (1.0 - normalizedH);
                if (rMax < 1.0) rMax = 1.0;
                
                const r = rMax + 3.0; 
                const angle = normalizedH * Math.PI * 2 * loops + (Math.PI/4); 

                p.posTree.set(Math.cos(angle) * r, y, Math.sin(angle) * r);
            });
        }

        function createParticles() {
            const sphereGeo = new THREE.SphereGeometry(0.5, 32, 32); 
            const boxGeo = new THREE.BoxGeometry(0.55, 0.55, 0.55); 
            const curve = new THREE.CatmullRomCurve3([
                new THREE.Vector3(0, -0.5, 0), new THREE.Vector3(0, 0.3, 0),
                new THREE.Vector3(0.1, 0.5, 0), new THREE.Vector3(0.3, 0.4, 0)
            ]);
            const candyGeo = new THREE.TubeGeometry(curve, 16, 0.08, 8, false);

            const goldMat = new THREE.MeshStandardMaterial({
                color: CONFIG.colors.champagneGold,
                metalness: 1.0, roughness: 0.1,
                envMapIntensity: 2.0, 
                emissive: CONFIG.colors.champagneGold,
                emissiveIntensity: 0.8
            });

            const greenMat = new THREE.MeshStandardMaterial({
                color: CONFIG.colors.deepGreen,
                metalness: 0.2, roughness: 0.8,
                emissive: CONFIG.colors.deepGreen,
                emissiveIntensity: 0.5 
            });

            const redMat = new THREE.MeshPhysicalMaterial({
                color: CONFIG.colors.accentRed,
                metalness: 0.8, roughness: 0.1, clearcoat: 1.0,
                emissive: CONFIG.colors.accentRed,
                emissiveIntensity: 1.0
            });
            
            const candyMat = new THREE.MeshStandardMaterial({ map: caneTexture, roughness: 0.4 });

            for (let i = 0; i < CONFIG.particles.count; i++) {
                const rand = Math.random();
                let mesh, type;
                
                if (rand < 0.40) {
                    mesh = new THREE.Mesh(boxGeo, greenMat);
                    type = 'BOX';
                } else if (rand < 0.70) {
                    mesh = new THREE.Mesh(boxGeo, goldMat);
                    type = 'GOLD_BOX';
                } else if (rand < 0.92) {
                    mesh = new THREE.Mesh(sphereGeo, goldMat);
                    type = 'GOLD_SPHERE';
                } else if (rand < 0.97) {
                    mesh = new THREE.Mesh(sphereGeo, redMat);
                    type = 'RED';
                } else {
                    mesh = new THREE.Mesh(candyGeo, candyMat);
                    type = 'CANE';
                }

                const s = 0.4 + Math.random() * 0.5;
                mesh.scale.set(s,s,s);
                mesh.rotation.set(Math.random()*6, Math.random()*6, Math.random()*6);
                
                mainGroup.add(mesh);
                particleSystem.push(new Particle(mesh, type, false));
            }

            const starShape = new THREE.Shape();
            const points = 5;
            const outerRadius = 1.5;
            const innerRadius = 0.7; 
            
            for (let i = 0; i < points * 2; i++) {
                const angle = (i * Math.PI) / points + Math.PI / 2;
                const r = (i % 2 === 0) ? outerRadius : innerRadius;
                const x = Math.cos(angle) * r;
                const y = Math.sin(angle) * r;
                if (i === 0) starShape.moveTo(x, y);
                else starShape.lineTo(x, y);
            }
            starShape.closePath();

            const starGeo = new THREE.ExtrudeGeometry(starShape, {
                depth: 0.4,
                bevelEnabled: true,
                bevelThickness: 0.1,
                bevelSize: 0.1,
                bevelSegments: 2
            });
            starGeo.center(); 

            const starMat = new THREE.MeshStandardMaterial({
                color: 0xffdd88, emissive: 0xffaa00, emissiveIntensity: 1.5,
                metalness: 1.0, roughness: 0
            });
            const star = new THREE.Mesh(starGeo, starMat);
            star.position.set(0, CONFIG.particles.treeHeight/2 + 1.2, 0);
            mainGroup.add(star);
            
            mainGroup.add(photoMeshGroup);
        }

        function createDust() {
            const geo = new THREE.TetrahedronGeometry(0.08, 0);
            const mat = new THREE.MeshBasicMaterial({ color: 0xa0a0ff, transparent: true, opacity: 0.8 });
            
            for(let i=0; i<CONFIG.particles.dustCount; i++) {
                 const mesh = new THREE.Mesh(geo, mat);
                 mesh.scale.setScalar(0.5 + Math.random());
                 mainGroup.add(mesh);
                 particleSystem.push(new Particle(mesh, 'DUST', true));
            }
        }

        function addPhotoToScene(texture) {
            const frameGeo = new THREE.BoxGeometry(1.4, 1.4, 0.05);
            const frameMat = new THREE.MeshStandardMaterial({ color: CONFIG.colors.champagneGold, metalness: 1.0, roughness: 0.1, emissive: CONFIG.colors.champagneGold, emissiveIntensity: 0.5 });
            const frame = new THREE.Mesh(frameGeo, frameMat);

            let width = 1.2;
            let height = 1.2;
            
            if (texture.image) {
                const aspect = texture.image.width / texture.image.height;
                if (aspect > 1) {
                    height = width / aspect;
                } else {
                    width = height * aspect;
                }
            }

            const photoGeo = new THREE.PlaneGeometry(width, height);
            const photoMat = new THREE.MeshBasicMaterial({ map: texture, side: THREE.DoubleSide });
            const photo = new THREE.Mesh(photoGeo, photoMat);
            photo.position.z = 0.04;

            const group = new THREE.Group();
            group.add(frame);
            group.add(photo);
            
            frame.scale.set(width/1.2, height/1.2, 1);

            const s = 0.8;
            group.scale.set(s,s,s);
            
            photoMeshGroup.add(group);
            particleSystem.push(new Particle(group, 'PHOTO', false));

            updatePhotoLayout();
        }
        
        function handleImageUpload(e) {
            const files = e.target.files;
            if(!files.length) return;
            
            Array.from(files).forEach(f => {
                if (!f.type.startsWith('image/')) return;
                const reader = new FileReader();
                reader.onload = (ev) => {
                    new THREE.TextureLoader().load(ev.target.result, (t) => {
                        t.colorSpace = THREE.SRGBColorSpace;
                        addPhotoToScene(t);
                    });
                }
                reader.readAsDataURL(f);
            });
        }

        async function initMediaPipe() {
            video = document.getElementById('webcam');
            
            const constraints = {
                video: {
                    width: { ideal: 640 },
                    height: { ideal: 480 },
                    frameRate: { ideal: 30 }
                }
            };

            const vision = await FilesetResolver.forVisionTasks(
                "https://cdn.jsdelivr.net/npm/@mediapipe/tasks-vision@0.10.3/wasm"
            );
            handLandmarker = await HandLandmarker.createFromOptions(vision, {
                baseOptions: {
                    modelAssetPath: `https://storage.googleapis.com/mediapipe-models/hand_landmarker/hand_landmarker/float16/1/hand_landmarker.task`,
                    delegate: "GPU"
                },
                runningMode: "VIDEO",
                numHands: 2
            });
            
            if (navigator.mediaDevices?.getUserMedia) {
                try {
                    const stream = await navigator.mediaDevices.getUserMedia(constraints);
                    video.srcObject = stream;
                    video.addEventListener("loadeddata", predictWebcam);
                    debugInfo.innerText = "Webcam active. Show hands.";
                } catch(e) {
                    console.warn("Webcam error", e);
                    debugInfo.innerText = "Camera error";
                    document.getElementById('webcam-wrapper').style.display = 'none';
                }
            }
        }

        let lastVideoTime = -1;
        async function predictWebcam() {
            if (video.currentTime !== lastVideoTime) {
                lastVideoTime = video.currentTime;
                if (handLandmarker) {
                    const result = handLandmarker.detectForVideo(video, performance.now());
                    processGestures(result);
                }
            }
            requestAnimationFrame(predictWebcam);
        }

        function processGestures(result) {
            const landmarks = result.landmarks;
            let isHeart = false;

            if (landmarks && landmarks.length >= 2) {
                const hand1 = landmarks[0];
                const hand2 = landmarks[1];

                const thumb1 = hand1[4];
                const thumb2 = hand2[4];
                const index1 = hand1[8];
                const index2 = hand2[8];
                const wrist1 = hand1[0];
                const wrist2 = hand2[0];

                const thumbDist = Math.hypot(thumb1.x - thumb2.x, thumb1.y - thumb2.y);
                const indexDist = Math.hypot(index1.x - index2.x, index1.y - index2.y);
                const wristDist = Math.abs(wrist1.x - wrist2.x);

                isHeart = thumbDist < 0.12 && indexDist < 0.12 && wristDist > 0.15;

                if (isHeart) {
                    STATE.mode = 'HEART';
                    STATE.focusTarget = null;
                    heartText.style.opacity = 1;
                    debugInfo.innerText = `Hands: 2 | Heart detected | Mode: HEART`;
                    return;
                }
            }

            heartText.style.opacity = 0;

            if (landmarks && landmarks.length >= 1) {
                const lm = landmarks[0];
                STATE.hand.detected = true;
                STATE.hand.x = (lm[9].x - 0.5) * 2;
                STATE.hand.y = (lm[9].y - 0.5) * 2;

                const thumb = lm[4];
                const index = lm[8];
                const wrist = lm[0];
                const middleMCP = lm[9];

                const handSize = Math.hypot(middleMCP.x - wrist.x, middleMCP.y - wrist.y);
                if (handSize < 0.02) return;

                const tips = [lm[8], lm[12], lm[16], lm[20]];
                let avgTipDist = 0;
                tips.forEach(t => avgTipDist += Math.hypot(t.x - wrist.x, t.y - wrist.y));
                avgTipDist /= 4;

                const pinchDist = Math.hypot(thumb.x - index.x, thumb.y - index.y);

                const extensionRatio = avgTipDist / handSize;
                const pinchRatio = pinchDist / handSize;

                debugInfo.innerText = `Hands: ${landmarks.length} | Ext: ${extensionRatio.toFixed(2)} | Pinch: ${pinchRatio.toFixed(2)} | Mode: ${STATE.mode}`;

                if (extensionRatio < 1.5) {
                    STATE.mode = 'TREE';
                    STATE.focusTarget = null;
                } else if (pinchRatio < 0.35) {
                    if (STATE.mode !== 'FOCUS') {
                        STATE.mode = 'FOCUS';
                        const photos = particleSystem.filter(p => p.type === 'PHOTO');
                        if (photos.length > 0) STATE.focusTarget = photos[Math.floor(Math.random() * photos.length)].mesh;
                    }
                } else if (extensionRatio > 1.7) {
                    STATE.mode = 'SCATTER';
                    STATE.focusTarget = null;
                }
            } else {
                STATE.hand.detected = false;
                debugInfo.innerText = "No hand detected";
            }
        }

        function setupControls() {
            const toggle = document.getElementById('toggle-controls');
            const colorPanel = document.getElementById('color-controls');
            const textPanel = document.getElementById('text-controls');

            toggle.addEventListener('click', () => {
                const anyShown = colorPanel.style.display === 'flex' || textPanel.style.display === 'flex';
                colorPanel.style.display = anyShown ? 'none' : 'flex';
                textPanel.style.display = anyShown ? 'none' : 'flex';
            });

            document.getElementById('color-gold').addEventListener('input', (e) => {
                CONFIG.colors.champagneGold = parseInt(e.target.value.slice(1), 16);
                updateMaterials();
            });
            document.getElementById('color-green').addEventListener('input', (e) => {
                CONFIG.colors.deepGreen = parseInt(e.target.value.slice(1), 16);
                updateMaterials();
            });
            document.getElementById('color-red').addEventListener('input', (e) => {
                CONFIG.colors.accentRed = parseInt(e.target.value.slice(1), 16);
                updateMaterials();
            });
            document.getElementById('color-heart').addEventListener('input', (e) => {
                heartText.style.color = e.target.value;
                heartText.style.textShadow = `0 0 30px ${e.target.value}`;
            });
            document.getElementById('bloom-strength').addEventListener('input', (e) => {
                composer.passes[1].strength = parseFloat(e.target.value);
            });

            document.getElementById('text-title').addEventListener('input', (e) => {
                titleElement.textContent = e.target.value;
            });
            document.getElementById('text-heart').addEventListener('input', (e) => {
                heartText.textContent = e.target.value;
            });
        }

        function updateMaterials() {
            scene.traverse((obj) => {
                if (obj.material && obj.material.color) {
                    if (obj.material.color.getHex() === 0xffd966 || (obj.material.emissive && obj.material.emissive.getHex() === 0xffd966)) {
                        obj.material.color.setHex(CONFIG.colors.champagneGold);
                        if (obj.material.emissive) obj.material.emissive.setHex(CONFIG.colors.champagneGold);
                    }
                    if (obj.material.color.getHex() === 0x03180a || (obj.material.emissive && obj.material.emissive.getHex() === 0x03180a)) {
                        obj.material.color.setHex(CONFIG.colors.deepGreen);
                        if (obj.material.emissive) obj.material.emissive.setHex(CONFIG.colors.deepGreen);
                    }
                    if (obj.material.color.getHex() === 0x990000 || (obj.material.emissive && obj.material.emissive.getHex() === 0x990000)) {
                        obj.material.color.setHex(CONFIG.colors.accentRed);
                        if (obj.material.emissive) obj.material.emissive.setHex(CONFIG.colors.accentRed);
                    }
                }
            });
        }

        function setupEvents() {
            window.addEventListener('resize', () => {
                camera.aspect = window.innerWidth / window.innerHeight;
                camera.updateProjectionMatrix();
                renderer.setSize(window.innerWidth, window.innerHeight);
                composer.setSize(window.innerWidth, window.innerHeight);
            });
            
            document.getElementById('file-input').addEventListener('change', handleImageUpload);
            document.getElementById('folder-input').addEventListener('change', handleImageUpload);
            
            window.addEventListener('keydown', (e) => {
                if (e.key.toLowerCase() === 'h') {
                    document.querySelector('.controls-wrapper').classList.toggle('ui-hidden');
                    document.getElementById('webcam-wrapper').classList.toggle('ui-hidden');
                }
            });
        }

        function animate() {
            requestAnimationFrame(animate);
            const dt = clock.getDelta();

            if ((STATE.mode === 'SCATTER' || STATE.mode === 'HEART') && STATE.hand.detected) {
                const targetRotY = STATE.hand.x * Math.PI * 0.9;
                const targetRotX = STATE.hand.y * Math.PI * 0.25;
                STATE.rotation.y += (targetRotY - STATE.rotation.y) * 3.0 * dt;
                STATE.rotation.x += (targetRotX - STATE.rotation.x) * 3.0 * dt;
            } else {
                if (STATE.mode === 'TREE') {
                    STATE.rotation.y += 0.3 * dt;
                    STATE.rotation.x += (0 - STATE.rotation.x) * 2.0 * dt;
                } else {
                    STATE.rotation.y += 0.1 * dt;
                }
            }

            mainGroup.rotation.y = STATE.rotation.y;
            mainGroup.rotation.x = STATE.rotation.x;

            particleSystem.forEach(p => p.update(dt, STATE.mode, STATE.focusTarget));
            composer.render();
        }

        init();
    </script>
</body>
</html>
