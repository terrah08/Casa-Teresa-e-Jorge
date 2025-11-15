# Casa-Teresa-e-Jorge

<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no" />
  <title>Portaria- Casa Teresa e Jorge🟩🩷</title>
  <link href="https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/dist/tailwind.min.css" rel="stylesheet">
  <meta name="theme-color" content="#10B981" />
  <link rel="manifest" href="manifest.json">
  <style>
    body { -webkit-tap-highlight-color: transparent; font-family: system-ui, sans-serif; }
    button:active { transform: scale(0.96); }
    table { width: 100%; }
  </style>
  <script>
    if ('serviceWorker' in navigator) {
      window.addEventListener('load', () => {
        navigator.serviceWorker.register('service-worker.js');
      });
    }
  </script>
</head>
<body class="min-h-screen bg-gray-100 p-3 sm:p-6">
  <div class="max-w-5xl mx-auto">
    <header class="flex flex-col sm:flex-row sm:items-center justify-between mb-6 gap-4">
      <div>
        <h1 class="text-2xl sm:text-3xl font-extrabold text-gray-900 text-center sm:text-left">Portaria - Casa Teresa e Jorge 🟩🩷</h1>
        <div class="flex flex-wrap items-center justify-center sm:justify-start gap-2 mt-3">
          <label class="text-sm text-gray-500">Dia:</label>
          <input id="currentDate" type="date" class="border rounded-md px-2 py-1 text-sm" />
          <button id="saveDay" class="px-3 py-2 rounded-lg bg-green-600 text-white hover:bg-green-700 shadow text-sm">Salvar</button>
        </div>
      </div>
      <div class="flex flex-wrap justify-center sm:justify-end gap-3">
        <div class="bg-white shadow rounded-lg px-4 py-3 text-center">
          <div class="text-xs text-gray-500">Pessoas</div>
          <div id="totalPeople" class="text-xl font-bold">0</div>
        </div>
        <div class="bg-white shadow rounded-lg px-4 py-3 text-center">
          <div class="text-xs text-gray-500">Total</div>
          <div id="totalCollected" class="text-xl font-bold">R$ 0,00</div>
        </div>
        <button id="exportCSV" class="px-3 py-2 rounded-lg bg-indigo-600 text-white hover:bg-indigo-700 shadow text-sm">Exportar CSV</button>
        <button id="resetDay" class="px-3 py-2 rounded-lg bg-red-600 text-white hover:bg-red-700 shadow text-sm">Resetar</button>
      </div>
    </header>

    <main class="grid grid-cols-1 lg:grid-cols-3 gap-6">
      <section class="bg-white p-5 rounded-2xl shadow col-span-1">
        <h2 class="text-lg font-semibold mb-3">Registrar entrada</h2>
        <p class="text-sm text-gray-500 mb-4">Escolha o tipo de entrada:</p>
        <div id="buttonsContainer" class="grid grid-cols-2 sm:grid-cols-3 gap-3 mb-4"></div>
        <textarea id="note" placeholder="Nota (ex: nome, grupo, observação)" class="w-full rounded-md border p-2 mb-3 h-20 text-sm"></textarea>
        <div class="text-xs text-gray-400"></div>
      </section>

      <section class="bg-white p-5 rounded-2xl shadow col-span-2 overflow-x-auto">
        <h2 class="text-lg font-semibold mb-3">Entradas do dia</h2>
        <table class="min-w-full text-left text-sm">
          <thead>
            <tr class="text-gray-500">
              <th class="py-2">Hora</th>
              <th class="py-2">Tipo</th>
              <th class="py-2">Preço</th>
              <th class="py-2">Pessoas</th>
              <th class="py-2">Nota</th>
            </tr>
          </thead>
          <tbody id="entriesBody"></tbody>
        </table>
      </section>
    </main>
  </div>

  <script>
    const PRICE_TYPES = [
      { id: "30", label: "Dinheiro - R$30 (Individual) 💸", price: 30, people: 1, color: "bg-yellow-400 text-gray-900" },
      { id: "50", label: "Dinheiro - R$50 (Casal) 💸", price: 50, people: 2, color: "bg-yellow-400 text-gray-900" },
      { id: "30_cartao", label: "Cartão - R$30 (Individual) 💳", price: 30, people: 1, color: "bg-green-500 text-white" },
      { id: "30_cartao", label: "Cartão - R$30 (Casal) 💳", price: 30, people: 1, color: "bg-green-500 text-white" },
      { id: "50_cartao", label: "Cartão - R$50 (Casal) 💳", price: 50, people: 2, color: "bg-green-500 text-white" },
      { id: "20_cartao", label: "Cartão - R$20 (Individual) 💳", price: 20, people: 1, color: "bg-blue-500 text-white" },
      { id: "30_pix", label: "Pix-R$30 (Individual) ❖", price: 30, people: 1, color: "bg-gray-500 text-white" },
      { id: "30_pix", label: "Pix-R$30 (Casal) ❖", price: 30, people: 1, color: "bg-gray-500 text-white" },
      { id: "50_pix", label: "Pix-R$50 (Casal) ❖", price: 50, people: 2, color: "bg-gray-500 text-white" },
      { id: "20_pix", label: "Pix-R$20 (Individual) ❖", price: 20, people: 1, color: "bg-gray-500 text-white" },	
      { id: "free", label: "Lista (Free)", price: 0, people: 1, color: "bg-gray-200 text-gray-800" },
      { id: "militar", label: "Militar", price: 0, people: 1, color: "bg-gray-200 text-gray-800" },
      { id: "aniversario", label: "Aniversário", price: 0, people: 1, color: "bg-gray-200 text-gray-800" }
    ];

    const todayKey = () => new Date().toISOString().slice(0, 10);
    const storageKey = (d) => `portaria_${d}`;

    const currentDateEl = document.getElementById('currentDate');
    const noteEl = document.getElementById('note');
    const entriesBody = document.getElementById('entriesBody');
    const totalPeopleEl = document.getElementById('totalPeople');
    const totalCollectedEl = document.getElementById('totalCollected');
    const buttonsContainer = document.getElementById('buttonsContainer');

    let currentDate = todayKey();
    let entries = [];
    currentDateEl.value = currentDate;

    function loadEntries() {
      const raw = localStorage.getItem(storageKey(currentDate));
      entries = raw ? JSON.parse(raw) : [];
      renderEntries();
    }

    function saveEntries() {
      localStorage.setItem(storageKey(currentDate), JSON.stringify(entries));
    }

    function renderEntries() {
      entriesBody.innerHTML = entries.map(e => `
        <tr class="border-t">
          <td class="py-2">${new Date(e.timestamp).toLocaleTimeString()}</td>
          <td class="py-2">${e.type}</td>
          <td class="py-2">R$ ${e.price.toFixed(2)}</td>
          <td class="py-2">${e.people}</td>
          <td class="py-2">${e.note || ''}</td>
        </tr>`).join('') || '<tr><td colspan="5" class="text-gray-400 py-4 text-center">Nenhum registro</td></tr>';

      const totals = entries.reduce((a, e) => {
        a.people += e.people;
        a.collected += e.price;
        return a;
      }, { people: 0, collected: 0 });

      totalPeopleEl.textContent = totals.people;
      totalCollectedEl.textContent = `R$ ${totals.collected.toFixed(2)}`;
    }

    function addEntry(id) {
      const t = PRICE_TYPES.find(p => p.id === id);
      if (!t) return;
      entries.unshift({
        id: Date.now(),
        timestamp: new Date().toISOString(),
        type: t.label,
        price: t.price,
        people: t.people,
        note: noteEl.value.trim(),
      });
      noteEl.value = '';
      saveEntries();
      renderEntries();
    }

    PRICE_TYPES.forEach(p => {
      const btn = document.createElement('button');
      btn.className = `flex flex-col items-center justify-center gap-1 p-3 rounded-xl border hover:scale-105 transition-transform ${p.color} text-center text-xs sm:text-sm font-semibold shadow-sm`;
      btn.innerHTML = `<span>${p.label}</span>`;
      btn.onclick = () => addEntry(p.id);
      buttonsContainer.appendChild(btn);
    });

    document.getElementById('resetDay').onclick = () => {
      if (confirm('Tem certeza que deseja limpar todos os registros do dia?')) {
        entries = [];
        saveEntries();
        renderEntries();
      }
    };

    document.getElementById('exportCSV').onclick = () => {
      const headers = ['timestamp','type','price','people','note'];
      const rows = entries.map(e => [e.timestamp, e.type, e.price, e.people, e.note || '']);
      const csv = [headers, ...rows].map(r => r.map(v => `"${String(v).replace(/"/g,'""')}"`).join(',')).join('\n');
      const blob = new Blob([csv], { type: 'text/csv;charset=utf-8;' });
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url;
      a.download = `portaria_${currentDate}.csv`;
      a.click();
      URL.revokeObjectURL(url);
    };

    document.getElementById('saveDay').onclick = () => saveEntries();

    currentDateEl.onchange = e => {
      currentDate = e.target.value;
      loadEntries();
    };

    loadEntries();
  </script>
</body>
</html>
