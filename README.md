<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Safe-Tag Intelligence | Dashboard Otimizado</title>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800&family=Bangers&family=Fredoka:wght@400;600;700&display=swap" rel="stylesheet">
    
    <style>
        :root {
            --energisa-green: #009640;
            --energisa-blue: #005596;
            --accent: #FFB703;
            --danger: #E63946;
            --success: #2A9D8F;
            --dark-bg: #0f1115;
            --card-bg: rgba(30, 41, 59, 0.7);
            --glass: rgba(255, 255, 255, 0.05);
            --border: rgba(255, 255, 255, 0.08);
            --text-main: #F8FAFC;
            --text-muted: #94A3B8;
            --focus-outline: #FFB703;
        }

        * { box-sizing: border-box; margin: 0; padding: 0; }
        body { font-family: 'Inter', sans-serif; background-color: var(--dark-bg); color: var(--text-main); overflow: hidden; height: 100vh; display: flex; flex-direction: column; }
        
        /* Acessibilidade */
        .sr-only { position: absolute; width: 1px; height: 1px; padding: 0; margin: -1px; overflow: hidden; clip: rect(0, 0, 0, 0); white-space: nowrap; border-width: 0; }
        :focus-visible { outline: 3px solid var(--focus-outline); outline-offset: 4px; }

        /* --- MODAL DE SCAN --- */
        #scan-modal { position: fixed; top: 0; left: 0; width: 100vw; height: 100vh; background: rgba(15, 23, 42, 0.95); backdrop-filter: blur(15px); z-index: 150; display: flex; align-items: center; justify-content: center; opacity: 0; pointer-events: none; transition: opacity 0.3s; }
        #scan-modal.active { opacity: 1; pointer-events: all; }
        .modal-card { background: #1e293b; border: 1px solid var(--border); width: 90%; max-width: 420px; padding: 30px; border-radius: 20px; box-shadow: 0 20px 50px rgba(0,0,0,0.5); text-align: center; display: flex; flex-direction: column; }
        .modal-icon { font-size: 3rem; color: var(--energisa-green); margin-bottom: 15px; }
        .form-group { margin-bottom: 15px; text-align: left; }
        .form-label { display: block; font-size: 1rem; color: var(--text-main); margin-bottom: 8px; font-weight: 600; }
        .form-select { width: 100%; padding: 14px; border-radius: 8px; border: 2px solid #334155; background: #0f172a; color: white; font-size: 1.1rem; outline: none; cursor: pointer; transition: border 0.3s; }
        .form-select:focus { border-color: var(--focus-outline); }
        .checkbox-wrapper { display: flex; align-items: flex-start; gap: 12px; margin-bottom: 25px; padding: 15px; background: rgba(0,0,0,0.2); border-radius: 8px; border: 1px solid var(--border); text-align: left; }
        .custom-checkbox { appearance: none; width: 24px; height: 24px; border: 2px solid var(--text-muted); border-radius: 4px; cursor: pointer; flex-shrink: 0; margin-top: 2px; position: relative; background: transparent; transition: all 0.2s; }
        .custom-checkbox:checked { background-color: var(--energisa-green); border-color: var(--energisa-green); }
        .custom-checkbox:checked::after { content: '✔'; position: absolute; color: white; font-size: 18px; left: 3px; top: -1px; font-weight: bold; }
        .epi-label { font-size: 1rem; color: var(--text-muted); line-height: 1.5; cursor: pointer; }
        .epi-label strong { color: var(--energisa-green); }
        .btn-confirm { width: 100%; padding: 18px; border-radius: 10px; border: none; background: var(--energisa-green); color: white; font-weight: 700; font-size: 1.1rem; cursor: pointer; box-shadow: 0 4px 15px rgba(0,150,64,0.3); transition: all 0.2s; }
        .btn-confirm:hover:not(:disabled) { transform: translateY(-2px); filter: brightness(1.1); }
        .btn-confirm:disabled { opacity: 0.5; cursor: not-allowed; filter: grayscale(100%); box-shadow: none; background: #334155; }

        /* --- GERAL --- */
        .hidden { display: none !important; opacity: 0; }
        .fade-in { animation: fadeIn 0.4s ease-in-out; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
        header { background: rgba(15, 23, 42, 0.9); backdrop-filter: blur(10px); padding: 1rem 2rem; display: flex; justify-content: space-between; align-items: center; border-bottom: 1px solid var(--border); z-index: 100; }
        .logo { font-weight: 800; font-size: 1.4rem; display: flex; align-items: center; gap: 10px; }
        .logo i { color: var(--energisa-green); }
        .nav-toggle { background: var(--glass); padding: 4px; border-radius: 12px; display: flex; border: 1px solid var(--border); }
        .nav-btn { padding: 8px 24px; border-radius: 8px; color: var(--text-muted); font-weight: 600; background: transparent; transition: all 0.3s; cursor: pointer; }
        .nav-btn.active { background: var(--energisa-green); color: white; box-shadow: 0 4px 12px rgba(0, 150, 64, 0.3); }

        #field-view { flex: 1; position: relative; background: radial-gradient(circle at 50% 30%, #e0f7fa, #80deea, #4caf50); display: flex; flex-direction: column; align-items: center; justify-content: center; overflow: hidden; }
        .demo-floating-btn { position: absolute; top: 20px; right: 20px; background: var(--energisa-blue); color: white; border: none; padding: 12px 24px; border-radius: 30px; font-weight: 600; box-shadow: 0 4px 15px rgba(0,0,0,0.2); cursor: pointer; z-index: 50; display: flex; align-items: center; gap: 8px; transition: transform 0.2s; font-size: 1rem; }
        .demo-floating-btn:hover { transform: scale(1.05); }

        .pole-container { position: relative; width: 100px; height: 70vh; background: linear-gradient(90deg, #424242, #757575, #424242); border-radius: 6px 6px 0 0; box-shadow: 0 20px 50px rgba(0,0,0,0.5); z-index: 10; }
        .pole-arm { position: absolute; top: 20%; left: -40px; width: 140px; height: 12px; background: #616161; border-radius: 6px; }
        .insulator { position: absolute; top: 17%; left: -30px; width: 24px; height: 50px; background: repeating-linear-gradient(45deg, #333, #333 6px, #f5f5f5 6px, #f5f5f5 8px); border-radius: 4px; }
        .safe-tag { position: absolute; top: 30%; left: 50%; transform: translateX(-50%) scale(1); width: 160px; height: 220px; background: #fff; border-radius: 12px; box-shadow: 0 15px 35px rgba(0,0,0,0.3); padding: 0; display: flex; flex-direction: column; align-items: center; cursor: pointer; transition: all 0.3s; z-index: 20; overflow: hidden; border: 1px solid #ddd; }
        .safe-tag:hover { transform: translateX(-50%) scale(1.05); box-shadow: 0 0 0 4px rgba(0, 150, 64, 0.2); }
        .tag-header-stripe { width: 100%; height: 50px; background: linear-gradient(135deg, var(--energisa-green), var(--energisa-blue)); display: flex; flex-direction: column; justify-content: center; align-items: center; color: white; padding-bottom: 5px; position: relative; }
        .tag-header-stripe::after { content: ''; position: absolute; bottom: -5px; left: 0; width: 0; height: 0; border-left: 80px solid transparent; border-right: 80px solid transparent; border-top: 5px solid var(--energisa-blue); }
        .tag-title { font-size: 0.7rem; font-weight: 800; letter-spacing: 1px; text-transform: uppercase; margin-top: 5px; }
        .tag-nfc-icon { font-size: 1.5rem; color: white; margin-top: -2px; animation: pulseIcon 2s infinite; }
        @keyframes pulseIcon { 0% { opacity: 0.6; transform: scale(1); } 50% { opacity: 1; transform: scale(1.1); } 100% { opacity: 0.6; transform: scale(1); } }
        .tag-body { flex: 1; width: 100%; padding: 15px; display: flex; flex-direction: column; align-items: center; background: #f8fafc; }
        .qr-placeholder { width: 100px; height: 100px; background: #fff; border: 1px solid #eee; background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100" fill="%23000"><rect x="10" y="10" width="30" height="30"/><rect x="60" y="10" width="30" height="30"/><rect x="10" y="60" width="30" height="30"/><rect x="50" y="50" width="10" height="10"/><rect x="70" y="70" width="20" height="20"/><rect x="15" y="15" width="20" height="20" fill="%23fff"/><rect x="65" y="15" width="20" height="20" fill="%23fff"/><rect x="15" y="65" width="20" height="20" fill="%23fff"/></svg>'); border-radius: 8px; margin-top: 10px; box-shadow: 0 2px 5px rgba(0,0,0,0.1); }
        .tag-footer { font-size: 0.6rem; color: #64748B; font-weight: 600; margin-top: 10px; text-align: center; }

        .nfc-scan-overlay { position: absolute; top: 0; left: 0; right: 0; bottom: 0; background: rgba(255,255,255,0.95); z-index: 50; display: flex; flex-direction: column; align-items: center; justify-content: center; opacity: 0; pointer-events: none; transition: opacity 0.3s; }
        .nfc-scan-overlay.active { opacity: 1; pointer-events: all; }
        .scan-line { width: 100%; height: 4px; background: var(--energisa-green); position: absolute; top: 0; animation: scanMove 1.5s infinite linear; box-shadow: 0 0 10px var(--energisa-green); }
        @keyframes scanMove { 0% { top: 0; opacity: 0; } 50% { opacity: 1; } 100% { top: 100%; opacity: 0; } }

        /* --- HQ VIEW --- */
        #hq-view { position: fixed; top: 0; left: 0; width: 100vw; height: 100vh; background: #0a0a0a; z-index: 200; display: flex; flex-direction: column; }
        .hq-top-bar { padding: 15px 20px; background: linear-gradient(to right, #1a1a1a, #2d2d2d); display: flex; justify-content: space-between; align-items: center; border-bottom: 1px solid #333; }
        .hq-brand { font-family: 'Bangers', cursive; font-size: 1.8rem; color: var(--accent); letter-spacing: 1px; text-shadow: 2px 2px 0 #000; }
        .close-hq { background: none; color: #fff; font-size: 2rem; border: none; cursor: pointer; padding: 10px; border-radius: 50%; transition: background 0.2s; }
        .close-hq:hover { background: rgba(255,255,255,0.1); }
        .comic-stage { flex: 1; display: flex; align-items: center; justify-content: center; position: relative; overflow: hidden; padding: 20px; }
        .panel-card { width: 100%; max-width: 600px; background: white; border: 6px solid #000; border-radius: 4px; overflow: hidden; display: none; animation: panelPop 0.4s; box-shadow: 0 20px 40px rgba(0,0,0,0.5); }
        .panel-card.active { display: block; }
        @keyframes panelPop { from { transform: scale(0.9); opacity: 0; } to { transform: scale(1); opacity: 1; } }
        .panel-image { min-height: 300px; background: #f0f0f0; position: relative; display: flex; align-items: center; justify-content: center; font-size: 5rem; }
        .panel-caption { padding: 25px; font-family: 'Fredoka', sans-serif; font-size: 1.4rem; font-weight: 700; text-align: center; border-top: 4px solid #000; min-height: auto; display: flex; align-items: center; justify-content: center; color: #111; line-height: 1.5; }
        .panel-caption strong { color: var(--danger); display: block; margin-bottom: 5px; }
        .panel-caption strong.safe-text { color: var(--energisa-green); }
        .speech-bubble { position: absolute; background: white; border: 3px solid #000; padding: 15px; border-radius: 20px; font-family: 'Fredoka', sans-serif; font-size: 1.2rem; font-weight: bold; box-shadow: 4px 4px 0 rgba(0,0,0,0.1); z-index: 5; max-width: 200px; }
        .speech-bubble::after { content: ''; position: absolute; bottom: -15px; left: 20px; border-width: 15px 10px 0; border-style: solid; border-color: #000 transparent; }
        .speech-bubble::before { content: ''; position: absolute; bottom: -10px; left: 22px; border-width: 12px 8px 0; border-style: solid; border-color: #fff transparent; z-index: 1; }
        .hq-controls { padding: 30px; display: flex; justify-content: center; gap: 20px; background: #111; }
        .btn-action { padding: 16px 40px; border-radius: 50px; border: none; font-weight: 700; cursor: pointer; font-size: 1.2rem; transition: transform 0.2s; min-width: 150px; }
        .btn-action:active { transform: scale(0.95); }
        .btn-sec { background: #333; color: #fff; }
        .btn-prim { background: var(--energisa-green); color: white; box-shadow: 0 4px 15px rgba(0,150,64,0.4); }

        /* --- NOVO DASHBOARD (MAIOR E ALINHADO) --- */
        #dashboard-view {
            display: grid;
            /* Col 1: Header/KPI, Col 2: Mapa, Col 3: Stats, Col 4: Logs */
            grid-template-columns: 300px 1fr 350px 350px;
            gap: 24px;
            padding: 24px;
            height: calc(100vh - 70px); /* Menos altura do header */
            background: var(--dark-bg);
        }
        
        .dash-card {
            background: var(--card-bg);
            backdrop-filter: blur(12px);
            border: 1px solid var(--border);
            border-radius: 20px;
            padding: 24px;
            color: var(--text-main);
            display: flex;
            flex-direction: column;
            box-shadow: 0 4px 20px rgba(0,0,0,0.2);
        }

        /* Coluna 1: Identidade */
        .dash-col-1 {
            grid-column: 1;
            display: flex;
            flex-direction: column;
            gap: 24px;
        }
        .dash-header {
            background: linear-gradient(135deg, var(--energisa-green), var(--energisa-blue));
            padding: 24px;
            border-radius: 20px;
            color: white;
            text-align: center;
            box-shadow: 0 10px 25px rgba(0,150,64,0.2);
        }
        .dash-header h1 { font-size: 1.8rem; font-weight: 800; margin-bottom: 5px; line-height: 1.1; }
        .dash-header p { font-size: 1rem; opacity: 0.9; }
        
        .big-kpi { text-align: center; padding: 20px 0; border: 1px solid var(--border); border-radius: 20px; background: rgba(0,0,0,0.2); }
        .big-kpi h2 { font-size: 0.9rem; color: var(--text-muted); text-transform: uppercase; letter-spacing: 1px; margin-bottom: 10px; }
        .big-kpi .number { font-size: 3.5rem; font-weight: 800; color: var(--text-main); line-height: 1; margin-bottom: 5px; }
        .big-kpi .sub { font-size: 1rem; color: var(--success); font-weight: 600; }

        /* Coluna 2: O MAPA GRANDE */
        .main-map {
            grid-column: 2;
            position: relative;
            background: #15181e;
            border-radius: 20px;
            overflow: hidden;
            border: 1px solid var(--border);
            box-shadow: inset 0 0 50px rgba(0,0,0,0.5);
        }
        .map-bg { position: absolute; width: 100%; height: 100%; background-color: #0f1115; background-image: linear-gradient(rgba(255,255,255,0.03) 1px, transparent 1px), linear-gradient(90deg, rgba(255,255,255,0.03) 1px, transparent 1px); background-size: 40px 40px; }
        .map-roads { position: absolute; width: 100%; height: 100%; }
        .road { fill: none; stroke: #2a2d35; stroke-width: 8; stroke-linecap: round; }
        
        /* Pins */
        .pin { position: absolute; width: 18px; height: 18px; border-radius: 50%; cursor: pointer; transform: translate(-50%, -50%); z-index: 10; transition: transform 0.3s; }
        .pin:hover { transform: translate(-50%, -50%) scale(1.5); z-index: 20; }
        .pin::after { content: ''; position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); width: 100%; height: 100%; border-radius: 50%; animation: pinPulse 2s infinite; }
        .pin.green { background: var(--success); box-shadow: 0 0 15px var(--success); } .pin.green::after { border: 2px solid var(--success); }
        .pin.yellow { background: var(--accent); box-shadow: 0 0 15px var(--accent); } .pin.yellow::after { border: 2px solid var(--accent); animation-delay: 0.5s; }
        .pin.red { background: var(--danger); box-shadow: 0 0 20px var(--danger); width: 24px; height: 24px; } .pin.red::after { border: 3px solid var(--danger); animation: pinPulse 1s infinite; background: rgba(230,57,70,0.2); }
        @keyframes pinPulse { 0% { width: 100%; height: 100%; opacity: 0.8; } 100% { width: 400%; height: 400%; opacity: 0; } }
        
        .map-legend {
            position: absolute; bottom: 24px; left: 24px;
            background: rgba(15, 23, 42, 0.9); padding: 16px;
            border-radius: 12px; border: 1px solid var(--border);
            font-size: 0.85rem; color: #ccc; z-index: 15; backdrop-filter: blur(5px);
        }
        .legend-item { display: flex; align-items: center; gap: 10px; margin-bottom: 6px; font-weight: 500; }
        .dot { width: 10px; height: 10px; border-radius: 50%; display: inline-block; }
        .dot.green { background: var(--success); }
        .dot.yellow { background: var(--accent); }
        .dot.red { background: var(--danger); }

        /* Coluna 3: Stats */
        .dash-col-3 {
            grid-column: 3;
            display: flex;
            flex-direction: column;
            gap: 24px;
        }
        .kpi-card h3 { font-size: 1.1rem; color: var(--text-main); margin-bottom: 15px; display: flex; align-items: center; justify-content: space-between; }
        .kpi-value { font-size: 2.5rem; font-weight: 700; color: #fff; margin-bottom: 5px; }
        .kpi-desc { font-size: 0.9rem; color: var(--text-muted); line-height: 1.4; }
        
        .chart-container { position: relative; width: 180px; height: 180px; margin: 10px auto; }
        svg.donut { transform: rotate(-90deg); width: 100%; height: 100%; }
        circle.donut-ring { fill: transparent; stroke: #334155; stroke-width: 25; }
        circle.donut-segment { fill: transparent; stroke-width: 25; transition: stroke-dasharray 0.5s ease; }
        .chart-legend { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; margin-top: 15px; font-size: 0.8rem; color: var(--text-muted); }
        .legend-box { display: flex; align-items: center; gap: 5px; }
        .mini-dot { width: 8px; height: 8px; border-radius: 50%; display: inline-block; }

        /* Coluna 4: Logs */
        .dash-col-4 {
            grid-column: 4;
            display: flex;
            flex-direction: column;
        }
        .log-panel { flex: 1; display: flex; flex-direction: column; }
        .log-panel h3 { font-size: 1.1rem; color: var(--text-main); margin-bottom: 15px; padding-bottom: 15px; border-bottom: 1px solid var(--border); display: flex; justify-content: space-between; align-items: center; }
        .log-feed { flex: 1; overflow-y: auto; display: flex; flex-direction: column; gap: 12px; padding-right: 5px; scrollbar-width: thin; scrollbar-color: #334155 transparent; }
        .log-entry { padding: 15px; background: rgba(0,0,0,0.2); border-radius: 12px; border-left: 4px solid transparent; font-size: 0.9rem; transition: background 0.2s; display: flex; flex-direction: column; gap: 5px; }
        .log-entry:hover { background: rgba(255,255,255,0.05); }
        .log-time { color: var(--text-muted); font-family: monospace; font-size: 0.8rem; font-weight: 600; }
        .log-msg { color: #e2e8f0; line-height: 1.4; }
        .log-entry.safe { border-color: var(--success); }
        .log-entry.warn { border-color: var(--accent); }
        .log-entry.danger { border-color: var(--danger); background: rgba(230,57,70,0.05); }

        .demo-overlay { position: fixed; top: 100px; left: 50%; transform: translateX(-50%); background: rgba(0,0,0,0.85); color: white; padding: 15px 25px; border-radius: 30px; border: 1px solid var(--energisa-green); z-index: 300; font-weight: 600; display: none; backdrop-filter: blur(5px); box-shadow: 0 10px 30px rgba(0,0,0,0.5); font-size: 1.1rem; }
        .demo-overlay.active { display: block; animation: fadeIn 0.3s; }

        /* Responsividade para Tablet/Desktop Pequeno */
        @media (max-width: 1400px) {
            #dashboard-view {
                grid-template-columns: 280px 1fr 350px;
                grid-template-rows: auto auto;
                overflow-y: auto;
                height: auto;
            }
            .dash-col-4 { grid-column: 1 / -1; height: 400px; }
        }

        /* Responsividade Mobile */
        @media (max-width: 900px) {
            #dashboard-view { 
                grid-template-columns: 1fr; 
                overflow-y: auto; 
                height: auto; 
                padding-bottom: 50px; 
            }
            .dash-col-1, .dash-col-3, .dash-col-4 { grid-column: 1; }
            .main-map { grid-column: 1; height: 400px; }
            .dash-header { text-align: left; }
        }
    </style>
</head>
<body>

    <header>
        <div class="logo">
            <i class="fas fa-shield-alt" aria-hidden="true"></i>
            Safe-Tag <span>Intelligence</span>
        </div>
        <div class="nav-toggle">
            <button class="nav-btn active" onclick="app.switchView('field')">Campo</button>
            <button class="nav-btn" onclick="app.switchView('dashboard')">Dashboard</button>
        </div>
    </header>

    <!-- DEMO OVERLAY -->
    <div id="demo-toast" class="demo-overlay" role="alert" aria-live="polite">Iniciando Simulação...</div>

    <!-- VISÃO CAMPO -->
    <main id="field-view" class="view-section fade-in">
        <button class="demo-floating-btn" onclick="app.runVideoDemo()">
            <i class="fas fa-play" aria-hidden="true"></i> Assistir Demonstração
        </button>

        <div class="pole-container" aria-hidden="true">
            <div class="pole-arm"></div>
            <div class="insulator"></div>
            
            <div class="safe-tag" onclick="app.scanNFC()" tabindex="0" role="button" aria-label="Tag NFC de Segurança. Clique ou toque para iniciar o processo de segurança.">
                <div class="nfc-scan-overlay" id="nfc-overlay">
                    <div class="scan-line"></div>
                    <i class="fas fa-wifi" style="font-size: 3rem; color: var(--energisa-green); margin-bottom: 20px;" aria-hidden="true"></i>
                    <h3 style="color: #333; font-weight: 700;">Lendo Tag...</h3>
                </div>

                <div class="tag-header-stripe">
                    <span class="tag-title">CUIDADO: REDE VIVA</span>
                    <i class="fas fa-wifi tag-nfc-icon" aria-hidden="true"></i>
                </div>
                
                <div class="tag-body">
                    <div style="font-size: 0.8rem; color: #64748B; margin-bottom: 10px; text-align: center; line-height: 1.2;">Aproxime seu celular para acessar o guia de segurança.</div>
                    <div class="qr-placeholder" aria-label="QR Code de Acesso"></div>
                    <div class="tag-footer">SAFE-TAG NFC SYSTEM<br>ID: 402-PT</div>
                </div>
            </div>
        </div>
    </main>

    <!-- MODAL DE IDENTIFICAÇÃO E RESPONSABILIDADE -->
    <div id="scan-modal" role="dialog" aria-modal="true" aria-labelledby="modal-title">
        <div class="modal-card fade-in">
            <div class="modal-icon"><i class="fas fa-user-shield" aria-hidden="true"></i></div>
            <h2 id="modal-title" style="margin-bottom: 5px; color: white;">Registro de Acesso</h2>
            <p style="color: var(--text-muted); margin-bottom: 20px; font-size: 0.9rem;">Identificação obrigatória para garantir a segurança e rastreabilidade.</p>

            <form onsubmit="event.preventDefault(); app.confirmScan();">
                <div class="form-group">
                    <label for="profile-select" class="form-label">Seu Perfil</label>
                    <select id="profile-select" class="form-select">
                        <option value="" disabled selected>Selecione...</option>
                        <option value="Pedreiro">Pedreiro / Construção Civil</option>
                        <option value="Pintor">Pintor de Fachada</option>
                        <option value="Eletricista">Eletricista / Manutenção</option>
                        <option value="Terceirizado">Equipe Terceirizada</option>
                        <option value="Cidadao">Cidadão Comum</option>
                    </select>
                </div>

                <div class="form-group">
                    <label for="work-select" class="form-label">Tipo de Obra / Serviço</label>
                    <select id="work-select" class="form-select">
                        <option value="" disabled selected>Selecione...</option>
                        <option value="Obra Civil">Obra Civil / Reforma</option>
                        <option value="Pintura">Pintura / Revestimento</option>
                        <option value="Manutenção">Manutenção Interna</option>
                        <option value="Podas">Poda de Árvores / Limpeza</option>
                        <option value="Outros">Outros</option>
                    </select>
                </div>

                <div class="checkbox-wrapper">
                    <input type="checkbox" id="epi-checkbox" class="custom-checkbox">
                    <label for="epi-checkbox" class="epi-label">
                        Confirmo que estou utilizando meus <strong>EPIs</strong> e assumo a responsabilidade sobre o que estou fazendo.
                    </label>
                </div>

                <button type="submit" id="btn-confirm-scan" class="btn-confirm" disabled>
                    Confirmar e Acessar HQ <i class="fas fa-arrow-right" style="margin-left: 8px;" aria-hidden="true"></i>
                </button>
            </form>
        </div>
    </div>

    <!-- VISÃO HQ (Acessível) -->
    <section id="hq-view" class="hidden" aria-live="polite" aria-labelledby="hq-title">
        <div class="hq-top-bar">
            <div class="hq-brand" id="hq-title"><i class="fas fa-book-open" aria-hidden="true"></i> Regra dos 3 Metros</div>
            <button class="close-hq" onclick="app.closeHQ()" aria-label="Fechar HQ e voltar">
                <i class="fas fa-times" aria-hidden="true"></i>
            </button>
        </div>

        <div class="comic-stage">
            <!-- Quadro 1: O Risco -->
            <article class="panel-card active" data-index="1" aria-label="Quadro 1: O Risco. Um trabalhador se aproxima perigosamente de um poste.">
                <div class="panel-image" style="background: linear-gradient(#87CEEB 70%, #81C784 70%);" aria-hidden="true">
                    <i class="fas fa-hard-hat" style="color: #FFA726; position: absolute; bottom: 30%; left: 30%; font-size: 6rem;"></i>
                    <i class="fas fa-paint-roller" style="color: #78909C; position: absolute; bottom: 32%; left: 32%; font-size: 4rem; transform: rotate(-45deg);"></i>
                    <div class="speech-bubble" style="top: 10%; right: 10%; color: #333;">Só um pouquinho mais pra cima...</div>
                </div>
                <span class="sr-only">Ilustração mostrando o pintor Antônio próximo a um poste elétrico, sem perceber o perigo.</span>
                <figcaption class="panel-caption">
                    <strong>O Risco:</strong> O Antônio, um pintor, se aproxima do poste sem perceber o perigo invisível.
                </figcaption>
            </article>

            <!-- Quadro 2: O Arco -->
            <article class="panel-card" data-index="2" aria-label="Quadro 2: O Perigo. Alerta de Arco Elétrico.">
                <div class="panel-image" style="background: #222;" aria-hidden="true">
                    <i class="fas fa-bolt" style="color: #FFEB3B; font-size: 7rem; text-shadow: 0 0 30px orange; animation: pulseIcon 0.2s infinite;"></i>
                    <div style="position: absolute; bottom: 20%; right: 20%; font-family: 'Bangers'; font-size: 4rem; color: #fff; transform: rotate(-10deg);">ZRUMMM!</div>
                </div>
                <span class="sr-only">Ilustração dramática de um raio elétrico, representando o perigo iminente de choque.</span>
                <figcaption class="panel-caption" style="background: #fffbe6; border-color: var(--danger);">
                    <strong style="color: var(--danger);">ALERTA DE ARCO ELÉTRICO!</strong><br>
                    A eletricidade não precisa te tocar para te ferir. Mantenha distância!
                </figcaption>
            </article>

            <!-- Quadro 3: A Regra -->
            <article class="panel-card" data-index="3" aria-label="Quadro 3: A Regra dos 3 Metros.">
                <div class="panel-image" style="background: linear-gradient(#87CEEB 70%, #81C784 70%);" aria-hidden="true">
                    <div style="position: absolute; bottom: 0; left: 20%; width: 60px; height: 120px; border: 4px dashed red; opacity: 0.8; border-radius: 10px;"></div>
                    <div style="position: absolute; bottom: 10%; left: 30%; color: red; font-weight: bold; font-size: 1rem;">3 METROS</div>
                    <i class="fas fa-hard-hat" style="color: #FFA726; position: absolute; bottom: 20%; left: 70%; font-size: 6rem;"></i>
                    <i class="fas fa-check-circle" style="color: var(--success); position: absolute; top: 20%; left: 20%; font-size: 4rem;"></i>
                </div>
                <span class="sr-only">Ilustração mostrando a zona de segurança de 3 metros ao redor do poste.</span>
                <figcaption class="panel-caption">
                    <strong>A Solução:</strong> A Regra dos 3 Metros garante que você esteja fora da zona de arco volante.
                </figcaption>
            </article>

             <!-- Quadro 4: Ajuste da Escada -->
             <article class="panel-card" data-index="4" aria-label="Quadro 4: Garantindo a Segurança. O trabalhador se afasta.">
                <div class="panel-image" style="background: linear-gradient(#87CEEB 70%, #81C784 70%);" aria-hidden="true">
                    <div style="position: absolute; bottom: 0; left: 20%; width: 60px; height: 120px; border: 4px dashed red; opacity: 0.3;"></div>
                    <i class="fas fa-hard-hat" style="color: #FFA726; position: absolute; bottom: 20%; left: 70%; font-size: 6rem; transform: translateX(80px);"></i>
                    <i class="fas fa-arrow-left" style="color: red; position: absolute; bottom: 25%; left: 50%; font-size: 3rem;"></i>
                    <div class="speech-bubble" style="top: 20%; right: 20%; color: #333;">Melhor seguro do que arrependido!</div>
                </div>
                <span class="sr-only">Ilustração do trabalhador movendo a escada para longe da zona de perigo.</span>
                <figcaption class="panel-caption">
                    <strong>Garantindo a Segurança:</strong> O trabalhador ajusta a posição da escada para fora da zona de risco.
                </figcaption>
            </article>

            <!-- Quadro 5: Sucesso -->
            <article class="panel-card" data-index="5" aria-label="Quadro 5: Trabalho Seguro. Conclusão.">
                <div class="panel-image" style="background: #E0F2F1; display: flex; flex-direction: column; justify-content: center; align-items: center;" aria-hidden="true">
                    <i class="fas fa-shield-alt" style="color: var(--success); font-size: 6rem; margin-bottom: 20px;"></i>
                    <h2 style="color: var(--success); font-family: 'Bangers';">TRABALHO SEGURO!</h2>
                </div>
                <span class="sr-only">Imagem final com escudo verde indicando que o trabalho foi realizado com segurança.</span>
                <figcaption class="panel-caption">
                    <strong class="safe-text">Obrigado, Antônio!</strong> Você entendeu o risco e garantiu sua volta para casa.
                </figcaption>
            </article>
        </div>

        <div class="hq-controls">
            <button class="btn-action btn-sec" id="prev-btn" onclick="app.prevPanel()" disabled aria-label="Painel Anterior">
                Anterior
            </button>
            <button class="btn-action btn-prim" id="next-btn" onclick="app.nextPanel()" aria-label="Próximo Painel">
                Próximo
            </button>
        </div>
    </section>

    <!-- NOVO DASHBOARD (MAIOR E ALINHADO) -->
    <main id="dashboard-view" class="view-section hidden">
        
        <!-- Coluna 1: Título e KPI Principal -->
        <div class="dash-col-1">
            <div class="dash-header">
                <h1>Safe-Tag<br>Intelligence</h1>
                <p>Monitoramento de Risco em Tempo Real</p>
                <div style="margin-top: 10px; font-size: 0.8rem; opacity: 0.8;">Unidade Operacional: São Paulo - Zona Sul</div>
            </div>
            
            <div class="big-kpi">
                <h2>Total de Impactos</h2>
                <div class="number" id="impact-counter">142</div>
                <div class="sub">Vidas protegidas hoje</div>
            </div>
        </div>

        <!-- Coluna 2: MAPA (O CENTRO) -->
        <div class="main-map" id="map-container">
            <div class="map-bg"></div>
            <svg class="map-roads" viewBox="0 0 800 600" preserveAspectRatio="none" aria-hidden="true">
                <path class="road" d="M0,300 Q400,300 800,300" />
                <path class="road" d="M400,0 Q400,300 400,600" />
                <path class="road" d="M100,100 Q200,200 100,400" />
                <path class="road" d="M700,500 Q600,400 700,200" />
            </svg>
            <div class="map-legend">
                <div class="legend-item"><span class="dot green"></span> Interações Seguras</div>
                <div class="legend-item"><span class="dot yellow"></span> Alerta de Proximidade</div>
                <div class="legend-item"><span class="dot red"></span> Obra Não Mapeada</div>
            </div>
        </div>

        <!-- Coluna 3: Estatísticas -->
        <div class="dash-col-3">
            <div class="kpi-card dash-card">
                <h3>Taxa de Retenção da HQ</h3>
                <div class="kpi-value">1m 42s</div>
                <div class="kpi-desc">Média de leitura por usuário.<br><span style="color: var(--success); font-size: 0.8rem;">↑ Neuroeducação Efetiva</span></div>
            </div>

            <div class="kpi-card dash-card">
                <h3>Acessos por Perfil</h3>
                <div class="chart-container">
                    <svg class="donut" viewBox="0 0 100 100" aria-label="Gráfico de Rosca mostrando 40% Construção Civil, 30% Pintura, 20% Manutenção, 10% Outros">
                        <circle class="donut-ring" cx="50" cy="50" r="40"></circle>
                        <circle class="donut-segment" cx="50" cy="50" r="40" stroke="#3B82F6" stroke-dasharray="100.5 150.8" stroke-dashoffset="0"></circle>
                        <circle class="donut-segment" cx="50" cy="50" r="40" stroke="#10B981" stroke-dasharray="75.4 175.9" stroke-dashoffset="-100.5"></circle>
                        <circle class="donut-segment" cx="50" cy="50" r="40" stroke="#F59E0B" stroke-dasharray="50.3 201" stroke-dashoffset="-175.9"></circle>
                        <circle class="donut-segment" cx="50" cy="50" r="40" stroke="#64748B" stroke-dasharray="25.1 226.2" stroke-dashoffset="-226.2"></circle>
                    </svg>
                </div>
                <div class="chart-legend">
                    <div class="legend-box"><span class="mini-dot" style="background:#3B82F6"></span> Const. 40%</div>
                    <div class="legend-box"><span class="mini-dot" style="background:#10B981"></span> Pintura 30%</div>
                    <div class="legend-box"><span class="mini-dot" style="background:#F59E0B"></span> Manut. 20%</div>
                    <div class="legend-box"><span class="mini-dot" style="background:#64748B"></span> Outros 10%</div>
                </div>
            </div>

            <div class="kpi-card dash-card">
                <h3>ROI Estimado</h3>
                <div class="kpi-value" style="color: var(--energisa-green);" id="roi-counter">R$ 45.200</div>
                <div class="kpi-desc">Economia diária com multas e acidentes evitados.</div>
            </div>
        </div>

        <!-- Coluna 4: Logs -->
        <div class="dash-col-4">
            <div class="log-panel dash-card">
                <h3>Feed de Atividade Recente</h3>
                <div class="log-feed" id="log-feed">
                    <!-- Logs via JS -->
                </div>
            </div>
        </div>
    </main>

    <script>
        const app = {
            currentPanel: 1,
            totalPanels: 5,
            currentUserData: { profile: '', work: '', epiConfirmed: false },
            
            init: function() {
                this.renderLogs();
                this.renderMap();
                
                const checkbox = document.getElementById('epi-checkbox');
                const btn = document.getElementById('btn-confirm-scan');
                
                checkbox.addEventListener('change', (e) => {
                    btn.disabled = !e.target.checked;
                    btn.style.opacity = e.target.checked ? '1' : '0.5';
                });
            },

            switchView: function(view) {
                const field = document.getElementById('field-view');
                const dash = document.getElementById('dashboard-view');
                const btns = document.querySelectorAll('.nav-btn');

                if (view === 'field') {
                    field.classList.remove('hidden');
                    dash.classList.add('hidden');
                    btns[0].classList.add('active');
                    btns[1].classList.remove('active');
                } else {
                    field.classList.add('hidden');
                    dash.classList.remove('hidden');
                    btns[0].classList.remove('active');
                    btns[1].classList.add('active');
                }
            },

            scanNFC: function() {
                const overlay = document.getElementById('nfc-overlay');
                overlay.classList.add('active');
                setTimeout(() => {
                    overlay.classList.remove('active');
                    this.openScanModal(); 
                }, 1000);
            },

            openScanModal: function() {
                document.getElementById('profile-select').value = "";
                document.getElementById('work-select').value = "";
                document.getElementById('epi-checkbox').checked = false;
                document.getElementById('btn-confirm-scan').disabled = true;
                document.getElementById('scan-modal').classList.add('active');
                
                setTimeout(() => document.getElementById('profile-select').focus(), 100);
            },

            confirmScan: function() {
                const profile = document.getElementById('profile-select').value;
                const work = document.getElementById('work-select').value;
                const epiChecked = document.getElementById('epi-checkbox').checked;

                if (!profile || !work || !epiChecked) {
                    alert("Preencha todos os campos e confirme o uso de EPIs.");
                    return;
                }

                this.currentUserData = { profile, work, epiConfirmed: true };
                document.getElementById('scan-modal').classList.remove('active');
                
                this.addLog(`Usuário [${profile}] - Obra: ${work}. EPIs confirmados.`, 'safe');
                
                this.openHQ();
            },

            openHQ: function() {
                document.getElementById('hq-view').classList.remove('hidden');
                this.currentPanel = 1;
                this.updateHQ();
                
                setTimeout(() => document.getElementById('prev-btn').focus(), 100);
            },

            closeHQ: function() {
                document.getElementById('hq-view').classList.add('hidden');
                if (this.currentPanel === this.totalPanels) {
                    const { profile, work } = this.currentUserData;
                    this.addLog(`${profile} completou a neuroeducação com sucesso.`, 'safe');
                }
            },

            nextPanel: function() {
                if (this.currentPanel < this.totalPanels) {
                    this.currentPanel++;
                    this.updateHQ();
                } else {
                    this.closeHQ();
                    if (this.isDemoRunning) {
                        setTimeout(() => this.switchView('dashboard'), 500);
                    }
                }
            },

            prevPanel: function() {
                if (this.currentPanel > 1) {
                    this.currentPanel--;
                    this.updateHQ();
                }
            },

            updateHQ: function() {
                document.querySelectorAll('.panel-card').forEach(el => {
                    el.classList.remove('active');
                    el.setAttribute('aria-hidden', 'true');
                });
                
                const activePanel = document.querySelector(`.panel-card[data-index="${this.currentPanel}"]`);
                activePanel.classList.add('active');
                activePanel.setAttribute('aria-hidden', 'false');

                const btnNext = document.getElementById('next-btn');
                const btnPrev = document.getElementById('prev-btn');
                btnPrev.disabled = this.currentPanel === 1;
                btnNext.innerText = this.currentPanel === this.totalPanels ? 'Concluir' : 'Próximo';
                btnNext.setAttribute('aria-label', this.currentPanel === this.totalPanels ? 'Concluir e Fechar' : 'Próximo Painel');
            },

            addLog: function(msg, type) {
                const feed = document.getElementById('log-feed');
                const now = new Date();
                const timeStr = now.getHours().toString().padStart(2,'0') + ':' + now.getMinutes().toString().padStart(2,'0');
                
                const div = document.createElement('div');
                div.className = `log-entry fade-in ${type}`;
                div.innerHTML = `<span class="log-time">${timeStr}</span> <span class="log-msg">${msg}</span>`;
                feed.prepend(div);
            },

            renderLogs: function() {
                const initialLogs = [
                    "Poste ID-9921: HQ de Arco Elétrico visualizada (Acesso via NFC).",
                    "Poste ID-4402: Alerta de Obra Civil detectado via Safe-Tag.",
                    "Poste ID-1120: Pedreiro 'João' acessou a HQ via WhatsApp Bot."
                ];
                initialLogs.forEach(log => this.addLog(log, 'safe'));
            },

            renderMap: function() {
                const map = document.getElementById('map-container');
                // Limpa pins anteriores
                const existingPins = map.querySelectorAll('.pin');
                existingPins.forEach(p => p.remove());

                const points = [
                    { x: 50, y: 50, type: 'green' },
                    { x: 25, y: 35, type: 'green' },
                    { x: 65, y: 65, type: 'yellow' },
                    { x: 85, y: 25, type: 'red' },
                    { x: 40, y: 75, type: 'green' },
                ];
                points.forEach(p => {
                    const el = document.createElement('div');
                    el.className = `pin ${p.type}`;
                    el.style.left = p.x + '%';
                    el.style.top = p.y + '%';
                    el.title = p.type === 'red' ? "Zona de Obra Não Mapeada" : (p.type === 'yellow' ? "Alerta de Proximidade" : "Interação Segura");
                    el.setAttribute('role', 'button');
                    el.setAttribute('aria-label', el.title);
                    el.onclick = () => {
                        alert(`Status do Pin: ${p.type.toUpperCase()}\n${el.title}`);
                    };
                    map.appendChild(el);
                });
            },

            isDemoRunning: false,
            runVideoDemo: async function() {
                this.isDemoRunning = true;
                const toast = document.getElementById('demo-toast');
                const show = (text, time = 3000) => {
                    toast.innerText = text;
                    toast.classList.add('active');
                    return new Promise(r => setTimeout(() => { toast.classList.remove('active'); r(); }, time));
                };

                await show("Cena 1: Trabalhador chega perto do poste...", 2000);
                await show("Cena 2: Aproxima celular da Safe-Tag...", 2000);
                this.scanNFC();
                await new Promise(r => setTimeout(r, 1500));
                
                await show("Identificação: Selecionando 'Pintor' e 'Pintura'...", 2500);
                document.getElementById('profile-select').value = "Pintor";
                document.getElementById('work-select').value = "Pintura";
                await new Promise(r => setTimeout(r, 500));
                
                await show("Validação: Trabalhador confirma uso de EPIs...", 2000);
                document.getElementById('epi-checkbox').checked = true;
                await new Promise(r => setTimeout(r, 500));
                
                this.confirmScan();

                await show("Cena 3: HQ com alerta de arco elétrico abriu!", 3000);
                this.nextPanel(); await new Promise(r => setTimeout(r, 1500));
                this.nextPanel(); await new Promise(r => setTimeout(r, 1500));
                await show("Cena 4: Trabalhador ajusta a escada (Segurança).", 3000);
                this.nextPanel(); await new Promise(r => setTimeout(r, 1500));
                await show("Cena 5: Registrando no Dashboard...", 2000);
                this.nextPanel(); await new Promise(r => setTimeout(r, 1000));
                this.switchView('dashboard');
                await show("Acesso seguro confirmado no Poste ID-402.", 4000);
                this.isDemoRunning = false;
            }
        };

        window.onload = () => app.init();
    </script>
</body>
</html>
