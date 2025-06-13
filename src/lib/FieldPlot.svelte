<script lang="ts">
	import { onMount } from 'svelte';
	import { browser } from '$app/environment';
	import * as THREE from 'three';
	import fieldTexture from '$lib/assets/field.png';
	import Slider from '$lib/Slider.svelte';

	interface Play {
		Index: number;
		Start: number;
		End: number;
		Mesh: THREE.Mesh<THREE.BoxGeometry, THREE.MeshBasicMaterial, THREE.Object3DEventMap>;
	}

	interface Field {
		HomeTeam: string;
		AwayTeam: string;
		Plays: Array<Play>;
		PlayIndex: number;
		Mesh: THREE.Mesh<THREE.PlaneGeometry, THREE.MeshBasicMaterial, THREE.Object3DEventMap>;
	}

	const ENDZONE_PIXELS = 121;
	const ALL_YARDS_PIXELS = 884;
	const MARGIN_PIXELS = 32;
	const PIXELS_PER_YARD = ALL_YARDS_PIXELS / 100;
	const FIELD_MESH_UNIT_LENGTH = 100 + (ENDZONE_PIXELS / PIXELS_PER_YARD) * 2; 
	const FIELD_MESH_UNIT_WIDTH = FIELD_MESH_UNIT_LENGTH * (1125/543);

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

	let cameraPosX = $state(125);
	let cameraPosY = $state(261);
	let cameraPosZ = $state(-62);

	let cameraRotX = $state(0);
	let cameraRotY = $state(0);
	let cameraRotZ = $state(0);

	onMount(() => {
		if (browser && !initialized) {
			$inspect(fields);
			initScene(renderWidth, renderHeight);
			initModels();
		}
	});

	function initScene(width: number, height: number) {
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
	}

	function animate() {
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
	}

	function render() {
		renderer.clear();
		renderer.render(scene, camera!);
	}

	function handleResize() {
		camera!.aspect = renderWidth / renderHeight;
		camera!.updateProjectionMatrix();
		renderer.setSize(renderWidth, renderHeight);
	}

	function initModels() {
		fields = [];
		const testField = $state(createField('home', 'away'));
		
		const sphere = new THREE.SphereGeometry(1);
		const origin_material = new THREE.MeshBasicMaterial({
			color: 'white'
		});
		const origin = new THREE.Mesh(sphere, origin_material);
		scene.add(origin);
		$inspect(testField.Plays);
	}

	function createField(home: string, away: string) {
		const geometery = new THREE.PlaneGeometry(FIELD_MESH_UNIT_WIDTH, FIELD_MESH_UNIT_LENGTH);
		const loader = new THREE.TextureLoader();

		const texture = loader.load(fieldTexture);
		texture.colorSpace = THREE.SRGBColorSpace;
		texture.magFilter = THREE.NearestFilter;
		texture.minFilter = THREE.NearestFilter;

		const material = new THREE.MeshBasicMaterial({
			map: texture
		});

		const plane = new THREE.Mesh(geometery, material);
		plane.position.x += (FIELD_MESH_UNIT_WIDTH/2) - ENDZONE_PIXELS / PIXELS_PER_YARD * 2;
		plane.position.y -= (FIELD_MESH_UNIT_LENGTH/2) - MARGIN_PIXELS / PIXELS_PER_YARD * 2;
		const newField: Field = {
			HomeTeam: home,
			AwayTeam: away,
			Mesh: plane,
			Plays: [],
			PlayIndex: 0
		};

		fields.push(newField);
		scene.add(newField.Mesh);

		return newField;
	}

	function createPlay(yards: number) {
		const geometry = new THREE.BoxGeometry(10, yards, 10);
		const material = new THREE.MeshBasicMaterial({
			color: 'red'
		});
		const mesh = new THREE.Mesh(geometry, material);
		const play: Play = {
			Index: 0,
			Start: 0,
			End: yards,
			Mesh: mesh,
		};
		return play;
	}

	function addPlayToField(play: Play, field: Field) {
		field.Plays.push(play);
		scene.add(play.Mesh);
	}

	function handleKeyDown(event: KeyboardEvent) {
		if (event.key === 'Enter') {
			const testPlay: Play = createPlay(7);
			addPlayToField(testPlay, fields[0]);
		}
	}
</script>

<svelte:head>
	<title>SvelteKit + ThreeJS</title>
</svelte:head>

<!-- <button on:click={addPlay}></button> -->
<div class="flex-container">
	<!-- {#if debug} -->
	<!-- <div class="debugTools">
		<h1>Position:</h1>
		<Slider name={'x position'} min={-500} max={500} bind:value={cameraPosX} />
		<Slider name={'y position'} min={-500} max={500} bind:value={cameraPosY} />
		<Slider name={'z position'} min={-500} max={500} bind:value={cameraPosZ} />
		<h1>Rotation:</h1>
		<Slider name={'x rotation'} min={-4} max={4} bind:value={cameraRotX} />
		<Slider name={'y rotation'} min={-4} max={4} bind:value={cameraRotY} />
		<Slider name={'z rotation'} min={-4} max={4} bind:value={cameraRotZ} />
	</div> -->
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
