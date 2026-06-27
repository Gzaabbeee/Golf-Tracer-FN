<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Golf Shot Tracer</title>
<style>
  @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600&family=Bebas+Neue&display=swap');

  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

  body {
    background: #0a0a0f;
    color: #e8e8e8;
    font-family: 'Inter', sans-serif;
    min-height: 100vh;
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  header {
    width: 100%;
    padding: 14px 24px;
    display: flex;
    align-items: center;
    gap: 12px;
    border-bottom: 1px solid #1e1e2e;
    background: #0d0d14;
  }

  .logo {
    font-family: 'Bebas Neue', sans-serif;
    font-size: 1.6rem;
    letter-spacing: 0.08em;
    color: #c8f04a;
  }

  .logo span {
    color: #fff;
  }

  .status-pill {
    margin-left: auto;
    padding: 4px 12px;
    border-radius: 999px;
    font-size: 0.7rem;
    font-weight: 500;
    letter-spacing: 0.06em;
    text-transform: uppercase;
    background: #1a1a2a;
    color: #666;
    border: 1px solid #2a2a3a;
    transition: all 0.3s;
  }

  .status-pill.live {
    background: #0f2a0f;
    color: #4ade80;
    border-color: #166534;
  }

  .status-pill.tracing {
    background: #2a1a00;
    color: #c8f04a;
    border-color: #4a3a00;
  }

  .status-pill.recording {
    background: #2a0a0a;
    color: #f87171;
    border-color: #7f1d1d;
    animation: pulse 1.2s ease-in-out infinite;
  }

  @keyframes pulse {
    0%, 100% { opacity: 1; }
    50% { opacity: 0.6; }
  }

  main {
    width: 100%;
    max-width: 1100px;
    padding: 20px 16px;
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 16px;
  }

  .canvas-wrap {
    position: relative;
    width: 100%;
    border-radius: 8px;
    overflow: hidden;
    background: #000;
    border: 1px solid #1e1e2e;
    cursor: crosshair;
  }

  #videoEl {
    position: absolute;
    top: 0; left: 0;
    width: 100%; height: 100%;
    object-fit: cover;
    display: block;
  }

  #tracerCanvas {
    position: absolute;
    top: 0; left: 0;
    width: 100%; height: 100%;
    display: block;
  }

  .placeholder {
    width: 100%;
    aspect-ratio: 16/9;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    gap: 12px;
    color: #333;
  }

  .placeholder svg { opacity: 0.3; }

  .placeholder p {
    font-size: 0.85rem;
    color: #444;
  }

  /* Controls */
  .controls {
    width: 100%;
    display: grid;
    grid-template-columns: 1fr 1fr 1fr;
    gap: 10px;
  }

  .ctrl-group {
    background: #0d0d14;
    border: 1px solid #1e1e2e;
    border-radius: 8px;
    padding: 14px 16px;
    display: flex;
    flex-direction: column;
    gap: 10px;
  }

  .ctrl-group label {
    font-size: 0.65rem;
    text-transform: uppercase;
    letter-spacing: 0.1em;
    color: #555;
    font-weight: 500;
  }

  .btn-row {
    display: flex;
    gap: 8px;
    flex-wrap: wrap;
  }

  button {
    font-family: 'Inter', sans-serif;
    font-size: 0.78rem;
    font-weight: 500;
    padding: 8px 16px;
    border-radius: 6px;
    border: none;
    cursor: pointer;
    transition: all 0.15s;
    white-space: nowrap;
  }

  .btn-primary {
    background: #c8f04a;
    color: #0a0a0f;
  }

  .btn-primary:hover { background: #d8ff5a; }
  .btn-primary:disabled { background: #2a2a1a; color: #555; cursor: not-allowed; }

  .btn-danger {
    background: #2a0a0a;
    color: #f87171;
    border: 1px solid #7f1d1d;
  }

  .btn-danger:hover { background: #3a0a0a; }
  .btn-danger:disabled { opacity: 0.4; cursor: not-allowed; }

  .btn-ghost {
    background: #1a1a2a;
    color: #aaa;
    border: 1px solid #2a2a3a;
  }

  .btn-ghost:hover { background: #222230; color: #ddd; }
  .btn-ghost:disabled { opacity: 0.4; cursor: not-allowed; }

  .btn-save {
    background: #0f2a4a;
    color: #60a5fa;
    border: 1px solid #1e3a6a;
  }

  .btn-save:hover { background: #1a3a5a; }
  .btn-save:disabled { opacity: 0.4; cursor: not-allowed; }

  /* Color swatches */
  .swatch-row {
    display: flex;
    gap: 8px;
    align-items: center;
  }

  .swatch {
    width: 26px;
    height: 26px;
    border-radius: 50%;
    border: 2px solid transparent;
    cursor: pointer;
    transition: transform 0.15s, border-color 0.15s;
  }

  .swatch:hover { transform: scale(1.15); }
  .swatch.active { border-color: #fff; transform: scale(1.1); }

  /* Point counter */
  .point-counter {
    font-family: 'Bebas Neue', sans-serif;
    font-size: 2rem;
    color: #c8f04a;
    line-height: 1;
  }

  .point-label {
    font-size: 0.65rem;
    color: #555;
    text-transform: uppercase;
    letter-spacing: 0.08em;
  }

  /* Thickness slider */
  input[type=range] {
    -webkit-appearance: none;
    width: 100%;
    height: 4px;
    border-radius: 2px;
    background: #1e1e2e;
    outline: none;
  }

  input[type=range]::-webkit-slider-thumb {
    -webkit-appearance: none;
    width: 14px;
    height: 14px;
    border-radius: 50%;
    background: #c8f04a;
    cursor: pointer;
  }

  .hint {
    font-size: 0.72rem;
    color: #444;
    text-align: center;
    padding-bottom: 8px;
  }

  .hint b { color: #666; font-weight: 500; }

  @media (max-width: 680px) {
    .controls { grid-template-columns: 1fr 1fr; }
    .ctrl-group:last-child { grid-column: 1 / -1; }
  }
</style>
</head>
<body>

<header>
  <div class="logo">Shot<span>Trace</span></div>
  <div class="status-pill" id="statusPill">No Camera</div>
</header>

<main>
  <div class="canvas-wrap" id="canvasWrap">
    <div class="placeholder" id="placeholder">
      <svg width="48" height="48" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.2">
        <circle cx="12" cy="12" r="10"/><circle cx="12" cy="12" r="3"/>
        <line x1="12" y1="2" x2="12" y2="5"/><line x1="12" y1="19" x2="12" y2="22"/>
        <line x1="2" y1="12" x2="5" y2="12"/><line x1="19" y1="12" x2="22" y2="12"/>
      </svg>
      <p>Start camera to begin tracing</p>
    </div>
    <video id="videoEl" autoplay muted playsinline style="display:none;"></video>
    <canvas id="tracerCanvas" style="display:none;"></canvas>
  </div>

  <div class="controls">
    <!-- Camera -->
    <div class="ctrl-group">
      <label>Camera</label>
      <div class="btn-row">
        <button class="btn-primary" id="btnCamera">Start Camera</button>
        <button class="btn-ghost" id="btnFlip" disabled>Flip</button>
      </div>
    </div>

    <!-- Tracer -->
    <div class="ctrl-group">
      <label>Tracer Color</label>
      <div class="swatch-row">
        <div class="swatch active" style="background:#c8f04a;" data-color="#c8f04a" title="Lime"></div>
        <div class="swatch" style="background:#ffffff;" data-color="#ffffff" title="White"></div>
        <div class="swatch" style="background:#f97316;" data-color="#f97316" title="Orange"></div>
        <div class="swatch" style="background:#60a5fa;" data-color="#60a5fa" title="Blue"></div>
        <div class="swatch" style="background:#f87171;" data-color="#f87171" title="Red"></div>
      </div>
      <label style="margin-top:4px;">Line thickness</label>
      <input type="range" id="thickness" min="2" max="10" value="4">
    </div>

    <!-- Points & Actions -->
    <div class="ctrl-group">
      <label>Points Placed</label>
      <div class="point-counter" id="pointCount">0</div>
      <div class="btn-row">
        <button class="btn-ghost" id="btnUndo" disabled>Undo</button>
        <button class="btn-ghost" id="btnClear" disabled>Clear</button>
      </div>
    </div>

    <!-- Record & Save -->
    <div class="ctrl-group">
      <label>Recording</label>
      <div class="btn-row">
        <button class="btn-danger" id="btnRecord" disabled>⏺ Record</button>
        <button class="btn-save" id="btnSave" disabled>⬇ Save</button>
      </div>
    </div>

    <!-- Mode -->
    <div class="ctrl-group">
      <label>Trace Mode</label>
      <div class="btn-row">
        <button class="btn-ghost active-mode" id="btnModeClick" style="border-color:#c8f04a;color:#c8f04a;">Click</button>
      </div>
      <p style="font-size:0.68rem;color:#444;margin-top:4px;">Click on the ball each frame to plot its path</p>
    </div>

    <!-- Help -->
    <div class="ctrl-group">
      <label>How to use</label>
      <p style="font-size:0.7rem;color:#555;line-height:1.6;">
        1. Start camera<br>
        2. Hit record<br>
        3. Click the ball as it flies<br>
        4. Hit save when done
      </p>
    </div>
  </div>

  <p class="hint"><b>Tip:</b> Click directly on the ball at each moment of flight — the tracer smooths your clicks into a clean arc automatically.</p>
</main>

<script>
  const videoEl = document.getElementById('videoEl');
  const canvas = document.getElementById('tracerCanvas');
  const ctx = canvas.getContext('2d');
  const canvasWrap = document.getElementById('canvasWrap');
  const placeholder = document.getElementById('placeholder');
  const statusPill = document.getElementById('statusPill');
  const pointCountEl = document.getElementById('pointCount');
  const btnCamera = document.getElementById('btnCamera');
  const btnFlip = document.getElementById('btnFlip');
  const btnRecord = document.getElementById('btnRecord');
  const btnSave = document.getElementById('btnSave');
  const btnUndo = document.getElementById('btnUndo');
  const btnClear = document.getElementById('btnClear');
  const thicknessSlider = document.getElementById('thickness');

  let stream = null;
  let facingMode = 'environment';
  let points = [];
  let tracerColor = '#c8f04a';
  let lineThickness = 4;
  let mediaRecorder = null;
  let recordedChunks = [];
  let isRecording = false;
  let animFrame = null;
  let cameraActive = false;

  // Resize canvas to match video
  function resizeCanvas() {
    const rect = canvasWrap.getBoundingClientRect();
    canvas.width = rect.width;
    canvas.height = rect.height;
  }

  // Main render loop — draws video + tracer overlay
  function renderLoop() {
    if (!cameraActive) return;
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    ctx.drawImage(videoEl, 0, 0, canvas.width, canvas.height);
    drawTracer();
    animFrame = requestAnimationFrame(renderLoop);
  }

  // Catmull-Rom spline through points for smooth curve
  function catmullRom(pts) {
    if (pts.length < 2) return;
    ctx.beginPath();
    ctx.moveTo(pts[0].x, pts[0].y);

    if (pts.length === 2) {
      ctx.lineTo(pts[1].x, pts[1].y);
      return;
    }

    for (let i = 0; i < pts.length - 1; i++) {
      const p0 = pts[Math.max(i - 1, 0)];
      const p1 = pts[i];
      const p2 = pts[i + 1];
      const p3 = pts[Math.min(i + 2, pts.length - 1)];

      const cp1x = p1.x + (p2.x - p0.x) / 6;
      const cp1y = p1.y + (p2.y - p0.y) / 6;
      const cp2x = p2.x - (p3.x - p1.x) / 6;
      const cp2y = p2.y - (p3.y - p1.y) / 6;

      ctx.bezierCurveTo(cp1x, cp1y, cp2x, cp2y, p2.x, p2.y);
    }
  }

  function drawTracer() {
    if (points.length < 1) return;

    // Glow pass
    ctx.save();
    ctx.shadowBlur = 18;
    ctx.shadowColor = tracerColor;
    ctx.strokeStyle = tracerColor;
    ctx.lineWidth = lineThickness;
    ctx.lineCap = 'round';
    ctx.lineJoin = 'round';
    ctx.globalAlpha = 0.35;
    catmullRom(points);
    ctx.stroke();
    ctx.restore();

    // Solid pass
    ctx.save();
    ctx.strokeStyle = tracerColor;
    ctx.lineWidth = lineThickness;
    ctx.lineCap = 'round';
    ctx.lineJoin = 'round';
    ctx.globalAlpha = 0.92;
    catmullRom(points);
    ctx.stroke();
    ctx.restore();

    // Draw dots at each point
    points.forEach((p, i) => {
      ctx.save();
      ctx.beginPath();
      ctx.arc(p.x, p.y, i === points.length - 1 ? 5 : 3, 0, Math.PI * 2);
      ctx.fillStyle = tracerColor;
      ctx.globalAlpha = i === points.length - 1 ? 1 : 0.5;
      ctx.shadowBlur = 10;
      ctx.shadowColor = tracerColor;
      ctx.fill();
      ctx.restore();
    });
  }

  // Start camera
  btnCamera.addEventListener('click', async () => {
    if (cameraActive) {
      stopCamera();
      return;
    }
    try {
      stream = await navigator.mediaDevices.getUserMedia({
        video: { facingMode, width: { ideal: 1920 }, height: { ideal: 1080 } },
        audio: false
      });
      videoEl.srcObject = stream;
      videoEl.style.display = 'block';
      canvas.style.display = 'block';
      placeholder.style.display = 'none';

      // Set wrap height based on video aspect
      videoEl.onloadedmetadata = () => {
        const ratio = videoEl.videoHeight / videoEl.videoWidth;
        canvasWrap.style.aspectRatio = `${videoEl.videoWidth} / ${videoEl.videoHeight}`;
        resizeCanvas();
        cameraActive = true;
        renderLoop();
        statusPill.textContent = 'Live';
        statusPill.className = 'status-pill live';
        btnCamera.textContent = 'Stop Camera';
        btnCamera.className = 'btn-ghost';
        btnFlip.disabled = false;
        btnRecord.disabled = false;
      };
    } catch (e) {
      alert('Camera access denied or unavailable. Please allow camera permissions.');
    }
  });

  function stopCamera() {
    if (stream) stream.getTracks().forEach(t => t.stop());
    if (animFrame) cancelAnimationFrame(animFrame);
    cameraActive = false;
    videoEl.style.display = 'none';
    canvas.style.display = 'none';
    placeholder.style.display = 'flex';
    statusPill.textContent = 'No Camera';
    statusPill.className = 'status-pill';
    btnCamera.textContent = 'Start Camera';
    btnCamera.className = 'btn-primary';
    btnFlip.disabled = true;
    btnRecord.disabled = true;
    if (isRecording) stopRecording();
  }

  // Flip camera
  btnFlip.addEventListener('click', async () => {
    facingMode = facingMode === 'environment' ? 'user' : 'environment';
    stopCamera();
    setTimeout(() => btnCamera.click(), 200);
  });

  // Click to place points
  canvasWrap.addEventListener('click', (e) => {
    if (!cameraActive) return;
    const rect = canvas.getBoundingClientRect();
    const scaleX = canvas.width / rect.width;
    const scaleY = canvas.height / rect.height;
    const x = (e.clientX - rect.left) * scaleX;
    const y = (e.clientY - rect.top) * scaleY;
    points.push({ x, y });
    updatePointUI();
  });

  function updatePointUI() {
    pointCountEl.textContent = points.length;
    btnUndo.disabled = points.length === 0;
    btnClear.disabled = points.length === 0;
    if (points.length > 0) {
      statusPill.textContent = 'Tracing';
      statusPill.className = 'status-pill tracing';
    } else if (cameraActive) {
      statusPill.textContent = 'Live';
      statusPill.className = 'status-pill live';
    }
  }

  btnUndo.addEventListener('click', () => {
    points.pop();
    updatePointUI();
  });

  btnClear.addEventListener('click', () => {
    points = [];
    updatePointUI();
  });

  // Color swatches
  document.querySelectorAll('.swatch').forEach(s => {
    s.addEventListener('click', () => {
      document.querySelectorAll('.swatch').forEach(x => x.classList.remove('active'));
      s.classList.add('active');
      tracerColor = s.dataset.color;
    });
  });

  thicknessSlider.addEventListener('input', () => {
    lineThickness = parseInt(thicknessSlider.value);
  });

  // Recording via canvas stream
  btnRecord.addEventListener('click', () => {
    if (isRecording) {
      stopRecording();
    } else {
      startRecording();
    }
  });

  function startRecording() {
    recordedChunks = [];
    const canvasStream = canvas.captureStream(30);

    const mimeType = MediaRecorder.isTypeSupported('video/webm;codecs=vp9')
      ? 'video/webm;codecs=vp9'
      : MediaRecorder.isTypeSupported('video/webm')
      ? 'video/webm'
      : 'video/mp4';

    try {
      mediaRecorder = new MediaRecorder(canvasStream, { mimeType });
    } catch (e) {
      mediaRecorder = new MediaRecorder(canvasStream);
    }

    mediaRecorder.ondataavailable = (e) => {
      if (e.data.size > 0) recordedChunks.push(e.data);
    };

    mediaRecorder.onstop = () => {
      btnSave.disabled = false;
    };

    mediaRecorder.start(100);
    isRecording = true;
    btnRecord.textContent = '⏹ Stop';
    btnRecord.className = 'btn-ghost';
    statusPill.textContent = '● Recording';
    statusPill.className = 'status-pill recording';
  }

  function stopRecording() {
    if (mediaRecorder && mediaRecorder.state !== 'inactive') {
      mediaRecorder.stop();
    }
    isRecording = false;
    btnRecord.textContent = '⏺ Record';
    btnRecord.className = 'btn-danger';
    if (cameraActive) {
      statusPill.textContent = points.length > 0 ? 'Tracing' : 'Live';
      statusPill.className = points.length > 0 ? 'status-pill tracing' : 'status-pill live';
    }
  }

  btnSave.addEventListener('click', () => {
    if (recordedChunks.length === 0) return;
    const blob = new Blob(recordedChunks, { type: recordedChunks[0].type || 'video/webm' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    const ext = blob.type.includes('mp4') ? 'mp4' : 'webm';
    a.download = `shot-trace-${Date.now()}.${ext}`;
    a.click();
    URL.revokeObjectURL(url);
  });

  window.addEventListener('resize', resizeCanvas);
</script>
</body>
</html>