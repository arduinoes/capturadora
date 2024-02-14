<script setup>
import { ref, reactive } from "vue";

let mediaRecorder = ref('');
let recordedChunks = reactive([]);

let grabar = ref(true)

async function startRecording() {
  try {
    // Obtenemos el strem de la captura de pantalla
    const stream = await navigator.mediaDevices.getDisplayMedia({
      video: {
        displaySurface: "window"
      }
    });
    // Declaramos el elemento por ID
    const videoPreview = document.getElementById("videoPreview");

    videoPreview.srcObject = stream;

    // Lo grabamos/guardamos en mediaRecorder
    mediaRecorder.value = new MediaRecorder(stream);

    // Lo gruardamos en trozos mientras esté disponible
    mediaRecorder.value.ondataavailable = (event) => {
      if (event.data.size > 0) {
        recordedChunks.push(event.data);
      }
    };

    // Creamos un video con los trozos cuando acaba
    mediaRecorder.value.onstop = () => {
      const videoBlob = new Blob(recordedChunks, { type: "video/mp4" });
      recordedChunks = [];
      const downloadLink = document.getElementById("downloadLink");
      const videoURL = URL.createObjectURL(videoBlob);
      downloadLink.href = videoURL;
      downloadLink.style.display = "block";
      downloadLink.download = "video.mp4";
    };

    mediaRecorder.value.start();
    grabar.value = !grabar.value;
  } catch (error) {
    console.error("Error al iniciar la grabación:", error);
  }
}

function stopRecording() {
  mediaRecorder.value.stop();
  grabar.value = !grabar.value
}
</script>

<template>
  <div class="container">
    <h1> Video Recorder</h1>
    <video id="videoPreview" autoplay></video>
    <div class="controls">
      <button :disabled="!grabar" @click="startRecording()" class="button start">🔴 Start Recording</button>
      <button :disabled="grabar" @click="stopRecording()" class="button stop">⏹️ Stop Recording</button>
    </div>
    <a id="downloadLink" class="download-link" style="display: none;">⬇️ Download Video</a>
  </div>
</template>

<style>
body {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  margin: 0;
  font-family: Arial, sans-serif;
  background-color: #f5f5f5;
}

.container {
  background-color: white;
  border-radius: 10px;
  box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.1);
  padding: 20px;
  text-align: center;
}

video {
  width: 100%;
  max-width: 400px;
  border-radius: 10px;
  margin-bottom: 20px;
}

.controls {
  display: flex;
  justify-content: center;
  gap: 10px;
}

.button {
  background-color: #4CAF50;
  border: none;
  color: white;
  padding: 10px 20px;
  border-radius: 5px;
  cursor: pointer;
  font-size: 14px;
  transition: background-color 0.3s, transform 0.2s;
}

.button:hover {
  background-color: #45a049;
  transform: scale(1.05);
}

.button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}

.download-link {
  display: block;
  margin-top: 10px;
  color: #333;
  text-decoration: none;
  font-size: 14px;
}
</style>