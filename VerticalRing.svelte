<script>
  import { onMount } from 'svelte';
  import * as THREE from 'three';

  let container;
  let currentIndex = 0;
  let icons = [];
  const total = 12;

  function rotateRing(direction = 1) {
    currentIndex = (currentIndex + direction + total) % total;
    updateIconTransforms();
  }

  function updateIconTransforms() {
    for (let i = 0; i < total; i++) {
      const angle = ((i - currentIndex) * (Math.PI / total));
      const icon = icons[i];
      icon.position.y = Math.sin(angle) * 4;
      icon.position.z = Math.cos(angle) * 4 - 5;
      icon.scale.setScalar(i === currentIndex ? 1.2 : 0.7);
      icon.material.opacity = i === currentIndex ? 1 : 0.4;
    }
  }

  onMount(() => {
    const scene = new THREE.Scene();
    const camera = new THREE.PerspectiveCamera(75, container.clientWidth / container.clientHeight, 0.1, 100);
    const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
    renderer.setSize(container.clientWidth, container.clientHeight);
    container.appendChild(renderer.domElement);

    const texture = new THREE.TextureLoader().load('/icon.png');

    for (let i = 0; i < total; i++) {
      const material = new THREE.SpriteMaterial({ map: texture, transparent: true, opacity: 0.5 });
      const sprite = new THREE.Sprite(material);
      scene.add(sprite);
      icons.push(sprite);
    }

    updateIconTransforms();

    camera.position.z = 5;

    function animate() {
      requestAnimationFrame(animate);
      renderer.render(scene, camera);
    }
    animate();

    window.addEventListener('wheel', e => rotateRing(Math.sign(e.deltaY)));
    window.addEventListener('keydown', e => {
      if (e.key === 'ArrowUp') rotateRing(-1);
      if (e.key === 'ArrowDown') rotateRing(1);
    });

    // Swipe gestures for mobile
    let startY = null;
    container.addEventListener('touchstart', e => {
      startY = e.touches[0].clientY;
    });
    container.addEventListener('touchend', e => {
      const endY = e.changedTouches[0].clientY;
      if (startY !== null) {
        const delta = startY - endY;
        if (Math.abs(delta) > 30) rotateRing(Math.sign(delta));
      }
    });

    window.addEventListener('resize', () => {
      camera.aspect = container.clientWidth / container.clientHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(container.clientWidth, container.clientHeight);
    });
  });
</script>

<div bind:this={container} style="width:100vw; height:100vh;" />
