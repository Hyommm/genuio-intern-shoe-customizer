<script>
  import { MeshStandardMaterial, SphereGeometry, BackSide } from "three";
  import * as THREE from "three";
  import {
    PerspectiveCamera,
    DirectionalLight,
    AmbientLight,
    OrbitControls,
    Mesh,
  } from "@threlte/core";
  import { GLTF } from "@threlte/extras";
  import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";
  import { createEventDispatcher } from "svelte";
  import { shoeData } from "./store";
  import { get } from "svelte/store";
  const eventCreater = createEventDispatcher();
  let clickedMesh;
  function showColorChangePopup(e) {
    clickedMesh = e.detail.object;
    showColorCandidates(clickedMesh);
  }
  function showColorCandidates(clickedMesh) {
    eventCreater("showColorPicker", {
      mesh: clickedMesh,
    });
  }
  function setColors(e) {
    // console.log("original model: ", e.detail);
    shoeData.set(e.detail);
    for (let i = 0; i < 7; i++) {
      $shoeData.scene.children[0].children[i].material =
        new THREE.MeshStandardMaterial({
          color: "#fff",
        });
    }
  }
  function setTextures(e) {
    let shoeData = e.detail.scene.children[0];
    for (let i = 0; i < 7; i++) {
      shoeData.children[i].material = new THREE.MeshStandardMaterial({
        map: "",
      });
    }
  }
</script>

<PerspectiveCamera position={{ x: 5, y: 10, z: 10 }} lookAt={{ y: 0 }}>
  <OrbitControls autoRotate autoRotateSpeed={0.1} />
</PerspectiveCamera>

<DirectionalLight position={{ x: 1, y: 10, z: -1 }} />
<AmbientLight />

<GLTF
  url={"/model/left_sneakers.gltf"}
  scale={2}
  position={{ x: -4, z: 1 }}
  interactive
  on:load={(event) => setColors(event)}
  on:click={(event) => showColorChangePopup(event)}
/>

<Mesh
  geometry={new SphereGeometry()}
  material={new MeshStandardMaterial({ color: "#353535", side: BackSide })}
  scale={400}
/>
