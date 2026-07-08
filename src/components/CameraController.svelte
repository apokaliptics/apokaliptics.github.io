<script lang="ts">
  import { useTask } from '@threlte/core';

  interface Props {
    camera: any;
    controls: any;
    cameraPosition: [number, number, number];
    cameraTarget: [number, number, number];
    cameraMode: string;
  }
  let { camera, controls, cameraPosition, cameraTarget, cameraMode }: Props = $props();

  let initialized = $state(false);
  let prevMode = $state<string>('');
  let lerpActive = $state(false);

  // On mount and mode-change: configure OrbitControls enabled state and trigger lerp
  $effect(() => {
    if (!camera || !controls) return;

    const isFree = cameraMode === 'free';

    if (!initialized) {
      camera.position.set(cameraPosition[0], cameraPosition[1], cameraPosition[2]);
      controls.target.set(cameraTarget[0], cameraTarget[1], cameraTarget[2]);
      // Free mode: enable orbit. All other modes: disable so it can't fight the lerp.
      controls.enabled = isFree;
      controls.update();
      initialized = true;
      prevMode = cameraMode;
      lerpActive = !isFree;
      return;
    }

    if (cameraMode !== prevMode) {
      prevMode = cameraMode;
      if (isFree) {
        // Re-enable orbit and jump immediately to free position
        controls.enabled = true;
        camera.position.set(cameraPosition[0], cameraPosition[1], cameraPosition[2]);
        controls.target.set(cameraTarget[0], cameraTarget[1], cameraTarget[2]);
        controls.update();
        lerpActive = false;
      } else {
        // Disable orbit so it cannot override the cinematic lerp
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

    camera.position.x += (targetX - camera.position.x) * 0.055;
    camera.position.y += (targetY - camera.position.y) * 0.055;
    camera.position.z += (targetZ - camera.position.z) * 0.055;

    controls.target.x += (tX - controls.target.x) * 0.055;
    controls.target.y += (tY - controls.target.y) * 0.055;
    controls.target.z += (tZ - controls.target.z) * 0.055;

    // Drive camera look-at manually (OrbitControls disabled)
    camera.lookAt(controls.target);

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
      lerpActive = false;
    }
  });
</script>
