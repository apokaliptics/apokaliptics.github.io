<script lang="ts">
  import { T, useTask } from '@threlte/core';
  import * as THREE from 'three';

  interface Props {
    tvPower: boolean;
    tvChannel: number;
    canvasElement: HTMLCanvasElement | null;
    textureUpdateTrigger: number;
  }
  let { tvPower, tvChannel, canvasElement, textureUpdateTrigger }: Props = $props();

  let canvasTexture = $state<THREE.CanvasTexture | null>(null);
  let tvScreenLightIntensity = $state(0);

  // Sync canvas texture creation and updates
  $effect(() => {
    if (canvasElement) {
      if (!canvasTexture) {
        canvasTexture = new THREE.CanvasTexture(canvasElement);
        canvasTexture.colorSpace = THREE.SRGBColorSpace;
      }
      canvasTexture.needsUpdate = true;
    }
  });

  // Re-upload texture when content frame updates
  $effect(() => {
    if (textureUpdateTrigger && canvasTexture) {
      canvasTexture.needsUpdate = true;
    }
  });

  // Rapid screen glow flicker for noise channels, stable glow for images
  useTask(() => {
    if (!tvPower) {
      tvScreenLightIntensity = 0;
      return;
    }
    const isStatic = [3, 7, 12].includes(tvChannel);
    if (isStatic) {
      tvScreenLightIntensity = 0.35 + Math.random() * 0.55;
    } else {
      tvScreenLightIntensity = 0.75;
    }
  });
</script>

<!-- Circular Platform -->
<T.Mesh position={[0, -0.1, 0]} receiveShadow>
  <T.CylinderGeometry args={[5, 5, 0.2, 64]} />
  <T.MeshStandardMaterial color={0x18181a} roughness={0.8} />
</T.Mesh>

<!-- Retro Velvet Couch -->
<!-- Seat Cushions -->
<T.Mesh position={[0, 0.2, 1.8]} castShadow receiveShadow>
  <T.BoxGeometry args={[2.4, 0.3, 1.0]} />
  <T.MeshStandardMaterial color={0x721c24} roughness={0.85} />
</T.Mesh>
<!-- Backrest -->
<T.Mesh position={[0, 0.65, 2.25]} castShadow receiveShadow>
  <T.BoxGeometry args={[2.4, 0.7, 0.3]} />
  <T.MeshStandardMaterial color={0x721c24} roughness={0.85} />
</T.Mesh>
<!-- Left Armrest -->
<T.Mesh position={[-1.2, 0.4, 1.8]} castShadow receiveShadow>
  <T.BoxGeometry args={[0.25, 0.6, 1.0]} />
  <T.MeshStandardMaterial color={0x721c24} roughness={0.85} />
</T.Mesh>
<!-- Right Armrest -->
<T.Mesh position={[1.2, 0.4, 1.8]} castShadow receiveShadow>
  <T.BoxGeometry args={[0.25, 0.6, 1.0]} />
  <T.MeshStandardMaterial color={0x721c24} roughness={0.85} />
</T.Mesh>

<!-- Console wooden TV Set -->
<!-- Stand -->
<T.Mesh position={[0, 0.1, -1.8]} castShadow>
  <T.CylinderGeometry args={[0.25, 0.25, 0.2, 16]} />
  <T.MeshStandardMaterial color={0x111111} roughness={0.6} />
</T.Mesh>
<!-- Wooden Cabinet -->
<T.Mesh position={[0, 0.75, -1.8]} castShadow receiveShadow>
  <T.BoxGeometry args={[1.7, 1.1, 0.75]} />
  <T.MeshStandardMaterial color={0x3e2723} roughness={0.6} />
</T.Mesh>
<!-- Front Charcoal Bezel -->
<T.Mesh position={[0, 0.75, -1.41]} castShadow>
  <T.BoxGeometry args={[1.6, 1.0, 0.03]} />
  <T.MeshStandardMaterial color={0x202020} roughness={0.65} />
</T.Mesh>

<!-- Screen Face (Active Canvas Texture) -->
{#if canvasTexture}
  <T.Mesh position={[-0.15, 0.75, -1.39]}>
    <T.BoxGeometry args={[1.1, 0.8, 0.02]} />
    <T.MeshBasicMaterial map={canvasTexture} />
  </T.Mesh>
{:else}
  <T.Mesh position={[-0.15, 0.75, -1.39]}>
    <T.BoxGeometry args={[1.1, 0.8, 0.02]} />
    <T.MeshBasicMaterial color={0x020202} />
  </T.Mesh>
{/if}

<!-- Curvature Bezel Gloss Overlay -->
<T.Mesh position={[-0.15, 0.75, -1.38]}>
  <T.BoxGeometry args={[1.105, 0.805, 0.015]} />
  <T.MeshStandardMaterial 
    color={0x000000} 
    transparent 
    opacity={0.12} 
    roughness={0.05} 
    metalness={0.9} 
  />
</T.Mesh>

<!-- Dials Control Bezel -->
<!-- Rotary Dial 1 -->
<T.Mesh position={[0.6, 0.9, -1.39]} rotation={[Math.PI / 2, 0, 0]}>
  <T.CylinderGeometry args={[0.07, 0.07, 0.04, 12]} />
  <T.MeshStandardMaterial color={0x2b2b2b} roughness={0.4} />
</T.Mesh>
<!-- Rotary Dial 2 -->
<T.Mesh position={[0.6, 0.72, -1.39]} rotation={[Math.PI / 2, 0, 0]}>
  <T.CylinderGeometry args={[0.07, 0.07, 0.04, 12]} />
  <T.MeshStandardMaterial color={0x2b2b2b} roughness={0.4} />
</T.Mesh>
<!-- Power Indicator Button -->
<T.Mesh position={[0.6, 0.5, -1.39]} rotation={[Math.PI / 2, 0, 0]}>
  <T.CylinderGeometry args={[0.04, 0.04, 0.05, 12]} />
  <T.MeshStandardMaterial color={tvPower ? 0xcc0000 : 0x333333} roughness={0.3} />
</T.Mesh>

<!-- Light Glow Projection from the TV Screen -->
{#if tvPower}
  <T.PointLight
    position={[0, 0.75, -1.2]}
    intensity={tvScreenLightIntensity}
    distance={5}
    decay={1.8}
    color={tvChannel === 11 ? 0x33ff33 : (tvChannel === 5 ? 0xffeb3b : 0xffffff)}
    castShadow
  />
{/if}
