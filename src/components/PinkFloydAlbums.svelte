<script lang="ts">
  import { T } from "@threlte/core";
  import { HTML } from "@threlte/extras";
  import * as THREE from "three";

  interface Props {
    albumLights?: boolean;
    onAlbumHover?: (album: any | null) => void;
    onAlbumClick?: (album: any) => void;
  }
  let { albumLights = true, onAlbumHover, onAlbumClick }: Props = $props();

  const baseUrl = import.meta.env.BASE_URL || "/";

  interface Album {
    title: string;
    year: number;
    description: string;
    fileName: string;
    angle: number;
    radius: number;
    yOffset: number;
    tiltAngle: number;
  }

  // 11 Pink Floyd albums distributed around the circular platform (Radius = 5.0)
  // We place them at radius ~5.5 - 5.8 with slight variations to look organically scattered.
  const albums: Album[] = [
    {
      title: "The Piper at the Gates of Dawn",
      year: 1967,
      description:
        "Pink Floyd's debut album, led by Syd Barrett, showcasing whimsical, psychedelic rock and experimental pop.",
      fileName: "the piper at the gates of dawn.jpg",
      angle: (0 * 2 * Math.PI) / 11 + 0.05,
      radius: 5.65,
      yOffset: -0.05,
      tiltAngle: -Math.PI / 12,
    },
    {
      title: "A Saucerful of Secrets",
      year: 1968,
      description:
        "The transition album introducing David Gilmour, moving toward space-rock and avant-garde soundscapes.",
      fileName: "saucerful of secrets.jpg",
      angle: (1 * 2 * Math.PI) / 11 - 0.03,
      radius: 5.5,
      yOffset: -0.08,
      tiltAngle: -Math.PI / 10,
    },
    {
      title: "More",
      year: 1969,
      description:
        "Soundtrack to the film of the same name, blending acoustic folk, heavy rock, and atmospheric instrumentals.",
      fileName: "more.jpg",
      angle: (2 * 2 * Math.PI) / 11 + 0.02,
      radius: 5.75,
      yOffset: -0.02,
      tiltAngle: -Math.PI / 11,
    },
    {
      title: "Ummagumma",
      year: 1969,
      description:
        "A double album featuring a live set of early classics alongside solo avant-garde studio pieces from each band member.",
      fileName: "ummagumma.jpg",
      angle: (3 * 2 * Math.PI) / 11 - 0.04,
      radius: 5.6,
      yOffset: -0.06,
      tiltAngle: -Math.PI / 13,
    },
    {
      title: "Atom Heart Mother",
      year: 1970,
      description:
        "Known for its iconic cow cover and the epic, orchestral title suite, marking progress towards symphonic rock.",
      fileName: "atom heart mother.jpeg",
      angle: (4 * 2 * Math.PI) / 11 + 0.04,
      radius: 5.8,
      yOffset: -0.04,
      tiltAngle: -Math.PI / 9,
    },
    {
      title: "Meddle",
      year: 1971,
      description:
        "The album where the band found their signature sound, highlighted by the side-long sonic masterpiece 'Echoes'.",
      fileName: "meddle.webp",
      angle: (5 * 2 * Math.PI) / 11 - 0.02,
      radius: 5.55,
      yOffset: -0.07,
      tiltAngle: -Math.PI / 12,
    },
    {
      title: "The Dark Side of the Moon",
      year: 1973,
      description:
        "One of the best-selling and most critically acclaimed albums of all time, exploring themes of madness, greed, time, and death.",
      fileName: "the dark side of the moon.jpg",
      angle: (6 * 2 * Math.PI) / 11 + 0.03,
      radius: 5.7,
      yOffset: -0.03,
      tiltAngle: -Math.PI / 11,
    },
    {
      title: "Wish You Were Here",
      year: 1975,
      description:
        "A poignant tribute to former band leader Syd Barrett, featuring 'Shine On You Crazy Diamond' and the title track.",
      fileName: "wish you were here.png",
      angle: (7 * 2 * Math.PI) / 11 - 0.05,
      radius: 5.65,
      yOffset: -0.05,
      tiltAngle: -Math.PI / 10,
    },
    {
      title: "Animals",
      year: 1977,
      description:
        "A scathing socio-political concept album loosely based on Orwell's Animal Farm, with long, guitar-driven suites.",
      fileName: "animals.jpg",
      angle: (8 * 2 * Math.PI) / 11 + 0.01,
      radius: 5.5,
      yOffset: -0.09,
      tiltAngle: -Math.PI / 12,
    },
    {
      title: "The Wall",
      year: 1979,
      description:
        "A monumental rock opera exploring isolation, trauma, and personal barriers, built around the character Pink. (It is my personal favourite.)",
      fileName: "the wall.jpg",
      angle: (9 * 2 * Math.PI) / 11 - 0.03,
      radius: 5.8,
      yOffset: -0.02,
      tiltAngle: -Math.PI / 11,
    },
    {
      title: "The Final Cut",
      year: 1983,
      description:
        "Roger Waters' anti-war concept album, dedicated to his father, serves as a thematic epilogue to The Wall.",
      fileName: "the final cut.jpg",
      angle: (10 * 2 * Math.PI) / 11 + 0.05,
      radius: 5.6,
      yOffset: -0.06,
      tiltAngle: -Math.PI / 10,
    },
  ];

  const textureLoader = new THREE.TextureLoader();

  // Load all textures and pre-compute 3D transformation values
  const preparedAlbums = albums.map((album) => {
    const texturePath = `${baseUrl}pink floyd albums/${album.fileName}`;
    const texture = textureLoader.load(texturePath);
    texture.colorSpace = THREE.SRGBColorSpace;

    // Calculate position
    const x = album.radius * Math.cos(album.angle);
    const z = album.radius * Math.sin(album.angle);
    const y = album.yOffset;

    // rotation around Y to make the album stand face the center
    const rotY = Math.atan2(x, z) + Math.PI;

    return {
      ...album,
      texture,
      position: [x, y, z] as [number, number, number],
      rotation: [0, rotY, 0] as [number, number, number],
    };
  });
</script>

{#each preparedAlbums as album (album.title)}
  <T.Group
    position={album.position}
    rotation={album.rotation}
    onpointerenter={() => {
      document.body.style.cursor = "pointer";
      onAlbumHover?.(album);
    }}
    onpointerleave={() => {
      document.body.style.cursor = "auto";
      onAlbumHover?.(null);
    }}
    onclick={(e: any) => {
      e.stopPropagation();
      onAlbumClick?.(album);
    }}
  >
    <!-- ══════════════════════════════════════════ -->
    <!-- MUSEUM PEDESTAL (Base stand column)        -->
    <!-- ══════════════════════════════════════════ -->
    <T.Mesh position={[0, 0.4, 0]} castShadow receiveShadow>
      <T.BoxGeometry args={[0.56, 0.8, 0.56]} />
      <T.MeshStandardMaterial color={0x151413} roughness={0.7} />
    </T.Mesh>

    <!-- Brass Trim / Case Floor -->
    <T.Mesh position={[0, 0.805, 0]} castShadow>
      <T.BoxGeometry args={[0.58, 0.01, 0.58]} />
      <T.MeshStandardMaterial
        color={0xc5a059}
        metalness={0.8}
        roughness={0.2}
      />
    </T.Mesh>

    <!-- ══════════════════════════════════════════ -->
    <!-- GLASS CASE & FRAME STRUCTURE               -->
    <!-- ══════════════════════════════════════════ -->
    <!-- Transparent Glass Cover -->
    <T.Mesh position={[0, 1.11, 0]} receiveShadow>
      <T.BoxGeometry args={[0.57, 0.6, 0.57]} />
      <T.MeshStandardMaterial
        color={0xffffff}
        transparent
        opacity={0.16}
        roughness={0.05}
        metalness={0.9}
        side={THREE.DoubleSide}
      />
    </T.Mesh>

    <!-- Brass Corner Pillars -->
    {#each [[-0.285, -0.285], [-0.285, 0.285], [0.285, -0.285], [0.285, 0.285]] as [px, pz]}
      <T.Mesh position={[px, 1.11, pz]} castShadow>
        <T.BoxGeometry args={[0.012, 0.6, 0.012]} />
        <T.MeshStandardMaterial
          color={0xc5a059}
          metalness={0.8}
          roughness={0.2}
        />
      </T.Mesh>
    {/each}

    <!-- Brass Top Horizontal Rails -->
    <!-- Front/Back -->
    <T.Mesh position={[0, 1.41, 0.285]} castShadow>
      <T.BoxGeometry args={[0.582, 0.012, 0.012]} />
      <T.MeshStandardMaterial
        color={0xc5a059}
        metalness={0.8}
        roughness={0.2}
      />
    </T.Mesh>
    <T.Mesh position={[0, 1.41, -0.285]} castShadow>
      <T.BoxGeometry args={[0.582, 0.012, 0.012]} />
      <T.MeshStandardMaterial
        color={0xc5a059}
        metalness={0.8}
        roughness={0.2}
      />
    </T.Mesh>
    <!-- Left/Right -->
    <T.Mesh position={[-0.285, 1.41, 0]} castShadow>
      <T.BoxGeometry args={[0.012, 0.012, 0.582]} />
      <T.MeshStandardMaterial
        color={0xc5a059}
        metalness={0.8}
        roughness={0.2}
      />
    </T.Mesh>
    <T.Mesh position={[0.285, 1.41, 0]} castShadow>
      <T.BoxGeometry args={[0.012, 0.012, 0.582]} />
      <T.MeshStandardMaterial
        color={0xc5a059}
        metalness={0.8}
        roughness={0.2}
      />
    </T.Mesh>

    <!-- ══════════════════════════════════════════ -->
    <!-- SHOWCASE SPOTLIGHT & VOLUMETRIC CONE       -->
    <!-- ══════════════════════════════════════════ -->
    {#if albumLights}
      <!-- Showcase SpotLight from high above -->
      <T.SpotLight
        position={[0, 4.5, 0]}
        target.position={[0, 0.98, 0]}
        intensity={4.5}
        angle={Math.PI / 10}
        penumbra={0.6}
        distance={6}
        decay={1.2}
        color={0xfffae2}
      />

      <!-- Volumetric Spotlight Cone -->
      <T.Mesh position={[0, 2.65, 0]}>
        <T.CylinderGeometry args={[0.02, 0.45, 3.7, 32, 1, true]} />
        <T.MeshBasicMaterial
          color={0xfffad0}
          transparent
          opacity={0.06}
          depthWrite={false}
          side={THREE.DoubleSide}
        />
      </T.Mesh>
    {/if}

    <!-- ══════════════════════════════════════════ -->
    <!-- ALBUM ARTWORK DISPLAY (Inside Case)        -->
    <!-- ══════════════════════════════════════════ -->
    <!-- Easel Wooden Base (rests on pedestal at y=0.8) -->
    <T.Mesh position={[0, 0.815, 0]} castShadow receiveShadow>
      <T.BoxGeometry args={[0.38, 0.03, 0.22]} />
      <T.MeshStandardMaterial color={0x2b1d12} roughness={0.75} />
    </T.Mesh>

    <!-- Easel Backrest Support (placed behind the album cover, tilted back) -->
    <T.Mesh
      position={[0, 0.95, -0.04]}
      rotation={[album.tiltAngle, 0, 0]}
      castShadow
    >
      <T.BoxGeometry args={[0.05, 0.26, 0.03]} />
      <T.MeshStandardMaterial color={0x2b1d12} roughness={0.8} />
    </T.Mesh>

    <!-- Easel Bottom Lip (placed in front of the bottom edge to hold it) -->
    <T.Mesh position={[0, 0.838, 0.06]} castShadow>
      <T.BoxGeometry args={[0.34, 0.015, 0.015]} />
      <T.MeshStandardMaterial color={0x2b1d12} roughness={0.7} />
    </T.Mesh>

    <!-- Cardboard Sleeve Backing -->
    <T.Mesh
      position={[0, 0.98, 0.0]}
      rotation={[album.tiltAngle, 0, 0]}
      castShadow
      receiveShadow
    >
      <T.BoxGeometry args={[0.304, 0.304, 0.008]} />
      <T.MeshStandardMaterial color={0x181818} roughness={0.85} />
    </T.Mesh>

    <!-- Front Cover image plane (offset by 0.006 to avoid z-fighting / black lines) -->
    <T.Mesh
      position={[0, 0.98, 0.006]}
      rotation={[album.tiltAngle, 0, 0]}
      castShadow
    >
      <T.PlaneGeometry args={[0.3, 0.3]} />
      <T.MeshStandardMaterial map={album.texture} roughness={0.65} />
    </T.Mesh>

    <!-- ══════════════════════════════════════════ -->
    <!-- MUSEUM PLAQUE (On Pedestal Front)          -->
    <!-- ══════════════════════════════════════════ -->
    <!-- Brass plaque backing plate -->
    <T.Mesh position={[0, 0.45, 0.281]} castShadow>
      <T.BoxGeometry args={[0.38, 0.26, 0.008]} />
      <T.MeshStandardMaterial
        color={0xc5a059}
        metalness={0.75}
        roughness={0.25}
      />
    </T.Mesh>

    <!-- Brass plaque text letters -->
    <HTML position={[0, 0.45, 0.286]} transform occlude pointerEvents="none" scale={0.1}>
      <div class="museum-plaque-text">
        <div class="plaque-title-text">{album.title}</div>
        <div class="plaque-year-text">{album.year}</div>
      </div>
    </HTML>
  </T.Group>
{/each}

<style>
  .museum-plaque-text {
    font-family: 'Space Grotesk', sans-serif;
    color: #ffffff;
    text-align: center;
    width: 170px;
    padding: 6px;
    box-sizing: border-box;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    user-select: none;
    line-height: 1.2;
    text-shadow: 0 0 3px #ffd700, 0 0 8px rgba(255, 215, 0, 0.8), 0 0 15px rgba(255, 215, 0, 0.4);
  }
  .plaque-title-text {
    font-size: 16px;
    font-weight: 700;
    text-transform: uppercase;
    letter-spacing: 0.5px;
    margin-bottom: 2px;
    word-wrap: break-word;
    width: 100%;
  }
  .plaque-year-text {
    font-size: 12px;
    font-weight: 500;
    opacity: 0.9;
  }
</style>
