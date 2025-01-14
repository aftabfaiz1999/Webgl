<script>
  import { onMount } from "svelte";
  import * as THREE from "three";
  import gsap from "gsap";

  let canvas;
  let collectionPercentage = 0;

  onMount(() => {
    const scene = new THREE.Scene();
    const camera = new THREE.PerspectiveCamera(
      75,
      canvas.clientWidth / canvas.clientHeight,
      0.1,
      1000
    );
    camera.position.set(0, 0, 10);

    const renderer = new THREE.WebGLRenderer({ 
      canvas, 
      antialias: true,
      alpha: true 
    });
    renderer.setSize(canvas.clientWidth, canvas.clientHeight);
    renderer.setClearColor(0x1a1a1a);

    // Create card texture function with proper design
    function createCardTexture(color, darkColor) {
      const canvas = document.createElement('canvas');
      const ctx = canvas.getContext('2d');
      canvas.width = 512;
      canvas.height = 712;

      // White border
      ctx.fillStyle = 'white';
      ctx.fillRect(0, 0, canvas.width, canvas.height);
      
      // Card inner area
      ctx.fillStyle = color;
      ctx.fillRect(8, 8, canvas.width - 16, canvas.height - 16);
      
      // Dark header area
      ctx.fillStyle = darkColor;
      ctx.fillRect(8, 60, canvas.width - 16, canvas.height * 0.3);
      
      // Name text
      ctx.fillStyle = 'white';
      ctx.font = 'bold 36px Arial';
      ctx.fillText('Name', 20, 40);
      
      // New tag
      ctx.fillStyle = '#ff4757';
      ctx.beginPath();
      ctx.roundRect(canvas.width - 100, 10, 80, 40, 20);
      ctx.fill();
      
      ctx.fillStyle = 'white';
      ctx.font = 'bold 24px Arial';
      ctx.fillText('New', canvas.width - 80, 37);

      return new THREE.CanvasTexture(canvas);
    }

    // Create booster pack texture with ridges
    function createBoosterTexture() {
      const canvas = document.createElement('canvas');
      const ctx = canvas.getContext('2d');
      canvas.width = 512;
      canvas.height = 768;

      // Gradient background
      const gradient = ctx.createLinearGradient(0, 0, 0, canvas.height);
      gradient.addColorStop(0, '#4ade80');
      gradient.addColorStop(1, '#22c55e');
      ctx.fillStyle = gradient;
      ctx.fillRect(0, 0, canvas.width, canvas.height);

      // Ridged top
      for (let i = 0; i < 20; i++) {
        ctx.fillStyle = i % 2 === 0 ? '#2f2f2f' : '#3f3f3f';
        ctx.fillRect(0, i * 4, canvas.width, 4);
      }

      return new THREE.CanvasTexture(canvas);
    }

    // Card settings
    const cardData = [
      { color: '#ff69b4', darkColor: '#c4527a', position: -4 }, // Pink
      { color: '#ff8c00', darkColor: '#cc7000', position: 0 },  // Orange
      { color: '#ffd700', darkColor: '#ccac00', position: 4 }   // Yellow
    ];

    // Create cards
    const cards = cardData.map(({ color, darkColor, position }, index) => {
      const geometry = new THREE.PlaneGeometry(3, 4.2);
      const texture = createCardTexture(color, darkColor);
      const material = new THREE.MeshBasicMaterial({ 
        map: texture,
        transparent: true 
      });
      const card = new THREE.Mesh(geometry, material);
      card.position.set(0, -10, 0.1 * index);
      scene.add(card);
      return { mesh: card, targetX: position };
    });

    // Create booster pack
    const boosterGeometry = new THREE.PlaneGeometry(4, 6);
    const boosterTexture = createBoosterTexture();
    const boosterMaterial = new THREE.MeshBasicMaterial({ 
      map: boosterTexture,
      transparent: true 
    });
    const boosterPack = new THREE.Mesh(boosterGeometry, boosterMaterial);
    boosterPack.position.y = -8;
    scene.add(boosterPack);

    function startAnimation() {
      // Reset
      boosterPack.position.y = -8;
      cards.forEach(card => {
        card.mesh.position.set(0, -10, 0);
        card.mesh.scale.set(1, 1, 1);
      });
      collectionPercentage = 0;

      const timeline = gsap.timeline({ 
        onComplete: startAnimation,
        delay: 1 
      });

      // Booster animation
      timeline.to(boosterPack.position, {
        y: 0,
        duration: 1,
        ease: "power2.out"
      })
      .to(boosterPack.position, {
        y: -8,
        duration: 1,
        ease: "power2.in"
      });

      // Cards appear and spread
      cards.forEach((card, index) => {
        timeline.to(card.mesh.position, {
          y: 2,
          duration: 0.8,
          ease: "back.out(1.7)",
          onStart: () => {
            collectionPercentage = ((index + 1) / cards.length) * 33.3;
          }
        })
        .to(card.mesh.position, {
          x: card.targetX,
          duration: 0.5,
          ease: "power2.inOut"
        });
      });

      // Cards move to corner and vanish one by one
      cards.forEach((card, index) => {
        timeline.to(card.mesh.position, {
          x: 8,
          y: 4,
          duration: 0.8,
          delay: index * 0.2,
          ease: "power2.inOut"
        })
        .to(card.mesh.scale, {
          x: 0.1,
          y: 0.1,
          duration: 0.8,
          ease: "power2.in",
          onComplete: () => {
            if (index === cards.length - 1) {
              collectionPercentage = 100;
            }
          }
        }, "-=0.8");
      });
    }

    function animate() {
      requestAnimationFrame(animate);
      renderer.render(scene, camera);
    }

    animate();
    startAnimation();

    window.addEventListener('resize', () => {
      camera.aspect = canvas.clientWidth / canvas.clientHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(canvas.clientWidth, canvas.clientHeight);
    });
  });
</script>

<div class="container">
  <canvas bind:this={canvas}></canvas>
  <div class="collection">
    Collection: {collectionPercentage.toFixed(1)}%
  </div>
</div>

<style>
  .container {
    position: relative;
    width: 100%;
    height: 100vh;
    background: #1a1a1a;
  }

  canvas {
    display: block;
    width: 100%;
    height: 100%;
  }

  .collection {
    position: absolute;
    top: 20px;
    right: 20px;
    background: rgba(0, 0, 0, 0.5);
    color: white;
    padding: 8px 16px;
    border-radius: 20px;
    font-family: Arial, sans-serif;
    backdrop-filter: blur(4px);
  }
</style>