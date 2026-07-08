<script lang="ts">
  import { onMount, onDestroy, untrack } from 'svelte';

  // Props in Svelte 5
  interface Props {
    tvPower: boolean;
    tvChannel: number;
    canvas: HTMLCanvasElement | null;
    onTextureUpdate: () => void;
  }
  let { tvPower, tvChannel, canvas = $bindable(null), onTextureUpdate }: Props = $props();

  const channelsData: Record<number, any> = {
    1: { type: 'image', src: 'hcm masoleum.jpg', title: 'HCM Mausoleum' },
    2: { type: 'image', src: 'hcm1.jpg', title: 'HCM Opera House' },
    3: { type: 'static' },
    4: { type: 'image', src: 'hcm3.jpg', title: 'Independence Palace' },
    5: { type: 'colorbars' },
    6: { type: 'image', src: 'landmark81.jpg', title: 'Landmark 81' },
    7: { type: 'static' },
    8: { type: 'image', src: 'nhatrang.jpg', title: 'Nha Trang' },
    9: { type: 'standby' },
    10: { type: 'image', src: 'vungtau.jpg', title: 'Vung Tau' },
    11: { type: 'lyrics', text: "I've got thirteen channels of shit on the T.V. to choose from...", source: "Nobody Home (1979)" },
    12: { type: 'static' },
    13: { type: 'image', src: 'the_wall_characters.jpg', title: 'The Wall Cast' }
  };

  let ctx: CanvasRenderingContext2D | null = null;
  let staticInterval: any = null;
  let loadedImages: Record<string, HTMLImageElement> = {};

  // Preload all images on mount
  onMount(() => {
    Object.values(channelsData).forEach(chan => {
      if (chan.type === 'image' && chan.src) {
        const img = new Image();
        img.src = chan.src;
        img.onload = () => {
          loadedImages[chan.src] = img;
          triggerRender();
        };
      }
    });
  });

  onDestroy(() => {
    stopStatic();
  });

  // Reactively obtain the 2D context whenever the canvas element becomes available
  $effect(() => {
    if (canvas && !ctx) {
      ctx = canvas.getContext('2d');
      triggerRender();
    }
  });

  // Re-run render when power or channel changes
  // Read both values explicitly so Svelte tracks them as dependencies
  $effect(() => {
    const _power = tvPower;
    const _channel = tvChannel;
    triggerRender();
  });

  function triggerRender() {
    const currentCanvas = canvas;
    const currentCtx = ctx;
    if (!currentCanvas || !currentCtx) return;
    
    // Clear
    currentCtx.fillStyle = '#000000';
    currentCtx.fillRect(0, 0, currentCanvas.width, currentCanvas.height);

    if (!tvPower) {
      stopStatic();
      untrack(() => onTextureUpdate());
      return;
    }

    const chan = channelsData[tvChannel];
    if (!chan) return;

    if (chan.type === 'static') {
      startStatic();
    } else {
      stopStatic();
      if (chan.type === 'image') {
        const img = loadedImages[chan.src];
        if (img) {
          // Draw image (cover/contain style)
          const scale = Math.min(currentCanvas.width / img.width, (currentCanvas.height - 40) / img.height);
          const w = img.width * scale;
          const h = img.height * scale;
          const x = (currentCanvas.width - w) / 2;
          const y = (currentCanvas.height - 40 - h) / 2;
          currentCtx.drawImage(img, x, y, w, h);
        } else {
          // Loading...
          currentCtx.fillStyle = '#111';
          currentCtx.fillRect(0, 0, currentCanvas.width, currentCanvas.height - 40);
          currentCtx.fillStyle = '#33ff33';
          currentCtx.font = '20px monospace';
          currentCtx.textAlign = 'center';
          currentCtx.fillText('LOADING IMAGE...', currentCanvas.width / 2, currentCanvas.height / 2 - 10);
        }

        // Draw title footer bar
        currentCtx.fillStyle = '#080808';
        currentCtx.fillRect(0, currentCanvas.height - 40, currentCanvas.width, 40);
        currentCtx.strokeStyle = '#181818';
        currentCtx.lineWidth = 2;
        currentCtx.beginPath();
        currentCtx.moveTo(0, currentCanvas.height - 40);
        currentCtx.lineTo(currentCanvas.width, currentCanvas.height - 40);
        currentCtx.stroke();

        currentCtx.fillStyle = '#33ff33';
        currentCtx.font = '16px monospace';
        currentCtx.textAlign = 'center';
        currentCtx.fillText(chan.title.toUpperCase(), currentCanvas.width / 2, currentCanvas.height - 15);

      } else if (chan.type === 'colorbars') {
        // Draw SMPTE color bars
        const colors = ['#e0e0e0', '#c0c000', '#00c0c0', '#00c000', '#c000c0', '#c00000', '#0000c0'];
        const barWidth = currentCanvas.width / colors.length;
        colors.forEach((color, i) => {
          currentCtx.fillStyle = color;
          currentCtx.fillRect(i * barWidth, 0, barWidth, currentCanvas.height);
        });

        // Test overlay
        currentCtx.fillStyle = 'rgba(0,0,0,0.85)';
        currentCtx.fillRect(currentCanvas.width / 2 - 100, currentCanvas.height / 2 - 25, 200, 50);
        currentCtx.strokeStyle = '#cc0000';
        currentCtx.lineWidth = 3;
        currentCtx.strokeRect(currentCanvas.width / 2 - 100, currentCanvas.height / 2 - 25, 200, 50);

        currentCtx.fillStyle = '#ffffff';
        currentCtx.font = '18px monospace';
        currentCtx.textAlign = 'center';
        currentCtx.fillText('SMPTE TEST', currentCanvas.width / 2, currentCanvas.height / 2 + 6);

      } else if (chan.type === 'standby') {
        // Background
        currentCtx.fillStyle = '#3a3a3a';
        currentCtx.fillRect(0, 0, currentCanvas.width, currentCanvas.height);

        // Circle
        currentCtx.strokeStyle = '#aaa';
        currentCtx.lineWidth = 4;
        currentCtx.beginPath();
        currentCtx.arc(currentCanvas.width / 2, currentCanvas.height / 2, 90, 0, Math.PI * 2);
        currentCtx.stroke();

        // Crosshairs
        currentCtx.lineWidth = 2;
        currentCtx.beginPath();
        currentCtx.moveTo(currentCanvas.width / 2 - 90, currentCanvas.height / 2);
        currentCtx.lineTo(currentCanvas.width / 2 + 90, currentCanvas.height / 2);
        currentCtx.moveTo(currentCanvas.width / 2, currentCanvas.height / 2 - 90);
        currentCtx.lineTo(currentCanvas.width / 2, currentCanvas.height / 2 + 90);
        currentCtx.stroke();

        // Banner box
        currentCtx.fillStyle = '#111';
        currentCtx.fillRect(currentCanvas.width / 2 - 80, currentCanvas.height / 2 - 20, 160, 40);
        currentCtx.strokeStyle = '#cc0000';
        currentCtx.lineWidth = 2;
        currentCtx.setLineDash([4, 4]);
        currentCtx.strokeRect(currentCanvas.width / 2 - 80, currentCanvas.height / 2 - 20, 160, 40);
        currentCtx.setLineDash([]); // reset

        currentCtx.fillStyle = '#ffffff';
        currentCtx.font = '14px monospace';
        currentCtx.textAlign = 'center';
        currentCtx.fillText('PLEASE STAND BY', currentCanvas.width / 2, currentCanvas.height / 2 + 5);

      } else if (chan.type === 'lyrics') {
        currentCtx.fillStyle = '#080808';
        currentCtx.fillRect(0, 0, currentCanvas.width, currentCanvas.height);

        // Typographic text lines
        currentCtx.fillStyle = '#33ff33';
        currentCtx.font = '16px monospace';
        currentCtx.textAlign = 'center';

        const lines = [
          "\"I've got thirteen channels of",
          "shit on the T.V. to choose from...\"",
          "",
          "- NOBODY HOME (1979)"
        ];

        lines.forEach((line, i) => {
          if (i === 3) currentCtx.fillStyle = '#00aa00';
          currentCtx.fillText(line, currentCanvas.width / 2, currentCanvas.height / 2 - 40 + i * 25);
        });
      }
      untrack(() => onTextureUpdate());
    }
  }

  function startStatic() {
    if (staticInterval) return;
    staticInterval = setInterval(() => {
      const currentCanvas = canvas;
      const currentCtx = ctx;
      if (!currentCanvas || !currentCtx) return;
      const imgData = currentCtx.createImageData(currentCanvas.width, currentCanvas.height);
      const data = imgData.data;
      for (let i = 0; i < data.length; i += 4) {
        const val = Math.floor(Math.random() * 255);
        data[i] = val;
        data[i+1] = val;
        data[i+2] = val;
        data[i+3] = 255;
      }
      currentCtx.putImageData(imgData, 0, 0);
      onTextureUpdate();
    }, 50);
  }

  function stopStatic() {
    if (staticInterval) {
      clearInterval(staticInterval);
      staticInterval = null;
    }
  }
</script>

<canvas
  bind:this={canvas}
  width="320"
  height="240"
  style="display: none;"
></canvas>
