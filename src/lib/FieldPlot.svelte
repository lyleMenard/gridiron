<script lang="ts">
	import { onMount } from 'svelte';
	import { browser } from '$app/environment';
	import * as THREE from 'three';
	import fieldTexture from '$lib/assets/field.png';
	import Slider from '$lib/Slider.svelte';

	interface Play {
		Quarter: number;
		Minute: number;
		Second: number;
		ToGo: number;
		Yards: number;
		Mesh: THREE.Mesh<THREE.BoxGeometry, THREE.MeshBasicMaterial, THREE.Object3DEventMap>;
	}

	interface Field {
		HomeTeam: string;
		AwayTeam: string;
		Plays: Array<Play>;
		PlayIndex: number;
		Mesh: THREE.Mesh<THREE.PlaneGeometry, THREE.MeshBasicMaterial, THREE.Object3DEventMap>;
	}

	let threeJSContainer: HTMLElement;

	let renderer: THREE.WebGLRenderer;
	let scene: THREE.Scene;
	let camera: THREE.PerspectiveCamera | undefined = $state();
	let meshes: THREE.Mesh<THREE.PlaneGeometry, THREE.MeshBasicMaterial>[] = [];

	let fields: Array<Field> = $state([]);

	let initialized: boolean = false;
	let debug: boolean = $state(false);

	let renderWidth = $state(0);
	let renderHeight = $state(0);

	let cameraPosX = $state(-1.6);
	let cameraPosY = $state(1.4);
	let cameraPosZ = $state(-1.4);

	let cameraRotX = $state(0.8);
	let cameraRotY = $state(-0.6);
	let cameraRotZ = $state(-0.5);

	onMount(() => {
		if (browser && !initialized) {
			$inspect(fields);
			initScene(renderWidth, renderHeight);
			initModels();
		}
	});

	const initScene = (width: number, height: number) => {
		// initialize renderer and append to DOM
		renderer = new THREE.WebGLRenderer({ antialias: true });
		renderer.setSize(width, height);
		renderer.setPixelRatio(window.devicePixelRatio);
		threeJSContainer.appendChild(renderer.domElement);

		scene = new THREE.Scene();
		camera = new THREE.PerspectiveCamera(60, width / height, 0.1, 1000);

		camera.position.x = cameraPosX;
		camera.position.z = cameraPosY;
		camera.position.y = cameraPosZ;

		camera.rotation.x = cameraRotX;
		camera.rotation.y = cameraRotY;
		camera.rotation.z = cameraRotZ;

		renderer.render(scene, camera);
		renderer.setAnimationLoop(animate);

		window.addEventListener('resize', handleResize);
		initialized = true;
	};

	const animate = () => {
		fields.forEach((field) => {
			// field.Mesh.rotation.x -= 0.005;
			// mesh.rotation.z += 0.005;
		});

		camera!.position.x = cameraPosX;
		camera!.position.z = cameraPosY;
		camera!.position.y = cameraPosZ;

		camera!.rotation.x = cameraRotX;
		camera!.rotation.y = cameraRotY;
		camera!.rotation.z = cameraRotZ;

		render();
	};

	const render = () => {
		renderer.clear();
		renderer.render(scene, camera!);
	};

	const handleResize = () => {
		camera!.aspect = renderWidth / renderHeight;
		camera!.updateProjectionMatrix();
		renderer.setSize(renderWidth, renderHeight);
	};

	const initModels = () => {
		fields = [];
		const testField = $state(createField('home', 'away'));
		$inspect(testField.Plays);

		fields.push(testField);
		scene.add(testField.Mesh);
	};

	const createField = (home: string, away: string) => {
		const planeDimensions = [1200, 579];
		const geometery = new THREE.PlaneGeometry(1200 / 579, 1);
		const loader = new THREE.TextureLoader();

		const texture = loader.load(fieldTexture);
		texture.colorSpace = THREE.SRGBColorSpace;
		texture.magFilter = THREE.NearestFilter;
		texture.minFilter = THREE.NearestFilter;

		const material = new THREE.MeshBasicMaterial({
			map: texture
		});
		const plane = new THREE.Mesh(geometery, material);
		const newField: Field = {
			HomeTeam: home,
			AwayTeam: away,
			Mesh: plane,
			Plays: [],
			PlayIndex: 0
		};
		return newField;
	};

	const createPlay = (
		quarter: number,
		minute: number,
		second: number,
		togo: number,
		yards: number
	) => {
		const geometry = new THREE.BoxGeometry(1, 5, yards);

		const material = new THREE.MeshBasicMaterial({
			color: 'red'
		});
		const playMesh = new THREE.Mesh(geometry, material);

		const play: Play = {
			Quarter: quarter,
			Minute: minute,
			Second: second,
			ToGo: togo,
			Yards: yards,
			Mesh: playMesh
		};
		return play;
	};

	const addPlayToField = (play: Play) => {
		fields.forEach((field) => {
			field.Plays.push(play);
		});
	};

	const handleKeyDown = (event: KeyboardEvent) => {
		if (event.key === 'Enter') {
			debug = true;
			const testPlay: Play = createPlay(1, 14, 59, 10, 10);
			addPlayToField(testPlay);
		}
	};
</script>

<svelte:head>
	<title>SvelteKit + ThreeJS</title>
</svelte:head>

<!-- <button on:click={addPlay}></button> -->
<div class="flex-container">
	<!-- {#if debug} -->
	<div class="debugTools">
		<h1>Position:</h1>
		<Slider name={'x position'} min={-5} max={5} bind:value={cameraPosX} />
		<Slider name={'y position'} min={-5} max={5} bind:value={cameraPosY} />
		<Slider name={'z position'} min={-5} max={5} bind:value={cameraPosZ} />
		<h1>Rotation:</h1>
		<Slider name={'x rotation'} min={-5} max={5} bind:value={cameraRotX} />
		<Slider name={'y rotation'} min={-5} max={5} bind:value={cameraRotY} />
		<Slider name={'z rotation'} min={-5} max={5} bind:value={cameraRotZ} />
	</div>
	<!-- {/if} -->
	<div
		class="threeJSCanvas"
		bind:this={threeJSContainer}
		bind:clientWidth={renderWidth}
		bind:clientHeight={renderHeight}
	></div>
</div>
<svelte:window onkeydown={handleKeyDown} />

<style>
	h1 {
		font-size: 14pt;
	}

	div.flex-container {
		display: flex;
	}

	div.debugTools {
		flex: 2;
		background-color: white;
	}

	div.threeJSCanvas {
		flex: 5;
		background-color: black;
		/* display: block; */
		/* position: fixed; */
		top: 0;
		left: 0;
		/* width: 100vw; */
		height: 100vh;
	}
</style>
