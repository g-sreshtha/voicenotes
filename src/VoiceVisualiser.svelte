<script lang="ts">
  import { onMount } from 'svelte';
  import p5 from 'p5';

  let sketchContainer: HTMLDivElement;
  let isStarted = false;
  let statusText = 'Click to start visualizing your voice';

  // Store mic and fft in component scope instead of window
  let micInstance: any;
  let fftInstance: any;

  function toggleMic() {
    if (micInstance) {
      if (!isStarted) {
        micInstance.start();
        fftInstance.setInput(micInstance);
        isStarted = true;
        statusText = 'Listening... Speak or make sounds!';
      } else {
        micInstance.stop();
        isStarted = false;
        statusText = 'Microphone stopped';
      }
    }
  }

  onMount(() => {
    const sketch = (p: any) => {
      let mic: any, fft: any;

      p.setup = () => {
        p.createCanvas(800, 600);
        // @ts-ignore - p5.sound loaded from CDN
        mic = new p5.AudioIn();
        // @ts-ignore - p5.sound loaded from CDN
        fft = new p5.FFT(0.8, 256);
        
        // Store in component scope instead of window
        micInstance = mic;
        fftInstance = fft;
      };

      p.draw = () => {
        p.background(20, 20, 40);

        if (isStarted) {
          let spectrum = fft.analyze();

          // Frequency bars
          p.noStroke();
          for (let i = 0; i < spectrum.length; i++) {
            let x = p.map(i, 0, spectrum.length, 0, p.width);
            let h = p.map(spectrum[i], 0, 255, 0, p.height);
            
            let hue = p.map(i, 0, spectrum.length, 0, 360);
            p.colorMode(p.HSB);
            p.fill(hue, 80, 90, 0.8);
            
            let barWidth = p.width / spectrum.length * 2;
            p.rect(x, p.height - h, barWidth, h);
          }

          p.colorMode(p.RGB);

          // Waveform
          p.stroke(255, 255, 255, 150);
          p.strokeWeight(2);
          p.noFill();
          p.beginShape();
          let waveform = fft.waveform();
          for (let i = 0; i < waveform.length; i++) {
            let x = p.map(i, 0, waveform.length, 0, p.width);
            let y = p.map(waveform[i], -1, 1, p.height * 0.3, p.height * 0.7);
            p.vertex(x, y);
          }
          p.endShape();

          // Energy levels
          let bass = fft.getEnergy("bass");
          let mid = fft.getEnergy("mid");
          let treble = fft.getEnergy("treble");

          drawEnergyCircle(p, 100, 100, bass, p.color(255, 100, 100));
          drawEnergyCircle(p, p.width/2, 100, mid, p.color(100, 255, 100));
          drawEnergyCircle(p, p.width - 100, 100, treble, p.color(100, 100, 255));

          // Labels
          p.fill(255);
          p.noStroke();
          p.textAlign(p.CENTER);
          p.textSize(12);
          p.text('BASS', 100, 180);
          p.text('MID', p.width/2, 180);
          p.text('TREBLE', p.width - 100, 180);
        } else {
          p.fill(255, 255, 255, 100);
          p.textAlign(p.CENTER, p.CENTER);
          p.textSize(24);
          p.text('Click "Start Microphone" to begin', p.width/2, p.height/2);
        }
      };

      function drawEnergyCircle(p: any, x: number, y: number, energy: number, col: any) {
        let size = p.map(energy, 0, 255, 20, 80);
        p.fill(col);
        p.noStroke();
        p.circle(x, y, size);
        
        p.stroke(col);
        p.strokeWeight(2);
        p.noFill();
        p.circle(x, y, size + 10);
      }
    };

    const myp5 = new p5(sketch, sketchContainer);

    return () => {
      if (micInstance) {
        micInstance.stop();
      }
      myp5.remove();
    };
  });
</script>

<style>
  .container {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    min-height: 100vh;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    padding: 20px;
  }

  .canvas-wrapper {
    box-shadow: 0 10px 40px rgba(0,0,0,0.3);
    border-radius: 10px;
    overflow: hidden;
  }

  .controls {
    margin-top: 20px;
    text-align: center;
  }

  button {
    background: white;
    border: none;
    padding: 15px 30px;
    font-size: 16px;
    border-radius: 25px;
    cursor: pointer;
    box-shadow: 0 4px 15px rgba(0,0,0,0.2);
    transition: all 0.3s ease;
    font-weight: bold;
    color: #667eea;
  }

  button:hover {
    transform: translateY(-2px);
    box-shadow: 0 6px 20px rgba(0,0,0,0.3);
  }

  button:active {
    transform: translateY(0);
  }

  .status {
    color: white;
    margin-top: 15px;
    font-size: 14px;
    text-shadow: 0 2px 4px rgba(0,0,0,0.3);
  }
</style>

<div class="container">
  <div class="canvas-wrapper">
    <div bind:this={sketchContainer}></div>
  </div>
  <div class="controls">
    <button on:click={toggleMic}>
      {isStarted ? 'Stop Microphone' : 'Start Microphone'}
    </button>
    <div class="status">{statusText}</div>
  </div>
</div>

