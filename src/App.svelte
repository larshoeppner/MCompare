<script lang="ts">
	import { onMount } from "svelte";
	import {
		Scene,
		PerspectiveCamera,
		WebGLRenderer,
		MeshBasicMaterial,
		BoxGeometry,
		Mesh,
		AmbientLight,
		Color,
		Box3,
		Vector3,
	} from "three";
	import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";
	import { DRACOLoader } from "three/examples/jsm/loaders/DRACOLoader.js";
	import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
	import { OBJLoader } from "three/examples/jsm/loaders/OBJLoader.js";

	onMount(async () => {
		createScene();
	});

	let loader: GLTFLoader;
	let scene: Scene;
	let camera: PerspectiveCamera;
	let renderer: WebGLRenderer;

	// TODO move all to ts classes
	function createScene() {
		scene = new Scene();
		scene.background = new Color(0xd3d3d3);

		camera = new PerspectiveCamera(
			75,
			window.innerWidth / window.innerHeight,
			0.1,
			1000
		);

		renderer = new WebGLRenderer();
		renderer.setSize(window.innerWidth, window.innerHeight);
		document
			.getElementById("scenecontainer")
			.appendChild(renderer.domElement);

		// const geometry = new BoxGeometry(1, 1, 1);
		// const material = new MeshBasicMaterial({ color: 0x00ff00 });
		// const cube = new Mesh(geometry, material);
		// scene.add(cube);

		camera.position.z = 5;
		// function animate() {
		// 	requestAnimationFrame(animate);

		// 	cube.rotation.x += 0.01;
		// 	cube.rotation.y += 0.01;

		// 	renderer.render(scene, camera);
		// }

		// animate();

		const directionalLight = new AmbientLight(0xffffff, 3);
		scene.add(directionalLight);

		const controls = new OrbitControls(camera, renderer.domElement);
		controls.addEventListener("change", render); // use if there is no animation loop
		// controls.minDistance = 2;
		// controls.maxDistance = 10;
		controls.target.set(0, 0, -0.2);
		controls.enableZoom = true;
		controls.update();

		loader = new GLTFLoader();

		// Optional: Provide a DRACOLoader instance to decode compressed mesh data
		const dracoLoader = new DRACOLoader();
		dracoLoader.setDecoderPath("/examples/js/libs/draco/");
		loader.setDRACOLoader(dracoLoader);
	}

	function render() {
		renderer.render(scene, camera);
	}

	function centerAlign(mroot) {
		var bbox = new Box3().setFromObject(mroot);
		var cent = bbox.getCenter(new Vector3());
		var size = bbox.getSize(new Vector3());

		// Rescale the object to normalized space
		var maxAxis = Math.max(size.x, size.y, size.z);
		mroot.scale.multiplyScalar(5.0 / maxAxis);
		bbox.setFromObject(mroot);
		bbox.getCenter(cent);
		bbox.getSize(size);
		// Reposition to 0,halfY,0
		mroot.position.copy(cent).multiplyScalar(-1);
		mroot.position.y -= size.y * 0.5;

		function animate() {
			requestAnimationFrame(animate);
			// gltf.scene.rotation.y += 0.005;
			renderer.render(scene, camera);
		}
		animate();
	}

	let currentUrl = "";
	function importGltf() {
		// get url from input

		// Load a glTF resource
		loader.load(
			// resource URL
			gltfUrl,
			// called when the resource is loaded
			function (gltf) {
				scene.add(gltf.scene);

				gltf.animations; // Array<THREE.AnimationClip>
				gltf.scene; // THREE.Group
				gltf.scenes; // Array<THREE.Group>
				gltf.cameras; // Array<THREE.Camera>
				gltf.asset; // Object

				centerAlign(gltf.scene);
			},
			// called while loading is progressing
			function (xhr) {
				console.log((xhr.loaded / xhr.total) * 100 + "% loaded");
			},
			// called when loading has errors
			function (error) {
				console.log("An error happened");
			}
		);
	}

	let gltfUrl = "http://localhost:8080/models/scene.gltf";

	let objUrl = "";

	function importObj() {
		const loader = new OBJLoader();

		// load a resource
		loader.load(
			// resource URL
			objUrl,
			// called when resource is loaded
			function (object) {
				scene.add(object);
				centerAlign(object);
			},
			// called when loading is in progresses
			function (xhr) {
				console.log((xhr.loaded / xhr.total) * 100 + "% loaded");
			},
			// called when loading has errors
			function (error) {
				console.log("An error happened");
			}
		);
	}
</script>

<main>
	<div>
		<div>
			<button class="btn btn-active" on:click={() => importGltf()}>
				Load GLTF
			</button>
			<input id="modelurl" bind:value={gltfUrl} />
		</div>
		<div>
			<button class="btn btn-active" on:click={() => importObj()}>
				Load OBJ
			</button>
			<input id="modelurl" bind:value={objUrl} />
		</div>
	</div>
	<div id="scenecontainer" />
</main>

<style>
	main {
		text-align: center;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}

	#modelurl {
		width: 600px;
	}

	#scenecontainer {
		width: calc(100% - 200px);
		height: 1200px;
	}

	h1 {
		color: #ff3e00;
		text-transform: uppercase;
		font-size: 4em;
		font-weight: 100;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>
