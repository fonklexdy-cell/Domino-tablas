
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Desafío Arrastra - Pega</title>
    <style>
        body { background: #1a237e; color: white; font-family: 'Arial', sans-serif; text-align: center; margin: 0; padding: 20px; }
        #display-score { font-size: 2.5rem; color: #ffea00; margin: 10px; font-weight: bold; }
        #tablero { width: 95%; min-height: 220px; border: 6px dashed #ffea00; margin: 20px auto; 
                   display: flex; flex-wrap: wrap; gap: 15px; padding: 20px; background: #303f9f; border-radius: 20px; align-items: center; justify-content: center; }
        #reserva { display: flex; flex-wrap: wrap; gap: 15px; justify-content: center; padding: 20px; }
        .ficha { width: 130px; height: 85px; display: flex; flex-direction: column; align-items: center; justify-content: center; 
                 font-weight: bold; border-radius: 12px; cursor: grab; font-size: 1.2rem; box-shadow: 0 5px #000; color: white; user-select: none; }
        .ficha-linea { width: 80%; height: 2px; background-color: rgba(255, 255, 255, 0.6); margin: 4px 0; }
        .controles { margin: 20px; display: flex; justify-content: center; gap: 15px; }
        button { padding: 15px 30px; font-size: 1.5rem; cursor: pointer; border: none; border-radius: 10px; color: white; font-weight: bold; box-shadow: 0 4px #000; }
        button:active { transform: translateY(4px); box-shadow: none; }
        #btn-iniciar { background: #4caf50; }
        #btn-reiniciar { background: #f44336; }
        #mensaje-final { font-size: 2.2rem; color: #00e676; margin: 15px auto; font-weight: bold; display: none; text-shadow: 2px 2px 4px #000; }
        
        /* BOTÓN FLOTANTE MATEKEN2 */
        .btn-flotante { 
            position: fixed; bottom: 25px; right: 25px; background: #ffea00; color: #1a237e; 
            padding: 15px 22px; font-size: 1.1rem; font-weight: bold; text-decoration: none; 
            border-radius: 50px; box-shadow: 0 6px 15px rgba(0,0,0,0.4); display: flex; 
            align-items: center; gap: 10px; transition: transform 0.2s, background 0.2s; z-index: 9999;
        }
        .btn-flotante:hover { transform: scale(1.1); background: #fffde7; }
        .btn-flotante:active { transform: scale(0.95); }
    </style>
</head>
<body>

    <h1>🚀 ¡Desafío Arrastra - Pega! 🚀</h1>
    <div id="display-score">Puntaje: 0</div>
    
    <div class="controles">
        <button id="btn-iniciar" onclick="iniciarJuego()">Empezar Juego</button>
        <button id="btn-reiniciar" onclick="reiniciarJuego()" style="display: none;">Reiniciar</button>
    </div>

    <div id="mensaje-final"></div>
    
    <div id="tablero" ondrop="drop(event)" ondragover="allowDrop(event)"></div>
    <div id="reserva"></div>

    <!-- Botón Flotante a MateKen2 -->
    <a href="https://mateken2.netlify.app/" target="_blank" class="btn-flotante" onclick="visitarMateKen(event)">
        🤖 Ir a MateKen2
    </a>

    <script>
        let audioCtx = null;
        function playBeep(tipo) {
            if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
            const osc = audioCtx.createOscillator(), gain = audioCtx.createGain();
            osc.connect(gain); gain.connect(audioCtx.destination);
            const t = audioCtx.currentTime;

            if (tipo === 'click') {
                osc.frequency.setValueAtTime(600, t); gain.gain.setValueAtTime(0.1, t);
                gain.gain.exponentialRampToValueAtTime(0.01, t + 0.08); osc.start(t); osc.stop(t + 0.08);
            } else if (tipo === 'acierto') {
                osc.frequency.setValueAtTime(440, t); osc.frequency.setValueAtTime(587, t + 0.1);
                gain.gain.setValueAtTime(0.15, t); gain.gain.exponentialRampToValueAtTime(0.01, t + 0.25); osc.start(t); osc.stop(t + 0.25);
            } else if (tipo === 'error') {
                osc.type = 'sawtooth'; osc.frequency.setValueAtTime(180, t);
                gain.gain.setValueAtTime(0.2, t); gain.gain.exponentialRampToValueAtTime(0.01, t + 0.3); osc.start(t); osc.stop(t + 0.3);
            } else if (tipo === 'start') {
                osc.frequency.setValueAtTime(440, t); osc.frequency.setValueAtTime(554, t + 0.1); osc.frequency.setValueAtTime(660, t + 0.2);
                gain.gain.setValueAtTime(0.1, t); gain.gain.exponentialRampToValueAtTime(0.01, t + 0.35); osc.start(t); osc.stop(t + 0.35);
            } else if (tipo === 'teleport') {
                // Sonido galáctico especial para el botón flotante
                osc.type = 'triangle'; osc.frequency.setValueAtTime(200, t);
                osc.frequency.exponentialRampToValueAtTime(1200, t + 0.4);
                gain.gain.setValueAtTime(0.15, t); gain.gain.exponentialRampToValueAtTime(0.01, t + 0.4); osc.start(t); osc.stop(t + 0.4);
            }
        }

        const bancoOperaciones = [
            { op: "6x8", res: 48, col: "#f44336" }, { op: "4x7", res: 28, col: "#2196f3" },
            { op: "7x8", res: 56, col: "#8bc34a" }, { op: "2x7", res: 14, col: "#ffc107" },
            { op: "5x8", res: 40, col: "#4caf50" }, { op: "9x9", res: 81, col: "#ff9800" },
            { op: "3x9", res: 27, col: "#03a9f4" }, { op: "6x4", res: 24, col: "#e91e63" },
            { op: "2x4", res: 8,  col: "#9c27b0" }, { op: "5x5", res: 25, col: "#795548" },
            { op: "3x4", res: 12, col: "#607d8b" }, { op: "9x5", res: 45, col: "#e91e63" },
            { op: "7x3", res: 21, col: "#9c27b0" }, { op: "8x8", res: 64, col: "#ff5722" },
            { op: "6x6", res: 36, col: "#009688" }, { op: "3x6", res: 18, col: "#3f51b5" },
            { op: "7x7", res: 49, col: "#00bcd4" }, { op: "4x8", res: 32, col: "#495057" }
        ];

        let score = 0, buscandoValor = 0, fichasPartida = [], juegoActivo = false;

        function iniciarJuego() {
            playBeep('start');
            document.getElementById('btn-iniciar').style.display = 'none';
            document.getElementById('btn-reiniciar').style.display = 'inline-block';
            juegoActivo = true;
            generarEjercicioAleatorio();
        }

        function generarEjercicioAleatorio() {
            document.getElementById('tablero').innerHTML = "";
            document.getElementById('reserva').innerHTML = "";
            document.getElementById('mensaje-final').style.display = 'none';
            score = 0;
            document.getElementById('display-score').innerText = "Puntaje: " + score;

            let copia = [...bancoOperaciones].sort(() => Math.random() - 0.5);
            let seleccionadas = copia.slice(0, 10); 
            fichasPartida = [];
            
            for(let i = 0; i < seleccionadas.length; i++) {
                let valorFicha = (i === 0) ? Math.floor(Math.random() * 8) + 2 : seleccionadas[i-1].res;
                fichasPartida.push({
                    id: "ficha_" + i, valBuscar: valorFicha, numVisual: valorFicha,
                    opVisual: seleccionadas[i].op, proximoValor: seleccionadas[i].res, col: seleccionadas[i].col
                });
            }

            buscandoValor = fichasPartida[0].valBuscar;

            const fInicio = document.createElement('div');
            fInicio.className = 'ficha'; fInicio.style.backgroundColor = '#607d8b'; fInicio.style.cursor = 'default';
            fInicio.innerHTML = `<div>INICIO</div><div class="ficha-linea"></div><div>Busca: ${buscandoValor}</div>`;
            document.getElementById('tablero').appendChild(fInicio);

            let desordenadas = [...fichasPartida].sort(() => Math.random() - 0.5);
            desordenadas.forEach((f) => {
                const div = document.createElement('div');
                div.className = 'ficha'; div.id = f.id; div.draggable = true; div.style.backgroundColor = f.col; 
                div.innerHTML = `<div>${f.numVisual}</div><div class="ficha-linea"></div><div>${f.opVisual}</div>`;
                div.ondragstart = (ev) => { if(!juegoActivo) return; ev.dataTransfer.setData("text", ev.target.id); playBeep('click'); };
                document.getElementById('reserva').appendChild(div);
            });
        }

        function drop(ev) {
            ev.preventDefault();
            if (!juegoActivo) return;
            const idData = ev.dataTransfer.getData("text");
            const el = document.getElementById(idData);
            if (!el) return;

            const ficha = fichasPartida.find(f => f.id === idData);
            if (ficha && ficha.valBuscar === buscandoValor) {
                document.getElementById('tablero').appendChild(el);
                el.draggable = false; el.style.cursor = 'default';
                buscandoValor = ficha.proximoValor;
                score += 10;
                document.getElementById('display-score').innerText = "Puntaje: " + score;
                playBeep('acierto');
                if (document.getElementById('reserva').children.length === 0) finalizarJuego();
            } else {
                playBeep('error');
            }
        }

        function allowDrop(ev) { ev.preventDefault(); }

        function finalizarJuego() {
            juegoActivo = false;
            const msg = document.getElementById('mensaje-final');
            msg.innerText = `🏆 ¡Excelente! Cadena de 10 completada: ${score} pts 🚀`;
            msg.style.display = 'block';
            playBeep('acierto');
        }

        function reiniciarJuego() {
            playBeep('start');
            juegoActivo = true;
            generarEjercicioAleatorio();
        }

        function visitarMateKen(ev) {
            ev.preventDefault();
            playBeep('teleport');
            // Pequeño retraso para que alcance a sonar el efecto antes de redirigir
            setTimeout(() => {
                window.open("https://mateken2.netlify.app/", "_blank");
            }, 300);
        }
    </script>
</body>
</html>

