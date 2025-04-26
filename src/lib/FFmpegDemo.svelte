<script lang="ts">
	import { onMount } from 'svelte';
	import { FFmpeg } from '../ffmpeg';
	// @ts-ignore
	import type { LogEvent } from '@ffmpeg/ffmpeg/dist/esm/types';
	import { fetchFile, toBlobURL } from '@ffmpeg/util';

	let audioEl: HTMLAudioElement;

	const baseURL = '/core-mt/dist/esm';
	const audioURL = '/audio.tak';

	let message = 'Click Start to Transcode';

	async function transcode() {
		const ffmpeg = new FFmpeg();
		message = 'Loading ffmpeg-core.js';
		ffmpeg.on('log', ({ message: msg }: LogEvent) => {
			message = msg;
			console.log(message);
		});
		await ffmpeg.load({
			coreURL: await toBlobURL(`${baseURL}/ffmpeg-core.js`, 'application/javascript'),
			wasmURL: await toBlobURL(`${baseURL}/ffmpeg-core.wasm`, 'application/wasm'),
			workerURL: await toBlobURL(`${baseURL}/ffmpeg-core.worker.js`, 'application/javascript')
		});
		message = 'Start transcoding';
		await ffmpeg.writeFile('input.tak', await fetchFile(audioURL));
		await ffmpeg.exec(['-i', 'input.tak', 'output.wav']);
		message = 'Complete transcoding';
		const data = await ffmpeg.readFile('output.wav');
		console.log('done');
		audioEl.src = URL.createObjectURL(
			new Blob([(data as Uint8Array).buffer], { type: 'audio/wave' })
		);
		await audioEl.play();
	}

	onMount(() => {
		transcode();
	});
</script>

<div>
	<!-- svelte-ignore a11y-media-has-caption -->
	<audio controls loop bind:this={audioEl}></audio>
	<br />
	<button on:click={transcode}>Transcode</button>
	<p>{message}</p>
</div>
