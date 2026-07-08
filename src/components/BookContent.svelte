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
    // Wait for fonts to be ready so text dimensions are correct
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

    // Left Page boundary line
    currentCtx.strokeStyle = 'rgba(0,0,0,0.15)';
    currentCtx.lineWidth = 2;
    currentCtx.beginPath();
    currentCtx.moveTo(currentCanvas.width / 2, 0);
    currentCtx.lineTo(currentCanvas.width / 2, currentCanvas.height);
    currentCtx.stroke();

    // Shadow in the center spine
    const gradient = currentCtx.createLinearGradient(
      currentCanvas.width / 2 - 15, 0,
      currentCanvas.width / 2 + 15, 0
    );
    gradient.addColorStop(0, 'rgba(0,0,0,0.0)');
    gradient.addColorStop(0.5, 'rgba(0,0,0,0.08)');
    gradient.addColorStop(1, 'rgba(0,0,0,0.0)');
    currentCtx.fillStyle = gradient;
    currentCtx.fillRect(currentCanvas.width / 2 - 15, 0, 30, currentCanvas.height);

    // DRAW LEFT PAGE CONTENT
    currentCtx.fillStyle = '#1c1c1c';
    currentCtx.font = 'bold 20px "Special Elite", monospace';
    currentCtx.textAlign = 'center';
    currentCtx.fillText('WELCOME TO', currentCanvas.width / 4, 80);
    currentCtx.fillText('MY WALL', currentCanvas.width / 4, 110);

    currentCtx.fillStyle = '#a30000';
    currentCtx.font = '13px "Special Elite", monospace';
    currentCtx.fillText('MEMORIES & STORIES', currentCanvas.width / 4, 140);

    // Small divider line
    currentCtx.strokeStyle = 'rgba(0,0,0,0.2)';
    currentCtx.lineWidth = 1;
    currentCtx.beginPath();
    currentCtx.moveTo(40, 160);
    currentCtx.lineTo(currentCanvas.width / 2 - 40, 160);
    currentCtx.stroke();

    // Body text
    currentCtx.fillStyle = '#333333';
    currentCtx.font = '13px "Cutive Mono", monospace';
    currentCtx.textAlign = 'left';
    
    const leftLines = [
      'This is an analog',
      'space built with',
      'Three.js and Svelte.',
      '',
      'Use the arrows < >',
      'in the menu to take',
      'a look around.',
      '',
      'Welcome to my wall.',
      '   - apokaliptics'
    ];

    leftLines.forEach((line, i) => {
      currentCtx.fillText(line, 35, 195 + i * 20);
    });

    // DRAW RIGHT PAGE CONTENT (POEMS)
    currentCtx.fillStyle = '#1c1c1c';
    currentCtx.font = 'bold 18px "Special Elite", monospace';
    currentCtx.textAlign = 'center';
    currentCtx.fillText('NOBODY HOME', (currentCanvas.width * 3) / 4, 80);

    currentCtx.fillStyle = '#777777';
    currentCtx.font = 'italic 11px "Special Elite", monospace';
    currentCtx.fillText('Pink Floyd (1979)', (currentCanvas.width * 3) / 4, 105);

    // Small divider line
    currentCtx.strokeStyle = 'rgba(0,0,0,0.2)';
    currentCtx.beginPath();
    currentCtx.moveTo(currentCanvas.width / 2 + 40, 125);
    currentCtx.lineTo(currentCanvas.width - 40, 125);
    currentCtx.stroke();

    // Poem stanza lines
    currentCtx.fillStyle = '#2b2b2b';
    currentCtx.font = '12px "Cutive Mono", monospace';
    currentCtx.textAlign = 'left';

    const poemLines = [
      "I've got a little",
      "black book with my",
      "poems in,",
      "",
      "Got a bag with a",
      "toothbrush and a",
      "comb in,",
      "",
      "When I'm a good dog",
      "they sometimes throw",
      "me the bone in."
    ];

    poemLines.forEach((line, i) => {
      currentCtx.fillText(line, currentCanvas.width / 2 + 35, 155 + i * 20);
    });

    untrack(() => onTextureUpdate());
  }
</script>

<canvas
  bind:this={canvas}
  width="512"
  height="512"
  style="display: none;"
></canvas>
