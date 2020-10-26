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
	import {OrbitControls} from 'three/examples/jsm/controls/OrbitControls.js';
	import {SVGLoader} from 'three/examples/jsm/loaders/SVGLoader.js'
	import dat from 'dat.gui'


	const gui = new dat.GUI();

	export default {
		name: 'RedMeter',
		data(){
			return {
            }
		},
		methods:{
            goToShadows(){
            	this.$router.push('/shadow');
            },
			goToSolarSystem(){
				this.$router.push('/');
			}
		},
        computed:{
        },
		mounted(){
			let globalTime;

			//set scene + lighting
			let color = 0xFFFFFF;
			let camera = new THREE.PerspectiveCamera(60, window.innerWidth/window.innerHeight, 0.1, 120 )
			let scene = new THREE.Scene()
			scene.background = new THREE.Color( 'rgb(230,230,230)');
			camera.position.set(0,0,40);
			camera.lookAt(0,0,0);
			let renderer = new THREE.WebGLRenderer({ antialias: true });
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

			let localPlane = new THREE.Plane( new THREE.Vector3( 0.1, 0, -15.5 ), 1 );
			let localPlane2 = new THREE.Plane( new THREE.Vector3( 0.1, 0, 15.5 ), 1 );
			let localPlane3 = new THREE.Plane( new THREE.Vector3( 0, -11, 0 ), 0.8 );
			let localPlane4 = new THREE.Plane( new THREE.Vector3( 0, -5.5, 0 ), 0.8 );
			let localPlane5 = new THREE.Plane( new THREE.Vector3( 0, -6.5, 0 ), 1 );
			let globalPlane = new THREE.Plane( new THREE.Vector3( 1, 0, 0 ), 1 );
			renderer.clippingPlanes = [ localPlane, localPlane2];
			renderer.localClippingEnabled = true;
			let clippingMaterial = new THREE.MeshPhongMaterial( {
				clippingPlanes: [ localPlane ],
				clipShadows: true
			} );

			const planeGeo = new THREE.PlaneBufferGeometry(planeSize, planeSize);
			const planeMat = new THREE.MeshBasicMaterial({
				map: texture,
				side: THREE.DoubleSide,
			});
			planeMat.color.setRGB(1.5, 1.5, 1.5);
			const mesh = new THREE.Mesh(planeGeo, planeMat);
			mesh.rotation.x = Math.PI * -.5;
			mesh.position.y = -5;
			// scene.add(mesh);
			console.log(mesh);



			let dIntensity = 2;
			let dLight = new THREE.DirectionalLight(color, dIntensity);
			dLight.position.set(5,15,5)
			scene.add(dLight);
			let dLight2 = new THREE.DirectionalLight(color, dIntensity);
			dLight2.position.set(-5,-8,-5)
			scene.add(dLight2);
			let dLight3 = new THREE.DirectionalLight(color, dIntensity);
			dLight3.position.set(5,-8,-5)
			scene.add(dLight3);
			let dLight4 = new THREE.DirectionalLight(color, dIntensity);
			dLight4.position.set(-5,15,-5)
			scene.add(dLight4);
			let dLight5 = new THREE.DirectionalLight(color, 0.5);
			dLight5.position.set(5,0,0)
			scene.add(dLight5);
			let dLight6 = new THREE.DirectionalLight(color, 0.5);
			dLight6.position.set(-5,0,0)
			scene.add(dLight6);

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
			let redMeterMaterial = new THREE.MeshStandardMaterial({color: 'red',  metalness: 1 / 9, roughness: 1 - 9 / 9, clippingPlanes: [localPlane3], side: THREE.DoubleSide,});
			let redMeter2Material = new THREE.MeshStandardMaterial({color: 'red',  metalness: 1 / 9, roughness: 1 - 9 / 9, clippingPlanes: [localPlane4], side: THREE.DoubleSide,});
			let redMeterMesh = new THREE.Mesh(geometry, redMeterMaterial);
			let redMeterMesh2 = new THREE.Mesh(geometry, redMeter2Material);
			redMeterMesh.position.y = 5.5;
			redMeterMesh2.position.y = 4;
			redMeterMesh.rotation.x = Math.PI * -0.5;
			redMeterMesh2.rotation.x = Math.PI * -0.5;
			redMeterMesh.rotation.y = 89.92
			redMeterMesh2.rotation.y = 89.92
			scene.add(redMeterMesh)
			scene.add(redMeterMesh2)

            let pipeRadiusTop = 1.8;
			let pipeRadiusBottom = 1.8;
			let pipeHeight = 2;
			let pipeRadialSegments = 25;
			let pipeGeometry = new THREE.CylinderBufferGeometry(pipeRadiusTop, pipeRadiusBottom, pipeHeight, pipeRadialSegments);
			let supportPipeGeometry = new THREE.CylinderBufferGeometry(0.75, 0.75, 3, pipeRadialSegments);
			let pipeGeometrySmall = new THREE.CylinderBufferGeometry(1.5, 1.5, 1, pipeRadialSegments);
			let pipeGeometrySmallWhite = new THREE.CylinderBufferGeometry(1.5, 1.5, 4, pipeRadialSegments);
			let flangeGeometry = new THREE.CylinderBufferGeometry(2.1, 2.1, 0.5, pipeRadialSegments);
			let pipeMaterial = new THREE.MeshStandardMaterial({color: 'white', metalness: 1 / 9, roughness: 1 - 9 / 9});
			let pipeMaterialSmall = new THREE.MeshStandardMaterial({color: 'rgb(30,30,30)', metalness: 1 / 9, roughness: 1 - 9 / 9});
			let pipeFront = new THREE.Mesh(pipeGeometry, pipeMaterial);
			let pipeBack = new THREE.Mesh(pipeGeometry, pipeMaterial);
			let pipeFrontFront = new THREE.Mesh(pipeGeometrySmallWhite, new THREE.MeshStandardMaterial({color: 'rgb(52,52,52)', metalness: 1 / 9, roughness: 1 - 9 / 9, side: THREE.DoubleSide}));
			let pipeBackBack = new THREE.Mesh(pipeGeometrySmallWhite, new THREE.MeshStandardMaterial({color: 'rgb(52,52,52)', metalness: 1 / 9, roughness: 1 - 9 / 9, side: THREE.DoubleSide}));
			let pipeSmallFront = new THREE.Mesh(pipeGeometrySmall, pipeMaterialSmall);
			let pipeSmallBack = new THREE.Mesh(pipeGeometrySmall, pipeMaterialSmall);
			let flangeBlackFront = new THREE.Mesh(flangeGeometry, pipeMaterialSmall);
			let flangeBlackBack = new THREE.Mesh(flangeGeometry, pipeMaterialSmall);
			let flangeWhiteFront = new THREE.Mesh(flangeGeometry, pipeMaterial);
			let flangeWhiteBack = new THREE.Mesh(flangeGeometry, pipeMaterial);
			let pipeSmallFrontWhite = new THREE.Mesh(pipeGeometrySmallWhite, pipeMaterial);
			let pipeSmallBackWhite = new THREE.Mesh(pipeGeometrySmallWhite, pipeMaterial);
			let flangeWhiteFrontFront = new THREE.Mesh(flangeGeometry, pipeMaterial);
			let flangeWhiteBackBack = new THREE.Mesh(flangeGeometry, pipeMaterial);
			let flangeBlackFrontFront = new THREE.Mesh(flangeGeometry, pipeMaterialSmall);
			let flangeBlackBackBack = new THREE.Mesh(flangeGeometry, pipeMaterialSmall);
			let supportPipeFront = new THREE.Mesh(supportPipeGeometry, pipeMaterial)
			let supportPipeBack = new THREE.Mesh(supportPipeGeometry, pipeMaterial)

			pipeFront.position.y = 4.75;
			pipeFront.position.z = 7.01
			pipeFront.rotation.x = Math.PI * -0.5;
			pipeFrontFront.position.y = 4.75;
			pipeFrontFront.position.z = 15.5
			pipeFrontFront.rotation.x = Math.PI * -0.5;
			pipeBack.position.y = 4.75;
			pipeBack.position.z = -7.01
			pipeBack.rotation.x = Math.PI * -0.5;
			pipeBackBack.position.y = 4.75;
			pipeBackBack.position.z = -15.5
			pipeBackBack.rotation.x = Math.PI * -0.5;

			pipeSmallFront.position.y = 4.75;
			pipeSmallFront.position.z = 8.5;
			pipeSmallFront.rotation.x = Math.PI * -0.5;
			pipeSmallBack.position.y = 4.75;
			pipeSmallBack.position.z = -8.5;
			pipeSmallBack.rotation.x = Math.PI * -0.5;

			flangeBlackFront.position.y = 4.75;
			flangeBlackFront.position.z = 9;
			flangeBlackFront.rotation.x = Math.PI * -0.5;
			flangeBlackBack.position.y = 4.75;
			flangeBlackBack.position.z = -9;
			flangeBlackBack.rotation.x = Math.PI * -0.5;

			flangeWhiteFront.position.y = 4.75;
			flangeWhiteFront.position.z = 9.5;
			flangeWhiteFront.rotation.x = Math.PI * -0.5;
			flangeWhiteBack.position.y = 4.75;
			flangeWhiteBack.position.z = -9.5;
			flangeWhiteBack.rotation.x = Math.PI * -0.5;

			pipeSmallFrontWhite.position.y = 4.75;
			pipeSmallFrontWhite.position.z = 11.5;
			pipeSmallFrontWhite.rotation.x = Math.PI * -0.5;
			pipeSmallBackWhite.position.y = 4.75;
			pipeSmallBackWhite.position.z = -11.5;
			pipeSmallBackWhite.rotation.x = Math.PI * -0.5;

			flangeWhiteFrontFront.position.y = 4.75;
			flangeWhiteFrontFront.position.z = 13.5;
			flangeWhiteFrontFront.rotation.x = Math.PI * -0.5;
			flangeWhiteBackBack.position.y = 4.75;
			flangeWhiteBackBack.position.z = -13.5;
			flangeWhiteBackBack.rotation.x = Math.PI * -0.5;

			flangeBlackFrontFront.position.y = 4.75;
			flangeBlackFrontFront.position.z = 14;
			flangeBlackFrontFront.rotation.x = Math.PI * -0.5;
			flangeBlackBackBack.position.y = 4.75;
			flangeBlackBackBack.position.z = -14;
			flangeBlackBackBack.rotation.x = Math.PI * -0.5;

			supportPipeFront.position.y = 1.75;
			supportPipeFront.position.z = 7.05;
			supportPipeBack.position.y = 1.75;
			supportPipeBack.position.z = -7.05;

			scene.add(pipeSmallFront);
			scene.add(pipeSmallBack);
			scene.add(pipeFrontFront);
			scene.add(pipeBackBack);
			scene.add(flangeBlackFront);
			scene.add(flangeBlackBack);
			scene.add(flangeWhiteFront);
			scene.add(flangeWhiteBack);
			scene.add(pipeSmallFrontWhite);
			scene.add(pipeSmallBackWhite);
			scene.add(flangeWhiteFrontFront);
			scene.add(flangeWhiteBackBack);
			scene.add(flangeBlackFrontFront);
			scene.add(flangeBlackBackBack);
			scene.add(supportPipeFront);
			scene.add(supportPipeBack);
			scene.add(pipeFront);
			scene.add(pipeBack)


            let flangeBoltsBlackFront = new THREE.Object3D();
            let flangeBoltsBlackBack = new THREE.Object3D();
            let flangeBoltsWhiteFront = new THREE.Object3D();
            let flangeBoltsWhiteBack = new THREE.Object3D();
            let flangeBoltsWhiteBackBack = new THREE.Object3D();
            let flangeBoltsWhiteFrontFront = new THREE.Object3D();
			let gridHelper = new THREE.GridHelper(10,10);
            flangeBoltsBlackFront.position.y = 4.75;
            flangeBoltsBlackFront.position.z = 8.75;
			flangeBoltsBlackBack.position.y = 4.75;
			flangeBoltsBlackBack.position.z = -8.75;
			flangeBoltsWhiteFront.position.y = 4.75;
			flangeBoltsWhiteFront.position.z = -9.75;
			flangeBoltsWhiteBack.position.y = 4.75;
			flangeBoltsWhiteBack.position.z = 9.75;
			flangeBoltsWhiteBackBack.position.y = 4.75;
			flangeBoltsWhiteBackBack.position.z = 13.25;
			flangeBoltsWhiteFrontFront.position.y = 4.75;
			flangeBoltsWhiteFrontFront.position.z = -13.25;
            // flangeBoltsWhiteFrontFront.rotation.x = Math.PI * -0.5;
            scene.add(flangeBoltsBlackFront);
            scene.add(flangeBoltsBlackBack);
            scene.add(flangeBoltsWhiteFront);
            scene.add(flangeBoltsWhiteBack);
			scene.add(flangeBoltsWhiteBackBack);
			scene.add(flangeBoltsWhiteFrontFront);

            let createBoltsBlackFront = () => {
            	let angle = 0;
	            for(let i = 0; i < 8; i++){
		            let boltGeometry = new THREE.CylinderBufferGeometry(0.25,0.25,0.25,6)
		            let boltMaterial = new THREE.MeshStandardMaterial( {color: 'rgb(230,230,230)', metalness: 1 / 9, roughness: 1 - 9 / 9,} );
		            let bolt = new THREE.Mesh(boltGeometry, boltMaterial)
                    bolt.position.x = 1.75 * Math.cos(angle)
                    bolt.position.y = 1.75 * Math.sin(angle)
                    bolt.rotation.x = Math.PI * -0.5;
                    console.log(bolt, i);
                    angle += 40;
		            flangeBoltsBlackFront.add(bolt)
                }
            }
			let createBoltsBlackBack = () => {
				let angle = 0;
				for(let i = 0; i < 8; i++){
					let boltGeometry = new THREE.CylinderBufferGeometry(0.25,0.25,0.25,6)
					let boltMaterial = new THREE.MeshStandardMaterial( {color: 'rgb(230,230,230)', metalness: 1 / 9, roughness: 1 - 9 / 9,} );
					let bolt = new THREE.Mesh(boltGeometry, boltMaterial)
					bolt.position.x = 1.75 * Math.cos(angle)
					bolt.position.y = 1.75 * Math.sin(angle)
					bolt.rotation.x = Math.PI * -0.5;
					console.log(bolt, i);
					angle += 40;
					flangeBoltsBlackBack.add(bolt)
				}
			}
			let createBoltsWhiteFront = () => {
				let angle = 0;
				for(let i = 0; i < 8; i++){
					let boltGeometry = new THREE.CylinderBufferGeometry(0.25,0.25,0.25,6)
					let boltMaterial = new THREE.MeshStandardMaterial( {color: 'rgb(230,230,230)', metalness: 1 / 9, roughness: 1 - 9 / 9,} );
					let bolt = new THREE.Mesh(boltGeometry, boltMaterial)
					bolt.position.x = 1.75 * Math.cos(angle)
					bolt.position.y = 1.75 * Math.sin(angle)
					bolt.rotation.x = Math.PI * -0.5;
					console.log(bolt, i);
					angle += 40;
					flangeBoltsWhiteFront.add(bolt)
				}
			}
			let createBoltsWhiteBack = () => {
				let angle = 0;
				for(let i = 0; i < 8; i++){
					let boltGeometry = new THREE.CylinderBufferGeometry(0.25,0.25,0.25,6)
					let boltMaterial = new THREE.MeshStandardMaterial( {color: 'rgb(230,230,230)', metalness: 1 / 9, roughness: 1 - 9 / 9,} );
					let bolt = new THREE.Mesh(boltGeometry, boltMaterial)
					bolt.position.x = 1.75 * Math.cos(angle)
					bolt.position.y = 1.75 * Math.sin(angle)
					bolt.rotation.x = Math.PI * -0.5;
					console.log(bolt, i);
					angle += 40;
					flangeBoltsWhiteBack.add(bolt)
				}
			}
			let createBoltsWhiteBackBack = () => {
				let angle = 0;
				for(let i = 0; i < 8; i++){
					let boltGeometry = new THREE.CylinderBufferGeometry(0.25,0.25,0.25,6)
					let boltMaterial = new THREE.MeshStandardMaterial( {color: 'rgb(230,230,230)', metalness: 1 / 9, roughness: 1 - 9 / 9,} );
					let bolt = new THREE.Mesh(boltGeometry, boltMaterial)
					bolt.position.x = 1.75 * Math.cos(angle)
					bolt.position.y = 1.75 * Math.sin(angle)
					bolt.rotation.x = Math.PI * -0.5;
					console.log(bolt, i);
					angle += 40;
					flangeBoltsWhiteBackBack.add(bolt)
				}
			}
			let createBoltsWhiteFrontFront = () => {
				let angle = 0;
				for(let i = 0; i < 8; i++){
					let boltGeometry = new THREE.CylinderBufferGeometry(0.25,0.25,0.25,6)
					let boltMaterial = new THREE.MeshStandardMaterial( {color: 'rgb(230,230,230)', metalness: 1 / 9, roughness: 1 - 9 / 9,} );
					let bolt = new THREE.Mesh(boltGeometry, boltMaterial)
					bolt.position.x = 1.75 * Math.cos(angle)
					bolt.position.y = 1.75 * Math.sin(angle)
					bolt.rotation.x = Math.PI * -0.5;
					console.log(bolt, i);
					angle += 40;
					flangeBoltsWhiteFrontFront.add(bolt)
				}
			}

            createBoltsBlackFront();
            createBoltsBlackBack();
			createBoltsWhiteFront();
			createBoltsWhiteBack();
			createBoltsWhiteBackBack();
			createBoltsWhiteFrontFront();


			let createBoxWithRoundedEdges = ( width, height, depth, radius0, smoothness ) => {
				let shape = new THREE.Shape();
				let eps = 0.00001;
				let radius = radius0 - eps;
				shape.absarc( eps, eps, eps, -Math.PI / 2, -Math.PI, true );
				shape.absarc( eps, height -  radius * 2, eps, Math.PI, Math.PI / 2, true );
				shape.absarc( width - radius * 2, height -  radius * 2, eps, Math.PI / 2, 0, true );
				shape.absarc( width - radius * 2, eps, eps, 0, -Math.PI / 2, true );
				let geometry = new THREE.ExtrudeBufferGeometry( shape, {
					amount: depth - radius0 * 2,
					bevelEnabled: true,
					bevelSegments: smoothness * 2,
					steps: 1,
					bevelSize: radius,
					bevelThickness: radius0,
					curveSegments: smoothness
				});
				geometry.center();
				return geometry;
			}
			let cubeMat = new THREE.MeshStandardMaterial( {
				color: 'white',
				metalness: 1 / 9,
				roughness: 1 - 9 / 9,
			} );
			let cubeTest = new THREE.Mesh( createBoxWithRoundedEdges( 2, 2, 30, 1 / 9, 16 ), cubeMat );
            cubeTest.position.y = -0.75;
            scene.add(cubeTest);


            //skid
            let middleHeight = 2;
			let middleWidth = 2;
			let middleDepth = 30;
			let middleSkidGeometry = new THREE.BoxBufferGeometry(middleHeight, middleWidth, middleDepth)
            let middleSkid = new THREE.Mesh(middleSkidGeometry, pipeMaterial);
			let skidPlateLeft = new THREE.Mesh( createBoxWithRoundedEdges( 2.5, 0.22, 5, 0.5 / 9, 16 ), cubeMat );
			let skidPlateLeft2 = new THREE.Mesh( createBoxWithRoundedEdges( 2.5, 0.22, 5, 0.5 / 9, 16 ), cubeMat );
			let skidPlateLeftSmall = new THREE.Mesh( createBoxWithRoundedEdges( 1, 0.22, 6, 0.5 / 9, 16 ), cubeMat );
			let skidPlateLeftSmall2 = new THREE.Mesh( createBoxWithRoundedEdges( 1, 0.22, 6, 0.5 / 9, 16 ), cubeMat );
			let skidPlateRightSmall = new THREE.Mesh( createBoxWithRoundedEdges( 1, 0.22, 6, 0.5 / 9, 16 ), cubeMat );
			let skidPlateRightSmall2 = new THREE.Mesh( createBoxWithRoundedEdges( 1, 0.22, 6, 0.5 / 9, 16 ), cubeMat );
			let skidPlateRight = new THREE.Mesh( createBoxWithRoundedEdges( 2.5, 0.22, 5, 0.25 / 9, 16 ), cubeMat );
			let skidPlateRight2 = new THREE.Mesh( createBoxWithRoundedEdges( 2.5, 0.22, 5, 0.25 / 9, 16 ), cubeMat );
            let leftSideSkid = new THREE.Mesh( createBoxWithRoundedEdges( 2, 2, 30, 1 / 9, 16 ), cubeMat );
            let rightSideSkid = new THREE.Mesh( createBoxWithRoundedEdges( 2, 2, 30, 1 / 9, 16 ), cubeMat );
			let middleNinetySkid = new THREE.Mesh( createBoxWithRoundedEdges( 2, 2, 10, 1 / 9, 16 ), cubeMat );
			let leftNinetySkid =  new THREE.Mesh( createBoxWithRoundedEdges( 2, 2, 10, 1 / 9, 16 ), cubeMat );
			let rightNinetySkid =  new THREE.Mesh( createBoxWithRoundedEdges( 2, 2, 10, 1 / 9, 16 ), cubeMat );
			middleSkid.position.y = -0.75;
			leftSideSkid.position.y = -0.75;
			leftSideSkid.position.x = -5;
			rightSideSkid.position.y = -0.75;
			rightSideSkid.position.x = 5;
			middleNinetySkid.position.y = -0.75;
			middleNinetySkid.rotation.y = Math.PI * -0.5
			leftNinetySkid.position.y = -0.75;
			leftNinetySkid.rotation.y = Math.PI * -0.5
			leftNinetySkid.rotation.z = Math.PI * -0.5
            leftNinetySkid.position.z = 14;
			rightNinetySkid.position.y = -0.75;
			rightNinetySkid.position.z = -14;
			rightNinetySkid.rotation.y = Math.PI * -0.5
			// middleSkid.rotation.x = Math.PI * -0.5;

            skidPlateLeft.position.y = 0.50;
			skidPlateLeft.position.z = 6.95;
			skidPlateLeft.rotation.y = Math.PI * -0.5
			skidPlateLeft2.position.y = 0.30;
			skidPlateLeft2.position.z = 6.95;
			skidPlateLeft2.rotation.y = Math.PI * -0.5

			skidPlateRight.position.y = 0.50;
			skidPlateRight.position.z = -6.95;
			skidPlateRight.rotation.y = Math.PI * -0.5
			skidPlateRight2.position.y = 0.30;
			skidPlateRight2.position.z = -6.95;
			skidPlateRight2.rotation.y = Math.PI * -0.5

			skidPlateLeftSmall.position.y = 4.75;
			skidPlateLeftSmall.position.z = 6.95;
			skidPlateLeftSmall.rotation.y = Math.PI * -0.5
			skidPlateLeftSmall2.position.y = 4.55;
			skidPlateLeftSmall2.position.z = 6.95;
			skidPlateLeftSmall2.rotation.y = Math.PI * -0.5

			skidPlateRightSmall.position.y = 4.75;
			skidPlateRightSmall.position.z = -6.95;
			skidPlateRightSmall.rotation.y = Math.PI * -0.5
			skidPlateRightSmall2.position.y = 4.55;
			skidPlateRightSmall2.position.z = -6.95;
			skidPlateRightSmall2.rotation.y = Math.PI * -0.5

            //bolts
			let boltGeometry = new THREE.CylinderBufferGeometry(0.3,0.3,0.25,6)
			let boltMaterial = new THREE.MeshStandardMaterial( {color: 'rgb(182,182,182)', metalness: 0.1, roughness: 1 - 100 / 9,} );
			let boltShortFrontLeftTop = new THREE.Mesh(boltGeometry, boltMaterial)
			let boltShortFrontLeftBottom = new THREE.Mesh(boltGeometry, boltMaterial)
			let boltShortFrontRightTop = new THREE.Mesh(boltGeometry, boltMaterial)
			let boltShortFrontRightBottom = new THREE.Mesh(boltGeometry, boltMaterial)
			let boltShortBackLeftTop = new THREE.Mesh(boltGeometry, boltMaterial)
			let boltShortBackLeftBottom = new THREE.Mesh(boltGeometry, boltMaterial)
			let boltShortBackRightTop = new THREE.Mesh(boltGeometry, boltMaterial)
			let boltShortBackRightBottom = new THREE.Mesh(boltGeometry, boltMaterial)
			let boltFrontRightTopLeft = new THREE.Mesh(boltGeometry, boltMaterial)
			let boltFrontRightTopLeft2 = new THREE.Mesh(boltGeometry, boltMaterial)
			let boltFrontRightBottomLeft = new THREE.Mesh(boltGeometry, boltMaterial)
			let boltFrontRightBottomLeft2 = new THREE.Mesh(boltGeometry, boltMaterial)
			let boltFrontRightTopRight = new THREE.Mesh(boltGeometry, boltMaterial)
			let boltFrontRightTopRight2 = new THREE.Mesh(boltGeometry, boltMaterial)
			let boltFrontRightBottomRight = new THREE.Mesh(boltGeometry, boltMaterial)
			let boltFrontRightBottomRight2 = new THREE.Mesh(boltGeometry, boltMaterial)
			let boltBackRightTopLeft = new THREE.Mesh(boltGeometry, boltMaterial)
			let boltBackRightTopLeft2 = new THREE.Mesh(boltGeometry, boltMaterial)
			let boltBackRightBottomLeft = new THREE.Mesh(boltGeometry, boltMaterial)
			let boltBackRightBottomLeft2 = new THREE.Mesh(boltGeometry, boltMaterial)
			let boltBackRightTopRight = new THREE.Mesh(boltGeometry, boltMaterial)
			let boltBackRightTopRight2 = new THREE.Mesh(boltGeometry, boltMaterial)
			let boltBackRightBottomRight = new THREE.Mesh(boltGeometry, boltMaterial)
			let boltBackRightBottomRight2 = new THREE.Mesh(boltGeometry, boltMaterial)
			boltShortFrontLeftBottom.position.set(2.4,4.95,6.95);
			boltShortFrontLeftTop.position.set(2.4,4.35,6.95);
			boltShortFrontRightTop.position.set(-2.4,4.35,6.95);
			boltShortFrontRightBottom.position.set(-2.4,4.95,6.95);
			boltShortBackLeftTop.position.set(2.4,4.35,-6.95);
			boltShortBackLeftBottom.position.set(2.4,4.95,-6.95);
			boltShortBackRightTop.position.set(-2.4,4.35,-6.95);
			boltShortBackRightBottom.position.set(-2.4,4.95,-6.95);
			boltShortBackRightTop.position.set(-2.4,4.35,-6.95);
			boltShortBackRightBottom.position.set(-2.4,4.95,-6.95);
			boltFrontRightTopLeft.position.set(-2.1,0.70,-7.65);
			boltFrontRightTopLeft2.position.set(-2.1,0.70,-6.35);
			boltFrontRightBottomLeft.position.set(-2.1,0.10,-7.65);
			boltFrontRightBottomLeft2.position.set(-2.1,0.10,-6.35);
			boltFrontRightTopRight.position.set(2.1,0.70,-7.65);
			boltFrontRightTopRight2.position.set(2.1,0.70,-6.35);
			boltFrontRightBottomRight.position.set(2.1,0.10,-7.65);
			boltFrontRightBottomRight2.position.set(2.1,0.10,-6.35);
			boltBackRightTopLeft.position.set(-2.1,0.70,7.65);
			boltBackRightTopLeft2.position.set(-2.1,0.70,6.35);
			boltBackRightBottomLeft.position.set(-2.1,0.10,7.65);
			boltBackRightBottomLeft2.position.set(-2.1,0.10,6.35);
			boltBackRightTopRight.position.set(2.1,0.70,7.65);
			boltBackRightTopRight2.position.set(2.1,0.70,6.35);
			boltBackRightBottomRight.position.set(2.1,0.10,7.65);
			boltBackRightBottomRight2.position.set(2.1,0.10,6.35);

			scene.add(middleNinetySkid);
			scene.add(rightSideSkid);
			scene.add(rightNinetySkid);
			scene.add(leftSideSkid);
			scene.add(leftNinetySkid);
			scene.add(skidPlateLeft);
			scene.add(skidPlateLeft2);
			scene.add(skidPlateLeftSmall);
			scene.add(skidPlateLeftSmall2);
			scene.add(skidPlateRight);
			scene.add(skidPlateRight2);
			scene.add(skidPlateRightSmall);
			scene.add(skidPlateRightSmall2);
			scene.add(boltShortFrontLeftBottom);
			scene.add(boltShortFrontLeftTop);
			scene.add(boltShortFrontRightTop);
			scene.add(boltShortFrontRightBottom);
			scene.add(boltShortBackRightTop);
			scene.add(boltShortBackRightBottom);
			scene.add(boltShortBackLeftBottom);
			scene.add(boltShortBackLeftTop);
			scene.add(boltFrontRightTopLeft);
			scene.add(boltFrontRightTopLeft2);
			scene.add(boltFrontRightBottomLeft);
			scene.add(boltFrontRightBottomLeft2);
			scene.add(boltFrontRightTopRight);
			scene.add(boltFrontRightTopRight2);
			scene.add(boltFrontRightBottomRight);
			scene.add(boltFrontRightBottomRight2);
			scene.add(boltBackRightTopLeft);
			scene.add(boltBackRightTopLeft2);
			scene.add(boltBackRightBottomLeft);
			scene.add(boltBackRightBottomLeft2);
			scene.add(boltBackRightTopRight);
			scene.add(boltBackRightTopRight2);
			scene.add(boltBackRightBottomRight);
			scene.add(boltBackRightBottomRight2);


			let pipeSupportBackOne = new THREE.Mesh( createBoxWithRoundedEdges( 1.25, 3.25, 0.1, 0.5 / 9, 16 ), cubeMat );
			let pipeSupportBackOneTwo = new THREE.Mesh( createBoxWithRoundedEdges( 1.25, 3.25, 0.1, 0.5 / 9, 16 ), cubeMat );
			let pipeSupportBackOneMiddle = new THREE.Mesh( createBoxWithRoundedEdges( 0.9, 3.25, 1, 0.5 / 9, 16 ), cubeMat );
			let pipeSupportBackTwo = new THREE.Mesh( createBoxWithRoundedEdges( 1.25, 3.25, 0.1, 0.5 / 9, 16 ), cubeMat );
			let pipeSupportBackTwoTwo = new THREE.Mesh( createBoxWithRoundedEdges( 1.25, 3.25, 0.1, 0.5 / 9, 16 ), cubeMat );
			let pipeSupportBackTwoMiddle = new THREE.Mesh( createBoxWithRoundedEdges( 0.9, 3.25, 1, 0.5 / 9, 16 ), cubeMat );
			let pipeSupportFrontOne = new THREE.Mesh( createBoxWithRoundedEdges( 1.25, 3.25, 0.1, 0.5 / 9, 16 ), cubeMat );
			let pipeSupportFrontOneTwo = new THREE.Mesh( createBoxWithRoundedEdges( 1.25, 3.25, 0.1, 0.5 / 9, 16 ), cubeMat );
			let pipeSupportFrontOneMiddle = new THREE.Mesh( createBoxWithRoundedEdges( 0.9, 3.25, 1, 0.5 / 9, 16 ), cubeMat );
			let pipeSupportFrontTwo = new THREE.Mesh( createBoxWithRoundedEdges( 1.25, 3.25, 0.1, 0.5 / 9, 16 ), cubeMat );
			let pipeSupportFrontTwoTwo = new THREE.Mesh( createBoxWithRoundedEdges( 1.25, 3.25, 0.1, 0.5 / 9, 16 ), cubeMat );
			let pipeSupportFrontTwoMiddle = new THREE.Mesh( createBoxWithRoundedEdges( 0.9, 3.25, 1, 0.5 / 9, 16 ), cubeMat );
			pipeSupportBackOne.position.z = 10.25;
			pipeSupportBackOne.position.y = 1.75;
			pipeSupportBackOneTwo.position.z = 11.25;
			pipeSupportBackOneTwo.position.y = 1.75;
			pipeSupportBackOneMiddle.position.z = 10.75;
			pipeSupportBackOneMiddle.position.y = 1.75;
			pipeSupportBackTwo.position.z = 11.75;
			pipeSupportBackTwo.position.y = 1.75
			pipeSupportBackTwoTwo.position.z = 12.75;
			pipeSupportBackTwoTwo.position.y = 1.75
			pipeSupportBackTwoMiddle.position.z = 12.25;
			pipeSupportBackTwoMiddle.position.y = 1.75
			pipeSupportFrontOne.position.z = -10.25;
			pipeSupportFrontOne.position.y = 1.75;
			pipeSupportFrontOneTwo.position.z = -11.25;
			pipeSupportFrontOneTwo.position.y = 1.75;
			pipeSupportFrontOneMiddle.position.z = -10.75;
			pipeSupportFrontOneMiddle.position.y = 1.75;
			pipeSupportFrontTwo.position.z = -11.75;
			pipeSupportFrontTwo.position.y = 1.75
			pipeSupportFrontTwoTwo.position.z = -12.75;
			pipeSupportFrontTwoTwo.position.y = 1.75
			pipeSupportFrontTwoMiddle.position.z = -12.25;
			pipeSupportFrontTwoMiddle.position.y = 1.75
			scene.add(pipeSupportBackOne);
			scene.add(pipeSupportBackOneTwo);
			scene.add(pipeSupportBackOneMiddle);
			scene.add(pipeSupportBackTwo);
			scene.add(pipeSupportBackTwoTwo);
			scene.add(pipeSupportBackTwoMiddle);
			scene.add(pipeSupportFrontOne);
			scene.add(pipeSupportFrontOneTwo);
			scene.add(pipeSupportFrontOneMiddle);
			scene.add(pipeSupportFrontTwo);
			scene.add(pipeSupportFrontTwoTwo);
			scene.add(pipeSupportFrontTwoMiddle);


			let redMeterMiddleLeft =  new THREE.Mesh( createBoxWithRoundedEdges( 0.25, 0.4, 12, 0.5 / 9, 16 ), new THREE.MeshStandardMaterial( {
				color: 'black',
				metalness: 1 / 9,
				roughness: 1 - 9 / 9,
			}));
			redMeterMiddleLeft.position.x = 2.9;
			redMeterMiddleLeft.position.y = 4.65;
			let redMeterMiddleRight =  new THREE.Mesh( createBoxWithRoundedEdges( 0.25, 0.4, 12, 0.5 / 9, 16 ), new THREE.MeshStandardMaterial( {
				color: 'black',
				metalness: 1 / 9,
				roughness: 1 - 9 / 9,
			}));
			redMeterMiddleRight.position.x = -2.9;
			redMeterMiddleRight.position.y = 4.65;

			scene.add(redMeterMiddleLeft);
			scene.add(redMeterMiddleRight);

			let circleRadius =  2;
			let circleSegments = 30;
			let circleGeometry = new THREE.CircleBufferGeometry(circleRadius, circleSegments);
			let circleMaterial = new THREE.MeshPhongMaterial({color: 'rgb(0,0,0)'});
			let circleMesh = new THREE.Mesh(circleGeometry, circleMaterial);
			let circleMesh2 = new THREE.Mesh(circleGeometry, circleMaterial);
			circleMesh.position.y = 4.75;
			circleMesh2.position.y = 4.75;
			circleMesh.position.z = 6.02;
			circleMesh2.position.z = -6.02;
			circleMesh2.rotation.x = Math.PI * -5;



			let slurryTextureLoader = new THREE.TextureLoader();
			let slurryTexture = slurryTextureLoader.load(require('.././assets/water-bumpmap.jpg'));

			let cartridgeTexture = slurryTextureLoader.load(require('.././assets/cartridge-bumpmap.jpg'))



			let slurryRadius =  2.0;
			let slurryTubeRadius =  1.0;
			let slurryRadialSegments = 8;
			let slurryTubularSegments =  30;
			let slurryGeometry = new THREE.CylinderGeometry(1, 1, 45, pipeRadialSegments);
			let slurryMaterial = new THREE.MeshPhongMaterial({color: 'rgb(75,73,73)', bumpMap: slurryTexture})
			let slurryMesh = new THREE.Mesh(slurryGeometry, slurryMaterial )
			let slurryMesh2 = new THREE.Mesh(slurryGeometry, slurryMaterial )
			slurryMesh.position.y = 4.75;
			slurryMesh.rotation.x = Math.PI * -0.5;
			slurryMesh2.position.y = 4.75;
			slurryMesh2.position.z = -45;
			slurryMesh2.rotation.x = Math.PI * -0.5;
			let slurryCapGeometry = new THREE.CircleGeometry( 1, 32 );
			let slurryCapMaterial = new THREE.MeshPhongMaterial({color: 'rgb(75,75,75)', bumpMap: slurryTexture})
			let slurryCapFront = new THREE.Mesh( slurryCapGeometry, slurryCapMaterial );
			let slurryCapBack = new THREE.Mesh( slurryCapGeometry, slurryCapMaterial );
            slurryCapFront.position.z = 15.49;
			slurryCapFront.position.y = 4.75;
            slurryCapBack.position.z = -15.49;
			slurryCapBack.position.y = 4.75;
			slurryCapBack.rotation.x = Math.PI * -5;

			let redMeterCartridgeGeometry = new THREE.CylinderGeometry(1.1, 1.1, 14, pipeRadialSegments);
			let redMeterCartridgeMaterial = new THREE.MeshPhongMaterial({color: 'rgb(0,0,0)', clippingPlanes: [localPlane5], side: THREE.DoubleSide, bumpMap: cartridgeTexture})
			let redMeterCartridge = new THREE.Mesh(redMeterCartridgeGeometry,redMeterCartridgeMaterial)
			redMeterCartridge.position.y = 4.75;
			redMeterCartridge.rotation.x = Math.PI * -0.5;

			scene.add( slurryCapFront );
			scene.add( slurryCapBack );
            scene.add(redMeterCartridge);


			scene.add(slurryMesh);
			scene.add(slurryMesh2);
			scene.add(circleMesh);
			scene.add(circleMesh2);

			const ringInnerRadius =  .99;
			const ringOuterRadius =  1.51;
			const thetaSegments = 30;
			const ringGeometry = new THREE.RingBufferGeometry(
				ringInnerRadius, ringOuterRadius, thetaSegments);
			let ringMesh = new THREE.Mesh(ringGeometry,new THREE.MeshStandardMaterial({color: 'rgb(52,52,52)', metalness: 1 / 9, roughness: 1 - 9 / 9, side: THREE.DoubleSide}) )
			let ringMesh2 = new THREE.Mesh(ringGeometry,new THREE.MeshStandardMaterial({color: 'rgb(52,52,52)', metalness: 1 / 9, roughness: 1 - 9 / 9, side: THREE.DoubleSide}) )
            ringMesh.position.y = 4.75;
			ringMesh.position.z = 15.49;
			ringMesh2.position.y = 4.75;
			ringMesh2.position.z = -15.49;

			scene.add(ringMesh);
			scene.add(ringMesh2);

			let cameraAngle = 0.1;
			let cameraSpeed = 0.005;
			let cameraHeight = 20.1;
			let reverseCamera = false;
			let slurrySpeed = 0.1;
			let cameraZoom = 40;

			const gui = new dat.GUI();
			let localFolder = gui.addFolder('Red Meter Vision');
			let localProps = {
				get 'Enabled'(){
					return renderer.localClippingEnabled;
                },
                set 'Enabled'(v){
					renderer.localClippingEnabled = v;
                },
                get 'Shadows'(){
					return redMeterCartridgeMaterial.clipShadows;
                },
                set 'Shadows'(v){
	                redMeterCartridgeMaterial.clipShadows = v;
                },
                get 'Red Meter X-Ray'(){
					return localPlane3.constant;
                },
                set 'Red Meter X-Ray'(v){
					localPlane3.constant = v
                },
				get 'Cartridge X-Ray'(){
					return localPlane5.constant;
				},
				set 'Cartridge X-Ray'(v){
					localPlane5.constant = v
				},
				get 'Camera Speed'(){
					return cameraSpeed;
				},
				set 'Camera Speed'(v){
					cameraSpeed = v
				},
				get 'Reverse Camera'(){
					return reverseCamera;
				},
				set 'Reverse Camera'(v){
					cameraSpeed = cameraSpeed*-1;
					reverseCamera = v
				},
				get 'Camera Height'(){
					return cameraHeight;
				},
				set 'Camera Height'(v){
					cameraHeight = v
					camera.lookAt(0,5,0);
					camera.position.set(cameraZoom*Math.sin(cameraAngle += cameraSpeed), cameraHeight, cameraZoom*Math.cos(cameraAngle += cameraSpeed ));
				},
				get 'Camera Zoom'(){
					return cameraZoom;
				},
				set 'Camera Zoom'(v){
					cameraZoom = v
                    camera.lookAt(0,5,0);
					camera.position.set(cameraZoom*Math.sin(cameraAngle += cameraSpeed), cameraHeight, cameraZoom*Math.cos(cameraAngle += cameraSpeed ));
				},
				get 'Slurry Speed'(){
					return slurrySpeed;
				},
				set 'Slurry Speed'(v){
					slurrySpeed = v
				},

            }
            localFolder.add(localProps, 'Enabled')
            localFolder.add(localProps, 'Shadows')
            localFolder.add(localProps, 'Red Meter X-Ray', 0.44, 0.8);
            localFolder.add(localProps, 'Cartridge X-Ray', 0.65, 1.00);
			localFolder.add(localProps, 'Camera Speed', 0.001, 0.02);
			localFolder.add(localProps, 'Camera Height', -70.0, 70.0);
			localFolder.add(localProps, 'Camera Zoom', 1, 80);
			localFolder.add(localProps, 'Reverse Camera');
			localFolder.add(localProps, 'Slurry Speed', 0.01, 1);


			let outlineMaterial = new THREE.MeshStandardMaterial( {color: 'rgb(156,0,0)',  metalness: 1 / 9, roughness: 1 - 9 / 9, side: THREE.BackSide });
			let outlineMesh = new THREE.Mesh( geometry, outlineMaterial );
			let outlineMesh2 = new THREE.Mesh( geometry, outlineMaterial );
			let outlineMesh3 = new THREE.Mesh( redMeterCartridgeGeometry, new THREE.MeshStandardMaterial( {color: 'rgb(30,30,30)',  metalness: 1 / 9, roughness: 1 - 9 / 9, side: THREE.BackSide }) );
			outlineMesh.position.x = redMeterMesh.position.x;
			outlineMesh.position.y = redMeterMesh.position.y;
			outlineMesh.position.z = redMeterMesh.position.z;
			outlineMesh3.position.x = redMeterCartridge.position.x;
			outlineMesh3.position.y = redMeterCartridge.position.y;
			outlineMesh3.position.z = redMeterCartridge.position.z;
			outlineMesh.scale.multiplyScalar(0.99);
			outlineMesh3.scale.multiplyScalar(1.05);
			outlineMesh.rotation.x = Math.PI * -0.5
			outlineMesh3.rotation.x = Math.PI * -0.5
			outlineMesh.rotation.y = 89.92
			outlineMesh3.rotation.y = 89.92
			scene.add(outlineMesh)
			scene.add(outlineMesh3)

			const animate = (time) =>{
				globalTime = requestAnimationFrame(animate);
				// slurryMesh.position.z += time;
                console.log(time);
                console.log(slurryMesh.position.z);
                if(slurryMesh.position.z >= 38){
                	slurryMesh.position.z = -43;
                }
                if(slurryMesh2.position.z >= 38){
                	slurryMesh2.position.z = -43;
                }
                camera.lookAt(0, 5, 0);
                camera.position.set(cameraZoom*Math.sin(cameraAngle), cameraHeight, cameraZoom*Math.cos(cameraAngle));
                cameraAngle += cameraSpeed;
                slurryMesh.position.z += slurrySpeed;
                slurryMesh.rotation.y += slurrySpeed * 0.1;
				slurryMesh2.position.z += slurrySpeed;
				slurryMesh2.rotation.y += slurrySpeed * 0.1;
				slurryCapFront.rotation.z -= slurrySpeed * 0.3;
				slurryCapBack.rotation.z -= slurrySpeed * 0.3;
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
    #localPlane3Slider{
        width: 10vw;
        position: absolute;
        top: 40px;
        left: 10px;
        display: block;
        height: 2%;
    }
    .plane-text{
        position:absolute;
        top: 40px;
        left: 10vw;
    }
</style>