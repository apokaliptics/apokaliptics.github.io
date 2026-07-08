<script lang="ts">
  import { useTask } from '@threlte/core';

interface Props {
    camera: any;
    controls: any;
    cameraPosition: [number, number, number];
    cameraTarget: [number, number, number];
    cameraMode: string;
    resetTrigger?: number;
  }
  let { camera, controls, cameraPosition, cameraTarget, cameraMode, resetTrigger = 0 }: Props = $props();

  let initialized = $state(false);
  let lerpActive = $state(false);
  let lastPosition = $state<[number, number, number] | null>(null);
  let lastResetTrigger = $state(0);

  // Monitor target cameraPosition changes and trigger smooth cinematic glide
  $effect(() => {
    if (!camera || !controls) return;

    const posChanged = !lastPosition || 
      lastPosition[0] !== cameraPosition[0] ||
      lastPosition[1] !== cameraPosition[1] ||
      lastPosition[2] !== cameraPosition[2];

    if (posChanged) {
      lastPosition = [...cameraPosition];
      
      if (!initialized) {
        // Initial load: snap immediately
        camera.position.set(cameraPosition[0], cameraPosition[1], cameraPosition[2]);
        controls.target.set(cameraTarget[0], cameraTarget[1], cameraTarget[2]);
        controls.enabled = (cameraMode === 'free');
        controls.update();
        initialized = true;
        lerpActive = false;
      } else {
        // Mode changes: smoothly lerp
        controls.enabled = false;
        lerpActive = true;
      }
    }
  });

  // Smoothly reset camera view back to default if resetTrigger increments in free mode
  $effect(() => {
    if (resetTrigger !== lastResetTrigger) {
      lastResetTrigger = resetTrigger;
      if (cameraMode === 'free' && camera && controls) {
        controls.enabled = false;
        lerpActive = true;
      }
    }
  });

  // Smooth cinematic glide — only active in non-free modes
  useTask(() => {
    if (!camera || !controls || !lerpActive) return;

    const [targetX, targetY, targetZ] = cameraPosition;
    const [tX, tY, tZ] = cameraTarget;

    camera.position.x += (targetX - camera.position.x) * 0.08;
    camera.position.y += (targetY - camera.position.y) * 0.08;
    camera.position.z += (targetZ - camera.position.z) * 0.08;

    controls.target.x += (tX - controls.target.x) * 0.08;
    controls.target.y += (tY - controls.target.y) * 0.08;
    controls.target.z += (tZ - controls.target.z) * 0.08;

    // Drive camera look-at manually (OrbitControls disabled)
    camera.lookAt(controls.target);
    controls.update();

    const posDist = Math.hypot(
      targetX - camera.position.x,
      targetY - camera.position.y,
      targetZ - camera.position.z,
    );
    const tgtDist = Math.hypot(
      tX - controls.target.x,
      tY - controls.target.y,
      tZ - controls.target.z,
    );

    if (posDist < 0.003 && tgtDist < 0.003) {
      camera.position.set(targetX, targetY, targetZ);
      controls.target.set(tX, tY, tZ);
      camera.lookAt(controls.target);
      controls.update();
      lerpActive = false;
      
      // Re-enable interactive OrbitControls when landing in free mode
      if (cameraMode === 'free') {
        controls.enabled = true;
        controls.update();
      }
    }
  });
</script>
