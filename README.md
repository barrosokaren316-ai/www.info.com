<!doctype html>

<html lang="es">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Página para informarte</title>
  <meta name="description" content="Página simple para que visitantes te informen (envíen novedades, avisos o feedback)." />
  <!-- Tailwind Play CDN (prototipo, no para producción) -->
  <script src="https://cdn.tailwindcss.com"></script>
  <style>
    /* Pequeñas mejoras visuales */
    .glass { background: rgba(255,255,255,0.06); backdrop-filter: blur(6px); }
  </style>
</head>
<body class="bg-gradient-to-b from-slate-900 to-slate-800 text-slate-100 min-h-screen">
  <header class="max-w-4xl mx-auto p-6 text-center">
    <h1 class="text-3xl md:text-4xl font-semibold">Página para informarte</h1>
    <p class="mt-2 text-slate-300">Usa este formulario para enviar noticias, avisos o información que quieras que reciba.</p>
  </header>  <main class="max-w-4xl mx-auto px-6 pb-12">
    <section class="glass p-6 rounded-2xl shadow-lg">
      <h2 class="text-2xl font-medium">Enviar una notificación</h2>
      <p class="text-slate-300 mt-1">Rellena los campos y pulsa <strong>Enviar</strong>. Las entradas se guardan en tu navegador (prototipo).</p><form id="notifyForm" class="mt-4 grid gap-4">
    <div>
      <label class="block text-sm text-slate-200">Nombre (opcional)</label>
      <input id="name" type="text" placeholder="Tu nombre" class="mt-1 w-full rounded-lg p-3 bg-slate-900 text-slate-100 border border-slate-700 focus:outline-none" />
    </div>

    <div>
      <label class="block text-sm text-slate-200">Asunto</label>
      <input id="subject" type="text" placeholder="Asunto breve" required class="mt-1 w-full rounded-lg p-3 bg-slate-900 text-slate-100 border border-slate-700 focus:outline-none" />
    </div>

    <div>
      <label class="block text-sm text-slate-200">Mensaje</label>
      <textarea id="message" rows="5" placeholder="Escribe aquí lo que quieras informarme..." required class="mt-1 w-full rounded-lg p-3 bg-slate-900 text-slate-100 border border-slate-700 focus:outline-none"></textarea>
    </div>

    <div class="flex gap-2 items-center">
      <input id="private" type="checkbox" />
      <label for="private" class="text-sm text-slate-300">Marcar como privado (no se mostrará públicamente)</label>
    </div>

    <div class="flex gap-2">
      <button type="submit" class="px-4 py-2 rounded-xl bg-blue-600 hover:bg-blue-500 font-medium">Enviar</button>
      <button id="clearBtn" type="button" class="px-4 py-2 rounded-xl bg-slate-700 hover:bg-slate-600">Borrar formulario</button>
    </div>
  </form>

  <div id="status" class="mt-3 text-sm"></div>
</section>

<section class="glass p-6 rounded-2xl shadow-lg mt-6">
  <h2 class="text-2xl font-medium">Últimas notificaciones</h2>
  <p class="text-slate-300 mt-1">Aquí se muestran las notificaciones guardadas en este navegador (solo tú las ves). Para que otros las envíen a un servidor, véase la sección "Cómo publicar" abajo.</p>

  <ul id="list" class="mt-4 space-y-3"></ul>

  <div class="mt-4 flex gap-2">
    <button id="exportBtn" class="px-3 py-2 rounded-lg bg-emerald-600 hover:bg-emerald-500">Exportar JSON</button>
    <button id="importBtn" class="px-3 py-2 rounded-lg bg-amber-600 hover:bg-amber-500">Importar JSON</button>
    <input id="fileInput" type="file" accept="application/json" class="hidden" />
  </div>
</section>

<section class="glass p-6 rounded-2xl shadow-lg mt-6">
  <h2 class="text-2xl font-medium">Cómo publicar (opcional)</h2>
  <ol class="list-decimal list-inside text-slate-300 mt-2">
    <li>Sube este archivo a GitHub y activa GitHub Pages (o súbelo a Netlify/Vercel).</li>
    <li>Si quieres recibir notificaciones en un servidor real, conecta el formulario a una API (fetch POST) en lugar de usar localStorage.</li>
    <li>Si quieres añadir autenticación o guardar en una base de datos, te puedo dar el ejemplo de backend (Node/Express o Firebase).</li>
  </ol>
</section>

<footer class="text-center text-slate-400 mt-8 pb-6">
  <small>Hecho con ❤️ — personaliza el texto y estilos como prefieras.</small>
</footer>

  </main>  <script>
    // Lógica simple: guarda/lee del localStorage
    const form = document.getElementById('notifyForm');
    const listEl = document.getElementById('list');
    const status = document.getElementById('status');
    const clearBtn = document.getElementById('clearBtn');
    const exportBtn = document.getElementById('exportBtn');
    const importBtn = document.getElementById('importBtn');
    const fileInput = document.getElementById('fileInput');

    function loadItems() {
      const raw = localStorage.getItem('informes') || '[]';
      try {
        return JSON.parse(raw);
      } catch(e) {
        return [];
      }
    }

    function saveItems(items) {
      localStorage.setItem('informes', JSON.stringify(items));
    }

    function render() {
      const items = loadItems();
      listEl.innerHTML = '';
      if (!items.length) {
        listEl.innerHTML = '<li class="text-slate-400">(No hay notificaciones aún)</li>';
        return;
      }
      items.slice().reverse().forEach((it, idx) => {
        const li = document.createElement('li');
        li.className = 'p-3 rounded-lg bg-slate-900 border border-slate-700';
        li.innerHTML = `<div class="flex justify-between"><div><strong>${escapeHtml(it.subject)}</strong> <span class="text-slate-400 text-sm">por ${escapeHtml(it.name||'Anónimo')}</span></div><div class="text-xs text-slate-400">${new Date(it.date).toLocaleString()}</div></div><p class="mt-2 text-slate-300">${escapeHtml(it.message)}</p>`;
        if (it.private) {
          const badge = document.createElement('div');
          badge.className = 'mt-2 text-xs text-amber-400';
          badge.textContent = 'Privado';
          li.appendChild(badge);
        }
        listEl.appendChild(li);
      });
    }

    function escapeHtml(s){
      if (!s) return '';
      return String(s).replace(/[&<>\"']/g, function(c){
        return {'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;','\'":"&#39;'}[c];
      });
    }

    form.addEventListener('submit', e => {
      e.preventDefault();
      const name = document.getElementById('name').value.trim();
      const subject = document.getElementById('subject').value.trim();
      const message = document.getElementById('message').value.trim();
      const priv = document.getElementById('private').checked;
      if (!subject || !message) {
        status.textContent = 'Asunto y mensaje son obligatorios.';
        return;
      }
      const items = loadItems();
      items.push({ name, subject, message, private: priv, date: new Date().toISOString() });
      saveItems(items);
      status.textContent = 'Enviado. Guardado localmente.';
      form.reset();
      render();
      setTimeout(()=> status.textContent='', 3000);
    });

    clearBtn.addEventListener('click', ()=>{
      form.reset();
      status.textContent = '';
    });

    exportBtn.addEventListener('click', ()=>{
      const items = loadItems();
      const blob = new Blob([JSON.stringify(items, null, 2)], {type:'application/json'});
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url;
      a.download = 'informes.json';
      a.click();
      URL.revokeObjectURL(url);
    });

    importBtn.addEventListener('click', ()=> fileInput.click());
    fileInput.addEventListener('change', async (ev)=>{
      const f = ev.target.files[0];
      if (!f) return;
      try{
        const text = await f.text();
        const imported = JSON.parse(text);
        if (!Array.isArray(imported)) throw new Error('Formato inválido');
        saveItems(imported);
        render();
        status.textContent = 'Importación completada.';
      } catch(e) {
        status.textContent = 'Error al importar: ' + e.message;
      }
      setTimeout(()=> status.textContent='',3000);
    });

    // Primer render
    render();
  </script></body>
</html>
