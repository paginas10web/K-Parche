# K-Parche
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
    const audioList = document.getElementById('audioList');
    const audioKey = "audiosPodcast";

    // Cargar audios guardados
    window.onload = function () {
        const savedAudios = JSON.parse(localStorage.getItem(audioKey)) || [];
        savedAudios.forEach(audio => addAudioToList(name, url) {
        const item = document.createElement('div');
        item.className = 'audio-item';
        item.innerHTML = `
            <p><strong>${name}</strong></p>
            <audio controls src="${url}"></audio>
            <button class="delete-btn">Eliminar</button>
        `;

        item.querySelector('.delete-btn').addEventListener('click', () => {
            deleteAudio(name, url);
            item.remove();
        });

        audioList.appendChild(item);
    }

    function uploadAudio() {
        const input = document.getElementById('audioInput');

        if (input.files.length === 0) {
            alert('Por favor selecciona un archivo de audio.');
            return;
        }

        const file = input.files[0];
        const url = URL.createObjectURL(file);

        addAudioToList(file.name, url);
        saveAudio(file.name, url);

        input.value = "";
    }

    function addAudioToList(name, url) {
        const item = document.createElement('div');
        item.className = 'audio-item';
        item.innerHTML = `
            <p><strong>${name}</strong></p>
            <audio controls src="${url}"></audio>
        `;
        audioList.appendChild(item);
    }

    function saveAudio(name, url) {(name, url) {
        const savedAudios = JSON.parse(localStorage.getItem(audioKey)) || [];
        savedAudios.push({ name, url });
        localStorage.setItem(audioKey, JSON.stringify(savedAudios));
    }
    function deleteAudio(name, url) {
        let savedAudios = JSON.parse(localStorage.getItem(audioKey)) || [];
        savedAudios = savedAudios.filter(a => !(a.name === name && a.url === url));
        localStorage.setItem(audioKey, JSON.stringify(savedAudios));
    }
</script>
</body>
</html>

