<script>
  import {
    videoReady,
    active,
    thereminPos,
    thereminMobilePos,
    poseNetRes,
    videoPos,
    WIDTH,
    toneOutput,
    bgPos,
    analyser,
    mouseOverride,
    gestures,
    enableMIDI,
    MIDITextSprite,
    videoMask,
  } from "../../stores.js";
  import { tweened } from "svelte/motion";
  import { constrain, getDistance } from "../../helpers.js";
  import { sineInOut } from "svelte/easing";

  export let app = null;
  export let stage = null;
  export let graphicsGroup = null;

  let ratio = 0;
  const sineInOut0_1 = tweened(0, {
    duration: 1000,
    easing: sineInOut,
  });
  const hideLights0_1 = tweened(1, {
    duration: 1000,
    easing: sineInOut,
  });

  let graphics = new PIXI.Graphics();
  graphics.parentGroup = graphicsGroup;
  graphics.alpha = 0.7;

  let bgGraphics = new PIXI.Graphics();
  bgGraphics.parentGroup = PIXI.lights.diffuseGroup;
  bgGraphics.alpha = 0.7;



  let graphics3 = new PIXI.Graphics();
  graphics3.parentGroup = PIXI.lights.diffuseGroup;

  stage.addChild(
    graphics,
    bgGraphics,
    graphics3
  );

  const createAudioPoints = (baseValues) => {
    let newArr = [];
    for (let i = 0; i < baseValues.length; i++) {
      const amplitude = baseValues[i];
      const x =
        $WIDTH > 600
          ? (($thereminPos.width * 0.2495) / (baseValues.length - 1)) * i
          : (($thereminMobilePos.width * 0.87) / (baseValues.length - 1)) * i;
      const y =
        $WIDTH > 600
          ? amplitude * $thereminPos.height * 0.04
          : amplitude * $thereminMobilePos.height * 0.15;
      newArr.push({ x: x, y: y });
    }
    return newArr;
  };


  let bgLights = [
    {
      x: 0.398,
      y: 0.205,
      r: 0.0025,
      color: 0xe54646,
      speed: 5,
      step: 0.4,
      pause: true,
    },
    {
      x: 0.359,
      y: 0.205,
      r: 0.0025,
      color: 0xe54646,
      speed: 2,
      step: 0.4,
      pause: false,
    },
    {
      x: 0.24,
      y: 0.577,
      w: 0.00525,
      color: 0xffff33,
      speed: 2,
      step: 0.2,
      pause: false,
    },
    {
      x: 0.14,
      y: 0.377,
      w: 0.0025,
      color: 0xffffff,
      speed: 2,
      step: 0.5,
      pause: true,
    },
    {
      x: 0.759,
      y: 0.3005,
      r: 0.00125,
      color: 0xffffff,
      speed: 1,
      step: 0.4,
      pause: false,
    },
    {
      x: 0.959,
      y: 0.5115,
      r: 0.00125,
      color: 0xffffff,
      speed: 1,
      step: 0.7,
      pause: false,
    },
    {
      x: 0.899,
      y: 0.6215,
      r: 0.00125,
      color: 0xffffff,
      speed: 1,
      step: 0.9,
      pause: false,
    },
    {
      x: 0.545,
      y: 0.541,
      r: 0.00125,
      color: 0xabf3fd,
      speed: 2,
      step: 0.1,
      pause: true,
    },
    {
      x: 0.545,
      y: 0.559,
      r: 0.00125,
      color: 0xabf3fd,
      speed: 2,
      step: 0.4,
      pause: true,
    },
    {
      x: 0.545,
      y: 0.577,
      r: 0.00125,
      color: 0xabf3fd,
      speed: 2,
      step: 0.7,
      pause: true,
    },
  ];

  let audioArr = [0, 0];
  let TIME = 0;
  var before;
  var delay = 1000 / 24;

  const drawAudio = (now = performance.now()) => {
    if (!before) before = now;

    if (now - before > delay) {
      //Draw Audio Analyser 24 FPS
      if ($analyser) {
        audioArr = createAudioPoints($analyser.getValue());
        graphics.clear();
        graphics.lineStyle(2, 0xe54646);
        graphics.lineAlpha =
          !$active && $WIDTH > 600 ? Math.abs(Math.sin(TIME * 5)) : 1;
        if (audioArr) {
          audioArr.forEach((e, i) => {
            if (i === 0) {
              graphics.moveTo(e.x, e.y);
            }
            graphics.lineTo(e.x, e.y);
          });
        }
      }

      before = now;
    }
  };



  const drawLights = () => {
    if ($bgPos) {
      bgGraphics.clear();
      bgLights.forEach((e, i) => {
        let opacity = e.pause
          ? constrain(Math.sin(TIME * e.speed), { min: 0, max: 1 })
          : Math.abs(Math.sin(TIME * e.speed));
        let opacityStepped = opacity > e.step ? 0.7 : 0;

        bgGraphics.beginFill(e.color, opacityStepped * $hideLights0_1);

        if (e.r) {
          bgGraphics.drawEllipse(
            $bgPos.x + $bgPos.width * e.x,
            $bgPos.y + $bgPos.height * e.y,
            $bgPos.width * e.r,
            $bgPos.width * e.r
          );
        }
        if (e.w) {
          bgGraphics.drawRoundedRect(
            $bgPos.x + $bgPos.width * e.x,
            $bgPos.y + $bgPos.height * e.y,
            $bgPos.width * e.w,
            $bgPos.width * e.w,
            $bgPos.width * e.w * 0.3
          );
        }
      });
    }
  };

  const drawGuides = () => {
    if (!$toneOutput) {
      return;
    } else {
      graphics3.clear();
      if (!$toneOutput.glide) {
        
        if(!$enableMIDI){
          for (let i = 0; i < $toneOutput.total + 1; i++) {
            graphics3.lineStyle(2, 0xffffff);
            graphics3.moveTo(
              $thereminPos.x + ($thereminPos.width * i) / $toneOutput.total,
              0
            );
            graphics3.lineTo(
              $thereminPos.x + ($thereminPos.width * i) / $toneOutput.total,
              $thereminPos.y + $thereminPos.height
            );
          }
        }
        
        let currentColor = $gestures ? 0xffff33 : 0xffff33;
        graphics3.lineStyle(2, currentColor);
        graphics3.beginFill(currentColor, 1);

        if(!$enableMIDI) {
          graphics3.drawRect(
            $thereminPos.x +
              ($thereminPos.width * $toneOutput.index.x) / $toneOutput.total,
            0,
            $thereminPos.width / $toneOutput.total,
            $thereminPos.y + $thereminPos.height
          );
        }
        
        if ($enableMIDI) {
          //trailing squares + shape
          graphics3.lineStyle(0, currentColor);
          let clampedXmin = Math.max($toneOutput.index.x-1,0)
          let clampedXmax = Math.min($toneOutput.index.x+1,$toneOutput.total-1)
          let clampedYmin = Math.max($toneOutput.index.y-1,0)
          let clampedYmax = Math.min($toneOutput.index.y+1,$toneOutput.total-1)
          let trails = {
            left:{
              x:clampedXmin,
              y:$toneOutput.index.y,
              a:.5
            },
            right:{
              x:clampedXmax,
              y:$toneOutput.index.y,
              a:.5
            },
            top:{
              x:$toneOutput.index.x,
              y:clampedYmin,
              a:.5
            },
            bottom:{
              x:$toneOutput.index.x,
              y:clampedYmax,
              a:.5
            },
            left2:{
              x:Math.max($toneOutput.index.x-2,0),
              y:$toneOutput.index.y,
              a:$WIDTH<600 ? .125 : 0
            },
            right2:{
              x:Math.min($toneOutput.index.x+2,$toneOutput.total-1),
              y:$toneOutput.index.y,
              a:$WIDTH<600 ? .125 : 0
            },
            top2:{
              x:$toneOutput.index.x,
              y:Math.max($toneOutput.index.y-2,0),
              a:$WIDTH>600 ? .125 : 0
            },
            bottom2:{
              x:$toneOutput.index.x,
              y:Math.min($toneOutput.index.y+2,$toneOutput.total-1),
              a:$WIDTH>600 ? .125 : 0
            },
            topleft:{
              x:clampedXmin,
              y:clampedYmin,
              a:.25
            },
            topright:{
              x:clampedXmax,
              y:clampedYmin,
              a:.25
            },
            bottomleft:{
              x:clampedXmin,
              y:clampedYmax,
              a:.25
            },
            bottomright:{
              x:clampedXmax,
              y:clampedYmax,
              a:.25
            },
            
          }
          
          Object.keys(trails).forEach(e=>{
            graphics3.beginFill(currentColor, trails[e].a);
            graphics3.drawRect(
              $thereminPos.x +
                ($thereminPos.width * trails[e].x) / $toneOutput.total,
              (($thereminPos.y + $thereminPos.height) * trails[e].y) /
                $toneOutput.total,
              $thereminPos.width / $toneOutput.total,
              ($thereminPos.y + $thereminPos.height) / $toneOutput.total
            );
          })          
          graphics3.lineStyle(2, currentColor);
          graphics3.beginFill(currentColor, 1);
          graphics3.drawRect(
            $thereminPos.x +
              ($thereminPos.width * $toneOutput.index.x) / $toneOutput.total,
            (($thereminPos.y + $thereminPos.height) * $toneOutput.index.y) /
              $toneOutput.total,
            $thereminPos.width / $toneOutput.total,
            ($thereminPos.y + $thereminPos.height) / $toneOutput.total
          );
        } 
      }
    }
  };

  app.ticker.add(() => {
    drawAudio();
    drawGuides();
    drawLights();
    TIME += 0.01;
  });

  $: {
    graphics.x =
      $WIDTH > 600
        ? $thereminPos.x + $thereminPos.width * 0.379
        : $thereminMobilePos.x + $thereminMobilePos.width * 0.065;
    graphics.y =
      $WIDTH > 600
        ? $thereminPos.y + $thereminPos.height * 0.852
        : $thereminMobilePos.y + $thereminMobilePos.height * 0.71 + 0;

    // graphics2.x = $videoPos.x;
    // graphics2.y = $videoPos.y;

    // maskGraphic.clear();
    // maskGraphic.beginFill(0xffffff, 1);

    graphics3.x = 0;
    graphics3.y = 0;
    graphics3.alpha = 0.5 * $sineInOut0_1;

    // if ($videoReady) {
    //   let offset = {
    //     x: $videoPos.width * 0.08,
    //     y: $videoPos.width * 0.035,
    //   };
    //   maskGraphic.drawRoundedRect(
    //     $videoPos.x + offset.x,
    //     $videoPos.y + offset.y,
    //     $videoPos.width - offset.x * 2,
    //     $videoPos.height - offset.y * 2,
    //     $videoPos.width * 0.25
    //   );
    // }

    if ($active && $sineInOut0_1 === 0) {
      sineInOut0_1.set(1);
    }
    if (!$active && $sineInOut0_1 === 1) {
      sineInOut0_1.set(0);
    }
  }

  $: {
    if ($enableMIDI) {
      if ($hideLights0_1 === 1) {
        hideLights0_1.set(0);
      }
    } else {
      if ($hideLights0_1 === 0) {
        hideLights0_1.set(1);
      }
    }
  }
</script>
