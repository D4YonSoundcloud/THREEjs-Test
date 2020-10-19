<template>
    <div class="three-container">
        <button id="start">start</button>
        <button id="stop">stop</button>
        <button id="red-meter" @click="goToRedMeter()">Red Meter</button>
    </div>
</template>

<script>
	import * as THREE from 'three'
	import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';
	import dat from 'dat.gui'


	const gui = new dat.GUI();

	export default {
		name: 'Shadow',
		data(){
			return({})
		},
		methods:{
            goToRedMeter(){
            	this.$router.push('/redMeter')
            }
		},
		mounted(){
			let globalTime;

			//set scene + lighting
			let color = 0xFFFFFF;
			let camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 100 )
			let scene = new THREE.Scene()
			scene.background = new THREE.Color( 'white');
			camera.position.set(0,10,20);
			camera.lookAt(0,0,0);
			let renderer = new THREE.WebGLRenderer();
			renderer.setSize(window.innerWidth, window.innerHeight);
			renderer.physicallyCorrectLights = true;
			document.body.appendChild( renderer.domElement )

			const planeSize = 40;

			const textureLoader = new THREE.TextureLoader();
			const shadowTexture = textureLoader.load('https://threejsfundamentals.org/threejs/resources/images/roundshadow.png');
			const texture = textureLoader.load('https://threejsfundamentals.org/threejs/resources/images/checker.png');
			console.log(texture);
			texture.wrapS = THREE.RepeatWrapping;
			texture.wrapT = THREE.RepeatWrapping;
			texture.magFilter = THREE.NearestFilter;
			const repeats = planeSize / 2;
			texture.repeat.set(repeats, repeats);

			const planeGeo = new THREE.PlaneBufferGeometry(planeSize, planeSize);
			const planeMat = new THREE.MeshBasicMaterial({
				map: texture,
				side: THREE.DoubleSide,
			});
			planeMat.color.setRGB(1.5, 1.5, 1.5);
			const mesh = new THREE.Mesh(planeGeo, planeMat);
			mesh.rotation.x = Math.PI * -.5;
			scene.add(mesh);
			console.log(mesh);

			let sphereShadowBases = [];

			let sphereRadius = 1;
			let sphereWidthDivisions = 32;
			let sphereHeightDivisions = 16;

			const sphereGeo = new THREE.SphereBufferGeometry(sphereRadius, sphereWidthDivisions, sphereHeightDivisions);

			let planeSize2 = 1;
			let shadowGeo = new THREE.PlaneBufferGeometry(planeSize2, planeSize2);

			let numSpheres = 15;
			for(let i = 0; i < numSpheres; i++){
				//base for shadow + sphere so they move in tandem
                let base = new THREE.Object3D();
                scene.add(base);

                //add shadow to base
                let shadowMat = new THREE.MeshBasicMaterial({
                    map: shadowTexture,
                    transparent: true, //so we can see the ground
                    depthWrite: false, //so no sorting has to take place
                })
                let shadowMesh = new THREE.Mesh(shadowGeo, shadowMat);
                shadowMesh.position.y = 0.001; //so we're slighty off the ground
                shadowMesh.rotation.x = Math.PI * -.5;
                let shadowSize = sphereRadius * 4;
                shadowMesh.scale.set(shadowSize, shadowSize, shadowSize);
                base.add(shadowMesh);

                //add sphere to the base
                let u = i / numSpheres; // goes from = to 1 as we iterate the spheres;
                let sphereMat = new THREE.MeshPhongMaterial();
                sphereMat.color.setHSL(u, 1, .75);
                let sphereMesh = new THREE.Mesh(sphereGeo, sphereMat);
                sphereMesh.position.set(0, sphereRadius + 2, 0);
                base.add(sphereMesh);

                sphereShadowBases.push({base, sphereMesh, shadowMesh, y: sphereMesh.position.y})
            }


			let skyColor = 0xB1E1FF;
			let groundColor = 0xB97A20;
			let intensity = 2;
			let light = new THREE.HemisphereLight(skyColor, groundColor, intensity);
            scene.add(light)

            let dIntensity = 1;
            let dLight = new THREE.DirectionalLight(color, dIntensity);
            dLight.position.set(0,10,5)
            dLight.target.position.set(-5,0,0)
            scene.add(dLight);

			let controls = new OrbitControls(camera, renderer.domElement);
			controls.target.set(0,5,0);
			controls.update();

			document.getElementById('start').addEventListener('click', () => {
				globalTime = requestAnimationFrame(animate)
			})
			document.getElementById('stop').addEventListener('click', () => {
				console.log('bruh', globalTime);
				cancelAnimationFrame(globalTime);
			})


			const animate = (time) =>{
				time *= 0.001;
				globalTime = requestAnimationFrame(animate);
				sphereShadowBases.forEach((sphereShadowBase, index) => {
					let { base, sphereMesh, shadowMesh, y } = sphereShadowBase;

					let u = index/sphereShadowBases.length

                    let speed = time * .2
                    let angle = speed + u * Math.PI * 2 * (index % 1 ? 1 : -1);
					let radius = Math.sin(speed - index) * 10;
					base.position.set(Math.cos(angle) * radius, 0, Math.sin(angle) * radius);

					// yOff is a value that goes from 0 to 1;
                    let yOff = Math.abs(Math.sin(time * 2 + index));
                    // move the sphere up and down
                    sphereMesh.position.y = y + THREE.MathUtils.lerp(-2, 2, yOff);
                    shadowMesh.material.opacity = THREE.MathUtils.lerp(1, 0.25, yOff);

                })
				renderer.render(scene, camera);
			}

			animate();
		}
	}
</script>

<style scoped>
    #start{
        position: absolute;
        top: 10px;
        width: 5%;
        height: 2%;
        text-align: center;
        z-index: 100;
        display:block;
        color: black;
    }
    #stop{
        position: absolute;
        top: 10px;
        left: 5%;
        width: 5%;
        height: 2%;
        text-align: center;
        z-index: 100;
        display:block;
        color: black;
    }
    #red-meter{
        position: absolute;
        top: 10px;
        left: 10%;
        width: 5%;
        height: 2%;
        text-align: center;
        z-index: 100;
        display:block;
        color: black;
    }
</style>