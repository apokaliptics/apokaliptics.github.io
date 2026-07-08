<script lang="ts">
  import { onDestroy, onMount } from 'svelte';
  import { fade } from 'svelte/transition';
  import StartupWall from './components/StartupWall.svelte';
  import Scene from './components/Scene.svelte';
  import TVContent from './components/TVContent.svelte';

  import BookContent from './components/BookContent.svelte';
  import SheetContent from './components/SheetContent.svelte';

  const musicUrl = new URL('../03 Nobody Home.mp3', import.meta.url).href;
  const baseUrl = import.meta.env.BASE_URL;

  // Svelte 5 reactive states
  let introCompleted = $state(false);
  let showWallIntro = $state(false);
  let showApp = $state(false);
  let cameraMode = $state<'free' | 'poem' | 'piano' | 'telephone' | 'tv'>('poem');
  
  let tvPower = $state(false);
  let tvChannel = $state(1);
  let canvasElement = $state<HTMLCanvasElement | null>(null);
  let textureUpdateTrigger = $state(0);

  let bookCanvasElement = $state<HTMLCanvasElement | null>(null);
  let sheetCanvasElement = $state<HTMLCanvasElement | null>(null);
  let bookTextureTrigger = $state(0);
  let sheetTextureTrigger = $state(0);

  // Audio state
  let musicAudio = $state<HTMLAudioElement | null>(null);
  let musicPlaying = $state(false);
  let musicVolume = $state(0.8);
  let isMuted = $state(false);

  // TV Controls dial state
  let knobRotation = $state(0);
  let introTimer: ReturnType<typeof setTimeout> | null = null;

  onMount(() => {
    musicAudio = new Audio(musicUrl);
    musicAudio.loop = true;
    musicAudio.volume = musicVolume;

    const handleKeyDown = (e: KeyboardEvent) => {
      // Ignore key events if focused on input elements (like the volume slider)
      if (document.activeElement?.tagName === 'INPUT') return;

      if (e.key === 'ArrowRight' || e.key === 'd' || e.key === 'D') {
        nextCamera();
      } else if (e.key === 'ArrowLeft' || e.key === 'a' || e.key === 'A') {
        prevCamera();
      }
    };
    window.addEventListener('keydown', handleKeyDown);
    return () => {
      window.removeEventListener('keydown', handleKeyDown);
    };
  });

  onDestroy(() => {
    if (introTimer) {
      clearTimeout(introTimer);
    }
  });

  function handleIntroComplete() {
    introCompleted = true;
    showWallIntro = true;

    if (introTimer) {
      clearTimeout(introTimer);
    }

    introTimer = setTimeout(() => {
      showWallIntro = false;
      showApp = true;
      playMusic();
    }, 2200);
  }

  function playMusic() {
    if (musicAudio) {
      musicAudio.play().then(() => {
        musicPlaying = true;
      }).catch(err => {
        console.warn("Autoplay audio blocked. Click logo or interact to unmute.", err);
      });
    }
  }

  function toggleMusic() {
    if (!musicAudio) return;
    if (musicPlaying) {
      musicAudio.pause();
      musicPlaying = false;
    } else {
      musicAudio.play().then(() => {
        musicPlaying = true;
      });
    }
  }

  function handleVolumeChange(e: Event) {
    const val = parseFloat((e.target as HTMLInputElement).value);
    musicVolume = val;
    if (musicAudio) {
      musicAudio.volume = val;
      isMuted = val === 0;
    }
  }

  function toggleMute() {
    if (!musicAudio) return;
    if (isMuted) {
      musicAudio.volume = musicVolume;
      isMuted = false;
    } else {
      musicAudio.volume = 0;
      isMuted = true;
    }
  }

  function triggerTextureUpdate() {
    textureUpdateTrigger += 1;
  }

  const cameras: Array<'free' | 'poem' | 'piano' | 'telephone' | 'tv'> = ['free', 'poem', 'piano', 'telephone', 'tv'];

  function nextCamera() {
    const idx = cameras.indexOf(cameraMode);
    cameraMode = cameras[(idx + 1) % cameras.length];
  }

  function prevCamera() {
    const idx = cameras.indexOf(cameraMode);
    cameraMode = cameras[(idx - 1 + cameras.length) % cameras.length];
  }

  function selectCamera(mode: 'free' | 'poem' | 'piano' | 'telephone' | 'tv') {
    cameraMode = mode;
  }

  function triggerBookUpdate() {
    bookTextureTrigger += 1;
  }

  function triggerSheetUpdate() {
    sheetTextureTrigger += 1;
  }

  function toggleTVPower() {
    tvPower = !tvPower;
  }

  function rotateChannel() {
    if (!tvPower) return;
    tvChannel = (tvChannel % 13) + 1;
    knobRotation += 27.69;
  }

  function setChannel(chan: number) {
    if (!tvPower) return;
    tvChannel = chan;
    knobRotation = (chan - 1) * 27.69;
  }
</script>

{#if !introCompleted}
  <StartupWall onComplete={handleIntroComplete} />
{:else if showWallIntro}
  <div class="wall-intro" transition:fade={{ duration: 500 }}>
    <div class="wall-intro-card">
      <span>welcome to my wall</span>
    </div>
  </div>
{:else if showApp}
  <div class="app-main" transition:fade={{ duration: 400 }}>
    <!-- 3D Canvas Background -->
    <Scene
      {tvPower}
      {tvChannel}
      {canvasElement}
      {textureUpdateTrigger}
      {cameraMode}
      {bookCanvasElement}
      {sheetCanvasElement}
      {bookTextureTrigger}
      {sheetTextureTrigger}
      onBookClick={() => selectCamera('poem')}
      onSheetClick={() => selectCamera('piano')}
      onTelephoneClick={() => selectCamera('telephone')}
      onTVClick={() => selectCamera('tv')}
    />

    <!-- Offscreen canvas drawers -->
    <TVContent
      {tvPower}
      {tvChannel}
      bind:canvas={canvasElement}
      onTextureUpdate={triggerTextureUpdate}
    />
    <BookContent
      bind:canvas={bookCanvasElement}
      onTextureUpdate={triggerBookUpdate}
    />
    <SheetContent
      bind:canvas={sheetCanvasElement}
      onTextureUpdate={triggerSheetUpdate}
    />

    <!-- Top Navigation Bar HUD -->
    <nav class="navbar-overlay">
      <a href={baseUrl} class="logo-link" onclick={(e) => { e.preventDefault(); toggleMusic(); }} title="Toggle Background Music">
        <img src={`${baseUrl}brick-logo.png`} alt="Brick Logo" class="logo-img" class:pulse={musicPlaying}>
      </a>

      <ul class="nav-links">
        <li><button onclick={() => selectCamera('poem')} class:active={cameraMode === 'poem'}>Welcome</button></li>
        <li><button onclick={() => selectCamera('tv')} class:active={cameraMode === 'tv'}>Projects</button></li>
        <li><button onclick={() => selectCamera('piano')} class:active={cameraMode === 'piano'}>Music</button></li>
        <li><button onclick={() => selectCamera('telephone')} class:active={cameraMode === 'telephone'}>Telephone</button></li>
      </ul>

      <div class="controls-right">
        <!-- Camera tour controls -->
        <div class="camera-tour-controls">
          <button class="tour-btn" onclick={prevCamera} aria-label="Previous camera">&lt;</button>
          <span class="tour-label">
            {cameraMode === 'free' ? 'FREE VIEW' :
             cameraMode === 'poem' ? 'POEM CAM' :
             cameraMode === 'piano' ? 'PIANO CAM' :
             cameraMode === 'telephone' ? 'TELEPHONE CAM' :
             'TV CAM'}
          </span>
          <button class="tour-btn" onclick={nextCamera} aria-label="Next camera">&gt;</button>
        </div>

        <!-- Volume controller -->
        <div class="volume-container">
          <button class="volume-toggle" class:muted={isMuted} onclick={toggleMute} title="Mute/Unmute">
            <div class="volume-bar"></div>
            <div class="volume-bar"></div>
            <div class="volume-bar"></div>
          </button>
          <input type="range" class="volume-slider" min="0" max="1" step="0.01" value={isMuted ? 0 : musicVolume} oninput={handleVolumeChange}>
        </div>
      </div>
    </nav>

    <!-- TV Control Panel (Interactive HUD) -->
    <div class="tv-hud-controls" class:minimized={cameraMode === 'tv'}>
      <h3 class="hud-title">TV DASHBOARD</h3>
      
      <div class="hud-row">
        <!-- Power Switch -->
        <div class="hud-control-group">
          <label for="hud-power-switch">POWER</label>
          <button id="hud-power-switch" class="tv-power-switch" class:on={tvPower} onclick={toggleTVPower} aria-label="Toggle TV Power">
            <span class="switch-knob"></span>
          </button>
        </div>

        <!-- Channel display -->
        <div class="hud-control-group">
          <label for="hud-channel-screen">CHANNEL</label>
          <div id="hud-channel-screen" class="tv-channel-display">{tvPower ? (tvChannel < 10 ? '0' + tvChannel : tvChannel) : '--'}</div>
        </div>

        <!-- Knob dial -->
        <div class="hud-control-group">
          <label for="hud-knob">DIAL</label>
          <button id="hud-knob" class="tv-knob" style="transform: rotate({knobRotation}deg)" onclick={rotateChannel} aria-label="Rotate TV Channel">
            <span class="knob-line"></span>
          </button>
        </div>
      </div>

      <!-- Quick select channel grids -->
      <div class="hud-channel-grid">
        {#each Array.from({ length: 13 }, (_, idx) => idx + 1) as chanNum (chanNum)}
          <button class="hud-channel-btn" class:active={tvChannel === chanNum && tvPower} onclick={() => setChannel(chanNum)}>
            {chanNum}
          </button>
        {/each}
      </div>
    </div>



    <footer class="site-credit">
      MAKE BY KIET MINH - TARGET APOKALIPTICS
    </footer>
  </div>
{/if}

<style>
  :global(body) {
    margin: 0;
    padding: 0;
    overflow: hidden;
    background: #000;
    user-select: none;
    font-family: 'Cutive Mono', monospace;
  }

  .app-main {
    width: 100vw;
    height: 100vh;
    position: relative;
    overflow: hidden;
  }

  .wall-intro {
    width: 100vw;
    height: 100vh;
    display: grid;
    place-items: center;
    background: rgba(0, 0, 0, 0.78);
    color: #f4f0e8;
    text-transform: lowercase;
    letter-spacing: 0.22em;
    font-family: 'Permanent Marker', cursive;
  }

  .wall-intro-card {
    padding: 22px 34px;
    border: 1px solid rgba(255, 255, 255, 0.14);
    background: rgba(0, 0, 0, 0.36);
    box-shadow: 0 0 60px rgba(0, 0, 0, 0.45);
    backdrop-filter: blur(2px);
    font-size: clamp(1.8rem, 3vw, 3.1rem);
    text-shadow: 0 2px 12px rgba(0, 0, 0, 0.6);
  }

  /* Navbar overlay styling */
  .navbar-overlay {
    position: absolute;
    top: 0; left: 0; width: 100%;
    z-index: 10;
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 15px 30px;
    background: linear-gradient(to bottom, rgba(0,0,0,0.85) 0%, transparent 100%);
    box-sizing: border-box;
  }

  .logo-img {
    height: 40px;
    width: auto;
    filter: drop-shadow(2px 2px 0px #000);
    transition: transform 0.2s;
  }

  .logo-img:hover {
    transform: scale(1.05);
  }

  .logo-img.pulse {
    animation: heartbeat 1.5s infinite;
  }

  @keyframes heartbeat {
    0% { transform: scale(1); }
    30% { transform: scale(1.05); }
    60% { transform: scale(1); }
  }

  .nav-links {
    list-style: none;
    display: flex;
    gap: 20px;
    margin: 0;
    padding: 0;
  }

  .nav-links button {
    background: transparent;
    border: none;
    color: #888;
    font-family: 'Permanent Marker', cursive;
    font-size: 1.5rem;
    cursor: pointer;
    text-shadow: 2px 2px 0px #000;
    transition: color 0.2s, transform 0.2s;
  }

  .nav-links button:hover, .nav-links button.active {
    color: #cc0000;
    transform: scale(1.05) rotate(-1deg);
  }

  .controls-right {
    display: flex;
    align-items: center;
    gap: 20px;
  }

  .camera-tour-controls {
    display: flex;
    align-items: center;
    background: #000;
    border: 2px solid #00ff66;
    box-shadow: 3px 3px 0px #000;
    overflow: hidden;
  }

  .tour-btn {
    background: transparent;
    border: none;
    color: #00ff66;
    font-family: 'Special Elite', monospace;
    font-size: 1.1rem;
    font-weight: bold;
    padding: 6px 14px;
    cursor: pointer;
    transition: all 0.2s;
  }

  .tour-btn:hover {
    background: #00ff66;
    color: #000;
  }

  .tour-label {
    color: #fff;
    font-family: 'Special Elite', monospace;
    font-size: 0.9rem;
    padding: 0 16px;
    letter-spacing: 1px;
    border-left: 1px solid rgba(0, 255, 102, 0.3);
    border-right: 1px solid rgba(0, 255, 102, 0.3);
    user-select: none;
    min-width: 90px;
    text-align: center;
  }

  /* Audio sliders */
  .volume-container {
    display: flex;
    align-items: center;
    gap: 10px;
    background: rgba(0,0,0,0.6);
    border: 2px solid #333;
    padding: 5px 12px;
    border-radius: 4px;
  }

  .volume-toggle {
    background: transparent;
    border: none;
    cursor: pointer;
    width: 25px;
    height: 18px;
    display: flex;
    align-items: flex-end;
    gap: 3px;
    padding: 0;
  }

  .volume-bar {
    width: 4px;
    background: #00ff66;
    transition: background-color 0.2s;
  }

  .volume-bar:nth-child(1) { height: 30%; }
  .volume-bar:nth-child(2) { height: 65%; }
  .volume-bar:nth-child(3) { height: 100%; }

  .volume-toggle.muted .volume-bar {
    background: #555;
    height: 10% !important;
  }

  .volume-slider {
    width: 80px;
    cursor: pointer;
    accent-color: #00ff66;
  }

  /* HUD dashboard overlays */
  .tv-hud-controls {
    position: absolute;
    bottom: 25px;
    right: 25px;
    z-index: 10;
    background: linear-gradient(135deg, #1b1b1b 0%, #0d0d0d 100%);
    border: 4px solid #000;
    box-shadow: 8px 8px 0px #000, inset 1px 1px 2px rgba(255,255,255,0.1);
    padding: 20px;
    border-radius: 8px;
    max-width: 320px;
    box-sizing: border-box;
    transition: opacity 0.3s, transform 0.3s;
  }

  .tv-hud-controls.minimized {
    opacity: 0.25;
  }

  .tv-hud-controls.minimized:hover {
    opacity: 1;
  }

  .hud-title {
    font-family: 'Special Elite', monospace;
    font-size: 1rem;
    color: #666;
    margin: 0 0 15px 0;
    border-bottom: 2px double #333;
    padding-bottom: 5px;
    text-align: center;
    letter-spacing: 1px;
  }

  .hud-row {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 15px;
    gap: 15px;
  }

  .hud-control-group {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 5px;
  }

  .hud-control-group label {
    font-family: 'Special Elite', monospace;
    font-size: 0.75rem;
    color: #555;
  }

  /* HUD Power switch */
  .tv-power-switch {
    width: 48px;
    height: 24px;
    background: #000;
    border: 2px solid #333;
    border-radius: 12px;
    position: relative;
    cursor: pointer;
  }

  .tv-power-switch.on {
    background: #008800;
    box-shadow: 0 0 8px rgba(0,255,0,0.4);
  }

  .switch-knob {
    width: 16px;
    height: 16px;
    background: #aaa;
    border-radius: 50%;
    position: absolute;
    top: 2px;
    left: 2px;
    transition: left 0.2s;
  }

  .tv-power-switch.on .switch-knob {
    left: 26px;
  }

  /* LED display */
  .tv-channel-display {
    font-family: 'Special Elite', monospace;
    font-size: 1.4rem;
    color: #ff3333;
    background: #0a0000;
    border: 2px solid #222;
    padding: 2px 8px;
    border-radius: 4px;
    text-shadow: 0 0 4px #ff0000;
    min-width: 32px;
    text-align: center;
  }

  /* Dial knob */
  .tv-knob {
    width: 40px;
    height: 40px;
    border-radius: 50%;
    background: #252525;
    border: 3px solid #000;
    position: relative;
    cursor: pointer;
    transition: transform 0.2s cubic-bezier(0.175, 0.885, 0.32, 1.275);
  }

  .knob-line {
    width: 3px;
    height: 12px;
    background: #ff3333;
    position: absolute;
    top: 3px;
    left: calc(50% - 1.5px);
    border-radius: 1.5px;
  }

  /* Channel Grid */
  .hud-channel-grid {
    display: grid;
    grid-template-columns: repeat(5, 1fr);
    gap: 5px;
    border-top: 2px solid #222;
    padding-top: 15px;
  }

  .hud-channel-btn {
    font-family: 'Special Elite', monospace;
    font-size: 0.75rem;
    color: #777;
    background: #0f0f0f;
    border: 1px solid #222;
    padding: 4px 0;
    cursor: pointer;
    transition: all 0.2s;
  }

  .hud-channel-btn:hover {
    color: #fff;
    background: #1a1a1a;
  }

  .hud-channel-btn.active {
    color: #fff;
    background: #cc0000;
    border-color: #ff3333;
    text-shadow: 0 0 3px #fff;
  }

  /* Credit HUD Footer */
  .site-credit {
    position: absolute;
    bottom: 25px;
    left: 25px;
    font-family: 'Cutive Mono', monospace;
    font-size: 0.75rem;
    color: #555;
    letter-spacing: 2px;
    text-shadow: 1px 1px 0px #000;
    pointer-events: none;
    z-index: 5;
  }

  /* Responsive styles overlay */
  @media (max-width: 900px) {
    .navbar-overlay {
      padding: 10px 15px;
      flex-wrap: wrap;
      gap: 10px;
    }
    .controls-right {
      width: 100%;
      justify-content: space-between;
    }
    .tv-hud-controls {
      left: 25px;
      width: calc(100% - 50px);
      max-width: none;
    }
  }
</style>
