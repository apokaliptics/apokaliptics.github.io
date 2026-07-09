<script lang="ts">
  import { T, useTask } from '@threlte/core';
  import { interactivity } from '@threlte/extras';
  import * as THREE from 'three';

  // Enable Raycasting pointer events inside Canvas subtree
  interactivity();

  interface Props {
    tvPower: boolean;
    tvChannel: number;
    canvasElement: HTMLCanvasElement | null;
    textureUpdateTrigger: number;
    bookCanvasElement: HTMLCanvasElement | null;
    sheetCanvasElement: HTMLCanvasElement | null;
    bookTextureTrigger: number;
    sheetTextureTrigger: number;
    onBookClick?: () => void;
    onSheetClick?: () => void;
    onTelephoneClick?: () => void;
    onTVClick?: () => void;
  }
  let { 
    tvPower, 
    tvChannel, 
    canvasElement, 
    textureUpdateTrigger, 
    bookCanvasElement, 
    sheetCanvasElement, 
    bookTextureTrigger, 
    sheetTextureTrigger,
    onBookClick,
    onSheetClick,
    onTelephoneClick,
    onTVClick
  }: Props = $props();

  let canvasTexture = $state<THREE.CanvasTexture | null>(null);
  let bookTexture = $state<THREE.CanvasTexture | null>(null);
  let sheetTexture = $state<THREE.CanvasTexture | null>(null);
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

  $effect(() => {
    if (bookCanvasElement) {
      if (!bookTexture) {
        bookTexture = new THREE.CanvasTexture(bookCanvasElement);
        bookTexture.colorSpace = THREE.SRGBColorSpace;
      }
      bookTexture.needsUpdate = true;
    }
  });

  $effect(() => {
    if (sheetCanvasElement) {
      if (!sheetTexture) {
        sheetTexture = new THREE.CanvasTexture(sheetCanvasElement);
        sheetTexture.colorSpace = THREE.SRGBColorSpace;
      }
      sheetTexture.needsUpdate = true;
    }
  });

  // Re-upload textures when content frames update
  $effect(() => {
    if (textureUpdateTrigger && canvasTexture) {
      canvasTexture.needsUpdate = true;
    }
  });

  $effect(() => {
    if (bookTextureTrigger && bookTexture) {
      bookTexture.needsUpdate = true;
    }
  });

  $effect(() => {
    if (sheetTextureTrigger && sheetTexture) {
      sheetTexture.needsUpdate = true;
    }
  });

  // Smoke particle system state
  interface SmokeParticle {
    x: number;
    y: number;
    z: number;
    scale: number;
    opacity: number;
    age: number;
  }

  let smokeParticles = $state<SmokeParticle[]>(
    Array.from({ length: 12 }, (_, i) => ({
      x: 0.025,
      y: 0.030,
      z: 0.019,
      scale: 0.005,
      opacity: 0,
      age: i * (2.0 / 12),
    }))
  );

  const tipLocalPos = { x: 0.025, y: 0.030, z: 0.019 };

  // Rapid screen glow flicker for noise channels, stable glow for images, and update smoke particles
  useTask((delta) => {
    // TV glow
    if (!tvPower) {
      tvScreenLightIntensity = 0;
    } else {
      const isStatic = [3, 7, 12].includes(tvChannel);
      tvScreenLightIntensity = isStatic ? (0.35 + Math.random() * 0.55) : 0.75;
    }

    // Smoke particles update
    const dt = delta || 0.016;
    const duration = 2.0; // 2 seconds particle lifetime

    for (let i = 0; i < smokeParticles.length; i++) {
      const p = smokeParticles[i];
      p.age += dt;

      if (p.age >= duration) {
        p.age = 0;
        p.x = tipLocalPos.x;
        p.y = tipLocalPos.y;
        p.z = tipLocalPos.z;
        p.scale = 0.003;
        p.opacity = 0.35 + Math.random() * 0.2;
      } else {
        const t = p.age / duration; // normalized life 0 to 1
        
        // Rise up
        p.y = tipLocalPos.y + t * 0.45;
        
        // Wavy drift using sine/cosine based on time and index
        const driftSpeed = 3.5;
        p.x = tipLocalPos.x + Math.sin(t * driftSpeed + i * 1.5) * 0.05;
        p.z = tipLocalPos.z + Math.cos(t * driftSpeed + i * 2.0) * 0.05;
        
        // Grow
        p.scale = 0.003 + t * 0.06;
        
        // Fade in / out
        if (t < 0.25) {
          p.opacity = (t / 0.25) * 0.55;
        } else {
          p.opacity = 0.55 * (1 - (t - 0.25) / 0.75);
        }
      }
    }
    // Trigger Svelte reactivity
    smokeParticles = [...smokeParticles];
  });
</script>

<!-- Circular Platform -->
<T.Mesh position={[0, -0.1, 0]} receiveShadow>
  <T.CylinderGeometry args={[5, 5, 0.2, 64]} />
  <T.MeshStandardMaterial color={0x18181a} roughness={0.8} />
</T.Mesh>

<!-- Retro Velvet Couch (Armchair size) -->
<T.Group
  onclick={(e: any) => { e.stopPropagation(); onBookClick?.(); }}
  onpointerenter={() => { document.body.style.cursor = 'pointer'; }}
  onpointerleave={() => { document.body.style.cursor = 'auto'; }}
>
  <!-- Seat Base -->
  <T.Mesh position={[0, 0.15, 1.8]} castShadow receiveShadow>
    <T.BoxGeometry args={[1.5, 0.25, 0.8]} />
    <T.MeshStandardMaterial color={0x721c24} roughness={0.85} />
  </T.Mesh>
  <!-- Backrest -->
  <T.Mesh position={[0, 0.55, 2.15]} castShadow receiveShadow>
    <T.BoxGeometry args={[1.9, 0.6, 0.2]} />
    <T.MeshStandardMaterial color={0x721c24} roughness={0.85} />
  </T.Mesh>
  <!-- Left Armrest -->
  <T.Mesh position={[-0.85, 0.3, 1.8]} castShadow receiveShadow>
    <T.BoxGeometry args={[0.2, 0.5, 0.8]} />
    <T.MeshStandardMaterial color={0x721c24} roughness={0.85} />
  </T.Mesh>
  <!-- Right Armrest -->
  <T.Mesh position={[0.85, 0.3, 1.8]} castShadow receiveShadow>
    <T.BoxGeometry args={[0.2, 0.5, 0.8]} />
    <T.MeshStandardMaterial color={0x721c24} roughness={0.85} />
  </T.Mesh>
</T.Group>

<!-- Discarded Satin Shirt with Pinhole Burns (scattered on floor near Couch) -->
<T.Group position={[-0.8, 0.0, 1.2]}>
  <!-- Shirt Fabric Main (On floor) -->
  <T.Mesh position={[0, 0.01, 0]} rotation={[0.05, 0.4, -0.02]} castShadow receiveShadow>
    <T.BoxGeometry args={[0.26, 0.02, 0.22]} />
    <T.MeshStandardMaterial color={0x1a365d} roughness={0.2} metalness={0.1} />
  </T.Mesh>
  <!-- Shirt Fabric Folded (Sleeve fold) -->
  <T.Mesh position={[0.12, 0.008, -0.06]} rotation={[-0.05, -0.2, 0.05]} castShadow receiveShadow>
    <T.BoxGeometry args={[0.22, 0.016, 0.12]} />
    <T.MeshStandardMaterial color={0x1a365d} roughness={0.2} metalness={0.1} />
  </T.Mesh>
  <!-- Shirt Fabric Folded (Other side) -->
  <T.Mesh position={[-0.1, 0.008, 0.08]} rotation={[0.1, 0.15, -0.1]} castShadow receiveShadow>
    <T.BoxGeometry args={[0.18, 0.018, 0.18]} />
    <T.MeshStandardMaterial color={0x1a365d} roughness={0.2} metalness={0.1} />
  </T.Mesh>
  
  <!-- Pinhole Cigarette Burns (tiny flat black circles on chest/top box) -->
  <T.Mesh position={[0.04, 0.021, 0.02]} rotation={[-Math.PI / 2, 0.4, 0]}>
    <T.CircleGeometry args={[0.008, 8]} />
    <T.MeshBasicMaterial color={0x000000} />
  </T.Mesh>
  <T.Mesh position={[-0.06, 0.021, -0.03]} rotation={[-Math.PI / 2, 0.4, 0]}>
    <T.CircleGeometry args={[0.006, 8]} />
    <T.MeshBasicMaterial color={0x000000} />
  </T.Mesh>
</T.Group>

<!-- Retro Side Table -->
<!-- Table Leg -->
<T.Mesh position={[-1.3, 0.175, 1.8]} castShadow>
  <T.CylinderGeometry args={[0.03, 0.03, 0.35, 12]} />
  <T.MeshStandardMaterial color={0x151515} roughness={0.6} />
</T.Mesh>
<!-- Table Base -->
<T.Mesh position={[-1.3, 0.01, 1.8]} receiveShadow>
  <T.CylinderGeometry args={[0.18, 0.18, 0.02, 16]} />
  <T.MeshStandardMaterial color={0x3e2723} roughness={0.6} />
</T.Mesh>
<!-- Table Top -->
<T.Mesh position={[-1.3, 0.37, 1.8]} castShadow receiveShadow>
  <T.BoxGeometry args={[0.5, 0.04, 0.5]} />
  <T.MeshStandardMaterial color={0x3e2723} roughness={0.6} />
</T.Mesh>

<!-- Table Lamp -->
<!-- Lamp Base -->
<T.Mesh position={[-1.3, 0.41, 1.8]} castShadow>
  <T.CylinderGeometry args={[0.07, 0.07, 0.04, 12]} />
  <T.MeshStandardMaterial color={0xd4af37} metalness={0.8} roughness={0.2} />
</T.Mesh>

<!-- Silver Spoon on Side Table -->
<T.Group position={[-1.22, 0.39, 1.92]} rotation={[0, -Math.PI / 5, 0.05]}>
  <!-- Spoon Bowl -->
  <T.Mesh position={[0, 0.003, 0.04]} scale={[1, 0.25, 1.4]} castShadow>
    <T.SphereGeometry args={[0.02, 8, 8]} />
    <!-- Scale to flatten it into an oval spoon scoop -->
    <T.MeshStandardMaterial color={0xdcdcdc} metalness={0.95} roughness={0.15} />
  </T.Mesh>
  <!-- Spoon Handle -->
  <T.Mesh position={[0, 0.005, -0.01]} castShadow>
    <T.BoxGeometry args={[0.006, 0.003, 0.08]} />
    <T.MeshStandardMaterial color={0xdcdcdc} metalness={0.95} roughness={0.15} />
  </T.Mesh>
  
  <!-- Spoon Chain (interconnected torus rings) -->
  <T.Group position={[0, 0.005, -0.05]}>
    <!-- Ring 1 -->
    <T.Mesh position={[0, -0.002, 0]} rotation={[Math.PI / 2, 0, 0]} castShadow>
      <T.TorusGeometry args={[0.012, 0.003, 8, 16]} />
      <T.MeshStandardMaterial color={0xdcdcdc} metalness={0.95} roughness={0.15} />
    </T.Mesh>
    <!-- Ring 2 -->
    <T.Mesh position={[0, -0.004, -0.018]} rotation={[0, Math.PI / 2, 0]} castShadow>
      <T.TorusGeometry args={[0.012, 0.003, 8, 16]} />
      <T.MeshStandardMaterial color={0xdcdcdc} metalness={0.95} roughness={0.15} />
    </T.Mesh>
    <!-- Ring 3 -->
    <T.Mesh position={[0.005, -0.006, -0.036]} rotation={[Math.PI / 2, 0, Math.PI / 4]} castShadow>
      <T.TorusGeometry args={[0.012, 0.003, 8, 16]} />
      <T.MeshStandardMaterial color={0xdcdcdc} metalness={0.95} roughness={0.15} />
    </T.Mesh>
    <!-- Ring 4 (spilling onto tabletop) -->
    <T.Mesh position={[0.015, -0.008, -0.054]} rotation={[0, Math.PI / 4, 0]} castShadow>
      <T.TorusGeometry args={[0.012, 0.003, 8, 16]} />
      <T.MeshStandardMaterial color={0xdcdcdc} metalness={0.95} roughness={0.15} />
    </T.Mesh>
  </T.Group>
</T.Group>
<!-- Lamp Stem -->
<T.Mesh position={[-1.3, 0.54, 1.8]} castShadow>
  <T.CylinderGeometry args={[0.015, 0.015, 0.25, 8]} />
  <T.MeshStandardMaterial color={0xd4af37} metalness={0.8} roughness={0.2} />
</T.Mesh>
<!-- Lamp Shade -->
<T.Mesh position={[-1.3, 0.7, 1.8]} castShadow>
  <T.ConeGeometry args={[0.15, 0.18, 12]} />
  <T.MeshStandardMaterial color={0xfaf0e6} roughness={0.9} />
</T.Mesh>

<!-- Warm lamp light projection -->
<T.PointLight
  position={[-1.3, 0.65, 1.8]}
  intensity={0.65}
  distance={4}
  decay={1.5}
  color={0xffa726}
  castShadow
/>

<!-- Baggy of Weed next to Ashtray -->
<T.Group position={[-1.28, 0.392, 1.68]} rotation={[0, Math.PI / 6, 0]}>
  <!-- Baggy (Outer Box) -->
  <T.Mesh castShadow receiveShadow>
    <T.BoxGeometry args={[0.07, 0.005, 0.07]} />
    <T.MeshStandardMaterial color={0xffffff} roughness={0.1} transparent opacity={0.6} />
  </T.Mesh>
  <!-- Weed (Inner Box) -->
  <T.Mesh position={[0, -0.0005, 0]}>
    <T.BoxGeometry args={[0.045, 0.0035, 0.045]} />
    <T.MeshStandardMaterial color={0x33691e} roughness={0.9} />
  </T.Mesh>
</T.Group>

<!-- Ashtray and Smoking Cigarette on Side Table -->
<T.Group position={[-1.42, 0.39, 1.70]}>
  <!-- Ashtray Glass Base -->
  <T.Mesh position={[0, 0.004, 0]} castShadow receiveShadow>
    <T.CylinderGeometry args={[0.055, 0.05, 0.008, 16]} />
    <T.MeshStandardMaterial color={0x2d5a27} roughness={0.15} metalness={0.1} transparent opacity={0.6} />
  </T.Mesh>
  
  <!-- Ashtray Glass Rim -->
  <T.Mesh position={[0, 0.012, 0]} rotation={[Math.PI / 2, 0, 0]} castShadow>
    <T.TorusGeometry args={[0.046, 0.008, 8, 24]} />
    <T.MeshStandardMaterial color={0x2d5a27} roughness={0.15} metalness={0.1} transparent opacity={0.6} />
  </T.Mesh>

  <!-- Cigarette resting on the rim -->
  <T.Group position={[0.022, 0.012, 0.022]} rotation={[0.25, Math.PI * 0.75, 0]}>
    <!-- Filter (Brown/Orange) -->
    <T.Mesh position={[0, -0.012, 0]} castShadow>
      <T.CylinderGeometry args={[0.003, 0.003, 0.012, 8]} />
      <T.MeshStandardMaterial color={0xc68e4f} roughness={0.6} />
    </T.Mesh>
    
    <!-- Cigarette Paper (White) -->
    <T.Mesh position={[0, 0.006, 0]} castShadow>
      <T.CylinderGeometry args={[0.003, 0.003, 0.024, 8]} />
      <T.MeshStandardMaterial color={0xeeeeee} roughness={0.5} />
    </T.Mesh>
    
    <!-- Glowing Cherry / Ash Tip -->
    <T.Mesh position={[0, 0.019, 0]} castShadow>
      <T.SphereGeometry args={[0.0032, 8, 8]} />
      <T.MeshStandardMaterial color={0xff3300} emissive={0xff3300} emissiveIntensity={3} roughness={0.2} />
    </T.Mesh>
    
    <!-- Small Point Light for the glowing cherry -->
    <T.PointLight
      position={[0, 0.02, 0]}
      intensity={0.15}
      distance={0.3}
      decay={2}
      color={0xff5500}
    />
  </T.Group>

  <!-- Smoke Particles (rising straight up in Ashtray's coordinate space) -->
  {#each smokeParticles as p, idx (idx)}
    {#if p.opacity > 0}
      <T.Mesh position={[p.x, p.y, p.z]}>
        <T.SphereGeometry args={[p.scale, 8, 8]} />
        <T.MeshBasicMaterial
          color={0xdddddd}
          transparent
          opacity={p.opacity}
          depthWrite={false}
        />
      </T.Mesh>
    {/if}
  {/each}
</T.Group>

<!-- Dog Bowl & Bone (on floor near TV Credenza) -->
<T.Group position={[-0.6, 0.0, -1.3]}>
  <!-- Dog Bowl Base -->
  <T.Mesh position={[0, 0.004, 0]} castShadow receiveShadow>
    <T.CylinderGeometry args={[0.08, 0.075, 0.008, 16]} />
    <T.MeshStandardMaterial color={0x2c3e50} roughness={0.4} />
  </T.Mesh>
  <!-- Dog Bowl Rim -->
  <T.Mesh position={[0, 0.018, 0]} rotation={[Math.PI / 2, 0, 0]} castShadow>
    <T.TorusGeometry args={[0.068, 0.012, 8, 24]} />
    <T.MeshStandardMaterial color={0x2c3e50} roughness={0.4} />
  </T.Mesh>
  <!-- Dog Food (Brown Kibble cylinder inside) -->
  <T.Mesh position={[0, 0.01, 0]} castShadow>
    <T.CylinderGeometry args={[0.062, 0.062, 0.012, 16]} />
    <T.MeshStandardMaterial color={0x5d4037} roughness={0.9} />
  </T.Mesh>
  
  <!-- Bone lying flat on the floor -->
  <T.Group position={[0.15, 0.007, 0.05]} rotation={[0, Math.PI / 3, 0]}>
    <!-- Central Shaft (lying along Z-axis) -->
    <T.Mesh rotation={[Math.PI / 2, 0, 0]} castShadow>
      <T.CylinderGeometry args={[0.006, 0.006, 0.04, 8]} />
      <T.MeshStandardMaterial color={0xffffff} roughness={0.8} />
    </T.Mesh>
    <!-- End 1 Spheres (at Z = 0.02) -->
    <T.Mesh position={[-0.006, 0, 0.02]} castShadow>
      <T.SphereGeometry args={[0.007, 8, 8]} />
      <T.MeshStandardMaterial color={0xffffff} roughness={0.8} />
    </T.Mesh>
    <T.Mesh position={[0.006, 0, 0.02]} castShadow>
      <T.SphereGeometry args={[0.007, 8, 8]} />
      <T.MeshStandardMaterial color={0xffffff} roughness={0.8} />
    </T.Mesh>
    <!-- End 2 Spheres (at Z = -0.02) -->
    <T.Mesh position={[-0.006, 0, -0.02]} castShadow>
      <T.SphereGeometry args={[0.007, 8, 8]} />
      <T.MeshStandardMaterial color={0xffffff} roughness={0.8} />
    </T.Mesh>
    <T.Mesh position={[0.006, 0, -0.02]} castShadow>
      <T.SphereGeometry args={[0.007, 8, 8]} />
      <T.MeshStandardMaterial color={0xffffff} roughness={0.8} />
    </T.Mesh>
  </T.Group>
</T.Group>

<!-- Premium Wooden TV Table / Credenza -->
<!-- Legs -->
<T.Mesh position={[-1.0, 0.05, -1.45]} castShadow>
  <T.CylinderGeometry args={[0.04, 0.04, 0.1, 8]} />
  <T.MeshStandardMaterial color={0x151515} roughness={0.6} />
</T.Mesh>
<T.Mesh position={[1.0, 0.05, -1.45]} castShadow>
  <T.CylinderGeometry args={[0.04, 0.04, 0.1, 8]} />
  <T.MeshStandardMaterial color={0x151515} roughness={0.6} />
</T.Mesh>
<T.Mesh position={[-1.0, 0.05, -2.15]} castShadow>
  <T.CylinderGeometry args={[0.04, 0.04, 0.1, 8]} />
  <T.MeshStandardMaterial color={0x151515} roughness={0.6} />
</T.Mesh>
<T.Mesh position={[1.0, 0.05, -2.15]} castShadow>
  <T.CylinderGeometry args={[0.04, 0.04, 0.1, 8]} />
  <T.MeshStandardMaterial color={0x151515} roughness={0.6} />
</T.Mesh>
<!-- Table Body -->
<T.Mesh position={[0, 0.275, -1.8]} castShadow receiveShadow>
  <T.BoxGeometry args={[2.2, 0.35, 0.9]} />
  <T.MeshStandardMaterial color={0x3e2723} roughness={0.65} />
</T.Mesh>

<!-- Console wooden TV Set -->
<!-- Wooden Cabinet -->
<T.Mesh position={[0, 1.0, -1.8]} castShadow receiveShadow>
  <T.BoxGeometry args={[1.7, 1.1, 0.75]} />
  <T.MeshStandardMaterial color={0x3e2723} roughness={0.6} />
</T.Mesh>
<!-- Front Charcoal Bezel -->
<T.Mesh position={[0, 1.0, -1.41]} castShadow>
  <T.BoxGeometry args={[1.6, 1.0, 0.03]} />
  <T.MeshStandardMaterial color={0x202020} roughness={0.65} />
</T.Mesh>

<!-- Screen Face (Active Canvas Texture) -->
{#if canvasTexture}
  <T.Mesh 
    position={[-0.15, 1.0, -1.39]}
    onclick={(e: any) => { e.stopPropagation(); onTVClick?.(); }}
    onpointerenter={() => { document.body.style.cursor = 'pointer'; }}
    onpointerleave={() => { document.body.style.cursor = 'auto'; }}
  >
    <T.BoxGeometry args={[1.1, 0.8, 0.02]} />
    <T.MeshBasicMaterial map={canvasTexture} />
  </T.Mesh>
{:else}
  <T.Mesh 
    position={[-0.15, 1.0, -1.39]}
    onclick={(e: any) => { e.stopPropagation(); onTVClick?.(); }}
    onpointerenter={() => { document.body.style.cursor = 'pointer'; }}
    onpointerleave={() => { document.body.style.cursor = 'auto'; }}
  >
    <T.BoxGeometry args={[1.1, 0.8, 0.02]} />
    <T.MeshBasicMaterial color={0x020202} />
  </T.Mesh>
{/if}

<!-- Curvature Bezel Gloss Overlay -->
<T.Mesh position={[-0.15, 1.0, -1.38]}>
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
<T.Mesh position={[0.6, 1.15, -1.39]} rotation={[Math.PI / 2, 0, 0]}>
  <T.CylinderGeometry args={[0.07, 0.07, 0.04, 12]} />
  <T.MeshStandardMaterial color={0x2b2b2b} roughness={0.4} />
</T.Mesh>
<!-- Rotary Dial 2 -->
<T.Mesh position={[0.6, 0.97, -1.39]} rotation={[Math.PI / 2, 0, 0]}>
  <T.CylinderGeometry args={[0.07, 0.07, 0.04, 12]} />
  <T.MeshStandardMaterial color={0x2b2b2b} roughness={0.4} />
</T.Mesh>
<!-- Power Indicator Button -->
<T.Mesh position={[0.6, 0.75, -1.39]} rotation={[Math.PI / 2, 0, 0]}>
  <T.CylinderGeometry args={[0.04, 0.04, 0.05, 12]} />
  <T.MeshStandardMaterial color={tvPower ? 0xcc0000 : 0x333333} roughness={0.3} />
</T.Mesh>

<!-- Light Glow Projection from the TV Screen -->
{#if tvPower}
  <T.PointLight
    position={[0, 1.0, -1.2]}
    intensity={tvScreenLightIntensity}
    distance={5}
    decay={1.8}
    color={tvChannel === 11 ? 0x33ff33 : (tvChannel === 5 ? 0xffeb3b : 0xffffff)}
    castShadow
  />
{/if}

<!-- Grand Piano in the Outfar -->
<T.Group
  position={[-3.2, 0, -2.5]}
  rotation={[0, Math.PI / 4, 0]}
  onclick={(e: any) => { e.stopPropagation(); onSheetClick?.(); }}
  onpointerenter={() => { document.body.style.cursor = 'pointer'; }}
  onpointerleave={() => { document.body.style.cursor = 'auto'; }}
>
  <!-- Spotlight from above -->
  <T.SpotLight
    position={[0, 4.5, 0]}
    target.position={[0, 0.5, 0]}
    intensity={3.5}
    angle={Math.PI / 7}
    penumbra={0.6}
    distance={6}
    decay={1.2}
    color={0xfffad0}
    castShadow
  />

  <!-- Volumetric Spotlight Cone -->
  <T.Mesh position={[0, 2.15, 0]} rotation={[0, 0, 0]}>
    <T.CylinderGeometry args={[0.08, 1.2, 4.3, 32, 1, true]} />
    <T.MeshBasicMaterial
      color={0xfffad0}
      transparent
      opacity={0.08}
      depthWrite={false}
      side={THREE.DoubleSide}
    />
  </T.Mesh>

  <!-- Piano Legs -->
  <!-- Leg 1 (Front-Left) -->
  <T.Mesh position={[-0.65, 0.3, 0.4]} castShadow>
    <T.CylinderGeometry args={[0.04, 0.02, 0.6, 12]} />
    <T.MeshStandardMaterial color={0x151515} roughness={0.7} />
  </T.Mesh>
  <!-- Leg 2 (Front-Right) -->
  <T.Mesh position={[0.65, 0.3, 0.4]} castShadow>
    <T.CylinderGeometry args={[0.04, 0.02, 0.6, 12]} />
    <T.MeshStandardMaterial color={0x151515} roughness={0.7} />
  </T.Mesh>
  <!-- Leg 3 (Back-Tail) -->
  <T.Mesh position={[0.0, 0.3, -0.8]} castShadow>
    <T.CylinderGeometry args={[0.04, 0.02, 0.6, 12]} />
    <T.MeshStandardMaterial color={0x151515} roughness={0.7} />
  </T.Mesh>

  <!-- Piano Body Base -->
  <!-- Main keyboard block -->
  <T.Mesh position={[0, 0.68, 0.25]} castShadow receiveShadow>
    <T.BoxGeometry args={[1.5, 0.16, 0.5]} />
    <T.MeshStandardMaterial color={0x121212} roughness={0.15} metalness={0.1} />
  </T.Mesh>
  <!-- Curved tail block -->
  <T.Mesh position={[-0.1, 0.68, -0.3]} castShadow receiveShadow>
    <T.BoxGeometry args={[1.3, 0.16, 0.7]} />
    <T.MeshStandardMaterial color={0x121212} roughness={0.15} metalness={0.1} />
  </T.Mesh>
  <!-- Tail end block -->
  <T.Mesh position={[0.05, 0.68, -0.75]} castShadow receiveShadow>
    <T.BoxGeometry args={[0.9, 0.16, 0.4]} />
    <T.MeshStandardMaterial color={0x121212} roughness={0.15} metalness={0.1} />
  </T.Mesh>

  <!-- Keys (Keyboard Bed) -->
  <T.Mesh position={[0, 0.77, 0.51]} castShadow receiveShadow>
    <T.BoxGeometry args={[1.4, 0.02, 0.09]} />
    <T.MeshStandardMaterial color={0xfdfdfd} roughness={0.2} />
  </T.Mesh>
  
  <!-- Black Keys -->
  {#each Array.from({ length: 12 }, (_, i) => i) as i (i)}
    {@const keyX = -0.6 + i * 0.11}
    {#if i % 7 !== 2 && i % 7 !== 6}
      <T.Mesh position={[keyX, 0.785, 0.485]} castShadow>
        <T.BoxGeometry args={[0.015, 0.015, 0.05]} />
        <T.MeshStandardMaterial color={0x000000} roughness={0.1} />
      </T.Mesh>
    {/if}
  {/each}

  <!-- Piano Lid (Open) -->
  <T.Mesh position={[0.3, 1.05, -0.2]} rotation={[0, 0, -Math.PI / 6]} castShadow>
    <T.BoxGeometry args={[1.2, 0.02, 1.0]} />
    <T.MeshStandardMaterial color={0x121212} roughness={0.15} metalness={0.1} />
  </T.Mesh>
  <!-- Prop Stick -->
  <T.Mesh position={[0.3, 0.88, -0.1]} rotation={[0, 0, Math.PI / 12]} castShadow>
    <T.CylinderGeometry args={[0.008, 0.008, 0.38, 8]} />
    <T.MeshStandardMaterial color={0x1a1a1a} roughness={0.8} />
  </T.Mesh>

  <!-- Music Stand -->
  <T.Mesh position={[0, 0.85, 0.05]} rotation={[-Math.PI / 6, 0, 0]} castShadow>
    <T.BoxGeometry args={[0.5, 0.16, 0.015]} />
    <T.MeshStandardMaterial color={0x121212} roughness={0.15} />
  </T.Mesh>

  <!-- Music Sheet Page with texture contents -->
  {#if sheetTexture}
    <T.Mesh 
      position={[0, 0.86, 0.058]} 
      rotation={[-Math.PI / 6, 0, 0]} 
      castShadow
      onclick={(e: any) => { e.stopPropagation(); onSheetClick?.(); }}
      onpointerenter={() => { document.body.style.cursor = 'pointer'; }}
      onpointerleave={() => { document.body.style.cursor = 'auto'; }}
    >
      <T.BoxGeometry args={[0.42, 0.14, 0.004]} />
      <T.MeshBasicMaterial map={sheetTexture} />
    </T.Mesh>
  {:else}
    <T.Mesh 
      position={[0, 0.86, 0.058]} 
      rotation={[-Math.PI / 6, 0, 0]} 
      castShadow
      onclick={(e: any) => { e.stopPropagation(); onSheetClick?.(); }}
      onpointerenter={() => { document.body.style.cursor = 'pointer'; }}
      onpointerleave={() => { document.body.style.cursor = 'auto'; }}
    >
      <T.BoxGeometry args={[0.42, 0.14, 0.004]} />
      <T.MeshStandardMaterial color={0xfbfaf0} roughness={0.6} />
    </T.Mesh>
  {/if}
</T.Group>

<!-- "Nobody Home" Props -->

<!--
  ══════════════════════════════════════════════════════════════
  BRITISH K6 RED TELEPHONE BOX — Premium Procedural Model
  Box group centre: [-2.6, 0, 2.55], rotation Y = PI/8
  Interior camera target: [-2.68, 0.9, 2.55] (the phone shelf)
  
  Layout (local space):
    X: -0.44 … +0.44  (door at +Z face)
    Y:  0.00 … 3.10
    Z: -0.44 … +0.44
  Faces:
    +Z = front door face
    -Z = solid back wall
    +X = left side (window)
    -X = right side (window)
  ══════════════════════════════════════════════════════════════
-->
<T.Group
  position={[-2.6, 0, 2.55]}
  rotation={[0, Math.PI / 8, 0]}
  onclick={(e: any) => { e.stopPropagation(); onTelephoneClick?.(); }}
  onpointerenter={() => { document.body.style.cursor = 'pointer'; }}
  onpointerleave={() => { document.body.style.cursor = 'auto'; }}
>

  <!-- ══════════════════════════════════════════ -->
  <!-- PLINTH BASE — 3 steps, dark arterial red   -->
  <!-- ══════════════════════════════════════════ -->
  <T.Mesh position={[0, 0.035, 0]} receiveShadow castShadow>
    <T.BoxGeometry args={[1.02, 0.07, 1.02]} />
    <T.MeshStandardMaterial color={0x6b0000} roughness={0.78} />
  </T.Mesh>
  <T.Mesh position={[0, 0.085, 0]} receiveShadow castShadow>
    <T.BoxGeometry args={[0.94, 0.04, 0.94]} />
    <T.MeshStandardMaterial color={0x7a0000} roughness={0.75} />
  </T.Mesh>
  <T.Mesh position={[0, 0.115, 0]} receiveShadow castShadow>
    <T.BoxGeometry args={[0.88, 0.02, 0.88]} />
    <T.MeshStandardMaterial color={0x880000} roughness={0.72} />
  </T.Mesh>

  <!-- ══════════════════════════════════════════ -->
  <!-- FOUR ROUND CAST-IRON CORNER COLUMNS        -->
  <!-- ══════════════════════════════════════════ -->
  {#each [[-0.4, 0.4], [0.4, 0.4], [-0.4, -0.4], [0.4, -0.4]] as [cx, cz] (`${cx},${cz}`)}
    <!-- Main column shaft -->
    <T.Mesh position={[cx, 1.37, cz]} castShadow receiveShadow>
      <T.CylinderGeometry args={[0.045, 0.052, 2.5, 14]} />
      <T.MeshStandardMaterial color={0xcc0000} roughness={0.52} metalness={0.08} />
    </T.Mesh>
    <!-- Capital (top flare) -->
    <T.Mesh position={[cx, 2.63, cz]} castShadow>
      <T.CylinderGeometry args={[0.07, 0.045, 0.12, 14]} />
      <T.MeshStandardMaterial color={0xbb0000} roughness={0.48} metalness={0.1} />
    </T.Mesh>
    <!-- Base flare -->
    <T.Mesh position={[cx, 0.17, cz]} castShadow>
      <T.CylinderGeometry args={[0.062, 0.072, 0.1, 14]} />
      <T.MeshStandardMaterial color={0xbb0000} roughness={0.52} metalness={0.08} />
    </T.Mesh>
  {/each}

  <!-- ══════════════════════════════════════════════════════════ -->
  <!-- STRUCTURAL HORIZONTAL RAILS (top + mid + sill)            -->
  <!--   These are the horizontal red bars spanning between cols  -->
  <!-- ══════════════════════════════════════════════════════════ -->
  <!-- Top rail — front/back -->
  <T.Mesh position={[0, 2.59, 0.4]} castShadow>
    <T.BoxGeometry args={[0.81, 0.06, 0.04]} />
    <T.MeshStandardMaterial color={0xcc0000} roughness={0.5} />
  </T.Mesh>
  <T.Mesh position={[0, 2.59, -0.4]} castShadow>
    <T.BoxGeometry args={[0.81, 0.06, 0.04]} />
    <T.MeshStandardMaterial color={0xcc0000} roughness={0.5} />
  </T.Mesh>
  <!-- Top rail — left/right -->
  <T.Mesh position={[-0.4, 2.59, 0]} castShadow>
    <T.BoxGeometry args={[0.04, 0.06, 0.81]} />
    <T.MeshStandardMaterial color={0xcc0000} roughness={0.5} />
  </T.Mesh>
  <T.Mesh position={[0.4, 2.59, 0]} castShadow>
    <T.BoxGeometry args={[0.04, 0.06, 0.81]} />
    <T.MeshStandardMaterial color={0xcc0000} roughness={0.5} />
  </T.Mesh>
  <!-- Sill rail (window bottom) — front/back -->
  <T.Mesh position={[0, 0.15, 0.4]} castShadow>
    <T.BoxGeometry args={[0.81, 0.04, 0.04]} />
    <T.MeshStandardMaterial color={0xcc0000} roughness={0.5} />
  </T.Mesh>
  <T.Mesh position={[0, 0.15, -0.4]} castShadow>
    <T.BoxGeometry args={[0.81, 0.04, 0.04]} />
    <T.MeshStandardMaterial color={0xcc0000} roughness={0.5} />
  </T.Mesh>
  <!-- Sill rail — left/right -->
  <T.Mesh position={[-0.4, 0.15, 0]} castShadow>
    <T.BoxGeometry args={[0.04, 0.04, 0.81]} />
    <T.MeshStandardMaterial color={0xcc0000} roughness={0.5} />
  </T.Mesh>
  <T.Mesh position={[0.4, 0.15, 0]} castShadow>
    <T.BoxGeometry args={[0.04, 0.04, 0.81]} />
    <T.MeshStandardMaterial color={0xcc0000} roughness={0.5} />
  </T.Mesh>

  <!-- ══════════════════════════════════════════════════════════ -->
  <!-- LOUVERED VENTILATION BAND (solid red, below crown)         -->
  <!--   Spans all 4 sides above the glass zone, Y 2.15-2.5       -->
  <!-- ══════════════════════════════════════════════════════════ -->
  <T.Mesh position={[0, 2.32, 0.405]} castShadow>
    <T.BoxGeometry args={[0.81, 0.34, 0.03]} />
    <T.MeshStandardMaterial color={0xcc0000} roughness={0.55} />
  </T.Mesh>
  <!-- Louver slots front (3 narrow cutout impressions via darker thin boxes) -->
  {#each [-0.22, -0.08, 0.06, 0.2] as lx (lx)}
    <T.Mesh position={[lx, 2.32, 0.42]}>
      <T.BoxGeometry args={[0.065, 0.28, 0.008]} />
      <T.MeshStandardMaterial color={0x880000} roughness={0.7} />
    </T.Mesh>
  {/each}

  <T.Mesh position={[0, 2.32, -0.405]} castShadow>
    <T.BoxGeometry args={[0.81, 0.34, 0.03]} />
    <T.MeshStandardMaterial color={0xcc0000} roughness={0.55} />
  </T.Mesh>
  {#each [-0.22, -0.08, 0.06, 0.2] as lx (lx)}
    <T.Mesh position={[lx, 2.32, -0.42]}>
      <T.BoxGeometry args={[0.065, 0.28, 0.008]} />
      <T.MeshStandardMaterial color={0x880000} roughness={0.7} />
    </T.Mesh>
  {/each}

  <T.Mesh position={[-0.405, 2.32, 0]} castShadow>
    <T.BoxGeometry args={[0.03, 0.34, 0.81]} />
    <T.MeshStandardMaterial color={0xcc0000} roughness={0.55} />
  </T.Mesh>
  <T.Mesh position={[0.405, 2.32, 0]} castShadow>
    <T.BoxGeometry args={[0.03, 0.34, 0.81]} />
    <T.MeshStandardMaterial color={0xcc0000} roughness={0.55} />
  </T.Mesh>

  <!-- ══════════════════════════════════════════════════════════ -->
  <!-- "TELEPHONE" EMISSIVE TEXT PANEL — top of each window face  -->
  <!--   Cream/white emissive rectangle, Y ~2.05 (above glass)    -->
  <!-- ══════════════════════════════════════════════════════════ -->
  <!-- Front face text panel -->
  <T.Mesh position={[0, 2.08, 0.415]} castShadow>
    <T.BoxGeometry args={[0.62, 0.085, 0.018]} />
    <T.MeshStandardMaterial color={0xf5f0e0} roughness={0.6} />
  </T.Mesh>
  <!-- Emissive inner glow strip -->
  <T.Mesh position={[0, 2.08, 0.416]}>
    <T.BoxGeometry args={[0.56, 0.055, 0.01]} />
    <T.MeshStandardMaterial color={0xfff8dc} emissive={0xfff8dc} emissiveIntensity={0.55} roughness={0.3} />
  </T.Mesh>
  <!-- Back face text panel -->
  <T.Mesh position={[0, 2.08, -0.415]} castShadow>
    <T.BoxGeometry args={[0.62, 0.085, 0.018]} />
    <T.MeshStandardMaterial color={0xf5f0e0} roughness={0.6} />
  </T.Mesh>
  <T.Mesh position={[0, 2.08, -0.416]}>
    <T.BoxGeometry args={[0.56, 0.055, 0.01]} />
    <T.MeshStandardMaterial color={0xfff8dc} emissive={0xfff8dc} emissiveIntensity={0.55} roughness={0.3} />
  </T.Mesh>
  <!-- Right face text panel -->
  <T.Mesh position={[0.415, 2.08, 0]} castShadow>
    <T.BoxGeometry args={[0.018, 0.085, 0.62]} />
    <T.MeshStandardMaterial color={0xf5f0e0} roughness={0.6} />
  </T.Mesh>
  <T.Mesh position={[0.416, 2.08, 0]}>
    <T.BoxGeometry args={[0.01, 0.055, 0.56]} />
    <T.MeshStandardMaterial color={0xfff8dc} emissive={0xfff8dc} emissiveIntensity={0.55} roughness={0.3} />
  </T.Mesh>

  <!-- ══════════════════════════════════════════════════════════ -->
  <!-- ROYAL CROWN RELIEF PANELS — above text strip, per face     -->
  <!--   A raised red plaque with a subtle silver crown motif      -->
  <!-- ══════════════════════════════════════════════════════════ -->
  <!-- Front crown plaque -->
  <T.Mesh position={[0, 2.17, 0.415]} castShadow>
    <T.BoxGeometry args={[0.18, 0.055, 0.02]} />
    <T.MeshStandardMaterial color={0xdd0000} roughness={0.45} />
  </T.Mesh>
  <!-- Crown arcs (3 small cylinders) -->
  {#each [-0.055, 0, 0.055] as cx (cx)}
    <T.Mesh position={[cx, 2.18, 0.427]} rotation={[Math.PI / 2, 0, 0]} castShadow>
      <T.CylinderGeometry args={[0.014, 0.014, 0.012, 8]} />
      <T.MeshStandardMaterial color={0xddddaa} metalness={0.6} roughness={0.3} />
    </T.Mesh>
  {/each}
  <!-- Back crown plaque -->
  <T.Mesh position={[0, 2.17, -0.415]} castShadow>
    <T.BoxGeometry args={[0.18, 0.055, 0.02]} />
    <T.MeshStandardMaterial color={0xdd0000} roughness={0.45} />
  </T.Mesh>
  {#each [-0.055, 0, 0.055] as cx (cx)}
    <T.Mesh position={[cx, 2.18, -0.427]} rotation={[Math.PI / 2, 0, 0]} castShadow>
      <T.CylinderGeometry args={[0.014, 0.014, 0.012, 8]} />
      <T.MeshStandardMaterial color={0xddddaa} metalness={0.6} roughness={0.3} />
    </T.Mesh>
  {/each}
  <!-- Right side crown plaque -->
  <T.Mesh position={[0.415, 2.17, 0]} castShadow>
    <T.BoxGeometry args={[0.02, 0.055, 0.18]} />
    <T.MeshStandardMaterial color={0xdd0000} roughness={0.45} />
  </T.Mesh>

  <!-- ══════════════════════════════════════════════════════════════ -->
  <!-- WINDOW FACES — Real glass via MeshPhysicalMaterial             -->
  <!--                                                                -->
  <!--  LEFT side  (+X face) — window panes                          -->
  <!--  RIGHT side (-X face) — window panes                          -->
  <!--  BACK face  (-Z face) — window panes                          -->
  <!--  FRONT face (+Z face) — DOOR (partial glazing + door panel)   -->
  <!-- ══════════════════════════════════════════════════════════════ -->

  <!-- ─── Helper macro: window face = outer surround + real glass + glazing bars ─── -->

  <!-- === BACK FACE (-Z): Solid Panel === -->
  <T.Mesh position={[0, 1.37, -0.402]} castShadow receiveShadow>
    <T.BoxGeometry args={[0.7, 2.28, 0.015]} />
    <T.MeshStandardMaterial color={0xcc0000} roughness={0.5} />
  </T.Mesh>
  <!-- Horizontal glazing bars (back) — 5 rows -->
  {#each [0.28, 0.61, 0.94, 1.27, 1.65, 2.01] as barY (barY)}
    <T.Mesh position={[0, barY, -0.415]}>
      <T.BoxGeometry args={[0.7, 0.022, 0.025]} />
      <T.MeshStandardMaterial color={0xcc0000} roughness={0.5} />
    </T.Mesh>
  {/each}
  <!-- Vertical glazing bars (back) — 2 dividers -->
  {#each [-0.215, 0.215] as barX (barX)}
    <T.Mesh position={[barX, 1.37, -0.415]}>
      <T.BoxGeometry args={[0.022, 2.24, 0.025]} />
      <T.MeshStandardMaterial color={0xcc0000} roughness={0.5} />
    </T.Mesh>
  {/each}

  <!-- === LEFT SIDE (+X): Full window ===-->
  <T.Mesh position={[0.402, 1.37, 0]}>
    <T.BoxGeometry args={[0.015, 2.28, 0.7]} />
    <T.MeshStandardMaterial
      color={0xc8e8f0}
      transparent
      opacity={0.18}
      roughness={0.05}
      metalness={0.35}
    />
  </T.Mesh>
  {#each [0.28, 0.61, 0.94, 1.27, 1.65, 2.01] as barY (barY)}
    <T.Mesh position={[0.415, barY, 0]}>
      <T.BoxGeometry args={[0.025, 0.022, 0.7]} />
      <T.MeshStandardMaterial color={0xcc0000} roughness={0.5} />
    </T.Mesh>
  {/each}
  {#each [-0.215, 0.215] as barZ (barZ)}
    <T.Mesh position={[0.415, 1.37, barZ]}>
      <T.BoxGeometry args={[0.025, 2.24, 0.022]} />
      <T.MeshStandardMaterial color={0xcc0000} roughness={0.5} />
    </T.Mesh>
  {/each}

  <!-- === RIGHT SIDE (-X): Full window ===-->
  <T.Mesh position={[-0.402, 1.37, 0]}>
    <T.BoxGeometry args={[0.015, 2.28, 0.7]} />
    <T.MeshStandardMaterial
      color={0xc8e8f0}
      transparent
      opacity={0.18}
      roughness={0.05}
      metalness={0.35}
    />
  </T.Mesh>
  {#each [0.28, 0.61, 0.94, 1.27, 1.65, 2.01] as barY (barY)}
    <T.Mesh position={[-0.415, barY, 0]}>
      <T.BoxGeometry args={[0.025, 0.022, 0.7]} />
      <T.MeshStandardMaterial color={0xcc0000} roughness={0.5} />
    </T.Mesh>
  {/each}
  {#each [-0.215, 0.215] as barZ (barZ)}
    <T.Mesh position={[-0.415, 1.37, barZ]}>
      <T.BoxGeometry args={[0.025, 2.24, 0.022]} />
      <T.MeshStandardMaterial color={0xcc0000} roughness={0.5} />
    </T.Mesh>
  {/each}

  <!-- === FRONT FACE (+Z) — DOOR ===
       Lower solid kick-panel + upper glass only, NO full-height opaque surround -->
  <!-- Door lower solid kick-panel (below 0.65) -->
  <T.Mesh position={[0, 0.34, 0.41]} castShadow>
    <T.BoxGeometry args={[0.62, 0.52, 0.025]} />
    <T.MeshStandardMaterial color={0xbb0000} roughness={0.6} />
  </T.Mesh>
  <!-- Door upper glazed section -->
  <T.Mesh position={[0, 1.6, 0.402]}>
    <T.BoxGeometry args={[0.64, 1.74, 0.015]} />
    <T.MeshStandardMaterial
      color={0xc8e8f0}
      transparent
      opacity={0.18}
      roughness={0.05}
      metalness={0.35}
    />
  </T.Mesh>
  <!-- Door glazing bars -->
  {#each [0.78, 1.1, 1.42, 1.75, 2.08] as barY (barY)}
    <T.Mesh position={[0, barY, 0.415]}>
      <T.BoxGeometry args={[0.64, 0.022, 0.025]} />
      <T.MeshStandardMaterial color={0xcc0000} roughness={0.5} />
    </T.Mesh>
  {/each}
  {#each [-0.19, 0.19] as barX (barX)}
    <T.Mesh position={[barX, 1.6, 0.415]}>
      <T.BoxGeometry args={[0.022, 1.76, 0.025]} />
      <T.MeshStandardMaterial color={0xcc0000} roughness={0.5} />
    </T.Mesh>
  {/each}
  <!-- Door vertical centre seam (opening edge) -->
  <T.Mesh position={[0.3, 1.37, 0.422]}>
    <T.BoxGeometry args={[0.006, 2.42, 0.008]} />
    <T.MeshStandardMaterial color={0x880000} roughness={0.7} />
  </T.Mesh>
  <!-- Door handle — brass sphere + rod -->
  <T.Mesh position={[0.22, 1.1, 0.435]} castShadow>
    <T.SphereGeometry args={[0.022, 10, 10]} />
    <T.MeshStandardMaterial color={0xc8a000} metalness={0.8} roughness={0.2} />
  </T.Mesh>
  <T.Mesh position={[0.22, 1.1, 0.425]} rotation={[Math.PI / 2, 0, 0]} castShadow>
    <T.CylinderGeometry args={[0.007, 0.007, 0.04, 8]} />
    <T.MeshStandardMaterial color={0xb89000} metalness={0.75} roughness={0.25} />
  </T.Mesh>

  <!-- ══════════════════════════════════════════════════════════ -->
  <!-- INTERIOR — BACK WALL (behind phone, facing door)            -->
  <!--   Solid red panel on the +Z interior wall                   -->
  <!-- ══════════════════════════════════════════════════════════ -->
  <T.Mesh position={[0, 1.37, -0.37]} receiveShadow>
    <T.BoxGeometry args={[0.74, 2.42, 0.01]} />
    <T.MeshStandardMaterial color={0x9a0000} roughness={0.75} />
  </T.Mesh>

  <!-- ══════════════════════════════════════════════════════════ -->
  <!-- INTERIOR FLOOR                                             -->
  <!-- ══════════════════════════════════════════════════════════ -->
  <T.Mesh position={[0, 0.128, 0]} receiveShadow>
    <T.BoxGeometry args={[0.78, 0.01, 0.78]} />
    <T.MeshStandardMaterial color={0x1e1e1e} roughness={0.92} />
  </T.Mesh>
  <!-- Floor chequerboard feel — alternating raised dark tile lines -->
  {#each [-0.15, 0, 0.15] as tx (tx)}
    <T.Mesh position={[tx, 0.131, 0]} receiveShadow>
      <T.BoxGeometry args={[0.01, 0.003, 0.76]} />
      <T.MeshStandardMaterial color={0x111111} roughness={0.95} />
    </T.Mesh>
  {/each}

  <!-- ══════════════════════════════════════════════════════════════ -->
  <!-- INTERIOR CEILING LIGHT (bulb + warm amber PointLight)          -->
  <!-- ══════════════════════════════════════════════════════════════ -->
  <!-- Ceiling rose / fitting -->
  <T.Mesh position={[0, 2.55, 0]} castShadow>
    <T.CylinderGeometry args={[0.06, 0.06, 0.025, 12]} />
    <T.MeshStandardMaterial color={0xdddddd} roughness={0.5} metalness={0.3} />
  </T.Mesh>
  <!-- Bulb socket -->
  <T.Mesh position={[0, 2.51, 0]} castShadow>
    <T.CylinderGeometry args={[0.018, 0.022, 0.06, 10]} />
    <T.MeshStandardMaterial color={0x888888} metalness={0.5} roughness={0.4} />
  </T.Mesh>
  <!-- Incandescent bulb — emissive sphere -->
  <T.Mesh position={[0, 2.46, 0]}>
    <T.SphereGeometry args={[0.038, 12, 12]} />
    <T.MeshStandardMaterial
      color={0xffee99}
      emissive={0xffcc44}
      emissiveIntensity={2.2}
      roughness={0.1}
    />
  </T.Mesh>
  <!-- Warm amber interior light -->
  <T.PointLight
    position={[0, 2.2, 0]}
    intensity={2.2}
    distance={3.2}
    decay={1.6}
    color={0xffd060}
  />
  <!-- Subtle secondary fill from below (bounce) -->
  <T.PointLight
    position={[0, 0.5, 0]}
    intensity={0.35}
    distance={1.8}
    decay={2.0}
    color={0xff9933}
  />

  <!-- ══════════════════════════════════════════════════════════════ -->
  <!-- INTERIOR TELEPHONE — Detailed prop on back wall                -->
  <!--   Wall-mounted shelf + bakelite body + rotary dial +           -->
  <!--   handset in cradle + coiled cord segments + coin box          -->
  <!--   + instruction card + 999 sticker                             -->
  <!--                                                                -->
  <!--   Located on the back-interior wall face (Z ~0.3, centre)      -->
  <!-- ══════════════════════════════════════════════════════════════ -->

  <!-- Wall-mount shelf bracket -->
  <T.Mesh position={[0, 0.86, -0.25]} castShadow receiveShadow>
    <T.BoxGeometry args={[0.32, 0.02, 0.18]} />
    <T.MeshStandardMaterial color={0x111111} roughness={0.72} metalness={0.2} />
  </T.Mesh>
  <!-- Bracket support triangle (visual only) -->
  <T.Mesh position={[0, 0.79, -0.32]} rotation={[Math.PI / 4, 0, 0]} castShadow>
    <T.BoxGeometry args={[0.04, 0.02, 0.1]} />
    <T.MeshStandardMaterial color={0x111111} roughness={0.7} />
  </T.Mesh>
 
  <!-- Coin / payment box (top unit, GPO-style red) -->
  <T.Mesh position={[0, 1.22, -0.26]} castShadow>
    <T.BoxGeometry args={[0.19, 0.22, 0.11]} />
    <T.MeshStandardMaterial color={0x880000} roughness={0.55} metalness={0.1} />
  </T.Mesh>
  <!-- Coin slot recess -->
  <T.Mesh position={[0, 1.3, -0.202]}>
    <T.BoxGeometry args={[0.06, 0.012, 0.006]} />
    <T.MeshStandardMaterial color={0x222222} roughness={0.8} />
  </T.Mesh>
  <!-- Coin return button -->
  <T.Mesh position={[-0.07, 1.18, -0.202]} rotation={[Math.PI / 2, 0, 0]}>
    <T.CylinderGeometry args={[0.012, 0.012, 0.01, 8]} />
    <T.MeshStandardMaterial color={0xc8a000} metalness={0.7} roughness={0.3} />
  </T.Mesh>
 
  <!-- Phone body (bakelite housing, wall-mounted below coin box) -->
  <T.Mesh position={[0, 0.95, -0.27]} castShadow>
    <T.BoxGeometry args={[0.21, 0.16, 0.13]} />
    <T.MeshStandardMaterial color={0x0c0c0c} roughness={0.68} />
  </T.Mesh>
  <!-- Dial recess face plate (angled upward) -->
  <T.Mesh position={[0, 0.98, -0.205]} rotation={[-Math.PI / 10, 0, 0]} castShadow>
    <T.BoxGeometry args={[0.18, 0.13, 0.018]} />
    <T.MeshStandardMaterial color={0x181818} roughness={0.6} />
  </T.Mesh>
  <!-- Rotary dial disc (cream/ivory) -->
  <T.Mesh position={[0, 1.005, -0.195]} rotation={[-Math.PI / 10, 0, 0]} castShadow>
    <T.CylinderGeometry args={[0.052, 0.052, 0.01, 16]} />
    <T.MeshStandardMaterial color={0xe8e4c2} roughness={0.38} />
  </T.Mesh>
  <!-- Dial finger guard (chrome ring) -->
  <T.Mesh position={[0, 1.005, -0.194]} rotation={[-Math.PI / 10, 0, 0]}>
    <T.CylinderGeometry args={[0.052, 0.052, 0.008, 16, 1, true]} />
    <T.MeshStandardMaterial color={0xcccccc} metalness={0.75} roughness={0.2} />
  </T.Mesh>
  <!-- Dial centre nub -->
  <T.Mesh position={[0, 1.005, -0.192]} rotation={[-Math.PI / 10, 0, 0]}>
    <T.CylinderGeometry args={[0.01, 0.01, 0.012, 8]} />
    <T.MeshStandardMaterial color={0x333333} roughness={0.5} />
  </T.Mesh>
  <!-- Dial holes (7 small cutout cylinders arranged in arc) -->
  {#each Array.from({ length: 7 }, (_, i) => i) as hi (hi)}
    {@const angle = (-Math.PI / 2) + (hi / 6) * Math.PI}
    {@const hx = Math.cos(angle) * 0.034}
    {@const hy = Math.sin(angle) * 0.034}
    <T.Mesh position={[hx, 1.005 + hy, -0.192]} rotation={[-Math.PI / 10, 0, 0]}>
      <T.CylinderGeometry args={[0.007, 0.007, 0.013, 6]} />
      <T.MeshStandardMaterial color={0x050505} roughness={0.9} />
    </T.Mesh>
  {/each}
 
  <!-- Handset cradle prongs (two humps on top of phone) -->
  <T.Mesh position={[-0.06, 1.05, -0.255]} castShadow>
    <T.CylinderGeometry args={[0.014, 0.014, 0.022, 8]} />
    <T.MeshStandardMaterial color={0x0a0a0a} roughness={0.65} />
  </T.Mesh>
  <T.Mesh position={[0.06, 1.05, -0.255]} castShadow>
    <T.CylinderGeometry args={[0.014, 0.014, 0.022, 8]} />
    <T.MeshStandardMaterial color={0x0a0a0a} roughness={0.65} />
  </T.Mesh>
 
  <!-- Handset resting in cradle (on top of phone, angled) -->
  <T.Group position={[0, 1.075, -0.255]} rotation={[0, 0.18, 0]}>
    <!-- Grip bar -->
    <T.Mesh castShadow>
      <T.BoxGeometry args={[0.155, 0.018, 0.026]} />
      <T.MeshStandardMaterial color={0x080808} roughness={0.65} />
    </T.Mesh>
    <!-- Earpiece cup -->
    <T.Mesh position={[-0.08, 0.005, 0]} castShadow>
      <T.CylinderGeometry args={[0.022, 0.018, 0.022, 10]} />
      <T.MeshStandardMaterial color={0x080808} roughness={0.6} />
    </T.Mesh>
    <!-- Mouthpiece cup -->
    <T.Mesh position={[0.08, 0.005, 0]} castShadow>
      <T.CylinderGeometry args={[0.022, 0.018, 0.022, 10]} />
      <T.MeshStandardMaterial color={0x080808} roughness={0.6} />
    </T.Mesh>
    <!-- Earpiece perforated cap (small disc, slightly lighter) -->
    <T.Mesh position={[-0.08, 0.016, 0]} castShadow>
      <T.CylinderGeometry args={[0.018, 0.018, 0.004, 10]} />
      <T.MeshStandardMaterial color={0x1a1a1a} roughness={0.55} />
    </T.Mesh>
    <T.Mesh position={[0.08, 0.016, 0]} castShadow>
      <T.CylinderGeometry args={[0.018, 0.018, 0.004, 10]} />
      <T.MeshStandardMaterial color={0x1a1a1a} roughness={0.55} />
    </T.Mesh>
  </T.Group>

  <!-- Coiled handset cord — 4 short cylinder segments looping down -->
  <T.Mesh position={[-0.07, 1.04, -0.27]} rotation={[0.3, 0.1, 0.55]} castShadow>
    <T.CylinderGeometry args={[0.003, 0.003, 0.055, 6]} />
    <T.MeshStandardMaterial color={0x111111} roughness={0.88} />
  </T.Mesh>
  <T.Mesh position={[-0.1, 0.98, -0.27]} rotation={[0.15, 0, 0.7]} castShadow>
    <T.CylinderGeometry args={[0.003, 0.003, 0.06, 6]} />
    <T.MeshStandardMaterial color={0x111111} roughness={0.88} />
  </T.Mesh>
  <T.Mesh position={[-0.12, 0.92, -0.26]} rotation={[-0.1, 0, 0.55]} castShadow>
    <T.CylinderGeometry args={[0.003, 0.003, 0.07, 6]} />
    <T.MeshStandardMaterial color={0x111111} roughness={0.88} />
  </T.Mesh>
  <T.Mesh position={[-0.1, 0.86, -0.265]} rotation={[0, 0, 0.3]} castShadow>
    <T.CylinderGeometry args={[0.003, 0.003, 0.065, 6]} />
    <T.MeshStandardMaterial color={0x111111} roughness={0.88} />
  </T.Mesh>
 
  <!-- BT instruction card (cream paper above phone) -->
  <T.Mesh position={[0, 1.42, -0.32]} rotation={[Math.PI / 14, 0, 0]} castShadow>
    <T.BoxGeometry args={[0.16, 0.1, 0.003]} />
    <T.MeshStandardMaterial color={0xf0edd5} roughness={0.8} />
  </T.Mesh>
  <!-- Card text lines (dark narrow strips) -->
  {#each [0.032, 0.012, -0.008, -0.028] as ly (ly)}
    <T.Mesh position={[0, 1.42 + ly, -0.318]} rotation={[Math.PI / 14, 0, 0]}>
      <T.BoxGeometry args={[0.12, 0.005, 0.002]} />
      <T.MeshStandardMaterial color={0x333333} roughness={0.8} />
    </T.Mesh>
  {/each}
 
  <!-- 999 emergency sticker (bright red small panel) -->
  <T.Mesh position={[0.07, 1.42, -0.318]} rotation={[Math.PI / 14, 0, 0]}>
    <T.BoxGeometry args={[0.042, 0.028, 0.002]} />
    <T.MeshStandardMaterial color={0xff0000} roughness={0.55} emissive={0xaa0000} emissiveIntensity={0.3} />
  </T.Mesh>
 
  <!-- Phone directory shelf (lower shelf under phone) -->
  <T.Mesh position={[0, 0.72, -0.26]} castShadow>
    <T.BoxGeometry args={[0.24, 0.015, 0.15]} />
    <T.MeshStandardMaterial color={0x0f0f0f} roughness={0.72} />
  </T.Mesh>
  <!-- Phone directory (yellow pages style) -->
  <T.Mesh position={[0.02, 0.74, -0.27]} rotation={[0, 0.05, 0.02]} castShadow>
    <T.BoxGeometry args={[0.14, 0.025, 0.1]} />
    <T.MeshStandardMaterial color={0xe8d840} roughness={0.85} />
  </T.Mesh>

  <!-- ══════════════════════════════════════════════════════════ -->
  <!-- ROOF ASSEMBLY                                              -->
  <!--   Cornice → flat slab → dome taper → finial ball + spike   -->
  <!-- ══════════════════════════════════════════════════════════ -->
  <!-- Lower cornice band -->
  <T.Mesh position={[0, 2.66, 0]} castShadow>
    <T.BoxGeometry args={[0.92, 0.05, 0.92]} />
    <T.MeshStandardMaterial color={0xcc0000} roughness={0.5} />
  </T.Mesh>
  <!-- Upper cornice band -->
  <T.Mesh position={[0, 2.72, 0]} castShadow>
    <T.BoxGeometry args={[0.86, 0.04, 0.86]} />
    <T.MeshStandardMaterial color={0xcc0000} roughness={0.48} />
  </T.Mesh>
  <!-- Flat roof slab -->
  <T.Mesh position={[0, 2.76, 0]} castShadow>
    <T.BoxGeometry args={[0.82, 0.06, 0.82]} />
    <T.MeshStandardMaterial color={0xcc0000} roughness={0.5} />
  </T.Mesh>
  <!-- Dome (3 stacked cones tapering upward) -->
  <T.Mesh position={[0, 2.88, 0]} castShadow>
    <T.CylinderGeometry args={[0.26, 0.41, 0.24, 18]} />
    <T.MeshStandardMaterial color={0xcc0000} roughness={0.46} />
  </T.Mesh>
  <T.Mesh position={[0, 3.02, 0]} castShadow>
    <T.CylinderGeometry args={[0.13, 0.26, 0.24, 16]} />
    <T.MeshStandardMaterial color={0xcc0000} roughness={0.44} />
  </T.Mesh>
  <T.Mesh position={[0, 3.14, 0]} castShadow>
    <T.CylinderGeometry args={[0.04, 0.13, 0.16, 14]} />
    <T.MeshStandardMaterial color={0xcc0000} roughness={0.42} />
  </T.Mesh>
  <!-- Finial ball -->
  <T.Mesh position={[0, 3.24, 0]} castShadow>
    <T.SphereGeometry args={[0.06, 14, 14]} />
    <T.MeshStandardMaterial color={0xcc0000} roughness={0.32} metalness={0.12} />
  </T.Mesh>
  <!-- Finial spike -->
  <T.Mesh position={[0, 3.33, 0]} castShadow>
    <T.CylinderGeometry args={[0.008, 0.028, 0.15, 8]} />
    <T.MeshStandardMaterial color={0xcc0000} roughness={0.28} metalness={0.18} />
  </T.Mesh>

</T.Group>


<!-- 2. Little Black Book (on Couch Seat Cushion) -->
<T.Group position={[0.3, 0.295, 1.75]} rotation={[0, 0.25, 0.02]}>
  <!-- Leather Cover -->
  <T.Mesh position={[0, 0, 0]} castShadow receiveShadow>
    <T.BoxGeometry args={[0.13, 0.015, 0.17]} />
    <T.MeshStandardMaterial color={0x0d0d0d} roughness={0.8} />
  </T.Mesh>
  <!-- Cream Pages showing slightly -->
  {#if bookTexture}
    <T.Mesh 
      position={[0, 0.002, 0.002]} 
      castShadow 
      receiveShadow
      onclick={(e: any) => { e.stopPropagation(); onBookClick?.(); }}
      onpointerenter={() => { document.body.style.cursor = 'pointer'; }}
      onpointerleave={() => { document.body.style.cursor = 'auto'; }}
    >
      <T.BoxGeometry args={[0.12, 0.012, 0.16]} />
      <T.MeshBasicMaterial map={bookTexture} />
    </T.Mesh>
  {:else}
    <T.Mesh 
      position={[0, 0.002, 0.002]} 
      castShadow 
      receiveShadow
      onclick={(e: any) => { e.stopPropagation(); onBookClick?.(); }}
      onpointerenter={() => { document.body.style.cursor = 'pointer'; }}
      onpointerleave={() => { document.body.style.cursor = 'auto'; }}
    >
      <T.BoxGeometry args={[0.12, 0.012, 0.16]} />
      <T.MeshStandardMaterial color={0xf9f5dd} roughness={0.9} />
    </T.Mesh>
  {/if}
</T.Group>

<!-- 3. Travel Bag (with Toothbrush & Comb peeking out, on floor right of couch) -->
<T.Group position={[1.15, 0.08, 1.9]}>
  <!-- Duffel Bag Body -->
  <T.Mesh position={[0, 0, 0]} castShadow receiveShadow>
    <T.BoxGeometry args={[0.3, 0.16, 0.18]} />
    <T.MeshStandardMaterial color={0x4d5d3f} roughness={0.85} />
  </T.Mesh>
  <!-- Handle Straps -->
  <T.Mesh position={[0, 0.08, 0.04]} castShadow>
    <T.BoxGeometry args={[0.22, 0.015, 0.01]} />
    <T.MeshStandardMaterial color={0x2b1e15} roughness={0.8} />
  </T.Mesh>
  <T.Mesh position={[0, 0.08, -0.04]} castShadow>
    <T.BoxGeometry args={[0.22, 0.015, 0.01]} />
    <T.MeshStandardMaterial color={0x2b1e15} roughness={0.8} />
  </T.Mesh>

  <!-- Toothbrush -->
  <T.Mesh position={[-0.04, 0.1, 0.02]} rotation={[0.4, 0.1, 0.6]} castShadow>
    <T.CylinderGeometry args={[0.004, 0.004, 0.07, 6]} />
    <T.MeshStandardMaterial color={0x4189e0} metalness={0.1} roughness={0.5} />
  </T.Mesh>
  <!-- Comb -->
  <T.Mesh position={[0.04, 0.1, -0.02]} rotation={[0.2, -0.1, -0.4]} castShadow>
    <T.BoxGeometry args={[0.003, 0.04, 0.015]} />
    <T.MeshStandardMaterial color={0x1c1c1c} roughness={0.6} />
  </T.Mesh>
</T.Group>

<!-- 4. Pair of Gohills Boots (on floor near front-right platform edge) -->
<T.Group position={[1.9, 0.0, 0.6]}>
  <!-- Left Boot -->
  <T.Group position={[0, 0, 0]} rotation={[0, 0.12, 0]}>
    <!-- Sole -->
    <T.Mesh position={[0, 0.01, 0]} castShadow>
      <T.BoxGeometry args={[0.08, 0.02, 0.18]} />
      <T.MeshStandardMaterial color={0x151515} roughness={0.9} />
    </T.Mesh>
    <!-- Upper Body -->
    <T.Mesh position={[0, 0.075, -0.01]} castShadow>
      <T.BoxGeometry args={[0.075, 0.12, 0.14]} />
      <T.MeshStandardMaterial color={0x52321c} roughness={0.8} />
    </T.Mesh>
    <!-- Elastic Band keeping split sole together -->
    <T.Mesh position={[0, 0.04, 0.01]} scale={[0.85, 1.1, 1]} castShadow>
      <T.TorusGeometry args={[0.048, 0.003, 8, 24]} />
      <T.MeshStandardMaterial color={0xd7ccc8} roughness={0.9} />
    </T.Mesh>
  </T.Group>

  <!-- Right Boot (kicked off at an angle) -->
  <T.Group position={[0.14, 0, 0.06]} rotation={[0, -0.18, 0]}>
    <!-- Sole -->
    <T.Mesh position={[0, 0.01, 0]} castShadow>
      <T.BoxGeometry args={[0.08, 0.02, 0.18]} />
      <T.MeshStandardMaterial color={0x151515} roughness={0.9} />
    </T.Mesh>
    <!-- Upper Body -->
    <T.Mesh position={[0, 0.075, -0.01]} castShadow>
      <T.BoxGeometry args={[0.075, 0.12, 0.14]} />
      <T.MeshStandardMaterial color={0x52321c} roughness={0.8} />
    </T.Mesh>
    <!-- Elastic Band keeping split sole together -->
    <T.Mesh position={[0, 0.04, 0.01]} scale={[0.85, 1.1, 1]} castShadow>
      <T.TorusGeometry args={[0.048, 0.003, 8, 24]} />
      <T.MeshStandardMaterial color={0xd7ccc8} roughness={0.9} />
    </T.Mesh>
  </T.Group>
</T.Group>

