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
  let cameraMode = $state<'free' | 'poem' | 'piano' | 'telephone' | 'tv'>('free');
  let resetTrigger = $state(0);
  let activeModal = $state<'none' | 'telephone' | 'poem' | 'piano'>('none');

  $effect(() => {
    // Whenever cameraMode changes, close any active modal
    if (cameraMode) {
      activeModal = 'none';
    }
  });
  
  let tvPower = $state(false);
  let tvChannel = $state(1);
  let pianoLight = $state(true);
  let telephoneLight = $state(true);
  let tableLamp = $state(true);
  let couchSpotlight = $state(true);
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

  // Automatically pause the "Nobody Home" background music when a YouTube video is playing, and resume it when TV is off or tuned away.
  $effect(() => {
    if (!musicAudio) return;
    const isYouTubeChannel = [7, 9, 12].includes(tvChannel);
    if (tvPower && isYouTubeChannel) {
      musicAudio.pause();
    } else if (musicPlaying) {
      musicAudio.play().catch(err => {
        console.warn("Autoplay audio blocked or could not play.", err);
      });
    }
  });

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
  }

  function enterApp() {
    showWallIntro = false;
    showApp = true;
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
    if (cameraMode === mode) {
      cameraMode = 'free';
      if (mode === 'free') {
        resetTrigger += 1;
      }
    } else {
      cameraMode = mode;
    }
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
  <!-- svelte-ignore a11y_click_events_have_key_events -->
  <!-- svelte-ignore a11y_no_static_element_interactions -->
  <div class="wall-intro" transition:fade={{ duration: 500 }} onclick={enterApp} style="cursor: pointer;">
    <div class="wall-intro-card">
      <span>welcome to my wall</span>
      <div class="enter-subtext">click to enter</div>
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
      {resetTrigger}
      {bookCanvasElement}
      {sheetCanvasElement}
      {bookTextureTrigger}
      {sheetTextureTrigger}
      onBookClick={() => selectCamera('poem')}
      onSheetClick={() => selectCamera('piano')}
      onTelephoneClick={() => selectCamera('telephone')}
      onTVClick={() => selectCamera('tv')}
      {pianoLight}
      {telephoneLight}
      {tableLamp}
      {couchSpotlight}
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

    <!-- Electric Light Control Panel (Interactive HUD) -->
    <div class="light-hud-controls" class:minimized={cameraMode === 'poem'}>
      <h3 class="hud-title">ELECTRIC LIGHTS</h3>
      
      <div class="light-rows-container">
        <!-- Piano Spotlight -->
        <div class="light-row">
          <span class="light-label">PIANO SPOT</span>
          <div class="light-control-wrapper">
            <span class="light-led" class:on={pianoLight}></span>
            <button class="light-switch" class:on={pianoLight} onclick={() => pianoLight = !pianoLight} aria-label="Toggle Piano Spotlight">
              <span class="switch-knob"></span>
            </button>
          </div>
        </div>

        <!-- Telephone Light -->
        <div class="light-row">
          <span class="light-label">PHONE BOOTH</span>
          <div class="light-control-wrapper">
            <span class="light-led" class:on={telephoneLight}></span>
            <button class="light-switch" class:on={telephoneLight} onclick={() => telephoneLight = !telephoneLight} aria-label="Toggle Telephone Light">
              <span class="switch-knob"></span>
            </button>
          </div>
        </div>

        <!-- Table Lamp -->
        <div class="light-row">
          <span class="light-label">TABLE LAMP</span>
          <div class="light-control-wrapper">
            <span class="light-led" class:on={tableLamp}></span>
            <button class="light-switch" class:on={tableLamp} onclick={() => tableLamp = !tableLamp} aria-label="Toggle Table Lamp">
              <span class="switch-knob"></span>
            </button>
          </div>
        </div>

        <!-- Couch Spotlight -->
        <div class="light-row">
          <span class="light-label">COUCH SPOT</span>
          <div class="light-control-wrapper">
            <span class="light-led" class:on={couchSpotlight}></span>
            <button class="light-switch" class:on={couchSpotlight} onclick={() => couchSpotlight = !couchSpotlight} aria-label="Toggle Couch Spotlight">
              <span class="switch-knob"></span>
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- Action buttons inside camera modes -->
    {#if cameraMode === 'telephone' && activeModal === 'none'}
      <div class="action-overlay-container" transition:fade={{ duration: 250 }}>
        <button class="action-overlay-btn call-btn" onclick={() => activeModal = 'telephone'}>
          CALL
        </button>
      </div>
    {:else if cameraMode === 'poem' && activeModal === 'none'}
      <div class="action-overlay-container" transition:fade={{ duration: 250 }}>
        <button class="action-overlay-btn view-poems-btn" onclick={() => activeModal = 'poem'}>
          VIEW POEMS
        </button>
      </div>
    {:else if cameraMode === 'piano' && activeModal === 'none'}
      <div class="action-overlay-container" transition:fade={{ duration: 250 }}>
        <button class="action-overlay-btn see-sheet-btn" onclick={() => activeModal = 'piano'}>
          SEE PIANO SHEET
        </button>
      </div>
    {/if}

    <!-- Modal Overlays -->
    {#if activeModal !== 'none'}
      <!-- svelte-ignore a11y_click_events_have_key_events -->
      <!-- svelte-ignore a11y_no_static_element_interactions -->
      <div class="modal-backdrop" transition:fade={{ duration: 300 }} onclick={() => activeModal = 'none'}>
        <div class="modal-wrapper" onclick={(e) => e.stopPropagation()}>
          
          {#if activeModal === 'telephone'}
            <!-- Telephone "Tear Down The Wall" Modal -->
            <div class="modal-card phone-modal" transition:fade={{ duration: 250 }}>
              <div class="polaroid-frame">
                <img src={`${baseUrl}the_wall_characters.jpg`} alt="Tear Down The Wall" class="polaroid-img">
                <div class="polaroid-caption">TEAR DOWN THE WALL</div>
              </div>
              <div class="phone-call-info">
                <div class="profile-header">APOKALIPTICS</div>
                <div class="profile-subtitle">OLD PROFILE CONNECTION</div>
                
                <div class="phone-projects-section">
                  <h4 class="phone-projects-title">PROJECTS:</h4>
                  <ul class="phone-projects-list">
                    <li>Brick</li>
                    <li>Frameprint</li>
                    <li><a href="https://community.obsidian.md/plugins/concrete" target="_blank" rel="noopener noreferrer" class="project-link-telephone">Concrete</a></li>
                  </ul>
                </div>

                <div class="call-status pulse-slow">CALLING...</div>
                <button class="hangup-btn" onclick={() => activeModal = 'none'}>HANG UP</button>
              </div>
            </div>
            
          {:else if activeModal === 'poem'}
            <!-- Poem Brick Wall Modal -->
            <div class="modal-card poem-modal" style="background-image: linear-gradient(rgba(0,0,0,0.65), rgba(0,0,0,0.65)), url('{baseUrl}My%20Wall%20(red).png')" transition:fade={{ duration: 250 }}>
              <button class="close-modal-btn" onclick={() => activeModal = 'none'}>&times;</button>
              <h2 class="poem-modal-title">welcome to my wall</h2>
              <div class="poem-scrollable-content">
                <div class="poem-stanza">
                  <p>I've got a little black book with my poems in</p>
                  <p>Got a bag with a toothbrush and a comb in</p>
                  <p>When I'm a good dog</p>
                  <p>They sometimes throw me the bone in</p>
                </div>
                <div class="poem-stanza">
                  <p>I got elastic bands keepin' my shoes on</p>
                  <p>Got those swollen hand blues</p>
                  <p>I got thirteen channels of shit on the TV to choose from</p>
                </div>
                <div class="poem-stanza">
                  <p>I've got electric light</p>
                  <p>And I've got second sight</p>
                  <p>I got amazing powers of observation</p>
                  <p>And that is how I know, when I try to get through</p>
                  <p>On the telephone to you, there'll be nobody home</p>
                </div>
                <div class="poem-stanza">
                  <p>I've got the obligatory Hendrix perm and the inevitable pinhole burns</p>
                  <p>Now all down the front of my favorite satin shirt</p>
                  <p>I've got nicotine stains on my fingers, I've got a silver spoon on a chain</p>
                  <p>Got a grand piano to prop up my mortal remains</p>
                </div>
                <div class="poem-stanza">
                  <p>I've got wild staring eyes</p>
                  <p>And I've got a strong urge to fly, but I got nowhere to fly to</p>
                  <p>Ooh, babe when I pick up the phone there is still nobody home</p>
                </div>
                <div class="poem-stanza">
                  <p>I've got a pair of Gohills boots and I got fading roots</p>
                </div>
                <div class="poem-signature">- Pink Floyd (Nobody Home, 1979)</div>
              </div>
            </div>
            
          {:else if activeModal === 'piano'}
            <!-- Piano Sheet Modal -->
            <div class="modal-card sheet-modal" transition:fade={{ duration: 250 }}>
              <button class="close-modal-btn dark" onclick={() => activeModal = 'none'}>&times;</button>
              <div class="sheet-paper">
                <div class="sheet-page left-page">
                  <h3 class="sheet-title">if i commit suicide</h3>
                  <div class="embed-container">
                    <iframe allow="autoplay *; encrypted-media *; fullscreen *; clipboard-write" frameborder="0" height="450" style="width:100%;max-width:660px;overflow:hidden;border-radius:10px;" sandbox="allow-forms allow-popups allow-same-origin allow-scripts allow-storage-access-by-user-activation allow-top-navigation-by-user-activation" src="https://embed.music.apple.com/vn/playlist/if-i-commit-suicide/pl.u-d2b08rWsMkEPVZX" title="if i commit suicide playlist"></iframe>
                  </div>
                </div>
                
                <div class="sheet-page right-page">
                  <h3 class="sheet-title">one at a time</h3>
                  <div class="embed-container">
                    <iframe allow="autoplay *; encrypted-media *; fullscreen *; clipboard-write" frameborder="0" height="450" style="width:100%;max-width:660px;overflow:hidden;border-radius:10px;" sandbox="allow-forms allow-popups allow-same-origin allow-scripts allow-storage-access-by-user-activation allow-top-navigation-by-user-activation" src="https://embed.music.apple.com/vn/playlist/one-at-a-time/pl.u-DdAN2Bdsa65DvWX" title="one at a time playlist"></iframe>
                  </div>
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
    background: rgba(0, 0, 0, 0.36);
    box-shadow: 0 0 60px rgba(0, 0, 0, 0.45);
    backdrop-filter: blur(2px);
    font-size: clamp(1.8rem, 3vw, 3.1rem);
    text-shadow: 0 2px 12px rgba(0, 0, 0, 0.6);
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  .enter-subtext {
    font-size: 1rem;
    color: #888;
    margin-top: 15px;
    text-align: center;
    letter-spacing: 0.1em;
    font-family: 'Cutive Mono', monospace;
    text-transform: uppercase;
    animation: pulse-slow 2s infinite;
  }

  @keyframes pulse-slow {
    0%, 100% { opacity: 0.4; }
    50% { opacity: 1; }
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
    bottom: 12px;
    left: 25px;
    font-family: 'Cutive Mono', monospace;
    font-size: 0.7rem;
    color: #444;
    letter-spacing: 2px;
    text-shadow: 1px 1px 0px #000;
    pointer-events: none;
    z-index: 5;
  }

  /* Light HUD dashboard overlay */
  .light-hud-controls {
    position: absolute;
    bottom: 35px;
    left: 25px;
    z-index: 10;
    background: linear-gradient(135deg, #1b1b1b 0%, #0d0d0d 100%);
    border: 4px solid #000;
    box-shadow: 8px 8px 0px #000, inset 1px 1px 2px rgba(255,255,255,0.1);
    padding: 20px;
    border-radius: 8px;
    min-width: 260px;
    box-sizing: border-box;
    transition: opacity 0.3s, transform 0.3s;
  }

  .light-hud-controls.minimized {
    opacity: 0.25;
  }

  .light-hud-controls.minimized:hover {
    opacity: 1;
  }

  .light-rows-container {
    display: flex;
    flex-direction: column;
    gap: 12px;
  }

  .light-row {
    display: flex;
    justify-content: space-between;
    align-items: center;
    border-bottom: 1px dashed #222;
    padding-bottom: 8px;
  }

  .light-row:last-child {
    border-bottom: none;
    padding-bottom: 0;
  }

  .light-label {
    font-family: 'Special Elite', monospace;
    font-size: 0.8rem;
    color: #888;
    letter-spacing: 0.5px;
  }

  .light-control-wrapper {
    display: flex;
    align-items: center;
    gap: 12px;
  }

  /* LED indicator light */
  .light-led {
    width: 8px;
    height: 8px;
    border-radius: 50%;
    background: #3a0e0e;
    box-shadow: inset 0 1px 1px rgba(0,0,0,0.5);
    transition: background-color 0.2s, box-shadow 0.2s;
  }

  .light-led.on {
    background: #00ff66;
    box-shadow: 0 0 8px #00ff66, inset 0 1px 1px rgba(255,255,255,0.5);
  }

  /* Retro light toggle switch */
  .light-switch {
    width: 42px;
    height: 20px;
    background: #000;
    border: 2px solid #333;
    border-radius: 10px;
    position: relative;
    cursor: pointer;
    padding: 0;
  }

  .light-switch.on {
    background: #008800;
    box-shadow: 0 0 6px rgba(0,255,0,0.3);
  }

  .light-switch .switch-knob {
    width: 12px;
    height: 12px;
    background: #aaa;
    border-radius: 50%;
    position: absolute;
    top: 2px;
    left: 2px;
    transition: left 0.2s;
  }

  .light-switch.on .switch-knob {
    left: 24px;
  }

  /* Float action buttons in camera modes */
  .action-overlay-container {
    position: absolute;
    bottom: 90px;
    left: 50%;
    transform: translateX(-50%);
    z-index: 10;
    pointer-events: auto;
  }

  .action-overlay-btn {
    font-family: 'Special Elite', monospace;
    font-size: 1.1rem;
    letter-spacing: 2px;
    text-transform: uppercase;
    color: #fff;
    background: rgba(18, 18, 18, 0.72);
    border: 2px solid rgba(255, 255, 255, 0.3);
    padding: 12px 28px;
    border-radius: 30px;
    cursor: pointer;
    box-shadow: 0 4px 20px rgba(0, 0, 0, 0.5);
    backdrop-filter: blur(8px);
    transition: all 0.3s cubic-bezier(0.19, 1, 0.22, 1);
  }

  .action-overlay-btn:hover {
    background: #fff;
    color: #000;
    border-color: #fff;
    box-shadow: 0 0 15px rgba(255, 255, 255, 0.6);
    transform: scale(1.05);
  }

  .call-btn {
    border-color: rgba(255, 51, 51, 0.4);
    box-shadow: 0 4px 20px rgba(255, 0, 0, 0.15);
  }
  .call-btn:hover {
    background: #ff3333;
    color: #fff;
    border-color: #ff3333;
    box-shadow: 0 0 18px rgba(255, 51, 51, 0.6);
  }

  /* Modal Backdrop */
  .modal-backdrop {
    position: absolute;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    background: rgba(0, 0, 0, 0.78);
    backdrop-filter: blur(10px);
    z-index: 100;
    display: grid;
    place-items: center;
  }

  .modal-wrapper {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 100%;
    height: 100%;
    pointer-events: auto;
  }

  /* Modal Cards Base */
  .modal-card {
    border-radius: 8px;
    box-shadow: 0 20px 60px rgba(0, 0, 0, 0.8);
    display: flex;
    overflow: hidden;
    position: relative;
    max-width: 90%;
    max-height: 90vh;
  }

  /* Close Button inside modals */
  .close-modal-btn {
    position: absolute;
    top: 15px;
    right: 20px;
    font-size: 2.2rem;
    color: rgba(255, 255, 255, 0.6);
    background: none;
    border: none;
    cursor: pointer;
    z-index: 10;
    transition: color 0.2s;
  }
  .close-modal-btn:hover {
    color: #fff;
  }
  .close-modal-btn.dark {
    color: rgba(0, 0, 0, 0.4);
  }
  .close-modal-btn.dark:hover {
    color: #000;
  }

  /* 1. Phone Modal ("Tear Down The Wall") */
  .phone-modal {
    width: 680px;
    background: #111;
    border: 1px solid rgba(255, 255, 255, 0.1);
    flex-direction: row;
    align-items: stretch;
  }
  .polaroid-frame {
    background: #faf7f2;
    padding: 20px 20px 35px 20px;
    box-shadow: 0 10px 25px rgba(0,0,0,0.5);
    transform: rotate(-1.5deg);
    margin: 25px;
    display: flex;
    flex-direction: column;
    align-items: center;
    border-radius: 2px;
    width: 250px;
  }
  .polaroid-img {
    width: 100%;
    aspect-ratio: 1;
    object-fit: cover;
    filter: sepia(0.2) contrast(1.1);
    border: 1px solid rgba(0, 0, 0, 0.15);
  }
  .polaroid-caption {
    font-family: 'Permanent Marker', cursive;
    font-size: 1.1rem;
    color: #2b2b2b;
    margin-top: 18px;
    letter-spacing: 1px;
    text-align: center;
  }
  .phone-call-info {
    flex: 1;
    display: flex;
    flex-direction: column;
    justify-content: center;
    padding: 40px;
    color: #fff;
  }
  .profile-header {
    font-family: 'Special Elite', monospace;
    font-size: 1.6rem;
    letter-spacing: 2px;
    color: #ff3333;
    margin-bottom: 5px;
  }
  .profile-subtitle {
    font-family: 'Cutive Mono', monospace;
    font-size: 0.8rem;
    color: #777;
    margin-bottom: 25px;
    letter-spacing: 1px;
  }
  .phone-projects-section {
    margin-bottom: 30px;
    border-top: 1px solid rgba(255, 255, 255, 0.1);
    border-bottom: 1px solid rgba(255, 255, 255, 0.1);
    padding: 15px 0;
  }
  .phone-projects-title {
    font-family: 'Special Elite', monospace;
    font-size: 0.9rem;
    color: #888;
    margin: 0 0 10px 0;
    letter-spacing: 1px;
  }
  .phone-projects-list {
    list-style: none;
    padding: 0;
    margin: 0;
    display: flex;
    flex-direction: column;
    gap: 8px;
    font-family: 'Cutive Mono', monospace;
    font-size: 0.95rem;
    text-align: left;
  }
  .phone-projects-list li {
    color: #eee;
    padding-left: 18px;
    position: relative;
  }
  .phone-projects-list li::before {
    content: '■';
    color: #ff3333;
    font-size: 0.65rem;
    position: absolute;
    left: 0;
    top: 0px;
  }
  .project-link-telephone {
    color: #ff3333;
    text-decoration: none;
    border-bottom: 1px dashed rgba(255, 51, 51, 0.5);
    transition: all 0.2s;
  }
  .project-link-telephone:hover {
    color: #ff6666;
    border-bottom-style: solid;
    text-shadow: 0 0 5px rgba(255, 51, 51, 0.3);
  }
  .call-status {
    font-family: 'Special Elite', monospace;
    font-size: 1.1rem;
    color: #fff;
    margin-bottom: 35px;
    letter-spacing: 2px;
  }
  .hangup-btn {
    font-family: 'Special Elite', monospace;
    background: #ff3333;
    color: #fff;
    border: none;
    padding: 12px 0;
    border-radius: 25px;
    font-size: 1rem;
    cursor: pointer;
    box-shadow: 0 0 15px rgba(255, 51, 51, 0.3);
    transition: all 0.2s;
  }
  .hangup-btn:hover {
    background: #ff1111;
    box-shadow: 0 0 20px rgba(255, 51, 51, 0.6);
  }

  /* 2. Poem Modal (Brick background) */
  .poem-modal {
    width: 580px;
    padding: 45px 35px;
    background-size: cover;
    background-position: center;
    flex-direction: column;
    border: 1px solid rgba(255, 255, 255, 0.15);
    text-align: center;
    color: #fff;
  }
  .poem-modal-title {
    font-family: 'Permanent Marker', cursive;
    font-size: 2.2rem;
    letter-spacing: 2px;
    margin-bottom: 25px;
    text-shadow: 0 3px 8px rgba(0, 0, 0, 0.8);
  }
  .poem-scrollable-content {
    max-height: 55vh;
    overflow-y: auto;
    padding-right: 10px;
  }
  .poem-scrollable-content::-webkit-scrollbar {
    width: 6px;
  }
  .poem-scrollable-content::-webkit-scrollbar-track {
    background: rgba(0,0,0,0.2);
  }
  .poem-scrollable-content::-webkit-scrollbar-thumb {
    background: rgba(255,255,255,0.2);
    border-radius: 3px;
  }
  .poem-stanza {
    font-family: 'Special Elite', monospace;
    font-size: 1rem;
    line-height: 1.6;
    margin-bottom: 30px;
    color: #eee;
    text-shadow: 0 2px 5px rgba(0, 0, 0, 0.8);
  }
  .poem-stanza p {
    margin: 5px 0;
  }
  .poem-signature {
    font-family: 'Cutive Mono', monospace;
    font-size: 0.8rem;
    color: #aaa;
    margin-top: 15px;
    font-style: italic;
    text-shadow: 0 1px 3px rgba(0,0,0,0.8);
  }

  /* 3. Piano Sheet Modal */
  .sheet-modal {
    width: 960px;
    background: #fdfaf0;
    box-shadow: 0 15px 40px rgba(0,0,0,0.6);
  }
  .sheet-paper {
    display: flex;
    flex-direction: row;
    width: 100%;
    padding: 40px 30px;
  }
  .sheet-page {
    flex: 1;
    padding: 0 25px;
    display: flex;
    flex-direction: column;
    align-items: center;
  }
  .sheet-page.left-page {
    border-right: 1px dashed rgba(0, 0, 0, 0.12);
  }
  .sheet-title {
    font-family: 'Special Elite', monospace;
    font-size: 1.3rem;
    color: #111;
    margin-bottom: 20px;
    text-align: center;
    letter-spacing: 1px;
    text-transform: uppercase;
  }
  .embed-container {
    width: 100%;
    display: flex;
    justify-content: center;
  }

  /* Responsive sheets */
  @media (max-width: 768px) {
    .phone-modal {
      flex-direction: column;
      width: 90%;
    }
    .polaroid-frame {
      align-self: center;
      margin: 20px 0 10px 0;
    }
    .phone-call-info {
      padding: 20px;
      text-align: center;
    }
    .sheet-paper {
      flex-direction: column;
      gap: 30px;
      padding: 30px 15px;
    }
    .sheet-page.left-page {
      border-right: none;
      border-bottom: 1px dashed rgba(0,0,0,0.12);
      padding-bottom: 30px;
    }
    .poem-modal {
      width: 95%;
      padding: 25px 15px;
    }
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
