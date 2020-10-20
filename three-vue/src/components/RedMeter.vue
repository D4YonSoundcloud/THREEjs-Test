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
	import { SVGObject } from "../../../three.js/examples/jsm/renderers/SVGRenderer";
	import dat from 'dat.gui'
    import { SVGLoader } from 'three/examples/jsm/loaders/SVGLoader.js'


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
			let camera = new THREE.PerspectiveCamera(60, window.innerWidth/window.innerHeight, 0.1, 100 )
			let scene = new THREE.Scene()
			scene.background = new THREE.Color( 'white');
			camera.position.set(-45,30,20);
			camera.lookAt(0,0,0);
			let renderer = new THREE.WebGLRenderer({ antialias: false });
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

			let loader = new SVGLoader();

			loader.load( require('.././assets/rm_logo_red-24px.svg'), function ( data ) {

				console.log(data.paths);
				let paths = data.paths;
                console.log(paths);
                console.log(paths[0].userData);
				let group = new THREE.Group();
				group.scale.multiplyScalar(0.25);
				group.position.x = -10;
				group.position.y = 10;
				group.scale.y *= -1;

				for (let i = 0; i < paths.length; i++) {
					let fillColor = 'black';
					if (fillColor !== 'none') {
						let material = new THREE.MeshBasicMaterial({
							color: new THREE.Color().setStyle(fillColor),
							opacity: paths[i].userData.style.fillOpacity,
							transparent: paths[i].userData.style.fillOpacity < 1,
							side: THREE.DoubleSide,
							depthWrite: false,
						});
						let shapes = paths[i].toShapes(true);
						for (let j = 0; j < shapes.length; j++) {
							let shape = shapes[j];
							let SVGgeometry = new THREE.ShapeBufferGeometry(shape);
							let mesh = new THREE.Mesh(SVGgeometry, material);
							group.add(mesh);
						}
					}
					let strokeColor = path.userData.style.stroke;
					if (guiData.drawStrokes && strokeColor !== undefined && strokeColor !== 'none') {
						let material = new THREE.MeshBasicMaterial({
							color: new THREE.Color().setStyle(strokeColor),
							opacity: paths[i].userData.style.strokeOpacity,
							transparent: paths[i].userData.style.strokeOpacity < 1,
							side: THREE.DoubleSide,
							depthWrite: false,
							wireframe: guiData.strokesWireframe
						});
						for (let j = 0, jl = path.subPaths.length; j < jl; j++) {
							let subPath = paths[i].subPaths[j];
							let SVGgeometry = SVGLoader.pointsToStroke(subPath.getPoints(), path.userData.style);
							if (SVGgeometry) {
								let mesh = new THREE.Mesh(SVGgeometry, material);
								group.add(mesh);
							}
						}
					}
				}
				console.log(group);
				scene.add(group);
			});


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
			let redMeterMaterial = new THREE.MeshStandardMaterial({color: 'red',  metalness: 1 / 9, roughness: 1 - 9 / 9});
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
			pipeFront.position.z = 7.0
			pipeFront.rotation.x = Math.PI * -0.5;
			pipeBack.position.y = 4.75;
			pipeBack.position.z = -7.0
			pipeBack.rotation.x = Math.PI * -0.5;

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