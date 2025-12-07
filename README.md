# K-Parche
<!doctype html>
<html lang="es">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Podcast - Subir y guardar en la nube</title>
  <style>
    :root{--bg:#0f1724;--card:#0b1220;--accent:#06b6d4;--muted:#9aa4b2}
    *{box-sizing:border-box}
    body{margin:0;font-family:Inter,system-ui,Arial;background:linear-gradient(180deg,#071025 0%, #0b1b2b 100%);color:#e6eef6;min-height:100vh;padding:24px}
    .wrap{max-width:920px;margin:0 auto}
    header{display:flex;align-items:center;justify-content:space-between;margin-bottom:18px}
    h1{font-size:20px;margin:0}
    .card{background:rgba(255,255,255,0.03);padding:18px;border-radius:12px;box-shadow:0 6px 30px rgba(2,6,23,0.6);margin-bottom:18px}
    input[type=file]{display:block;margin-top:10px}
    .controls{display:flex;gap:8px;margin-top:12px;flex-wrap:wrap}
    button{background:var(--accent);color:#042027;border:0;padding:10px 12px;border-radius:8px;cursor:pointer}
    button.ghost{background:transparent;border:1px solid rgba(255,255,255,0.06);color:var(--muted)}
    .progress{height:8px;background:rgba(255,255,255,0.06);border-radius:8px;overflow:hidden;margin-top:12px}
    .progress > i{display:block;height:100%;width:0;background:linear-gradient(90deg,#06b6d4,#3b82f6)}
    .list{display:grid;gap:12px;margin-top:14px}
    .item{display:flex;align-items:center;gap:12px;background:rgba(255,255,255,0.02);padding:12px;border-radius:10px}
    .meta{flex:1}
    .meta p{margin:0;font-size:13px;color:#dfe8f2}
    .meta small{display:block;color:var(--muted);margin-top:6px}
    audio{width:260px}
    .danger{background:#ff6b6b;color:#2b0f0f}
    footer{color:var(--muted);font-size:13px;margin-top:18px}
    .note{font-size:13px;color:var(--muted);margin-top:8px}
  </style>
</head>
<body>
  <div class="wrap">
    <header>
      <h1>Podcast (subir → guardar en la nube)</h1>
      <div style="font-size:13px;color:#9aa4b2">Build: Firebase Storage + Firestore</div>
    </header>

    <div class="card" id="uploadCard">
      <strong>Sube un audio (mp3, wav, m4a, ...)</strong>
      <div class="note">Al subir, el archivo se guarda en Firebase Storage y su registro en Firestore. Tras recargar la página los audios seguirán disponibles.</div>

      <input id="fileInput" type="file" accept="audio/*" />
      <div class="controls">
        <button id="uploadBtn">Subir audio</button>
        <button class="ghost" id="clearLocalBtn">Borrar lista local (no borra en nube)</button>
      </div>

      <div class="progress" style="display:none" id="progressWrap"><i id="progressBar"></i></div>
      <div id="status" class="note"></div>
    </div>

    <div class="card">
      <strong>Audios guardados en la nube</strong>
      <div class="note">Lista sincronizada con Firestore. Puedes reproducir o eliminar (eliminar borra Storage y el registro).</div>
      <div id="list" class="list"></div>
    </div>

    <footer>
      <div>Instrucciones: reemplaza <code>firebaseConfig</code> por tu configuración de Firebase (ver consola Firebase → app web).</div>
      <div style="margin-top:8px;color:var(--muted)">Recuerda proteger tu proyecto antes de hacerlo público (reglas de seguridad).</div>
    </footer>
  </div>

  <!-- Firebase (compat) SDKs vía CDN: usando compat para que funcione directamente en un HTML estático -->
  <script src="https://www.gstatic.com/firebasejs/9.22.2/firebase-app-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.22.2/firebase-storage-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.22.2/firebase-firestore-compat.js"></script>

  <script>
  /*************************************************************************
   * 1) REEMPLAZA esta configuración con la tuya (Firebase Console → app)
   *************************************************************************/
  const firebaseConfig = {
    apiKey: "TU_API_KEY",
    authDomain: "TU_AUTH_DOMAIN",
    projectId: "TU_PROJECT_ID",
    storageBucket: "TU_STORAGE_BUCKET.appspot.com",
    appId: "TU_APP_ID"
    // demás campos según Firebase (opcional)
  };

  // Inicializar Firebase
  firebase.initializeApp(firebaseConfig);
  const storage = firebase.storage();
  const db = firebase.firestore();

  // Referencias UI
  const fileInput = document.getElementById('fileInput');
  const uploadBtn = document.getElementById('uploadBtn');
  const listEl = document.getElementById('list');
  const progressWrap = document.getElementById('progressWrap');
  const progressBar = document.getElementById('progressBar');
  const status = document.getElementById('status');
  const clearLocalBtn = document.getElementById('clearLocalBtn');

  // Colección Firestore donde guardaremos metadatos
  const COLLECTION = 'podcast_audios';

  // Cargar la lista desde Firestore al inicio
  window.addEventListener('load', loadListFromFirestore);

  uploadBtn.addEventListener('click', async () => {
    const file = fileInput.files[0];
    if (!file) { alert('Selecciona un archivo de audio primero'); return; }

    try {
      uploadBtn.disabled = true;
      status.textContent = 'Iniciando subida...';
      const timestamp = Date.now();
      // crear path único en Storage
      const path = `audios/${timestamp}_${sanitizeFilename(file.name)}`;
      const storageRef = storage.ref(path);

      const uploadTask = storageRef.put(file);

      progressWrap.style.display = 'block';
      uploadTask.on('state_changed',
        snapshot => {
          const pct = Math.round((snapshot.bytesTransferred / snapshot.totalBytes) * 100);
          progressBar.style.width = pct + '%';
          status.textContent = `Subiendo: ${pct}%`;
        },
        error => {
          console.error('Upload error', error);
          status.textContent = 'Error al subir: ' + error.message;
          uploadBtn.disabled = false;
        },
        async () => {
          // subida completa
          const downloadURL = await storageRef.getDownloadURL();
          // guardar metadata en Firestore
          const doc = {
            name: file.name,
            path,
            url: downloadURL,
            size: file.size,
            type: file.type,
            uploadedAt: firebase.firestore.FieldValue.serverTimestamp()
          };
          const res = await db.collection(COLLECTION).add(doc);
          status.textContent = 'Subida completa ✅';
          progressBar.style.width = '0%';
          progressWrap.style.display = 'none';
          fileInput.value = '';
          uploadBtn.disabled = false;
          // actualizar lista en UI
          addItemToUI(res.id, { id: res.id, ...doc, uploadedAt: new Date() });
        }
      );
    } catch (err) {
      console.error(err);
      status.textContent = 'Error: ' + err.message;
      uploadBtn.disabled = false;
    }
  });

  clearLocalBtn.addEventListener('click', () => {
    // No hay "lista local" persistente aparte de Firestore; podemos recargar la lista
    if (confirm('Recargar la lista desde la nube?')) loadListFromFirestore();
  });

  // cargar y renderizar la lista desde Firestore
  async function loadListFromFirestore(){
    status.textContent = 'Cargando lista...';
    listEl.innerHTML = '';
    try {
      const snapshot = await db.collection(COLLECTION).orderBy('uploadedAt', 'desc').get();
      if (snapshot.empty) {
        listEl.innerHTML = '<div class="note" style="padding:8px">No hay audios aún.</div>';
        status.textContent = '';
        return;
      }
      snapshot.forEach(doc => {
        const data = doc.data();
        addItemToUI(doc.id, { id: doc.id, ...data });
      });
      status.textContent = '';
    } catch (err) {
      console.error(err);
      status.textContent = 'No se pudo cargar la lista: ' + err.message;
    }
  }

  // añadir un item al DOM (sin duplicados)
  function addItemToUI(id, data) {
    if (document.getElementById('item-' + id)) return; // ya está

    const item = document.createElement('div');
    item.className = 'item';
    item.id = 'item-' + id;

    const meta = document.createElement('div');
    meta.className = 'meta';

    const title = document.createElement('p');
    title.textContent = data.name || 'Audio ' + id;
    const info = document.createElement('small');
    const uploaded = data.uploadedAt && data.uploadedAt.toDate ? data.uploadedAt.toDate().toLocaleString() : (data.uploadedAt ? new Date(data.uploadedAt.seconds*1000).toLocaleString() : '');
    info.textContent = `${formatBytes(data.size||0)} • ${uploaded}`;

    meta.appendChild(title);
    meta.appendChild(info);

    const audioEl = document.createElement('audio');
    audioEl.controls = true;
    audioEl.src = data.url;

    const deleteBtn = document.createElement('button');
    deleteBtn.className = 'danger';
    deleteBtn.textContent = 'Eliminar';
    deleteBtn.addEventListener('click', () => onDelete(id, data.path));

    item.appendChild(meta);
    item.appendChild(audioEl);
    item.appendChild(deleteBtn);

    listEl.appendChild(item);
  }

  // eliminar audio: borrar Storage y luego la entrada Firestore
  async function onDelete(id, storagePath){
    if (!confirm('¿Eliminar este audio permanentemente?')) return;
    try {
      status.textContent = 'Eliminando...';
      // borrar en Storage
      const ref = storage.ref(storagePath);
      await ref.delete();
      // borrar registro en Firestore
      await db.collection(COLLECTION).doc(id).delete();
      // remover del DOM
      const el = document.getElementById('item-' + id);
      if (el) el.remove();
      status.textContent = 'Audio eliminado';
    } catch (err) {
      console.error(err);
      status.textContent = 'Error al eliminar: ' + err.message;
    }
  }

  // utilidades
  function formatBytes(bytes){
    if (bytes === 0) return '0 B';
    const k = 1024, sizes = ['B','KB','MB','GB','TB'];
    const i = Math.floor(Math.log(bytes) / Math.log(k));
    return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i];
  }
  function sanitizeFilename(name){
    return name.replace(/[^a-zA-Z0-9.\-_]/g, '_');
  }
  </script>
</body>
</html>
