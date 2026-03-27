<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>PiKVM on Orange Pi One H3 — Complete Guide</title>
<link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@300;400;600;700&family=Syne:wght@400;600;800&display=swap" rel="stylesheet">
<style>
  :root {
    --bg: #0a0e13;
    --surface: #111720;
    --surface2: #1a2230;
    --border: #1e2d42;
    --accent: #00d4ff;
    --accent2: #00ff88;
    --accent3: #ff6b35;
    --warn: #ffcc00;
    --text: #c8d8e8;
    --text-dim: #5a7a99;
    --text-bright: #e8f4ff;
    --code-bg: #0d1620;
    --success: #00ff88;
    --danger: #ff4444;
  }

  * { margin: 0; padding: 0; box-sizing: border-box; }

  body {
    background: var(--bg);
    color: var(--text);
    font-family: 'JetBrains Mono', monospace;
    font-size: 14px;
    line-height: 1.7;
    min-height: 100vh;
  }

  /* Animated background grid */
  body::before {
    content: '';
    position: fixed;
    inset: 0;
    background-image: 
      linear-gradient(rgba(0,212,255,0.03) 1px, transparent 1px),
      linear-gradient(90deg, rgba(0,212,255,0.03) 1px, transparent 1px);
    background-size: 40px 40px;
    pointer-events: none;
    z-index: 0;
  }

  .container {
    max-width: 900px;
    margin: 0 auto;
    padding: 40px 24px 80px;
    position: relative;
    z-index: 1;
  }

  /* Header */
  .header {
    text-align: center;
    padding: 60px 0 50px;
    position: relative;
  }

  .header::after {
    content: '';
    display: block;
    width: 100%;
    height: 1px;
    background: linear-gradient(90deg, transparent, var(--accent), transparent);
    margin-top: 40px;
  }

  .badge {
    display: inline-block;
    background: rgba(0,212,255,0.1);
    border: 1px solid rgba(0,212,255,0.3);
    color: var(--accent);
    font-size: 11px;
    font-weight: 600;
    letter-spacing: 3px;
    padding: 5px 16px;
    text-transform: uppercase;
    margin-bottom: 20px;
  }

  h1 {
    font-family: 'Syne', sans-serif;
    font-size: clamp(28px, 5vw, 48px);
    font-weight: 800;
    color: var(--text-bright);
    line-height: 1.1;
    margin-bottom: 16px;
  }

  h1 span { color: var(--accent); }

  .subtitle {
    color: var(--text-dim);
    font-size: 13px;
    letter-spacing: 1px;
  }

  /* Hardware list */
  .hardware-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
    gap: 12px;
    margin: 24px 0;
  }

  .hw-card {
    background: var(--surface);
    border: 1px solid var(--border);
    padding: 16px;
    position: relative;
    transition: border-color 0.2s;
  }

  .hw-card:hover { border-color: var(--accent); }

  .hw-card .hw-status {
    font-size: 10px;
    letter-spacing: 2px;
    text-transform: uppercase;
    margin-bottom: 8px;
  }

  .hw-card .hw-status.have { color: var(--accent2); }
  .hw-card .hw-status.need { color: var(--accent3); }

  .hw-card .hw-name {
    color: var(--text-bright);
    font-weight: 600;
    font-size: 13px;
  }

  .hw-card .hw-note {
    color: var(--text-dim);
    font-size: 11px;
    margin-top: 4px;
  }

  /* Wiring diagram */
  .wiring {
    background: var(--code-bg);
    border: 1px solid var(--border);
    border-left: 3px solid var(--accent2);
    padding: 20px 24px;
    font-size: 13px;
    margin: 16px 0;
    overflow-x: auto;
  }

  .wiring .wire-line {
    display: flex;
    align-items: center;
    gap: 8px;
    margin: 4px 0;
    white-space: nowrap;
  }

  .wire-from { color: var(--accent); }
  .wire-arrow { color: var(--text-dim); }
  .wire-to { color: var(--accent2); }
  .wire-note { color: var(--text-dim); font-size: 11px; }

  /* Stage sections */
  .stage {
    margin: 40px 0;
    animation: fadeIn 0.5s ease both;
  }

  @keyframes fadeIn {
    from { opacity: 0; transform: translateY(10px); }
    to { opacity: 1; transform: translateY(0); }
  }

  .stage-header {
    display: flex;
    align-items: center;
    gap: 16px;
    margin-bottom: 20px;
    padding-bottom: 12px;
    border-bottom: 1px solid var(--border);
  }

  .stage-num {
    background: var(--accent);
    color: var(--bg);
    font-family: 'Syne', sans-serif;
    font-weight: 800;
    font-size: 12px;
    width: 32px;
    height: 32px;
    display: flex;
    align-items: center;
    justify-content: center;
    flex-shrink: 0;
  }

  .stage-title {
    font-family: 'Syne', sans-serif;
    font-size: 18px;
    font-weight: 700;
    color: var(--text-bright);
  }

  .stage-desc {
    font-size: 12px;
    color: var(--text-dim);
    margin-left: auto;
    text-align: right;
  }

  /* Steps */
  .step {
    margin: 20px 0;
    padding-left: 16px;
    border-left: 2px solid var(--border);
    transition: border-color 0.2s;
  }

  .step:hover { border-left-color: var(--accent); }

  .step-title {
    color: var(--accent);
    font-size: 12px;
    font-weight: 600;
    letter-spacing: 1px;
    text-transform: uppercase;
    margin-bottom: 8px;
  }

  .step p {
    color: var(--text);
    margin-bottom: 10px;
    font-size: 13px;
  }

  /* Code blocks */
  .code-block {
    background: var(--code-bg);
    border: 1px solid var(--border);
    padding: 0;
    margin: 10px 0;
    position: relative;
    overflow: hidden;
  }

  .code-label {
    background: var(--surface2);
    border-bottom: 1px solid var(--border);
    padding: 6px 14px;
    font-size: 10px;
    color: var(--text-dim);
    letter-spacing: 2px;
    text-transform: uppercase;
    display: flex;
    justify-content: space-between;
    align-items: center;
  }

  .copy-btn {
    background: none;
    border: 1px solid var(--border);
    color: var(--text-dim);
    font-family: 'JetBrains Mono', monospace;
    font-size: 9px;
    padding: 2px 8px;
    cursor: pointer;
    letter-spacing: 1px;
    transition: all 0.2s;
  }

  .copy-btn:hover {
    border-color: var(--accent);
    color: var(--accent);
  }

  .copy-btn.copied {
    border-color: var(--accent2);
    color: var(--accent2);
  }

  pre {
    padding: 16px;
    overflow-x: auto;
    font-family: 'JetBrains Mono', monospace;
    font-size: 13px;
    line-height: 1.6;
    color: var(--text);
  }

  pre .comment { color: var(--text-dim); font-style: italic; }
  pre .cmd { color: var(--accent2); }
  pre .path { color: var(--accent); }
  pre .string { color: var(--warn); }

  /* Alert boxes */
  .alert {
    padding: 14px 18px;
    margin: 16px 0;
    border-left: 3px solid;
    font-size: 12px;
    line-height: 1.6;
  }

  .alert-warn {
    background: rgba(255,204,0,0.05);
    border-color: var(--warn);
    color: var(--warn);
  }

  .alert-info {
    background: rgba(0,212,255,0.05);
    border-color: var(--accent);
    color: var(--accent);
  }

  .alert-success {
    background: rgba(0,255,136,0.05);
    border-color: var(--accent2);
    color: var(--accent2);
  }

  .alert-danger {
    background: rgba(255,68,68,0.05);
    border-color: var(--danger);
    color: var(--danger);
  }

  .alert strong { font-weight: 700; letter-spacing: 1px; }

  /* Table */
  table {
    width: 100%;
    border-collapse: collapse;
    margin: 16px 0;
    font-size: 12px;
  }

  th {
    background: var(--surface2);
    color: var(--accent);
    font-weight: 600;
    letter-spacing: 1px;
    text-transform: uppercase;
    font-size: 10px;
    padding: 10px 14px;
    text-align: left;
    border-bottom: 1px solid var(--border);
  }

  td {
    padding: 10px 14px;
    border-bottom: 1px solid var(--border);
    color: var(--text);
  }

  tr:hover td { background: rgba(0,212,255,0.03); }

  .tag-ok { color: var(--accent2); font-weight: 700; }
  .tag-no { color: var(--danger); font-weight: 700; }
  .tag-warn { color: var(--warn); font-weight: 700; }

  /* Link */
  a {
    color: var(--accent);
    text-decoration: none;
    border-bottom: 1px solid rgba(0,212,255,0.3);
    transition: border-color 0.2s;
  }
  a:hover { border-color: var(--accent); }

  /* Divider */
  .divider {
    height: 1px;
    background: linear-gradient(90deg, transparent, var(--border), transparent);
    margin: 40px 0;
  }

  /* Section heading */
  h2 {
    font-family: 'Syne', sans-serif;
    font-size: 13px;
    font-weight: 600;
    color: var(--text-dim);
    letter-spacing: 3px;
    text-transform: uppercase;
    margin: 24px 0 12px;
  }

  /* Footer */
  .footer {
    text-align: center;
    padding: 40px 0 20px;
    color: var(--text-dim);
    font-size: 11px;
    letter-spacing: 1px;
    border-top: 1px solid var(--border);
    margin-top: 60px;
  }

  /* Scrollbar */
  ::-webkit-scrollbar { width: 6px; height: 6px; }
  ::-webkit-scrollbar-track { background: var(--bg); }
  ::-webkit-scrollbar-thumb { background: var(--border); }
  ::-webkit-scrollbar-thumb:hover { background: var(--accent); }

  @media (max-width: 600px) {
    .stage-desc { display: none; }
    .hardware-grid { grid-template-columns: 1fr 1fr; }
  }
</style>
</head>
<body>
<div class="container">

  <!-- Header -->
  <div class="header">
    <div class="badge">DIY KVM over IP</div>
    <h1>PiKVM on <span>Orange Pi One H3</span></h1>
    <p class="subtitle">Complete Setup Guide — Armbian Trixie + kvmd-armbian + Tailscale</p>
  </div>

  <!-- Hardware -->
  <div class="stage">
    <div class="stage-header">
      <div class="stage-num">HW</div>
      <div class="stage-title">Hardware Required</div>
    </div>

    <div class="hardware-grid">
      <div class="hw-card">
        <div class="hw-status have">✓ Already Have</div>
        <div class="hw-name">Orange Pi One H3</div>
        <div class="hw-note">Allwinner H3, 512MB/1GB RAM</div>
      </div>
      <div class="hw-card">
        <div class="hw-status have">✓ Already Have</div>
        <div class="hw-name">USB HDMI Capture Card</div>
        <div class="hw-note">720p 30fps via USB 2.0</div>
      </div>
      <div class="hw-card">
        <div class="hw-status need">↓ Need</div>
        <div class="hw-name">MicroSD Card</div>
        <div class="hw-note">32GB, Class 10 / A1</div>
      </div>
      <div class="hw-card">
        <div class="hw-status need">↓ Need</div>
        <div class="hw-name">5V 2A DC Barrel PSU</div>
        <div class="hw-note">Powers OPi (OTG is busy)</div>
      </div>
      <div class="hw-card">
        <div class="hw-status need">↓ Need</div>
        <div class="hw-name">Micro USB Data Cable</div>
        <div class="hw-note">Old Android cable — OTG → Target PC</div>
      </div>
      <div class="hw-card">
        <div class="hw-status need">↓ Need</div>
        <div class="hw-name">HDMI Cable</div>
        <div class="hw-note">Target PC → Capture card</div>
      </div>
      <div class="hw-card">
        <div class="hw-status need">↓ Need</div>
        <div class="hw-name">USB Hub (optional)</div>
        <div class="hw-note">If adding WiFi dongle</div>
      </div>
      <div class="hw-card">
        <div class="hw-status need">↓ Need</div>
        <div class="hw-name">USB WiFi Dongle (optional)</div>
        <div class="hw-note">RTL8188 chipset recommended</div>
      </div>
    </div>

    <h2>Wiring</h2>
    <div class="wiring">
      <div class="wire-line">
        <span class="wire-from">[Target PC] HDMI out</span>
        <span class="wire-arrow">──────►</span>
        <span class="wire-to">[Capture Card]</span>
        <span class="wire-arrow">──USB──►</span>
        <span class="wire-to">[OPi One USB-A]</span>
      </div>
      <div class="wire-line">
        <span class="wire-from">[Target PC] USB-A</span>
        <span class="wire-arrow">◄──────</span>
        <span class="wire-to">[OPi One Micro USB OTG]</span>
        <span class="wire-note">&nbsp;← fake keyboard/mouse</span>
      </div>
      <div class="wire-line">
        <span class="wire-from">[OPi One DC Jack]</span>
        <span class="wire-arrow">◄──────</span>
        <span class="wire-to">[5V 2A Power Supply]</span>
      </div>
      <div class="wire-line">
        <span class="wire-from">[OPi One Ethernet]</span>
        <span class="wire-arrow">──────►</span>
        <span class="wire-to">[Router]</span>
        <span class="wire-note">&nbsp;← primary network</span>
      </div>
    </div>
  </div>

  <div class="divider"></div>

  <!-- Stage 1 -->
  <div class="stage">
    <div class="stage-header">
      <div class="stage-num">01</div>
      <div class="stage-title">Flash Armbian to SD Card</div>
      <div class="stage-desc">On your PC/laptop</div>
    </div>

    <div class="step">
      <div class="step-title">Download Armbian</div>
      <p>Go to <a href="https://www.armbian.com/orange-pi-one/" target="_blank">armbian.com/orange-pi-one</a> and download <strong>Armbian Debian Trixie Minimal</strong> (CLI, ~283MB). Do NOT use Ubuntu Noble.</p>
    </div>

    <div class="step">
      <div class="step-title">Flash with Balena Etcher</div>
      <p>Download <a href="https://etcher.balena.io" target="_blank">Balena Etcher</a>, select the .img.xz file, select your SD card, click Flash.</p>
    </div>
  </div>

  <div class="divider"></div>

  <!-- Stage 2 -->
  <div class="stage">
    <div class="stage-header">
      <div class="stage-num">02</div>
      <div class="stage-title">First Boot & System Prep</div>
      <div class="stage-desc">SSH as root</div>
    </div>

    <div class="alert alert-info"><strong>DEFAULT LOGIN:</strong> user: root / password: 1234 — it will ask you to change it on first login.</div>

    <div class="step">
      <div class="step-title">SSH In</div>
      <div class="code-block">
        <div class="code-label">terminal <button class="copy-btn" onclick="copyCode(this)">COPY</button></div>
        <pre>ssh root@&lt;OrangePi-IP&gt;</pre>
      </div>
      <p>Find the OPi IP from your router's admin page.</p>
    </div>

    <div class="step">
      <div class="step-title">Update System</div>
      <div class="code-block">
        <div class="code-label">bash <button class="copy-btn" onclick="copyCode(this)">COPY</button></div>
        <pre>apt update && apt upgrade -y
apt install git wget curl nano device-tree-compiler -y</pre>
      </div>
    </div>

    <div class="step">
      <div class="step-title">Fix DNS (if git clone fails)</div>
      <div class="code-block">
        <div class="code-label">bash <button class="copy-btn" onclick="copyCode(this)">COPY</button></div>
        <pre>echo "nameserver 8.8.8.8" > /etc/resolv.conf
echo "nameserver 8.8.4.4" >> /etc/resolv.conf</pre>
      </div>
    </div>
  </div>

  <div class="divider"></div>

  <!-- Stage 3 -->
  <div class="stage">
    <div class="stage-header">
      <div class="stage-num">03</div>
      <div class="stage-title">Enable OTG Peripheral Mode</div>
      <div class="stage-desc">Critical step — enables HID emulation</div>
    </div>

    <div class="alert alert-warn"><strong>⚠ IMPORTANT:</strong> This makes the Micro USB OTG port act as a fake keyboard/mouse to the target PC. Do not skip this.</div>

    <div class="step">
      <div class="step-title">Backup & Decompile DTB</div>
      <div class="code-block">
        <div class="code-label">bash <button class="copy-btn" onclick="copyCode(this)">COPY</button></div>
        <pre>cp /boot/dtb/sun8i-h3-orangepi-one.dtb /boot/dtb/sun8i-h3-orangepi-one.dtb.bak
dtc -I dtb -O dts /boot/dtb/sun8i-h3-orangepi-one.dtb -o /tmp/opi-one.dts</pre>
      </div>
    </div>

    <div class="step">
      <div class="step-title">Edit OTG Node</div>
      <div class="code-block">
        <div class="code-label">bash <button class="copy-btn" onclick="copyCode(this)">COPY</button></div>
        <pre>nano /tmp/opi-one.dts</pre>
      </div>
      <p>Press <strong>CTRL+W</strong>, search for <strong>dr_mode</strong>. Find the node inside <code>usb@01c19000</code> and change:</p>
      <div class="code-block">
        <div class="code-label">change this</div>
        <pre>dr_mode = "host";</pre>
      </div>
      <div class="code-block">
        <div class="code-label">to this</div>
        <pre>dr_mode = "peripheral";</pre>
      </div>
    </div>

    <div class="step">
      <div class="step-title">Recompile DTB & Update Boot Config</div>
      <div class="code-block">
        <div class="code-label">bash <button class="copy-btn" onclick="copyCode(this)">COPY</button></div>
        <pre>dtc -I dts -O dtb /tmp/opi-one.dts -o /boot/dtb/sun8i-h3-orangepi-one.dtb
echo "usbdev=otg" >> /boot/armbianEnv.txt
reboot</pre>
      </div>
    </div>

    <div class="step">
      <div class="step-title">Verify OTG Active (after reboot)</div>
      <div class="code-block">
        <div class="code-label">bash <button class="copy-btn" onclick="copyCode(this)">COPY</button></div>
        <pre>ls /sys/class/udc/</pre>
      </div>
      <div class="alert alert-success"><strong>✓ SUCCESS:</strong> You should see <code>musb-hdrc.4.auto</code> — OTG peripheral mode is active!</div>
    </div>
  </div>

  <div class="divider"></div>

  <!-- Stage 4 -->
  <div class="stage">
    <div class="stage-header">
      <div class="stage-num">04</div>
      <div class="stage-title">Install PiKVM (kvmd-armbian)</div>
      <div class="stage-desc">Run script twice</div>
    </div>

    <div class="step">
      <div class="step-title">Clone & Run Install — Part 1</div>
      <div class="code-block">
        <div class="code-label">bash <button class="copy-btn" onclick="copyCode(this)">COPY</button></div>
        <pre>git clone https://github.com/srepac/kvmd-armbian.git
cd kvmd-armbian
bash install.sh</pre>
      </div>
      <p>Let it run. It will ask to reboot after Part 1. Let it reboot.</p>
    </div>

    <div class="step">
      <div class="step-title">Run Install — Part 2 (after reboot)</div>
      <div class="code-block">
        <div class="code-label">bash <button class="copy-btn" onclick="copyCode(this)">COPY</button></div>
        <pre>cd kvmd-armbian
bash install.sh</pre>
      </div>
    </div>
  </div>

  <div class="divider"></div>

  <!-- Stage 5 - Python 3.13 Fixes -->
  <div class="stage">
    <div class="stage-header">
      <div class="stage-num">05</div>
      <div class="stage-title">Python 3.13 Compatibility Fixes</div>
      <div class="stage-desc">Required for Debian Trixie</div>
    </div>

    <div class="alert alert-warn"><strong>⚠ NOTE:</strong> Debian Trixie ships with Python 3.13 which has breaking changes. Apply ALL fixes below before answering 'y' to the script.</div>

    <div class="step">
      <div class="step-title">Fix 1 — Install Missing Python Modules</div>
      <div class="code-block">
        <div class="code-label">bash <button class="copy-btn" onclick="copyCode(this)">COPY</button></div>
        <pre>apt install -y python3-cffi python3-dev libffi-dev gcc
pip3 install dbus-next aiohttp aiofiles pyserial setproctitle mako pygments python-xlib zstandard --break-system-packages</pre>
      </div>
    </div>

    <div class="step">
      <div class="step-title">Fix 2 — Install ustreamer</div>
      <div class="code-block">
        <div class="code-label">bash <button class="copy-btn" onclick="copyCode(this)">COPY</button></div>
        <pre>apt install -y build-essential libevent-dev libjpeg-dev libbsd-dev libgpiod-dev
git clone https://github.com/pikvm/ustreamer.git
cd ustreamer && make && make install && cd ..</pre>
      </div>
    </div>

    <div class="step">
      <div class="step-title">Fix 3 — Fix gpiod Version</div>
      <div class="code-block">
        <div class="code-label">bash <button class="copy-btn" onclick="copyCode(this)">COPY</button></div>
        <pre>rm -rf /usr/lib/python3/dist-packages/gpiod
rm -rf /usr/lib/python3/dist-packages/gpiod-2.2.0.dist-info
pip3 install gpiod==1.5.4 --break-system-packages</pre>
      </div>
    </div>

    <div class="step">
      <div class="step-title">Fix 4 — Patch aiogp.py (gpiod API changes)</div>
      <div class="code-block">
        <div class="code-label">bash <button class="copy-btn" onclick="copyCode(this)">COPY</button></div>
        <pre>sed -i 's/gpiod\.Line\b/object/g' /usr/lib/python3/dist-packages/kvmd/aiogp.py
sed -i 's/gpiod\.LineEvent\b/object/g' /usr/lib/python3/dist-packages/kvmd/aiogp.py
sed -i 's/gpiod\.LineEdge\b/object/g' /usr/lib/python3/dist-packages/kvmd/aiogp.py
sed -i 's/gpiod\.LineBulk\b/object/g' /usr/lib/python3/dist-packages/kvmd/aiogp.py</pre>
      </div>
    </div>

    <div class="step">
      <div class="step-title">Fix 5 — Patch keysym.py (Python 3.13 removed find_module)</div>
      <div class="code-block">
        <div class="code-label">bash <button class="copy-btn" onclick="copyCode(this)">COPY</button></div>
        <pre>sed -i 's/        loader = finder.find_module(module_name)/        spec = importlib.util.spec_from_file_location(module_name, finder.path + "\/" + module_name + ".py")\n        loader = spec.loader if spec else None/' /usr/lib/python3/dist-packages/kvmd/keyboard/keysym.py
sed -i 's/        module = loader.load_module(module_name)/        module = importlib.util.module_from_spec(spec)\n        loader.exec_module(module)/' /usr/lib/python3/dist-packages/kvmd/keyboard/keysym.py</pre>
      </div>
    </div>

    <div class="step">
      <div class="step-title">Fix 6 — Patch gpio.py (gpiod.Chip API change)</div>
      <div class="code-block">
        <div class="code-label">bash <button class="copy-btn" onclick="copyCode(this)">COPY</button></div>
        <pre>sed -i 's/gpiod\.Chip/gpiod.chip/g' /usr/lib/python3/dist-packages/kvmd/plugins/ugpio/gpio.py
sed -i 's/gpiod\.LINE_REQ_DIR_OUT/gpiod.line.DIRECTION_OUTPUT/g' /usr/lib/python3/dist-packages/kvmd/plugins/ugpio/gpio.py</pre>
      </div>
    </div>

    <div class="step">
      <div class="step-title">Verify kvmd -m runs clean</div>
      <div class="code-block">
        <div class="code-label">bash <button class="copy-btn" onclick="copyCode(this)">COPY</button></div>
        <pre>kvmd -m</pre>
      </div>
      <p>When it runs without errors, answer <strong>y</strong> in the install script terminal.</p>
    </div>
  </div>

  <div class="divider"></div>

  <!-- Stage 6 -->
  <div class="stage">
    <div class="stage-header">
      <div class="stage-num">06</div>
      <div class="stage-title">Configure PiKVM</div>
      <div class="stage-desc">override.yaml + password</div>
    </div>

    <div class="step">
      <div class="step-title">Edit override.yaml</div>
      <div class="code-block">
        <div class="code-label">bash <button class="copy-btn" onclick="copyCode(this)">COPY</button></div>
        <pre>nano /etc/kvmd/override.yaml</pre>
      </div>
      <p>Make the file contain exactly this:</p>
      <div class="code-block">
        <div class="code-label">yaml <button class="copy-btn" onclick="copyCode(this)">COPY</button></div>
        <pre>kvmd:
    hid:
        type: otg
        mouse_alt:
            device: /dev/kvmd-hid-mouse-alt
    msd:
        type: disabled
    atx:
        type: disabled
    streamer:
        forever: true
        cmd_append:
            - "--slowdown"
        resolution:
            default: 1280x720
    gpio:
        drivers: {}
        scheme: {}</pre>
      </div>
    </div>

    <div class="step">
      <div class="step-title">Set Admin Password</div>
      <div class="code-block">
        <div class="code-label">bash <button class="copy-btn" onclick="copyCode(this)">COPY</button></div>
        <pre>kvmd-htpasswd set admin</pre>
      </div>
    </div>

    <div class="step">
      <div class="step-title">Restart & Verify Services</div>
      <div class="code-block">
        <div class="code-label">bash <button class="copy-btn" onclick="copyCode(this)">COPY</button></div>
        <pre>systemctl daemon-reload
systemctl restart kvmd kvmd-nginx
systemctl status kvmd kvmd-nginx kvmd-otg</pre>
      </div>
    </div>
  </div>

  <div class="divider"></div>

  <!-- Stage 7 -->
  <div class="stage">
    <div class="stage-header">
      <div class="stage-num">07</div>
      <div class="stage-title">Install Tailscale (Remote Access)</div>
      <div class="stage-desc">Works even without port forwarding</div>
    </div>

    <div class="step">
      <div class="step-title">Install & Connect</div>
      <div class="code-block">
        <div class="code-label">bash <button class="copy-btn" onclick="copyCode(this)">COPY</button></div>
        <pre>curl -fsSL https://tailscale.com/install.sh | sh
tailscale up</pre>
      </div>
      <p>Open the URL it gives you, log in to authorize the device.</p>
    </div>

    <div class="step">
      <div class="step-title">Get Tailscale IP & Enable on Boot</div>
      <div class="code-block">
        <div class="code-label">bash <button class="copy-btn" onclick="copyCode(this)">COPY</button></div>
        <pre>tailscale ip -4
systemctl enable tailscaled</pre>
      </div>
    </div>
  </div>

  <div class="divider"></div>

  <!-- Stage 8 - WiFi Backup -->
  <div class="stage">
    <div class="stage-header">
      <div class="stage-num">08</div>
      <div class="stage-title">WiFi Backup Connection (Optional)</div>
      <div class="stage-desc">Requires USB WiFi dongle via USB hub</div>
    </div>

    <div class="alert alert-info"><strong>NOTE:</strong> OPi One has no built-in WiFi. You need a USB WiFi dongle (RTL8188 chipset recommended) connected via USB hub alongside the capture card.</div>

    <div class="step">
      <div class="step-title">Connect to WiFi</div>
      <div class="code-block">
        <div class="code-label">bash <button class="copy-btn" onclick="copyCode(this)">COPY</button></div>
        <pre>nmcli dev wifi connect "YourWiFiName" password "YourWiFiPassword"</pre>
      </div>
    </div>

    <div class="step">
      <div class="step-title">Set LAN as Priority, WiFi as Backup</div>
      <div class="code-block">
        <div class="code-label">/etc/network/interfaces <button class="copy-btn" onclick="copyCode(this)">COPY</button></div>
        <pre>auto eth0
iface eth0 inet static
    address 192.168.1.100
    netmask 255.255.255.0
    gateway 192.168.1.1
    dns-nameservers 8.8.8.8
    metric 100

auto wlan0
iface wlan0 inet dhcp
    wpa-ssid YourWiFiName
    wpa-psk YourWiFiPassword
    metric 700</pre>
      </div>
    </div>
  </div>

  <div class="divider"></div>

  <!-- Access -->
  <div class="stage">
    <div class="stage-header">
      <div class="stage-num">✓</div>
      <div class="stage-title">Access PiKVM</div>
    </div>

    <table>
      <tr>
        <th>Method</th>
        <th>URL</th>
        <th>When</th>
      </tr>
      <tr>
        <td>Local Network</td>
        <td><code>https://192.168.1.100</code></td>
        <td>Same network</td>
      </tr>
      <tr>
        <td>Tailscale (Remote)</td>
        <td><code>https://&lt;tailscale-ip&gt;</code></td>
        <td>Anywhere in the world</td>
      </tr>
    </table>

    <p style="margin: 16px 0; font-size: 13px;">Login: <strong style="color:var(--accent)">admin</strong> / your password. Accept the SSL certificate warning.</p>

    <div class="alert alert-success"><strong>🎉 DONE!</strong> You now have a fully working KVM over IP — see your target PC screen and control keyboard/mouse remotely from any browser.</div>
  </div>

  <div class="divider"></div>

  <!-- Troubleshooting -->
  <div class="stage">
    <div class="stage-header">
      <div class="stage-num">?</div>
      <div class="stage-title">Troubleshooting</div>
    </div>

    <table>
      <tr>
        <th>Problem</th>
        <th>Fix</th>
      </tr>
      <tr>
        <td>git clone fails</td>
        <td><code>echo "nameserver 8.8.8.8" > /etc/resolv.conf</code></td>
      </tr>
      <tr>
        <td>No /dev/video0</td>
        <td>Replug capture card, run <code>dmesg | grep video</code></td>
      </tr>
      <tr>
        <td>500 Internal Server Error</td>
        <td><code>journalctl -u kvmd -n 50 --no-pager</code></td>
      </tr>
      <tr>
        <td>kvmd not running</td>
        <td><code>systemctl restart kvmd && systemctl status kvmd</code></td>
      </tr>
      <tr>
        <td>OTG not detected</td>
        <td>Re-check DTB edit, verify <code>ls /sys/class/udc/</code></td>
      </tr>
      <tr>
        <td>Wrong password</td>
        <td><code>kvmd-htpasswd set admin</code></td>
      </tr>
      <tr>
        <td>Tailscale not connecting</td>
        <td><code>tailscale up</code> again</td>
      </tr>
    </table>
  </div>

  <div class="footer">
    PiKVM on Orange Pi One H3 — Built with kvmd-armbian by srepac<br>
    <a href="https://github.com/srepac/kvmd-armbian" target="_blank">github.com/srepac/kvmd-armbian</a> &nbsp;·&nbsp;
    <a href="https://pikvm.org" target="_blank">pikvm.org</a>
  </div>

</div>

<script>
function copyCode(btn) {
  const pre = btn.closest('.code-block').querySelector('pre');
  navigator.clipboard.writeText(pre.innerText).then(() => {
    btn.textContent = 'COPIED!';
    btn.classList.add('copied');
    setTimeout(() => {
      btn.textContent = 'COPY';
      btn.classList.remove('copied');
    }, 2000);
  });
}
</script>
</body>
</html>
