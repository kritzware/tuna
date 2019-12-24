<template>
  <div class="container">
    <div class="d-flex justify-content-center">
      <h1>{{ getNoteFromFrequency(currentFrequency) }}</h1>
      <h2>{{ currentFrequency }}hz</h2>
      <button @click="active = !active">Toggle signal</button>
    </div>
  </div>
</template>

<script>
const N = 1024;
const E2 = 82.41;
const A2 = 110;
const D3 = 146.8;
const G3 = 196;
const B3 = 246.9;
const E4 = 329.6;

export default {
  data() {
    return {
      analyser: null,
      frequencies: null,
      amplitude: null,
      currentFrequency: 0,
      active: true,
    };
  },
  mounted() {
    const success = stream => {
      const audioContext = new AudioContext();
      this.analyser = audioContext.createAnalyser();

      this.frequencies = new Float32Array(this.analyser.frequencyBinCount);
      this.amplitude = new Uint8Array(this.analyser.frequencyBinCount);

      const volume = audioContext.createGain();
      const microphone = audioContext.createMediaStreamSource(stream);

      microphone.connect(volume);
      microphone.connect(this.analyser);

      renderFrame();
    };

    navigator.getUserMedia(
      {
        audio: true,
        video: false,
      },
      success,
      error => {
        console.log("Failed to access microphone input:");
        console.log(error);
      }
    );

    const renderFrame = () => {
      setTimeout(requestAnimationFrame(renderFrame), 100);

      if (!this.active) {
        return;
      }

      this.analyser.getFloatFrequencyData(this.frequencies);
      this.analyser.getByteTimeDomainData(this.amplitude);

      const frequency = this.getHighestFrequency(this.frequencies);
      this.currentFrequency = frequency;

      console.log(frequency, this.getNoteFromFrequency(frequency));
    };

    // const handleSuccess = stream => {
    //   const context = new AudioContext();
    //   const source = context.createMediaStreamSource(stream);
    //   const processor = context.createScriptProcessor(N, 1, 1);

    //   source.connect(processor);
    //   processor.connect(context.destination);

    //   processor.onaudioprocess = e => {
    //     // Do something with the data, e.g. convert it to WAV
    //     const data = e.inputBuffer.getChannelData(0);
    //     const ft = this.calculateHertz(data);
    //     console.log(ft);
    //   };
    // };

    // navigator.mediaDevices
    //   .getUserMedia({ audio: true, video: false })
    //   .then(handleSuccess);
  },
  methods: {
    fourierTransform() {
      let ft = 0;
      for (let i = 0; i < N; i++) {
        const x = data[i];
        const y = -2 * Math.PI * x * (i / N);
        const val = x * Math.pow(Math.E, y);
        ft += val;
      }
      return ft / N;
    },
    getHighestFrequency(frequencies) {
      const rate = 22050 / 1024;

      let maxI = frequencies[0];
      let max = frequencies[0];

      for (let i = 0; frequencies.length > i; i++) {
        let oldmax = parseFloat(max);
        let newmax = Math.max(max, frequencies[i]);
        if (oldmax !== newmax) {
          max = newmax;
          maxI = i;
        }
      }
      return maxI * rate;
    },
    getNoteFromFrequency(freq) {
      // E2=82.41Hz, A2=110Hz, D3=146.8Hz, G3=196Hz, B3=246.9Hz, E4=329.6Hz
      const closeTo = (i, target, precision = 10) =>
        i <= target + precision && i >= target - precision;

      if (closeTo(this.currentFrequency, E4)) {
        return "E4";
      }
      if (closeTo(this.currentFrequency, B3)) {
        return "B3";
      }
      if (closeTo(this.currentFrequency, G3)) {
        return "G3";
      }
      if (closeTo(this.currentFrequency, D3)) {
        return "D3";
      }
      if (closeTo(this.currentFrequency, A2)) {
        return "A2";
      }
      if (closeTo(this.currentFrequency, E2)) {
        return "E2";
      }

      return "unknown";
    },
  },
};
</script>

<style>
.container {
  margin: 0 auto;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}

.title {
  font-family: "Quicksand", "Source Sans Pro", -apple-system, BlinkMacSystemFont,
    "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
  display: block;
  font-weight: 300;
  font-size: 100px;
  color: #35495e;
  letter-spacing: 1px;
}

.subtitle {
  font-weight: 300;
  font-size: 42px;
  color: #526488;
  word-spacing: 5px;
  padding-bottom: 15px;
}

.links {
  padding-top: 15px;
}
</style>
