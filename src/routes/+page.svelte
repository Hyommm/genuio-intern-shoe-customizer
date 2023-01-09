<script>
  // @ts-nocheck

  import * as THREE from "three";
  import { Canvas } from "@threlte/core";
  import ShoeModel from "$lib/ShoeModel.svelte";
  import Modal from "$lib/Modal.svelte";
  import { shoeData } from "$lib/store";
  import { colorCandidates } from "$lib/shoeColorData";
  import { shoeTexture } from "$lib/shoeTextureData";

  /**
   * @type {string | any[]}
   */
  let areaNames = [];
  let isAreaCustomized = [];
  let selectedIndex;
  let selectedColor;
  $: if ($shoeData) {
    areaNames = $shoeData.scene.children[0].children[0].name
      .split(",")
      .map((/** @type {any} */ mesh) => mesh);
    // console.log($shoeData);
    isAreaCustomized = new Array(areaNames.length).map(() => false);
  }

  let menuChecked = false;

  /**
   * @type {{ material: THREE.MeshStandardMaterial | THREE.MeshBasicMaterial; } | null}
   */
  let clickedMesh = null;

  function colorPicker(e) {
    clickedMesh = e.detail.mesh;
    selectedIndex = $shoeData.scene.children[0].children.findIndex(
      (mesh) => mesh === clickedMesh
    );
    selectedArea = areaNames[selectedIndex];
    menuChecked = false;
  }

  function askToSelectArea() {
    alert("먼저 수정하려는 영역을 골라주세요");
    menuChecked = true;
  }

  /**
   * @param {string} colorRGB
   */
  let selectedArea = "영역을 선택해주세요";

  function setColor(colorRGB) {
    if (!clickedMesh) {
      askToSelectArea();
      return;
    }
    selectedColor = colorRGB;
    clickedMesh.material = new THREE.MeshStandardMaterial({
      color: colorRGB,
    });
    isAreaCustomized[selectedIndex] = true;
  }

  const loader = new THREE.TextureLoader();
  /**
   * @param {string} textureImg
   */
  function setTexture(textureImg) {
    if (!clickedMesh) {
      askToSelectArea();
      return;
    }
    clickedMesh.material = new THREE.MeshBasicMaterial({
      color: new THREE.Color(selectedColor),
      map: loader.load(textureImg),
      combine: THREE.MultiplyOperation,
      reflectivity: 1,
      // side: THREE.TwoPassDoubleSide,
    });
    isAreaCustomized[selectedIndex] = true;
  }

  // $: console.log("clickedMesh changed", clickedMesh);

  $: AreaEl = new Array(areaNames.length);
  function colorSelect(i) {
    clickedMesh = $shoeData.scene.children[0].children[i];
    selectedArea = areaNames[i];
    selectedIndex = i;
    menuChecked = false;
  }

  let showModal = false;
  const toggleModal = () => {
    showModal = !showModal;
  };

  // 진행바
  let progressBarStatus = 0;
  $: if (clickedMesh) {
    if (progressBarStatus < 7) {
      progressBarStatus++;
    }
  }
</script>

<section>
  <div class="absolute top-10 left-10 text-4xl text-white font-bold">
    G E N U I O
  </div>
  <div
    class="verticalText absolute top-32 left-10 text-2xl text-white font-bold"
  >
    Customize my shoes!
  </div>
  <a href="https://genu.io/" target="_blank">
    <div
      class="absolute bottom-10 left-10 rounded-full w-20 h-20 border-4 border-white hover:bg-white ease-in duration-50"
    >
      <img
        src={"icon/home.svg"}
        alt="home icon"
        class="absolute top-4 left-4 w-10"
      />
    </div>
  </a>
  <div class="w-full h-screen">
    <Canvas>
      <ShoeModel on:showColorPicker={colorPicker} />
    </Canvas>
  </div>
</section>

<div
  class="fixed top-0 right-0 w-2/7 bg-gray-700 bg-opacity-20 backdrop-blur h-full z-50 border-l-2 border-white"
>
  <div class="menu absolute top-12 left-7">
    <input
      type="checkbox"
      id="collapsible"
      bind:checked={menuChecked}
      class="hidden"
    />
    <label for="collapsible" class="text-7xl" />
  </div>
  <div
    class="h-28 border-b-2 border-white p-10 text-center text-white text-lg font-bold"
  >
    {selectedArea}
  </div>
  {#if menuChecked}
    <div class="menuitems h-1/2 p-7 border-b-2 border-white">
      <ul class="text-white text-xl font-bold">
        {#each areaNames as areaList, i}
          <li>
            <button on:click={() => colorSelect(i)}>{areaList}</button>
          </li>
        {/each}
      </ul>
    </div>
  {:else}
    <div class="h-1/2 p-7 border-b-2 border-white">
      <div class="text-white text-xl font-bold pb-3">Color</div>
      <div class="grid grid-cols-6 gap-y-4">
        <!-- color -->
        {#each colorCandidates as color}
          <div>
            <div
              class="rounded-full w-14 h-14"
              style={`background-color: ${color.colorCode}`}
              on:click={() => setColor(color.colorCode)}
            />
            <p class="arrowBox">{color.colorName}</p>
          </div>
        {/each}
      </div>
      <div class="text-white text-xl font-bold pt-7 pb-3">Material</div>
      <div class="grid grid-cols-6 gap-y-4">
        <!-- 텍스쳐 -->
        {#each shoeTexture as texture}
          <div>
            <div
              class="rounded-full bg-cover w-14 h-14"
              style={`background-image: url(${texture.backurl})`}
              on:click={() => setTexture(texture.backurl)}
            />
            <p class="arrowBox">{texture.texturename}</p>
          </div>
        {/each}
      </div>
    </div>
  {/if}
  <div class="p-7 mt-[20px]">
    <progress
      value={isAreaCustomized.filter((isCustomized) => isCustomized).length}
      max={areaNames.length}
    />
    <div class="flex justify-end mr-[6px]">
      <p class="text-white font-semibold">
        {isAreaCustomized.filter((isCustomized) => isCustomized).length} / {areaNames.length}
      </p>
    </div>
    <div class="mt-[15px]">
      <button
        class="relative border-2 border-white rounded-full w-48 h-14 mt-[20px] mb-[10px] text-white text-lg font-bold hover:bg-white hover:text-gray-500 ease-in duration-50"
        on:click={() => window.location.reload()}
      >
        다시하기
        <img
          src={"icon/arrow.svg"}
          alt="clockwise arrow icon"
          class="icon absolute top-3 right-3 w-8"
        />
      </button>
      <button
        class="payment relative border-2 border-white rounded-full w-48 h-14 text-white text-lg font-bold hover:bg-white hover:text-gray-500 ease-in duration-50"
      >
        저장하기
        <img
          src={"icon/heart.svg"}
          alt="heart icon"
          class="icon2 absolute top-3 right-3 w-8"
        />
      </button>
    </div>
    <button
      class="bg-gradient-to-r from-fuchsia-500 to-cyan-500 rounded-full w-96 h-14 text-white text-lg font-bold mt-3 hover:from-pink-500 hover:to-yellow-500 ease-in duration-300"
      on:click={toggleModal}>구매하기</button
    >
  </div>
</div>

<Modal {showModal} on:click={toggleModal}>
  <p class="text-xl font-semibold">주문 전 선택한 내용을 확인해주세요.</p>
  <p class="mb-5">
    반영한 내용들이 선택한 것과 같다면 주문 버튼을 클릭해주세요.
  </p>
  <button
    class="bg-gradient-to-r from-fuchsia-500 to-cyan-500 rounded-full w-72 h-14 text-white text-lg font-bold hover:from-pink-500 hover:to-yellow-500"
    >주문</button
  >
</Modal>

<style>
  /* 세로정렬 */
  .verticalText {
    writing-mode: vertical-rl;
  }

  /* 메뉴 리스트 */
  .menu input + label {
    cursor: pointer;
    padding-left: 25px;
    background-image: url("/icon/menu.svg");
    background-repeat: no-repeat;
    background-size: contain;
  }

  .menu input:checked + label {
    background-image: url("/icon/cancel.svg");
  }

  li {
    padding: 5px;
  }

  /* 진행바 */
  progress {
    width: 384px;
    margin: 10px 0 5px 0;
  }

  ::-webkit-progress-bar {
    background-color: lightgrey;
    border-radius: 10px;
  }

  ::-webkit-progress-value {
    background-color: white;
    border-radius: 10px;
  }

  /* 색상 팔레트 hover 말풍선 */
  .rounded-full:hover {
    cursor: pointer;
  }

  .arrowBox {
    display: none;
    position: absolute;
    min-width: 56px;
    width: fit-content;
    padding: 8px;
    border-radius: 20px;
    background: #fff;
    font-size: 11px;
    font-weight: bold;
    text-align: center;
    color: #000000;
  }

  .arrowBox:after {
    position: absolute;
    bottom: 100%;
    left: 50%;
    margin-left: -10px;
    border: solid transparent;
    border-bottom-color: #fff;
    border-width: 10px;
    pointer-events: none;
    content: " ";
  }

  .rounded-full:hover + p.arrowBox {
    display: block;
  }
</style>
