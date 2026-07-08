<script lang="ts">
  import { onMount, untrack } from 'svelte';

  interface Props {
    canvas: HTMLCanvasElement | null;
    onTextureUpdate: () => void;
  }
  let { canvas = $bindable(null), onTextureUpdate }: Props = $props();

  let ctx: CanvasRenderingContext2D | null = null;

  // Reactively obtain the 2D context
  $effect(() => {
    if (canvas && !ctx) {
      ctx = canvas.getContext('2d');
      triggerRender();
    }
  });

  // Re-run render once when mounted/fonts loaded
  onMount(() => {
    document.fonts.ready.then(() => {
      triggerRender();
    });
  });

  function triggerRender() {
    const currentCanvas = canvas;
    const currentCtx = ctx;
    if (!currentCanvas || !currentCtx) return;

    // Fill Page background (warm cream/paper color)
    currentCtx.fillStyle = '#fdfaf0';
    currentCtx.fillRect(0, 0, currentCanvas.width, currentCanvas.height);

    // Left Page / Right Page dividing line in the middle
    currentCtx.strokeStyle = 'rgba(0,0,0,0.15)';
    currentCtx.lineWidth = 1;
    currentCtx.beginPath();
    currentCtx.moveTo(currentCanvas.width / 2, 0);
    currentCtx.lineTo(currentCanvas.width / 2, currentCanvas.height);
    currentCtx.stroke();

    // DRAW LEFT MUSIC SHEET PAGE
    currentCtx.fillStyle = '#1c1c1c';
    currentCtx.font = 'bold 15px "Special Elite", monospace';
    currentCtx.textAlign = 'center';
    currentCtx.fillText('MUSIC TASTE', currentCanvas.width / 4, 35);

    currentCtx.fillStyle = '#a30000';
    currentCtx.font = '9px "Special Elite", monospace';
    currentCtx.fillText('TUNES OVER THE BRICKS', currentCanvas.width / 4, 52);

    // Music Staff lines on left page background
    currentCtx.strokeStyle = 'rgba(0,0,0,0.08)';
    currentCtx.lineWidth = 1;
    for (let row = 0; row < 3; row++) {
      const startY = 75 + row * 50;
      for (let line = 0; line < 5; line++) {
        currentCtx.beginPath();
        currentCtx.moveTo(25, startY + line * 5);
        currentCtx.lineTo(currentCanvas.width / 2 - 25, startY + line * 5);
        currentCtx.stroke();
      }
    }

    // Playlists/tunes text
    currentCtx.fillStyle = '#222222';
    currentCtx.font = 'bold 10px "Cutive Mono", monospace';
    currentCtx.textAlign = 'left';

    const leftList = [
      '1. PINK FLOYD',
      '   - Nobody Home (1979)',
      '   - Comfortably Numb',
      '',
      '2. MY CHEMICAL ROMANCE',
      '   - The End. (2006)',
      '   - Welcome to the Black Parade'
    ];

    leftList.forEach((line, i) => {
      currentCtx.fillText(line, 30, 80 + i * 15);
    });


    // DRAW RIGHT MUSIC SHEET PAGE
    currentCtx.fillStyle = '#1c1c1c';
    currentCtx.font = 'bold 15px "Special Elite", monospace';
    currentCtx.textAlign = 'center';
    currentCtx.fillText('FAVORITES', (currentCanvas.width * 3) / 4, 35);

    currentCtx.fillStyle = '#777777';
    currentCtx.font = '9px "Special Elite", monospace';
    currentCtx.fillText('STAPLE SOUNDS', (currentCanvas.width * 3) / 4, 52);

    // Music Staff lines on right page background
    for (let row = 0; row < 3; row++) {
      const startY = 75 + row * 50;
      for (let line = 0; line < 5; line++) {
        currentCtx.beginPath();
        currentCtx.moveTo(currentCanvas.width / 2 + 25, startY + line * 5);
        currentCtx.lineTo(currentCanvas.width - 25, startY + line * 5);
        currentCtx.stroke();
      }
    }

    // Favorites list
    currentCtx.fillStyle = '#222222';
    currentCtx.font = 'bold 10px "Cutive Mono", monospace';
    currentCtx.textAlign = 'left';

    const rightList = [
      '3. [ALEXANDROS]',
      '   - Senkou / Flash',
      '   - In Me (2014)',
      '',
      '4. A DAY TO REMEMBER',
      '   - Have Faith In Me',
      '   - If It Means Lots To You'
    ];

    rightList.forEach((line, i) => {
      currentCtx.fillText(line, currentCanvas.width / 2 + 30, 80 + i * 15);
    });

    untrack(() => onTextureUpdate());
  }
</script>

<canvas
  bind:this={canvas}
  width="512"
  height="256"
  style="display: none;"
></canvas>
