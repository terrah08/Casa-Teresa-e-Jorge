<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <title>Portaria - Segunda Sem Leite (Relatório Profissional)</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/chartjs-plugin-datalabels@2.2.0/dist/chartjs-plugin-datalabels.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/jspdf@2.5.1/dist/jspdf.umd.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/html2canvas@1.4.1/dist/html2canvas.min.js"></script>
    <style>
        .btn { @apply px-3 py-2 rounded-xl text-white shadow-md transition-all duration-200; }
        .small { font-size: .85rem; }
        .table-fixed td, .table-fixed th { vertical-align: middle; }
        .hidden-value { filter: blur(6px); pointer-events: none; user-select: none; }
    </style>
</head>
<body class="bg-gray-100 p-4">

<header class="flex flex-col md:flex-row md:items-center md:justify-between gap-4 mb-6">
  <div class="flex items-center gap-4">
    <div>
      <h1 class="text-2xl md:text-3xl font-extrabold text-gray-800">Portaria - Segunda Sem Leite 🥛🚫</h1>
      <div class="text-sm text-gray-500">Controle de entradas e arrecadação</div>
    </div>
  </div>

  <div class="flex gap-3 items-center">
    <label class="small text-gray-600 font-bold">Dia:</label>
    <input id="currentDate" type="date" class="border rounded px-2 py-1" />
    <button id="saveDay" class="btn bg-green-600 hover:bg-green-700 small">Salvar</button>
    <button id="resetDay" class="btn bg-red-600 hover:bg-red-700 small">Resetar</button>
  </div>
</header>

<main class="grid grid-cols-1 lg:grid-cols-3 gap-6">

  <section class="bg-white p-4 rounded-xl shadow col-span-1">
    <h2 class="font-semibold mb-4 border-b pb-2 text-gray-700">Registrar entrada</h2>
    <div id="buttonsContainer" class="grid grid-cols-2 sm:grid-cols-3 gap-2 mb-6"></div>
    <div class="flex flex-col gap-2">
      <button id="generateReport" class="btn bg-blue-600 hover:bg-blue-700 font-bold">📄 Gerar Relatório (PDF)</button>
      <button id="exportCSV" class="btn bg-indigo-600 hover:bg-indigo-700 small">Exportar CSV</button>
    </div>
  </section>

  <section class="bg-white p-4 rounded-xl shadow col-span-2">
    <div class="flex flex-col md:flex-row md:items-center md:justify-between gap-4">
      <div class="flex gap-4">
        <div class="bg-gray-100 p-3 rounded-lg text-center min-w-[100px]">
          <div class="text-xs text-gray-500 uppercase font-bold mb-1">Pessoas</div>
          <div id="totalPeople" class="text-2xl font-bold">0</div>
        </div>
        <div class="bg-gray-100 p-3 rounded-lg text-center min-w-[140px] relative">
          <div class="flex items-center justify-center gap-2 mb-1">
             <span class="text-xs text-gray-500 uppercase font-bold">Valor Total</span>
             <button id="toggleValue" class="text-sm opacity-60 hover:opacity-100 transition-opacity">
                <span id="eyeIcon">👁️</span>
             </button>
          </div>
          <div id="totalCollected" class="text-2xl font-bold">R$ 0,00</div>
        </div>
      </div>
      <div class="flex gap-2">
        <button id="openReportPanel" class="btn bg-emerald-600 hover:bg-emerald-700 small">Ver Painel</button>
        <button id="toggleTable" class="btn bg-slate-600 hover:bg-slate-700 small">Tabela</button>
      </div>
    </div>

    <div id="tableWrapper" class="mt-4 overflow-x-auto border-t">
      <table class="min-w-full table-fixed text-sm bg-white">
        <thead class="bg-gray-50">
          <tr>
            <th class="p-2 text-left">Hora</th>
            <th class="p-2 text-left">Tipo</th>
            <th class="p-2 text-right">Valor</th>
            <th class="p-2 text-center">Pessoas</th>
            <th class="p-2 text-center">Ações</th>
          </tr>
        </thead>
        <tbody id="entriesBody" class="divide-y"></tbody>
      </table>
    </div>
  </section>

  <section id="reportPanel" class="bg-white p-6 rounded-xl shadow-2xl col-span-3 hidden border-2 border-emerald-100">
    <div class="flex items-center justify-between mb-6 border-b pb-4">
      <h2 class="text-xl font-bold text-gray-800">Resumo Profissional do Dia</h2>
      <div class="flex gap-2">
        <button id="downloadPdf" class="btn bg-cyan-600 hover:bg-cyan-700 small">💾 Baixar PDF</button>
        <button id="closeReport" class="btn bg-gray-500 hover:bg-gray-600 small">Fechar</button>
      </div>
    </div>
    <div class="grid grid-cols-1 md:grid-cols-2 gap-6 mb-6">
      <div class="bg-gray-50 p-4 rounded-lg"><div id="reportSummary" class="space-y-1"></div></div>
      <div class="bg-gray-50 p-4 rounded-lg"><div id="reportTotals" class="grid grid-cols-2 gap-2"></div></div>
    </div>
    <div class="flex justify-center"><div class="w-full max-w-md"><canvas id="reportChart"></canvas></div></div>
  </section>

</main>

<script>
/* TIPOS DE ENTRADA (ATUALIZADO) */
const PRICE_TYPES = [
    { id: "20_dinheiro", label: "Dinheiro - R$20", price: 20, people: 1, kind: "Dinheiro" },
    { id: "30_dinheiro", label: "Dinheiro - R$30", price: 30, people: 1, kind: "Dinheiro" },
    { id: "50_dinheiro", label: "Dinheiro - R$50", price: 50, people: 2, kind: "Dinheiro" },
    { id: "20_credito", label: "Crédito - R$20", price: 20, people: 1, kind: "Crédito" },
    { id: "30_credito", label: "Crédito - R$30", price: 30, people: 1, kind: "Crédito" },
    { id: "50_credito", label: "Crédito - R$50", price: 50, people: 2, kind: "Crédito" },
    { id: "20_debito", label: "Débito - R$20", price: 20, people: 1, kind: "Débito" },
    { id: "30_debito", label: "Débito - R$30", price: 30, people: 1, kind: "Débito" },
    { id: "50_debito", label: "Débito - R$50", price: 50, people: 2, kind: "Débito" },
    { id: "20_pix", label: "Pix - R$20", price: 20, people: 1, kind: "Pix" },
    { id: "30_pix", label: "Pix - R$30", price: 30, people: 1, kind: "Pix" },
    { id: "50_pix", label: "Pix - R$50", price: 50, people: 2, kind: "Pix" },
    { id: "free100", label: "Free 100 Pessoas", price: 0, people: 100, kind: "Gratuidade" },
    { id: "free", label: "Lista (Free)", price: 0, people: 1, kind: "Gratuidade" },
    { id: "militar", label: "Militar", price: 0, people: 1, kind: "Gratuidade" },
    { id: "aniversario", label: "Aniversário", price: 0, people: 1, kind: "Gratuidade" }
];

const buttonsContainer = document.getElementById('buttonsContainer');
PRICE_TYPES.forEach(p => {
    const b = document.createElement('button');
    b.className = 'px-2 py-2 rounded-xl text-white text-[10px] md:text-xs font-bold shadow-md transition-all active:scale-95';
    
    if (p.kind === "Dinheiro") b.classList.add("bg-green-600", "hover:bg-green-700");
    else if (p.kind === "Crédito") b.classList.add("bg-amber-500", "hover:bg-amber-600");
    else if (p.kind === "Débito") b.classList.add("bg-blue-600", "hover:bg-blue-700");
    else if (p.kind === "Pix") b.classList.add("bg-teal-600", "hover:bg-teal-700");
    else if (p.id === "free100") b.classList.add("bg-purple-600", "hover:bg-purple-700"); // Destaque para o 100
    else b.classList.add("bg-gray-400", "hover:bg-gray-500");

    b.textContent = p.label;
    b.onclick = () => addEntry(p.id);
    buttonsContainer.appendChild(b);
});

/* LÓGICA CORE */
const todayKey = () => new Date().toISOString().slice(0,10);
const storageKey = d => `portaria_${d}`;
let currentDate = todayKey();
let entries = [];
let isValueVisible = true;

const currentDateEl = document.getElementById('currentDate');
const entriesBody = document.getElementById('entriesBody');
const totalPeopleEl = document.getElementById('totalPeople');
const totalCollectedEl = document.getElementById('totalCollected');
const toggleValueBtn = document.getElementById('toggleValue');
const eyeIcon = document.getElementById('eyeIcon');
const reportPanel = document.getElementById('reportPanel');
const reportSummary = document.getElementById('reportSummary');
const reportTotals = document.getElementById('reportTotals');
const reportChartEl = document.getElementById('reportChart').getContext('2d');
let reportChart = null;

currentDateEl.value = currentDate;

toggleValueBtn.onclick = () => {
    isValueVisible = !isValueVisible;
    eyeIcon.textContent = isValueVisible ? '👁️' : '🙈';
    totalCollectedEl.classList.toggle('hidden-value', !isValueVisible);
    document.querySelectorAll('.table-price').forEach(el => el.classList.toggle('hidden-value', !isValueVisible));
};

function loadEntries() {
    const raw = localStorage.getItem(storageKey(currentDate));
    entries = raw ? JSON.parse(raw) : [];
    renderEntries();
}

function saveEntries() {
    localStorage.setItem(storageKey(currentDate), JSON.stringify(entries));
}

function renderEntries() {
    const blurClass = isValueVisible ? '' : 'hidden-value';
    entriesBody.innerHTML = entries.length ? entries.map(e => `
        <tr>
            <td class="p-2">${new Date(e.timestamp).toLocaleTimeString([], {hour:'2-digit', minute:'2-digit'})}</td>
            <td class="p-2 font-medium">${e.type}</td>
            <td class="p-2 text-right font-mono table-price ${blurClass}">R$ ${e.price.toFixed(2)}</td>
            <td class="p-2 text-center">${e.people}</td>
            <td class="p-2 text-center space-x-1">
                <button onclick="editEntry(${e.id})" class="text-blue-600 text-xs">Editar</button>
                <button onclick="deleteEntry(${e.id})" class="text-red-600 text-xs">Excluir</button>
            </td>
        </tr>
    `).join('') : `<tr><td colspan="5" class="p-4 text-center text-gray-400">Sem registros</td></tr>`;

    const totals = entries.reduce((acc, e) => {
        acc.people += e.people;
        acc.collected += e.price;
        return acc;
    }, { people:0, collected:0 });

    totalPeopleEl.textContent = totals.people;
    totalCollectedEl.textContent = `R$ ${totals.collected.toFixed(2)}`;
}

function addEntry(id) {
    const t = PRICE_TYPES.find(p => p.id === id);
    if (!t) return;
    entries.unshift({ id: Date.now(), timestamp: new Date().toISOString(), type: t.label, price: t.price, people: t.people, kind: t.kind });
    saveEntries();
    renderEntries();
}

function deleteEntry(id) {
    if (!confirm('Excluir registro?')) return;
    entries = entries.filter(x => x.id !== id);
    saveEntries();
    renderEntries();
}

function editEntry(id) {
    const item = entries.find(x => x.id === id);
    if (!item) return;
    const newPrice = prompt('Preço:', item.price);
    const newPeople = prompt('Pessoas:', item.people);
    if (newPrice !== null && newPeople !== null) {
        item.price = Number(newPrice);
        item.people = Number(newPeople);
        saveEntries();
        renderEntries();
    }
}

/* CSV E RELATÓRIO */
document.getElementById('exportCSV').addEventListener('click', () => {
    const headers = ['Data','Tipo','Valor','Pessoas'];
    const rows = entries.map(e => [`"${new Date(e.timestamp).toLocaleString()}"`, `"${e.type}"`, e.price, e.people]);
    const csv = [headers, ...rows].map(r => r.join(',')).join('\n');
    const blob = new Blob(["\ufeff" + csv], { type: 'text/csv;charset=utf-8;' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url; a.download = `portaria_${currentDate}.csv`; a.click();
});

document.getElementById('generateReport').addEventListener('click', () => {
    reportPanel.classList.remove('hidden');
    generateReportVisual();
    reportPanel.scrollIntoView({ behavior: 'smooth' });
});

document.getElementById('closeReport').addEventListener('click', () => reportPanel.classList.add('hidden'));

function generateReportVisual() {
    const totals = entries.reduce((a,e) => { a.people += e.people; a.collected += e.price; return a; }, { people:0, collected:0 });
    const byKind = {};
    entries.forEach(e => {
        byKind[e.kind] = byKind[e.kind] || { count:0, collected:0 };
        byKind[e.kind].count += e.people;
        byKind[e.kind].collected += e.price;
    });

    reportSummary.innerHTML = `
        <p><strong>📅 Data:</strong> ${currentDate}</p>
        <p><strong>💰 Valor Total:</strong> R$ ${totals.collected.toFixed(2)}</p>
        <p><strong>👥 Total Pessoas:</strong> ${totals.people}</p>
    `;

    let totalsHtml = '';
    Object.keys(byKind).forEach(k => {
        totalsHtml += `<div class="bg-white p-2 rounded shadow-sm border-l-4 border-emerald-500 text-xs font-bold">${k}: R$ ${byKind[k].collected.toFixed(2)} (${byKind[k].count}p)</div>`;
    });
    reportTotals.innerHTML = totalsHtml || 'Sem dados';

    if (reportChart) reportChart.destroy();
    reportChart = new Chart(reportChartEl, {
        type: 'doughnut',
        data: { labels: Object.keys(byKind), datasets: [{ data: Object.values(byKind).map(v => v.collected), backgroundColor: ['#10B981','#06B6D4','#F59E0B','#6366F1','#F43F5E'] }] },
        plugins: [ChartDataLabels],
        options: { plugins: { datalabels: { color: '#fff', formatter: (v) => `R$${v.toFixed(0)}` } } }
    });
}

document.getElementById('downloadPdf').addEventListener('click', async () => {
    const panel = document.getElementById('reportPanel');
    document.getElementById('downloadPdf').style.display = 'none';
    document.getElementById('closeReport').style.display = 'none';
    const canvas = await html2canvas(panel, { scale: 2 });
    const imgData = canvas.toDataURL('image/png');
    const { jsPDF } = window.jspdf;
    const pdf = new jsPDF('p', 'mm', 'a4');
    pdf.addImage(imgData, 'PNG', 10, 15, 190, (canvas.height * 190) / canvas.width);
    pdf.save(`relatorio_${currentDate}.pdf`);
    document.getElementById('downloadPdf').style.display = '';
    document.getElementById('closeReport').style.display = '';
});

currentDateEl.onchange = e => { currentDate = e.target.value; loadEntries(); };
loadEntries();
</script>
</body>
</html>
