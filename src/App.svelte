<script>
	import {isSafari,mousePos, loaded, toneLoaded,darkMode, pwa, update, version,camera,posenetLoaded,mediapipeHandsLoaded,hasHover } from './stores.js';
	import Canvas from './components/Canvas.svelte'
	import Nav from './components/UI/UI.nav.svelte'
  import Notif from './components/UI/UI.notif.svelte'
	import Shortcuts from './components/UI/UI.shortcuts.svelte'
	import Tone from './components/Tone.svelte'
  import DropZone from './components/UI/UI.dropzone.svelte'
	import Webcam from './components/Video.webcam.svelte'
	import PoseNet from './components/Video.posenet.svelte'
  import Hands from './components/Video.hands.svelte'

	if(typeof dataLayer === 'undefined'){
		window.dataLayer = []
    console.log('dataLayer not available')
	}

	const initSW = () => {
		if ('serviceWorker' in navigator) {
			//Version Control Prompt User Refresh
			navigator.serviceWorker.addEventListener('controllerchange', () => {
				update.set(true)
			});
			//Received Messges from SW
			navigator.serviceWorker.addEventListener('message', event => {
				version.set(event.data)
				console.log('%c%s',
				'color: rgb(229,70,70); background: rgb(25,25,25);padding:4px 8px 4px 8px;border-radius:4px',
				'THEREMIX ~~~ '+event.data)
			});
			// Register service worker
			navigator.serviceWorker.register('/sw.js')
				.then((reg) => {
				}).catch((e) => {
					console.log(e);
			});
			// Get version from service worker
			navigator.serviceWorker.ready.then( registration => {
				registration.active.postMessage({ action: 'version' });
			});
		}
	}

	const getDeviceInfo = () => {
		if (window.matchMedia('(display-mode: fullscreen)').matches) {  
			pwa.set(true);
			dataLayer.push({'event':'standalone'});
		}
		if (window.matchMedia('(prefers-color-scheme: dark)').matches){
			dataLayer.push({
			'event':'theme',
			'eventAction' : 'dark'
			})
		}else{
			dataLayer.push({
			'event':'theme',
			'eventAction' : 'light'
			})
		}
		hasHover.set(window.matchMedia( "(hover: hover)" ).matches)
	}

	initSW()
	getDeviceInfo()

	let tfLoaded = false,
	toneJSLoaded = false,
	teoriaLoaded = false,
	toneMIDILoaded = false
	let cameraTriggered = false
	$:{
		if($camera){
			cameraTriggered = true
		}
	}

</script>

<svelte:head>
  {#if !window.MediaRecorder}
  <script defer src="/libraries/polyfill.js"></script>
  {/if}

	{#if $loaded}
  	<script defer src="https://cdn.jsdelivr.net/npm/tone@14.8.17/build/Tone.min.js" on:load={()=>{toneJSLoaded=true}}></script>
	{/if}

	{#if toneJSLoaded}
	<script defer src="/libraries/teoria.min.js" on:load={()=>{teoriaLoaded=true}}></script>
	{/if}

	{#if teoriaLoaded}
  <script defer src="https://cdn.jsdelivr.net/npm/@tonejs/midi@2.0.27/build/Midi.min.js" on:load={()=>{toneMIDILoaded=true}}></script>
  
	{/if}

    {#if (!isSafari)}
      <script src="https://cdn.jsdelivr.net/npm/@mediapipe/holistic/holistic.js" crossorigin="anonymous" on:load={()=>{mediapipeHandsLoaded.set(true)}}></script>
    {/if}
  
  <script defer src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@2.0.1/dist/tf.min.js" on:load={()=>{tfLoaded=true}}></script>
  
  {#if tfLoaded}
    <script defer src="https://cdn.jsdelivr.net/npm/@tensorflow-models/posenet@2.2.1/dist/posenet.min.js" on:load={()=>{posenetLoaded.set(true)}}></script>
  {/if}
</svelte:head>

<main>
  <Webcam/>
	<Canvas/>
	<Shortcuts/>
	<Nav/>
  <Notif/>
  <DropZone/>
	
	{#if toneLoaded && toneMIDILoaded}
		<Tone/>
	{/if}
	{#if $posenetLoaded}
		<PoseNet/>
	{/if}
  {#if $mediapipeHandsLoaded}
    <Hands/>
  {/if}
</main>

<style>
  @font-face {
    font-family: "Whirly Birdie";
    src: url("https://theremin.app/assets/fonts/WhirlyBirdieGX.woff2")
      format("woff2");
	font-display: swap;
  }

  @font-face {
      font-family: "Whirly Birdie Regular";
      src: url("https://theremin.app/assets/fonts/WhirlyBirdie-Regular.woff")
          format("woff");
		font-display: swap;
  }

	@font-face {
	font-family: "Nicholson Beta";
	src: url("https://theremin.app/assets/fonts/nicholsonbeta-webfont.woff2")
		format("woff2");
		font-display: swap;
	}

  :root {

    --sun: 252, 216, 54;
    --blue:62, 139, 202;
    --navy:29, 55, 103;

    --offwhite: 250, 236, 197;
    --darkoffwhite: 230 216,177;

    --offwhitetint:255, 221, 176;

    --offwhitegradient: linear-gradient(0deg, rgb(var(--offwhitetint)) 0%, rgba(var(--offwhitetint),0.920045518207283) 70%, rgba(var(--offwhitetint),0) 100%);
    --offwhitegradient2: linear-gradient(0deg, rgb(var(--offwhitetint)) 0%, rgba(var(--offwhitetint),0.920045518207283) 70%, rgba(var(--offwhitetint),0) 100%);

    --crimson:229,70,70;
    --deepCrimson: 139,39,39;

    --charcoal:60,60,60;
    --lightcharcoal:85,85,85;
    --darkcharcoal:35,35,35;

    --wave:url(https://theremin.app/assets/global/h.svg);
    --waveRed:url(https://theremin.app/assets/global/hred.svg);

  }


  :global(*) {  
    -webkit-backface-visibility:  hidden;
    -webkit-tap-highlight-color:  transparent;
   }
		
	main {
		position: fixed;
		/* display:grid;
		grid-template-columns: repeat(12, 1fr); */
		column-gap: 16px;
		margin: 0 0 0 0;
		width:100%;
		height:100%;
		overflow:hidden;
		background:black;
		cursor:url(https://theremin.app/assets/global/cursor1.svg), auto;
	}

	:global(button){
     background: transparent;
    border: none;
		padding:0;
	}
  :global(button:hover){
     cursor: url(https://theremin.app/assets/global/cursor4.svg) 21 20, pointer;
  }

	/* a subtle focus style for keyboard-input elements */
	:global(.text-input:focus) {
	  outline: 1px solid #aaa; /* Adjust to suit your tastes */
	}

	/* no outline for non-keyboard-inputs elements */
	:global(button:focus),
	:global(input:focus),
	:global(label:focus),
	:global(select:focus){
	  outline: none;
	}

	:global(body.user-is-tabbing *:focus),
	:global(body.user-is-tabbing input:focus + container) {
	  outline: 2px solid #7AACFE !important; /* for non-webkit browsers */
	  outline: 5px auto -webkit-focus-ring-color !important;
	}

	:global(a:hover){
	cursor: url(https://theremin.app/assets/global/cursor4.svg) 21 20, pointer;
	}
</style>

