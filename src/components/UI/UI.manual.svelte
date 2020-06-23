<script>
import Carousel from '@beyonk/svelte-carousel'
import {loaded,active,WIDTH,HEIGHT,SCALE,manual} from '../stores.js';
import { onMount } from 'svelte';
import Logo from '../icons/logo.svelte'; 
import Play from '../icons/play.svelte'; 
import About from '../icons/about.svelte'; 
import closeIcon from '../icons/close.svelte'; 
import ButtonCircle from './UIElements/UI.buttoncircle.svelte';
import BasicControls from '../icons/basicControls.svelte'
import GestureControls from '../icons/gestureControls.svelte'
import MIDIControls from '../icons/MIDISupport.svelte'
import Arrow from '../icons/arrow.svelte'
import {midiList} from '../config.js'

let startSlide = 0
let currentSlide = startSlide
let slideCount = 4
const handleChange = (e) => {
	currentSlide = e.currentSlide
	slideCount = e.slideCount
	handlebuttons($manual)
}

const handlebuttons = () => {
	document.querySelectorAll('.carousel button').forEach((e,i)=>{
		e.tabIndex = ($manual) ? 0 : -1
		if(i===0 && currentSlide===0){
			e.classList.add('disabled')
		}else if(i===1 && currentSlide===slideCount-1){
			e.classList.add('disabled')
		}else{
			e.classList.remove('disabled')
		}
		
	})
}

onMount(async()=>{
	handlebuttons()
})
$:{
	handlebuttons()
}

</script>

<section class={$manual ? '' : 'hide'}>
	<ButtonCircle 
    icon={closeIcon} 
    setting={manual}
	settingState={false} 
	selected={true}
    styles={'--bgColor:var(--buttonColor);--svgColor:var(--iconColor);--svgColorHover:var(--iconColor);position:absolute;top:0px;right:0px;border-radius:0;border-top-right-radius: 16px;border-bottom-left-radius: 16px;z-index:2'}
	tabindex={$manual ? 0 : -1}
	/>
	<Carousel loop={false} perPage={1} startIndex={startSlide} duration={400} threshold={$WIDTH>600 ? 200 : 100} on:change={e=>{handleChange(e.detail)}}>

	<span class="control" slot="left-control">
		<Arrow color={'rgb(var(--arrowColor))'}/>
	</span>
		<div class="slide-content {(currentSlide === 0 && $loaded) ? 'current' : ''}" >
			<container>
			<div class="graphic">
				<h2>Theremin<br>Basics</h2>
				<div class="svg">
					<BasicControls color="rgb(var(--textColor1))" bgColor="rgb(var(--cardColor))"/>
				</div>
			</div>
			<div class="description">
				<p>The theremin is an electic musical instrument played by the movement of both hands in the space surrounding it. THEREMIX's default controls translates these 2 movements into 1 fluid cursor movement.</p>
				<hr>
				<ol>
					<li>
						<h3>Volume Antenna</h3>
						<p>
						Movement towards this antenna <span>(y-axis)</span> increases the <span>volume</span> of the current oscillator. 
						</p>
					</li>
					<li>
						<h3>Pitch Antenna</h3>
						<p>
						Movement towards this antenna <span>(x-axis)</span> increases the frequency of the current oscillator, raising the <span>pitch</span>. You can modify the <span>octave</span> and <span>scale</span> of the pitch antenna under the Settings panel.
						</p>
					</li>
					<li>
						<h3>Master Volume</h3>
						<p>
						Controls the overall volume <span>(-48db — 0db)</span> of the app.
						</p>
					</li>
					<li>
						<h3>Oscillator</h3>
						<p>
						Choose from 18 different oscillator waveforms to change the timbre of the theremin. Includes <span>Sine</span>, <span>Triangle</span>, <span>Sawtooth</span>, <span>Square</span>, <span>Pulse</span> oscillators as well as many other alternates. 
						</p>
						<p class="sub">
						<span class="keycap">O</span> to cycle between oscillators. 
						</p>
						<p class="sub">
						<span class="keycap">O</span> + Arrow Keys to switch back and forth.
						</p>
					</li>
					<li>
						<h3>Glide</h3>
						<p>
						Glide is how the theremin tone operates traditionally, <span>sliding smoothly</span> between pitches. This is on by default. Turning this off will snap the tone to a <span>perfect note</span>.  
						</p> 
						<p class="sub">
						Hold/Release <span class="keycap">G</span> to toggle on/off glide.
						</p>
					</li>
					<li>
						<h3>Key/Scale</h3>
						<p>
						Modifying this setting changes the freqency range of the theremin as well as the tones played when Glide is off. Choose between <span>12 different tonic keys</span> and <span>18 different scales types</span>. Octave range is also adjustable under Settings.
						</p>
						<p class="sub">
						<span class="keycap">K</span> or <span class="keycap">S</span> to cycle between keys and scales.
						</p>
						<p class="sub">
						Hold <span class="keycap">K</span> or <span class="keycap">S</span> + Arrow Keys to switch back and forth. 
						</p>   
					</li>
				</ol>
			</div>
			</container>
			<div class="gradient"></div>
		</div> 
		<div class="slide-content {(currentSlide === 1 && $loaded) ? 'current' : ''}">
			<container>
			<div class="graphic">
				<h2>Gesture<br>Controls</h2>
				<div class="svg">
					<img alt="Gesture controls" style="width:100%" src="https://cdn.glitch.com/bbfb2dd7-a8b0-4835-bdc2-c2fdffc99849%2FGif1.mp4.gif?v=1592903208068"/>
					<!-- <GestureControls color="rgb(var(--textColor1))" bgColor="rgb(var(--cardColor))"/> -->
				</div>
			</div>
			<div class="description">
				<p>Webcam motion capture allows you to experience the gestural movements of playing a physical theremin. Powered by <a href='https://github.com/tensorflow/tfjs-models/tree/master/posenet' target="blank">PoseNet</a>, a machine learning model which allows for real-time human pose estimation in the browser.</p>
				<hr>
				<ul>
					<li>
						<h3>2-Hands Controls</h3>
						<p>
						The <span>right hand controls pitch</span> while the <span>left hand controls volume</span>. These controls are mapped to how a physical theremin would operate. Movement towards the right antenna increases the frequency while movement towards the left antenna increases the amplitude of the waveform. 
						</p>
					</li>
					<li>
						<h3>1-Hand Controls</h3>
						<p>
						If only one hand is detected, <span>both volume and pitch are controlled by the dominate hand</span>. Mainly used when a user is viewing on mobile and needs one hand to hold the phone.
						</p>
					</li>
				</ul>
			</div>
			</container>
			<div class="gradient"></div>
		</div> 
		<div class="slide-content {(currentSlide === 2 && $loaded) ? 'current' : ''}">
			<container>
			<div class="graphic">
				<h2>Midi<br>Controls<span style="font-size:9px;margin-left: 8px">Beta<span></h2>
				<div class="svg">
					<img alt="MIDI controls" style="width:100%" src="https://cdn.glitch.com/bbfb2dd7-a8b0-4835-bdc2-c2fdffc99849%2Fgif2.mp4.gif?v=1592903207301"/>
					<!-- <MIDIControls color="rgb(var(--textColor1))" bgColor="rgb(var(--cardColor))"/> -->
				</div>
			</div>
			<div class="description">

				<p>A theremin can also be used as an <a href='https://www.youtube.com/watch?v=Hae0g-lDOqw' target="blank">alternative controller</a> for MIDI and other musical applications. This mode is currently in a beta and works best in Chrome. Movement towards X and Y axis modifies the timbre of the oscillators, changing variables such as <span>attack</span>, <span>sustain</span>, <span>decay</span>, and <span>release</span>. User uploaded MIDI to come in a future update.</p>
				<p class="sub small">
				<span class="keycap">M</span> to toggle Midi on/off. Arrow Keys to switch between songs in queue. 
				<br><br>
				List of the available MIDI demos. All credit for music, illustrations, and MIDI arranges go to their original creators.
				</p>
				<hr>
				<ol>
					{#each Object.keys(midiList) as midiTitle}
					<li>
						<h3 style="font-family:'Nicholson Beta';font-size:16px;margin-bottom:0;padding-top: 14px;">
							{midiTitle}
						</h3>
						<p style="font-family:'Nicholson Beta';font-size:14px;margin:0 0 8px 0;">{midiList[midiTitle].artist}</p>	
						<p style="margin:0;">		
						<a href="{midiList[midiTitle].original}" target="blank" style="margin-right:6px">Original Video</a><a href="{midiList[midiTitle].midi}" target="blank">MIDI</a> 
						</p>
					</li>
					{/each}
				</ol>
			</div>
			</container>
			<div class="gradient"></div>
		</div> 
		<div class="slide-content {(currentSlide === 3 && $loaded) ? 'current' : ''}">
				<container>
			<div class="graphic">
				<h2>Resources<br>and Links</h2>
				<div class="svg">
					<About color="rgb(var(--textColor1))" bgColor="rgb(var(--cardColor))"/>
				</div>
			</div>
			<div class="description">
				<p>This app celebrates the <span>100th anniversary of the theremin</span>, invented by Léon Theremin in 1920. A century after its conception, the theremin continues to be a flexible electronic instrument with an ethereal tone. <a href="https://glitch.com" target="blank">View Project on Github</a></p>
				<p class="small sub">This project was made possible with the following platforms and open-source resources:</p>
				<hr>
				<ul>
					<li>
						<h3>Glitch</h3>
						<p>Glitch is a friendly platform for all your web project needs, from something as simple as a webpage to something more complex involving servers. All code written is hosted on Glitch's servers, making it easy to preview your work without needing to install packages or starting a web server. Remixing on Glitch also allows you to use projects from other creators as a starting template and explore their code. Feel free to <a href="https://glitch.com/~theremix" target="blank">Remix this Project</a> on Glitch.
						</p>
						<p>
						<a href="https://glitch.com/" target="blank">Glitch.com</a>
						</p>
					</li>
					<li>
						<h3>Pixijs</h3>
						<p>Pixi.js is a lightweight open source 2D WebGL renderer. This app is currently running on Pixi V4 with <a href="https://github.com/pixijs/pixi-lights" target="blank">Pixi Lights</a> and <a href="https://github.com/pixijs/pixi-particles" target="blank">Pixi Particles</a> plugins.</p>
						<p>
						<a href="https://www.pixijs.com/" target="blank">Pixijs.com</a>
						</p>
					</li>
					<li>
						<h3>Tonejs</h3>
						<p>Tone.js is a framework for creating interactive music in the browser. It provides advanced scheduling capabilities, synths and effects, and intuitive musical abstractions built on top of the Web Audio API.</p>
						<p>
						<a href="https://tonejs.github.io/" target="blank">Tonejs.github.io</a>
						</p>
					</li>
					<li>
						<h3>PoseNet</h3>
						<p>PoseNet is a machine learning model which allows for real-time human pose estimation in the browser.</p>
						<p>
						<a href="https://github.com/tensorflow/tfjs-models/tree/master/posenet" target="blank">Github</a>
						</p>
					</li>
				</ul>
				<hr>
				<p class="sub small">This project is typeset in <span>Whirly Birdie</span> and <span>Nicholson Gothic</span>.</p>
			</div>
			</container>
			<div class="gradient"></div>
		</div> 
		
		
		
	<span class="control" slot="right-control">
		<Arrow color={'rgb(var(--arrowColor))'}/>
	</span>
	</Carousel>
</section>

<style>
	/* Colors */
	section{
		--cardColor: var(--offwhite);
		--containerColor:var(--crimson);
		--manualColor: 255, 211, 166;
		--descriptionColor:var(--offwhitetint);
		--cardGradient:linear-gradient(0deg, rgb(var(--descriptionColor)) 0%, rgba(var(--descriptionColor),0.920045518207283) 70%, rgba(var(--descriptionColor),0) 100%);
		--containerGradient:linear-gradient(0deg, rgb(var(--descriptionColor)) 0%, rgb(var(--descriptionColor)) 26%, rgb(var(--cardColor)) 30%, rgb(var(--cardColor)) 32%, rgb(var(--cardColor)) 100%);
		--textColor1:var(--crimson);
		--textColor2:var(--charcoal);
		--spanColor: var(--navy);
		--listColor:var(--charcoal);
		--buttonColor:var(--crimson);
		--iconColor:var(--offwhite);
		--arrowColor:var(--offwhite);
		--hrImage:var(--wave);
	}

	@media (prefers-color-scheme: dark) {
		section{
			--cardColor: 17,17,17;
			--manualColor: 50,42,42;
			--descriptionColor: 30,25,25;
			--arrowColor:var(--crimson);
			--textColor2:var(--offwhite);
			--listColor:var(--crimson);
			--hrImage:var(--waveRed);
			--spanColor: var(--sun);
		}
	}
	/* Colors */

	section{
		position:absolute;
		width:80vw;
		height:80vh;
		color:rgb(var(--textColor1));
		max-height: 100%;
		max-width:1200px;
		max-height:800px;
		display: flex;
		align-items: center;
		justify-content: center;
		transform-origin: bottom;
		-webkit-transform: translate3d(0, 0, 0);
		-moz-transform: translate3d(0, 0, 0);
		transform:translate3d(0, 0, 0);
		transition: transform .7s cubic-bezier(0.55, 1.32, 0.51, 0.97);
		
	}

	section.hide{
		-moz-transform: translate3d(111vw, 8vh, 0) rotate(10deg);
		-webkit-transform: translate3d(111vw, 8vh, 0) rotate(10deg);
		transform: translate3d(111vw, 8vh, 0) rotate(10deg);
		pointer-events: none;
	}
	:global(.slides){
		border-radius:16px;
		background:#FBDDCC;
		background: rgb(var(--manualColor));
		overflow: hidden;
		-webkit-backface-visibility: hidden;
		-moz-backface-visibility: hidden;
		-webkit-transform: translate3d(0, 0, 0);
		-moz-transform: translate3d(0, 0, 0);
	}
	:global(.slides:before){
	content: '';
    background-size: auto 10px;
    opacity: 0.05;
    position: absolute;
    border: none;
    height: 200%;
    width: 200%;
    left: -50%;
    top: -50%;
    background-position: 100% 100%;
    background-image:var(--wave);
    transform-origin: center;
    transform: rotate(45deg);
	}

	.slide-content{
		pointer-events:fill;
		width:80vw;
		height:80vh;
		max-width:1200px;
		max-height:800px;
		margin-left:0;
		background:rgb(var(--cardColor));
		opacity:0;
		transition: opacity .75s linear;
		border-radius: 16px;
		justify-content: center;
		align-items: center;
		overflow: hidden;
		-webkit-box-shadow: 0px 0px 53px -11px rgba(0,0,0,0.15);
		-moz-box-shadow: 0px 0px 53px -11px rgba(0,0,0,0.15);
		box-shadow: 0px 0px 53px -11px rgba(0,0,0,0.15);
		position:relative;  
		-webkit-backface-visibility: hidden;
		-moz-backface-visibility: hidden;
		-webkit-transform: translate3d(0, 0, 0);
		-moz-transform: translate3d(0, 0, 0);
	}
	.slide-content.current{
		opacity:1;
		transition: opacity .75s linear .5s;
	}
	container{
		 position:absolute;
		  height:100%;
		  width:100%;
		  display: flex;
		flex-direction: row;
	}
	.fullscreen container{
		overflow-y:scroll;
	}
	.graphic{
		height:calc(100% - 64px);
		flex:100;
		margin: 24px 24px 40px 24px;
		display: flex;
		flex-direction: column;
		pointer-events:none;
	}
	.svg{
		width:100%;
		flex-grow: 1;
		height:0;
		display: flex;
		flex-direction: column;
		align-content: center;
		justify-content: center;
	}
	h2{
		margin: 0;
		min-height: 56px;
		font-size: 40px;
		font-weight: normal;
		font-family:'Whirly Birdie';
		font-variation-settings: "wght" 100, "wdth" 140, "ital" 0;
		line-height: 1.1;
		/* font-feature-settings: 'ss01'; */
	}
	.description{
		width: calc(40% - 80px);
		min-width:320px;
		max-width:400px;
		padding: 40px 40px 40px 40px;
    	min-height: calc(100% - 80px);
		margin: 0;
		background:rgb(var(--descriptionColor));
		overflow-y: scroll;
		font-family: 'Nicholson Beta';
		font-size:18px;
		-webkit-backface-visibility: hidden;
		-moz-backface-visibility: hidden;
		-webkit-transform: translate3d(0, 0, 0);
		-moz-transform: translate3d(0, 0, 0);
	}

	.fullscreen .description{
		width:calc(100% - 80px);
		column-count: 2;
		column-fill: auto;
		column-gap:40px;
		padding: 40px;
		max-width:none;
		height:max-content;
		/* overflow-y:scroll; */
	}

	.gradient{
		opacity:0;
		width:100%;
		position:absolute;
		bottom:0;
		left:0;
		height:60px;
		background: rgb(var(--descriptionColor));
		background: var(--cardGradient);
		pointer-events:none;
		-webkit-backface-visibility: hidden;
		-moz-backface-visibility: hidden;
		-webkit-transform: translate3d(0, 0, 0);
		-moz-transform: translate3d(0, 0, 0);
	}

	.fullscreen .gradient{
		opacity:1
	}
	.description p{
		margin:0 0 24px 0;
		width:100%;
		color:rgba(var(--textColor2),1);
		font-size:18px;
		/* line-height:1.4; */
	}
	.description li p{
		font-size:14px;
		margin:0 0 8px 0;
		/* line-height: 1.5; */
	}
	.description p.sub{
		color:rgba(var(--textColor2),.6);
	}
	.description p.small{
		margin:0 0 8px 0;
		font-size:14px;
		/* line-height:1.5; */
	}


	hr{
		background-size: auto 4px;
		/* opacity:0.6; */
		border: none;
    	height: 8px;
        background-repeat: repeat-x;
        background-position: 100% 100%;
        background-image:var(--hrImage);
		margin:24px 0 24px 0;
	}
	a{
		color:rgb(var(--spanColor));
		display: inline-block;
		position: relative;
		/* text-decoration: none; */
		width: max-content;
	}
	ol{
		list-style: none;
		margin:0 0 24px 0;
		width: calc(100% - 72px);
    	padding-inline-start: 72px;
	}
	ul{
		list-style: none;
		margin:0 0 24px 0;
		width: 100%;
    	padding-inline-start: 0;
	}
	li{
		counter-increment: step-counter;
		position: relative;
		margin:0 0 24px 0;
		color:rgba(var(--textColor2),1);
		font-size:14px;
		line-height: 1.5;
	}
	span{
		color:rgb(var(--spanColor))
	}

	ol li::before {
	position: absolute;
    top: 0;
    content: counter(step-counter);
    left: -68px;
    font-size: 48px;
    color: rgb(var(--listColor));
    font-family: 'Whirly Birdie';
    font-weight: normal;
    border-radius: 4px;
    display: flex;
    opacity: .3;
    align-items: center;
    justify-content: center;
    width: 32px;
    font-variation-settings: "wght" 60, "wdth" 80, "ital" 0;
	}

	h3{
	font-family: 'Whirly Birdie';
    font-size: 14px;
    margin: 0 0px 8px 0px;
    padding-top: 12px;
    font-weight: normal;
	font-variation-settings: "wght" 90, "wdth" 110, "ital" 0;
	color:rgb(var(--textColor1));
	display:flex;
	}
	.keycap{
		padding: 3px 4px 3px 4px;
		font-family:'Whirly Birdie';
		display:inline-block;
		text-align: center;
		width:14px;
		font-variation-settings: "wght" 80, "wdth" 110, "ital" 0;
		background: rgba(var(--crimson),.3);
		color:rgb(var(--textColor2));
		font-size: 12px;
		border-radius: 2px;
	}
	
	:global(.carousel>ul>li.active){
		background-color:rgb(var(--textColor1)) !important;
		
	}
	:global(.carousel>ul>li){
		background-color:rgba(var(--textColor1),.2) !important;
	}

	:global(.right){
		width:56px !important;
		height:56px !important;
		right: calc(-56px - 16px) !important;
		transform-origin: center;
		transform: rotate(180deg)
	}
	:global(.left){
		width:56px !important;
		height:56px !important;
    	left: calc(-56px - 16px) !important;
	}
	:global(.left),:global(.right){
		border-radius: 50%;
		/* background:rgba(255,255,255,0.1) !important */
	}
	:global(.left:hover),:global(.right:hover){
		background:rgba(var(--offwhite),0.15) !important
	}
	:global(.left.disabled),:global(.right.disabled){
		display: none;
	}

	@media screen and (max-width:1024px){
		h2{
			font-size: 32px;
		}
		section{
			width:95vw;
		}
		.slide-content{
			width:95vw;
		}
		.description{
			max-width:unset
		}
		:global(.right){
			display: none;
		}
		:global(.left){
			display: none;
		}
	
	}
	@media screen and (max-width: 800px) {
      section{
		  width:95vw;
		  height:calc(80vh);
	  }
	  h2{
		  font-size: 28px;
		  margin-bottom: 24px;
	  }
	  .slide-content{
		  width:95vw;
		  height: calc(80vh); 
			background: var(--containerGradient)
	  }
	  .slide-content.fullscreen{
		  background:rgb(var(--cardColor))
	  }
	  container{
    	flex-direction: column;
		  overflow-y: scroll;
	  }
	  .gradient{
		  opacity:1;
	  }
	  .graphic{
		  height:max-content;
		  min-height: max-content;
		  width:calc(100% - 48px);
		  margin: 24px 24px 0px 24px;
	  }
	  .svg{
		  height:auto;
		  flex-grow: unset;
		  display: block;
	  }
	  .fullscreen .svg{
		  flex:100;
		  display: flex;
		  justify-content: center;
		  align-items: center;
	  }
	  .description{
		  margin: 24px 0px 0px 0px;
		  font-size:16px;
		  min-width:0;
		  min-height: max-content;
		  flex:100;
			-webkit-box-flex: 100;
    		-webkit-flex: 100;
		  padding: 24px 24px 40px 24px;
    		width: calc(100% - 48px);
	  }
	  .fullscreen .description{
		  margin:0;
		  column-count:1;
		  padding:24px 24px 40px 24px;
		  width: calc(100% - 48px);
		  background:rgb(var(--descriptionColor));
	  }
	  .description>p{
		  margin:0px 0px 24px 0px;
	  }
	  ol li::before {
		  left: -64px;
	  }
	 
	  	:global(.right){
			display: none;
		}
		:global(.left){
			display: none;
		}
	  
    }
</style>