<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Pedro Responde - Master Secuencial</title>
    <style>
        :root {
            --bg: #0d0d0f;
            --card: #16161a;
            --gold: #d4af37;
            --text: #ffffff;
            --border: #2a2a2e;
        }

        body {
            background-color: var(--bg);
            color: var(--text);
            font-family: 'Segoe UI', sans-serif;
            margin: 0;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            height: 100vh;
            overflow: hidden;
        }

        .trigger {
            position: fixed;
            width: 60px;
            height: 60px;
            z-index: 100;
        }
        #trig-preg { top: 0; right: 0; }
        #trig-fe { top: 0; left: 50%; transform: translateX(-50%); }

        .app-container {
            width: 85%;
            max-width: 400px;
            text-align: center;
        }

        h1 { color: var(--gold); letter-spacing: 3px; font-weight: 300; margin-bottom: 40px; }

        input {
            width: 100%;
            padding: 15px;
            margin-bottom: 20px;
            background: var(--card);
            border: 1px solid var(--border);
            border-radius: 10px;
            color: white;
            box-sizing: border-box;
            font-size: 16px;
            outline: none;
        }

        .action-row {
            display: flex;
            gap: 10px;
        }

        .btn-main {
            flex: 4;
            background: var(--gold);
            color: black;
            border: none;
            padding: 15px;
            border-radius: 10px;
            font-weight: bold;
            font-size: 16px;
        }

        .btn-reset {
            flex: 1;
            background: #222;
            color: var(--gold);
            border: 1px solid var(--gold);
            border-radius: 10px;
            font-size: 20px;
            cursor: pointer;
        }

        #output {
            margin-top: 30px;
            font-style: italic;
            color: var(--gold);
            min-height: 80px;
            line-height: 1.4;
            padding: 0 10px;
        }

        .modal {
            display: none;
            position: fixed;
            top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.95);
            z-index: 200;
            padding: 20px;
            box-sizing: border-box;
        }

        .modal-content {
            background: var(--card);
            padding: 20px;
            border-radius: 15px;
            border: 1px solid var(--gold);
        }

        textarea {
            width: 100%;
            height: 150px;
            background: #000;
            color: #0f0;
            border: 1px solid #333;
            margin-bottom: 15px;
            padding: 10px;
            box-sizing: border-box;
            font-family: monospace;
        }

        .admin-btn {
            width: 100%;
            padding: 12px;
            margin-bottom: 10px;
            border: none;
            border-radius: 5px;
            font-weight: bold;
            cursor: pointer;
        }

        .b-del { background: #dc3545; color: white; }
        .b-cls { background: #6c757d; color: white; }
        .b-reset-seq { background: #28a745; color: white; }
    </style>
</head>
<body>

    <div id="trig-fe" class="trigger" onclick="abrir('m-fe')"></div>
    <div id="trig-preg" class="trigger" onclick="abrir('m-preg')"></div>

    <div class="app-container">
        <h1>PEDRO RESPONDE</h1>
        <input type="text" id="peticion" placeholder="Petición" autocomplete="off">
        <input type="text" id="pregunta" placeholder="Pregunta" autocomplete="off">
        <div class="action-row">
            <button class="btn-main" onclick="consultar()">CONSULTAR</button>
            <button class="btn-reset" onclick="limpiarPantalla()">✎</button>
        </div>
        <div id="output"></div>
    </div>

    <div id="m-fe" class="modal">
        <div class="modal-content">
            <h3>Frases de Fe (Secuenciales)</h3>
            <textarea id="db-fe"></textarea>
            <button class="admin-btn b-reset-seq" onclick="reiniciarContador()">REINICIAR SECUENCIA</button>
            <button class="admin-btn b-del" onclick="borrarDB('fe')">BORRAR TODO</button>
            <button class="admin-btn b-cls" onclick="cerrar('m-fe')">CERRAR</button>
        </div>
    </div>

    <div id="m-preg" class="modal">
        <div class="modal-content">
            <h3>Historial</h3>
            <div id="log" style="height: 200px; overflow-y: auto; text-align: left; font-size: 12px; color: #aaa;"></div>
            <button class="admin-btn b-del" onclick="borrarHistorial()">BORRAR</button>
            <button class="admin-btn b-cls" onclick="cerrar('m-preg')">CERRAR</button>
        </div>
    </div>

    <script>
        const LLAVE = "Pedro por favor responde";

        window.onload = () => {
            document.getElementById('db-fe').value = localStorage.getItem('p_fe') || "Frase 1\nFrase 2";
            renderLog();
        };

        function abrir(id) { document.getElementById(id).style.display = 'block'; }
        
        function cerrar(id) { 
            document.getElementById(id).style.display = 'none'; 
            localStorage.setItem('p_fe', document.getElementById('db-fe').value);
        }

        function borrarDB(tipo) {
            if(confirm("¿Borrar datos?")) {
                document.getElementById('db-fe').value = "";
                localStorage.setItem('p_idx_fe', '0');
            }
        }

        function reiniciarContador() {
            localStorage.setItem('p_idx_fe', '0');
            alert("Secuencia reiniciada. Pedro responderá desde la primera frase.");
        }

        function limpiarPantalla() {
            document.getElementById('output').innerText = "";
            document.getElementById('peticion').value = "";
            document.getElementById('pregunta').value = "";
        }

        function consultar() {
            const pet = document.getElementById('peticion').value.trim();
            const pre = document.getElementById('pregunta').value.trim();
            const out = document.getElementById('output');

            // BLOQUEO: Solo responde si la pantalla está limpia (sin texto previo en output)
            if (out.innerText !== "") {
                return; 
            }

            if(!pet || !pre) return; 

            saveLog(pre);

            if(pet === LLAVE) {
                out.innerText = getFeSecuencial();
            } else {
                out.innerText = "No tienes suficiente fe.";
            }
        }

        function getFeSecuencial() {
            const frases = document.getElementById('db-fe').value.split('\n').filter(f => f.trim() !== "");
            if (frases.length === 0) return "No tienes suficiente fe.";

            let idx = parseInt(localStorage.getItem('p_idx_fe')) || 0;

            // Al final de la lista, muestra el nuevo mensaje de reordenamiento espiritual
            if (idx >= frases.length) {
                return "No insistas, no pongas en peligro nuestra comunión";
            }

            const respuesta = frases[idx];
            localStorage.setItem('p_idx_fe', (idx + 1).toString());
            return respuesta;
        }

        function saveLog(q) {
            let h = JSON.parse(localStorage.getItem('p_log') || "[]");
            h.push(q);
            localStorage.setItem('p_log', JSON.stringify(h));
            renderLog();
        }

        function renderLog() {
            const h = JSON.parse(localStorage.getItem('p_log') || "[]");
            document.getElementById('log').innerHTML = h.reverse().map(q => `> ${q}`).join('<br>');
        }

        function borrarHistorial() {
            localStorage.removeItem('p_log');
            renderLog();
        }
    </script>
</body>
</html>
<link rel="manifest" href="manifest.json">
