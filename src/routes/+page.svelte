<script lang="ts">
	import { onMount } from 'svelte';
	import * as cocoSsd from '@tensorflow-models/coco-ssd';
	import * as tf from '@tensorflow/tfjs';

	let videoElement:any;
	let predictions: any = [];
	let isModelLoaded = false;
	let isVideoReady = false;
	let model;

	onMount(async () => {
		await loadModel();
		startWebcam();
	});

	// Load COCO-SSD pre-trained model
	const loadModel = async () => {
		await tf.ready();
		const model = await cocoSsd.load();
		isModelLoaded = true;
		console.log('Model loaded successfully');
		tryStartDetection();
		startDetection();
	};

	const tryStartDetection = () => {
		if (isModelLoaded && isVideoReady) {
			console.log('🚀 Starting detection...');
			startDetection();
		}
	};

	// Start webcam feed
	const startWebcam = () => {
		console.log('Video dimensions:', videoElement.videoWidth, videoElement.videoHeight);

		navigator.mediaDevices
			.getUserMedia({ video: true })
			.then((stream) => {
				if (!stream) return;
				console.log('Tried to start Stream');

				videoElement.srcObject = stream;
			})
			.catch((err) => {
				console.error('Error accessing webcam: ', err);
			});
	};

	// Detect objects in the webcam feed
	const startDetection = async () => {
		// Only proceed if dimensions are valid
		if (videoElement.videoWidth === 0 || videoElement.videoHeight === 0) {
			console.warn('Video dimensions not ready yet');
			return;
		}

		const predictions = await model.detect(videoElement);
		console.log(predictions);

		setTimeout(() => {
			requestAnimationFrame(startDetection);
		}, 5000);
	};
</script>

<div class="flex flex-col items-center justify-center">
	<h1>Sky Object Detection with TensorFlow.js</h1>

	<!-- Webcam feed -->

	<video
		class="h-auto w-[100%] border-gray-100"
		id="webcam"
		bind:this={videoElement}
		autoplay
		playsinline
	>
		<track kind="captions" srclang="en" label="English captions" default />
	</video>

	<!-- Display detected objects -->
	{#if predictions.length > 0}
		<div>
			{#each predictions as prediction}
				<div
					class="prediction"
					style="top: {prediction.bbox[1]}px; left: {prediction.bbox[0]}px; width: {prediction
						.bbox[2]}px; height: {prediction.bbox[3]}px;"
				>
					{prediction.class} - {Math.round(prediction.score * 100)}%
				</div>
			{/each}
		</div>
	{/if}
</div>

<style>
	.prediction {
		position: absolute;
		background-color: rgba(255, 0, 0, 0.5);
		color: white;
		padding: 5px;
		font-size: 16px;
	}
</style>
