<template>
  <div id="cesiumContainer"></div>
  <div id="ThreeContainer"></div>
</template>
<script setup lang="ts">
(window as any).CESIUM_BASE_URL = '/Cesium/';

import * as Cesium from "cesium";
import "cesium/Build/Cesium/Widgets/widgets.css";
import * as THREE from "three";
import { onMounted } from 'vue'

function main() {
  // boundaries in WGS84 to help with syncing the renderers
  var minWGS84 = [115.23, 39.55];
  var maxWGS84 = [116.23, 41.55];
  var cesiumContainer: HTMLElement | null  = document.getElementById("cesiumContainer");
  var ThreeContainer: HTMLElement | null = document.getElementById("ThreeContainer");

  if (!ThreeContainer || !cesiumContainer) return;

  var _3Dobjects: any = []; //Could be any Three.js object mesh
  var three: any = {
    renderer: null,
    camera: null,
    scene: null,
  };

  var cesium: any = {
    viewer: null,
  };

  class _3DObject {
    graphMesh: any;
    minWGS84: any;
    maxWGS84: any;
    threeMesh: any;

    constructor() {
        this.graphMesh = null; //Three.js 3DObject.mesh
        this.minWGS84 = null; //location bounding box
        this.maxWGS84 = null;
        this.threeMesh = null;
    }
  }

  function initCesium() {
    if (!cesiumContainer) return;

    cesium.viewer = new Cesium.Viewer(cesiumContainer, {
      useDefaultRenderLoop: false,
      selectionIndicator: false,
      homeButton: false,
      sceneModePicker: false,
      navigationHelpButton: false,
      infoBox: false,
      navigationInstructionsInitiallyVisible: false,
      animation: false,
      timeline: false,
      fullscreenButton: false,
      allowTextureFilterAnisotropic: false,
      contextOptions: {
        webgl: {
          alpha: false,
          antialias: true,
          preserveDrawingBuffer: true,
          failIfMajorPerformanceCaveat: false,
          depth: true,
          stencil: false,
          // @ts-ignore
          anialias: false,
        },
      },
      targetFrameRate: 60,
      resolutionScale: 0.1,
      orderIndependentTranslucency: true,
    //   creditContainer: "hidecredit",
    //   imageryProvider: new Cesium.TileMapServiceImageryProvider({
    //     url: "Assets/imagery/NaturalEarthII/",
    //     maximumLevel: 5,
    //   }),
      baseLayerPicker: false,
      geocoder: false,
      automaticallyTrackDataSourceClocks: false,
      dataSources: undefined,
      clock: null,
      terrainShadows: Cesium.ShadowMode.DISABLED,
    });

    var center = Cesium.Cartesian3.fromDegrees(
      (minWGS84[0] + maxWGS84[0]) / 2,
      (minWGS84[1] + maxWGS84[1]) / 2 - 1,
      200000
    );
    cesium.viewer.camera.flyTo({
      destination: center,
      orientation: {
        heading: Cesium.Math.toRadians(0),
        pitch: Cesium.Math.toRadians(-60),
        roll: Cesium.Math.toRadians(0),
      },
      duration: 3,
    });
  }

  function initThree() {
    var fov = 45;
    var width = window.innerWidth;
    var height = window.innerHeight;
    var aspect = width / height;
    var near = 1;
    var far = 10 * 1000 * 1000; // needs to be far to support Cesium's world-scale rendering

    three.scene = new THREE.Scene();
    three.camera = new THREE.PerspectiveCamera(fov, aspect, near, far);
    three.renderer = new THREE.WebGLRenderer({ alpha: true });
    ThreeContainer?.appendChild(three.renderer.domElement);
  }

  function init3DObject() {
    //Cesium entity
    var entity: any = {
      name: "Polygon",
      polygon: {
        hierarchy: Cesium.Cartesian3.fromDegreesArray([
          minWGS84[0],
          minWGS84[1],
          maxWGS84[0],
          minWGS84[1],
          maxWGS84[0],
          maxWGS84[1],
          minWGS84[0],
          maxWGS84[1],
        ]),
        material: Cesium.Color.RED.withAlpha(0.2),
      },
    };
    cesium.viewer.entities.add(entity);

    // Lathe geometry
    var doubleSideMaterial = new THREE.MeshNormalMaterial({
      side: THREE.DoubleSide,
    });
    var segments = 10;
    var points = [];
    for (var i = 0; i < segments; i++) {
      points.push(
        new THREE.Vector2(Math.sin(i * 0.2) * segments + 5, (i - 5) * 2)
      );
    }
    var geometry: any = new THREE.LatheGeometry(points);
    var latheMesh = new THREE.Mesh(geometry, doubleSideMaterial);
    latheMesh.scale.set(1500, 1500, 1500); //scale object to be visible at planet scale
    latheMesh.position.z += 15000.0; // translate "up" in Three.js space so the "bottom" of the mesh is the handle
    latheMesh.rotation.x = Math.PI / 2; // rotate mesh for Cesium's Y-up system
    var latheMeshYup = new THREE.Group();
    latheMeshYup.add(latheMesh);
    three.scene.add(latheMeshYup); // don’t forget to add it to the Three.js scene manually

    //Assign Three.js object mesh to our object array
    var _3DOB = new _3DObject();
    _3DOB.threeMesh = latheMeshYup;
    _3DOB.minWGS84 = minWGS84;
    _3DOB.maxWGS84 = maxWGS84;
    _3Dobjects.push(_3DOB);

    // dodecahedron
    geometry = new THREE.DodecahedronGeometry();
    var dodecahedronMesh = new THREE.Mesh(
      geometry,
      new THREE.MeshNormalMaterial()
    );
    dodecahedronMesh.scale.set(5000, 5000, 5000); //scale object to be visible at planet scale
    dodecahedronMesh.position.z += 15000.0; // translate "up" in Three.js space so the "bottom" of the mesh is the handle
    dodecahedronMesh.rotation.x = Math.PI / 2; // rotate mesh for Cesium's Y-up system
    var dodecahedronMeshYup = new THREE.Group();
    dodecahedronMeshYup.add(dodecahedronMesh);
    three.scene.add(dodecahedronMeshYup); // don’t forget to add it to the Three.js scene manually

    //Assign Three.js object mesh to our object array
    _3DOB = new _3DObject();
    _3DOB.threeMesh = dodecahedronMeshYup;
    _3DOB.minWGS84 = minWGS84;
    _3DOB.maxWGS84 = maxWGS84;
    _3Dobjects.push(_3DOB);
  }

  function renderCesium() {
    cesium.viewer.render();
  }

  function renderThreeObj() {
    // register Three.js scene with Cesium
    three.camera.fov = Cesium.Math.toDegrees(cesium.viewer.camera.frustum.fovy); // ThreeJS FOV is vertical
    three.camera.updateProjectionMatrix();

    var cartToVec = function (cart: any) {
      return new THREE.Vector3(cart.x, cart.y, cart.z);
    };

    // Configure Three.js meshes to stand against globe center position up direction
    for (var id in _3Dobjects) {
      minWGS84 = _3Dobjects[id].minWGS84;
      maxWGS84 = _3Dobjects[id].maxWGS84;
      // convert lat/long center position to Cartesian3
      var center = Cesium.Cartesian3.fromDegrees(
        (minWGS84[0] + maxWGS84[0]) / 2,
        (minWGS84[1] + maxWGS84[1]) / 2
      );

      // get forward direction for orienting model
      var centerHigh = Cesium.Cartesian3.fromDegrees(
        (minWGS84[0] + maxWGS84[0]) / 2,
        (minWGS84[1] + maxWGS84[1]) / 2,
        1
      );

      // use direction from bottom left to top left as up-vector
      var bottomLeft = cartToVec(
        Cesium.Cartesian3.fromDegrees(minWGS84[0], minWGS84[1])
      );
      var topLeft = cartToVec(
        Cesium.Cartesian3.fromDegrees(minWGS84[0], maxWGS84[1])
      );
      var latDir = new THREE.Vector3()
        .subVectors(bottomLeft, topLeft)
        .normalize();

      // configure entity position and orientation
      _3Dobjects[id].graphMesh?.position?.copy(center);
      _3Dobjects[id].graphMesh?.lookAt(centerHigh);
      _3Dobjects[id].graphMesh?.up?.copy(latDir);
    }

    // Clone Cesium Camera projection position so the
    // Three.js Object will appear to be at the same place as above the Cesium Globe
    three.camera.matrixAutoUpdate = false;
    var cvm = cesium.viewer.camera.viewMatrix;
    var civm = cesium.viewer.camera.inverseViewMatrix;
    three.camera.matrixWorld.set(
      civm[0],
      civm[4],
      civm[8],
      civm[12],
      civm[1],
      civm[5],
      civm[9],
      civm[13],
      civm[2],
      civm[6],
      civm[10],
      civm[14],
      civm[3],
      civm[7],
      civm[11],
      civm[15]
    );
    three.camera.matrixWorldInverse.set(
      cvm[0],
      cvm[4],
      cvm[8],
      cvm[12],
      cvm[1],
      cvm[5],
      cvm[9],
      cvm[13],
      cvm[2],
      cvm[6],
      cvm[10],
      cvm[14],
      cvm[3],
      cvm[7],
      cvm[11],
      cvm[15]
    );
    three.camera.lookAt(new THREE.Vector3(0, 0, 0));

    var width = ThreeContainer?.clientWidth || 100;
    var height = ThreeContainer?.clientHeight || 100;
    var aspect = width / height;
    three.camera.aspect = aspect;
    three.camera.updateProjectionMatrix();

    three.renderer.setSize(width, height);
    three.renderer.render(three.scene, three.camera);
  }

  function loop() {
    requestAnimationFrame(loop);
    renderCesium();
    renderThreeObj();
  }

  initCesium(); // Initialize Cesium renderer
  initThree(); // Initialize Three.js renderer
  init3DObject(); // Initialize Three.js object mesh with Cesium Cartesian coordinate system
  loop(); // Looping renderer
}

onMounted(() => {
    main()
})
</script>
<style scoped>
#cesiumContainer {
    width: 100%;
    height: 100vh;
}

#ThreeContainer {
    width: 100%;
    height: 100vh;
}
</style>