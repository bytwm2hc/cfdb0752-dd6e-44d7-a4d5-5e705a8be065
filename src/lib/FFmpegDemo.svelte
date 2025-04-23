<script lang="ts">
	import { FFmpeg } from '@ffmpeg/ffmpeg';
	// @ts-ignore
	import type { LogEvent } from '@ffmpeg/ffmpeg/dist/esm/types';
	import { fetchFile, toBlobURL } from '@ffmpeg/util';

	let videoEl: HTMLVideoElement;

	const baseURL = 'https://unpkg.com/@ffmpeg/core@0.12.9/dist/esm';
	const videoURL = '//cdn.mecx.dev/input.tak';

	let message = 'Click Start to Transcode';

	async function transcode() {
		const ffmpeg = new FFmpeg();
		message = 'Loading ffmpeg-core.js';
		ffmpeg.on('log', ({ message: msg }: LogEvent) => {
			message = msg;
			console.log(message);
		});
		await ffmpeg.load({
			coreURL: await toBlobURL(`${baseURL}/ffmpeg-core.js`, 'text/javascript'),
			wasmURL: await toBlobURL(`${baseURL}/ffmpeg-core.wasm`, 'application/wasm'),
			//workerURL: await toBlobURL(`${baseURL}/ffmpeg-core.worker.js`, 'text/javascript')
		});
		message = 'Start transcoding';
		await ffmpeg.writeFile('input.tak', await fetchFile(videoURL));
		await ffmpeg.exec(['-i', 'input.tak', 'output.wav']);
		message = 'Complete transcoding';
		const data = await ffmpeg.readFile('output.wav');
		console.log('done');
		videoEl.src = URL.createObjectURL(
			new Blob([(data as Uint8Array).buffer], { type: 'audio/wave' })
		);
	}
</script>

<div>
	<!-- svelte-ignore a11y-media-has-caption -->
	<audio bind:this={videoEl} controls />
	<br />
	<button on:click={transcode}>Start</button>
	<p>{message}</p>
</div>
