<script lang="ts">
	import { browser } from '$app/environment';
	import * as THREE from 'three';
	import field from '$lib/assets/field.png';

	if (browser) {
		let camera: THREE.PerspectiveCamera;
		let scene: THREE.Scene;
		let renderer: THREE.WebGLRenderer;
		let plane: THREE.Mesh<THREE.PlaneGeometry, THREE.MeshBasicMaterial>;

		const init = () => {
			scene = new THREE.Scene();
			camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);

			renderer = new THREE.WebGLRenderer();
			renderer.setSize(window.innerWidth, window.innerHeight);
			document.body.appendChild(renderer.domElement);

			const loader = new THREE.TextureLoader();
			const texture = loader.load(field);
			texture.colorSpace = THREE.SRGBColorSpace;

			const geometery = new THREE.PlaneGeometry(1, 1);
			const material = new THREE.MeshBasicMaterial({ map: texture });

			plane = new THREE.Mesh(geometery, material);
			scene.add(plane);

			camera.position.z = 5;
		};

		const render = () => {
			renderer.clear();
			renderer.render(scene, camera);
		};

		const animate = () => {
			requestAnimationFrame(animate);

			plane.rotation.x += 0.005;
			plane.rotation.y += 0.005;

			render();
		};

		init();
		animate();
	}
</script>

<svelte:head>
	<title>SvelteKit + ThreeJS</title>
</svelte:head>

<section>Hello World</section>
