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

  class DegRadHelper {
    constructor(obj, prop){
      this.obj = obj;
      this.prop = prop;
    }
    get value(){
      return THREE.MathUtils.radToDeg(this.obj[this.prop]);
    }
    set value(v){
      this.obj[this.prop] = THREE.MathUtils.degToRad(v);
    }
  }
  class StringToNumberHelper {
    constructor(obj, prop){
      this.obj = obj;
      this.prop = prop;
    }
    get value() {
      return this.obj[this.prop]
    }
    set value(v) {
      this.obj[this.prop] = parseFloat(v);
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
      let intensity = 1;
      let camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000 )
      let scene = new THREE.Scene()
      scene.background = new THREE.Color( 0x000000);
      camera.position.set(0,50,0);
      camera.up.set(0,0,1);
      camera.lookAt(0,0,0);
      let renderer = new THREE.WebGLRenderer();
      renderer.setSize(window.innerWidth, window.innerHeight);
      document.body.appendChild( renderer.domElement )

      let light = new THREE.PointLight(color, intensity);
      light.distance = 100;
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


      let createStars = (x, radius, widthSegments, heightSegments) => {
        for(let i = 0; i < x; i++){
          let starGeometry = new THREE.SphereBufferGeometry(radius, widthSegments, heightSegments);
          let starMaterial = new THREE.MeshPhongMaterial({color: 'rgb(147,214,250)', emissive: 'rgb(255,255,255)'});
          let star = new THREE.Mesh(starGeometry, starMaterial)
          star.position.y = Math.random() < 0.5 ? -Math.floor((Math.random() * 200) + 1) : Math.floor((Math.random() * 200) + 1);
          star.position.x = Math.random() < 0.5 ? -Math.floor((Math.random() * 200) + 1) : Math.floor((Math.random() * 200) + 1);
          star.position.z = Math.random() < 0.5 ? -Math.floor((Math.random() * 200) + 1) : Math.floor((Math.random() * 200) + 1);
          scene.add(star)
        }
      }



      createStars(300, 0.1, 5, 5);


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
      let createAstroids = (radius, widthSegments, heightSegments, astroidBeltRadius) => {
        for(let i = 0; i < 255; i++){
          let astroidGeometry = new THREE.SphereBufferGeometry(radius, widthSegments, heightSegments);
          let astroidMaterial = new THREE.MeshPhongMaterial({color: 'rgb(80,80,80)', emissive: 'rgb(132,132,132)'});
          let astroid = new THREE.Mesh(astroidGeometry, astroidMaterial)
          astroid.position.y = astroidBeltRadius * Math.sin(i);
          astroid.position.x = astroidBeltRadius * Math.cos(i);
          astroid.position.z = Math.random() < 0.5 ? -Math.floor((Math.random() * 3) + 1) : Math.floor((Math.random() * 3) + 1);
          astroid.scale.set(Math.random(), Math.random(), Math.random());
          astroidBelt.add(astroid)
        }
      }

      createAstroids(0.2, 2, 2, 35);
      createAstroids(0.1, 2, 2, 36);
      createAstroids(0.1, 1, 1 , 37);





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
      let sunMesh = new THREE.Mesh(sphereGeometry, sunMaterial);
      sunMesh.scale.set(5,5,5);
      solarSystem.add(sunMesh);
      objects.push(sunMesh);

      //mercury orbit
      let mercuryOrbit = new THREE.Object3D();
      mercuryOrbit.position.x = 6;
      solarSystem.add(mercuryOrbit);
      objects.push(mercuryOrbit);
      planets['mercury'] = {
        orbit: mercuryOrbit,
        radius: 6,
        speed: 40,
      };

      //mercury
      let mercuryMaterial = new THREE.MeshPhongMaterial({color: 'rgb(106,73,51)', emissive: 'rgb(78,50,30)'});
      let mercuryMesh = new THREE.Mesh(sphereGeometry, mercuryMaterial);
      mercuryMesh.scale.set(.5,.5,.5);
      mercuryOrbit.add(mercuryMesh);
      objects.push(mercuryMesh);

      //venus orbit
      let venusOrbit = new THREE.Object3D();
      venusOrbit.position.x = 10;
      solarSystem.add(venusOrbit);
      objects.push(venusOrbit);
      planets['venus'] = {
        orbit: venusOrbit,
        radius: 10,
        speed: 35,
      }

      //venus
      let venusMaterial = new THREE.MeshPhongMaterial({color: 'rgb(28,43,2)', emissive: 'rgb(32,47,1)'});
      let venusMesh = new THREE.Mesh(sphereGeometry, venusMaterial);
      venusMesh.scale.set(.65,.65,.65);
      venusOrbit.add(venusMesh);
      objects.push(venusMesh);

      //earth orbit
      let earthOrbit = new THREE.Object3D();
      earthOrbit.position.x = 20;
      solarSystem.add(earthOrbit);
      objects.push(earthOrbit);
      planets['earth'] = {
        orbit: earthOrbit,
        radius: 20,
        speed: 25
      }

      //earth
      let earthMaterial = new THREE.MeshPhongMaterial({color: 0x2233FF, emissive: 0x112244});
      let earthMesh = new THREE.Mesh(sphereGeometry, earthMaterial);
      earthOrbit.add(earthMesh);
      objects.push(earthMesh);

      //mars orbit
      let marsOrbit = new THREE.Object3D();
      marsOrbit.position.x = 30;
      solarSystem.add(marsOrbit);
      objects.push(marsOrbit);
      planets['mars'] = {
        orbit: marsOrbit,
        radius: 30,
        speed: 20
      }

      //mars
      let marsMaterial = new THREE.MeshPhongMaterial({color: 'rgb(170,52,7)', emissive: 'rgb(180,94,31)'});
      let marsMesh = new THREE.Mesh(sphereGeometry, marsMaterial);
      marsMesh.scale.set(.85,.85,.85);
      marsOrbit.add(marsMesh);
      objects.push(marsMesh);

      //jupiter orbit
      let jupiterOrbit = new THREE.Object3D();
      jupiterOrbit.position.x = 45;
      solarSystem.add(jupiterOrbit);
      objects.push(jupiterOrbit);
      planets['jupiter'] = {
        orbit: jupiterOrbit,
        radius: 45,
        speed: 10,
      }

      //jupiter
      let jupiterMaterial = new THREE.MeshPhongMaterial({color: 'rgb(213,92,48)', emissive: 'rgb(85,70,61)'});
      let jupiterMesh = new THREE.Mesh(sphereGeometry, jupiterMaterial);
      jupiterMesh.scale.set(2,2,2);
      jupiterOrbit.add(jupiterMesh);
      objects.push(jupiterMesh);

      //saturn orbit
      let saturnOrbit = new THREE.Object3D();
      saturnOrbit.position.x = 60;
      solarSystem.add(saturnOrbit);
      objects.push(saturnOrbit);
      planets['saturn'] = {
        orbit: saturnOrbit,
        radius: 60,
        speed: 5,
      }

      //saturn
      let saturnMaterial = new THREE.MeshPhongMaterial({color: 'rgb(253,206,188)', emissive: 'rgb(68,56,51)'});
      let saturnMesh = new THREE.Mesh(sphereGeometry, saturnMaterial);
      saturnMesh.scale.set(1.5,1.5,1.5);
      saturnOrbit.add(saturnMesh);
      objects.push(saturnMesh);

      let saturnBeltMaterial = new THREE.MeshPhongMaterial({
        color: 'rgb(253,206,188)',
        emissive: 'rgb(135,48,12)',
        opacity: 0.8,
        transparent: true,
      });
      const tGeometry = new THREE.TorusBufferGeometry(
              1.5, 0.1, 5, 100 );
      let torusMesh = new THREE.Mesh(tGeometry, saturnBeltMaterial )
      saturnMesh.add(torusMesh);
      const tGeometry2 = new THREE.TorusBufferGeometry(
              1.75, 0.1, 5, 100 );
      let torusMesh2 = new THREE.Mesh(tGeometry2, saturnBeltMaterial )
      saturnMesh.add(torusMesh2);
      const tGeometry3 = new THREE.TorusBufferGeometry(
              2, 0.1, 5, 100 );
      let torusMesh3 = new THREE.Mesh(tGeometry3, saturnBeltMaterial )
      saturnMesh.add(torusMesh3);

      //uranus orbit
      let uranusOrbit = new THREE.Object3D();
      uranusOrbit.position.x = 70;
      solarSystem.add(uranusOrbit);
      objects.push(uranusOrbit);
      planets['uranus'] = {
        orbit: uranusOrbit,
        radius: 70,
        speed: 2,
      }

      //uranus
      let uranusMaterial = new THREE.MeshPhongMaterial({color: 'rgb(55,245,240)', emissive: 'rgb(2,64,58)'});
      let uranusMesh = new THREE.Mesh(sphereGeometry, uranusMaterial);
      uranusMesh.scale.set(1.1,1.1,1.1);
      uranusOrbit.add(uranusMesh);
      objects.push(uranusMesh);


      //neptune orbit
      let neptuneOrbit = new THREE.Object3D();
      neptuneOrbit.position.x = 80;
      solarSystem.add(neptuneOrbit);
      objects.push(neptuneOrbit);
      planets['neptune'] = {
        orbit: neptuneOrbit,
        radius: 80,
        speed: 1,
      }

      //neptune
      let neptuneMaterial = new THREE.MeshPhongMaterial({color: 'rgb(4,145,199)', emissive: 'rgb(1,51,62)'});
      let neptuneMesh = new THREE.Mesh(sphereGeometry, neptuneMaterial);
      neptuneMesh.scale.set(1.12,1.12,1.12);
      neptuneOrbit.add(neptuneMesh);
      objects.push(neptuneMesh);

      //moon orbit
      let moonOrbit = new THREE.Object3D();
      moonOrbit.position.x = 2;
      earthOrbit.add(moonOrbit);
      planets['moon'] = {
        orbit: moonOrbit,
        radius: 2,
        speed: 1,
      }

      //moon
      let moonMaterial = new THREE.MeshPhongMaterial({color: 0x888888, emissive: 0x222222});
      let moonMesh = new THREE.Mesh(sphereGeometry, moonMaterial);
      moonMesh.scale.set(.25,.25,.25);
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
        console.log(planets);
        for(let planet in planets){
          planets[planet].orbit.position.x = planets[planet].radius * Math.cos(orbitAngle + planets[planet].speed);
          console.log(planets[planet].radius)
          planets[planet].orbit.position.y = planets[planet].radius * Math.sin(orbitAngle + planets[planet].speed);
          orbitAngle += 0.01;
          console.log( planets[planet].orbit.position.x);
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
