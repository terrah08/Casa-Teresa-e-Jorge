<html lang="pt-BR">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Portaria - Segunda Sem Leite (Relatório Profissional)</title>

<script src="https://cdn.tailwindcss.com"></script>

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

<script src="https://cdn.jsdelivr.net/npm/chartjs-plugin-datalabels@2.2.0/dist/chartjs-plugin-datalabels.min.js"></script>

<script src="https://cdn.jsdelivr.net/npm/jspdf@2.5.1/dist/jspdf.umd.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/html2canvas@1.4.1/dist/html2canvas.min.js"></script>

<style>
  .btn {
  @apply px-3 py-2 rounded-xl text-white shadow-md transition-all duration-200;
}
  .small { font-size: .85rem; }
  .table-fixed td, .table-fixed th { vertical-align: middle; }
  /* Estilo para esconder o valor sem quebrar o layout */
  .hidden-value { filter: blur(5px); pointer-events: none; user-select: none; }
</style>
</head>
<body class="bg-gray-50 min-h-screen p-4 md:p-8">

  <div class="max-w-6xl mx-auto">

    <header class="flex flex-col md:flex-row md:items-center md:justify-between gap-4 mb-6">
      <div class="flex items-center gap-4">
        <div>
          <h1 class="text-2xl md:text-3xl font-extrabold">Portaria - Segunda Sem Leite 🥛🚫</h1>
          <div class="text-sm text-gray-500">Controle de entradas, arrecadação e relatórios profissionais</div>
        </div>
      </div>

      <div class="flex gap-3 items-center">
        <label class="small text-gray-600">Dia:</label>
        <input id="currentDate" type="date" class="border rounded px-2 py-1" />
        <button id="saveDay" class="btn bg-green-600 hover:bg-green-700 small">Salvar</button>
        <button id="resetDay" class="btn bg-red-600 hover:bg-red-700 small">Resetar</button>
      </div>
    </header>

    <main class="grid grid-cols-1 lg:grid-cols-3 gap-6">

      <section class="bg-white p-4 rounded-xl shadow col-span-1">
        <h2 class="font-semibold mb-2">Registrar entrada</h2>

        <div id="buttonsContainer" class="grid grid-cols-2 sm:grid-cols-3 gap-2 mb-3"></div>

        <div class="flex gap-2">
          <button id="exportCSV" class="btn bg-indigo-600 hover:bg-indigo-700 small">Exportar CSV</button>
          <button id="generateReport" class="btn bg-blue-600 hover:bg-blue-700 small">Gerar Relatório (PDF)</button>
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
                 <button id="toggleValue" class="text-sm opacity-60 hover:opacity-100 transition-opacity" title="Mostrar/Ocultar valor">
                    <span id="eyeIcon">👁️</span>
                 </button>
              </div>
              <div id="totalCollected" class="text-2xl font-bold">R$ 0,00</div>
            </div>
          </div>

          <div class="flex gap-2">
            <button id="openReportPanel" class="btn bg-emerald-600 hover:bg-emerald-700 small">Painel de Relatório</button>
            <button id="toggleTable" class="btn bg-slate-600 hover:bg-slate-700 small">Mostrar/Ocultar Tabela</button>
          </div>
        </div>

        <div id="tableWrapper" class="mt-4 overflow-x-auto">
          <table class="min-w-full table-fixed text-sm bg-white">
            <thead class="bg-gray-50">
              <tr>
                <th class="p-2">Hora</th>
                <th class="p-2">Tipo</th>
                <th class="p-2 text-right">Valor</th>
                <th class="p-2 text-center">Pessoas</th>
                <th class="p-2">Ações</th>
              </tr>
            </thead>
            <tbody id="entriesBody"></tbody>
          </table>
        </div>
      </section>

      <section id="reportPanel" class="bg-white p-4 rounded-xl shadow col-span-3 hidden">
        <div class="flex items-center justify-between mb-4 gap-4">
          <h2 class="font-semibold">Relatório Profissional - Resumo do Dia</h2>
          <div class="flex gap-2">
            <button id="downloadPdf" class="btn bg-cyan-600 hover:bg-cyan-700 small">Salvar PDF</button>
            <button id="closeReport" class="btn bg-gray-500 hover:bg-gray-600 small">Fechar</button>
          </div>
        </div>

        <div class="grid grid-cols-1 md:grid-cols-2 gap-4 mb-4">
          <div>
            <div class="text-sm text-gray-600 mb-1">Resumo</div>
            <div id="reportSummary" class="bg-gray-50 p-3 rounded"></div>
          </div>

          <div>
            <div class="text-sm text-gray-600 mb-1">Totais por forma de pagamento</div>
            <div id="reportTotals" class="bg-gray-50 p-3 rounded"></div>
          </div>
        </div>

        <div>
          <canvas id="reportChart" height="140"></canvas>
        </div>
      </section>

    </main>
  </div>

<script>
/* TIPOS */
const PRICE_TYPES = [
  { id: "20", label: "Dinheiro - R$20", price: 20, people: 1, kind: "Dinheiro" },
  { id: "30", label: "Dinheiro - R$30", price: 30, people: 1, kind: "Dinheiro" },
  { id: "50", label: "Dinheiro - R$50", price: 50, people: 2, kind: "Dinheiro" },

  { id: "20_credito", label: "Crédito - R$20", price: 20, people: 1, kind: "Crédito" },
  { id: "30_credito", label: "Crédito - R$30", price: 30, people: 1, kind: "Crédito" },
  { id: "50_credito", label: "Crédito - R$50", price: 50, people: 2, kind: "Crédito" },

  { id: "20_debito", label: "Débito - R$20", price: 20, people: 1, kind: "Débito" },
  { id: "30_debito", label: "Débito - R$30", price: 30, people: 1, kind: "Débito" },
  { id: "50_debito", label: "Débito - R$50", price: 50, people: 2, kind: "Débito" },

  { id: "20_pix", label: "Pix - R$20", price: 20, people: 1, kind: "Pix" },
  { id: "30_pix", label: "Pix - R$30", price: 30, people: 1, kind: "Pix" },
  { id: "50_pix", label: "Pix - R$50", price: 50, people: 2, kind: "Pix" },

  { id: "free50", label: "Free 50 Pessoas", price: 0, people: 1, kind: "Gratuidade" },
  { id: "free", label: "Lista (Free)", price: 0, people: 1, kind: "Gratuidade" },
  { id: "militar", label: "Militar", price: 0, people: 1, kind: "Gratuidade" },
  { id: "aniversario", label: "Aniversário", price: 0, people: 1, kind: "Gratuidade" }
];

/* BOTÕES */
const buttonsContainer = document.getElementById('buttonsContainer');
PRICE_TYPES.forEach(p => {
  const b = document.createElement('button');
  b.className = 'px-3 py-2 rounded-xl text-white text-sm shadow-md transition-all duration-200';

  if (p.kind === "Dinheiro") {
    b.classList.add("text-xs");
    b.classList.add("bg-green-600", "hover:bg-green-700");
  } else if (p.kind === "Crédito") {
    b.classList.add("bg-yellow-400", "hover:bg-yellow-700", "text-black");
  } else if (p.kind === "Débito") {
    b.classList.add("bg-blue-600", "hover:bg-blue-700");
  } else if (p.kind === "Pix") {
    b.classList.add("bg-gray-600", "hover:bg-gray-700");
  } else {
    b.classList.add("bg-slate-400", "hover:bg-slate-700");
  }

  b.textContent = p.label;
  b.onclick = () => addEntry(p.id);
  buttonsContainer.appendChild(b);
});

/* VARIÁVEIS */
const todayKey = () => new Date().toISOString().slice(0,10);
const storageKey = d => `portaria_${d}`;

let currentDate = todayKey();
let entries = [];
let isValueVisible = true; // Controle de visibilidade

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

/* Lógica do Olho */
toggleValueBtn.onclick = () => {
    isValueVisible = !isValueVisible;
    eyeIcon.textContent = isValueVisible ? '👁️' : '🙈';
    totalCollectedEl.classList.toggle('hidden-value', !isValueVisible);
};

/* LOAD */
function loadEntries() {
  const raw = localStorage.getItem(storageKey(currentDate));
  entries = raw ? JSON.parse(raw) : [];
  renderEntries();
}

function saveEntries() {
  localStorage.setItem(storageKey(currentDate), JSON.stringify(entries));
}

/* RENDER TABELA */
function renderEntries() {
  entriesBody.innerHTML = entries.length ? entries.map(e => `
    <tr class="border-t">
      <td class="p-2">${new Date(e.timestamp).toLocaleTimeString([], {hour:'2-digit', minute:'2-digit'})}</td>
      <td class="p-2">${e.type}</td>
      <td class="p-2 text-right">R$ ${e.price.toFixed(2)}</td>
      <td class="p-2 text-center">${e.people}</td>
      <td class="p-2 flex gap-2">
        <button class="px-2 py-1 bg-yellow-500 text-white rounded text-xs" onclick="editEntry(${e.id})">Editar</button>
        <button class="px-2 py-1 bg-red-500 text-white rounded text-xs" onclick="deleteEntry(${e.id})">Excluir</button>
      </td>
    </tr>`).join('') :
    `<tr><td colspan="5" class="text-center p-4 text-gray-400">Nenhum registro</td></tr>`;

  const totals = entries.reduce((acc, e) => {
    acc.people += e.people;
    acc.collected += e.price;
    return acc;
  }, { people:0, collected:0 });

  totalPeopleEl.textContent = totals.people;
  totalCollectedEl.textContent = `R$ ${totals.collected.toFixed(2)}`;
}

/* ADD ENTRY */
function addEntry(id) {
  const t = PRICE_TYPES.find(p => p.id === id);
  if (!t) return;

  entries.unshift({
    id: Date.now(),
    timestamp: new Date().toISOString(),
    type: t.label,
    price: t.price,
    people: t.people,
    kind: t.kind
  });

  saveEntries();
  renderEntries();
}

/* DELETE */
function deleteEntry(id) {
  if (!confirm('Deseja excluir este registro?')) return;
  entries = entries.filter(x => x.id !== id);
  saveEntries();
  renderEntries();
}

/* EDIT */
function editEntry(id) {
  const item = entries.find(x => x.id === id);
  if (!item) return;

  const newPrice = prompt('Editar preço (somente números):', item.price);
  if (newPrice === null) return;

  const newPeople = prompt('Editar número de pessoas:', item.people);
  if (newPeople === null) return;

  item.price = Number(newPrice) || 0;
  item.people = Number(newPeople) || 0;

  saveEntries();
  renderEntries();
}

/* CSV */
document.getElementById('exportCSV').addEventListener('click', () => {
  const headers = ['timestamp','type','price','people'];
  const rows = entries.map(e => [e.timestamp, e.type, e.price, e.people]);
  const csv = [headers, ...rows].map(r => r.join(',')).join('\n');
  const blob = new Blob([csv], { type: 'text/csv' });
  const url = URL.createObjectURL(blob);

  const a = document.createElement('a');
  a.href = url;
  a.download = `portaria_${currentDate}.csv`;
  a.click();
  URL.revokeObjectURL(url);
});

/* RESET */
document.getElementById('resetDay').addEventListener('click', () => {
  if (!confirm('Deseja realmente limpar todos os registros do dia?')) return;
  entries = [];
  saveEntries();
  renderEntries();
});

document.getElementById('saveDay').addEventListener('click', () => saveEntries());

/* RELATÓRIO */
document.getElementById('openReportPanel').addEventListener('click', () => {
  reportPanel.classList.remove('hidden');
  generateReportVisual();
});

document.getElementById('closeReport').addEventListener('click', () => {
  reportPanel.classList.add('hidden');
});

/* TABELA */
document.getElementById('toggleTable').addEventListener('click', () => {
  const w = document.getElementById('tableWrapper');
  w.style.display = (w.style.display === 'none') ? 'block' : 'none';
});

/* GERAR RESUMO + GRÁFICO COM VALORES */
function generateReportVisual() {
  const totals = entries.reduce((a,e) => {
    a.people += e.people;
    a.collected += e.price;
    return a;
  }, { people:0, collected:0 });

  const byKind = {};
  entries.forEach(e => {
    byKind[e.kind] = byKind[e.kind] || { count:0, collected:0 };
    byKind[e.kind].count += e.people;
    byKind[e.kind].collected += e.price;
  });

  const first = entries.length ? new Date(entries[entries.length-1].timestamp).toLocaleTimeString() : '-';
  const last = entries.length ? new Date(entries[0].timestamp).toLocaleTimeString() : '-';
  const avg = totals.people ? (totals.collected / totals.people) : 0;

  reportSummary.innerHTML = `
    <div class="text-sm">
      <p><strong>Data:</strong> ${currentDate}</p>
      <p><strong>Valor Total:</strong> R$ ${totals.collected.toFixed(2)}</p>
      <p><strong>Total de pessoas:</strong> ${totals.people}</p>
      <p><strong>Primeira entrada:</strong> ${first}</p>
      <p><strong>Última entrada:</strong> ${last}</p>
      <p><strong>Média por pessoa:</strong> R$ ${avg.toFixed(2)}</p>
    </div>
  `;

  let totalsHtml = '<div class="grid grid-cols-1 gap-2">';
  Object.keys(byKind).forEach(k => {
    totalsHtml += `<div class="flex justify-between">
      <div class="text-sm">${k}</div>
      <div class="font-semibold">R$ ${byKind[k].collected.toFixed(2)} / ${byKind[k].count}p</div>
    </div>`;
  });
  totalsHtml += '</div>';
  reportTotals.innerHTML = totalsHtml || '<div class="text-sm text-gray-500">Sem registros</div>';

  const labels = Object.keys(byKind);
  const data = labels.map(l => byKind[l].collected);

  if (reportChart) { reportChart.destroy(); reportChart = null; }

  reportChart = new Chart(reportChartEl, {
    type: 'doughnut',
    data: {
      labels,
      datasets: [{ data, backgroundColor: ['#10B981','#06B6D4','#F59E0B','#6366F1','#F43F5E'] }]
    },
    plugins: [ChartDataLabels],
    options: {
      responsive: true,
      plugins: {
        legend: { position: 'right' },
        datalabels: {
          color: '#ffffff',
          font: { weight: 'bold', size: 14 },
          formatter: (value) => `R$ ${value.toFixed(2)}`
        }
      }
    }
  });
}

/* PDF */
document.getElementById('downloadPdf').addEventListener('click', async () => {
  const panel = document.querySelector('#reportPanel');
  const closeBtn = document.getElementById('closeReport');
  const downloadBtn = document.getElementById('downloadPdf');

  closeBtn.style.display = 'none';
  downloadBtn.style.display = 'none';

  await new Promise(r => setTimeout(r, 200));

  const canvas = await html2canvas(panel, { scale: 2, useCORS: true, backgroundColor: '#ffffff' });
  const imgData = canvas.toDataURL('image/png');

  closeBtn.style.display = '';
  downloadBtn.style.display = '';

  const { jsPDF } = window.jspdf;
  const pdf = new jsPDF({ orientation: 'portrait', unit: 'pt', format: 'a4' });

  const pageWidth = pdf.internal.pageSize.getWidth();
  const margin = 20;
  const imgProps = pdf.getImageProperties(imgData);
  const imgWidth = pageWidth - margin*2;
  const imgHeight = (imgProps.height * imgWidth) / imgProps.width;

  pdf.addImage(imgData, 'PNG', margin, margin, imgWidth, imgHeight);
  pdf.setFontSize(10);
  pdf.text(`Relatório gerado: ${new Date().toLocaleString()}`, margin, pdf.internal.pageSize.getHeight() - 10);

  pdf.save(`relatorio_portaria_${currentDate}.pdf`);
});

/* DATA CHANGE */
currentDateEl.onchange = e => {
  currentDate = e.target.value;
  loadEntries();
};

function init() {
  currentDateEl.value = currentDate;
  loadEntries();
  renderEntries();
}
init();

window.deleteEntry = deleteEntry;
window.editEntry = editEntry;
window.addEntry = addEntry;
</script>

</body>
</html>
