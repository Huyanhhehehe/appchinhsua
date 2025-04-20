<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="UTF-8">
  <title>Trình chỉnh sửa ảnh</title>
  <style>
    body {
      font-family: 'Segoe UI', sans-serif;
      text-align: center;
      background-color: #f9f9f9;
      margin: 20px;
    }


    h1 {
      color: #2c3e50;
      margin-bottom: 20px;
    }


    canvas {
      border: 2px solid #ccc;
      border-radius: 10px;
      margin-top: 20px;
      max-width: 100%;
    }


    .controls {
      margin-top: 30px;
      padding: 20px;
      background: white;
      border-radius: 10px;
      box-shadow: 0 2px 8px rgba(0,0,0,0.1);
      display: inline-block;
    }


    .category {
      display: inline-block;
      margin: 10px;
      padding: 12px 25px;
      background-color: #3498db;
      color: white;
      border-radius: 25px;
      cursor: pointer;
      transition: background 0.3s ease;
    }


    .category:hover {
      background-color: #2980b9;
    }


    .group {
      display: none;
      text-align: left;
      padding: 15px;
      background-color: #f0f0f0;
      border-radius: 8px;
      margin-top: 20px;
    }


    .group h3 {
      margin-top: 0;
      color: #2c3e50;
    }


    input[type=range] {
      width: 200px;
    }


    .switch {
      display: flex;
      align-items: center;
      margin: 10px 0;
    }


    .switch input {
      display: none;
    }


    .slider {
      position: relative;
      width: 40px;
      height: 20px;
      background-color: #ccc;
      border-radius: 20px;
      margin-left: 10px;
      cursor: pointer;
    }


    .slider:before {
      content: "";
      position: absolute;
      height: 16px;
      width: 16px;
      left: 2px;
      bottom: 2px;
      background-color: white;
      border-radius: 50%;
      transition: .4s;
    }


    input:checked + .slider {
      background-color: #3498db;
    }


    input:checked + .slider:before {
      transform: translateX(20px);
    }


    .btn-group {
      margin-top: 20px;
    }


    .btn {
      padding: 12px 24px;
      background-color: #2ecc71;
      color: white;
      border: none;
      border-radius: 6px;
      font-size: 16px;
      margin: 0 10px;
      cursor: pointer;
      transition: background 0.3s ease;
    }


    .btn:hover {
      background-color: #27ae60;
    }


    .btn.secondary {
      background-color: #e67e22;
    }


    .btn.secondary:hover {
      background-color: #d35400;
    }
  </style>
</head>
<body>


  <h1>Trình chỉnh sửa ảnh</h1>
  <input type="file" id="upload" accept="image/*">
  <br>
  <canvas id="canvas"></canvas>


  <div class="controls">
    <div class="category" onclick="toggleGroup('group1')">Tách kênh màu</div>
    <div class="category" onclick="toggleGroup('group2')">Đen trắng</div>
    <div class="category" onclick="toggleGroup('group3')">Ánh sáng</div>


    <div id="group1" class="group">
      <h3>Kênh màu</h3>
      <label class="switch">Kênh đỏ
        <input type="checkbox" id="redChannel" checked>
        <span class="slider"></span>
      </label>
      <label class="switch">Kênh xanh lá
        <input type="checkbox" id="greenChannel" checked>
        <span class="slider"></span>
      </label>
      <label class="switch">Kênh xanh dương
        <input type="checkbox" id="blueChannel" checked>
        <span class="slider"></span>
      </label>
    </div>


    <div id="group2" class="group">
      <h3>Chế độ</h3>
      <label class="switch">Đen trắng
        <input type="checkbox" id="grayscaleSwitch">
        <span class="slider"></span>
      </label>
    </div>


    <div id="group3" class="group">
      <h3>Ánh sáng & tương phản</h3>
      <label>Độ sáng: <input type="range" id="brightnessRange" min="-100" max="100" value="0"></label><br>
      <label>Độ tương phản: <input type="range" id="contrastRange" min="-100" max="100" value="0"></label>
    </div>


    <div class="btn-group">
      <button class="btn secondary" onclick="resetImage()">Đặt lại</button>
      <button class="btn" onclick="saveImage()">Lưu ảnh</button>
    </div>
  </div>


  <script>
    const canvas = document.getElementById('canvas');
    const ctx = canvas.getContext('2d');
    let originalImage = null;


    document.getElementById('upload').addEventListener('change', function(e) {
      const file = e.target.files[0];
      if (!file) return;


      const img = new Image();
      img.onload = function() {
        canvas.width = img.width;
        canvas.height = img.height;
        ctx.drawImage(img, 0, 0);
        originalImage = ctx.getImageData(0, 0, canvas.width, canvas.height);
        applyFilters();
      };
      img.src = URL.createObjectURL(file);
    });


    function toggleGroup(id) {
      const group = document.getElementById(id);
      group.style.display = group.style.display === 'block' ? 'none' : 'block';
    }


    const controls = ['redChannel', 'greenChannel', 'blueChannel', 'grayscaleSwitch', 'brightnessRange', 'contrastRange'];
    controls.forEach(id => {
      document.getElementById(id).addEventListener('input', applyFilters);
    });


    function clamp(v) {
      return Math.max(0, Math.min(255, v));
    }


    function applyFilters() {
      if (!originalImage) return;


      const redOn = document.getElementById('redChannel').checked;
      const greenOn = document.getElementById('greenChannel').checked;
      const blueOn = document.getElementById('blueChannel').checked;
      const grayscale = document.getElementById('grayscaleSwitch').checked;
      const brightness = parseInt(document.getElementById('brightnessRange').value);
      const contrast = parseInt(document.getElementById('contrastRange').value);
      const factor = (259 * (contrast + 255)) / (255 * (259 - contrast));


      let imageData = new ImageData(new Uint8ClampedArray(originalImage.data), originalImage.width, originalImage.height);
      let data = imageData.data;


      for (let i = 0; i < data.length; i += 4) {
        let r = data[i], g = data[i+1], b = data[i+2];


        if (grayscale) {
          const avg = (r + g + b) / 3;
          r = g = b = avg;
        }


        r += brightness;
        g += brightness;
        b += brightness;


        r = factor * (r - 128) + 128;
        g = factor * (g - 128) + 128;
        b = factor * (b - 128) + 128;


        data[i]   = redOn ? clamp(r) : 0;
        data[i+1] = greenOn ? clamp(g) : 0;
        data[i+2] = blueOn ? clamp(b) : 0;
      }


      ctx.putImageData(imageData, 0, 0);
    }


    function resetImage() {
      if (!originalImage) return;
      ctx.putImageData(originalImage, 0, 0);
      document.getElementById('redChannel').checked = true;
      document.getElementById('greenChannel').checked = true;
      document.getElementById('blueChannel').checked = true;
      document.getElementById('grayscaleSwitch').checked = false;
      document.getElementById('brightnessRange').value = 0;
      document.getElementById('contrastRange').value = 0;
    }


    function saveImage() {
      const link = document.createElement('a');
      link.download = 'edited-image.png';
      link.href = canvas.toDataURL('image/png');
      link.click();
    }
  </script>


</body>
</html>







