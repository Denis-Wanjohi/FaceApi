<template>
  <div id="app" style="display: flex;height: 500;">
    <video ref="video" width="200"  autoplay></video>
    <!-- <canvas ref="canvas" width="640" height="480"></canvas> -->
  </div>
  <!-- <p v-if="detection">{{detection.length}}</p> -->
   <p v-if="expression">
    {{ face }}
   </p>

</template>

<script setup>
import { ref, watch,onMounted } from 'vue';
import * as faceapi from 'face-api.js';

const video = ref(null);
const canvas = ref(null);
const detection = ref(null);
const expression = ref([])
const max = ref([])
const face = ref([])

watch(detection,()=>{
  if(detection.length != 0){
    // console.log(detection)
    detection.value.forEach(element => {
      // expression.value = element.expressions
      expression.value = []
      for (const key in element.expressions) {
          if (element.expressions.hasOwnProperty(key)) {
 
              expression.value.push(element.expressions[key])
          }
      }

      max.value = Math.max(...expression.value)

      for (const key in element.expressions) {
          if (element.expressions.hasOwnProperty(key)) {
              if(element.expressions[key] == max.value){
               
                 face.value = key
              }else{
                console.log('face not detected')
              }
          }
      }
    });
  }

})

const loadModels = async () => {
  try {
    await faceapi.nets.tinyFaceDetector.loadFromUri('/models');
    await faceapi.nets.faceLandmark68Net.loadFromUri('/models');
    await faceapi.nets.faceRecognitionNet.loadFromUri('/models');
    await faceapi.nets.faceExpressionNet.loadFromUri('/models');
    console.log("Models loaded successfully");
  } catch (error) {
    console.error("Error loading models:", error);
  }
};

const startVideoStream = async () => {
  const stream = await navigator.mediaDevices.getUserMedia({ video: {} });
  video.value.srcObject = stream;
  console.log("Video stream started");
  detectFaces();
};

const detectFaces = async () => {
  const displaySize = { width: canvas.value.width, height: canvas.value.height };
  faceapi.matchDimensions(canvas.value, displaySize);

  setInterval(async () => {
    const detections = await faceapi.detectAllFaces(video.value,
      new faceapi.TinyFaceDetectorOptions()).withFaceLandmarks().withFaceDescriptors().withFaceExpressions();
    
    // console.log("Detections:", detections); // Log detections
    detection.value = detections
    const resizedDetections = faceapi.resizeResults(detections, displaySize);
    const ctx = canvas.value.getContext('2d');

    // Clear the canvas
    ctx.clearRect(0, 0, canvas.value.width, canvas.value.height);

    // Draw detections and landmarks
    faceapi.draw.drawDetections(canvas.value, resizedDetections);
    faceapi.draw.drawFaceLandmarks(canvas.value, resizedDetections);
    
    // Draw expressions if any detections are found
    if (resizedDetections.length > 0) {
      faceapi.draw.drawFaceExpressions(canvas.value, resizedDetections);
      // console.log("Expressions drawn");
    }
    
  }, 1000);
};

onMounted(async () => {
  await loadModels();
  await startVideoStream();
});
</script>

<style>
/* Add any styles you need here */
</style>