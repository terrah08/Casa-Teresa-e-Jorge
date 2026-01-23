<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Portaria - Casa Teresa e Jorge🟩🩷</title>

<script src="https://cdn.tailwindcss.com"></script>
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script src="https://cdn.jsdelivr.net/npm/chartjs-plugin-datalabels@2.2.0/dist/chartjs-plugin-datalabels.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/jspdf@2.5.1/dist/jspdf.umd.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/html2canvas@1.4.1/dist/html2canvas.min.js"></script>

<style>
  .btn { @apply px-3 py-2 rounded-xl text-white shadow-md transition-all duration-200 active:scale-95; }
  .small { font-size: .85rem; }
  .hidden-value { filter: blur(6px); pointer-events: none; user-select: none; }
  /* Ajuste para o PDF não cortar no celular */
  #reportPanel { min-width: 350px; }
</style>
</head>
<body class="bg-gray-50 min-h-screen p-2 md:p-8">

  <div class="max-w-6xl mx-auto">
    <header class="flex flex-col md:flex-row md:items-center md:justify-between gap-4 mb-6 bg-white p-4 rounded-xl shadow-sm">
      <div>
        <h1 class="text-xl md:text-3xl font-extrabold text-gray-800">Casa Teresa e Jorge 🟩🩷</h1>
        <div class="text-xs text-gray-500 font-bold uppercase tracking-wider">Gestão de Portaria Profissional</div>
      </div>

      <div class="flex gap-2 items-center bg-gray-100 p-2 rounded-lg">
        <input id="currentDate" type="date" class="border rounded px-2 py-1 text-sm" />
        <button id="resetDay" class="btn bg-red-500 hover:bg-red-600 p-2" title="Resetar Dia">🗑️</button>
      </div>
    </header>

    <main class="grid grid-cols-1 lg:grid-cols-3 gap-6">
      
      <section class="bg-white p-4 rounded-xl shadow col-span-1">
        <h2 class="font-bold text-gray-700 mb-4 flex items-center gap-2">
          <span>📝 Registrar</span>
        </h2>
        <div id="buttonsContainer" class="grid grid-cols-2 gap-2 mb-6"></div>
        <div class="space-y-2 border-t pt-4">
          <button id="generateReport" class="w-full btn bg-blue-600 font-bold">📄 Gerar Relatório PDF</button>
          <button id="exportCSV" class="w-full btn bg-indigo-500 small opacity-80">Exportar CSV</button>
        </div>
      </section>

      <section class="bg-white p-4 rounded-xl shadow col-span-2">
        <div class="flex flex-wrap gap-4 mb-6">
          <div class="flex-1 bg-gray-50 p-3 rounded-xl border border-gray-100">
            <div class="text-[10px] text-gray-400 font-bold uppercase">Pessoas</div>
            <div id="totalPeople" class="text-2xl font-black text-gray-800">0</div>
          </div>
          <div class="flex-1 bg-gray-50 p-3 rounded-xl border border-gray-100 relative">
            <div class="flex justify-between items-center">
              <div class="text-[10px] text-gray-400 font-bold uppercase">Total</div>
              <button id="toggleValue" class="text-lg"> <span id="eyeIcon">👁️</span> </button>
            </div>
            <div id="totalCollected" class="text-2xl font-black text-emerald-600">R$ 0,00</div>
          </div>
        </div>

        <div class="flex justify-between items-center mb-4">
          <h3 class="font-bold text-gray-700">Últimas Entradas</h3>
          <button id="toggleTable" class="text-xs font-bold text-blue-600">Ver/Ocultar Lista</button>
        </div>

        <div id="tableWrapper" class="overflow-x-auto rounded-lg border border-gray-100">
          <table class="min-w-full text-sm">
            <thead class="bg-gray-50 text-gray-500">
              <tr>
                <th class="p-2 text-left">Hora</th>
                <th class="p-2 text-left">Tipo</th>
                <th class="p-2 text-right">Valor</th>
                <th class="p-2 text-center">Ações</th>
              </tr>
            </thead>
            <tbody id="entriesBody" class="divide-y divide-gray-50"></tbody>
          </table>
        </div>
      </section>

      <section id="reportPanel" class="bg-white p-6 rounded-xl shadow-2xl col-span-3 hidden border-t-8 border-emerald-500">
        <div class="flex justify-between items-center mb-6">
          <h2 class="text-xl font-black text-gray-800 uppercase italic">Fechamento de Caixa</h2>
          <div class="flex gap-2">
            <button id="downloadPdf" class="btn bg-emerald-600 font-bold">💾 Baixar PDF</button>
            <button id="closeReport" class="btn bg-gray-400">Fechar</button>
          </div>
        </div>

        <div class="grid grid-cols-1 md:grid-cols-2 gap-8">
          <div id="reportSummary" class="space-y-2 text-gray-700 bg-gray-50 p-4 rounded-xl"></div>
          <div id="reportTotals" class="space-y-2"></div>
          <div class="col-span-1 md:col-span-2 flex justify-center py-4">
            <div style="width: 100%; max-width: 400px;">
              <canvas id="reportChart"></canvas>
            </div>
          </div>
        </div>
      </section>
    </main>
  </div>

<script>
const PRICE_TYPES = [
  { id: "20_din", label: "Dinheiro R$20 - Individual", price: 20, people: 1, kind: "Dinheiro" },
  { id: "30_din", label: "Dinheiro R$30 - Individual", price: 30, people: 1, kind: "Dinheiro" },
  { id: "50_din", label: "Dinheiro R$50 - Dupla", price: 50, people: 2, kind: "Dinheiro" },
  { id: "20_cart", label: "Cartão R$20 - Individual", price: 20, people: 1, kind: "Cartão" },
  { id: "30_cart", label: "Cartão R$30 - Individual", price: 30, people: 1, kind: "Cartão" },
  { id: "50_cart", label: "Cartão R$50 - Dupla", price: 50, people: 2, kind: "Cartão" },
  { id: "20_pix", label: "Pix R$20 - Individual", price: 20, people: 1, kind: "Pix" },
  { id: "30_pix", label: "Pix R$30 - Individual", price: 30, people: 1, kind: "Pix" },
  { id: "50_pix", label: "Pix R$50 - Dupla", price: 50, people: 2, kind: "Pix" },
  { id: "free100", label: "100 Pessoas FREE", price: 0, people: 100, kind: "Gratuidade" },
  { id: "free", label: "Lista (Individual)", price: 0, people: 1, kind: "Gratuidade" },
  { id: "Aniv", label: "Aniversário", price: 0, people: 1, kind: "Gratuidade" },
  { id: "militar", label: "Militar", price: 0, people: 1, kind: "Gratuidade" }
];

const buttonsContainer = document.getElementById('buttonsContainer');
PRICE_TYPES.forEach(p => {
  const b = document.createElement('button');
  b.className = 'p-2 rounded-lg text-white text-[10px] font-bold shadow uppercase transition-all';
  if (p.kind === "Dinheiro") b.classList.add("bg-green-600");
  else if (p.kind === "Cartão") b.classList.add("bg-amber-500");
  else if (p.kind === "Pix") b.classList.add("bg-cyan-600");
  else if (p.id === "free100") b.classList.add("bg-purple-600");
  else b.classList.add("bg-gray-400");
  
  b.textContent = p.label;
  b.onclick = () => addEntry(p.id);
  buttonsContainer.appendChild(b);
});

let currentDate = new Date().toISOString().slice(0,10);
let entries = [];
let isValueVisible = true;
let reportChart = null;

const currentDateEl = document.getElementById('currentDate');
currentDateEl.value = currentDate;

// Toggle Visibilidade
document.getElementById('toggleValue').onclick = () => {
  isValueVisible = !isValueVisible;
  document.getElementById('eyeIcon').textContent = isValueVisible ? '👁️' : '🙈';
  document.getElementById('totalCollected').classList.toggle('hidden-value', !isValueVisible);
  document.querySelectorAll('.val-td').forEach(td => td.classList.toggle('hidden-value', !isValueVisible));
};

function load() {
  const data = localStorage.getItem(`pj_${currentDate}`);
  entries = data ? JSON.parse(data) : [];
  render();
}

function render() {
  const body = document.getElementById('entriesBody');
  const blur = isValueVisible ? '' : 'hidden-value';
  
  body.innerHTML = entries.map(e => `
    <tr>
      <td class="p-2 text-gray-400">${new Date(e.ts).toLocaleTimeString([], {hour:'2-digit', minute:'2-digit'})}</td>
      <td class="p-2 font-bold text-gray-700">${e.type}</td>
      <td class="p-2 text-right font-mono val-td ${blur}">R$ ${e.price.toFixed(2)}</td>
      <td class="p-2 text-center">
        <button onclick="deleteEntry(${e.id})" class="text-red-400">✕</button>
      </td>
    </tr>
  `).join('');

  const totals = entries.reduce((a, b) => ({ p: a.p + b.people, v: a.v + b.price }), { p: 0, v: 0 });
  document.getElementById('totalPeople').textContent = totals.p;
  document.getElementById('totalCollected').textContent = `R$ ${totals.v.toFixed(2)}`;
}

function addEntry(id) {
  const t = PRICE_TYPES.find(x => x.id === id);
  entries.unshift({ id: Date.now(), ts: new Date().toISOString(), type: t.label, price: t.price, people: t.people, kind: t.kind });
  localStorage.setItem(`pj_${currentDate}`, JSON.stringify(entries));
  render();
}

window.deleteEntry = (id) => {
  if(!confirm('Excluir?')) return;
  entries = entries.filter(e => e.id !== id);
  localStorage.setItem(`pj_${currentDate}`, JSON.stringify(entries));
  render();
};

document.getElementById('generateReport').onclick = () => {
  document.getElementById('reportPanel').classList.remove('hidden');
  generateVisual();
  document.getElementById('reportPanel').scrollIntoView({ behavior: 'smooth' });
};

function generateVisual() {
  const totals = entries.reduce((a, b) => ({ p: a.p + b.people, v: a.v + b.price }), { p: 0, v: 0 });
  const byKind = {};
  entries.forEach(e => {
    byKind[e.kind] = byKind[e.kind] || { p: 0, v: 0 };
    byKind[e.kind].p += e.people;
    byKind[e.kind].v += e.price;
  });

  document.getElementById('reportSummary').innerHTML = `
    <h3 class="font-black text-lg text-emerald-700">RESUMO GERAL</h3>
    <p>Data: <b>${currentDate.split('-').reverse().join('/')}</b></p>
    <p>Pessoas Totais: <b>${totals.p}</b></p>
    <p>Arrecadação: <b>R$ ${totals.v.toFixed(2)}</b></p>
    <p>Ticket Médio: <b>R$ ${totals.p ? (totals.v / totals.p).toFixed(2) : '0.00'}</b></p>
  `;

  document.getElementById('reportTotals').innerHTML = Object.entries(byKind).map(([k, v]) => `
    <div class="flex justify-between border-b py-1">
      <span class="text-gray-500 font-bold">${k}</span>
      <span class="font-mono">R$ ${v.v.toFixed(2)} (${v.p}p)</span>
    </div>
  `).join('');

  if(reportChart) reportChart.destroy();
  const ctx = document.getElementById('reportChart').getContext('2d');
  reportChart = new Chart(ctx, {
    type: 'doughnut',
    data: {
      labels: Object.keys(byKind),
      datasets: [{ data: Object.values(byKind).map(x => x.v), backgroundColor: ['#059669', '#d97706', '#0891b2', '#7c3aed', '#4b5563'] }]
    },
    plugins: [ChartDataLabels],
    options: { 
      plugins: { 
        legend: { position: 'bottom' },
        datalabels: { color: '#fff', font: { weight: 'bold' }, formatter: (v) => v > 0 ? `R$${v}` : '' }
      }
    }
  });
}

document.getElementById('downloadPdf').onclick = async () => {
  const btnPdf = document.getElementById('downloadPdf');
  const btnClose = document.getElementById('closeReport');
  btnPdf.style.display = 'none';
  btnClose.style.display = 'none';

  // Pequeno delay para o Chart.js terminar a animação
  await new Promise(r => setTimeout(r, 500));

  const panel = document.getElementById('reportPanel');
  const canvas = await html2canvas(panel, { 
    scale: 2, 
    useCORS: true, 
    logging: false,
    windowWidth: panel.scrollWidth,
    windowHeight: panel.scrollHeight
  });
  
  const imgData = canvas.toDataURL('image/png');
  const { jsPDF } = window.jspdf;
  const pdf = new jsPDF('p', 'mm', 'a4');
  const imgW = 190;
  const imgH = (canvas.height * imgW) / canvas.width;

  pdf.setFontSize(18);
  pdf.text("RELATÓRIO DE PORTARIA", 10, 20);
  pdf.addImage(imgData, 'PNG', 10, 30, imgW, imgH);
  pdf.save(`relatorio_${currentDate}.pdf`);

  btnPdf.style.display = '';
  btnClose.style.display = '';
};

document.getElementById('closeReport').onclick = () => document.getElementById('reportPanel').classList.add('hidden');
document.getElementById('toggleTable').onclick = () => document.getElementById('tableWrapper').classList.toggle('hidden');
document.getElementById('resetDay').onclick = () => { if(confirm('Limpar tudo?')) { entries=[]; localStorage.removeItem(`pj_${currentDate}`); render(); } };
currentDateEl.onchange = (e) => { currentDate = e.target.value; load(); };

load();
</script>
</body>
</html>
