<template>
    <div class="three-container">
        <button id="start">start</button>
        <button id="stop">stop</button>
        <button id="shadows" @click="goToShadows()">Shadows</button>
        <button id="solar-system" @click="goToSolarSystem()">Solar System</button>
    </div>
</template>

<script>
	import * as THREE from 'three'
	import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';
	import dat from 'dat.gui'


	const gui = new dat.GUI();

	export default {
		name: 'RedMeter',
		data(){
			return({})
		},
		methods:{
            goToShadows(){
            	this.$router.push('/shadow');
            },
			goToSolarSystem(){
				this.$router.push('/');
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
			mesh.position.y = -5;
			scene.add(mesh);
			console.log(mesh);



			let dIntensity = 3;
			let dLight = new THREE.DirectionalLight(color, dIntensity);
			dLight.position.set(5,10,5)
			scene.add(dLight);
			let dLight2 = new THREE.DirectionalLight(color, dIntensity);
			dLight2.position.set(-5,0,-5)
			scene.add(dLight2);

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

			let radiusTop =  3.1;
			let radiusBottom =  3.1;
			let height =  12;
			let radialSegments =  8;
			let geometry = new THREE.CylinderBufferGeometry(
				radiusTop, radiusBottom, height, radialSegments);
			let redMeterMaterial = new THREE.MeshStandardMaterial({color: 'red', metalness: 0.8, roughness: 0.8});
			let redMeterMesh = new THREE.Mesh(geometry, redMeterMaterial);
			let redMeterMesh2 = new THREE.Mesh(geometry, redMeterMaterial);
			redMeterMesh.position.y = 5.5;
			redMeterMesh2.position.y = 4;
			redMeterMesh.rotation.x = Math.PI * -0.5;
			redMeterMesh2.rotation.x = Math.PI * -0.5;
			redMeterMesh.rotation.y = 89.92
			redMeterMesh2.rotation.y = 89.92
			scene.add(redMeterMesh)
			scene.add(redMeterMesh2)

			let circleRadius =  2;
			let circleSegments = 30;
			let circleGeometry = new THREE.CircleBufferGeometry(circleRadius, circleSegments);
			let circleMaterial = new THREE.MeshPhongMaterial({color: 'rgb(0,0,0)'});
			let circleMesh = new THREE.Mesh(circleGeometry, circleMaterial);
			let circleMesh2 = new THREE.Mesh(circleGeometry, circleMaterial);
			circleMesh.position.y = 4.75;
			circleMesh2.position.y = 4.75;
			circleMesh.position.z = 6.01;
			circleMesh2.position.z = -6.01;
			circleMesh2.rotation.x = Math.PI * -5;

			scene.add(circleMesh);
			scene.add(circleMesh2);

			const animate = (time) =>{
				time *= 0.001;
				globalTime = requestAnimationFrame(animate);
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
    #shadows{
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
    #solar-system{
        position: absolute;
        top: 10px;
        left: 15%;
        width: 5%;
        height: 2%;
        text-align: center;
        z-index: 100;
        display:block;
        color: black;
    }
</style>