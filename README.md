# K-Parche
Página del podcast colombiano para parchar...
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Podcast Interactivo</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background: #f5f5f5;
            margin: 0;
            padding: 0;
        }
        header {
            background: #292929;
            color: white;
            padding: 20px;
            text-align: center;
            font-size: 24px;
        }
        .container {
            max-width: 900px;
            margin: auto;
            padding: 20px;
        }
        .upload-box {
            background: white;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 0 10px rgba(0,0,0,0.1);
            margin-bottom: 20px;
        }
        .upload-box input[type="file"] {
            margin-top: 10px;
        }
        audio {
            width: 100%;
            margin-top: 15px;
        }
        .audio-list {
            margin-top: 20px;
        }
        .audio-item {
            background: #ffffff;
            padding: 15px;
            border-radius: 10px;
            margin-bottom: 10px;
            box-shadow: 0 0 8px rgba(0,0,0,0.1);
        }
        button {
            padding: 10px 20px;
            border: none;
            cursor: pointer;
            background: #292929;
            color: white;
            border-radius: 8px;
            font-size: 14px;
        }
        button:hover {
            background: #444;
        }
    </style>
</head>
<body>
    <header>
        Plataforma Interactiva de Podcast
    </header>

    <div class="container">
        <div class="upload-box">
            <h3>Sube tu audio tipo podcast</h3>
            <input type="file" id="audioInput" accept="audio/*" />
            <button onclick="uploadAudio()">Subir</button>
        </div>

        <div class="audio-list" id="audioList">
            <h3>Audios Subidos</h3>
        </div>
    </div>

    <script>
        function uploadAudio() {
            const input = document.getElementById('audioInput');
            const list = document.getElementById('audioList');

            if (input.files.length === 0) {
                alert('Por favor selecciona un archivo de audio.');
                return;
            }

            const file = input.files[0];
            const url = URL.createObjectURL(file);

            const item = document.createElement('div');
            item.className = 'audio-item';
            item.innerHTML = `
                <p><strong>${file.name}</strong></p>
                <audio controls src="${url}"></audio>
            `;

            list.appendChild(item);

            input.value = "";
        }
    </script>
</body>
</html>

