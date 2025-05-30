<script lang="ts">
	import { onMount } from 'svelte';
	import { browser } from '$app/environment';
	import * as THREE from 'three';
	import fieldTexture from '$lib/assets/field.png';

	type Field = {
		HomeTeam: string;
		AwayTeam: string;
		Mesh: THREE.Mesh<THREE.PlaneGeometry, THREE.MeshBasicMaterial, THREE.Object3DEventMap>;
	};

	let threeJSContainer: HTMLElement;
	let renderer: THREE.WebGLRenderer;
	let scene: THREE.Scene;
	let camera: THREE.PerspectiveCamera;
	let initialized: boolean = false;
	let meshes: THREE.Mesh<THREE.PlaneGeometry, THREE.MeshBasicMaterial>[] = [];

	onMount(() => {
		if (browser && !initialized) {
			initScene(window.innerWidth, window.innerHeight);
			initModels();
		}
	});

	const initScene = (width: number, height: number) => {
		// initialize renderer and append to DOM
		renderer = new THREE.WebGLRenderer({ antialias: true });
		renderer.setSize(width, height);
		threeJSContainer.appendChild(renderer.domElement);

		scene = new THREE.Scene();
		camera = new THREE.PerspectiveCamera(75, width / height, 0.1, 1000);
		camera.position.z = 5;

		renderer.render(scene, camera);
		renderer.setAnimationLoop(animate);

		window.addEventListener('resize', handleResize);
		initialized = true;
	};

	const animate = () => {
		meshes.forEach((mesh) => {
			mesh.rotation.x += 0.005;
			mesh.rotation.y += 0.005;
		});

		render();
	};

	const render = () => {
		renderer.clear();
		renderer.render(scene, camera);
	};

	const handleResize = () => {
		camera.aspect = window.innerWidth / window.innerHeight;
		camera.updateProjectionMatrix();
		renderer.setSize(window.innerWidth, window.innerHeight);
	};

	const initModels = () => {
		const testField = createField('home', 'away');
		$inspect(testField);

		scene.add(testField);
		meshes.push(testField);
	};

	const createField = (home: string, away: string) => {
		const geometery = new THREE.PlaneGeometry(1, 1);
		const loader = new THREE.TextureLoader();

		const texture = loader.load(fieldTexture);
		texture.colorSpace = THREE.SRGBColorSpace;
		texture.magFilter = THREE.NearestFilter;
		texture.minFilter = THREE.NearestFilter;

		const material = new THREE.MeshBasicMaterial({
			map: texture
		});
		const plane = new THREE.Mesh(geometery, material);
		// const newField: Field = { home, away, plane };
		return plane;
	};
</script>

<svelte:head>
	<title>SvelteKit + ThreeJS</title>
</svelte:head>

<div class="threeJSCanvas" bind:this={threeJSContainer}></div>

<style>
	div.threeJSCanvas {
		display: block;
		position: fixed;
		top: 0;
		left: 0;
		width: 100vw;
		height: 100vh;
	}
</style>
