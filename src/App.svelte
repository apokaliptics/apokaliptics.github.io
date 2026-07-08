<script lang="ts">
  import { onMount } from 'svelte';
  import { fade, scale } from 'svelte/transition';
  import StartupWall from './components/StartupWall.svelte';
  import Scene from './components/Scene.svelte';
  import TVContent from './components/TVContent.svelte';

  // Svelte 5 reactive states
  let introCompleted = $state(false);
  let activePopup = $state<'welcome' | 'projects' | 'taste' | null>(null);
  let cameraMode = $state<'room' | 'tv'>('room');
  
  let tvPower = $state(false);
  let tvChannel = $state(1);
  let canvasElement = $state<HTMLCanvasElement | null>(null);
  let textureUpdateTrigger = $state(0);

  // Audio state
  let musicAudio = $state<HTMLAudioElement | null>(null);
  let musicPlaying = $state(false);
  let musicVolume = $state(0.8);
  let isMuted = $state(false);

  // TV Controls dial state
  let knobRotation = $state(0);

  onMount(() => {
    musicAudio = new Audio('/10 One Of My Turns.mp3');
    musicAudio.loop = true;
    musicAudio.volume = musicVolume;
  });

  function handleIntroComplete() {
    introCompleted = true;
    playMusic();
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

  function closePopup() {
    activePopup = null;
  }

  function openPopup(type: 'welcome' | 'projects' | 'taste') {
    activePopup = type;
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
{:else}
  <div class="app-main" transition:fade={{ duration: 400 }}>
    <!-- 3D Canvas Background -->
    <Scene
      {tvPower}
      {tvChannel}
      {canvasElement}
      {textureUpdateTrigger}
      {cameraMode}
    />

    <!-- Offscreen canvas drawer -->
    <TVContent
      {tvPower}
      {tvChannel}
      bind:canvas={canvasElement}
      onTextureUpdate={triggerTextureUpdate}
    />

    <!-- Top Navigation Bar HUD -->
    <nav class="navbar-overlay">
      <a href="/" class="logo-link" onclick={(e) => { e.preventDefault(); toggleMusic(); }} title="Toggle Background Music">
        <img src="/brick-logo.png" alt="Brick Logo" class="logo-img" class:pulse={musicPlaying}>
      </a>

      <ul class="nav-links">
        <li><button onclick={() => openPopup('welcome')} class:active={activePopup === 'welcome'}>Welcome</button></li>
        <li><button onclick={() => openPopup('projects')} class:active={activePopup === 'projects'}>Projects</button></li>
        <li><button onclick={() => openPopup('taste')} class:active={activePopup === 'taste'}>Taste</button></li>
      </ul>

      <div class="controls-right">
        <!-- Zoom focus toggler -->
        <button class="view-toggle-btn" onclick={() => cameraMode = cameraMode === 'room' ? 'tv' : 'room'}>
          {cameraMode === 'room' ? '[ ZOOM TO TV ]' : '[ VIEW ROOM ]'}
        </button>

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
        {#each Array.from({length: 13}) as _, idx}
          {@const chanNum = idx + 1}
          <button class="hud-channel-btn" class:active={tvChannel === chanNum && tvPower} onclick={() => setChannel(chanNum)}>
            {chanNum}
          </button>
        {/each}
      </div>
    </div>

    <!-- Svelte popups overlay (modal pages) -->
    {#if activePopup}
      <div class="modal-overlay" onclick={closePopup} transition:fade={{ duration: 150 }} role="button" tabindex="0" onkeydown={(e) => { if (e.key === 'Escape' || e.key === 'Enter') closePopup(); }} aria-label="Close modal overlay">
        <div class="modal-card" onclick={(e) => e.stopPropagation()} transition:scale={{ duration: 250, start: 0.95 }} role="document">
          <button class="modal-close-btn" onclick={closePopup}>[X] CLOSE</button>

          {#if activePopup === 'welcome'}
            <div class="welcome-modal-content">
              <div class="profile-frame">
                <img src="/My Wall (red).png" alt="My Wall Red Profile" class="profile-img">
              </div>
              <div class="welcome-textual">
                <h1 class="scarfe-title">WELCOME TO MY WALL</h1>
                <p class="scarfe-sub">ANALOG MEMORIES & STORIES</p>
              </div>
            </div>
          {:else if activePopup === 'projects'}
            <div class="projects-modal-content">
              <div class="wall-textual-assault">
                <h2 class="scarfe-title">TEAR DOWN THE WALL</h2>
                <p class="scarfe-sub">PROJECTS AND MISC OUT THERE</p>
              </div>
              <div class="graffiti-grid">
                <div class="graffiti-item">
                  <h4 class="blood-text">Brick</h4>
                  <p>Music visualizer and player built with Svelte, Three.js, and advanced audio rendering capabilities.</p>
                </div>
                <div class="graffiti-item">
                  <h4 class="blood-text">Rebar</h4>
                  <p>Custom Svelte design system foundation and structural UI layout utility framework.</p>
                </div>
                <div class="graffiti-item">
                  <h4 class="blood-text">Palisade</h4>
                  <p>Secure credentials tunnel proxy and gateway management framework.</p>
                </div>
              </div>
            </div>
          {:else if activePopup === 'taste'}
            <div class="taste-modal-content">
              <div class="wall-textual-assault">
                <h2 class="scarfe-title">MUSIC TASTE</h2>
                <p class="scarfe-sub">TUNES PLAYED OVER THE BRICKS</p>
              </div>
              <div class="spotify-grid">
                <div class="brick-card brick-card-1">
                  <iframe title="Spotify Playlist 1" style="border-radius:0px; border:none;"
                    src="https://open.spotify.com/embed/playlist/5nlfLOnW6GLqG2rbg8bBAm?utm_source=generator" width="100%"
                    height="352" allowfullscreen={true}
                    allow="autoplay; clipboard-write; encrypted-media; fullscreen; picture-in-picture"
                    loading="lazy"></iframe>
                </div>
                <div class="brick-card brick-card-2">
                  <iframe title="Spotify Playlist 2" style="border-radius:0px; border:none;"
                    src="https://open.spotify.com/embed/playlist/5nlfLOnW6GLqG2rbg8bBAm?utm_source=generator" width="100%"
                    height="352" allowfullscreen={true}
                    allow="autoplay; clipboard-write; encrypted-media; fullscreen; picture-in-picture"
                    loading="lazy"></iframe>
                </div>
                <div class="brick-card brick-card-3">
                  <iframe title="Spotify Playlist 3" style="border-radius:0px; border:none;"
                    src="https://open.spotify.com/embed/playlist/683NDQxAOWDsQXYBNaGc6b?utm_source=generator" width="100%"
                    height="352" allowfullscreen={true}
                    allow="autoplay; clipboard-write; encrypted-media; fullscreen; picture-in-picture"
                    loading="lazy"></iframe>
                </div>
              </div>
            </div>
          {/if}
        </div>
      </div>
    {/if}

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

  .view-toggle-btn {
    background: #000;
    color: #00ff66;
    border: 2px solid #00ff66;
    font-family: 'Special Elite', monospace;
    font-size: 0.9rem;
    padding: 6px 12px;
    cursor: pointer;
    box-shadow: 3px 3px 0px #000;
    transition: all 0.2s;
  }

  .view-toggle-btn:hover {
    color: #fff;
    border-color: #fff;
    transform: translate(1px, 1px);
    box-shadow: 2px 2px 0px #000;
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

  /* Modal overlay and popups */
  .modal-overlay {
    position: absolute;
    top: 0; left: 0; width: 100%; height: 100%;
    background: rgba(0, 0, 0, 0.82);
    backdrop-filter: blur(5px);
    z-index: 100;
    display: flex;
    justify-content: center;
    align-items: center;
    padding: 20px;
    box-sizing: border-box;
  }

  .modal-card {
    background: #fff;
    border: 6px solid #000;
    box-shadow: 15px 15px 0px #000;
    max-width: 900px;
    width: 100%;
    max-height: 80vh;
    overflow-y: auto;
    position: relative;
    padding: 40px;
    box-sizing: border-box;
    border-radius: 4px;
    background-image: radial-gradient(rgba(0,0,0,0.05) 1px, transparent 1px);
    background-size: 15px 15px;
  }

  .modal-close-btn {
    position: absolute;
    top: 15px;
    right: 15px;
    background: #cc0000;
    color: #fff;
    border: 3px solid #000;
    padding: 4px 12px;
    font-family: 'Special Elite', monospace;
    font-size: 0.85rem;
    font-weight: bold;
    cursor: pointer;
    box-shadow: 3px 3px 0px #000;
    transition: all 0.2s;
  }

  .modal-close-btn:hover {
    transform: translate(1px, 1px);
    box-shadow: 1px 1px 0px #000;
    background: #ff3333;
  }

  /* Welcome Card inside Modal */
  .welcome-modal-content {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 50px;
    padding: 20px 0;
  }

  .profile-frame {
    background: #fdfbf7;
    padding: 15px;
    padding-bottom: 40px;
    box-shadow: 10px 10px 0px #000;
    transform: rotate(-2deg);
    border: 4px solid #000;
    transition: transform 0.3s, box-shadow 0.3s, border-color 0.3s;
  }

  .profile-frame:hover {
    transform: scale(1.02) rotate(1deg);
    box-shadow: 12px 12px 0px #cc0000;
    border-color: #cc0000;
  }

  .profile-img {
    width: 250px;
    height: 250px;
    object-fit: cover;
    border: 3px solid #000;
    display: block;
  }

  .welcome-textual {
    text-align: left;
  }

  .scarfe-title {
    font-family: 'Permanent Marker', cursive;
    font-size: 3.8rem;
    color: #000;
    text-shadow: 2px 2px 0 #fff;
    transform: rotate(-2deg);
    margin: 0;
  }

  .scarfe-sub {
    font-family: 'Permanent Marker', cursive;
    font-size: 1.4rem;
    color: #a30000;
    transform: rotate(1deg);
    margin: 5px 0 0 0;
  }

  /* Projects Card inside Modal */
  .projects-modal-content {
    text-align: center;
  }

  .wall-textual-assault {
    margin-bottom: 40px;
  }

  .graffiti-grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 30px;
  }

  .graffiti-item {
    background: transparent;
    padding: 25px;
    border: 3px solid #000;
    box-shadow: -8px 8px 0px rgba(0, 0, 0, 0.85);
    text-align: left;
    transition: all 0.3s;
  }

  .graffiti-item p {
    font-family: 'Cutive Mono', monospace;
    font-size: 0.95rem;
    line-height: 1.4;
    margin: 10px 0 0 0;
  }

  .graffiti-item:hover {
    transform: scale(1.04) rotate(0deg) !important;
    box-shadow: 0px 0px 20px rgba(255, 255, 0, 0.85), -8px 8px 0px rgba(0,0,0,0.85) !important;
  }

  .graffiti-item:nth-child(1) {
    transform: rotate(-2deg);
    background: rgba(255, 255, 255, 0.9);
  }
  .graffiti-item:nth-child(1):hover { background: #fff !important; }

  .graffiti-item:nth-child(2) {
    transform: rotate(2deg);
    background: rgba(15, 15, 15, 0.95);
    border-color: #000;
  }
  .graffiti-item:nth-child(2):hover { background: #000 !important; }
  .graffiti-item:nth-child(2) h4 { color: #fff; }
  .graffiti-item:nth-child(2) p { color: #ccc; }

  .graffiti-item:nth-child(3) {
    transform: rotate(-1deg);
    background: rgba(163, 0, 0, 0.9);
    border-color: #000;
  }
  .graffiti-item:nth-child(3):hover { background: #a30000 !important; }
  .graffiti-item:nth-child(3) h4 { color: #000; }
  .graffiti-item:nth-child(3) p { color: #fff; }

  .blood-text {
    font-family: 'Permanent Marker', cursive;
    font-size: 1.6rem;
    margin: 0;
    color: #a30000;
  }

  /* Music Taste inside Modal */
  .taste-modal-content {
    text-align: center;
  }

  .spotify-grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 20px;
  }

  .brick-card {
    background: #000;
    border: 4px solid #000;
    box-shadow: 6px 6px 0px #000;
    padding: 0;
    transition: transform 0.2s, box-shadow 0.2s;
  }

  .brick-card:hover {
    transform: scale(1.02) rotate(-0.5deg);
    box-shadow: 10px 10px 0px #cc0000;
  }

  .brick-card-1 { transform: rotate(-1.5deg); }
  .brick-card-2 { transform: rotate(1deg); }
  .brick-card-3 { transform: rotate(-0.5deg); }

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
    .welcome-modal-content {
      flex-direction: column;
      text-align: center;
    }
    .welcome-textual {
      text-align: center;
    }
    .graffiti-grid, .spotify-grid {
      grid-template-columns: 1fr;
      gap: 20px;
    }
  }
</style>
