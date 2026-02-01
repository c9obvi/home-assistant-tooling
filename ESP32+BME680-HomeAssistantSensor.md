<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>ESP32 + BME680 → Home Assistant</title>
<style>
  @import url('https://fonts.googleapis.com/css2?family=IBM+Plex+Mono:wght@400;500;600&family=IBM+Plex+Sans:wght@300;400;500;600&display=swap');

  :root {
    --bg: #0e0f11;
    --surface: #161820;
    --surface-alt: #1c1f2a;
    --border: #2a2d3a;
    --text: #c8cad4;
    --text-dim: #6b6f7e;
    --accent: #e8a838;
    --accent-dim: rgba(232,168,56,0.15);
    --green: #5ecc6e;
    --green-dim: rgba(94,204,110,0.12);
    --red: #e85c5c;
    --blue: #5ea3e8;
    --blue-dim: rgba(94,163,232,0.12);
    --code-bg: #111318;
  }

  * { margin:0; padding:0; box-sizing:border-box; }

  body {
    font-family: 'IBM Plex Sans', sans-serif;
    background: var(--bg);
    color: var(--text);
    min-height: 100vh;
    line-height: 1.6;
    overflow-x: hidden;
  }

  /* ── HEADER ─────── */
  .header {
    position: relative;
    padding: 56px 32px 40px;
    border-bottom: 1px solid var(--border);
    background: linear-gradient(135deg, #0e0f11 0%, #131520 50%, #0e0f11 100%);
  }
  .header::before {
    content:'';
    position:absolute;
    top:-60px; right:-80px;
    width:340px; height:340px;
    border-radius:50%;
    background: radial-gradient(circle, rgba(232,168,56,0.08) 0%, transparent 70%);
    pointer-events:none;
  }
  .header-tag {
    display: inline-block;
    font-family: 'IBM Plex Mono', monospace;
    font-size: 10px;
    letter-spacing: 2.5px;
    text-transform: uppercase;
    color: var(--accent);
    margin-bottom: 14px;
    opacity: 0.8;
  }
  .header h1 {
    font-family: 'IBM Plex Sans', sans-serif;
    font-weight: 300;
    font-size: clamp(28px, 5vw, 42px);
    color: #fff;
    letter-spacing: -0.5px;
    line-height: 1.2;
  }
  .header h1 span { color: var(--accent); font-weight: 500; }
  .header-sub {
    margin-top: 12px;
    font-size: 13px;
    color: var(--text-dim);
    max-width: 560px;
  }

  /* ── LAYOUT ─────── */
  .container { max-width: 880px; margin: 0 auto; padding: 40px 24px 80px; }

  /* ── SECTION TITLES ── */
  .section-label {
    display: flex;
    align-items: center;
    gap: 12px;
    margin-bottom: 20px;
    margin-top: 48px;
  }
  .section-label:first-child { margin-top: 0; }
  .section-num {
    font-family: 'IBM Plex Mono', monospace;
    font-size: 11px;
    color: var(--accent);
    background: var(--accent-dim);
    padding: 3px 8px;
    border-radius: 4px;
    letter-spacing: 1px;
  }
  .section-label h2 {
    font-size: 18px;
    font-weight: 500;
    color: #fff;
    letter-spacing: -0.2px;
  }

  /* ── CARDS ─────── */
  .card {
    background: var(--surface);
    border: 1px solid var(--border);
    border-radius: 10px;
    overflow: hidden;
  }
  .card + .card { margin-top: 16px; }

  /* ── CONFIRM SENSOR ── */
  .confirm-box {
    display: flex;
    gap: 24px;
    align-items: flex-start;
    padding: 24px;
    background: var(--surface);
    border: 1px solid var(--border);
    border-radius: 10px;
  }
  .confirm-icon {
    flex-shrink: 0;
    width: 44px; height: 44px;
    border-radius: 10px;
    background: var(--green-dim);
    border: 1px solid rgba(94,204,110,0.25);
    display: flex; align-items:center; justify-content:center;
    font-size: 20px;
  }
  .confirm-box h3 { font-size: 15px; color: #fff; margin-bottom: 4px; }
  .confirm-box p { font-size: 13px; color: var(--text-dim); line-height:1.5; }
  .confirm-box .chip {
    display:inline-block;
    font-family:'IBM Plex Mono',monospace;
    font-size:11px;
    background: var(--green-dim);
    color: var(--green);
    padding: 2px 7px;
    border-radius: 4px;
    margin-top: 6px;
    margin-right: 4px;
  }

  /* ── WIRING DIAGRAM ── */
  .wiring-wrap {
    padding: 28px 24px 24px;
  }
  .wiring-label {
    font-family:'IBM Plex Mono',monospace;
    font-size:10px;
    letter-spacing:1.5px;
    text-transform:uppercase;
    color: var(--text-dim);
    margin-bottom:16px;
  }
  .wire-diagram {
    display: flex;
    justify-content: center;
    gap: 48px;
    flex-wrap: wrap;
  }
  .board {
    background: var(--code-bg);
    border: 1px solid var(--border);
    border-radius: 8px;
    padding: 18px 20px;
    min-width: 200px;
  }
  .board-title {
    font-family:'IBM Plex Mono',monospace;
    font-size:11px;
    color: var(--accent);
    margin-bottom:12px;
    letter-spacing:0.5px;
  }
  .pin-row {
    display:flex;
    align-items:center;
    gap:10px;
    padding: 5px 0;
    border-bottom: 1px solid rgba(42,45,58,0.4);
  }
  .pin-row:last-child { border-bottom:none; }
  .pin-dot {
    width:10px; height:10px;
    border-radius:50%;
    flex-shrink:0;
  }
  .pin-name {
    font-family:'IBM Plex Mono',monospace;
    font-size:12px;
    color:#fff;
    width:50px;
  }
  .pin-desc {
    font-size:11px;
    color:var(--text-dim);
  }
  .dot-red { background:#e85c5c; box-shadow:0 0 6px rgba(232,92,92,0.4); }
  .dot-orange { background:#e8a838; box-shadow:0 0 6px rgba(232,168,56,0.4); }
  .dot-yellow { background:#e8d85c; box-shadow:0 0 6px rgba(232,216,92,0.4); }
  .dot-blue { background:#5ea3e8; box-shadow:0 0 6px rgba(94,163,232,0.4); }

  .wire-connector {
    display:flex;
    flex-direction:column;
    justify-content:center;
    align-items:center;
    gap: 18px;
    padding-top:30px;
  }
  .wire-line {
    width:80px;
    height:2px;
    border-radius:1px;
    position:relative;
  }
  .wire-line::before {
    content:'';
    position:absolute;
    left:0; top:50%;
    width:100%; height:2px;
    transform:translateY(-50%);
  }
  .wl-red { background:#e85c5c; }
  .wl-orange { background:#e8a838; }
  .wl-yellow { background:#e8d85c; }
  .wl-blue { background:#5ea3e8; }

  /* ── JUMPER NOTE ── */
  .jumper-note {
    margin-top:20px;
    padding:14px 18px;
    background: var(--accent-dim);
    border: 1px solid rgba(232,168,56,0.2);
    border-radius:8px;
    display:flex;
    gap:12px;
    align-items:flex-start;
  }
  .jumper-note .jn-icon { font-size:18px; flex-shrink:0; margin-top:1px; }
  .jumper-note p { font-size:12.5px; color:var(--text); line-height:1.55; }
  .jumper-note strong { color:var(--accent); }

  /* ── CODE BLOCK ── */
  .code-header {
    display:flex;
    align-items:center;
    justify-content:space-between;
    padding: 10px 18px;
    background: var(--surface-alt);
    border-bottom: 1px solid var(--border);
  }
  .code-header .dots {
    display:flex; gap:6px;
  }
  .code-header .dots span {
    width:10px; height:10px; border-radius:50%;
  }
  .code-header .lang {
    font-family:'IBM Plex Mono',monospace;
    font-size:10px;
    color:var(--text-dim);
    letter-spacing:1px;
    text-transform:uppercase;
  }
  pre {
    background: var(--code-bg);
    padding: 20px 22px;
    overflow-x: auto;
    font-family:'IBM Plex Mono',monospace;
    font-size:12.5px;
    line-height:1.7;
    color:#c8cad4;
    white-space:pre;
  }
  .c-comment { color:#4a4f63; font-style:italic; }
  .c-key { color:var(--accent); }
  .c-val { color:var(--green); }
  .c-str { color:var(--blue); }
  .c-plat { color:#c77dff; }

  /* ── STEPS LIST ── */
  .steps { padding: 22px 24px; }
  .step {
    display:flex;
    gap:16px;
    padding: 12px 0;
    border-bottom: 1px solid rgba(42,45,58,0.5);
  }
  .step:last-child { border-bottom:none; }
  .step-num {
    flex-shrink:0;
    width:26px; height:26px;
    border-radius:6px;
    background:var(--accent-dim);
    border:1px solid rgba(232,168,56,0.3);
    color:var(--accent);
    font-family:'IBM Plex Mono',monospace;
    font-size:11px;
    display:flex; align-items:center; justify-content:center;
  }
  .step h4 { font-size:14px; color:#fff; margin-bottom:3px; }
  .step p { font-size:12.5px; color:var(--text-dim); }
  .step code {
    font-family:'IBM Plex Mono',monospace;
    font-size:11px;
    background:var(--surface-alt);
    color:var(--blue);
    padding:2px 6px;
    border-radius:3px;
  }

  /* ── 3D PRINT SPEC ── */
  .print-grid {
    display:grid;
    grid-template-columns:1fr 1fr;
    gap:12px;
    padding: 20px 24px;
  }
  .print-spec {
    background:var(--surface-alt);
    border:1px solid var(--border);
    border-radius:8px;
    padding:14px 16px;
  }
  .print-spec .ps-label {
    font-family:'IBM Plex Mono',monospace;
    font-size:9px;
    letter-spacing:1.5px;
    text-transform:uppercase;
    color:var(--text-dim);
    margin-bottom:4px;
  }
  .print-spec .ps-val {
    font-size:14px;
    color:#fff;
    font-weight:500;
  }
  .print-spec .ps-note {
    font-size:11px;
    color:var(--text-dim);
    margin-top:2px;
  }

  /* ── ENCLOSURE SVG ── */
  .enclosure-wrap {
    padding:24px;
    display:flex;
    justify-content:center;
  }
  .enclosure-svg {
    width:100%;
    max-width:620px;
  }

  /* ── CHECKLIST ── */
  .checklist { padding:20px 24px; }
  .check-item {
    display:flex;
    align-items:center;
    gap:10px;
    padding:8px 0;
    border-bottom:1px solid rgba(42,45,58,0.4);
  }
  .check-item:last-child{border-bottom:none;}
  .check-box {
    width:18px;height:18px;
    border-radius:4px;
    border:1.5px solid var(--border);
    flex-shrink:0;
    display:flex;align-items:center;justify-content:center;
    background:var(--surface-alt);
  }
  .check-item span { font-size:13px; color:var(--text); }

  /* ── ADDR NOTE ── */
  .addr-note {
    margin-top:14px;
    padding:12px 16px;
    background:var(--blue-dim);
    border:1px solid rgba(94,163,232,0.2);
    border-radius:8px;
    font-size:12px;
    color:var(--text);
    line-height:1.6;
  }
  .addr-note strong { color:var(--blue); }

  @media(max-width:600px){
    .wire-diagram{flex-direction:column;align-items:center;}
    .wire-connector{flex-direction:row;gap:8px;padding:0;padding-top:0;margin:12px 0;}
    .print-grid{grid-template-columns:1fr;}
  }
</style>
</head>
<body>

<!-- HEADER -->
<div class="header">
  <div class="header-tag">Setup Guide · ESPHome · Home Assistant</div>
  <h1>ESP32 + <span>BME680</span><br>→ Home Assistant Sensor</h1>
  <p class="header-sub">Full wiring, firmware config, and 3D-printable enclosure specs. Zero soldering — jumper wires only.</p>
</div>

<div class="container">

  <!-- ── 0. CONFIRM SENSOR ── -->
  <div class="section-label">
    <span class="section-num">00</span>
    <h2>Confirm Your Sensor</h2>
  </div>
  <div class="confirm-box">
    <div class="confirm-icon">✓</div>
    <div>
      <h3>You have a BME680</h3>
      <p>The BME280 only does temp + humidity + pressure. The <strong style="color:#fff">BME680</strong> is the one that adds volatile organic compounds (VOC) / gas resistance — that's your board. It outputs a single gas resistance value (in ohms) which ESPHome can turn into an IAQ score.</p>
      <span class="chip">Temperature</span>
      <span class="chip">Humidity</span>
      <span class="chip">Pressure</span>
      <span class="chip" style="background:var(--accent-dim);color:var(--accent);">VOC / Gas</span>
    </div>
  </div>

  <!-- ── 1. WIRING ── -->
  <div class="section-label">
    <span class="section-num">01</span>
    <h2>Wiring — Jumper Cables Only</h2>
  </div>
  <div class="card">
    <div class="wiring-wrap">
      <div class="wiring-label">I²C Connection · 4 Wires Total</div>
      <div class="wire-diagram">
        <!-- ESP32 side -->
        <div class="board">
          <div class="board-title">ESP32 DevKit V1</div>
          <div class="pin-row">
            <div class="pin-dot dot-red"></div>
            <div class="pin-name">3V3</div>
            <div class="pin-desc">Power out</div>
          </div>
          <div class="pin-row">
            <div class="pin-dot dot-orange"></div>
            <div class="pin-name">GPIO21</div>
            <div class="pin-desc">SDA (data)</div>
          </div>
          <div class="pin-row">
            <div class="pin-dot dot-yellow"></div>
            <div class="pin-name">GPIO22</div>
            <div class="pin-desc">SCL (clock)</div>
          </div>
          <div class="pin-row">
            <div class="pin-dot dot-blue"></div>
            <div class="pin-name">GND</div>
            <div class="pin-desc">Ground</div>
          </div>
        </div>
        <!-- wires -->
        <div class="wire-connector">
          <div class="wire-line wl-red"></div>
          <div class="wire-line wl-orange"></div>
          <div class="wire-line wl-yellow"></div>
          <div class="wire-line wl-blue"></div>
        </div>
        <!-- BME680 side -->
        <div class="board">
          <div class="board-title">BME680 Breakout</div>
          <div class="pin-row">
            <div class="pin-dot dot-red"></div>
            <div class="pin-name">VCC</div>
            <div class="pin-desc">Power in</div>
          </div>
          <div class="pin-row">
            <div class="pin-dot dot-orange"></div>
            <div class="pin-name">SDA</div>
            <div class="pin-desc">Data</div>
          </div>
          <div class="pin-row">
            <div class="pin-dot dot-yellow"></div>
            <div class="pin-name">SCL</div>
            <div class="pin-desc">Clock</div>
          </div>
          <div class="pin-row">
            <div class="pin-dot dot-blue"></div>
            <div class="pin-name">GND</div>
            <div class="pin-desc">Ground</div>
          </div>
        </div>
      </div>

      <div class="jumper-note">
        <div class="jn-icon">🔌</div>
        <p><strong>Keep the jumper wires.</strong> Seriously — the ESP32 DevKit pins are designed for breadboard/jumper use. Clipping them off and soldering is how people kill these boards. The BME680 breakout board also has standard 2.54mm header pins, so just plug female-to-female jumpers directly. No breadboard even needed unless you want one for strain relief.</p>
      </div>
    </div>
  </div>

  <!-- CSJ0 note -->
  <div class="addr-note" style="margin-top:12px;">
    <strong>I²C Address:</strong> Most BME680 breakouts default to <strong>0x76</strong>. Some (especially Adafruit) default to <strong>0x77</strong>. The YAML config below uses <code style="color:var(--blue);background:var(--surface-alt);padding:1px 5px;border-radius:3px;font-size:11px;">0x76</code> — if Home Assistant shows the sensor as unavailable after flashing, swap it to <code style="color:var(--blue);background:var(--surface-alt);padding:1px 5px;border-radius:3px;font-size:11px;">0x77</code> in the config and re-upload.
  </div>

  <!-- ── 2. ESPHOME YAML ── -->
  <div class="section-label">
    <span class="section-num">02</span>
    <h2>ESPHome Configuration</h2>
  </div>
  <div class="card">
    <div class="code-header">
      <div class="dots"><span style="background:#e85c5c"></span><span style="background:#e8a838"></span><span style="background:#5ecc6e"></span></div>
      <div class="lang">YAML · esphome</div>
    </div>
    <pre><span class="c-comment"># ─── esphome ────────────────────────────────────</span>
<span class="c-key">esphome:</span>
  <span class="c-key">name:</span> <span class="c-str">"living-room-env"</span>          <span class="c-comment"># change this to whatever</span>
  <span class="c-key">friendly_name:</span> <span class="c-str">"Living Room Environment"</span>

<span class="c-comment"># ─── board ──────────────────────────────────────</span>
<span class="c-key">esp32:</span>
  <span class="c-key">board:</span> <span class="c-val">esp32dev</span>               <span class="c-comment"># standard ESP32 DevKit V1</span>

<span class="c-comment"># ─── logging / api / ota ────────────────────────</span>
<span class="c-key">logger:</span>

<span class="c-key">api:</span>
  <span class="c-key">encryption:</span>
    <span class="c-key">key:</span> <span class="c-str">"PASTE_YOUR_HA_ENCRYPTION_KEY"</span>  <span class="c-comment"># from HA → ESPHome device page</span>

<span class="c-key">ota:</span>
  - <span class="c-key">platform:</span> <span class="c-plat">esphome</span>
    <span class="c-key">password:</span> <span class="c-str">"your_ota_password"</span>

<span class="c-comment"># ─── wifi ───────────────────────────────────────</span>
<span class="c-key">wifi:</span>
  <span class="c-key">ssid:</span> <span class="c-str">"YourWiFiNetwork"</span>
  <span class="c-key">password:</span> <span class="c-str">"YourWiFiPassword"</span>
  <span class="c-key">ap:</span>                             <span class="c-comment"># fallback hotspot if wifi fails</span>
    <span class="c-key">ssid:</span> <span class="c-str">"living-room-env-fallback"</span>
    <span class="c-key">password:</span> <span class="c-str">"12345678"</span>

<span class="c-key">captive_portal:</span>

<span class="c-comment"># ─── i2c bus ────────────────────────────────────</span>
<span class="c-key">i2c:</span>
  <span class="c-key">sda:</span> <span class="c-val">21</span>                        <span class="c-comment"># GPIO21</span>
  <span class="c-key">scl:</span> <span class="c-val">22</span>                        <span class="c-comment"># GPIO22</span>

<span class="c-comment"># ─── bme680 sensor (with BSEC for IAQ) ──────────</span>
<span class="c-key">bme680_bsec:</span>
  <span class="c-key">address:</span> <span class="c-val">0x76</span>                  <span class="c-comment"># swap to 0x77 if not found</span>
  <span class="c-key">sample_rate:</span> <span class="c-val">lp</span>               <span class="c-comment"># lp = every 3s, ulp = every 5min</span>
  <span class="c-key">state_save_interval:</span> <span class="c-val">6h</span>     <span class="c-comment"># saves IAQ calibration to flash</span>

<span class="c-key">sensor:</span>
  - <span class="c-key">platform:</span> <span class="c-plat">bme680_bsec</span>
    <span class="c-key">temperature:</span>
      <span class="c-key">name:</span> <span class="c-str">"Temperature"</span>
      <span class="c-key">filters:</span>
        - <span class="c-val">median</span>
    <span class="c-key">pressure:</span>
      <span class="c-key">name:</span> <span class="c-str">"Pressure"</span>
      <span class="c-key">filters:</span>
        - <span class="c-val">median</span>
    <span class="c-key">humidity:</span>
      <span class="c-key">name:</span> <span class="c-str">"Humidity"</span>
      <span class="c-key">filters:</span>
        - <span class="c-val">median</span>
    <span class="c-key">gas_resistance:</span>
      <span class="c-key">name:</span> <span class="c-str">"Gas Resistance"</span>
      <span class="c-key">filters:</span>
        - <span class="c-val">median</span>
    <span class="c-key">iaq:</span>
      <span class="c-key">name:</span> <span class="c-str">"Indoor Air Quality"</span>
      <span class="c-key">filters:</span>
        - <span class="c-val">median</span>
    <span class="c-key">iaq_accuracy:</span>
      <span class="c-key">name:</span> <span class="c-str">"IAQ Accuracy"</span>
    <span class="c-key">co2_equivalent:</span>
      <span class="c-key">name:</span> <span class="c-str">"CO2 Equivalent"</span>
      <span class="c-key">filters:</span>
        - <span class="c-val">median</span>
    <span class="c-key">breath_voc_equivalent:</span>
      <span class="c-key">name:</span> <span class="c-str">"Breath VOC"</span>
      <span class="c-key">filters:</span>
        - <span class="c-val">median</span></pre>
  </div>

  <!-- ── 3. FLASH STEPS ── -->
  <div class="section-label">
    <span class="section-num">03</span>
    <h2>Flash It</h2>
  </div>
  <div class="card">
    <div class="steps">
      <div class="step">
        <div class="step-num">1</div>
        <div>
          <h4>Install ESPHome</h4>
          <p>If you already have Home Assistant running, go to <strong style="color:#fff">Settings → Add-ons → ESPHome</strong> and install it there. That's the easiest path — everything lives in one place.</p>
        </div>
      </div>
      <div class="step">
        <div class="step-num">2</div>
        <div>
          <h4>Create a new device</h4>
          <p>Open the ESPHome dashboard, click <code>+ New Device</code>. Give it a name, pick <code>ESP32</code> as the board type. It'll generate a base config.</p>
        </div>
      </div>
      <div class="step">
        <div class="step-num">3</div>
        <div>
          <h4>Paste the YAML above</h4>
          <p>Replace the entire generated config with the YAML from section 02. Fill in your WiFi credentials and the encryption key from Home Assistant.</p>
        </div>
      </div>
      <div class="step">
        <div class="step-num">4</div>
        <div>
          <h4>First flash via USB</h4>
          <p>Plug the ESP32 into your computer via USB. In ESPHome, click <code>Upload</code>. It'll compile (takes a min) then flash over USB. After this first flash, all future updates are OTA over WiFi — no cable needed.</p>
        </div>
      </div>
      <div class="step">
        <div class="step-num">5</div>
        <div>
          <h4>It just shows up in HA</h4>
          <p>Go to Home Assistant → <strong style="color:#fff">Settings → Devices & Entities → Devices</strong>. Your "Living Room Environment" device will appear automatically with all 8 sensors. No manual entity setup needed.</p>
        </div>
      </div>
    </div>
  </div>

  <!-- ── 4. 3D PRINT ENCLOSURE ── -->
  <div class="section-label">
    <span class="section-num">04</span>
    <h2>3D Print Enclosure</h2>
  </div>
  <div class="card">
    <div class="print-grid">
      <div class="print-spec">
        <div class="ps-label">Outer Dimensions</div>
        <div class="ps-val">72 × 52 × 38 mm</div>
        <div class="ps-note">Fits ESP32 DevKit + BME680 breakout</div>
      </div>
      <div class="print-spec">
        <div class="ps-label">Wall Thickness</div>
        <div class="ps-val">2.0 mm</div>
        <div class="ps-note">Standard for FDM, solid enough</div>
      </div>
      <div class="print-spec">
        <div class="ps-label">USB Cutout</div>
        <div class="ps-val">12 × 6 mm</div>
        <div class="ps-note">Micro-USB or USB-C depending on your board</div>
      </div>
      <div class="print-spec">
        <div class="ps-label">Vent Holes</div>
        <div class="ps-val">6× Ø3mm</div>
        <div class="ps-note">Critical — BME680 needs air flow to read VOC</div>
      </div>
      <div class="print-spec">
        <div class="ps-label">Mounting</div>
        <div class="ps-val">2× M3 inserts</div>
        <div class="ps-note">Or just print with two screw ears</div>
      </div>
      <div class="print-spec">
        <div class="ps-label">Material</div>
        <div class="ps-val">PLA or PETG</div>
        <div class="ps-note">Nothing gets hot — PLA is fine</div>
      </div>
    </div>

    <!-- SVG enclosure diagram -->
    <div class="enclosure-wrap">
      <svg class="enclosure-svg" viewBox="0 0 620 320" xmlns="http://www.w3.org/2000/svg">
        <defs>
          <linearGradient id="boxGrad" x1="0" y1="0" x2="0" y2="1">
            <stop offset="0%" stop-color="#2a2d3a"/>
            <stop offset="100%" stop-color="#1a1c24"/>
          </linearGradient>
        </defs>
        <!-- outer shell top view -->
        <rect x="60" y="40" width="260" height="180" rx="8" fill="url(#boxGrad)" stroke="#3a3d4a" stroke-width="1.5"/>
        <!-- inner cavity -->
        <rect x="80" y="60" width="220" height="140" rx="4" fill="#111318" stroke="#2a2d3a" stroke-width="0.8" stroke-dasharray="4,3"/>

        <!-- ESP32 board -->
        <rect x="90" y="75" width="56" height="108" rx="3" fill="#1e2230" stroke="#5ea3e8" stroke-width="1" opacity="0.8"/>
        <text x="118" y="130" text-anchor="middle" font-family="IBM Plex Mono" font-size="7.5" fill="#5ea3e8">ESP32</text>
        <!-- ESP32 pins left side -->
        <line x1="90" y1="85" x2="82" y2="85" stroke="#5ea3e8" stroke-width="1.2" opacity="0.6"/>
        <line x1="90" y1="95" x2="82" y2="95" stroke="#5ea3e8" stroke-width="1.2" opacity="0.6"/>
        <line x1="90" y1="105" x2="82" y2="105" stroke="#5ea3e8" stroke-width="1.2" opacity="0.6"/>
        <!-- ESP32 pins right side -->
        <line x1="146" y1="85" x2="154" y2="85" stroke="#5ea3e8" stroke-width="1.2" opacity="0.6"/>
        <line x1="146" y1="95" x2="154" y2="95" stroke="#5ea3e8" stroke-width="1.2" opacity="0.6"/>
        <line x1="146" y1="105" x2="154" y2="105" stroke="#5ea3e8" stroke-width="1.2" opacity="0.6"/>

        <!-- USB cutout on left wall -->
        <rect x="56" y="125" width="8" height="14" rx="2" fill="#e8a838" opacity="0.7"/>
        <text x="60" y="146" text-anchor="middle" font-family="IBM Plex Mono" font-size="6" fill="#e8a838">USB</text>

        <!-- BME680 board -->
        <rect x="200" y="100" width="50" height="36" rx="3" fill="#1e2230" stroke="#5ecc6e" stroke-width="1" opacity="0.8"/>
        <text x="225" y="122" text-anchor="middle" font-family="IBM Plex Mono" font-size="7.5" fill="#5ecc6e">BME680</text>
        <!-- BME680 pins -->
        <line x1="210" y1="100" x2="210" y2="93" stroke="#5ecc6e" stroke-width="1.2" opacity="0.6"/>
        <line x1="220" y1="100" x2="220" y2="93" stroke="#5ecc6e" stroke-width="1.2" opacity="0.6"/>
        <line x1="230" y1="100" x2="230" y2="93" stroke="#5ecc6e" stroke-width="1.2" opacity="0.6"/>
        <line x1="240" y1="100" x2="240" y2="93" stroke="#5ecc6e" stroke-width="1.2" opacity="0.6"/>

        <!-- Jumper wires -->
        <path d="M146,82 C170,82 175,82 210,93" fill="none" stroke="#e85c5c" stroke-width="1.5" stroke-linecap="round"/>
        <path d="M146,90 C172,90 178,95 210,95" fill="none" stroke="#e8a838" stroke-width="1.5" stroke-linecap="round" opacity="0.9"/>
        <path d="M154,98 C178,98 184,97 220,93" fill="none" stroke="#e8d85c" stroke-width="1.5" stroke-linecap="round"/>
        <path d="M146,100 C170,103 176,107 210,107" fill="none" stroke="#5ea3e8" stroke-width="1.5" stroke-linecap="round"/>

        <!-- Vent holes on right side -->
        <circle cx="316" cy="72" r="4" fill="none" stroke="#6b6f7e" stroke-width="1" opacity="0.8"/>
        <circle cx="316" cy="90" r="4" fill="none" stroke="#6b6f7e" stroke-width="1" opacity="0.8"/>
        <circle cx="316" cy="108" r="4" fill="none" stroke="#6b6f7e" stroke-width="1" opacity="0.8"/>
        <circle cx="316" cy="126" r="4" fill="none" stroke="#6b6f7e" stroke-width="1" opacity="0.8"/>
        <circle cx="316" cy="144" r="4" fill="none" stroke="#6b6f7e" stroke-width="1" opacity="0.8"/>
        <circle cx="316" cy="162" r="4" fill="none" stroke="#6b6f7e" stroke-width="1" opacity="0.8"/>

        <!-- Labels / Legend -->
        <text x="400" y="55" font-family="IBM Plex Mono" font-size="8" fill="#6b6f7e" letter-spacing="1">LEGEND</text>
        <!-- ESP32 -->
        <rect x="398" y="68" width="14" height="10" rx="2" fill="none" stroke="#5ea3e8" stroke-width="1"/>
        <text x="418" y="77" font-family="IBM Plex Sans" font-size="9" fill="#c8cad4">ESP32 DevKit</text>
        <!-- BME680 -->
        <rect x="398" y="86" width="14" height="10" rx="2" fill="none" stroke="#5ecc6e" stroke-width="1"/>
        <text x="418" y="95" font-family="IBM Plex Sans" font-size="9" fill="#c8cad4">BME680</text>
        <!-- USB -->
        <rect x="398" y="104" width="14" height="10" rx="2" fill="#e8a838" opacity="0.7"/>
        <text x="418" y="113" font-family="IBM Plex Sans" font-size="9" fill="#c8cad4">USB cutout</text>
        <!-- Vents -->
        <circle cx="405" cy="130" r="3.5" fill="none" stroke="#6b6f7e" stroke-width="1"/>
        <text x="418" y="133" font-family="IBM Plex Sans" font-size="9" fill="#c8cad4">Vent holes</text>
        <!-- Wires -->
        <line x1="398" y1="150" x2="412" y2="150" stroke="#e85c5c" stroke-width="2" stroke-linecap="round"/>
        <text x="418" y="153" font-family="IBM Plex Sans" font-size="9" fill="#c8cad4">3V3 / VCC</text>
        <line x1="398" y1="166" x2="412" y2="166" stroke="#e8a838" stroke-width="2" stroke-linecap="round"/>
        <text x="418" y="169" font-family="IBM Plex Sans" font-size="9" fill="#c8cad4">SDA</text>
        <line x1="398" y1="182" x2="412" y2="182" stroke="#e8d85c" stroke-width="2" stroke-linecap="round"/>
        <text x="418" y="185" font-family="IBM Plex Sans" font-size="9" fill="#c8cad4">SCL</text>
        <line x1="398" y1="198" x2="412" y2="198" stroke="#5ea3e8" stroke-width="2" stroke-linecap="round"/>
        <text x="418" y="201" font-family="IBM Plex Sans" font-size="9" fill="#c8cad4">GND</text>

        <!-- Dimension callouts -->
        <text x="190" y="248" text-anchor="middle" font-family="IBM Plex Mono" font-size="8" fill="#6b6f7e">← 72 mm →</text>
        <line x1="60" y1="238" x2="320" y2="238" stroke="#3a3d4a" stroke-width="0.5"/>
        <line x1="60" y1="234" x2="60" y2="242" stroke="#3a3d4a" stroke-width="0.5"/>
        <line x1="320" y1="234" x2="320" y2="242" stroke="#3a3d4a" stroke-width="0.5"/>

        <text x="355" y="140" text-anchor="middle" font-family="IBM Plex Mono" font-size="8" fill="#6b6f7e" transform="rotate(90,355,140)">← 52 mm →</text>
        <line x1="345" y1="40" x2="345" y2="220" stroke="#3a3d4a" stroke-width="0.5"/>
        <line x1="341" y1="40" x2="349" y2="40" stroke="#3a3d4a" stroke-width="0.5"/>
        <line x1="341" y1="220" x2="349" y2="220" stroke="#3a3d4a" stroke-width="0.5"/>

        <!-- Title -->
        <text x="60" y="290" font-family="IBM Plex Mono" font-size="7.5" fill="#4a4f63" letter-spacing="0.5">TOP VIEW · NOT TO SCALE · Vent holes on BME680 side for airflow</text>
      </svg>
    </div>
  </div>

  <!-- ── 5. HARDWARE CHECKLIST ── -->
  <div class="section-label">
    <span class="section-num">05</span>
    <h2>What You Need</h2>
  </div>
  <div class="card">
    <div class="checklist">
      <div class="check-item">
        <div class="check-box"></div>
        <span>ESP32 DevKit V1 (the standard 30-pin one)</span>
      </div>
      <div class="check-item">
        <div class="check-box"></div>
        <span>BME680 breakout board (any brand — they're all the same chip)</span>
      </div>
      <div class="check-item">
        <div class="check-box"></div>
        <span>4× female-to-female jumper wires (any length, 10cm is plenty)</span>
      </div>
      <div class="check-item">
        <div class="check-box"></div>
        <span>USB cable for first flash (Micro-USB or USB-C depending on your ESP32)</span>
      </div>
      <div class="check-item">
        <div class="check-box"></div>
        <span>A computer with ESPHome installed (or HA add-on running)</span>
      </div>
      <div class="check-item">
        <div class="check-box"></div>
        <span>3D printer access for the enclosure (or just leave it loose, it works either way)</span>
      </div>
    </div>
  </div>

</div><!-- /container -->
</body>
</html>
