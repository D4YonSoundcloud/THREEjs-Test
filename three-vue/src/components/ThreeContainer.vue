<template>
  <div class="three-container">
    <button id="start">start</button>
    <button id="stop">stop</button>
    <button id="shadows" @click="goToShadows()">shadows</button>
  </div>
</template>

<script>

  import * as THREE from 'three'
  import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';
  import dat from 'dat.gui'


  const gui = new dat.GUI();

  class AxisGridHelper {
    constructor(node, units = 10){
      let axes = new THREE.AxesHelper();
      axes.material.depthTest = false;
      axes.renderOrder = 2 // for after the grid has rendered
      node.add(axes);

      let grid = new THREE.GridHelper(units, units);
      grid.material.depthTest = false;
      grid.renderOrder = 1;
      node.add(grid);

      this.grid = grid;
      this.axes = axes;
      this.visible = false;
    }
    get visible() {
      return this._visible;
    }
    set visible(v) {
      this._visible = v;
      this.grid.visible = v;
      this.axes.visible = v;
    }
  }


  export default {
    name: 'ThreeContainer',
    data(){
      return({})

    },
    methods:{
      goToShadows(){
        this.$router.push('/shadow')
      }
    },
    mounted(){
      let time;
      let planets = {}

      //set scene + lighting
      let color = 0xFFFFFF;
      let intensity = 1.25;
      let camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1500 )
      let scene = new THREE.Scene()
      scene.background = new THREE.Color( 0x000000);
      camera.position.set(0,50,0);
      camera.up.set(0,0,1);
      camera.lookAt(0,0,0);
      let renderer = new THREE.WebGLRenderer();
      renderer.setSize(window.innerWidth, window.innerHeight);
      document.body.appendChild( renderer.domElement )

      let light = new THREE.PointLight(color, intensity);
      light.distance = 600;
      light.position.y = 5;
      scene.add(light)

        let controls = new OrbitControls(camera, renderer.domElement);
        controls.target.set(0,5,0);
        controls.update();


      let planeSize = 40;

      let loader = new THREE.TextureLoader();
      let sunTexture = loader.load(require('.././assets/sun-texture.png'))
      let sunMaterial = new THREE.MeshBasicMaterial({
        map: sunTexture,
      })


      let wrapNodes = {
        'ClampToEdgeWrapping': THREE.ClampToEdgeWrapping,
        'RepeatWrapping': THREE.RepeatWrapping,
        'MirroredRepeatWrapping': THREE.MirroredRepeatWrapping
      }


      const updateTexture = () => {
        sunTexture.needsUpdate = true;
      }

      // setTimeout(() => {
      //   gui.add(new StringToNumberHelper(sunTexture, 'wrapS'), 'value', wrapNodes).name('texture.wrapS').onChange(updateTexture);
      //   gui.add(new StringToNumberHelper(sunTexture, 'wrapT'), 'value', wrapNodes).name('texture.wrapT').onChange(updateTexture);
      //   gui.add(sunTexture.repeat, 'x', 0, 5, 0.1 ).name('sunTexture.repeat.x')
      //   gui.add(sunTexture.repeat, 'y', 0, 5, .01).name('sunTexture.repeat.y');
      //   gui.add(sunTexture.offset, 'x', -2, 2, .01).name('sunTexture.offset.x');
      //   gui.add(sunTexture.offset, 'y', -2, 2, .01).name('sunTexture.offset.y');
      //   gui.add(sunTexture.center, 'x', -.5, 1.5, .01).name('sunTexture.center.x');
      //   gui.add(sunTexture.center, 'y', -.5, 1.5, .01).name('sunTexture.center.y');
      //   gui.add(new DegRadHelper(sunTexture, 'rotation'), 'value', -360, 360).name('sunTexture.rotation');
      // }, 2000)


      //set sphere constants
      let objects = [];
      let radius = 1;
      let widthSegments = 12;
      let heightSegments = 12;
      let sphereGeometry = new THREE.SphereBufferGeometry(radius, widthSegments, heightSegments)
      let sunGeometry = new THREE.SphereBufferGeometry(radius, 25,25);


      let createStars = (x, radius, widthSegments, heightSegments, distance) => {
        for(let i = 0; i < x; i++){
          let starGeometry = new THREE.SphereBufferGeometry(radius, widthSegments, heightSegments);
          let starMaterial = new THREE.MeshPhongMaterial({color: 'rgb(147,214,250)', emissive: 'rgb(255,255,255)'});
          let star = new THREE.Mesh(starGeometry, starMaterial)
          // star.position.y = Math.random() < 0.5 ? -Math.floor((Math.random() * (1000) + 500)) : Math.floor((Math.random() * (1000) + 500));
          // star.position.x = Math.random() < 0.5 ? -Math.floor((Math.random() * (1000) + 500)) : Math.floor((Math.random() * (1000) + 500));
          // star.position.z = Math.random() < 0.5 ? -Math.floor((Math.random() * (1000) + 500)) : Math.floor((Math.random() * (1000) + 500));
          star.position.y = Math.floor((Math.random() * (1000) + 500)) * Math.sin(i);
          star.position.x = Math.floor((Math.random() * (1000) + 500)) * Math.cos(i);
          star.position.z = Math.random() < 0.5 ? -Math.floor((Math.random() * 1000) + 1) : Math.floor((Math.random() * 1000) + 1);
          scene.add(star)
        }
      }



      createStars(300, 1, 3, 3);


      //solar system
      let solarSystem = new THREE.Object3D();
      scene.add(solarSystem);
      // objects.push(solarSystem);
      solarSystem.position.y = 5;

      let astroidBelt = new THREE.Object3D();
      solarSystem.add(astroidBelt);


      // let astroidBeltGeometry = new THREE.TorusBufferGeometry( 35, 1, 16, 100 );
      //
      // let astroidBelt = new THREE.Mesh( astroidBeltGeometry, AstroidBeltMaterial );
      // solarSystem.add( astroidBelt );
      // astroidBelt.rotation.x = 80
      //
      // const astroidMaterial = new THREE.PointsMaterial({
      //   color: 'white',
      //   size: 0.2,
      // });
      // const astroids = new THREE.Points(astroidBelt, astroidMaterial);
      // scene.add(astroids);
      // var dotGeometry = new THREE.Geometry();
      // dotGeometry.vertices.push(new THREE.Vector3( 0, 0, 0));
      // var dotMaterial = new THREE.PointsMaterial( { size: 1, sizeAttenuation: false } );
      // var dot = new THREE.Points( dotGeometry, dotMaterial );
      // scene.add( dot );
      let createAstroids = (radius, widthSegments, heightSegments, astroidBeltRadius) => {
        let gradient = 255;
        let gradientInterval = 0.5;
        console.log(gradientInterval)
        for(let i = 0; i < 180; i+=5){
          for(let j = 0; j < 360; j+=5) {
            let astroidGeometry = new THREE.Geometry();
            astroidGeometry.vertices.push(new THREE.Vector3( 0, 0, 0));
            console.log(`rgb(${gradient},${gradient},${gradient})`);
            let astroidMaterial = new THREE.PointsMaterial( { size: 1, sizeAttenuation: false, color: `rgb(${gradient},${gradient},${gradient})`} );
            let astroid = new THREE.Points(astroidGeometry, astroidMaterial)
            astroid.position.x = astroidBeltRadius * Math.sin(i) * Math.cos(j);
            astroid.position.y = astroidBeltRadius * Math.sin(i) * Math.sin(j);
            astroid.position.z = astroidBeltRadius * Math.cos(i);
            // let r = Math.round(astroid.position.x+astroidBeltRadius * 2.5)
            // let g = Math.round(astroid.position.y+astroidBeltRadius * 2.5)
            // let b = Math.round(astroid.position.z+astroidBeltRadius * 2.5)
            // astroid.material.color.r = r;
            // astroid.material.color.g = g;
            // astroid.material.color.b = b;
            console.log(astroid.material.color, i);
            astroidBelt.add(astroid)
          }
            gradientInterval += 0.5;
            gradient = Math.floor(gradient - gradientInterval);
        }
      }

      createAstroids(0.1, 1, 1, 30);
      // createAstroids(0.01, 2, 2, 28.53 + 35.71);
      // createAstroids(0.01, 2, 2, 28.53 + 41.51);
      // createAstroids(0.01, 2, 2, 28.53 + 43.51);
      // createAstroids(0.01, 1, 1 , 28.53 + 47.87);





      let AstroidBeltMaterial = new THREE.MeshPhongMaterial({
          color: 'rgb(255,0,0)',
          opacity: 0.8,
          transparent: true,
        });
      // const tGeometry = new THREE.TorusBufferGeometry(
      //         35, 4, 5, 600 );
      // let torusMesh = new THREE.Mesh(tGeometry, AstroidBeltMaterial )

      // scene.add(torusMesh);

      // const pMaterial = new THREE.PointsMaterial({
      //   color: 'white',
      //   size: 0.2,
      // });
      // const points = new THREE.Points(tGeometry, pMaterial);
      // points.rotation.x = 80;
      // console.log(points);
      // solarSystem.add(points);


      //sun
      let sunMesh = new THREE.Mesh(sunGeometry, sunMaterial);
      sunMesh.scale.set(28.53,28.53,28.53);
      solarSystem.add(sunMesh);
      objects.push(sunMesh);

      //mercury orbit
      let mercuryOrbit = new THREE.Object3D();
      mercuryOrbit.position.x = 28.53 + 5.79;
      solarSystem.add(mercuryOrbit);
      objects.push(mercuryOrbit);
      planets['mercury'] = {
        orbit: mercuryOrbit,
        radius: 28.53 + 5.79,
        speed: 0.0297,
        angle: 1,
        rotationSpeed: 0.00170,
      };

      //mercury
      let mercuryMaterial = new THREE.MeshPhongMaterial({color: 'rgb(106,73,51)', emissive: 'rgb(78,50,30)'});
      let mercuryMesh = new THREE.Mesh(sphereGeometry, mercuryMaterial);
      mercuryMesh.scale.set(.1,.1,.1);
      mercuryOrbit.add(mercuryMesh);
      objects.push(mercuryMesh);

      //venus orbit
      let venusOrbit = new THREE.Object3D();
      venusOrbit.position.x = 28.53 + 10.82;
      solarSystem.add(venusOrbit);
      objects.push(venusOrbit);
      planets['venus'] = {
        orbit: venusOrbit,
        radius: 28.53 + 10.82,
        speed: 0.0218,
        angle: 1,
        rotationSpeed: 0.00041,
      }

      //venus
      let venusMaterial = new THREE.MeshPhongMaterial({color: 'rgb(28,43,2)', emissive: 'rgb(32,47,1)'});
      let venusMesh = new THREE.Mesh(sphereGeometry, venusMaterial);
      venusMesh.scale.set(0.248,0.248,0.248);
      venusOrbit.add(venusMesh);
      objects.push(venusMesh);

      //earth orbit
      let earthOrbit = new THREE.Object3D();
      earthOrbit.position.x = 28.53 + 14.96;
      solarSystem.add(earthOrbit);
      objects.push(earthOrbit);
      planets['earth'] = {
        orbit: earthOrbit,
        radius: 28.53 + 14.96,
        speed: 0.0185,
        angle: 1,
        rotationSpeed: 0.01,
      }

      //earth
      let earthMaterial = new THREE.MeshPhongMaterial({color: 0x2233FF, emissive: 0x112244});
      let earthMesh = new THREE.Mesh(sphereGeometry, earthMaterial);
      earthMesh.scale.set(0.261,0.261,0.261);
      earthOrbit.add(earthMesh);
      objects.push(earthMesh);

      //mars orbit
      let marsOrbit = new THREE.Object3D();
      marsOrbit.position.x = 28.53 + 22.79;
      solarSystem.add(marsOrbit);
      objects.push(marsOrbit);
      planets['mars'] = {
        orbit: marsOrbit,
        radius: 28.53 + 22.79,
        speed: 0.015,
        angle: 1,
        rotationSpeed: 0.097,
      }

      //mars
      let marsMaterial = new THREE.MeshPhongMaterial({color: 'rgb(170,52,7)', emissive: 'rgb(180,94,31)'});
      let marsMesh = new THREE.Mesh(sphereGeometry, marsMaterial);
      marsMesh.scale.set(0.139,0.139,0.139);
      marsOrbit.add(marsMesh);
      objects.push(marsMesh);

      //jupiter orbit
      let jupiterOrbit = new THREE.Object3D();
      jupiterOrbit.position.x = 28.53 + 77.83;
      solarSystem.add(jupiterOrbit);
      objects.push(jupiterOrbit);
      planets['jupiter'] = {
        orbit: jupiterOrbit,
        radius:  28.53 + 77.83,
        speed: 0.00811,
        angle: 1,
        rotationSpeed: 0.243
      }

      //jupiter
      let jupiterMaterial = new THREE.MeshPhongMaterial({color: 'rgb(213,92,48)', emissive: 'rgb(85,70,61)'});
      let jupiterMesh = new THREE.Mesh(sphereGeometry, jupiterMaterial);
      jupiterMesh.scale.set(2.93,2.93,2.93);
      jupiterOrbit.add(jupiterMesh);
      objects.push(jupiterMesh);

      //saturn orbit
      let saturnOrbit = new THREE.Object3D();
      saturnOrbit.position.x =  28.53 + 142.7;
      solarSystem.add(saturnOrbit);
      objects.push(saturnOrbit);
      planets['saturn'] = {
        orbit: saturnOrbit,
        radius: 28.53 + 142.7,
        speed: 0.006,
        angle: 1,
        rotationSpeed: 0.2439
      }

      //saturn
      let saturnMaterial = new THREE.MeshPhongMaterial({color: 'rgb(253,206,188)', emissive: 'rgb(68,56,51)'});
      let saturnMesh = new THREE.Mesh(sphereGeometry, saturnMaterial);
      saturnMesh.scale.set(2.38,2.38,2.38);
      saturnOrbit.add(saturnMesh);
      objects.push(saturnMesh);

      let saturnBeltMaterial = new THREE.MeshPhongMaterial({
        color: 'rgb(253,206,188)',
        emissive: 'rgb(135,48,12)',
        opacity: 0.8,
        transparent: true,
      });
      const tGeometry = new THREE.TorusBufferGeometry(
              2, 0.1, 5, 100 );
      let torusMesh = new THREE.Mesh(tGeometry, saturnBeltMaterial )
      saturnMesh.add(torusMesh);
      const tGeometry2 = new THREE.TorusBufferGeometry(
              2.25, 0.1, 5, 100 );
      let torusMesh2 = new THREE.Mesh(tGeometry2, saturnBeltMaterial )
      saturnMesh.add(torusMesh2);
      const tGeometry3 = new THREE.TorusBufferGeometry(
              2.5, 0.1, 5, 100 );
      let torusMesh3 = new THREE.Mesh(tGeometry3, saturnBeltMaterial )
      saturnMesh.add(torusMesh3);

      //uranus orbit
      let uranusOrbit = new THREE.Object3D();
      uranusOrbit.position.x = 28.53 + 287.1;
      solarSystem.add(uranusOrbit);
      objects.push(uranusOrbit);
      planets['uranus'] = {
        orbit: uranusOrbit,
        radius: 28.53 + 287.1,
        speed: 0.0042,
        angle: 1,
        rotationSpeed: 0.1388,
      }

      //uranus
      let uranusMaterial = new THREE.MeshPhongMaterial({color: 'rgb(55,245,240)', emissive: 'rgb(2,64,58)'});
      let uranusMesh = new THREE.Mesh(sphereGeometry, uranusMaterial);
      uranusMesh.scale.set(1.04,1.04,1.04);
      uranusOrbit.add(uranusMesh);
      objects.push(uranusMesh);


      //neptune orbit
      let neptuneOrbit = new THREE.Object3D();
      neptuneOrbit.position.x = 28.53 + 449.71;
      solarSystem.add(neptuneOrbit);
      objects.push(neptuneOrbit);
      planets['neptune'] = {
        orbit: neptuneOrbit,
        radius: 28.53 + 449.71,
        speed: 0.0034,
        angle: 1,
        rotationSpeed: 0.1492,
      }

      //neptune
      let neptuneMaterial = new THREE.MeshPhongMaterial({color: 'rgb(4,145,199)', emissive: 'rgb(1,51,62)'});
      let neptuneMesh = new THREE.Mesh(sphereGeometry, neptuneMaterial);
      neptuneMesh.scale.set(1.015,1.015,1.015);
      neptuneOrbit.add(neptuneMesh);
      objects.push(neptuneMesh);

      //moon orbit
      let moonOrbit = new THREE.Object3D();
      moonOrbit.position.x = 0.5;
      earthOrbit.add(moonOrbit);
      planets['moon'] = {
        orbit: moonOrbit,
        radius: 0.5,
        speed: 0.001,
        angle: 1,
      }

      //moon
      let moonMaterial = new THREE.MeshPhongMaterial({color: 0x888888, emissive: 0x222222});
      let moonMesh = new THREE.Mesh(sphereGeometry, moonMaterial);
      moonMesh.scale.set(0.071,0.071,0.071);
      moonOrbit.add(moonMesh);
      objects.push(moonMesh);

      //axis grid function
      let makeAxisGrid = (node,label,units) => {
        let helper = new AxisGridHelper(node, units);
        gui.add(helper, 'visible').name(label);
      }

      makeAxisGrid(solarSystem, 'solarSystem', 25);
      makeAxisGrid(sunMesh, 'sunMesh');
      makeAxisGrid(earthOrbit, 'earthOrbit');
      makeAxisGrid(earthMesh, 'earthMesh');
      makeAxisGrid(moonOrbit, 'moonOrbit');
      makeAxisGrid(moonMesh, 'moonMesh');



      sunTexture.wrapS = THREE.RepeatWrapping;
      sunTexture.repeat.y = 0;

      console.log(sunTexture);


      document.getElementById('start').addEventListener('click', () => {
        time = requestAnimationFrame(animate)
      })
      document.getElementById('stop').addEventListener('click', () => {
        console.log('bruh', time);
        cancelAnimationFrame(time);
      })

      let orbitAngle = 1;

      const animate = (globalTime) =>{
        globalTime *= 0.001;
        time = requestAnimationFrame(animate);
        objects.forEach((obj,index) => {
          obj.rotation.y += 0.01/(index+1);
        })
        // planets['mercury'] = {
        //   orbit: marsOrbit,
        //   radius: 6,
        //   speed: 0.001,
        //   angle: 1,
        // }
        for(let planet in planets){
          planets[planet].orbit.position.x = planets[planet].radius * Math.cos(planets[planet].angle);
          planets[planet].orbit.position.y = planets[planet].radius * Math.sin(planets[planet].angle);
          planets[planet].angle += planets[planet].speed;
          // planets[planet].orbit.rotation.y += planets[planet].rotationSpeed;
        }
        astroidBelt.rotation.z += 0.01
        if(sunTexture.repeat.y >= 0.5 ){
          sunTexture.repeat.y -= 0.25
        } else {
          sunTexture.repeat.y += 0.001;
        }
        sunTexture.repeat.x = 0;
        // sunTexture.repeat.y += 0.001;
        renderer.render(scene, camera);
      }

      animate();
    }
  }

</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
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
</style>
