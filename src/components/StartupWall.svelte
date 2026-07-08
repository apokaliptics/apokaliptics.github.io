<!--
  StartupWall.svelte — replaces src-react/components/StartupWall.tsx
  
  Brick masonry animation → logo impact → shake → collapse → handoff.
  Uses Svelte transitions instead of framer-motion.
-->
<script lang="ts">
	import { onMount } from 'svelte';
	import { cubicOut } from 'svelte/easing';
	import { fade, scale } from 'svelte/transition';

	type Props = {
		onComplete: () => void;
	};

	let { onComplete }: Props = $props();

	// Assets via Vite URL imports
	const brickTexturePng = '/brick-texture.png';
	const brickLogoPng = '/brick-logo.png';

	// Animation states
	let assetsLoaded = $state(false);
	let showLogo = $state(false);
	let showParticles = $state(false);
	let collapseBricks = $state(false);
	let bgTransparent = $state(false);
	let shaking = $state(false);

	// Audio status
	let audioStatus = $state<{
		hasBitPerfectDevice: boolean;
		deviceName: string | null;
		strictMode: boolean;
	}>({ hasBitPerfectDevice: false, deviceName: null, strictMode: false });

	// Window size
	let innerWidth = $state(typeof window !== 'undefined' ? window.innerWidth : 1920);
	let innerHeight = $state(typeof window !== 'undefined' ? window.innerHeight : 1080);

	// Brick layout computed from window size
	const brickLayout = $derived.by(() => {
		const brickWidth = 180;
		const brickHeight = 70;
		const gap = 3;

		const cols = Math.ceil(innerWidth / (brickWidth + gap)) + 1;
		const rows = Math.ceil(innerHeight / (brickHeight + gap)) + 1;

		const stoneColors = [
			'#5a4a3a', '#6b5b4b', '#7c6c5c', '#8d7d6d',
			'#4a3a2a', '#5b4b3b', '#6c5c4c',
		];

		const bricks: Array<{
			id: number;
			x: number;
			y: number;
			width: number;
			height: number;
			color: string;
			delay: number;
			collapseX: number;
			collapseY: number;
			collapseDelay: number;
		}> = [];

		const centerX = innerWidth / 2;
		const centerY = innerHeight / 2;
		let id = 0;

		for (let row = rows - 1; row >= 0; row--) {
			const rowFromBottom = rows - 1 - row;
			const isLeftToRight = rowFromBottom % 2 === 0;

			for (let col = 0; col < cols; col++) {
				const offsetX = row % 2 === 0 ? 0 : -(brickWidth / 2);
				const actualCol = isLeftToRight ? col : cols - 1 - col;
				const brickIndex = rowFromBottom * cols + col;
				const x = actualCol * (brickWidth + gap) + offsetX;
				const y = row * (brickHeight + gap);

				const brickCenterX = x + brickWidth / 2;
				const brickCenterY = y + brickHeight / 2;
				const distFromCenter = Math.sqrt(
					(brickCenterX - centerX) ** 2 + (brickCenterY - centerY) ** 2
				);
				const angle = Math.atan2(brickCenterY - centerY, brickCenterX - centerX);

				bricks.push({
					id: id++,
					x, y, width: brickWidth, height: brickHeight,
					color: stoneColors[Math.floor(Math.random() * stoneColors.length)],
					delay: brickIndex * 0.008,
					collapseX: Math.cos(angle) * (300 + distFromCenter * 0.5),
					collapseY: Math.sin(angle) * (300 + distFromCenter * 0.5),
					collapseDelay: (distFromCenter / 1000) * 0.3,
				});
			}
		}

		return bricks;
	});

	// Dust particles
	const particles = Array.from({ length: 12 }, (_, i) => ({
		id: i,
		angle: (360 / 12) * i,
		distance: 100 + Math.random() * 100,
		size: 4 + Math.random() * 8,
	}));

	// Preload images
	function preloadImages(urls: string[]): Promise<void> {
		return Promise.all(
			urls.map(
				(url) =>
					new Promise<void>((resolve) => {
						const img = new Image();
						img.onload = () => resolve();
						img.onerror = () => resolve();
						img.src = url;
					})
			)
		).then(() => {});
	}

	// Check device capabilities
	async function checkDeviceCapabilities() {
		return { hasBitPerfectDevice: false, deviceName: null, strictMode: false };
	}

	onMount(() => {
		const init = async () => {
			const [, deviceInfo] = await Promise.all([
				preloadImages([brickTexturePng, brickLogoPng]),
				checkDeviceCapabilities(),
			]);
			audioStatus = deviceInfo;
			assetsLoaded = true;

			// Phase 1: Masonry (bricks animate in via CSS)
			await new Promise((r) => setTimeout(r, 1500));

			// Phase 2: Logo impact
			showLogo = true;
			await new Promise((r) => setTimeout(r, 300));

			// Phase 3: Shake
			shaking = true;
			await new Promise((r) => setTimeout(r, 200));
			shaking = false;

			// Phase 4: Collapse
			showParticles = true;
			bgTransparent = true;
			collapseBricks = true;
			await new Promise((r) => setTimeout(r, 800));
			onComplete();
		};

		void init();
	});
</script>

<svelte:window bind:innerWidth bind:innerHeight />

{#if !assetsLoaded}
	<!-- Loading placeholder -->
	<div class="fixed inset-0 bg-black" style="z-index: 9999;"></div>
{:else}
	<!-- Main wall container -->
	<div
		class="fixed inset-0 overflow-hidden"
		class:startup-shake={shaking}
		style="perspective: 1000px; z-index: 9999;"
	>
		<!-- Solid black background -->
		<div
			class="absolute inset-0 transition-opacity duration-300"
			class:opacity-0={bgTransparent}
			style="background-color: #000000;"
		></div>

		<!-- Brick masonry layer -->
		<div class="absolute inset-0" style="z-index: 1;">
			{#each brickLayout as brick (brick.id)}
				<div
					class="absolute brick-element"
					class:brick-collapse={collapseBricks}
					style="
						left: {brick.x}px;
						top: {brick.y}px;
						width: {brick.width}px;
						height: {brick.height}px;
						background-color: {brick.color};
						background-image: url({brickTexturePng});
						background-size: cover;
						background-position: center;
						box-shadow: inset 0 1px 2px rgba(0,0,0,0.3), inset 0 -1px 2px rgba(255,255,255,0.05);
						border: 1px solid rgba(0,0,0,0.4);
						animation-delay: {brick.delay}s;
						--collapse-x: {brick.collapseX}px;
						--collapse-y: {brick.collapseY}px;
						--collapse-delay: {brick.collapseDelay}s;
						--collapse-rotate: {Math.random() * 360 - 180}deg;
					"
				></div>
			{/each}
		</div>

		<!-- Logo impact -->
		{#if showLogo}
			<div
				class="absolute inset-0 flex items-center justify-center"
				class:logo-collapse={collapseBricks}
				style="transform-style: preserve-3d; z-index: 2;"
				in:scale={{ duration: 300, start: 1.6, easing: cubicOut }}
			>
				<div class="relative">
					<img
						src={brickLogoPng}
						alt="BRICK"
						class="w-96 h-96 object-contain"
						style="filter: drop-shadow(0 20px 40px rgba(0,0,0,0.8)) drop-shadow(0 0 20px rgba(0,0,0,0.5));"
					/>
					<!-- Audio engine status -->
					<div
						class="absolute left-1/2 -translate-x-1/2 text-center"
						style="top: calc(100% + 20px); white-space: nowrap;"
						in:fade={{ delay: 300, duration: 300 }}
					>
						{#if audioStatus.strictMode && !audioStatus.hasBitPerfectDevice}
							<div class="text-amber-400 text-sm font-medium">
								<span
									class="inline-block w-2 h-2 bg-amber-400 rounded-full mr-2 animate-pulse"
								></span>
								No bit-perfect capable device detected
							</div>
						{:else if audioStatus.hasBitPerfectDevice}
							<div class="text-emerald-400 text-sm opacity-70">
								<span class="inline-block w-2 h-2 bg-emerald-400 rounded-full mr-2"></span>
								<span style="font-family: 'Chakra Petch', 'Syne', sans-serif; font-weight: 700;">
									{audioStatus.deviceName ?? 'Audio Ready'}
								</span>
							</div>
						{:else}
							<div class="text-gray-400 text-sm opacity-50">
								<span class="inline-block w-2 h-2 bg-gray-400 rounded-full mr-2"></span>
								<span style="font-family: 'Chakra Petch', 'Syne', sans-serif; font-weight: 700;"
									>Audio Ready</span
								>
							</div>
						{/if}
					</div>
				</div>
			</div>
		{/if}

		<!-- Dust particles -->
		{#if showParticles}
			{#each particles as particle (particle.id)}
				{@const radians = (particle.angle * Math.PI) / 180}
				<div
					class="absolute bg-gray-400 rounded-full particle-burst"
					style="
						left: 50%;
						top: 50%;
						width: {particle.size}px;
						height: {particle.size}px;
						box-shadow: 0 0 4px rgba(255,255,255,0.5);
						--end-x: {Math.cos(radians) * particle.distance}px;
						--end-y: {Math.sin(radians) * particle.distance}px;
					"
				></div>
			{/each}
		{/if}
	</div>
{/if}

<style>
	/* Brick entrance animation */
	.brick-element {
		animation: brick-fall 0.4s cubic-bezier(0.4, 0, 0.2, 1) both;
	}

	@keyframes brick-fall {
		from {
			opacity: 1;
			transform: translateY(-100px) scale(0.9);
		}
		to {
			opacity: 1;
			transform: translateY(0) scale(1);
		}
	}

	/* Brick collapse animation */
	.brick-collapse {
		animation: brick-explode 0.8s cubic-bezier(0.6, 0.01, 0.05, 0.95) both !important;
		animation-delay: var(--collapse-delay) !important;
	}

	@keyframes brick-explode {
		to {
			transform: translate(var(--collapse-x), var(--collapse-y)) scale(0.3)
				rotate(var(--collapse-rotate));
			opacity: 0;
		}
	}

	/* Logo collapse */
	.logo-collapse {
		animation: logo-shrink 0.4s ease-in forwards;
	}

	@keyframes logo-shrink {
		to {
			transform: scale(0.8);
			opacity: 0;
		}
	}

	/* Screen shake */
	.startup-shake {
		animation: wall-shake 0.2s ease-out;
	}

	@keyframes wall-shake {
		0% { transform: translate(0, 0); }
		15% { transform: translate(-8px, 6px); }
		30% { transform: translate(6px, -4px); }
		50% { transform: translate(-6px, 5px); }
		70% { transform: translate(4px, -3px); }
		85% { transform: translate(-2px, 2px); }
		100% { transform: translate(0, 0); }
	}

	/* Dust particles burst */
	.particle-burst {
		animation: particle-fly 0.6s ease-out forwards;
	}

	@keyframes particle-fly {
		from {
			transform: translate(0, 0) scale(1);
			opacity: 1;
		}
		to {
			transform: translate(var(--end-x), var(--end-y)) scale(0);
			opacity: 0;
		}
	}
</style>
