<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Panel de Audio Multicanal con Vista Previa</title>
    <style>
        /* Estilos generales */
        body { 
            margin: 0; 
            padding: 0; 
            font-family: Arial, sans-serif; 
            background-color: #f4f4f9; 
        }
        
        #reloj-contenedor {
            position: absolute; 
            top: 0px; 
            left: 0px; 
            background-color: #333; 
            color: #fff;
            padding: 10px 15px; 
            border-radius: 8px; 
            font-size: 1.2rem; 
            font-weight: bold; 
            box-shadow: 0 4px 6px rgba(0,0,0,0.1); 
            z-index: 10;
        }

        /* BARRA SUPERIOR */
        .barra-superior {
            position: absolute; 
            top: 15px; 
            left: 140px; 
            right: 40px; 
            height: 42px;       
            background-color: #4A90E2; 
            border-radius: 8px; 
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
            display: flex; 
            justify-content: space-between; 
            align-items: center; 
            padding: 0 15px; 
            color: #fff; 
            font-weight: bold;  
            z-index: 9;
        }

        .btn-navegacion {
            background-color: #333;
            color: white;
            border: none;
            padding: 6px 12px;
            border-radius: 4px;
            cursor: pointer;
            font-weight: bold;
            transition: 0.2s;
        }
        .btn-navegacion:hover { background-color: #111; }

        /* SECCIONES (PÁGINAS VIRTUALES) */
        .seccion {
            display: none; 
            flex-direction: column; 
            justify-content: center; 
            align-items: center; 
            height: 100vh; 
            text-align: center; 
            gap: 20px; 
        }
        .seccion.activa { display: flex; }

        /* CONTENEDORES DE BOTONES */
        .cuadro-botones { 
            background-color: rgba(255, 255, 255, 0.5); 
            padding: 20px; 
            border-radius: 12px; 
            box-shadow: 0 8px 16px rgba(0, 0, 0, 0.1); 
            display: grid; 
            grid-template-columns: repeat(4, 1fr); 
            gap: 12px; 
        }

        .boton-cuadrado { 
            width: 70px; 
            height: 70px; 
            background-color: #333; 
            color: #fff; 
            border: none; 
            border-radius: 8px; 
            font-size: 1rem; 
            font-weight: bold; 
            cursor: pointer; 
            transition: 0.2s; 
            box-shadow: 0 3px 5px rgba(0,0,0,0.2); 
        }
        .boton-cuadrado:hover { 
            background-color: #4A90E2; 
            transform: scale(1.05); 
        }

        /* Estados del botón */
        .boton-cuadrado.asignado { background-color: #28a745; }
        .boton-cuadrado.sonando {
            background-color: #ffc107 !important; 
            color: #000;
            box-shadow: 0 0 12px #ffc107;
        }
        .especial { border: 2px dashed #4A90E2; }

        /* VENTANA EMERGENTE CON VISTA PREVIA */
        .reproductor-popup {
            position: fixed;
            top: 50%; left: 50%;
            transform: translate(-50%, -50%);
            background: #ffffff;
            padding: 25px; border-radius: 12px;
            box-shadow: 0 10px 35px rgba(0,0,0,0.4);
            display: none; z-index: 1000; text-align: center; min-width: 340px;
        }

        .zona-corte { background: #f0f0f5; padding: 12px; border-radius: 8px; margin-bottom: 15px; text-align: left; }
        .zona-corte label { font-size: 0.85rem; font-weight: bold; color: #555; display: block; margin-top: 5px; }
        .zona-corte input[type="range"] { width: 100%; margin-bottom: 5px; }

        /* Botones de acción internos */
        .btn-accion { border: none; padding: 8px 15px; border-radius: 5px; cursor: pointer; font-weight: bold; margin: 5px; transition: 0.2s; }
        .btn-escuchar { background-color: #007bff; color: white; width: 93%; }
        .btn-escuchar.reproduciendo { background-color: #ffc107; color: black; }
        .btn-guardar { background-color: #28a745; color: white; }
        .btn-cancelar { background-color: #dc3545; color: white; }

        .consola-inferior {
            position: fixed; bottom: 15px; background: #333; color: #fff; padding: 8px 15px; border-radius: 20px;
            box-shadow: 0 4px 10px rgba(0,0,0,0.3); display: flex; gap: 15px; align-items: center; font-size: 0.85rem;
        }
        .btn-silenciar-todo { background: #ff5b5b; border: none; color: white; padding: 4px 10px; border-radius: 12px; cursor: pointer; font-weight: bold; }
    </style>
</head>
<body>

    <div id="reloj-contenedor">00:00:00</div>
    
    <div class="barra-superior">
        <span id="titulo-barra">Página Principal</span>
        <button id="btn-cambiar-pantalla" class="btn-navegacion" onclick="conmutarPantalla()">⚙️ Configurar Botones</button>
    </div>

    <div id="pantalla-principal" class="seccion activa">
        <h1>Mezclador de Audio Simultáneo</h1>
        <p>Los botones del 1 al 16 se intercalan. La fila inferior (17-20) se activa en paralelo al mismo tiempo.</p>
        <div class="cuadro-botones" id="contenedor-reproduccion"></div>
        
        <div class="consola-inferior">
            <span>🎛️ Mezcladora Activa</span>
            <button class="btn-silenciar-todo" onclick="detenerAbsolutamenteTodo()">🛑 Detener Todo</button>
        </div>
    </div>

    <div id="pantalla-configuracion" class="seccion">
        <h1>Asignar Sonidos a Canales</h1>
        <p>Elige cualquier botón para configurar su recorte con vista previa auditiva.</p>
        <div class="cuadro-botones" id="contenedor-configuracion"></div>
    </div>

    <input type="file" id="lector-archivos" accept="audio/mp3" style="display: none;" onchange="procesarArchivoEdicion(event)">

    <div id="popup-editor" class="reproductor-popup">
        <h3 id="editor-titulo-boton">Configurando Botón</h3>
        <p id="nombre-archivo-cargado" style="font-size:0.85rem; color:#666; margin-bottom:10px;"></p>
        
        <div class="zona-corte">
            <strong>✂️ Configurar Rango del Bucle:</strong>
            <label>Empezar en: <span id="val-inicio">0</span>s</label>
            <input type="range" id="corte-inicio" min="0" max="100" value="0" oninput="actualizarTextoCorte()">
            
            <label>Terminar y repetir en: <span id="val-fin">100</span>s</label>
            <input type="range" id="corte-fin" min="0" max="100" value="100" oninput="actualizarTextoCorte()">
        </div>

        <button id="btn-vista-previa" class="btn-accion btn-escuchar" onclick="conmutarVistaPrevia()">▶️ Escuchar fragmento</button>
        <br>
        <button class="btn-accion btn-guardar" onclick="guardarConfiguracionCompleta()">💾 Guardar en Canal</button>
        <button class="btn-accion btn-cancelar" onclick="cerrarEditor()">Cancelar</button>
    </div>

    <script>
        // --- RELOJ ---
        function actualizarReloj() {
            const ahora = new Date();
            let horas = ahora.getHours(), minutos = ahora.getMinutes(), segundos = ahora.getSeconds();
            horas = horas < 10 ? '0' + horas : horas; minutos = minutos < 10 ? '0' + minutes : minutos; segundos = segundos < 10 ? '0' + segundos : segundos;
            document.getElementById('reloj-contenedor').textContent = `${horas}:${minutos}:${segundos}`;
        }
        actualizarReloj(); setInterval(actualizarReloj, 1000);

        // --- BASE DE DATOS LOCAL (IndexedDB) ---
        let db;
        const request = indexedDB.open("BaseDatosAudioMulticanal", 1);
        request.onupgradeneeded = (e) => { db = e.target.result; db.createObjectStore("botones", { keyPath: "id" }); };
        request.onsuccess = (e) => { db = e.target.result; inicializarBotonesEInterfaz(); };

        let botonEnEdicion = null;
        let blobArchivoActual = null;
        let nombreArchivoActual = "";

        // NUEVO: Instancia de audio exclusiva para escuchar el preview dentro del popup
        let audioPreview = new Audio();
        let modoPreviewActivo = false;

        let reproductoresActivos = {}; 
        let limitesBucle = {}; 

        const panelRep = document.getElementById('contenedor-reproduccion');
        const panelConf = document.getElementById('contenedor-configuracion');

        function inicializarBotonesEInterfaz() {
            panelRep.innerHTML = ""; panelConf.innerHTML = "";
            const transaction = db.transaction(["botones"], "readonly");
            const store = transaction.objectStore("botones");
            const getAllRequest = store.getAll();

            getAllRequest.onsuccess = () => {
                const registros = getAllRequest.result;
                const asignados = {};
                registros.forEach(r => asignados[r.id] = true);

                for(let i = 1; i <= 20; i++) {
                    const tieneAudio = asignados[i] ? 'asignado' : '';
                    const esEspecial = i >= 17 ? 'especial' : '';

                    const btnPlay = document.createElement('button');
                    btnPlay.className = `boton-cuadrado ${tieneAudio} ${esEspecial}`;
                    btnPlay.textContent = i; btnPlay.id = `play-${i}`;
                    btnPlay.onclick = () => gestionarReproduccionMulticanal(i);
                    panelRep.appendChild(btnPlay);

                    const btnSetup = document.createElement('button');
                    btnSetup.className = `boton-cuadrado ${tieneAudio} ${esEspecial}`;
                    btnSetup.textContent = i;
                    btnSetup.onclick = () => abrirConfiguradorBoton(i);
                    panelConf.appendChild(btnSetup);
                }
            };
        }

        function conmutarPantalla() {
            const pPrincipal = document.getElementById('pantalla-principal');
            const pConfig = document.getElementById('pantalla-configuracion');
            const btnNav = document.getElementById('btn-cambiar-pantalla');
            const tituloBarra = document.getElementById('titulo-barra');
            detenerAbsolutamenteTodo();
            cerrarEditor();

            if (pPrincipal.classList.contains('activa')) {
                pPrincipal.classList.remove('activa'); pConfig.classList.add('activa');
                btnNav.textContent = "⬅️ Volver al Panel"; tituloBarra.textContent = "Página de Configuración";
            } else {
                pConfig.classList.remove('activa'); pPrincipal.classList.add('activa');
                btnNav.textContent = "⚙️ Configurar Botones"; tituloBarra.textContent = "Página Principal";
            }
        }

        // --- INTERFAZ DEL EDITOR ---
        function abrirConfiguradorBoton(numeroBoton) {
            botonEnEdicion = numeroBoton;
            document.getElementById('editor-titulo-boton').textContent = `Configurando Botón ${numeroBoton}`;
            document.getElementById('lector-archivos').click();
        }

        function procesarArchivoEdicion(event) {
            const archivo = event.target.files[0];
            if (archivo) {
                nombreArchivoActual = archivo.name;
                blobArchivoActual = archivo;

                const urlTemporal = URL.createObjectURL(archivo);
                audioPreview.src = urlTemporal; // Asignamos la pista al reproductor de pruebas

                audioPreview.onloadedmetadata = () => {
                    const maxSec = Math.floor(audioPreview.duration);
                    document.getElementById('corte-inicio').max = maxSec;
                    document.getElementById('corte-inicio').value = 0;
                    document.getElementById('corte-fin').max = maxSec;
                    document.getElementById('corte-fin').value = maxSec;

                    document.getElementById('nombre-archivo-cargado').textContent = `Archivo: ${archivo.name}`;
                    actualizarTextoCorte();
                    document.getElementById('popup-editor').style.display = "block";
                };
            }
        }

        function actualizarTextoCorte() {
            const inVal = document.getElementById('corte-inicio').value;
            const finVal = document.getElementById('corte-fin').value;
            document.getElementById('val-inicio').textContent = inVal;
            document.getElementById('val-fin').textContent = finVal;
            
            // Si el usuario mueve las barras mientras escucha, reacomodamos la aguja del preview
            if (modoPreviewActivo) {
                audioPreview.currentTime = parseInt(inVal);
            }
        }

        // NUEVO: Control dinámico para la Vista Previa Auditiva (Play / Pausa)
        function conmutarVistaPrevia() {
            const btnPreview = document.getElementById('btn-vista-previa');
            const inicio = parseInt(document.getElementById('corte-inicio').value);
            const fin = parseInt(document.getElementById('corte-fin').value);

            if (inicio >= fin) { alert("El inicio debe ser menor al final."); return; }

            if (!modoPreviewActivo) {
                // Iniciar reproducción de prueba
                modoPreviewActivo = true;
                btnPreview.textContent = "⏸️ Pausar Prueba";
                btnPreview.classList.add('reproduciendo');
                
                audioPreview.currentTime = inicio;
                audioPreview.play();
            } else {
                // Pausar prueba
                apagarAudioPreview();
            }
        }

        // NUEVO: Monitoreo constante del bucle exclusivo del Preview
        audioPreview.addEventListener('timeupdate', () => {
            if (modoPreviewActivo) {
                const inicio = parseInt(document.getElementById('corte-inicio').value);
                const fin = parseInt(document.getElementById('corte-fin').value);

                // Si en la prueba llega al tiempo final asignado, regresa al inicio de inmediato
                if (audioPreview.currentTime >= fin) {
                    audioPreview.currentTime = inicio;
                }
            }
        });

        function apagarAudioPreview() {
            audioPreview.pause();
            modoPreviewActivo = false;
            const btnPreview = document.getElementById('btn-vista-previa');
            if(btnPreview) {
                btnPreview.textContent = "▶️ Escuchar fragmento";
                btnPreview.classList.remove('reproduciendo');
            }
        }

        function guardarConfiguracionCompleta() {
            const inicio = parseInt(document.getElementById('corte-inicio').value);
            const fin = parseInt(document.getElementById('corte-fin').value);

            if (inicio >= fin) { alert("El inicio debe ser menor al final."); return; }

            apagarAudioPreview(); // Apaga la prueba antes de guardar

            const transaction = db.transaction(["botones"], "readwrite");
            const store = transaction.objectStore("botones");

            store.put({
                id: botonEnEdicion,
                nombre: nombreArchivoActual,
                inicio: inicio,
                fin: fin,
                archivoBlob: blobArchivoActual
            });

            transaction.oncomplete = () => {
                alert(`¡Audio inyectado en canal ${botonEnEdicion}!`);
                cerrarEditor();
                inicializarBotonesEInterfaz();
            };
        }

        function cerrarEditor() {
            apagarAudioPreview();
            document.getElementById('popup-editor').style.display = "none";
            document.getElementById('lector-archivos').value = "";
        }

        // ================= MULTICANAL PRINCIPAL =================
        function gestionarReproduccionMulticanal(idBoton) {
            if (reproductoresActivos[idBoton]) { apagarCanalEspecifico(idBoton); return; }

            const transaction = db.transaction(["botones"], "readonly");
            const store = transaction.objectStore("botones");
            const requestGet = store.get(idBoton);

            requestGet.onsuccess = () => {
                const data = requestGet.result;
                if (!data) { alert(`El canal ${idBoton} no tiene audio.`); return; }

                if (idBoton <= 16) {
                    for (let i = 1; i <= 16; i++) { if (reproductoresActivos[i]) apagarCanalEspecifico(i); }
                }

                const nuevoAudio = new Audio(URL.createObjectURL(data.archivoBlob));
                nuevoAudio.currentTime = data.inicio;
                
                limitesBucle[idBoton] = { inicio: data.inicio, fin: data.fin };
                reproductoresActivos[idBoton] = nuevoAudio;
                document.getElementById(`play-${idBoton}`).classList.add('sonando');

                nuevoAudio.addEventListener('timeupdate', () => {
                    if (nuevoAudio.currentTime >= limitesBucle[idBoton].fin) {
                        nuevoAudio.currentTime = limitesBucle[idBoton].inicio;
                        nuevoAudio.play();
                    }
                });
                nuevoAudio.play();
            };
        }

        function apagarCanalEspecifico(idBoton) {
            if (reproductoresActivos[idBoton]) {
                reproductoresActivos[idBoton].pause();
                reproductoresActivos[idBoton].src = "";
                delete reproductoresActivos[idBoton]; delete limitesBucle[idBoton];
                document.getElementById(`play-${idBoton}`).classList.remove('sonando');
            }
        }

        function detenerAbsolutamenteTodo() {
            Object.keys(reproductoresActivos).forEach(id => { apagarCanalEspecifico(id); });
            apagarAudioPreview();
        }
    </script>
</body>
</html>
