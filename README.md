# yusei82817.github.io
自作音ゲーのプレイ用サイト。
4曲を中心に音ゲーを製作。
HTMLコードの入力は基本自分。
曲は8曲まで重ねられるよう設定。

<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <title>FGJK Rhythm Game - 8 Tracks Collection</title>
    <style>
        :root { --pink: #ffb7c5; --neon-blue: #00f2ff; --yuragi-purple: #a855f7; }
        body { background: #000; color: white; font-family: 'Share Tech Mono', monospace; display: flex; align-items: center; justify-content: center; height: 100vh; margin: 0; overflow: hidden; }
        
        /* セレクト画面：スクロール対応 */
        #select-screen { 
            position: absolute; width: 100%; height: 100%; background: #050505; 
            display: flex; flex-direction: column; align-items: center; z-index: 20; 
            padding-top: 50px;
        }
        #song-list { 
            width: 100%; max-width: 400px; height: 75vh; 
            overflow-y: auto; padding: 20px; 
            box-sizing: border-box;
            scrollbar-width: thin; scrollbar-color: var(--pink) #111;
        }
        #song-list::-webkit-scrollbar { width: 8px; }
        #song-list::-webkit-scrollbar-thumb { background: var(--pink); border-radius: 10px; }

        .song-card { 
            background: #111; border: 2px solid var(--pink); border-radius: 10px; 
            padding: 15px; text-align: center; cursor: pointer; transition: 0.3s; margin-bottom: 15px; 
        }
        .song-card:hover { transform: scale(1.02); box-shadow: 0 0 15px currentColor; }
        .song-card.plazma { border-color: var(--neon-blue); color: var(--neon-blue); }
        .song-card.yuragi { border-color: var(--yuragi-purple); color: var(--yuragi-purple); }
        .song-card.whitelight { border-color: #fff; color: #fff; }
        
        /* ゲーム画面 */
        #game-container { position: relative; width: 440px; height: 600px; background: #000; border: 4px solid var(--pink); border-radius: 15px; overflow: hidden; }
        #game-bg { position: absolute; top: 0; left: 0; width: 100%; height: 100%; background-size: cover; background-position: center; opacity: 0; transition: 1s; z-index: 0; }
        .white-style #game-bg { opacity: 0.5; filter: brightness(1.2); }

        .lane-container { display: flex; height: 100%; position: relative; z-index: 1; background: rgba(0,0,0,0.4); }
        .lane { flex: 1; border-right: 1px solid rgba(255, 255, 255, 0.1); position: relative; }
        .judgment-line { position: absolute; bottom: 80px; width: 100%; height: 6px; background: var(--pink); z-index: 2; }
        .white-style .judgment-line { background: #fff; box-shadow: 0 0 20px #fff; }

        .note { position: absolute; width: 80%; left: 10%; height: 12px; background: #fff; border-radius: 4px; box-shadow: 0 0 10px #fff; z-index: 3; }
        .white-style .note { box-shadow: 0 0 15px #fff; border-radius: 0; }

        #ui { position: absolute; top: 50px; width: 100%; text-align: center; font-size: 1.8rem; font-weight: bold; z-index: 5; }
        #combo-container { position: absolute; top: 38%; width: 100%; text-align: center; pointer-events: none; z-index: 5; }
        #combo-num { font-size: 70px; font-weight: bold; margin: 0; }
        #flash-overlay { position: absolute; width: 100%; height: 100%; background: #fff; opacity: 0; pointer-events: none; z-index: 15; }
    </style>
</head>
<body>

    <div id="select-screen">
        <h1 style="letter-spacing: 5px; margin-bottom: 10px;">SELECT TRACK</h1>
        <div id="song-list">
            <div class="song-card" onclick="initGame('桜雨 by jjhs3213.wav', '桜雨')"><h3>桜雨</h3><p>Emotional Mode</p></div>
            <div class="song-card plazma" onclick="initGame('Plazma.mp3', 'Plazma')"><h3>Plazma</h3><p>Cyber / Enter Key</p></div>
            <div class="song-card yuragi" onclick="initGame('揺 by jjhs3213.wav', '揺')"><h3>揺</h3><p>Abstract / Wobble</p></div>
            <div class="song-card whitelight" onclick="initGame('whitelight.mp3', 'whitelight', 'https://images.unsplash.com/photo-1550684848-fac1c5b4e853?q=80&w=1000')">
                <h3>whitelight</h3><p>Flash / Background Mode</p>
            </div>
            
            <div class="song-card" onclick="initGame('track5.mp3', '桜雨')"><h3>Track 5</h3><p>New Song</p></div>
            <div class="song-card" onclick="initGame('track6.mp3', '桜雨')"><h3>Track 6</h3><p>New Song</p></div>
            <div class="song-card" onclick="initGame('track7.mp3', '桜雨')"><h3>Track 7</h3><p>New Song</p></div>
            <div class="song-card" onclick="initGame('track8.mp3', '桜雨')"><h3>Track 8</h3><p>New Song</p></div>
        </div>
    </div>

    <div id="game-container">
        <div id="game-bg"></div>
        <div id="flash-overlay"></div>
        <div id="ui">SCORE: <span id="score">000000</span></div>
        <div id="combo-container"><p id="combo-num"></p><div id="judge-text"></div></div>
        <div class="lane-container">
            <div class="lane" id="lane-0"></div><div class="lane" id="lane-1"></div>
            <div class="lane" id="lane-2"></div><div class="lane" id="lane-3"></div>
            <div id="judge-line" class="judgment-line"></div>
        </div>
    </div>

    <audio id="bgm"></audio>

    <script>
        const bgm = document.getElementById('bgm');
        const container = document.getElementById('game-container');
        const gameBg = document.getElementById('game-bg');
        const flash = document.getElementById('flash-overlay');
        const scoreEl = document.getElementById('score');
        const comboEl = document.getElementById('combo-num');
        const judgeEl = document.getElementById('judge-text');

        let score = 0, combo = 0, isPlaying = false, activeNotes = [], chart = [], currentMode = '';
        let noteSpeed = 500, judgmentY = 514;

        function initGame(src, mode, bgImage = '') {
            bgm.src = src;
            currentMode = mode;
            container.className = ''; 
            gameBg.style.backgroundImage = '';

            if(mode === 'Plazma') container.classList.add('plazma-style');
            if(mode === '揺') container.classList.add('yuragi-style');
            if(mode === 'whitelight') { 
                container.classList.add('white-style'); 
                noteSpeed = 750;
                if(bgImage) gameBg.style.backgroundImage = `url('${bgImage}')`;
            } else {
                noteSpeed = 550;
            }
            
            document.getElementById('select-screen').style.display = 'none';
            generateChart(mode);
            isPlaying = true;
            bgm.play();
            requestAnimationFrame(update);
        }

        function generateChart(mode) {
            chart = []; // リセット
            const endSeconds = 200; 
            const bpm = (mode === 'whitelight') ? 160 : 120;
            const secPerBeat = 60 / bpm;
            for (let beat = 0; beat <= (endSeconds/secPerBeat); beat += 0.25) {
                const time = beat * secPerBeat + 0.8;
                if (Math.random() < 0.35) {
                    chart.push({ time, lane: Math.floor(Math.random() * 4), type: 'normal' });
                }
            }
            chart.sort((a, b) => a.time - b.time);
        }

        function update() {
            if (!isPlaying) return;
            const now = bgm.currentTime;

            while (chart.length > 0 && chart[0].time <= now + 1.2) {
                const data = chart.shift();
                const el = document.createElement('div');
                el.className = 'note';
                document.getElementById(`lane-${data.lane}`).appendChild(el);
                activeNotes.push({ el, lane: data.lane, targetTime: data.time, hit: false });
            }

            for (let i = activeNotes.length - 1; i >= 0; i--) {
                const n = activeNotes[i];
                const diff = n.targetTime - now;
                n.el.style.top = (judgmentY - (diff * noteSpeed)) + 'px';
                
                if (currentMode === 'whitelight' && diff < 0.25) {
                    n.el.style.opacity = Math.max(0, diff * 4);
                }

                if (diff < -0.15 && !n.hit) {
                    n.hit = true;
                    combo = 0; comboEl.innerText = "";
                    judgeEl.innerText = "MISS"; judgeEl.style.color = "#888";
                }
                if (diff < -0.3) { n.el.remove(); activeNotes.splice(i, 1); }
            }
            requestAnimationFrame(update);
        }

        window.addEventListener('keydown', (e) => {
            const keys = { 'f': 0, 'g': 1, 'j': 2, 'k': 3 };
            const lane = keys[e.key.toLowerCase()];

            if (lane !== undefined && isPlaying) {
                const now = bgm.currentTime;
                const note = activeNotes.find(n => n.lane === lane && !n.hit);
                if (note) {
                    const diff = Math.abs(note.targetTime - now);
                    if (diff < 0.2) {
                        note.hit = true; note.el.style.display = 'none';
                        combo++; score += 100;
                        scoreEl.innerText = score.toString().padStart(6, '0');
                        comboEl.innerText = combo;
                        judgeEl.innerText = diff < 0.07 ? "PERFECT" : "GREAT";
                        judgeEl.style.color = "#fff";
                        
                        if(currentMode === 'whitelight') { 
                            flash.style.opacity = 0.4; 
                            setTimeout(()=>flash.style.opacity = 0, 40); 
                        }
                    }
                }
            }
        });
    </script>
</body>
</html>
