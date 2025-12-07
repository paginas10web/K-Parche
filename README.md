# K-Parche
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Reproductor de Audios desde GitHub</title>

    <style>
        body {
            font-family: Arial;
            background: #101820;
            color: white;
            padding: 20px;
        }
        .box {
            background: #1c2733;
            padding: 20px;
            border-radius: 10px;
            margin-bottom: 20px;
        }
        input, button {
            padding: 10px;
            border-radius: 8px;
            border: none;
            margin-top: 10px;
        }
        button {
            background: #4cc9f0;
            color: black;
            cursor: pointer;
        }
        button:hover {
            background: #3bb8dd;
        }
        audio {
            width: 100%;
            margin-top: 15px;
        }
    </style>
</head>
<body>

    <h1>Reproductor de Audios desde GitHub</h1>

    <!-- Subir audio local -->
    <div class="box">
        <h3>Sube un audio desde tu dispositivo</h3>
        <input type="file" id="localAudio" accept="audio/*">
        <button onclick="playLocalAudio()">Reproducir</button>
        <audio id="localPlayer" controls></audio>
    </div>

    <!-- Reproducir audio desde GitHub -->
    <div class="box">
        <h3>Reproducir audio alojado en GitHub</h3>
        <p>Pega aquí el enlace RAW del archivo mp3/subido a tu repositorio.</p>
        
        <input type="text" id="githubURL" 
               placeholder="https://raw.githubusercontent.com/usuario/repositorio/main/audio.mp3"
               style="width: 100%;">
        <button onclick="playGithubAudio()">Reproducir desde GitHub</button>

        <audio id="githubPlayer" controls></audio>
    </div>

<script>
function playLocalAudio() {
    const input = document.getElementById("localAudio");
    const player = document.getElementById("localPlayer");

    if (input.files.length === 0) {
        alert("Selecciona un archivo de audio.");
        return;
    }

    const file = input.files[0];
    const url = URL.createObjectURL(file);
    player.src = url;
}

function playGithubAudio() {
    const url = document.getElementById("githubURL").value;
    const player = document.getElementById("githubPlayer");

    if (!url.startsWith("https://raw.githubusercontent.com")) {
        alert("Debe ser un enlace RAW válido de GitHub.");
        return;
    }

    player.src = url;
}
</script>

</body>
</html>
