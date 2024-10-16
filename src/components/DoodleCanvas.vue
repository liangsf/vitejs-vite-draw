<script setup>
import { ref, onMounted, watch } from 'vue';

const canvasRef = ref(null);
const ctx = ref(null);
const isDrawing = ref(false);
const color = ref('#000000');
const lineWidth = ref(5);
const tool = ref('pencil');
const history = ref([]);
const historyIndex = ref(-1);
const scale = ref(1);
const backgroundImage = ref(null);

const startPos = ref({ x: 0, y: 0 });

onMounted(() => {
  const canvas = canvasRef.value;
  ctx.value = canvas.getContext('2d');
  canvas.width = 800;
  canvas.height = 600;
  ctx.value.lineCap = 'round';
  saveState();
});

watch([color, lineWidth, tool], () => {
  ctx.value.strokeStyle = color.value;
  ctx.value.lineWidth = lineWidth.value;
});

watch(scale, () => {
  redrawCanvas();
});

function startDrawing(event) {
  isDrawing.value = true;
  const { x, y } = getMousePos(event);
  startPos.value = { x, y };
  if (tool.value === 'spray') {
    spray(x, y);
  } else {
    ctx.value.beginPath();
    ctx.value.moveTo(x, y);
  }
}

function draw(event) {
  if (!isDrawing.value) return;

  const { x, y } = getMousePos(event);

  switch (tool.value) {
    case 'pencil':
    case 'marker':
      ctx.value.lineTo(x, y);
      ctx.value.stroke();
      break;
    case 'spray':
      spray(x, y);
      break;
    case 'rectangle':
    case 'circle':
    case 'line':
      drawShape(x, y);
      break;
  }
}

function stopDrawing() {
  if (isDrawing.value) {
    isDrawing.value = false;
    if (['rectangle', 'circle', 'line'].includes(tool.value)) {
      const { x, y } = getMousePos(event);
      drawShape(x, y, true);
    }
    saveState();
  }
}

function getMousePos(event) {
  const rect = canvasRef.value.getBoundingClientRect();
  return {
    x: (event.clientX - rect.left) / scale.value,
    y: (event.clientY - rect.top) / scale.value
  };
}

function spray(x, y) {
  ctx.value.beginPath();
  for (let i = 0; i < 20; i++) {
    const offsetX = (Math.random() - 0.5) * lineWidth.value * 2;
    const offsetY = (Math.random() - 0.5) * lineWidth.value * 2;
    ctx.value.fillRect(x + offsetX, y + offsetY, 1, 1);
  }
  ctx.value.fill();
}

function drawShape(x, y, commit = false) {
  const tempCanvas = document.createElement('canvas');
  tempCanvas.width = canvasRef.value.width;
  tempCanvas.height = canvasRef.value.height;
  const tempCtx = tempCanvas.getContext('2d');
  tempCtx.drawImage(canvasRef.value, 0, 0);

  tempCtx.beginPath();
  tempCtx.strokeStyle = color.value;
  tempCtx.lineWidth = lineWidth.value;

  switch (tool.value) {
    case 'rectangle':
      tempCtx.rect(startPos.value.x, startPos.value.y, x - startPos.value.x, y - startPos.value.y);
      break;
    case 'circle':
      const radius = Math.sqrt(Math.pow(x - startPos.value.x, 2) + Math.pow(y - startPos.value.y, 2));
      tempCtx.arc(startPos.value.x, startPos.value.y, radius, 0, 2 * Math.PI);
      break;
    case 'line':
      tempCtx.moveTo(startPos.value.x, startPos.value.y);
      tempCtx.lineTo(x, y);
      break;
  }

  tempCtx.stroke();

  if (commit) {
    ctx.value.drawImage(tempCanvas, 0, 0);
  } else {
    ctx.value.clearRect(0, 0, canvasRef.value.width, canvasRef.value.height);
    ctx.value.drawImage(tempCanvas, 0, 0);
  }
}

function clearCanvas() {
  ctx.value.clearRect(0, 0, canvasRef.value.width, canvasRef.value.height);
  if (backgroundImage.value) {
    drawBackgroundImage();
  }
  saveState();
}

function saveImage() {
  const link = document.createElement('a');
  link.download = 'doodle.png';
  link.href = canvasRef.value.toDataURL();
  link.click();
}

function saveState() {
  historyIndex.value++;
  history.value = history.value.slice(0, historyIndex.value);
  history.value.push(canvasRef.value.toDataURL());
}

function undo() {
  if (historyIndex.value > 0) {
    historyIndex.value--;
    loadState();
  }
}

function redo() {
  if (historyIndex.value < history.value.length - 1) {
    historyIndex.value++;
    loadState();
  }
}

function loadState() {
  const img = new Image();
  img.src = history.value[historyIndex.value];
  img.onload = () => {
    ctx.value.clearRect(0, 0, canvasRef.value.width, canvasRef.value.height);
    ctx.value.drawImage(img, 0, 0);
  };
}

function uploadBackground(event) {
  const file = event.target.files[0];
  const reader = new FileReader();

  reader.onload = (e) => {
    backgroundImage.value = new Image();
    backgroundImage.value.onload = () => {
      drawBackgroundImage();
      saveState();
    };
    backgroundImage.value.src = e.target.result;
  };

  reader.readAsDataURL(file);
}

function drawBackgroundImage() {
  if (backgroundImage.value) {
    ctx.value.clearRect(0, 0, canvasRef.value.width, canvasRef.value.height);
    ctx.value.drawImage(backgroundImage.value, 0, 0, canvasRef.value.width, canvasRef.value.height);
  }
}

function addText() {
  const text = prompt('Enter text:');
  if (text) {
    const x = canvasRef.value.width / 2;
    const y = canvasRef.value.height / 2;
    ctx.value.font = `${lineWidth.value * 5}px Arial`;
    ctx.value.fillStyle = color.value;
    ctx.value.textAlign = 'center';
    ctx.value.fillText(text, x, y);
    saveState();
  }
}

function zoomIn() {
  scale.value = Math.min(scale.value + 0.1, 3);
}

function zoomOut() {
  scale.value = Math.max(scale.value - 0.1, 0.5);
}

function redrawCanvas() {
  const tempCanvas = document.createElement('canvas');
  tempCanvas.width = canvasRef.value.width;
  tempCanvas.height = canvasRef.value.height;
  const tempCtx = tempCanvas.getContext('2d');
  tempCtx.drawImage(canvasRef.value, 0, 0);

  canvasRef.value.style.width = `${canvasRef.value.width * scale.value}px`;
  canvasRef.value.style.height = `${canvasRef.value.height * scale.value}px`;

  ctx.value.clearRect(0, 0, canvasRef.value.width, canvasRef.value.height);
  if (backgroundImage.value) {
    drawBackgroundImage();
  }
  ctx.value.drawImage(tempCanvas, 0, 0);
}
</script>

<template>
  <div class="doodle-canvas">
    <div class="controls">
      <select v-model="tool">
        <option value="pencil">铅笔</option>
        <option value="marker">马克笔</option>
        <option value="spray">喷漆</option>
        <option value="rectangle">矩形</option>
        <option value="circle">圆形</option>
        <option value="line">直线</option>
      </select>
      <input type="color" v-model="color" title="选择颜色">
      <input type="range" v-model="lineWidth" min="1" max="20" title="线条宽度">
      <button @click="undo" :disabled="historyIndex <= 0">撤销</button>
      <button @click="redo" :disabled="historyIndex >= history.length - 1">重做</button>
      <button @click="clearCanvas">清除画布</button>
      <button @click="saveImage">保存图片</button>
      <input type="file" @change="uploadBackground" accept="image/*" id="backgroundUpload">
      <label for="backgroundUpload">上传背景图片</label>
      <button @click="addText">添加文本</button>
      <button @click="zoomIn">放大</button>
      <button @click="zoomOut">缩小</button>
    </div>
    <div class="canvas-container">
      <canvas
        ref="canvasRef"
        @mousedown="startDrawing"
        @mousemove="draw"
        @mouseup="stopDrawing"
        @mouseout="stopDrawing"
      ></canvas>
    </div>
  </div>
</template>

<style scoped>
.doodle-canvas {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.controls {
  margin-bottom: 20px;
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
}

.canvas-container {
  overflow: auto;
  border: 1px solid #ccc;
  max-width: 100%;
  max-height: 80vh;
}

canvas {
  cursor: crosshair;
}

button, select, label {
  padding: 5px 10px;
  background-color: #4CAF50;
  color: white;
  border: none;
  cursor: pointer;
  border-radius: 4px;
}

button:hover, select:hover, label:hover {
  background-color: #45a049;
}

button:disabled {
  background-color: #cccccc;
  cursor: not-allowed;
}

input[type="color"] {
  width: 50px;
  height: 30px;
  padding: 0;
  border: none;
  cursor: pointer;
}

input[type="range"] {
  width: 100px;
}

input[type="file"] {
  display: none;
}

input[type="file"] + label {
  padding: 5px 10px;
  background-color: #4CAF50;
  color: white;
  border: none;
  cursor: pointer;
  border-radius: 4px;
}

input[type="file"] + label:hover {
  background-color: #45a049;
}
</style>