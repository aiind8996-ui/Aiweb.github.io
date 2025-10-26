# Aiweb.github.io
<!doctype html>
<html lang="hi">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>JACK — Login</title>
  <style>
    :root {
      --blue: #000080;
      --bg: #f2f2f2;
    }
    * { box-sizing: border-box; }
    body {
      margin: 0;
      font-family: Inter, "Segoe UI", Arial, sans-serif;
      background: var(--bg);
    }
    .header {
      background: var(--blue);
      color: #fff;
      padding: 14px 16px;
      text-align: center;
      font-weight: 800;
      letter-spacing: 2px;
      font-size: 20px;
      position: sticky;
      top: 0;
      z-index: 20;
    }
    .container {
      max-width: 420px;
      margin: 40px auto;
      padding: 12px;
    }
    .card {
      background: #fff;
      border-radius: 12px;
      padding: 22px;
      box-shadow: 0 8px 30px rgba(2,6,23,0.08);
      text-align: center;
      transition: all 0.3s ease;
    }
    .title { font-size: 28px; margin: 6px 0 10px; }
    .subtitle { color: #555; margin-bottom: 12px; }
    input[type="text"], input[type="password"], input[type="number"] {
      width: 100%;
      padding: 12px;
      border-radius: 8px;
      border: 1px solid #d0d0d0;
      margin: 8px 0;
      font-size: 15px;
    }
    .btn {
      display: inline-block;
      width: 100%;
      padding: 12px;
      border-radius: 8px;
      background: var(--blue);
      color: #fff;
      border: 0;
      font-weight: 700;
      cursor: pointer;
      margin-top: 12px;
      font-size: 16px;
      transition: background 0.3s ease, transform 0.2s ease;
    }
    .btn:hover { background: #0010cc; transform: translateY(-2px); }
    .hidden { display: none; }
    .welcome-name { font-size: 28px; margin: 12px 0; }
    .name-style {
      font-size: 32px; font-weight: 900;
      background: linear-gradient(90deg, #ff0000, #ff9900, #33cc33, #3399ff, #9933ff);
      -webkit-background-clip: text; -webkit-text-fill-color: transparent;
      text-shadow: 2px 2px 4px rgba(0,0,0,0.3); letter-spacing: 2px;
    }
    .qr { margin-top: 18px; }
    .qr img {
      width: 200px; height: 200px; border-radius: 10px;
      border: 6px solid var(--blue); background: #fff; padding: 6px;
    }
    .logout, .scanner-btn {
      display: inline-block; margin-top: 18px; padding: 10px 18px;
      background: #444; color: #fff; border-radius: 8px; text-decoration: none;
      transition: background 0.3s ease, transform 0.2s ease;
    }
    .logout:hover, .scanner-btn:hover { background: #000; transform: scale(1.05); }

    /* Stylish Scanner Design (Design-only: camera + frame overlay) */
    #scannerBox {
      position: relative;
      width: 100%;
      max-width: 360px;
      margin: 20px auto;
      border-radius: 16px;
      overflow: hidden;
      background: #000;
      box-shadow: 0 8px 25px rgba(0,0,0,0.3);
      opacity: 0; transform: scale(0.95); transition: all 0.5s ease;
    }
    #scannerBox.active { opacity: 1; transform: scale(1); }
    #cameraView { width: 100%; display: block; border-radius: 16px; filter: brightness(0.9); }
    #frameOverlay {
      position: absolute; top: 0; left: 0; width: 100%; height: 100%;
      object-fit: cover; pointer-events: none; animation: glow 2s infinite alternate;
    }
    @keyframes glow {
      0% { filter: drop-shadow(0 0 5px rgba(255,255,255,0.4)); }
      100% { filter: drop-shadow(0 0 15px rgba(0,150,255,0.8)); }
    }

    /* small responsive tweak */
    @media (max-width: 420px){
      .container{ margin: 20px 12px; }
    }
  </style>
</head>
<body>

  <div class="header">🌟 <span class="name-style">WELCOME JACK</span> 🌟</div>

  <div class="container">
    <!-- LOGIN -->
    <div id="loginCard" class="card">
      <div class="title"><span class="name-style">JACK</span></div>
      <div class="subtitle">Enter your mobile number and PIN</div>
      <input id="mobileField" type="text" placeholder="Enter your mobile number" />
      <input id="pinField" type="password" placeholder="Enter your PIN" />
      <button class="btn" onclick="sendCode()">Send Code</button>

      <div id="codeBox" class="hidden">
        <input id="codeField" type="number" placeholder="Enter confirmation code" />
        <button class="btn" onclick="handleLogin()">Confirm & Login</button>
      </div>
    </div>

    <!-- WELCOME -->
    <div id="welcomeCard" class="card hidden">
      <div class="welcome-name" id="welcomeName">
        <span class="name-style">WELCOME JACK</span> 🔞
      </div>
      <div class="subtitle">You have successfully logged in.</div>

      <div class="qr">
        <h3>Your QR Code (scan to open YouTube)</h3>
        <!-- QR image will contain the YouTube channel link -->
        <img id="qrImage" src="" alt="QR Code" />
      </div>

      <!-- Scanner Button -->
      <a href="#" class="scanner-btn" onclick="toggleScanner();return false;">Open Scanner</a>

      <!-- New Stylish Scanner (design-only camera view + frame overlay) -->
      <div id="scannerBox" class="hidden" aria-hidden="true">
        <video id="cameraView" autoplay playsinline></video>
        <img id="frameOverlay" src="frame.png" alt="Frame Overlay" />
      </div>

      <!-- YouTube Link (fallback clickable) -->
      <div style="margin-top:15px;">
        <a id="ytFallback" href="https://youtube.com/@aiai-g5b?si=kBMRFoeYFpFhhT8f"
           target="_blank"
           style="display:inline-block;padding:10px 18px;background:#cc0000;color:#fff;
                  border-radius:8px;text-decoration:none;font-weight:600;">
           ▶ Visit My YouTube
        </a>
      </div>

      <!-- Logout -->
      <a href="#" class="logout" onclick="doLogout();return false;">Log Out</a>
    </div>
  </div>

<script>
  // --- CONFIG: set your YouTube link here (this is the link embedded in QR) ---
  const YT_LINK = "https://youtube.com/@aiai-g5b?si=kBMRFoeYFpFhhT8f";

  let generatedCode = null;
  let cameraStream = null;

  // Send confirmation code (simulated)
  function sendCode() {
    const mobile = document.getElementById("mobileField").value.trim();
    const pin = document.getElementById("pinField").value.trim();
    if (mobile === "" || pin === "") {
      alert("❌ Please enter mobile number and PIN first.");
      return;
    }
    generatedCode = Math.floor(100000 + Math.random() * 900000);
    alert("✅ Your confirmation code is: " + generatedCode);
    document.getElementById("codeBox").classList.remove("hidden");
  }

  // Handle login — on success create QR that points to YT_LINK
  function handleLogin() {
    const code = document.getElementById("codeField").value.trim();
    if (code === "" || code != generatedCode) {
      alert("❌ Invalid confirmation code.");
      return;
    }

    // show welcome
    document.getElementById("loginCard").classList.add("hidden");
    document.getElementById("welcomeCard").classList.remove("hidden");

    // Build QR image that encodes YouTube link (so mobile scanning opens YouTube)
    const qrUrl = "https://api.qrserver.com/v1/create-qr-code/?size=300x300&data=" + encodeURIComponent(YT_LINK);
    document.getElementById("qrImage").src = qrUrl;

    // also update fallback link href (redundant but safe)
    document.getElementById("ytFallback").href = YT_LINK;

    // clear sensitive inputs
    document.getElementById("mobileField").value = "";
    document.getElementById("pinField").value = "";
    document.getElementById("codeField").value = "";
    document.getElementById("codeBox").classList.add("hidden");
    generatedCode = null;
  }

  // Toggle camera view (design-only)
  function toggleScanner() {
    const box = document.getElementById("scannerBox");
    if (box.classList.contains("hidden")) {
      box.classList.remove("hidden");
      // trigger active animation slightly after showing
      setTimeout(() => box.classList.add("active"), 10);
      startCamera();
      // for accessibility: mark visible
      box.setAttribute("aria-hidden", "false");
    } else {
      box.classList.remove("active");
      setTimeout(() => box.classList.add("hidden"), 400);
      stopCamera();
      box.setAttribute("aria-hidden", "true");
    }
  }

  // Start camera (design-only preview). No QR decoding performed.
  async function startCamera() {
    try {
      cameraStream = await navigator.mediaDevices.getUserMedia({ video: { facingMode: "environment" } });
      const video = document.getElementById("cameraView");
      video.srcObject = cameraStream;
      // play may return a promise
      try { await video.play(); } catch(e) { /* ignore */ }
    } catch (err) {
      alert("Camera access denied or unavailable.");
    }
  }

  function stopCamera() {
    if (cameraStream) {
      const tracks = cameraStream.getTracks();
      tracks.forEach(track => track.stop());
      cameraStream = null;
      // clear video srcObject
      const video = document.getElementById("cameraView");
      try { video.srcObject = null; } catch(e){}
    }
  }

  function doLogout() {
    // hide welcome, stop camera if any
    document.getElementById("welcomeCard").classList.add("hidden");
    document.getElementById("loginCard").classList.remove("hidden");
    stopCamera();
  }

  // Prevent accidental camera use on page load
  // (camera starts only after user clicks "Open Scanner")
</script>

</body>
</html>
