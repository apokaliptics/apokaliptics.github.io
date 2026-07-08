<script lang="ts">
  import { Canvas, T } from '@threlte/core';
  import { OrbitControls } from '@threlte/extras';
  import Room from './Room.svelte';

  interface Props {
    tvPower: boolean;
    tvChannel: number;
    canvasElement: HTMLCanvasElement | null;
    textureUpdateTrigger: number;
    cameraMode: 'room' | 'tv';
  }
  let { tvPower, tvChannel, canvasElement, textureUpdateTrigger, cameraMode }: Props = $props();

  let cameraPosition = $state<[number, number, number]>([5, 4, 7]);
  let cameraTarget = $state<[number, number, number]>([0, 0.5, 0]);

  // Adjust camera position and controls look-at target based on viewing mode
  $effect(() => {
    if (cameraMode === 'tv') {
      cameraPosition = [0, 0.75, -0.6];
      cameraTarget = [0, 0.75, -1.8];
    } else {
      cameraPosition = [4.5, 3.8, 6.5];
      cameraTarget = [0, 0.5, 0.2];
    }
  });
</script>

<div class="threlte-canvas-holder">
  <Canvas shadows>
    <T.PerspectiveCamera
      makeDefault
      position={cameraPosition}
      fov={45}
      near={0.1}
      far={100}
    >
      <OrbitControls
        enableDamping
        dampingFactor={0.05}
        target={cameraTarget}
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
    />
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
