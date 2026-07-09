<script lang="ts">
  import { Canvas, T } from '@threlte/core';
  import { OrbitControls } from '@threlte/extras';
  import Room from './Room.svelte';
  import CameraController from './CameraController.svelte';

  interface Props {
    tvPower: boolean;
    tvChannel: number;
    canvasElement: HTMLCanvasElement | null;
    textureUpdateTrigger: number;
    cameraMode: 'free' | 'poem' | 'piano' | 'telephone' | 'tv';
    resetTrigger?: number;
    bookCanvasElement: HTMLCanvasElement | null;
    sheetCanvasElement: HTMLCanvasElement | null;
    bookTextureTrigger: number;
    sheetTextureTrigger: number;
    onBookClick?: () => void;
    onSheetClick?: () => void;
    onTelephoneClick?: () => void;
    onTVClick?: () => void;
    pianoLight: boolean;
    telephoneLight: boolean;
    tableLamp: boolean;
    couchSpotlight: boolean;
  }
  let { 
    tvPower, 
    tvChannel, 
    canvasElement, 
    textureUpdateTrigger, 
    cameraMode,
    resetTrigger = 0,
    bookCanvasElement,
    sheetCanvasElement,
    bookTextureTrigger,
    sheetTextureTrigger,
    onBookClick,
    onSheetClick,
    onTelephoneClick,
    onTVClick,
    pianoLight,
    telephoneLight,
    tableLamp,
    couchSpotlight
  }: Props = $props();

  let cameraPosition = $state<[number, number, number]>([0.3, 0.85, 1.3]);
  let cameraTarget = $state<[number, number, number]>([0.3, 0.29, 1.75]);

  let camera = $state<any>(null);
  let controls = $state<any>(null);

  // Adjust camera position and controls look-at target based on viewing mode
  $effect(() => {
    if (cameraMode === 'poem') {
      cameraPosition = [0.3, 0.85, 1.3];
      cameraTarget = [0.3, 0.29, 1.75];
    } else if (cameraMode === 'piano') {
      cameraPosition = [-2.9, 1.08, -1.7];
      cameraTarget = [-3.16, 0.92, -2.46];
    } else if (cameraMode === 'telephone') {
      // Inside the booth — eye height near the door side, looking at the phone shelf
      cameraPosition = [-2.65, 1.62, 2.74];
      cameraTarget = [-2.66, 0.9, 2.49];
    } else if (cameraMode === 'tv') {
      cameraPosition = [0.0, 0.95, 1.55];
      cameraTarget = [0.0, 0.95, -1.39];
    } else { // 'free'
      cameraPosition = [4.5, 3.8, 6.5];
      cameraTarget = [0, 0.5, 0.2];
    }
  });
</script>

<div class="threlte-canvas-holder">
  <Canvas shadows>
    <T.PerspectiveCamera
      makeDefault
      bind:ref={camera}
      fov={45}
      near={0.1}
      far={100}
    >
      <OrbitControls
        bind:ref={controls}
        enableDamping
        dampingFactor={0.05}
        minDistance={1.2}
        maxDistance={12}
        maxPolarAngle={Math.PI / 2 - 0.05}
      />
    </T.PerspectiveCamera>

    <!-- Global Room Lighting -->
    <T.AmbientLight intensity={0.4} />

    <!-- Shadows key light -->
    <T.DirectionalLight
      position={[4, 7, 5]}
      intensity={0.8}
      castShadow
      shadow.mapSize.width={1024}
      shadow.mapSize.height={1024}
      shadow.camera.near={0.5}
      shadow.camera.far={20}
      shadow.camera.left={-5}
      shadow.camera.right={5}
      shadow.camera.top={5}
      shadow.camera.bottom={-5}
    />

    <!-- Fill light -->
    <T.DirectionalLight
      position={[-4, 4, -4]}
      intensity={0.2}
    />

    <Room
      {tvPower}
      {tvChannel}
      {canvasElement}
      {textureUpdateTrigger}
      {bookCanvasElement}
      {sheetCanvasElement}
      {bookTextureTrigger}
      {sheetTextureTrigger}
      {onBookClick}
      {onSheetClick}
      {onTelephoneClick}
      {onTVClick}
      {pianoLight}
      {telephoneLight}
      {tableLamp}
      {couchSpotlight}
    />

    {#if camera && controls}
      <CameraController
        {camera}
        {controls}
        {cameraPosition}
        {cameraTarget}
        {cameraMode}
        {resetTrigger}
      />
    {/if}
  </Canvas>
</div>

<style>
  .threlte-canvas-holder {
    width: 100%;
    height: 100%;
    position: absolute;
    top: 0;
    left: 0;
    z-index: 1;
  }
</style>
