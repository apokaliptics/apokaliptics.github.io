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
  let cameraMode = $state<'menu' | 'free' | 'poem' | 'piano' | 'telephone' | 'tv'>('free');
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
  let albumLights = $state(true);
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

  // Album plaque states
  let hoveredAlbum = $state<any | null>(null);
  let selectedAlbum = $state<any | null>(null);

  // Room Element visibility states
  let showAlbums = $state(true);
  let showCouchAndDesk = $state(true);
  let showTV = $state(true);
  let showTelephone = $state(true);
  let showPiano = $state(true);

  // Panel drawer states
  let activePanel = $state<'none' | 'tv' | 'lights' | 'elements'>('none');
  let showCreditModal = $state(false);

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

      // In free camera mode, let WASD and arrow keys control the camera instead of switching modes
      if (cameraMode === 'free') {
        const key = e.key.toLowerCase();
        if (['w', 'a', 's', 'd', 'arrowup', 'arrowdown', 'arrowleft', 'arrowright'].includes(key)) {
          return;
        }
      }

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
    cameraMode = 'menu';
  }

  function actionEnterRoom() {
    cameraMode = 'free';
    playMusic();
  }

  function actionFreeView() {
    cameraMode = 'free';
    playMusic();
  }

  function actionShowCredit() {
    showCreditModal = true;
  }

  function togglePanel(panel: 'tv' | 'lights' | 'elements') {
    if (activePanel === panel) {
      activePanel = 'none';
    } else {
      activePanel = panel;
    }
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

  const cameras: Array<'menu' | 'free' | 'poem' | 'piano' | 'telephone' | 'tv'> = ['free', 'poem', 'piano', 'telephone', 'tv'];

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
{:else}
  <!-- svelte-ignore a11y_click_events_have_key_events -->
  <!-- svelte-ignore a11y_no_static_element_interactions -->
  <div class="app-main" transition:fade={{ duration: 400 }} onclick={() => { selectedAlbum = null; }}>
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
      {albumLights}
      onAlbumHover={(album) => hoveredAlbum = album}
      onAlbumClick={(album) => {
        if (selectedAlbum?.title === album.title) {
          selectedAlbum = null;
        } else {
          selectedAlbum = album;
        }
      }}
      {showAlbums}
      {showCouchAndDesk}
      {showTV}
      {showTelephone}
      {showPiano}
    />

    {#if hoveredAlbum || selectedAlbum}
      {@const activeAlbum = hoveredAlbum || selectedAlbum}
      <!-- svelte-ignore a11y_click_events_have_key_events -->
      <!-- svelte-ignore a11y_no_static_element_interactions -->
      <div class="album-plaque-overlay" transition:fade={{ duration: 200 }} onclick={(e) => e.stopPropagation()}>
        {#if activeAlbum === selectedAlbum}
          <button class="plaque-close-btn" onclick={() => selectedAlbum = null} aria-label="Close Plaque">&times;</button>
        {/if}
        <img class="plaque-cover-img" src={`${baseUrl}pink floyd albums/${activeAlbum.fileName}`} alt={activeAlbum.title} />
        <div class="plaque-title">{activeAlbum.title}</div>
        <div class="plaque-year">{activeAlbum.year}</div>
        <div class="plaque-desc">{activeAlbum.description}</div>
        {#if activeAlbum !== selectedAlbum}
          <div class="plaque-hint">Click album to lock display</div>
        {/if}
      </div>
    {/if}

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

    <!-- Game Start Menu Overlay -->
    {#if cameraMode === 'menu'}
      <!-- svelte-ignore a11y_click_events_have_key_events -->
      <!-- svelte-ignore a11y_no_static_element_interactions -->
      <div class="game-menu-overlay" transition:fade={{ duration: 300 }} onclick={(e) => e.stopPropagation()}>
        <div class="game-menu-card">
          <h1 class="game-title">is there anybody out there?</h1>
          <p class="game-subtitle">Welcome to apokaliptics' personal wall</p>
          
          <div class="menu-options">
            <button class="menu-btn" onclick={actionEnterRoom}>
              <span class="btn-arrow">&gt;</span> ENTER ROOM
            </button>
            <button class="menu-btn" onclick={actionFreeView}>
              <span class="btn-arrow">&gt;</span> FREE VIEW
            </button>
            <button class="menu-btn" onclick={actionShowCredit}>
              <span class="btn-arrow">&gt;</span> CREDITS
            </button>
          </div>
        </div>
      </div>

      {#if showCreditModal}
        <!-- svelte-ignore a11y_click_events_have_key_events -->
        <!-- svelte-ignore a11y_no_static_element_interactions -->
        <div class="menu-credits-overlay" transition:fade={{ duration: 250 }} onclick={() => showCreditModal = false}>
          <div class="credits-card" onclick={(e) => e.stopPropagation()}>
            <button class="close-credits-btn" onclick={() => showCreditModal = false}>&times;</button>
            <h2 class="credits-title">PROJECT CREDITS</h2>
            <div class="credits-content">
              <p class="credits-author">CREATED BY KIET MINH</p>
              <p class="credits-target">TARGET: APOKALIPTICS</p>
              <p class="credits-dedication">Inspired by Pink Floyd's "The Wall" (1979) and the track "Nobody Home". Built with Svelte 5, Threlte, and Three.js.</p>
            </div>
          </div>
        </div>
      {/if}
    {/if}

    {#if cameraMode !== 'menu'}
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

      <!-- Unified Slide-out HUD panels on the right edge -->
      <!-- svelte-ignore a11y_click_events_have_key_events -->
      <!-- svelte-ignore a11y_no_static_element_interactions -->
      <div class="hud-drawer-container" onclick={(e) => e.stopPropagation()}>
        <!-- Tab Columns -->
        <div class="hud-tabs-column">
          <button class="hud-tab-btn" class:active={activePanel === 'tv'} onclick={() => togglePanel('tv')} title="TV Control Dashboard">
            📺 <span class="tab-label">TV</span>
          </button>
          <button class="hud-tab-btn" class:active={activePanel === 'lights'} onclick={() => togglePanel('lights')} title="Electric Lights Switchboard">
            💡 <span class="tab-label">LIGHTS</span>
          </button>
          <button class="hud-tab-btn" class:active={activePanel === 'elements'} onclick={() => togglePanel('elements')} title="Scene Elements Toggles">
            👁️ <span class="tab-label">OBJECTS</span>
          </button>
        </div>

        <!-- Drawer Content Container -->
        <div class="hud-drawer-content" class:open={activePanel !== 'none'}>
          {#if activePanel === 'tv'}
            <div class="panel-content tv-panel">
              <h3 class="panel-title">TV DASHBOARD</h3>
              
              <div class="panel-row">
                <!-- Power Switch -->
                <div class="panel-control-group">
                  <label for="hud-power-switch">POWER</label>
                  <button id="hud-power-switch" class="tv-power-switch" class:on={tvPower} onclick={toggleTVPower} aria-label="Toggle TV Power">
                    <span class="switch-knob"></span>
                  </button>
                </div>

                <!-- Channel display -->
                <div class="panel-control-group">
                  <label for="hud-channel-screen">CHANNEL</label>
                  <div id="hud-channel-screen" class="tv-channel-display">{tvPower ? (tvChannel < 10 ? '0' + tvChannel : tvChannel) : '--'}</div>
                </div>

                <!-- Knob dial -->
                <div class="panel-control-group">
                  <label for="hud-knob">DIAL</label>
                  <button id="hud-knob" class="tv-knob" style="transform: rotate({knobRotation}deg)" onclick={rotateChannel} aria-label="Rotate TV Channel">
                    <span class="knob-line"></span>
                  </button>
                </div>
              </div>

              <!-- Quick select channel grids -->
              <div class="panel-channel-grid">
                {#each Array.from({ length: 13 }, (_, idx) => idx + 1) as chanNum (chanNum)}
                  <button class="panel-channel-btn" class:active={tvChannel === chanNum && tvPower} onclick={() => setChannel(chanNum)}>
                    {chanNum}
                  </button>
                {/each}
              </div>
            </div>

          {:else if activePanel === 'lights'}
            <div class="panel-content lights-panel">
              <h3 class="panel-title">ELECTRIC LIGHTS</h3>
              
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

                <!-- Album Spotlight -->
                <div class="light-row">
                  <span class="light-label">ALBUM LIGHTS</span>
                  <div class="light-control-wrapper">
                    <span class="light-led" class:on={albumLights}></span>
                    <button class="light-switch" class:on={albumLights} onclick={() => albumLights = !albumLights} aria-label="Toggle Album Lights">
                      <span class="switch-knob"></span>
                    </button>
                  </div>
                </div>
              </div>
            </div>

          {:else if activePanel === 'elements'}
            <div class="panel-content elements-panel">
              <h3 class="panel-title">ROOM OBJECTS</h3>
              
              <div class="elements-list-container">
                <label class="element-checkbox-row">
                  <input type="checkbox" bind:checked={showAlbums}>
                  <span class="checkbox-custom"></span>
                  <span class="checkbox-label">ALBUMS DISPLAY</span>
                </label>
                
                <label class="element-checkbox-row">
                  <input type="checkbox" bind:checked={showCouchAndDesk}>
                  <span class="checkbox-custom"></span>
                  <span class="checkbox-label">COUCH & TABLE</span>
                </label>
                
                <label class="element-checkbox-row">
                  <input type="checkbox" bind:checked={showTV}>
                  <span class="checkbox-custom"></span>
                  <span class="checkbox-label">TV CREDENZA</span>
                </label>
                
                <label class="element-checkbox-row">
                  <input type="checkbox" bind:checked={showTelephone}>
                  <span class="checkbox-custom"></span>
                  <span class="checkbox-label">TELEPHONE BOOTH</span>
                </label>
                
                <label class="element-checkbox-row">
                  <input type="checkbox" bind:checked={showPiano}>
                  <span class="checkbox-custom"></span>
                  <span class="checkbox-label">GRAND PIANO</span>
                </label>
              </div>
            </div>
          {/if}
        </div>
      </div>
    {/if}

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

  /* Album Plaque 2D HUD Card */
  .album-plaque-overlay {
    position: absolute;
    top: 110px;
    right: 25px;
    z-index: 20;
    width: 320px;
    box-sizing: border-box;
    padding: 18px;
    background: rgba(16, 14, 12, 0.94);
    border: 2px solid #c5a059;
    border-radius: 6px;
    color: #f5f0e0;
    font-family: 'Space Grotesk', sans-serif;
    text-align: center;
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.95);
    backdrop-filter: blur(8px);
    pointer-events: auto;
    animation: slide-in 0.25s ease-out;
  }

  .album-plaque-overlay .plaque-cover-img {
    width: 140px;
    height: 140px;
    object-fit: cover;
    margin: 0 auto 12px auto;
    display: block;
    border: 1px solid rgba(197, 160, 89, 0.4);
    box-shadow: 0 4px 10px rgba(0,0,0,0.5);
  }

  @keyframes slide-in {
    from {
      opacity: 0;
      transform: translateY(-10px);
    }
    to {
      opacity: 1;
      transform: translateY(0);
    }
  }

  .album-plaque-overlay .plaque-title {
    font-size: 1.1rem;
    font-weight: bold;
    color: #ffd700;
    margin-bottom: 6px;
    text-transform: uppercase;
    letter-spacing: 1.2px;
    line-height: 1.3;
    border-bottom: 1px double rgba(197, 160, 89, 0.5);
    padding-bottom: 8px;
    margin-top: 5px;
  }

  .album-plaque-overlay .plaque-year {
    font-size: 0.85rem;
    color: #bfa883;
    margin-bottom: 12px;
    font-style: italic;
  }

  .album-plaque-overlay .plaque-desc {
    font-size: 0.75rem;
    line-height: 1.5;
    color: #e3dec9;
    text-align: left;
    margin-bottom: 8px;
  }

  .album-plaque-overlay .plaque-hint {
    font-size: 0.65rem;
    color: #888;
    margin-top: 10px;
    text-transform: uppercase;
    letter-spacing: 0.5px;
    border-top: 1px dashed rgba(255, 255, 255, 0.1);
    padding-top: 8px;
  }

  .album-plaque-overlay .plaque-close-btn {
    position: absolute;
    top: 6px;
    right: 12px;
    background: none;
    border: none;
    color: #c5a059;
    font-size: 1.6rem;
    cursor: pointer;
    line-height: 1;
    padding: 0;
    transition: color 0.2s;
  }

  .album-plaque-overlay .plaque-close-btn:hover {
    color: #ffd700;
  }

  /* Game menu styles */
  .game-menu-overlay {
    position: absolute;
    top: 0; left: 0; width: 100vw; height: 100vh;
    z-index: 100;
    display: flex;
    justify-content: flex-end;
    align-items: center;
    background: transparent;
    pointer-events: none;
  }

  .game-menu-card {
    background: transparent;
    border: none;
    padding: 40px 80px;
    text-align: right;
    box-shadow: none;
    max-width: 600px;
    width: auto;
    box-sizing: border-box;
    font-family: 'Special Elite', monospace;
    color: #f4f0e8;
    pointer-events: auto;
  }

  .game-title {
    font-family: 'Permanent Marker', cursive;
    font-size: clamp(2rem, 4vw, 3rem);
    color: #cc0000;
    text-transform: lowercase;
    margin: 0 0 10px 0;
    letter-spacing: 2px;
    text-shadow: 0 4px 12px rgba(0, 0, 0, 0.9);
    text-align: right;
  }

  .game-subtitle {
    font-family: 'Cutive Mono', monospace;
    font-size: 0.9rem;
    color: #888;
    margin-bottom: 35px;
    text-transform: uppercase;
    letter-spacing: 1px;
    text-align: right;
  }

  .menu-options {
    display: flex;
    flex-direction: column;
    gap: 18px;
    align-items: flex-end;
  }

  .menu-btn {
    background: transparent;
    border: none;
    color: #888;
    font-family: 'Special Elite', monospace;
    font-size: 1.25rem;
    cursor: pointer;
    display: flex;
    align-items: center;
    gap: 12px;
    transition: all 0.25s ease;
    outline: none;
    text-shadow: 2px 2px 0px #000;
  }

  .menu-btn .btn-arrow {
    opacity: 0;
    transform: translateX(-5px);
    transition: all 0.2s ease;
    color: #cc0000;
  }

  .menu-btn:hover {
    color: #cc0000;
    transform: scale(1.05);
  }

  .menu-btn:hover .btn-arrow {
    opacity: 1;
    transform: translateX(0);
  }

  /* Credits overlay in menu */
  .menu-credits-overlay {
    position: absolute;
    top: 0; left: 0; width: 100vw; height: 100vh;
    z-index: 110;
    display: grid;
    place-items: center;
    background: rgba(0, 0, 0, 0.85);
    backdrop-filter: blur(8px);
  }

  .credits-card {
    background: rgba(16, 14, 12, 0.96);
    border: 2px solid #c5a059;
    border-radius: 6px;
    padding: 40px 35px;
    max-width: 450px;
    width: 90%;
    box-sizing: border-box;
    position: relative;
    text-align: center;
    color: #f5f0e0;
    font-family: 'Cutive Mono', monospace;
    box-shadow: 0 15px 40px rgba(0, 0, 0, 0.95);
  }

  .credits-title {
    font-family: 'Special Elite', monospace;
    font-size: 1.5rem;
    color: #ffd700;
    margin-top: 0;
    margin-bottom: 25px;
    border-bottom: 1px double rgba(197, 160, 89, 0.5);
    padding-bottom: 10px;
    letter-spacing: 1.5px;
  }

  .credits-author {
    font-size: 1.1rem;
    color: #fff;
    margin-bottom: 5px;
    font-weight: bold;
  }

  .credits-target {
    font-size: 0.9rem;
    color: #bfa883;
    margin-bottom: 20px;
  }

  .credits-dedication {
    font-size: 0.8rem;
    line-height: 1.6;
    color: #e3dec9;
    text-align: justify;
    margin-bottom: 0;
  }

  .close-credits-btn {
    position: absolute;
    top: 15px;
    right: 20px;
    font-size: 2rem;
    color: #c5a059;
    background: none;
    border: none;
    cursor: pointer;
    line-height: 1;
    padding: 0;
  }

  .close-credits-btn:hover {
    color: #ffd700;
  }

  /* HUD drawer container and slide-out panel layout on the right */
  .hud-drawer-container {
    position: absolute;
    right: 0;
    top: 430px;
    bottom: 60px;
    z-index: 30;
    display: flex;
    align-items: flex-start;
    pointer-events: none;
  }

  .hud-tabs-column {
    display: flex;
    flex-direction: column;
    gap: 12px;
    pointer-events: auto;
    background: rgba(10, 10, 10, 0.88);
    border: 2px solid #333;
    border-right: none;
    border-radius: 8px 0 0 8px;
    padding: 12px 6px;
    box-shadow: -5px 5px 15px rgba(0,0,0,0.5);
  }

  .hud-tab-btn {
    background: transparent;
    border: none;
    color: #888;
    cursor: pointer;
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 4px;
    font-size: 1.2rem;
    padding: 8px;
    transition: all 0.2s ease;
    border-radius: 4px;
  }

  .hud-tab-btn:hover, .hud-tab-btn.active {
    color: #cc0000;
    background: rgba(255, 255, 255, 0.05);
  }

  .hud-tab-btn .tab-label {
    font-family: 'Special Elite', monospace;
    font-size: 0.6rem;
    font-weight: bold;
    letter-spacing: 0.5px;
  }

  .hud-drawer-content {
    width: 0;
    opacity: 0;
    overflow: hidden;
    height: 100%;
    transition: width 0.3s cubic-bezier(0.19, 1, 0.22, 1), opacity 0.2s ease;
    background: linear-gradient(135deg, #1b1b1b 0%, #0d0d0d 100%);
    border: 0 solid #000;
    border-radius: 0 0 0 8px;
    box-shadow: -8px 8px 20px rgba(0,0,0,0.7), inset 1px 1px 2px rgba(255,255,255,0.1);
  }

  .hud-drawer-content.open {
    width: 280px;
    opacity: 1;
    border-left: 4px solid #000;
    border-bottom: 4px solid #000;
    border-top: 4px solid #000;
    pointer-events: auto;
  }

  .panel-content {
    padding: 20px;
    height: 100%;
    box-sizing: border-box;
    display: flex;
    flex-direction: column;
  }

  .panel-title {
    font-family: 'Special Elite', monospace;
    font-size: 1rem;
    color: #666;
    margin: 0 0 18px 0;
    border-bottom: 2px double #333;
    padding-bottom: 5px;
    text-align: center;
    letter-spacing: 1px;
  }

  .panel-row {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 18px;
    gap: 12px;
  }

  .panel-control-group {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 5px;
  }

  .panel-control-group label {
    font-family: 'Special Elite', monospace;
    font-size: 0.75rem;
    color: #555;
  }

  .panel-channel-grid {
    display: grid;
    grid-template-columns: repeat(5, 1fr);
    gap: 5px;
    border-top: 2px solid #222;
    padding-top: 15px;
  }

  .panel-channel-btn {
    font-family: 'Special Elite', monospace;
    font-size: 0.75rem;
    color: #777;
    background: #0f0f0f;
    border: 1px solid #222;
    padding: 4px 0;
    cursor: pointer;
    transition: all 0.2s;
  }

  .panel-channel-btn:hover {
    color: #fff;
    background: #1a1a1a;
  }

  .panel-channel-btn.active {
    color: #fff;
    background: #cc0000;
    border-color: #ff3333;
    text-shadow: 0 0 3px #fff;
  }

  /* Room Elements Visibility Toggles styling */
  .elements-list-container {
    display: flex;
    flex-direction: column;
    gap: 16px;
    padding-top: 5px;
  }

  .element-checkbox-row {
    display: flex;
    align-items: center;
    gap: 12px;
    cursor: pointer;
    font-family: 'Special Elite', monospace;
    font-size: 0.8rem;
    color: #aaa;
    transition: color 0.2s;
    user-select: none;
  }

  .element-checkbox-row:hover {
    color: #f4f0e8;
  }

  .element-checkbox-row input {
    display: none;
  }

  .checkbox-custom {
    width: 18px;
    height: 18px;
    border: 2px solid #444;
    border-radius: 3px;
    background: #050505;
    position: relative;
    transition: all 0.2s;
  }

  .element-checkbox-row input:checked + .checkbox-custom {
    border-color: #cc0000;
    background: rgba(204, 0, 0, 0.2);
    box-shadow: 0 0 8px rgba(204, 0, 0, 0.4);
  }

  .element-checkbox-row input:checked + .checkbox-custom::after {
    content: '';
    position: absolute;
    left: 5px;
    top: 1px;
    width: 5px;
    height: 10px;
    border: solid #cc0000;
    border-width: 0 2.5px 2.5px 0;
    transform: rotate(45deg);
  }

  .checkbox-label {
    letter-spacing: 0.5px;
  }

  @media (max-width: 900px) {
    .hud-drawer-container {
      top: auto;
      bottom: 25px;
      right: 0;
      left: 0;
      flex-direction: column-reverse;
      align-items: stretch;
      bottom: 12px;
    }
    .hud-tabs-column {
      flex-direction: row;
      border-radius: 8px 8px 0 0;
      border: 2px solid #333;
      border-bottom: none;
      justify-content: space-around;
      padding: 6px 12px;
    }
    .hud-drawer-content.open {
      width: 100%;
      height: 200px;
      border-left: 2px solid #000;
      border-right: 2px solid #000;
      border-bottom: 2px solid #000;
      border-top: none;
      border-radius: 0;
    }
    .panel-channel-grid {
      grid-template-columns: repeat(7, 1fr);
      padding-top: 10px;
    }
    .album-plaque-overlay {
      top: auto;
      bottom: 230px;
      right: 25px;
      left: 25px;
      width: calc(100% - 50px);
      box-shadow: 0 -5px 20px rgba(0,0,0,0.5);
    }
  }
</style>
