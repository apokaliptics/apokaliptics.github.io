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

    // DRAW LEFT PAGE CONTENT (POEMS PART 1)
    currentCtx.fillStyle = '#1c1c1c';
    currentCtx.font = 'bold 14px "Special Elite", monospace';
    currentCtx.textAlign = 'center';
    currentCtx.fillText('MY POEMS', currentCanvas.width / 4, 45);

    // Small divider line
    currentCtx.strokeStyle = 'rgba(0,0,0,0.2)';
    currentCtx.lineWidth = 1;
    currentCtx.beginPath();
    currentCtx.moveTo(30, 58);
    currentCtx.lineTo(currentCanvas.width / 2 - 30, 58);
    currentCtx.stroke();

    // Body text (Part 1 of Lyrics)
    currentCtx.fillStyle = '#2b2b2b';
    currentCtx.font = '10.5px "Cutive Mono", monospace';
    currentCtx.textAlign = 'left';
    
    const leftLines = [
      "I've got a little black book",
      "  with my poems in",
      "Got a bag with a toothbrush",
      "  and a comb in",
      "When I'm a good dog",
      "  they sometimes throw",
      "  me the bone in",
      "",
      "I got elastic bands",
      "  keepin' my shoes on",
      "Got those swollen hand blues",
      "I got thirteen channels of",
      "  shit on the TV",
      "  to choose from",
      "",
      "I've got electric light",
      "And I've got second sight",
      "I got amazing powers",
      "  of observation"
    ];

    leftLines.forEach((line, i) => {
      currentCtx.fillText(line, 30, 80 + i * 16);
    });

    // DRAW RIGHT PAGE CONTENT (POEMS PART 2)
    currentCtx.fillStyle = '#1c1c1c';
    currentCtx.font = 'bold 14px "Special Elite", monospace';
    currentCtx.textAlign = 'center';
    currentCtx.fillText('NOBODY HOME', (currentCanvas.width * 3) / 4, 45);

    currentCtx.fillStyle = '#666666';
    currentCtx.font = 'italic 9px "Special Elite", monospace';
    currentCtx.fillText('Pink Floyd (1979)', (currentCanvas.width * 3) / 4, 58);

    // Small divider line
    currentCtx.strokeStyle = 'rgba(0,0,0,0.2)';
    currentCtx.beginPath();
    currentCtx.moveTo(currentCanvas.width / 2 + 30, 70);
    currentCtx.lineTo(currentCanvas.width - 30, 70);
    currentCtx.stroke();

    // Body text (Part 2 of Lyrics)
    currentCtx.fillStyle = '#2b2b2b';
    currentCtx.font = '10.5px "Cutive Mono", monospace';
    currentCtx.textAlign = 'left';

    const rightLines = [
      "And that is how I know,",
      "  when I try to get through",
      "On the telephone to you,",
      "  there'll be nobody home",
      "",
      "I've got the obligatory",
      "  Hendrix perm and the",
      "  inevitable pinhole burns",
      "Now all down the front of",
      "  my favorite satin shirt",
      "I've got nicotine stains",
      "  on my fingers, I've got",
      "  a silver spoon on a chain",
      "Got a grand piano to prop",
      "  up my mortal remains",
      "",
      "I've got wild staring eyes",
      "And I've got a strong urge",
      "  to fly, but I got",
      "  nowhere to fly to",
      "Ooh, babe when I pick up",
      "  the phone there is still",
      "  nobody home",
      "",
      "I've got a pair of Gohills",
      "  boots and I got",
      "  fading roots"
    ];

    rightLines.forEach((line, i) => {
      currentCtx.fillText(line, currentCanvas.width / 2 + 30, 92 + i * 14.5);
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
