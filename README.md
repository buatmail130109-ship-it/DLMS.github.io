# DLMS.github.io
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>3-Way DLMS with Video & Playlist Support</title>
    <style>
        :root {
            --bg-main: #121318;
            --panel-bg: #1e2029;
            --card-bg: #272a37;
            --text-main: #e0e6ed;
            --text-muted: #8a93a0;
            --accent-blue: #4f46e5;
            --accent-cyan: #06b6d4;
            --accent-red: #ef4444;
            --accent-yellow: #f59e0b;
            --accent-green: #10b981;
            --border: #374151;
        }

        * { box-sizing: border-box; margin: 0; padding: 0; font-family: 'Segoe UI', system-ui, sans-serif; }
        body { background-color: var(--bg-main); color: var(--text-main); padding: 20px; }

        .dlms-container { max-width: 1200px; margin: 0 auto; display: flex; flex-direction: column; gap: 20px; }

        /* HEADER */
        .header { background: var(--panel-bg); border: 1px solid var(--border); padding: 15px 20px; border-radius: 8px; display: flex; justify-content: space-between; align-items: center; }
        .header h1 { font-size: 1.5rem; color: var(--accent-cyan); letter-spacing: 1px; }

        /* MEDIA LAYOUT (VIDEO & PLAYLIST) */
        .media-layout { display: flex; gap: 20px; flex-wrap: wrap; }
        
        .video-box { flex: 2; min-width: 320px; background: #000; border: 1px solid var(--border); border-radius: 8px; overflow: hidden; display: flex; flex-direction: column; justify-content: center; position: relative; min-height: 250px; }
        video { width: 100%; height: 100%; max-height: 350px; object-fit: contain; }

        .playlist-box { flex: 1; min-width: 280px; background: var(--panel-bg); border: 1px solid var(--border); border-radius: 8px; padding: 15px; display: flex; flex-direction: column; gap: 10px; }
        
        .btn { background: var(--border); color: var(--text-main); border: none; padding: 8px 14px; border-radius: 6px; cursor: pointer; font-weight: bold; transition: 0.2s; font-size: 0.85rem; }
        .btn:hover { background: var(--accent-blue); }
        .btn-danger { background: var(--accent-red); }
        .btn-danger:hover { background: #dc2626; }
        .btn-danger.muted { background: #7f1d1d; color: var(--accent-red); border: 1px solid var(--accent-red); }

        .playlist-items { background: var(--card-bg); border: 1px solid var(--border); border-radius: 6px; flex-grow: 1; max-height: 200px; overflow-y: auto; padding: 5px; }
        .playlist-item { padding: 8px 10px; border-radius: 4px; cursor: pointer; font-size: 0.85rem; display: flex; justify-content: space-between; margin-bottom: 2px; }
        .playlist-item:hover { background: #323648; }
        .playlist-item.active { background: var(--accent-blue); color: #fff; font-weight: bold; }

        /* OUTPUT MODULES */
        .outputs-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(320px, 1fr)); gap: 15px; }
        .out-card { background: var(--panel-bg); border: 1px solid var(--border); border-radius: 8px; padding: 15px; display: flex; flex-direction: column; gap: 15px; }
        .out-card.low { border-top: 4px solid var(--accent-red); }
        .out-card.mid { border-top: 4px solid var(--accent-yellow); }
        .out-card.high { border-top: 4px solid var(--accent-cyan); }

        .card-header { display: flex; justify-content: space-between; align-items: center; border-bottom: 1px solid var(--border); padding-bottom: 8px; }
        .card-title { font-weight: bold; text-transform: uppercase; }

        /* CONTROL GROUPS */
        .control-group { background: var(--card-bg); padding: 10px; border-radius: 6px; display: flex; flex-direction: column; gap: 8px; }
        .control-group label { font-size: 0.8rem; color: var(--text-muted); text-transform: uppercase; font-weight: bold; display: flex; justify-content: space-between; }
        
        input[type="range"] { width: 100%; accent-color: var(--accent-cyan); cursor: pointer; }
        .val-disp { font-family: monospace; color: var(--accent-cyan); }

        /* VISUALIZER */
        .visualizer-box { background: #000; border: 1px solid var(--border); border-radius: 8px; height: 120px; overflow: hidden; }
        canvas { width: 100%; height: 100%; }
    </style>
</head>
<body>

<div class="dlms-container">
    <div class="header">
        <h1>DLMS-3000 Media Processor</h1>
        <span>3-Way DSP System + Video Playlist</span>
    </div>

    <!-- VIDEO PLAYER & PLAYLIST CONTROL -->
    <div class="media-layout">
        <div class="video-box">
            <video id="media-element" controls></video>
        </div>

        <div class="playlist-box">
            <input type="file" id="media-files" accept="audio/*,video/*" multiple style="display: none;">
            <button class="btn" onclick="document.getElementById('media-files').click()">+ Load Audio/Video Files</button>
            
            <div style="display: flex; gap: 8px; margin-top: 5px;">
                <button class="btn" id="btn-prev">Prev</button>
                <button class="btn" id="btn-play">Play</button>
                <button class="btn" id="btn-pause">Pause</button>
                <button class="btn" id="btn-next">Next</button>
            </div>

            <div class="playlist-items" id="playlist-container">
                <div style="color: var(--text-muted); font-size: 0.8rem; text-align: center; margin-top: 10px;">Playlist Kosong</div>
            </div>
        </div>
    </div>

    <!-- SPECTRUM ANALYZER -->
    <div class="visualizer-box">
        <canvas id="spectrum"></canvas>
    </div>

    <!-- OUTPUT CHANNELS (LOW, MID, HIGH) -->
    <div class="outputs-grid">
        
        <!-- OUTPUT 1: LOW / SUB -->
        <div class="out-card low">
            <div class="card-header">
                <span class="card-title" style="color: var(--accent-red);">Out 1: Low / Sub</span>
                <button class="btn btn-danger" id="mute-low">MUTE</button>
            </div>
            
            <div class="control-group">
                <label>Gain <span class="val-disp" id="val-gain-low">0 dB</span></label>
                <input type="range" id="gain-low" min="-24" max="12" step="0.5" value="0">
            </div>

            <div class="control-group">
                <label>HPF Cutoff (Subsonic) <span class="val-disp" id="val-hpf-low">30 Hz</span></label>
                <input type="range" id="hpf-low" min="20" max="100" step="1" value="30">
            </div>

            <div class="control-group">
                <label>LPF Cutoff (Crossover) <span class="val-disp" id="val-lpf-low">100 Hz</span></label>
                <input type="range" id="lpf-low" min="60" max="250" step="1" value="100">
            </div>

            <div class="control-group">
                <label>PEQ Gain (60 Hz Boost) <span class="val-disp" id="val-eq-low">0 dB</span></label>
                <input type="range" id="eq-low" min="-12" max="12" step="0.5" value="0">
            </div>

            <div class="control-group">
                <label>Delay <span class="val-disp" id="val-delay-low">0.0 ms</span></label>
                <input type="range" id="delay-low" min="0" max="20" step="0.1" value="0">
            </div>
        </div>

        <!-- OUTPUT 2: MID -->
        <div class="out-card mid">
            <div class="card-header">
                <span class="card-title" style="color: var(--accent-yellow);">Out 2: Middle</span>
                <button class="btn btn-danger" id="mute-mid">MUTE</button>
            </div>

            <div class="control-group">
                <label>Gain <span class="val-disp" id="val-gain-mid">0 dB</span></label>
                <input type="range" id="gain-mid" min="-24" max="12" step="0.5" value="0">
            </div>

            <div class="control-group">
                <label>HPF Cutoff <span class="val-disp" id="val-hpf-mid">100 Hz</span></label>
                <input type="range" id="hpf-mid" min="60" max="500" step="1" value="100">
            </div>

            <div class="control-group">
                <label>LPF Cutoff <span class="val-disp" id="val-lpf-mid">2500 Hz</span></label>
                <input type="range" id="lpf-mid" min="1000" max="6000" step="10" value="2500">
            </div>

            <div class="control-group">
                <label>PEQ Gain (1 kHz Vocal) <span class="val-disp" id="val-eq-mid">0 dB</span></label>
                <input type="range" id="eq-mid" min="-12" max="12" step="0.5" value="0">
            </div>

            <div class="control-group">
                <label>Delay <span class="val-disp" id="val-delay-mid">0.0 ms</span></label>
                <input type="range" id="delay-mid" min="0" max="20" step="0.1" value="0">
            </div>
        </div>

        <!-- OUTPUT 3: HIGH -->
        <div class="out-card high">
            <div class="card-header">
                <span class="card-title" style="color: var(--accent-cyan);">Out 3: High / Treble</span>
                <button class="btn btn-danger" id="mute-high">MUTE</button>
            </div>

            <div class="control-group">
                <label>Gain <span class="val-disp" id="val-gain-high">0 dB</span></label>
                <input type="range" id="gain-high" min="-24" max="12" step="0.5" value="0">
            </div>

            <div class="control-group">
                <label>HPF Cutoff <span class="val-disp" id="val-hpf-high">2500 Hz</span></label>
                <input type="range" id="hpf-high" min="1000" max="8000" step="50" value="2500">
            </div>

            <div class="control-group">
                <label>LPF Cutoff (Air Band) <span class="val-disp" id="val-lpf-high">18000 Hz</span></label>
                <input type="range" id="lpf-high" min="10000" max="20000" step="100" value="18000">
            </div>

            <div class="control-group">
                <label>PEQ Gain (10 kHz High) <span class="val-disp" id="val-eq-high">0 dB</span></label>
                <input type="range" id="eq-high" min="-12" max="12" step="0.5" value="0">
            </div>

            <div class="control-group">
                <label>Delay <span class="val-disp" id="val-delay-high">0.0 ms</span></label>
                <input type="range" id="delay-high" min="0" max="20" step="0.1" value="0">
            </div>
        </div>

    </div>
</div>

<script>
    let audioCtx, sourceNode, analyser;
    let playlist = [];
    let currentIndex = 0;
    const mediaEl = document.getElementById('media-element');
    
    const dsp = { low: {}, mid: {}, high: {} };

    function initAudio() {
        if (audioCtx) return;
        audioCtx = new (window.AudioContext || window.webkitAudioContext)();
        
        sourceNode = audioCtx.createMediaElementSource(mediaEl);
        analyser = audioCtx.createAnalyser();
        analyser.fftSize = 256;

        ['low', 'mid', 'high'].forEach(ch => {
            dsp[ch].hpf = audioCtx.createBiquadFilter();
            dsp[ch].hpf.type = 'highpass';

            dsp[ch].lpf = audioCtx.createBiquadFilter();
            dsp[ch].lpf.type = 'lowpass';

            dsp[ch].eq = audioCtx.createBiquadFilter();
            dsp[ch].eq.type = 'peaking';

            dsp[ch].delay = audioCtx.createDelay();
            dsp[ch].gain = audioCtx.createGain();

            sourceNode.connect(dsp[ch].hpf);
            dsp[ch].hpf.connect(dsp[ch].lpf);
            dsp[ch].lpf.connect(dsp[ch].eq);
            dsp[ch].eq.connect(dsp[ch].delay);
            dsp[ch].delay.connect(dsp[ch].gain);
            dsp[ch].gain.connect(analyser);
        });

        analyser.connect(audioCtx.destination);
        renderSpectrum();
    }

    // MANAJEMEN PLAYLIST
    document.getElementById('media-files').onchange = (e) => {
        initAudio();
        const files = Array.from(e.target.files);
        if (!files.length) return;

        playlist = files;
        const container = document.getElementById('playlist-container');
        container.innerHTML = '';

        playlist.forEach((file, idx) => {
            const item = document.createElement('div');
            item.className = `playlist-item ${idx === 0 ? 'active' : ''}`;
            item.innerText = `${idx + 1}. ${file.name}`;
            item.onclick = () => loadTrack(idx);
            container.appendChild(item);
        });

        loadTrack(0);
    };

    function loadTrack(index) {
        if (!playlist[index]) return;
        currentIndex = index;
        mediaEl.src = URL.createObjectURL(playlist[index]);
        
        document.querySelectorAll('.playlist-item').forEach((item, i) => {
            item.classList.toggle('active', i === index);
        });

        mediaEl.play();
    }

    // OTOMATIS LANJUT KE LAGU/VIDEO NEXT JIKA SELESAI
    mediaEl.onended = () => {
        if (currentIndex < playlist.length - 1) {
            loadTrack(currentIndex + 1);
        }
    };

    // CONTROLS MEDIA
    document.getElementById('btn-play').onclick = () => { initAudio(); mediaEl.play(); };
    document.getElementById('btn-pause').onclick = () => mediaEl.pause();
    document.getElementById('btn-prev').onclick = () => loadTrack(Math.max(0, currentIndex - 1));
    document.getElementById('btn-next').onclick = () => loadTrack(Math.min(playlist.length - 1, currentIndex + 1));

    // BINDING CONTROL DSP
    function bindControl(ch, type, elementId, dispId, unit = '') {
        const el = document.getElementById(elementId);
        const disp = document.getElementById(dispId);
        el.oninput = () => {
            const val = parseFloat(el.value);
            disp.innerText = `${val} ${unit}`;
            if (!audioCtx) return;

            if (type === 'hpf') dsp[ch].hpf.frequency.value = val;
            if (type === 'lpf') dsp[ch].lpf.frequency.value = val;
            if (type === 'eq') dsp[ch].eq.gain.value = val;
            if (type === 'gain') dsp[ch].gain.gain.value = Math.pow(10, val / 20);
            if (type === 'delay') dsp[ch].delay.delayTime.value = val / 1000;
        };
    }

    ['low', 'mid', 'high'].forEach(ch => {
        bindControl(ch, 'gain', `gain-${ch}`, `val-gain-${ch}`, 'dB');
        bindControl(ch, 'hpf', `hpf-${ch}`, `val-hpf-${ch}`, 'Hz');
        bindControl(ch, 'lpf', `lpf-${ch}`, `val-lpf-${ch}`, 'Hz');
        bindControl(ch, 'eq', `eq-${ch}`, `val-eq-${ch}`, 'dB');
        bindControl(ch, 'delay', `delay-${ch}`, `val-delay-${ch}`, 'ms');

        let isMuted = false;
        const btn = document.getElementById(`mute-${ch}`);
        btn.onclick = () => {
            isMuted = !isMuted;
            btn.classList.toggle('muted', isMuted);
            if (audioCtx) {
                const gainVal = parseFloat(document.getElementById(`gain-${ch}`).value);
                dsp[ch].gain.gain.value = isMuted ? 0 : Math.pow(10, gainVal / 20);
            }
        };
    });

    // VISUALIZER SPECTRUM
    function renderSpectrum() {
        requestAnimationFrame(renderSpectrum);
        const canvas = document.getElementById('spectrum');
        const ctx = canvas.getContext('2d');
        canvas.width = canvas.parentElement.clientWidth;
        canvas.height = canvas.parentElement.clientHeight;

        if (!analyser) return;

        const bufferLength = analyser.frequencyBinCount;
        const dataArray = new Uint8Array(bufferLength);
        analyser.getByteFrequencyData(dataArray);

        ctx.fillStyle = '#000';
        ctx.fillRect(0, 0, canvas.width, canvas.height);

        const barWidth = (canvas.width / bufferLength) * 2;
        let x = 0;

        for (let i = 0; i < bufferLength; i++) {
            const barHeight = (dataArray[i] / 255) * canvas.height;
            ctx.fillStyle = `hsl(${i * 3}, 80%, 50%)`;
            ctx.fillRect(x, canvas.height - barHeight, barWidth, barHeight);
            x += barWidth + 1;
        }
    }
</script>

</body>
</html>
