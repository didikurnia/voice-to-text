<template>
<!--  <Tutorial/>-->
  <div>
    <button @click="startRecognition">Start Recognition</button>
    <button @click="stopRecognition">Stop Recognition</button>
    <button @click="generateAndDownloadTxtFile">Generate Text</button>
    <button @click="startRecording">Start Recording</button>
    <button @click="stopRecording">Stop Recording</button>
    <button @click="playAudio">Play Audio</button>
    <button @click="downloadAudioFile(recordedChunks, 'audio.mp3')">Download Audio</button>

    <div>
      <textarea
        v-model="textVoice"
        rows="25"
        cols="75"
      >

      </textarea>
    </div>
  </div>

</template>
<script>
import { SpeechConfig, AudioConfig, SpeechRecognizer } from 'microsoft-cognitiveservices-speech-sdk';

export default {
  name: 'IndexPage',
  data: () => ({
    recognizerVoice: null,
    recognizer: null,
    textVoice: '',
    recording: false,
    recordedChunks: [],
  }),
  methods: {
    startRecognition() {
      const sdk = require("microsoft-cognitiveservices-speech-sdk");

      // Replace with your Azure Speech resource's subscription key and region
      const subscriptionKey = "06db53b450d84de49bae534d82a95d1e";
      const region = "southeastasia"; // e.g., "westus"

      const speechConfig = sdk.SpeechConfig.fromSubscription(subscriptionKey, region);
      speechConfig.speechRecognitionLanguage = "en-US"; // Replace with your desired language

      const audioConfig = sdk.AudioConfig.fromDefaultMicrophoneInput();

      this.recognizer = new sdk.SpeechRecognizer(speechConfig, audioConfig);

      this.recognizer.recognizing = (_, { result }) => {
        console.log(`Recognizing: ${result.text}`);
      };

      this.recognizer.recognized = (_, { result }) => {
        this.textVoice += result.text + '\n';
        console.log(`Recognized: ${result.text}`);
      };

      this.recognizer.startContinuousRecognitionAsync(() => {
        console.log("Recognition started. Listening for speech...");
      }, (error) => {
        console.error(error);
      });
    },
    stopRecognition() {
      if (this.recognizer) {
        this.recognizer.stopContinuousRecognitionAsync(() => {
          console.log("Recognition stopped.");
        }, (error) => {
          console.error(error);
        });
      }
    },
    downloadTextFile(textData, fileName) {
      // Create a new Blob object with the string data and the desired file type
      const blob = new Blob([textData], { type: 'text/plain' });

      // Create a new URL object from the Blob object
      const url = URL.createObjectURL(blob);

      // Create a new anchor element and set its href and download attributes
      const a = document.createElement('a');
      a.href = url;
      a.download = fileName;

      // Dispatch a click event on the anchor element to trigger the file download
      a.dispatchEvent(new MouseEvent('click'));

      // Clean up by revoking the URL object
      URL.revokeObjectURL(url);
    },
    downloadAudioFile(audioData, fileName) {
      // const audio = new Audio(URL.createObjectURL(new Blob(this.recordedChunks, { type: 'audio/mp3' })));

      // Create a new Blob object with the string data and the desired file type
      const blob = new Blob(audioData, { type: 'audio/mp3' });

      // Create a new URL object from the Blob object
      const url = URL.createObjectURL(blob);

      // Create a new anchor element and set its href and download attributes
      const a = document.createElement('a');
      a.href = url;
      a.download = fileName;

      // Dispatch a click event on the anchor element to trigger the file download
      a.dispatchEvent(new MouseEvent('click'));

      // Clean up by revoking the URL object
      URL.revokeObjectURL(url);
    },
    generateAndDownloadTxtFile() {
      // Call the downloadTextFile function with the text data and a file name
      this.downloadTextFile(this.textVoice, 'example.txt');
    },
    async startRecording() {
      if (navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {
        const mediaStream = await navigator.mediaDevices.getUserMedia({ audio: true });
        const audioConfig = AudioConfig.fromStreamInput(mediaStream);
        const speechConfig = SpeechConfig.fromSubscription('06db53b450d84de49bae534d82a95d1e', 'southeastasia');
        const recognizer = new SpeechRecognizer(speechConfig, audioConfig);

        recognizer.recognizeOnceAsync((result) => {
          console.log('Transcription:', result.text);
        });

        recognizer.startContinuousRecognitionAsync(() => {
          this.recording = true;
          console.log('Recording started');
        }, (error) => {
          console.log(`Error starting recording: ${error}`);
        });

        recognizer.recognized = (_, event) => {
          if (event.result.text) {
            this.textVoice += event.result.text + '\n';
            console.log(`Transcription: ${event.result.text}`);
          }
        };

        recognizer.recognizing = (_, event) => {
          if (event.result.text) {
            console.log(`Recognizing: ${event.result.text}`);
          }
        };

        this.mediaRecorder = new MediaRecorder(mediaStream);
        this.mediaRecorder.addEventListener('dataavailable', (event) => {
          this.recordedChunks.push(event.data);
        });
        this.mediaRecorder.start();

        // recognizer.sessionStopped = () => {
        //   this.recording = false;
        //   console.log('Recording stopped');
        //
        //   const blob = new Blob(this.recordedChunks, { type: 'audio/mp3' });
        //   saveAs(blob, 'audio.mp3');
        //
        //   this.recordedChunks = [];
        //   recognizer.close();
        // };

        this.recognizerVoice = recognizer;
      }
    },
    stopRecording() {
      if (this.recognizerVoice && this.mediaRecorder) {
        this.mediaRecorder.stop();
        this.mediaRecorder = null;

        this.recognizerVoice.stopContinuousRecognitionAsync(() => {
          // this.downloadAudioFile(this.recordedChunks, 'audio.mp3')
          console.log('Stopping recording');
        });
      }
    },
    playAudio() {
      const audio = new Audio(URL.createObjectURL(new Blob(this.recordedChunks, { type: 'audio/mp3' })));
      audio.play();
    }
  },
  beforeDestroy() {
    this.stopRecognition();
  },
}
</script>
