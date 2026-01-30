<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>GridOS | Stable Split</title>
    
    <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.1/dist/chart.umd.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/hammer.js/2.0.8/hammer.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/chartjs-plugin-zoom/2.0.1/chartjs-plugin-zoom.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/animejs@3.2.1/lib/anime.min.js"></script>
    
    <link href="https://fonts.googleapis.com/css2?family=Merriweather:ital,wght@0,300;0,400;0,700;1,700&family=Inter:wght@400;500;600;700&family=JetBrains+Mono:wght@500&display=swap" rel="stylesheet">
    
    <style>
        :root {
            --c-solar: #fbbf24;    --c-wind: #34d399;     --c-gas: #3b82f6;
            --c-nuclear: #8b5cf6;  --c-hydro: #0ea5e9;    --c-geo: #d97706;
            --c-bio: #166534;      --c-batteries: #ec4899;--c-imports: #06b6d4;

            /* Apple's-style responsive page margins (safe-area aware) */
            --page-padding: clamp(20px, 3.2vw, 64px);
            /* Orb sizing and shadow tuning */
            --orb-size: 110px; /* base orb diameter */
            --orb-shadow: 0 0 80px rgba(255,255,255,0.45);
            --glass-bg: rgba(255, 255, 255, 0.88);
            --glass-border: rgba(255, 255, 255, 0.6);
            --text-dark: #0f172a;
        }

        body {
            margin: 0; height: 100vh; width: 100vw;
            font-family: Georgia, 'Merriweather', serif; /* Georgia + Merriweather per request */
            background: #000;
            /* allow page to scroll when vertical space is constrained (laptops) */
            overflow: auto;
            display: flex;
            align-items: stretch;
            flex-wrap: nowrap;
            min-height: 100vh;
            padding-left: env(safe-area-inset-left);
            padding-right: env(safe-area-inset-right);
        }

        #sky-backdrop {
            position: absolute; inset: 0; z-index: 0;
            background: linear-gradient(to bottom, #0f172a, #000);
            transition: background 1.5s ease;
        }

        /* --- LEFT PANEL (NARRATIVE) --- */
        .stage-left {
            flex: 1 1 55%; min-width: 300px; height: 100%;
            position: relative; z-index: 1;
            display: flex; flex-direction: column;
            justify-content: center; padding: calc(var(--page-padding) * 2.16); /* ~20% larger spacing */
            box-sizing: border-box;
            color: white;
            pointer-events: none;
        }

        .orb-container {
            position: absolute; top: 12%; left: 8%;
            width: calc(var(--orb-size) * 2.4); height: calc(var(--orb-size) * 2.4); /* container scales with orb */
            pointer-events: none;
        }
        .orb {
            position: absolute; left: 50%; transform: translateX(-50%);
            width: var(--orb-size); height: var(--orb-size); border-radius: 50%;
            background: #fff; box-shadow: var(--orb-shadow);
            transition: top 300ms cubic-bezier(.22,.9,.32,1), opacity 200ms ease, transform 200ms ease;
        }
        #sun { background: #facc15; }
        #moon { background: #f1f5f9; }

        .narrative-box { position: relative; z-index: 2; text-shadow: 0 6px 24px rgba(0,0,0,0.55); }
        .icon-box svg { width: 64px; height: 64px; fill: white; margin-bottom: 24px; opacity: 0.98; filter: drop-shadow(0 6px 12px rgba(0,0,0,0.35)); }

        /* Typography: use Merriweather for headings, Georgia for body copy */
        h1 { font-family: 'Merriweather', Georgia, serif; font-size: 3.5rem; font-weight: 700; margin: 0; letter-spacing: -0.02em; }
        p { font-family: Georgia, 'Merriweather', serif; font-size: 1.25rem; opacity: 0.9; margin-top: 1rem; max-width: 560px; line-height: 1.6; font-weight: 400; }

        /* Story Insights - Real-world context callouts */
        .story-insights {
            display: flex; flex-direction: column; gap: 10px; margin-top: 2.2rem;
        }

        .insight-item {
            display: grid; grid-template-columns: 40px 1fr;
            align-items: center; gap: 14px;
            padding: 14px 18px; border-radius: 14px;
            background: linear-gradient(135deg, rgba(255,255,255,0.08) 0%, rgba(255,255,255,0.04) 100%);
            backdrop-filter: blur(20px);
            border: 1px solid rgba(255,255,255,0.15);
            font-size: 0.95rem; color: white; font-family: Georgia, 'Merriweather', serif;
            transition: all 350ms cubic-bezier(.22,.9,.32,1), background 350ms ease, border-color 350ms ease;
            animation: insightSlideIn 600ms cubic-bezier(.22,.9,.32,1) backwards;
            position: relative;
            overflow: hidden;
        }

        .insight-item::before {
            content: '';
            position: absolute;
            left: 0; top: 0; bottom: 0;
            width: 3px;
            background: rgba(255,255,255,0.3);
            transition: background 350ms ease;
        }

        .insight-item:hover {
            background: linear-gradient(135deg, rgba(255,255,255,0.14) 0%, rgba(255,255,255,0.08) 100%);
            border-color: rgba(255,255,255,0.25);
        }

        .insight-item:hover::before {
            background: rgba(255,255,255,0.5);
        }

        .insight-item:nth-child(1) { animation-delay: 100ms; }
        .insight-item:nth-child(2) { animation-delay: 200ms; }

        @keyframes insightSlideIn {
            from {
                opacity: 0;
                transform: translateX(-16px);
            }
            to {
                opacity: 1;
                transform: translateX(0);
            }
        }

        .insight-icon {
            font-size: 1.6rem; display: flex; align-items: center; justify-content: center; width: 40px; height: 40px;
            flex-shrink: 0; border-radius: 10px;
            background: rgba(255,255,255,0.08);
        }

        .insight-text {
            font-size: 0.9rem; opacity: 0.95; line-height: 1.4; font-weight: 400;
        }

        /* Entrance animation for narrative text */
        @keyframes fadeUp { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
        .text-entrance { animation: fadeUp 560ms cubic-bezier(.22,.9,.32,1) both; }
        .scene-subtle { transition: opacity 420ms ease, transform 420ms ease; }

        /* Respect user preference for reduced motion */
        @media (prefers-reduced-motion: reduce) {
            .text-entrance { animation: none; }
            * { transition: none !important; }
        }

        /* --- RIGHT PANEL (DATA CONSOLE) --- */
        .stage-right {
            flex: 0 0 45%; min-width: 300px; height: 100%;
            position: relative; z-index: 10;
            padding: 1.8rem; box-sizing: border-box; /* slightly increased padding for balance */
            display: flex; flex-direction: column; justify-content: center;
        }

        .glass-panel {
            background: var(--glass-bg);
            backdrop-filter: blur(30px);
            -webkit-backdrop-filter: blur(30px);
            border: 1px solid var(--glass-border);
            border-radius: 24px;
            box-shadow: 0 30px 80px rgba(0,0,0,0.35);
            height: auto; width: 100%;
            /* cap height to viewport so content doesn't get clipped on smaller displays */
            max-height: calc(100vh - 64px);
            display: flex; flex-direction: column;
            overflow: hidden;
            transform: translateY(0);
            opacity: 0.98;
            transition: transform 420ms cubic-bezier(.22,.9,.32,1), box-shadow 420ms ease;
        }

        /* 1. Header */
        .panel-header {
            padding: 1.5rem 2rem;
            display: flex; justify-content: space-between; align-items: center;
            border-bottom: 1px solid rgba(0,0,0,0.05);
            flex-shrink: 0;
        }
        .header-left h2 { margin: 0; font-size: 0.75rem; text-transform: uppercase; color: #64748b; letter-spacing: 1px; font-weight: 700; }
        .big-clock { font-size: 2rem; font-weight: 700; color: var(--text-dark); font-variant-numeric: tabular-nums; margin-top: 2px; }
        
        .total-box { text-align: right; }
        .total-val { font-family: 'JetBrains Mono'; font-weight: 700; font-size: clamp(0.95rem, 1.6vw, 1.2rem); color: var(--text-dark); white-space: nowrap; overflow: visible; }
    .total-lbl { display: block; font-size: 0.7rem; color: #64748b; font-weight: 700; font-family: 'Merriweather', Georgia, serif; }

        /* 2. Main Content (Split Grid) */
        .console-body {
            display: grid;
            grid-template-columns: 3fr 1fr; /* left column ~75% (3fr / 4fr) */
            flex-grow: 1;
            overflow: hidden;
            min-height: 0; /* allow children to shrink properly inside flex container */
        }

        /* Graph Cell */
        .chart-cell {
            position: relative;
            padding: 1.2rem; /* slightly more breathing room */
            border-right: 1px solid rgba(0,0,0,0.05);
            display: flex; flex-direction: column;
        }
        .chart-wrapper {
            flex-grow: 1;
            position: relative;
            width: 100%; height: 100%;
            min-height: 0; /* Important for flex/grid nesting */
        }

        /* Ensure the canvas is fully responsive and never clipped */
        #mainChart {
            width: 100% !important;
            height: 100% !important;
            display: block;
        }

        /* List Cell */
        .list-cell {
            padding: 1.8rem 1.2rem;
            overflow-y: auto;
            background: rgba(255,255,255,0.4);
            display: flex; flex-direction: column; gap: 12px;
        }

        /* Fuel Item */
        .fuel-item {
            display: flex; flex-direction: column; gap: 4px; transform-origin: center; will-change: transform;
            transition: transform 300ms ease-out;
            transform: scale(1);
        }
        
        .fuel-item.dominant {
            transform: scale(1.08);
        }
        .fuel-meta {
            display: flex; justify-content: space-between; align-items: center;
            font-size: 0.75rem; font-weight: 600;
        }
    .fuel-name { display: flex; align-items: center; gap: 6px; color: #475569; font-family: 'Merriweather', Georgia, serif; }
        .fuel-dot { width: 8px; height: 8px; border-radius: 50%; }
        .fuel-val { font-family: 'JetBrains Mono'; color: var(--text-dark); font-weight: 600; }
        
        .fuel-track { height: 4px; background: rgba(0,0,0,0.05); border-radius: 2px; width: 100%; overflow: visible; transform-origin: left; }
        .fuel-bar { height: 100%; width: 0%; border-radius: 2px; transition: width 0.1s linear; transform-origin: left; will-change: transform; }

        /* 3. Slider Footer */
        .control-dock {
            padding: 1.5rem 2rem;
            background: #fff;
            border-top: 1px solid rgba(0,0,0,0.05);
            flex-shrink: 0;
            transition: background 300ms ease, transform 300ms ease;
        }
        
        input[type=range] {
            width: 100%; appearance: none; -webkit-appearance: none; background: transparent; cursor: pointer;
        }

        input[type=range]:focus { outline: none; box-shadow: 0 0 0 6px rgba(99,102,241,0.08); border-radius: 999px; }

        /* WebKit (Chrome, Safari, Edge) */
        input[type=range]::-webkit-slider-runnable-track {
            height: 6px; background: #000; border-radius: 3px;
        }
        input[type=range]::-webkit-slider-thumb {
            -webkit-appearance: none; height: 24px; width: 24px;
            background: #000; border: 3px solid #fff; border-radius: 50%;
            margin-top: -9px; box-shadow: 0 4px 10px rgba(0,0,0,0.2);
        }

        /* Firefox */
        input[type=range]::-moz-range-track {
            height: 6px; background: #000; border-radius: 3px;
        }
        input[type=range]::-moz-range-thumb {
            height: 24px; width: 24px; background: #000; border: 3px solid #fff; border-radius: 50%;
            box-shadow: 0 4px 10px rgba(0,0,0,0.2);
        }

        /* IE / Edge Legacy */
        input[type=range]::-ms-track {
            height: 6px; background: transparent; border-color: transparent; color: transparent;
        }
        input[type=range]::-ms-fill-lower, input[type=range]::-ms-fill-upper {
            background: #000; border-radius: 3px;
        }
        input[type=range]::-ms-thumb {
            height: 24px; width: 24px; background: #000; border: 3px solid #fff; border-radius: 50%;
            margin-top: 0;
        }

        /* Responsive stacking for smaller screens */
        @media (max-width: 900px) {
            .stage-left { flex: 1 1 100%; padding: calc(var(--page-padding) * 1.2); }
            .stage-right { display: block; position: relative; flex: 0 0 100%; padding: 1rem; }
            body { flex-direction: column; }
            .console-body { grid-template-columns: 1fr; }
            .orb-container { left: 50%; top: 8%; transform: translateX(-50%); }
            :root { --orb-size: 84px; }
            h1 { font-size: 2.4rem; }
            .glass-panel { max-height: none; }
        }

        @media (max-width: 480px) {
            :root { --orb-size: 64px; }
            h1 { font-size: 1.6rem; }
            p { font-size: 1rem; max-width: 100%; }
            .panel-header { padding: 1rem; }
            .control-dock { padding: 1rem; }
        }

        /* Energy Mesh Background */
        #energy-mesh {
            position: absolute; inset: 0; z-index: 0;
            width: 100%; height: 100%;
            opacity: 0.15;
            mix-blend-mode: screen;
        }

        .mesh-line {
            stroke-width: 1.5;
            stroke-linecap: round;
            opacity: 0.8;
            transition: stroke 600ms ease-in-out;
        }

        .zoom-hint {
            position: absolute; top: 10px; right: 10px;
            font-size: 0.65rem; color: #94a3b8; font-weight: 600;
            background: rgba(255, 255, 255, 0.527); padding: 2px 6px; border-radius: 4px;
            pointer-events: none; z-index: 20;
        }

    </style>
</head>
<body>

    <div id="sky-backdrop"></div>

    <svg id="energy-mesh" viewBox="0 0 1000 1000" preserveAspectRatio="none">
        <!-- Dynamic mesh lines will be injected by JavaScript -->
    </svg>

    <div class="stage-left">
        <div class="orb-container">
            <div id="sun" class="orb" style="top:85%; opacity:0"></div>
            <div id="moon" class="orb" style="top:85%; opacity:0; transform: translateX(-50%) scale(0.9);"></div>
        </div>
        
        <div class="narrative-box">
            <div class="icon-box" id="scene-icon"></div>
            <h1 id="scene-title">Initializing...</h1>
            <p id="scene-desc">Loading grid telemetry.</p>
            <div class="story-insights">
                <div class="insight-item" id="insight-1">
                    <span class="insight-icon">⚡</span>
                    <span class="insight-text" id="insight-text-1">Grid initializing...</span>
                </div>
                <div class="insight-item" id="insight-2">
                    <span class="insight-icon">🌍</span>
                    <span class="insight-text" id="insight-text-2">Loading data...</span>
                </div>
            </div>
        </div>
    </div>

    <div class="stage-right">
        <div class="glass-panel">
            
            <div class="panel-header">
                <div class="header-left">
                    <h2>Time of Day</h2>
                    <div class="big-clock" id="clock-display">--:--</div>
                </div>
                <div class="total-box">
                    <span class="total-val" id="total-val">0 MW</span>
                    <span class="total-lbl">NET DEMAND</span>
                </div>
            </div>

            <div class="console-body">
                
                <div class="chart-cell">
                    <div class="zoom-hint">SCROLL TO ZOOM</div>
                    <div class="chart-wrapper">
                        <canvas id="mainChart"></canvas>
                    </div>
                </div>

                <div class="list-cell" id="fuel-list">
                    </div>

            </div>

                <div class="control-dock">
                <input type="range" min="0" max="287" value="0" id="scrubber" aria-label="Time scrubber" />
            </div>

        </div>
    </div>

    <div style="display:none">
        <svg id="svg-moon" viewBox="0 0 24 24"><path d="M12 3a7.5 7.5 0 0 0 7.92 12.446a9 9 0 1 1 -8.313 -12.454z"></path></svg>
        <svg id="svg-coffee" viewBox="0 0 24 24"><path d="M18 8h1a4 4 0 0 1 0 8h-1"></path><path d="M2 8h16v9a4 4 0 0 1 -4 4h-8a4 4 0 0 1 -4 -4z"></path><path d="M6 1v3"></path><path d="M10 1v3"></path><path d="M14 1v3"></path></svg>
        <svg id="svg-sun" viewBox="0 0 24 24"><circle cx="12" cy="12" r="4"></circle><path d="M3 12h1m8 -9v1m8 8h1m-9 8v1m-6.4 -15.4l.7 .7m12.1 -.7l-.7 .7m0 11.4l.7 .7m-12.1 -.7l-.7 .7"></path></svg>
        <svg id="svg-home" viewBox="0 0 24 24"><path d="M5 12l-2 0l9 -9l9 9l-2 0"></path><path d="M5 12v7a2 2 0 0 0 2 2h10a2 2 0 0 0 2 -2v-7"></path><path d="M9 21v-6a2 2 0 0 1 2 -2h2a2 2 0 0 1 2 2v6"></path></svg>
        <svg id="svg-bed" viewBox="0 0 24 24"><path d="M2 21h18"></path><path d="M4 11v10"></path><path d="M20 11v10"></path><rect x="4" y="11" width="16" height="6"></rect><path d="M6 7h4v4h-4z"></path></svg>
    </div>

<script>
    // --- 1. DATA (Jan 5 2026 - 5 Min Intervals) ---
    const gridData = {
        time: ['00:00', '00:05', '00:10', '00:15', '00:20', '00:25', '00:30', '00:35', '00:40', '00:45', '00:50', '00:55', '01:00', '01:05', '01:10', '01:15', '01:20', '01:25', '01:30', '01:35', '01:40', '01:45', '01:50', '01:55', '02:00', '02:05', '02:10', '02:15', '02:20', '02:25', '02:30', '02:35', '02:40', '02:45', '02:50', '02:55', '03:00', '03:05', '03:10', '03:15', '03:20', '03:25', '03:30', '03:35', '03:40', '03:45', '03:50', '03:55', '04:00', '04:05', '04:10', '04:15', '04:20', '04:25', '04:30', '04:35', '04:40', '04:45', '04:50', '04:55', '05:00', '05:05', '05:10', '05:15', '05:20', '05:25', '05:30', '05:35', '05:40', '05:45', '05:50', '05:55', '06:00', '06:05', '06:10', '06:15', '06:20', '06:25', '06:30', '06:35', '06:40', '06:45', '06:50', '06:55', '07:00', '07:05', '07:10', '07:15', '07:20', '07:25', '07:30', '07:35', '07:40', '07:45', '07:50', '07:55', '08:00', '08:05', '08:10', '08:15', '08:20', '08:25', '08:30', '08:35', '08:40', '08:45', '08:50', '08:55', '09:00', '09:05', '09:10', '09:15', '09:20', '09:25', '09:30', '09:35', '09:40', '09:45', '09:50', '09:55', '10:00', '10:05', '10:10', '10:15', '10:20', '10:25', '10:30', '10:35', '10:40', '10:45', '10:50', '10:55', '11:00', '11:05', '11:10', '11:15', '11:20', '11:25', '11:30', '11:35', '11:40', '11:45', '11:50', '11:55', '12:00', '12:05', '12:10', '12:15', '12:20', '12:25', '12:30', '12:35', '12:40', '12:45', '12:50', '12:55', '13:00', '13:05', '13:10', '13:15', '13:20', '13:25', '13:30', '13:35', '13:40', '13:45', '13:50', '13:55', '14:00', '14:05', '14:10', '14:15', '14:20', '14:25', '14:30', '14:35', '14:40', '14:45', '14:50', '14:55', '15:00', '15:05', '15:10', '15:15', '15:20', '15:25', '15:30', '15:35', '15:40', '15:45', '15:50', '15:55', '16:00', '16:05', '16:10', '16:15', '16:20', '16:25', '16:30', '16:35', '16:40', '16:45', '16:50', '16:55', '17:00', '17:05', '17:10', '17:15', '17:20', '17:25', '17:30', '17:35', '17:40', '17:45', '17:50', '17:55', '18:00', '18:05', '18:10', '18:15', '18:20', '18:25', '18:30', '18:35', '18:40', '18:45', '18:50', '18:55', '19:00', '19:05', '19:10', '19:15', '19:20', '19:25', '19:30', '19:35', '19:40', '19:45', '19:50', '19:55', '20:00', '20:05', '20:10', '20:15', '20:20', '20:25', '20:30', '20:35', '20:40', '20:45', '20:50', '20:55', '21:00', '21:05', '21:10', '21:15', '21:20', '21:25', '21:30', '21:35', '21:40', '21:45', '21:50', '21:55', '22:00', '22:05', '22:10', '22:15', '22:20', '22:25', '22:30', '22:35', '22:40', '22:45', '22:50', '22:55', '23:00', '23:05', '23:10', '23:15', '23:20', '23:25', '23:30', '23:35', '23:40', '23:45', '23:50', '23:55'],
        solar: [-57, -57, -58, -58, -58, -58, -58, -59, -58, -58, -59, -58, -58, -59, -59, -59, -59, -58, -58, -59, -58, -58, -58, -59, -59, -59, -59, -59, -59, -59, -59, -60, -59, -59, -59, -59, -59, -59, -59, -59, -59, -59, -59, -59, -59, -59, -59, -59, -59, -59, -59, -59, -59, -59, -59, -59, -59, -59, -59, -59, -59, -59, -59, -59, -59, -59, -59, -59, -59, -59, -59, -59, -59, -58, -58, -58, -58, -58, -58, -57, -54, -43, -20, 20, 102, 233, 478, 735, 1212, 1551, 1935, 2576, 3122, 3845, 4641, 5536, 6842, 7740, 8345, 8866, 9809, 10571, 11198, 11875, 12390, 12724, 12894, 13060, 13458, 13666, 13541, 13582, 13919, 13913, 13881, 14305, 14483, 14513, 14580, 14646, 14565, 14624, 14526, 14305, 13883, 14027, 14142, 14074, 13839, 13445, 13588, 13401, 13299, 13214, 13385, 13131, 13132, 13076, 13122, 13261, 13334, 13598, 13302, 13016, 12900, 12978, 12980, 12979, 13130, 13336, 13208, 13222, 13253, 12996, 12815, 12659, 12589, 12566, 12440, 12275, 11609, 11366, 11182, 10772, 10679, 10709, 10664, 10344, 9831, 9719, 9762, 9875, 9690, 9222, 9022, 8945, 8611, 8312, 8267, 7973, 7685, 7320, 6743, 6199, 5537, 5119, 4383, 3713, 3260, 3020, 2725, 2188, 1776, 1369, 1079, 766, 532, 393, 264, 128, 33, -13, -39, -51, -53, -53, -51, -48, -45, -43, -42, -41, -39, -38, -38, -37, -37, -38, -37, -37, -37, -37, -37, -36, -36, -36, -35, -35, -37, -37, -38, -37, -37, -37, -37, -37, -36, -37, -37, -38, -36, -36, -37, -37, -36, -37, -37, -37, -37, -37, -37, -37, -36, -35, -35, -35, -35, -35, -35, -35, -35, -35, -35, -35, -35, -35, -35, -35, -35, -36, -36, -36, -35, -35, -35, -35, -35, -35, -35, -35, -35, -36, -36, -36, -36, -36],
        wind: [3548, 3562, 3550, 3555, 3536, 3458, 3397, 3362, 3329, 3301, 3288, 3309, 3302, 3333, 3370, 3391, 3398, 3354, 3306, 3299, 3283, 3290, 3303, 3302, 3302, 3296, 3275, 3250, 3214, 3198, 3190, 3204, 3205, 3186, 3192, 3183, 3188, 3175, 3173, 3203, 3224, 3241, 3229, 3243, 3268, 3299, 3313, 3331, 3322, 3289, 3283, 3325, 3374, 3431, 3469, 3487, 3498, 3507, 3516, 3489, 3467, 3528, 3527, 3512, 3467, 3429, 3409, 3421, 3405, 3421, 3439, 3373, 3303, 3204, 3133, 3090, 3067, 3057, 3044, 3019, 3012, 2949, 2964, 2871, 2795, 2793, 2775, 2762, 2752, 2768, 2798, 2797, 2788, 2725, 2687, 2646, 2591, 2505, 2459, 2450, 2443, 2431, 2438, 2441, 2434, 2441, 2433, 2426, 2340, 2288, 2321, 2399, 2374, 2369, 2422, 2402, 2391, 2392, 2317, 2302, 2299, 2404, 2426, 2422, 2419, 2422, 2423, 2415, 2419, 2436, 2462, 2473, 2497, 2519, 2525, 2518, 2514, 2515, 2508, 2522, 2552, 2554, 2538, 2557, 2615, 2620, 2642, 2637, 2634, 2624, 2660, 2646, 2653, 2632, 2614, 2649, 2731, 2746, 2757, 2764, 2761, 2724, 2698, 2717, 2714, 2743, 2770, 2741, 2720, 2735, 2721, 2739, 2765, 2703, 2667, 2618, 2579, 2576, 2577, 2580, 2574, 2660, 2718, 2727, 2694, 2639, 2599, 2583, 2555, 2546, 2550, 2594, 2619, 2631, 2614, 2573, 2551, 2534, 2526, 2512, 2470, 2404, 2355, 2313, 2277, 2213, 2195, 2178, 2176, 2173, 2151, 2154, 2141, 2102, 2072, 2052, 2030, 2036, 2041, 2058, 2043, 2032, 2044, 2038, 2046, 2076, 2103, 2138, 2139, 2141, 2164, 2204, 2198, 2182, 2196, 2208, 2229, 2249, 2257, 2283, 2278, 2294, 2307, 2323, 2344, 2355, 2368, 2378, 2360, 2352, 2356, 2368, 2378, 2404, 2421, 2423, 2433, 2445, 2435, 2426, 2429, 2433, 2441, 2418, 2390, 2351, 2320, 2278, 2252, 2243, 2224, 2225, 2252, 2280, 2303, 2315, 2340, 2318, 2287, 2229, 2205, 2184, 2159, 2147, 2134, 2107, 2107, 2134],
        geo: [769, 769, 770, 770, 769, 770, 771, 770, 770, 770, 770, 770, 770, 770, 772, 772, 772, 772, 772, 773, 772, 772, 772, 771, 771, 771, 771, 771, 771, 771, 771, 771, 770, 771, 771, 771, 771, 771, 771, 772, 772, 772, 772, 772, 772, 771, 772, 771, 771, 771, 771, 771, 771, 771, 770, 770, 771, 771, 771, 770, 771, 769, 770, 770, 770, 768, 768, 769, 770, 769, 769, 770, 771, 780, 787, 787, 784, 779, 774, 777, 787, 786, 786, 774, 776, 779, 781, 783, 774, 775, 776, 775, 777, 775, 771, 770, 766, 766, 765, 764, 763, 763, 763, 762, 761, 758, 756, 745, 743, 745, 754, 754, 755, 757, 754, 760, 755, 753, 752, 753, 753, 753, 753, 762, 776, 779, 765, 760, 759, 759, 762, 763, 764, 765, 766, 766, 766, 766, 767, 770, 767, 767, 765, 766, 766, 766, 765, 765, 766, 764, 764, 763, 765, 765, 765, 764, 763, 763, 764, 763, 763, 764, 763, 763, 764, 763, 763, 762, 761, 761, 761, 760, 759, 760, 758, 759, 758, 758, 758, 758, 757, 757, 758, 697, 700, 702, 704, 702, 703, 704, 704, 705, 702, 698, 699, 699, 699, 700, 699, 699, 699, 699, 699, 699, 700, 706, 707, 707, 706, 707, 708, 708, 708, 708, 707, 708, 708, 707, 706, 706, 707, 707, 707, 707, 707, 705, 702, 703, 700, 704, 706, 706, 705, 705, 705, 705, 706, 706, 705, 707, 706, 707, 707, 708, 709, 709, 710, 709, 709, 710, 709, 710, 710, 707, 706, 705, 705, 705, 705, 704, 704, 704, 704, 704, 708, 744, 760, 755, 762, 767, 772, 776, 775, 776, 776, 772, 772, 770, 768, 768, 768, 768, 768, 768, 768, 766, 764, 764],
        bio: [363, 363, 363, 363, 363, 362, 362, 362, 362, 361, 363, 362, 363, 364, 364, 363, 363, 362, 363, 364, 363, 362, 362, 361, 362, 363, 362, 363, 363, 363, 363, 371, 370, 372, 374, 375, 378, 383, 386, 386, 386, 387, 387, 388, 386, 385, 384, 385, 386, 386, 384, 384, 384, 385, 385, 389, 388, 387, 390, 389, 387, 385, 382, 385, 386, 384, 384, 385, 384, 382, 381, 384, 384, 386, 385, 386, 387, 387, 386, 388, 386, 386, 385, 385, 385, 386, 381, 383, 384, 384, 383, 383, 382, 380, 379, 381, 383, 381, 379, 372, 361, 362, 361, 362, 361, 364, 362, 364, 362, 357, 359, 362, 364, 369, 367, 369, 371, 372, 374, 375, 377, 378, 378, 380, 379, 380, 380, 381, 382, 384, 384, 384, 386, 382, 382, 382, 380, 378, 377, 379, 378, 378, 379, 378, 378, 378, 379, 380, 378, 386, 388, 378, 370, 364, 355, 356, 358, 356, 355, 350, 354, 354, 355, 353, 354, 354, 351, 350, 353, 352, 351, 350, 350, 349, 354, 356, 355, 355, 355, 355, 357, 356, 357, 356, 356, 354, 354, 350, 355, 352, 353, 356, 353, 353, 351, 351, 353, 355, 354, 352, 354, 350, 351, 350, 350, 351, 350, 355, 355, 357, 353, 353, 353, 351, 351, 349, 341, 336, 335, 334, 334, 333, 334, 336, 337, 335, 335, 335, 335, 338, 337, 335, 335, 334, 335, 335, 335, 336, 336, 336, 337, 336, 337, 337, 337, 336, 338, 336, 335, 335, 334, 335, 336, 337, 336, 335, 333, 336, 335, 336, 334, 335, 333, 332, 332, 330, 330, 329, 330, 329, 328, 326, 328, 327, 326, 328, 329, 327, 327, 327, 327, 328, 330, 329, 328, 329, 331, 330],
        hydro: [3026, 3049, 3040, 3062, 3082, 3033, 2986, 2976, 2957, 2918, 2932, 2940, 2948, 3051, 3063, 3077, 3087, 3001, 2941, 2904, 2887, 2925, 2923, 2920, 2916, 2938, 3009, 3039, 3044, 2968, 2893, 2903, 2908, 2899, 2906, 2895, 2884, 2935, 2967, 2957, 2951, 2851, 2817, 2810, 2808, 2853, 2823, 2860, 2958, 3129, 3197, 3192, 3178, 3139, 3084, 3083, 3086, 3065, 3076, 3099, 3099, 3184, 3227, 3210, 3238, 3182, 3130, 3118, 3088, 3073, 3092, 2959, 2505, 2396, 2448, 2502, 2517, 2511, 2512, 2525, 2491, 2511, 2519, 2527, 2484, 2382, 2334, 2369, 2379, 2374, 2373, 2358, 2362, 2353, 2286, 2289, 2294, 2453, 2500, 2508, 2514, 2492, 2453, 2470, 2443, 2428, 2410, 2373, 2390, 2405, 2428, 2420, 2328, 2029, 2018, 2035, 2039, 2017, 1984, 1776, 1671, 1633, 1598, 1576, 1582, 1499, 1547, 1551, 1526, 1556, 1563, 1559, 1567, 1527, 1519, 1526, 1529, 1542, 1520, 1234, 1245, 1242, 1234, 1243, 1218, 1229, 1255, 1252, 1265, 1264, 1255, 1267, 1270, 1281, 1243, 1206, 1244, 1472, 1566, 1484, 1853, 1858, 1783, 1678, 1620, 1632, 1704, 1793, 1846, 1823, 1868, 1905, 1917, 1919, 1853, 1994, 2042, 2096, 2066, 2045, 2094, 2318, 2457, 2541, 2607, 2665, 2787, 2851, 2880, 2906, 2889, 2889, 2991, 3200, 3389, 3395, 3412, 3464, 3470, 3464, 3456, 3434, 3459, 3468, 3463, 3451, 3414, 3433, 3398, 3343, 3348, 3341, 3304, 3330, 3342, 3340, 3329, 3240, 3253, 3268, 3260, 3266, 3239, 3208, 3237, 3236, 3227, 3192, 3167, 3182, 3193, 3184, 3160, 3125, 3121, 3146, 3162, 3159, 3109, 3109, 3136, 3191, 3206, 3212, 3195, 3186, 3225, 3209, 3213, 3196, 3189, 3207, 3227, 3291, 3299, 3293, 3308, 3298, 3295, 3287, 3226, 3157, 3158, 3150, 3094, 2992, 2973, 3045, 3056, 3053, 3042, 3029, 3044, 3050, 3055, 3051, 3028, 3059, 3075, 3068, 3063, 3068, 3088, 3103, 3102, 3096, 3098, 3060],
        nuclear: [2255, 2255, 2254, 2256, 2256, 2255, 2256, 2256, 2256, 2257, 2256, 2257, 2256, 2256, 2256, 2255, 2256, 2257, 2256, 2257, 2258, 2257, 2256, 2255, 2255, 2255, 2255, 2255, 2256, 2256, 2255, 2255, 2255, 2256, 2256, 2256, 2257, 2256, 2255, 2257, 2257, 2256, 2257, 2256, 2256, 2257, 2256, 2256, 2257, 2257, 2257, 2257, 2257, 2258, 2257, 2257, 2258, 2257, 2258, 2258, 2259, 2258, 2258, 2259, 2260, 2259, 2260, 2259, 2259, 2260, 2260, 2261, 2262, 2262, 2261, 2260, 2260, 2260, 2261, 2260, 2260, 2260, 2261, 2259, 2260, 2260, 2259, 2259, 2258, 2260, 2260, 2259, 2260, 2260, 2260, 2261, 2260, 2260, 2260, 2261, 2260, 2261, 2261, 2260, 2261, 2260, 2262, 2260, 2261, 2261, 2263, 2262, 2261, 2261, 2262, 2261, 2261, 2262, 2261, 2261, 2260, 2259, 2259, 2259, 2260, 2259, 2259, 2260, 2260, 2259, 2260, 2259, 2260, 2260, 2260, 2259, 2259, 2260, 2259, 2259, 2259, 2258, 2258, 2259, 2260, 2259, 2260, 2259, 2259, 2259, 2259, 2260, 2259, 2258, 2258, 2260, 2258, 2258, 2258, 2258, 2258, 2259, 2258, 2259, 2259, 2259, 2260, 2260, 2259, 2259, 2258, 2258, 2258, 2258, 2258, 2258, 2257, 2258, 2258, 2258, 2257, 2257, 2257, 2256, 2256, 2256, 2256, 2256, 2255, 2255, 2256, 2256, 2256, 2254, 2254, 2256, 2254, 2256, 2255, 2255, 2256, 2255, 2255, 2255, 2255, 2256, 2255, 2256, 2255, 2256, 2256, 2256, 2257, 2255, 2256, 2256, 2256, 2256, 2256, 2256, 2256, 2256, 2256, 2255, 2254, 2254, 2255, 2254, 2255, 2254, 2254, 2255, 2255, 2256, 2255, 2256, 2256, 2256, 2255, 2256, 2256, 2256, 2256, 2255, 2257, 2257, 2257, 2257, 2257, 2257, 2257, 2258, 2258, 2257, 2259, 2258, 2257, 2259, 2258, 2259, 2259, 2259, 2259, 2258, 2258, 2258, 2258, 2257, 2257, 2258, 2259, 2257, 2257, 2257, 2257, 2258, 2258, 2258, 2259, 2258, 2258, 2258, 2258, 2258, 2257, 2258, 2257, 2258],
        gas: [4219, 4039, 3845, 3722, 3690, 3684, 3679, 3676, 3719, 3757, 3775, 3770, 3761, 3759, 3769, 3751, 3698, 3666, 3673, 3707, 3728, 3727, 3728, 3724, 3726, 3775, 3801, 3806, 3808, 3810, 3811, 3808, 3809, 3810, 3809, 3808, 3810, 3809, 3810, 3803, 3809, 3810, 3823, 3833, 3848, 3861, 3857, 3843, 3829, 3770, 3706, 3680, 3664, 3661, 3660, 3662, 3663, 3666, 3670, 3669, 3663, 3664, 3662, 3663, 3667, 3673, 3673, 3706, 3731, 3812, 3861, 3912, 3906, 3895, 3904, 3903, 3904, 3916, 3915, 3932, 3939, 3988, 4055, 4117, 4134, 4126, 4102, 4115, 4071, 4040, 3954, 3716, 3547, 3484, 3422, 3333, 3279, 3211, 3119, 3020, 2963, 2955, 2942, 2942, 2943, 2939, 2929, 2932, 2873, 2699, 2637, 2609, 2500, 2478, 2475, 2476, 2477, 2482, 2482, 2495, 2499, 2475, 2457, 2445, 2443, 2441, 2437, 2436, 2433, 2431, 2426, 2365, 2350, 2347, 2352, 2359, 2357, 2369, 2379, 2389, 2398, 2428, 2492, 2559, 2642, 2683, 2722, 2662, 2586, 2541, 2527, 2520, 2521, 2519, 2521, 2524, 2524, 2520, 2522, 2523, 2514, 2516, 2534, 2569, 2612, 2656, 2691, 2686, 2685, 2695, 2713, 2758, 2794, 2873, 2987, 3049, 3108, 3210, 3260, 3295, 3321, 3338, 3370, 3471, 3628, 3708, 3738, 3825, 4104, 4315, 4494, 4637, 4703, 4535, 4384, 4337, 4454, 4604, 4758, 4903, 5035, 5118, 5176, 5251, 5238, 5121, 5177, 5317, 5420, 5454, 5470, 5508, 5552, 5567, 5574, 5574, 5447, 5183, 5013, 4982, 4956, 4949, 4949, 4944, 4929, 4910, 4881, 4913, 4983, 5117, 5157, 5141, 5135, 5137, 5095, 5090, 5059, 5035, 5008, 5003, 4999, 4902, 4893, 4897, 4885, 4886, 4890, 4903, 4909, 4902, 4847, 4809, 4779, 4845, 4867, 4859, 4802, 4783, 4761, 4727, 4693, 4691, 4688, 4689, 4706, 4781, 4786, 4765, 4784, 4779, 4777, 4762, 4735, 4737, 4732, 4678, 4705, 4790, 4796, 4712, 4656, 4624, 4654, 4623, 4615, 4611, 4616, 4655],
        batteries: [-615, -791, -929, -1109, -1142, -1076, -1096, -1094, -1166, -1263, -1347, -1431, -1427, -1267, -1309, -1342, -1365, -1311, -1318, -1399, -1422, -1540, -1767, -1829, -1999, -2013, -2117, -2297, -2281, -2177, -2164, -2134, -2164, -2204, -2278, -2343, -2295, -2359, -2399, -2414, -2402, -2329, -2410, -2521, -2494, -2510, -2534, -2437, -2356, -2126, -2048, -2025, -2006, -1862, -1798, -1695, -1642, -1615, -1509, -1357, -1274, -1507, -1462, -1516, -1384, -1129, -974, -698, -394, -242, -180, 126, 974, 1934, 2816, 3397, 3589, 3655, 3764, 3800, 3909, 3879, 3719, 3651, 4035, 4480, 4870, 4632, 4190, 3719, 3430, 3233, 2803, 2231, 1805, 1258, 804, -352, -1076, -1740, -2226, -2772, -3522, -3879, -4440, -5157, -5334, -5433, -5616, -6377, -6815, -6711, -6421, -6157, -6472, -6556, -6894, -7264, -7035, -6797, -6580, -6493, -6423, -6309, -6037, -5845, -6131, -6645, -6791, -6595, -6288, -6452, -6533, -6573, -6528, -6594, -6292, -5955, -5848, -5894, -5888, -5802, -5900, -5517, -5222, -5658, -5581, -5477, -5051, -4771, -4776, -4696, -4421, -4486, -3982, -4054, -3914, -4234, -3965, -3898, -3907, -3374, -2775, -2436, -2101, -2235, -2549, -2815, -2502, -2369, -2653, -2631, -2480, -1860, -1325, -1156, -1180, -1045, -791, -734, -731, -1002, -1248, -832, -461, -218, -92, 26, 307, 884, 1354, 1669, 1990, 1926, 1965, 2409, 2734, 3094, 3367, 3651, 3847, 4141, 4569, 5039, 5327, 5627, 5766, 5863, 6064, 6240, 6331, 6379, 6372, 6370, 6367, 6329, 6351, 6225, 6307, 6336, 6302, 6247, 6186, 6229, 6236, 6159, 6173, 6067, 5888, 5917, 5811, 5757, 5661, 5559, 5531, 5490, 5360, 5333, 5326, 5241, 5089, 5018, 4898, 4795, 4714, 4648, 4558, 4544, 4450, 4362, 4293, 4233, 4043, 3711, 3650, 3514, 3486, 3384, 3265, 3254, 3185, 3106, 3068, 2927, 2645, 2389, 2158, 1860, 1615, 1567, 1558, 1518, 1408, 1319, 1172, 1080, 898, 770, 722, 716, 626, 607, 580, 517, 377, 291, 216, 13],
        imports: [8401, 8958, 9340, 9484, 9482, 9497, 9548, 9530, 9536, 9543, 9524, 9524, 9462, 9137, 9041, 8991, 9005, 9022, 9066, 9095, 9080, 9155, 9275, 9308, 9464, 9322, 9364, 9415, 9457, 9412, 9398, 9326, 9342, 9362, 9399, 9397, 9402, 9406, 9423, 9416, 9353, 9313, 9379, 9360, 9288, 9216, 9283, 9263, 9174, 8777, 8800, 8805, 8777, 8680, 8648, 8622, 8623, 8663, 8613, 8630, 8696, 8892, 9053, 9243, 9272, 9212, 9238, 9098, 8953, 8811, 8775, 8714, 8522, 8186, 7698, 7353, 7365, 7419, 7454, 7579, 7706, 7874, 8082, 8240, 8070, 7718, 7477, 7527, 7806, 7864, 7921, 7972, 7923, 7980, 7802, 7601, 7178, 7372, 7321, 7322, 7202, 6705, 6545, 6227, 5987, 6026, 5745, 5598, 5693, 6150, 6211, 6056, 5828, 5511, 5647, 5694, 5534, 5645, 5351, 5187, 4936, 4792, 4658, 4727, 4641, 4897, 5011, 5393, 5530, 5388, 5375, 5385, 5691, 5706, 5829, 5801, 5800, 5428, 5482, 5683, 5528, 5326, 5100, 4885, 4678, 5387, 5207, 5308, 5090, 4729, 4693, 4761, 4526, 4616, 4457, 4741, 4673, 4837, 4504, 4724, 4653, 4874, 4814, 4792, 4909, 5186, 5377, 5596, 5570, 6051, 6551, 6410, 6132, 5751, 5821, 5737, 5847, 5943, 6058, 6020, 6347, 6719, 7173, 7282, 7424, 7670, 7995, 8459, 8673, 8515, 8386, 8335, 8573, 8739, 8942, 8982, 8960, 8834, 8704, 8526, 8550, 8546, 8399, 8102, 7971, 7778, 7668, 7616, 7608, 7569, 7564, 7512, 7547, 7569, 7610, 7675, 7831, 8136, 8182, 8113, 8193, 8194, 8207, 8144, 8125, 8187, 8140, 8270, 8348, 8115, 8156, 8124, 8160, 8220, 8252, 8277, 8363, 8346, 8326, 8427, 8513, 8586, 8639, 8660, 8672, 8666, 8664, 8607, 8631, 8656, 8709, 8721, 8835, 9104, 9113, 9204, 9213, 9297, 9350, 9277, 9328, 9357, 9284, 9327, 9515, 9568, 9683, 9833, 9899, 9843, 9744, 9638, 9593, 9488, 9471, 9486, 9482, 9340, 9294, 9310, 9361, 9261, 9138, 9157, 9182, 9222, 9221, 9257]
    };

    // Helper: 12h Clock
    function to12h(timeStr) {
        if(!timeStr) return "--";
        let [h, m] = timeStr.split(':');
        let hours = parseInt(h);
        const ampm = hours >= 12 ? 'PM' : 'AM';
        hours = hours % 12;
        hours = hours ? hours : 12; 
        return `${hours}:${m} ${ampm}`;
    }

    function clamp(v, a, b) { return Math.max(a, Math.min(b, v)); }

    // Helper: convert hex color to rgba string with alpha
    function hexToRgba(hex, alpha = 1) {
        if (!hex) return `rgba(0,0,0,${alpha})`;
        const h = hex.replace('#','');
        const long = h.length === 6 ? h : h.split('').map(c => c + c).join('');
        const bigint = parseInt(long, 16);
        const r = (bigint >> 16) & 255;
        const g = (bigint >> 8) & 255;
        const b = bigint & 255;
        return `rgba(${r}, ${g}, ${b}, ${alpha})`;
    }

    // Helper: rounded rect for canvas drawing
    function roundRect(ctx, x, y, w, h, r) {
        if (w < 2 * r) r = w / 2;
        if (h < 2 * r) r = h / 2;
        ctx.beginPath();
        ctx.moveTo(x + r, y);
        ctx.arcTo(x + w, y, x + w, y + h, r);
        ctx.arcTo(x + w, y + h, x, y + h, r);
        ctx.arcTo(x, y + h, x, y, r);
        ctx.arcTo(x, y, x + w, y, r);
        ctx.closePath();
    }

    // --- 2. CONFIGURATION ---
    const ctx = document.getElementById('mainChart').getContext('2d');
    
    // Create energy mesh once, cache references to lines for efficient color updates
    let meshLines = [];
    let lastMeshColor = '#ffffff'; // Cache last color to avoid unnecessary updates
    
    function initializeEnergyMesh() {
        const svg = document.getElementById('energy-mesh');
        svg.innerHTML = '';
        meshLines = [];
        
        // Create fewer, more efficient vertical lines (reduced from 12 to 8)
        const verticalLines = 8;
        for (let i = 0; i < verticalLines; i++) {
            const line = document.createElementNS('http://www.w3.org/2000/svg', 'line');
            line.setAttribute('class', 'mesh-line');
            
            const start = (i / verticalLines) * 1000;
            const offset = (i % 2) * 80;
            
            line.setAttribute('x1', start);
            line.setAttribute('y1', '0');
            line.setAttribute('x2', start + offset);
            line.setAttribute('y2', '1000');
            
            svg.appendChild(line);
            meshLines.push(line);
        }
        
        // Create fewer diagonal lines (reduced from 8 to 6)
        const diagonalLines = 6;
        for (let i = 0; i < diagonalLines; i++) {
            const line = document.createElementNS('http://www.w3.org/2000/svg', 'line');
            line.setAttribute('class', 'mesh-line');
            
            const x = (i / diagonalLines) * 1000;
            line.setAttribute('x1', '0');
            line.setAttribute('y1', x);
            line.setAttribute('x2', '1000');
            line.setAttribute('y2', x + 200);
            
            svg.appendChild(line);
            meshLines.push(line);
        }
    }
    
    // Update only the color of existing mesh lines - with caching to skip redundant updates
    function updateMeshColor(dominantColor) {
        if (dominantColor === lastMeshColor) return; // Skip if color unchanged
        lastMeshColor = dominantColor;
        
        meshLines.forEach(line => {
            line.style.stroke = dominantColor || '#ffffff';
        });
    }
    
    // Initialize mesh once on page load
    initializeEnergyMesh();
    updateMeshColor('#ffffff');
    
    // Stack Config
    const stacks = [
        { key: 'nuclear', label: 'Nuclear', color: '#8B5CF6', max: 2500 },
        { key: 'geo', label: 'Geothermal', color: '#d97706', max: 1000 },
        { key: 'bio', label: 'Biomass', color: '#166534', max: 600 },
        { key: 'hydro', label: 'Hydro', color: '#0ea5e9', max: 5000 },
        { key: 'wind', label: 'Wind', color: '#34d399', max: 4000 },
        { key: 'solar', label: 'Solar', color: '#fbbf24', max: 15000 },
        { key: 'batteries', label: 'Batteries', color: '#ec4899', max: 7000 },
        { key: 'imports', label: 'Imports', color: '#06b6d4', max: 10000 },
        { key: 'gas', label: 'Natural Gas', color: '#3b82f6', max: 6000 }
    ];

    // INJECT LIST
    const listContainer = document.getElementById('fuel-list');
    
    [...stacks].reverse().forEach(s => {
        const div = document.createElement('div');
        div.className = 'fuel-item';
        div.id = `item-${s.key}`;
        div.innerHTML = `
            <div class="fuel-meta">
                <div class="fuel-name">
                    <div class="fuel-dot" style="background:${s.color}"></div>
                    <span>${s.label}</span>
                </div>
                <span class="fuel-val" id="val-${s.key}">0</span>
            </div>
            <div class="fuel-track">
                <div class="fuel-bar" id="bar-${s.key}" style="background:${s.color}"></div>
            </div>
        `;
        listContainer.appendChild(div);
    });

    const chart = new Chart(ctx, {
        type: 'line',
        data: {
            labels: gridData.time,
            datasets: stacks.map(s => ({
                label: s.label,
                data: gridData[s.key].map(v => Math.max(0, v)),
                backgroundColor: hexToRgba(s.color, 0.34),
                borderColor: hexToRgba(s.color, 0.92),
                borderWidth: 1,
                fill: true,
                pointRadius: 0,
                tension: 0.32
            }))
        },
        options: {
            responsive: true,
            maintainAspectRatio: false,
            animation: false, 
            interaction: { mode: 'index', intersect: false },
            scales: {
                x: { 
                    grid: { display: false },
                    ticks: { 
                        maxRotation: 0, 
                        font: {family: "'Merriweather', Georgia, serif"}, 
                        color: '#94a3b8',
                        callback: function(val, index) {
                            const label = this.getLabelForValue(val);
                            if(index % 36 === 0) return to12h(label); 
                            return '';
                        }
                    },
                    min: 0, 
                    max: 287
                },
                y: { 
                    stacked: true, 
                    beginAtZero: true,
                    display: true,
                    title: { display: true, text: 'MW', color: '#94a3b8', font: { family: "'Merriweather', Georgia, serif", weight: '700' } },
                    ticks: { color: '#94a3b8', font: { family: "'JetBrains Mono'" } },
                    grid: { color: 'rgba(15,23,42,0.06)' }
                }
            },
            plugins: {
                legend: { display: false },
                tooltip: {
                    enabled: true,
                    backgroundColor: 'rgba(15,23,42,0.95)',
                    titleFont: { family: "'Merriweather', Georgia, serif", weight: '700' },
                    bodyFont: { family: "'JetBrains Mono'" },
                    callbacks: {
                        title: (items) => {
                            if (!items || !items.length) return '';
                            const label = items[0].label || '';
                            return to12h(label);
                        },
                        label: (ctx) => {
                            const v = ctx.raw || 0;
                            return `${ctx.dataset.label}: ${Number(v).toLocaleString()} MW`;
                        }
                    }
                },
                zoom: {
                    pan: { enabled: false },
                    zoom: {
                        wheel: { enabled: true },
                        pinch: { enabled: true },
                        mode: 'x',
                        onZoom: ({chart}) => {
                            const min = chart.scales.x.min;
                            const max = chart.scales.x.max;
                            window.currentZoomSpan = max - min;
                        }
                    },
                    limits: { x: { min: 0, max: 287 } }
                }
            }
        },
        plugins: [{
            id: 'scannerLine',
            afterDraw: (chart) => {
                if (window.activeIndex === undefined) return;
                
                const ctx = chart.ctx;
                const xScale = chart.scales.x;
                const yScale = chart.scales.y;
                const safeX = Math.min(287, Math.max(0, window.activeIndex));
                const activeX = xScale.getPixelForValue(safeX);
                
                if (activeX >= xScale.left && activeX <= xScale.right) {
                    ctx.save();
                    ctx.beginPath();
                    ctx.moveTo(activeX, yScale.top);
                    ctx.lineTo(activeX, yScale.bottom);
                    ctx.lineWidth = 2;
                    ctx.strokeStyle = '#0f172a';
                    ctx.setLineDash([5, 5]);
                    ctx.stroke();
                    ctx.restore();
                }
            }
        }]
    });

    // --- 3. LOGIC ---
    const scrubber = document.getElementById('scrubber');
    const clock = document.getElementById('clock-display');
    const totalLoad = document.getElementById('total-val');
    
    const stage = document.getElementById('sky-backdrop');
    const sunEl = document.getElementById('sun');
    const moonEl = document.getElementById('moon');
    const storyTitle = document.getElementById('scene-title');
    const storyDesc = document.getElementById('scene-desc');
    const storyIcon = document.getElementById('scene-icon');
    const insightText1 = document.getElementById('insight-text-1');
    const insightText2 = document.getElementById('insight-text-2');
    const insightIcon1 = document.querySelector('#insight-1 .insight-icon');
    const insightIcon2 = document.querySelector('#insight-2 .insight-icon');
    const glassPanel = document.querySelector('.glass-panel');
    
    window.currentZoomSpan = 287;
    let lastDominantKey = null; // Cache to skip unnecessary updates

    function updateState(index) {
        const safeIndex = Math.min(287, Math.max(0, index));
        window.activeIndex = safeIndex;
        try { scrubber.setAttribute('aria-valuenow', String(safeIndex)); scrubber.setAttribute('aria-valuetext', to12h(gridData.time[safeIndex])); } catch(e){}
        
        let total = 0;
        let maxVal = 0;
        let dominantKey = null;
        
        stacks.forEach(s => {
            const val = Math.max(0, gridData[s.key][safeIndex] || 0);
            total += val;
            
            document.getElementById(`val-${s.key}`).innerText = val.toLocaleString();
            const pct = Math.min(100, (val / s.max) * 100);
            document.getElementById(`bar-${s.key}`).style.width = `${pct}%`;
            
            if (val > maxVal) {
                maxVal = val;
                dominantKey = s.key;
            }
        });
        
        // Only update animations if dominant fuel changed (avoid redundant DOM operations)
        if (dominantKey !== lastDominantKey) {
            lastDominantKey = dominantKey;
            
            // Use CSS class instead of anime.js for better performance
            stacks.forEach(s => {
                const itemEl = document.getElementById(`item-${s.key}`);
                if (s.key === dominantKey) {
                    itemEl.classList.add('dominant');
                } else {
                    itemEl.classList.remove('dominant');
                }
            });
            
            // Update energy mesh color only when dominant fuel changes
            if (dominantKey) {
                const dominantStack = stacks.find(s => s.key === dominantKey);
                if (dominantStack) {
                    updateMeshColor(dominantStack.color);
                }
            }
        }
        
        totalLoad.innerText = total.toLocaleString() + " MW";
        
        const time24 = gridData.time[safeIndex];
        clock.innerText = to12h(time24);

        const [h, m] = time24.split(':');
        const hour = parseInt(h) + (parseInt(m)/60);
        updateStory(hour);

        if (window.currentZoomSpan < 280) {
            const halfSpan = window.currentZoomSpan / 2;
            let min = safeIndex - halfSpan;
            let max = safeIndex + halfSpan;
            if (min < 0) { min = 0; max = window.currentZoomSpan; }
            if (max > 287) { max = 287; min = 287 - window.currentZoomSpan; }
            chart.options.scales.x.min = min;
            chart.options.scales.x.max = max;
        }
        chart.update('none');
    }

    function updateStory(h) {
        // Continuous sky interpolation + simple sun/moon up-down motion
        let bg, title, desc, iconId, insight1, insight2, icon1, icon2;

        // keep the same narrative buckets for text + add real-world insights
        if (h < 5.5) { 
            bg = 'linear-gradient(to bottom, #020617, #1e293b)'; 
            title = "Sleeping City"; 
            desc = "Demand is low (Base Load). Streetlights and industrial refrigeration are the main power draw."; 
            iconId = 'svg-moon';
            insight1 = "📊 Demand: 15–20 GW (off-peak)";
            insight2 = "🌊 Hydro + nuclear covering baseload";
        }
        else if (h < 9) { 
            bg = 'linear-gradient(to bottom, #3b82f6, #f97316)'; 
            title = "Waking Up"; 
            desc = "Morning Ramp. Coffee machines, showers, and toasters spike demand as millions wake up."; 
            iconId = 'svg-coffee';
            insight1 = "🔴 Peak demand: 29–30 GW (critical)";
            insight2 = "⚡ Gas plants ramping up to meet load";
        }
        else if (h < 17) { 
            bg = 'linear-gradient(to bottom, #0ea5e9, #bae6fd)'; 
            title = "The Work Day"; 
            desc = "Solar output peaks. Offices are buzzing. Notice how gas plants throttle down as solar floods the grid."; 
            iconId = 'svg-sun';
            insight1 = "☀️ Solar: 35–45% of instantaneous load";
            insight2 = "⬇️ Natural gas at minimum. Wind weak.";
        }
        else if (h < 21) { 
            bg = 'linear-gradient(to bottom, #4338ca, #be185d)'; 
            title = "The Evening Peak"; 
            desc = "Solar vanishes, but people come home. AC units, TVs, and ovens turn on. Batteries discharge to help."; 
            iconId = 'svg-home';
            insight1 = "Demand rising: +2.5 GW/hour";
            insight2 = "🔋 Batteries + imports keeping lights on";
        }
        else { 
            bg = 'linear-gradient(to bottom, #0f172a, #000000)'; 
            title = "Wind Down"; 
            desc = "Lights out. Demand drops rapidly as the grid resets for tomorrow."; 
            iconId = 'svg-bed';
            insight1 = "🌬️ Night wind: 20–30% of generation";
            insight2 = "📉 Demand declining to overnight low";
        }

        stage.style.background = bg;

        // Dim / brighten the translucent graph background based on sun position.
        // sunFactor ranges 0..1. We blend between white (day) and a dark navy (night).
        try {
            const day = clamp(sunFactor, 0, 1);
            const r = Math.round(255 * day + 15 * (1 - day));
            const g = Math.round(255 * day + 23 * (1 - day));
            const b = Math.round(255 * day + 42 * (1 - day));
            const alpha = 0.92 * day + 0.6 * (1 - day); // more opaque at night so it reads darker
            glassPanel.style.background = `rgba(${r}, ${g}, ${b}, ${alpha})`;
        } catch (e) {}

        // Deterministic mapping for vertical position (top percent) along a single up/down path
        const frac = (h % 24) / 24; // 0..1
        // sunFactor peaks at midday (~0.5), 0 at night
        const sunFactor = Math.max(0, Math.sin((frac - 0.25) * 2 * Math.PI));
        const moonFactor = Math.max(0, Math.sin((frac + 0.25) * 2 * Math.PI));

        // Map factors to top % inside orb-container (lower number = higher on screen)
        const topFor = (factor) => 85 - factor * 65; // 85% (horizon) -> 20% (top)
        const sunTop = topFor(sunFactor);
        const moonTop = topFor(moonFactor);

        // Simple up/down bobbing while dragging: small percent offset applied only during dragging
        let bobOffset = 0;
        if (window.__dragging) {
            const t = performance.now() / 180; // time-based oscillation
            const amp = 2.2; // percent amplitude
            bobOffset = Math.sin(t * 2 + frac * Math.PI * 2) * amp;
        }

        // Apply deterministic final positions + transient bob while dragging
        sunEl.style.top = `${sunTop + bobOffset}%`;
        moonEl.style.top = `${moonTop + bobOffset}%`;

        // visibility: visible when factor > 0
        sunEl.style.opacity = sunFactor > 0.01 ? `${clamp(sunFactor,0,1)}` : '0';
        moonEl.style.opacity = moonFactor > 0.01 ? `${clamp(moonFactor,0,1)}` : '0';

        // narrative text swap
        if (storyTitle.innerText !== title) {
            // update content
            storyTitle.innerText = title;
            storyDesc.innerText = desc;
            storyIcon.innerHTML = document.getElementById(iconId).innerHTML;
            
            // Update insights with real-world context
            insightText1.innerText = insight1;
            insightText2.innerText = insight2;

            // trigger a short entrance animation for the updated narrative pieces
            [storyTitle, storyDesc, storyIcon].forEach(el => {
                el.classList.remove('text-entrance');
                // force reflow to restart animation
                void el.offsetWidth;
                el.classList.add('text-entrance');
            });
            
            // Animate insight items
            [insightText1, insightText2].forEach((el, idx) => {
                el.parentElement.style.opacity = '0';
                el.parentElement.style.transform = 'translateX(-12px)';
                setTimeout(() => {
                    el.parentElement.style.transition = 'all 300ms ease';
                    el.parentElement.style.opacity = '1';
                    el.parentElement.style.transform = 'translateX(0)';
                }, 50 + (idx * 100));
            });

            // clear the classes after the animation completes to keep DOM clean
            setTimeout(() => {
                [storyTitle, storyDesc, storyIcon].forEach(el => el.classList.remove('text-entrance'));
            }, 700);
        }
    }

    scrubber.addEventListener('input', (e) => {
        updateState(parseInt(e.target.value));
    });

    // Accessibility: keyboard controls for the scrubber
    try {
        scrubber.setAttribute('role', 'slider');
        scrubber.setAttribute('aria-valuemin', '0');
        scrubber.setAttribute('aria-valuemax', '287');
    } catch (e) {}

    scrubber.addEventListener('keydown', (ev) => {
        const step = ev.shiftKey ? 12 : 1; // shift for larger steps (~1 hour)
        let v = parseInt(scrubber.value || 0);
        if (ev.key === 'ArrowLeft' || ev.key === 'ArrowDown') { v = Math.max(0, v - step); scrubber.value = v; updateState(v); ev.preventDefault(); }
        else if (ev.key === 'ArrowRight' || ev.key === 'ArrowUp') { v = Math.min(287, v + step); scrubber.value = v; updateState(v); ev.preventDefault(); }
        else if (ev.key === 'PageDown') { v = Math.max(0, v - 12); scrubber.value = v; updateState(v); ev.preventDefault(); }
        else if (ev.key === 'PageUp') { v = Math.min(287, v + 12); scrubber.value = v; updateState(v); ev.preventDefault(); }
        else if (ev.key === 'Home') { scrubber.value = 0; updateState(0); ev.preventDefault(); }
        else if (ev.key === 'End') { scrubber.value = 287; updateState(287); ev.preventDefault(); }
    });

    // Drag state: add pointer events to animate bobbing while user drags the scrubber
    window.__dragging = false;
    scrubber.addEventListener('pointerdown', () => {
        window.__dragging = true;
        // start a small RAF loop to animate bob while dragging
        if (!window.__dragRAF) {
            const loop = () => {
                updateState(window.activeIndex || 0);
                window.__dragRAF = requestAnimationFrame(loop);
            };
            window.__dragRAF = requestAnimationFrame(loop);
        }
    });
    const stopDrag = () => {
        window.__dragging = false;
        if (window.__dragRAF) {
            cancelAnimationFrame(window.__dragRAF);
            window.__dragRAF = null;
            // final snap to exact position
            updateState(window.activeIndex || 0);
        }
    };
    window.addEventListener('pointerup', stopDrag);
    scrubber.addEventListener('pointercancel', stopDrag);

    // Init
    updateState(0);

</script>
</body>
</html>
