<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>To-Do List — Single File (PWA + Export/Import + Drag)</title>
<meta name="theme-color" content="#071233" />
<style>
  :root{
    --bg: #0f172a;
    --card: #0b1220;
    --muted: #9aa4b2;
    --accent: #60a5fa;
    --accent-2: #7c3aed;
    --success: #10b981;
    --danger: #ef4444;
    --glass: rgba(255,255,255,0.03);
  }
  *{box-sizing:border-box}
  html,body{height:100%}
  body{
    margin:0;
    font-family:Inter,ui-sans-serif,system-ui,-apple-system,"Segoe UI",Roboto,"Helvetica Neue",Arial;
    background:
      radial-gradient(1200px 600px at 10% 10%, rgba(124,58,237,0.12), transparent 10%),
      radial-gradient(900px 500px at 90% 90%, rgba(96,165,250,0.08), transparent 10%),
      linear-gradient(180deg, #020617 0%, #071233 100%);
    color: #e6eef8;
    display:flex;
    align-items:center;
    justify-content:center;
    padding:24px;
  }

  .app {
    width:100%;
    max-width:880px;
    background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));
    border-radius:14px;
    padding:18px;
    box-shadow: 0 10px 30px rgba(2,6,23,0.6);
    border:1px solid rgba(255,255,255,0.03);
  }

  header{
    display:flex;
    gap:12px;
    align-items:center;
    margin-bottom:12px;
  }
  .logo {
    width:52px;
    height:52px;
    border-radius:12px;
    display:grid;
    place-items:center;
    background:linear-gradient(135deg,var(--accent),var(--accent-2));
    font-weight:700;
    color:white;
    font-size:18px;
    flex:0 0 52px;
    box-shadow:0 6px 18px rgba(12,18,36,0.6);
  }
  h1{
    margin:0;
    font-size:20px;
    letter-spacing:0.2px;
  }
  p.lead{
    margin:0;
    color:var(--muted);
    font-size:13px;
  }

  .input-row{
    display:flex;
    gap:8px;
    margin-top:12px;
  }
  .input-row input[type="text"]{
    flex:1;
    padding:12px 14px;
    border-radius:10px;
    border:1px solid rgba(255,255,255,0.03);
    background:var(--glass);
    color:inherit;
    outline:none;
    font-size:15px;
    box-shadow:inset 0 1px 0 rgba(255,255,255,0.02);
  }
  .input-row button{
    border:0;
    padding:0 14px;
    border-radius:10px;
    background:linear-gradient(90deg,var(--accent),var(--accent-2));
    color:white;
    font-weight:600;
    cursor:pointer;
    min-width:90px;
  }

  .controls{
    display:flex;
    align-items:center;
    justify-content:space-between;
    gap:12px;
    margin-top:12px;
    flex-wrap:wrap;
  }
  .filters{
    display:flex;
    gap:8px;
  }
  .filters button{
    background:transparent;
    color:var(--muted);
    border:1px solid rgba(255,255,255,0.03);
    padding:8px 12px;
    border-radius:10px;
    cursor:pointer;
    font-size:13px;
  }
  .filters button.active{
    color:white;
    background:linear-gradient(90deg, rgba(255,255,255,0.04), rgba(255,255,255,0.02));
    border-color: rgba(255,255,255,0.06);
  }
  .meta {
    color:var(--muted);
    font-size:13px;
  }

  ul.todo-list{
    list-style:none;
    padding:0;
    margin:18px 0 8px 0;
    display:flex;
    flex-direction:column;
    gap:8px;
    min-height:60px;
  }
  li.todo{
    background:linear-gradient(180deg, rgba(255,255,255,0.015), rgba(255,255,255,0.01));
    border-radius:10px;
    padding:8px;
    display:flex;
    align-items:center;
    gap:10px;
    border:1px solid rgba(255,255,255,0.02);
    touch-action: manipulation;
  }
  .todo .left{
    display:flex;
    gap:10px;
    align-items:center;
    flex:1;
    min-width:0;
  }
  .todo input[type="checkbox"]{
    width:18px;
    height:18px;
    cursor:pointer;
  }
  .todo .title{
    font-size:15px;
    overflow:hidden;
    text-overflow:ellipsis;
    white-space:nowrap;
  }
  .todo.completed .title{
    text-decoration:line-through;
    color:var(--muted);
    opacity:0.9;
  }
  .todo .actions{
    display:flex;
    gap:8px;
  }
  .icon-btn{
    border:0;
    background:transparent;
    color:var(--muted);
    cursor:pointer;
    padding:6px;
    border-radius:8px;
    font-size:13px;
  }
  .icon-btn:hover{color:white}

  /* drag handle */
  .drag-handle{
    cursor:grab;
    user-select:none;
    display:inline-grid;
    place-items:center;
    width:34px;height:34px;border-radius:8px;
  }
  .dragging{
    opacity:0.6;
    transform:scale(0.98);
    box-shadow:0 10px 30px rgba(0,0,0,0.4);
  }

  /* small edit input inside item */
  .todo input.edit {
    padding:8px 10px;
    border-radius:8px;
    border:1px solid rgba(255,255,255,0.04);
    background:transparent;
    color:inherit;
    font-size:14px;
    width:100%;
  }

  .empty{
    text-align:center;
    color:var(--muted);
    padding:14px 8px;
    border-radius:10px;
    border:1px dashed rgba(255,255,255,0.02);
  }

  footer.row{
    display:flex;
    gap:12px;
    align-items:center;
    justify-content:space-between;
    margin-top:12px;
    flex-wrap:wrap;
  }

  .secondary{
    background:transparent;
    color:var(--muted);
    border:1px solid rgba(255,255,255,0.03);
    padding:8px 12px;
    border-radius:10px;
    cursor:pointer;
  }

  .right-controls{display:flex;gap:8px;align-items:center}

  /* file input hidden */
  #importFile { display:none; }

  @media (max-width:720px){
    .input-row button{min-width:72px;padding:10px}
    header{gap:10px}
    .logo{width:44px;height:44px;font-size:16px}
  }

  /* Printable view */
  @media print {
    body{background:white;color:black}
    .app{box-shadow:none;border-radius:0;padding:6px;background:transparent}
    header, .controls, footer.row { display:none !important; }
    .todo-list { gap:4px; }
    li.todo { border:1px solid #ddd; background:white; color:black; padding:8px; }
    input[type="checkbox"]{ transform:scale(0.9) }
  }
</style>
</head>
<body>
  <main class="app" aria-labelledby="appTitle">
    <header>
      <div class="logo">TD</div>
      <div>
        <h1 id="appTitle">To-Do List</h1>
        <p class="lead">All-in-one: drag, export/import, print, and installable PWA.</p>
      </div>
    </header>

    <section aria-label="Add task">
      <div class="input-row">
        <input id="taskInput" type="text" placeholder="Add a new task and press Enter…" aria-label="New task">
        <button id="addBtn" aria-label="Add task">Add</button>
      </div>

      <div class="controls" style="margin-top:12px">
        <div style="display:flex;gap:12px;align-items:center">
          <div class="filters" role="tablist" aria-label="Filters">
            <button data-filter="all" class="active" role="tab">All</button>
            <button data-filter="active" role="tab">Active</button>
            <button data-filter="completed" role="tab">Completed</button>
          </div>
          <div class="meta" style="margin-left:8px">
            <span id="count">0</span> tasks
          </div>
        </div>

        <div class="right-controls">
          <button id="exportBtn" class="secondary" title="Export tasks to JSON">Export</button>
          <label for="importFile" class="secondary" style="cursor:pointer;">Import</label>
          <input id="importFile" type="file" accept="application/json">
          <button id="printBtn" class="secondary" title="Printable view">Print</button>
          <button id="installBtn" class="secondary" title="Install app" style="display:none">Install</button>
        </div>
      </div>
    </section>

    <section aria-label="Tasks list" style="margin-top:8px">
      <ul class="todo-list" id="todoList" aria-live="polite"></ul>
      <div id="empty" class="empty" style="display:none">No tasks yet — add one above ✨</div>
    </section>

    <footer class="row" style="margin-top:12px">
      <div style="display:flex;gap:8px;align-items:center">
        <button id="clearCompleted" class="secondary">Clear completed</button>
        <button id="resetAll" class="secondary" title="Delete everything">Reset all</button>
      </div>
      <div class="meta">Local storage sync — uses this browser only</div>
    </footer>
  </main>

<script>
(() => {
  /* =========================
     Data & Storage
     ========================= */
  const LS_KEY = 'todo.tasks.v2';
  let tasks = [];
  let filter = 'all';

  // DOM
  const taskInput = document.getElementById('taskInput');
  const addBtn = document.getElementById('addBtn');
  const todoList = document.getElementById('todoList');
  const emptyEl = document.getElementById('empty');
  const countEl = document.getElementById('count');
  const filters = document.querySelectorAll('.filters button');
  const clearCompletedBtn = document.getElementById('clearCompleted');
  const resetAllBtn = document.getElementById('resetAll');
  const exportBtn = document.getElementById('exportBtn');
  const importFile = document.getElementById('importFile');
  const printBtn = document.getElementById('printBtn');
  const installBtn = document.getElementById('installBtn');

  // helpers
  const idNow = () => 't_'+Date.now()+'_'+Math.floor(Math.random()*9999);
  const save = () => localStorage.setItem(LS_KEY, JSON.stringify(tasks));
  const load = () => {
    try{
      const raw = localStorage.getItem(LS_KEY);
      tasks = raw ? JSON.parse(raw) : [];
      tasks = tasks.map(t => ({ id: t.id || idNow(), title: t.title || '', completed: !!t.completed }));
    }catch(e){
      console.error('Failed to load tasks', e);
      tasks = [];
    }
  };

  /* =========================
     CRUD operations
     ========================= */
  const addTask = (title, options={prepend:true}) => {
    const trimmed = (title||'').trim();
    if(!trimmed) return;
    const newTask = { id: idNow(), title: trimmed, completed: false };
    if(options.prepend) tasks.unshift(newTask);
    else tasks.push(newTask);
    save(); render();
    return newTask;
  };

  const updateTask = (id, patch) => {
    const i = tasks.findIndex(t=>t.id===id);
    if(i===-1) return;
    tasks[i] = { ...tasks[i], ...patch };
    save(); render();
  };

  const deleteTask = (id) => {
    tasks = tasks.filter(t=>t.id!==id);
    save(); render();
  };

  const clearCompleted = () => {
    tasks = tasks.filter(t=>!t.completed);
    save(); render();
  };

  const resetAll = () => {
    if(!confirm('Delete ALL tasks? This cannot be undone.')) return;
    tasks = [];
    save(); render();
  };

  /* =========================
     Filtering & Rendering
     ========================= */
  const filtered = () => {
    if(filter==='active') return tasks.filter(t=>!t.completed);
    if(filter==='completed') return tasks.filter(t=>t.completed);
    return tasks;
  };

  function render(){
    const list = filtered();
    todoList.innerHTML = '';

    if(list.length === 0){
      emptyEl.style.display = 'block';
    } else {
      emptyEl.style.display = 'none';
    }

    list.forEach((t, idx) => {
      const li = document.createElement('li');
      li.className = 'todo';
      if(t.completed) li.classList.add('completed');
      li.dataset.id = t.id;
      li.draggable = true;
      li.setAttribute('aria-grabbed', 'false');

      // left container
      const left = document.createElement('div');
      left.className = 'left';

      // drag handle
      const handle = document.createElement('div');
      handle.className = 'drag-handle';
      handle.title = 'Drag to reorder';
      handle.innerHTML = '&#x2261;'; // ≡

      // checkbox
      const checkbox = document.createElement('input');
      checkbox.type = 'checkbox';
      checkbox.checked = !!t.completed;
      checkbox.title = t.completed ? 'Mark as active' : 'Mark as complete';
      checkbox.addEventListener('change', () => updateTask(t.id, { completed: checkbox.checked }));

      const titleWrap = document.createElement('div');
      titleWrap.style.flex = '1';
      titleWrap.style.minWidth = '0';

      const title = document.createElement('div');
      title.className = 'title';
      title.textContent = t.title;
      title.title = t.title;
      title.tabIndex = 0;
      // double click to edit
      title.addEventListener('dblclick', () => startEdit(t.id, title, li));
      title.addEventListener('keydown', (e) => { if(e.key === 'Enter') startEdit(t.id, title, li); });

      titleWrap.appendChild(title);
      left.appendChild(handle);
      left.appendChild(checkbox);
      left.appendChild(titleWrap);

      // actions
      const actions = document.createElement('div');
      actions.className = 'actions';

      const editBtn = document.createElement('button');
      editBtn.className = 'icon-btn';
      editBtn.innerHTML = 'Edit';
      editBtn.title = 'Edit';
      editBtn.addEventListener('click', () => startEdit(t.id, title, li));

      const delBtn = document.createElement('button');
      delBtn.className = 'icon-btn';
      delBtn.innerHTML = 'Delete';
      delBtn.title = 'Delete';
      delBtn.addEventListener('click', () => {
        if(confirm('Delete this task?')) deleteTask(t.id);
      });

      actions.appendChild(editBtn);
      actions.appendChild(delBtn);

      li.appendChild(left);
      li.appendChild(actions);

      // drag events
      li.addEventListener('dragstart', (ev) => {
        li.classList.add('dragging');
        li.setAttribute('aria-grabbed', 'true');
        ev.dataTransfer.setData('text/plain', t.id);
        // allowed effect
        try { ev.dataTransfer.effectAllowed = 'move'; } catch(e){}
      });
      li.addEventListener('dragend', () => {
        li.classList.remove('dragging');
        li.setAttribute('aria-grabbed', 'false');
      });

      li.addEventListener('dragover', (ev) => {
        ev.preventDefault();
        ev.dataTransfer.dropEffect = 'move';
        const dragging = document.querySelector('.dragging');
        if(!dragging) return;
        // visualize position (insert before)
        const bounding = li.getBoundingClientRect();
        const offset = ev.clientY - bounding.top;
        if(offset > bounding.height / 2) {
          li.style.borderBottom = '2px dashed rgba(255,255,255,0.06)';
          li.style.borderTop = '';
        } else {
          li.style.borderTop = '2px dashed rgba(255,255,255,0.06)';
          li.style.borderBottom = '';
        }
      });

      li.addEventListener('dragleave', () => {
        li.style.borderBottom = '';
        li.style.borderTop = '';
      });

      li.addEventListener('drop', (ev) => {
        ev.preventDefault();
        li.style.borderBottom = '';
        li.style.borderTop = '';
        const draggedId = ev.dataTransfer.getData('text/plain');
        if(!draggedId) return;
        reorder(draggedId, t.id, ev.clientY < li.getBoundingClientRect().top + li.getBoundingClientRect().height/2);
      });

      todoList.appendChild(li);
    });

    // update count
    countEl.textContent = tasks.length;
    // update filter buttons active
    filters.forEach(b => b.classList.toggle('active', b.dataset.filter===filter));
  }

  function reorder(draggedId, targetId, insertBefore=true){
    const fromIndex = tasks.findIndex(t=>t.id===draggedId);
    const toIndex = tasks.findIndex(t=>t.id===targetId);
    if(fromIndex === -1 || toIndex === -1) return;
    const [moved] = tasks.splice(fromIndex,1);
    let newIndex = toIndex;
    if(!insertBefore) newIndex = toIndex + (fromIndex < toIndex ? 0 : 1);
    tasks.splice(newIndex, 0, moved);
    save(); render();
  }

  /* =========================
     Editing flow
     ========================= */
  function startEdit(id, titleEl, liEl){
    const oldText = titleEl.textContent;
    const input = document.createElement('input');
    input.type = 'text';
    input.className = 'edit';
    input.value = oldText;
    // replace
    const wrap = titleEl.parentElement;
    wrap.replaceChild(input, titleEl);
    input.focus();
    input.setSelectionRange(input.value.length, input.value.length);

    function commit(){
      const val = input.value.trim();
      if(val) updateTask(id, { title: val });
      else {
        if(confirm('Empty title — delete task?')) deleteTask(id);
        else updateTask(id, { title: oldText });
      }
    }
    function cancel(){
      wrap.replaceChild(titleEl, input);
    }

    input.addEventListener('keydown', (ev) => {
      if(ev.key === 'Enter') { commit(); }
      else if(ev.key === 'Escape') { cancel(); }
    });
    input.addEventListener('blur', () => {
      setTimeout(() => {
        if(document.activeElement !== input) commit();
      }, 120);
    });
  }

  /* =========================
     Export / Import (JSON)
     ========================= */
  function exportTasks(){
    const data = { exportedAt: new Date().toISOString(), tasks };
    const blob = new Blob([JSON.stringify(data, null, 2)], { type: 'application/json' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = `todo-export-${(new Date()).toISOString().slice(0,19).replace(/[:T]/g,'-')}.json`;
    document.body.appendChild(a);
    a.click();
    a.remove();
    URL.revokeObjectURL(url);
  }

  importFile.addEventListener('change', (e) => {
    const file = e.target.files && e.target.files[0];
    if(!file) return;
    const reader = new FileReader();
    reader.onload = () => {
      try{
        const parsed = JSON.parse(reader.result);
        // support both {tasks: [...] } and direct array
        const imported = Array.isArray(parsed) ? parsed : (parsed.tasks || parsed);
        if(!Array.isArray(imported)) throw new Error('Invalid format: expected array of tasks');
        // normalize
        const normalized = imported.map(t => ({ id: t.id || idNow(), title: t.title || '', completed: !!t.completed }));
        // ask user: merge or replace
        if(tasks.length === 0){
          tasks = normalized;
          save(); render();
          alert('Imported tasks (no existing tasks to merge).');
        } else {
          const choice = confirm('Press OK to MERGE imported tasks with existing ones. Press Cancel to REPLACE existing tasks.');
          if(choice){
            // merge (keep existing first)
            // avoid id collisions
            const existingIds = new Set(tasks.map(t=>t.id));
            normalized.forEach(n => {
              if(existingIds.has(n.id)) n.id = idNow();
              tasks.push(n);
            });
            save(); render();
            alert('Imported and merged tasks.');
          } else {
            tasks = normalized;
            save(); render();
            alert('Existing tasks replaced with imported tasks.');
          }
        }
      }catch(err){
        alert('Failed to import file: ' + err.message);
      } finally {
        importFile.value = '';
      }
    };
    reader.onerror = () => {
      alert('Failed to read file.');
      importFile.value = '';
    };
    reader.readAsText(file, 'utf-8');
  });

  /* =========================
     Events & Shortcuts
     ========================= */
  addBtn.addEventListener('click', () => {
    addTask(taskInput.value);
    taskInput.value = '';
    taskInput.focus();
  });

  taskInput.addEventListener('keydown', (e) => {
    if(e.key === 'Enter'){
      addTask(taskInput.value);
      taskInput.value = '';
    }
  });

  filters.forEach(b => {
    b.addEventListener('click', () => {
      filter = b.dataset.filter;
      render();
    });
  });

  clearCompletedBtn.addEventListener('click', () => {
    if(!tasks.some(t=>t.completed)) return alert('No completed tasks to clear.');
    if(confirm('Clear all completed tasks?')) clearCompleted();
  });

  resetAllBtn.addEventListener('click', resetAll);

  exportBtn.addEventListener('click', exportTasks);
  printBtn.addEventListener('click', () => window.print());

  // quick focus 'n'
  window.addEventListener('keydown', (e) => {
    if(e.target.tagName === 'INPUT' || e.target.tagName === 'TEXTAREA') return;
    if(e.key === 'n' || e.key === 'N') {
      taskInput.focus(); taskInput.select();
    }
  });

  // allow tapping import label to trigger file dialog
  document.querySelector('label[for="importFile"]').addEventListener('keydown', (e) => {
    if(e.key === 'Enter' || e.key === ' ') importFile.click();
  });

  /* =========================
     PWA: Manifest + Service Worker (created from blobs in-page)
     ========================= */
  function createManifestAndSW(){
    try{
      // Manifest
      const manifest = {
        name: "To-Do List (Single File)",
        short_name: "ToDo",
        start_url: ".",
        display: "standalone",
        background_color: "#071233",
        theme_color: "#071233",
        icons: [
          { src: "data:image/svg+xml;base64," + btoa(`<svg xmlns='http://www.w3.org/2000/svg' width='192' height='192'><rect width='100%' height='100%' fill='#7c3aed'/><text x='50%' y='55%' font-size='80' text-anchor='middle' fill='white' font-family='Arial'>TD</text></svg>`), sizes: "192x192", type: "image/svg+xml" }
        ]
      };
      const manifestBlob = new Blob([JSON.stringify(manifest)], {type: 'application/json'});
      const manifestURL = URL.createObjectURL(manifestBlob);
      // link manifest
      let l = document.querySelector('link[rel="manifest"]');
      if(!l){
        l = document.createElement('link');
        l.rel = 'manifest';
        document.head.appendChild(l);
      }
      l.href = manifestURL;

      // Service worker script
      const swScript = `
        const CACHE_NAME = 'todo-cache-v1';
        const FILES = ['.'];
        self.addEventListener('install', (ev) => {
          self.skipWaiting();
          ev.waitUntil(
            caches.open(CACHE_NAME).then(cache => cache.addAll(FILES).catch(()=>{}))
          );
        });
        self.addEventListener('activate', (ev) => {
          ev.waitUntil(
            caches.keys().then(keys => Promise.all(keys.map(k => {
              if(k !== CACHE_NAME) return caches.delete(k);
            }))).catch(()=>{})
          );
        });
        self.addEventListener('fetch', (ev) => {
          // network-first for navigation, cache fallback for others
          if(ev.request.mode === 'navigate' || ev.request.method === 'GET'){
            ev.respondWith(
              fetch(ev.request).then(resp => {
                // optionally cache GET responses
                return resp;
              }).catch(() => caches.match(ev.request).then(m => m || caches.match('.')))
            );
          } else {
            ev.respondWith(fetch(ev.request).catch(() => caches.match(ev.request)));
          }
        });
      `;
      const swBlob = new Blob([swScript], {type: 'text/javascript'});
      const swURL = URL.createObjectURL(swBlob);
      // register
      if('serviceWorker' in navigator){
        navigator.serviceWorker.register(swURL).then(reg => {
          console.log('Service worker registered:', reg.scope);
        }).catch(err => console.warn('SW registration failed:', err));
      }
    }catch(err){
      console.warn('PWA setup failed', err);
    }
  }

  // show install button if available (beforeinstallprompt)
  let deferredPrompt = null;
  window.addEventListener('beforeinstallprompt', (e) => {
    e.preventDefault();
    deferredPrompt = e;
    installBtn.style.display = 'inline-block';
  });

  installBtn.addEventListener('click', async () => {
    if(deferredPrompt){
      deferredPrompt.prompt();
      const choice = await deferredPrompt.userChoice;
      deferredPrompt = null;
      installBtn.style.display = 'none';
    } else {
      alert('Install not available on this browser or already installed.');
    }
  });

  // create manifest and SW now
  createManifestAndSW();

  /* =========================
     Initial load & expose debug
     ========================= */
  load();
  render();

  // expose for debugging
  window.__todo = {
    get tasks(){ return tasks.slice(); },
    add: addTask,
    save,
    load,
    resetAll,
    clearCompleted
  };

})();
</script>
</body>
</html>
