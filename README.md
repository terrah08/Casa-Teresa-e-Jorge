<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Portaria - Casa Teresa e Jorge 🟩🩷</title>

<script src="https://cdn.tailwindcss.com"></script>
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script src="https://cdn.jsdelivr.net/npm/chartjs-plugin-datalabels@2.2.0/dist/chartjs-plugin-datalabels.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/jspdf@2.5.1/dist/jspdf.umd.min.js"></script>

<style>
  .btn { @apply px-3 py-2 rounded-xl text-white shadow-md transition-all duration-200 active:scale-95 flex items-center justify-center gap-2; }
  .hidden-value { filter: blur(6px); pointer-events: none; user-select: none; }
  #chartWrapper { width: 100%; max-width: 300px; margin: 0 auto; }
  .btn-disabled { @apply bg-gray-300 shadow-none cursor-not-allowed scale-100 opacity-60 !important; }
</style>
</head>
<body class="bg-gray-50 min-h-screen p-2 md:p-8 text-gray-800 font-sans">

  <div class="max-w-6xl mx-auto">
    <header class="flex flex-col md:flex-row md:items-center md:justify-between gap-4 mb-6 bg-white p-4 rounded-xl shadow-sm border-b-2 border-pink-200">
      <div>
        <h1 class="text-xl md:text-3xl font-extrabold text-gray-800">Casa Teresa e Jorge 🟩🩷</h1>
        <p class="text-xs text-gray-400 font-bold uppercase tracking-widest">Gestão de Portaria</p>
      </div>
      <div class="flex gap-2 items-center bg-gray-100 p-2 rounded-lg border border-gray-200">
        <input id="currentDate" type="date" class="border rounded px-2 py-1 text-sm font-bold bg-white" />
        <button id="resetDay" class="btn bg-red-500 hover:bg-red-600 p-2">🗑️</button>
      </div>
    </header>

    <main class="grid grid-cols-1 lg:grid-cols-3 gap-6">
      
      <section class="bg-white p-4 rounded-xl shadow col-span-1 border-t-4 border-blue-500 h-fit">
        <h2 class="font-bold text-gray-400 text-[10px] uppercase mb-4">Registros</h2>
        <div id="buttonsContainer" class="grid grid-cols-2 gap-2 mb-6"></div>
        <button id="btnOpenReport" class="w-full btn bg-blue-600 font-bold py-3 text-sm">
          <span>📄 GERAR RELATÓRIO PDF</span>
        </button>
      </section>

      <section class="bg-white p-4 rounded-xl shadow col-span-2 border-t-4 border-emerald-500">
        <div class="grid grid-cols-2 gap-4 mb-6">
          <div class="bg-emerald-50 p-4 rounded-xl border border-emerald-100 text-center">
            <div class="text-[10px] text-emerald-600 font-bold uppercase">Público Total</div>
            <div id="totalPeople" class="text-4xl font-black text-emerald-900">0</div>
          </div>
          <div class="bg-blue-50 p-4 rounded-xl border border-blue-100 relative text-center">
            <div class="flex justify-between items-center px-2">
              <div class="text-[10px] text-blue-600 font-bold uppercase">Caixa Total</div>
              <button onclick="toggleBlur()" class="text-sm"> <span id="eyeIcon">👁️</span> </button>
            </div>
            <div id="totalCollected" class="text-4xl font-black text-blue-900">R$ 0,00</div>
          </div>
        </div>

        <div class="overflow-hidden rounded-xl border border-gray-100 bg-white">
          <table class="w-full text-xs text-left">
            <thead class="bg-gray-50 text-gray-400 uppercase font-bold">
              <tr>
                <th class="p-3">Hora</th>
                <th class="p-3">Tipo</th>
                <th class="p-3 text-right">Valor</th>
                <th class="p-3 text-center">✕</th>
              </tr>
            </thead>
            <tbody id="entriesBody" class="divide-y divide-gray-50"></tbody>
          </table>
        </div>
      </section>

      <section id="reportPanel" class="bg-white p-6 rounded-xl shadow-2xl col-span-3 hidden border-b-8 border-emerald-600">
        <div class="flex flex-col sm:flex-row justify-between items-center mb-6 gap-4 border-b pb-4">
          <h2 class="text-xl font-black uppercase italic text-gray-700">Conferência de Fechamento</h2>
          <div class="flex gap-2 w-full sm:w-auto">
            <button id="downloadPdf" class="btn bg-emerald-600 font-bold flex-1 px-6 text-sm">💾 BAIXAR PDF</button>
            <button onclick="document.getElementById('reportPanel').classList.add('hidden')" class="btn bg-gray-400 px-4">X</button>
          </div>
        </div>

        <div class="grid grid-cols-1 md:grid-cols-3 gap-8">
          <div id="reportSummary" class="bg-gray-50 p-5 rounded-2xl border space-y-3"></div>
          <div class="space-y-4">
             <h3 class="text-xs font-black text-gray-400 uppercase tracking-widest border-b pb-1">Pessoas por Categoria</h3>
             <div id="reportTotals" class="space-y-1"></div>
          </div>
          <div id="chartWrapper">
            <canvas id="reportChart"></canvas>
          </div>
        </div>
      </section>

    </main>
  </div>

<script>
const PRICE_TYPES = [
  { id: "20_din", label: "Dinheiro R$20", sub: "Individual", price: 20, people: 1, kind: "Dinheiro" },
  { id: "30_din", label: "Dinheiro R$30", sub: "Individual", price: 30, people: 1, kind: "Dinheiro" },
  { id: "50_din", label: "Dinheiro R$50", sub: "Dupla", price: 50, people: 2, kind: "Dinheiro" },
  { id: "20_cart", label: "Cartão R$20", sub: "Individual", price: 20, people: 1, kind: "Cartão" },
  { id: "30_cart", label: "Cartão R$30", sub: "Individual", price: 30, people: 1, kind: "Cartão" },
  { id: "50_cart", label: "Cartão R$50", sub: "Dupla", price: 50, people: 2, kind: "Cartão" },
  { id: "20_pix", label: "Pix R$20", sub: "Individual", price: 20, people: 1, kind: "Pix" },
  { id: "30_pix", label: "Pix R$30", sub: "Individual", price: 30, people: 1, kind: "Pix" },
  { id: "50_pix", label: "Pix R$50", sub: "Dupla", price: 50, people: 2, kind: "Pix" },
  { id: "f100", label: "100 Pessoas", sub: "FREE", price: 0, people: 1, kind: "Gratuidade", isCounter: true, limit: 100 },
  { id: "list", label: "Lista", sub: "", price: 0, people: 1, kind: "Gratuidade" },
  { id: "Aniv", label: "Aniversário", sub: "", price: 0, people: 1, kind: "Gratuidade" },
  { id: "milt", label: "Militar", sub: "", price: 0, people: 1, kind: "Gratuidade" }
];

let entries = [];
let isValueVisible = true;
let chartInstance = null;
let currentDate = new Date().toISOString().slice(0,10);

function renderButtons() {
  const container = document.getElementById('buttonsContainer');
  container.innerHTML = '';
  const count100 = entries.filter(e => e.type.includes("100 Pessoas")).length;

  PRICE_TYPES.forEach(p => {
    const b = document.createElement('button');
    let sub = p.sub;
    let disabled = false;

    if (p.isCounter) {
        const rest = p.limit - count100;
        sub = rest > 0 ? `Restam ${rest}` : "ESGOTADO";
        if (rest <= 0) disabled = true;
    }

    b.className = `p-2 h-14 rounded-lg text-white flex flex-col items-center justify-center transition-all ${disabled ? 'btn-disabled' : (p.kind === 'Dinheiro' ? 'bg-green-600' : p.kind === 'Cartão' ? 'bg-amber-500' : p.kind === 'Pix' ? 'bg-cyan-600' : 'bg-gray-400')}`;
    let subHtml = p.sub || p.isCounter ? `<span class="text-[9px] mt-1 font-black bg-black/20 px-2 rounded-full border border-white/10">${sub}</span>` : "";
    b.innerHTML = `<span class="text-[10px] font-black uppercase leading-none">${p.label}</span>${subHtml}`;
    if (!disabled) b.onclick = () => addEntry(p);
    container.appendChild(b);
  });
}

function load() {
  const data = localStorage.getItem(`ctj_final_${currentDate}`);
  entries = data ? JSON.parse(data) : [];
  render();
}

function render() {
  renderButtons();
  const body = document.getElementById('entriesBody');
  const blurClass = isValueVisible ? '' : 'hidden-value';
  body.innerHTML = entries.map(e => `
    <tr class="hover:bg-gray-50 border-b border-gray-50">
      <td class="p-3 text-gray-400 font-mono">${e.time}</td>
      <td class="p-3 font-bold text-gray-700 uppercase text-[10px]">${e.type}</td>
      <td class="p-3 text-right font-black ${blurClass}">R$ ${e.price.toFixed(2)}</td>
      <td class="p-3 text-center"><button onclick="deleteEntry(${e.id})" class="text-red-200 hover:text-red-500">✕</button></td>
    </tr>
  `).join('');

  const totals = entries.reduce((a, b) => ({ p: a.p + b.people, v: a.v + b.price }), { p: 0, v: 0 });
  document.getElementById('totalPeople').textContent = totals.p;
  document.getElementById('totalCollected').textContent = `R$ ${totals.v.toFixed(2)}`;
}

function addEntry(p) {
  const fullLabel = p.sub ? `${p.label} ${p.sub}` : p.label;
  entries.unshift({ id: Date.now(), time: new Date().toLocaleTimeString([], {hour:'2-digit', minute:'2-digit'}), timestamp: new Date().getTime(), type: fullLabel, price: p.price, people: p.people, kind: p.kind });
  localStorage.setItem(`ctj_final_${currentDate}`, JSON.stringify(entries));
  render();
}

window.deleteEntry = (id) => {
  if(confirm('Remover registro?')) { entries = entries.filter(e => e.id !== id); localStorage.setItem(`ctj_final_${currentDate}`, JSON.stringify(entries)); render(); }
};

document.getElementById('btnOpenReport').onclick = () => {
  if(entries.length === 0) return alert("Sem dados.");
  document.getElementById('reportPanel').classList.remove('hidden');
  
  const totals = entries.reduce((a, b) => ({ p: a.p + b.people, v: a.v + b.price }), { p: 0, v: 0 });
  const stats = {};
  entries.forEach(e => {
    if(!stats[e.kind]) stats[e.kind] = { money: 0, people: 0 };
    stats[e.kind].money += e.price;
    stats[e.kind].people += e.people;
  });

  const times = entries.map(e => e.timestamp).sort();
  const first = new Date(times[0]).toLocaleTimeString([], {hour:'2-digit', minute:'2-digit'});
  const last = new Date(times[times.length-1]).toLocaleTimeString([], {hour:'2-digit', minute:'2-digit'});

  document.getElementById('reportSummary').innerHTML = `
    <h3 class="font-black text-emerald-700 uppercase text-[10px]">Resumo do Evento</h3>
    <div class="flex justify-between text-xs"><span>Início:</span><b>${first}</b></div>
    <div class="flex justify-between text-xs"><span>Fim:</span><b>${last}</b></div>
    <div class="flex justify-between text-sm border-t pt-2 mt-2"><span>Ticket Médio:</span><b class="text-blue-500">R$ ${(totals.v/totals.p).toFixed(2)}</b></div>
    <div class="flex justify-between text-2xl font-black mt-4 border-t-2 pt-2"><span>CAIXA:</span><span class="text-emerald-600">R$ ${totals.v.toFixed(2)}</span></div>
  `;

  document.getElementById('reportTotals').innerHTML = Object.entries(stats).map(([k, v]) => `
    <div class="flex justify-between items-center border-b py-2">
      <div>
        <div class="text-[10px] font-black uppercase text-gray-700">${k}</div>
        <div class="text-[9px] text-gray-400 font-bold">${v.people} pessoas</div>
      </div>
      <div class="font-black text-gray-600 text-xs">R$ ${v.money.toFixed(2)}</div>
    </div>
  `).join('');

  if(chartInstance) chartInstance.destroy();
  chartInstance = new Chart(document.getElementById('reportChart'), {
    type: 'doughnut',
    data: { labels: Object.keys(stats), datasets: [{ data: Object.values(stats).map(v => v.money), backgroundColor: ['#10b981', '#f59e0b', '#06b6d4', '#cbd5e1'] }] },
    plugins: [ChartDataLabels],
    options: { responsive: true, animation: false, plugins: { legend: { position: 'bottom', labels: { boxWidth: 10, font: { size: 9 } } }, datalabels: { color: '#000', font: { weight: 'bold', size: 9 }, formatter: v => v > 0 ? `R$${v.toFixed(0)}` : '' } } }
  });
  document.getElementById('reportPanel').scrollIntoView({ behavior: 'smooth' });
};

document.getElementById('downloadPdf').onclick = () => {
  const { jsPDF } = window.jspdf;
  const doc = new jsPDF();
  const totals = entries.reduce((a, b) => ({ p: a.p + b.people, v: a.v + b.price }), { p: 0, v: 0 });
  const stats = {};
  entries.forEach(e => {
    if(!stats[e.kind]) stats[e.kind] = { money: 0, people: 0 };
    stats[e.kind].money += e.price;
    stats[e.kind].people += e.people;
  });

  doc.setFontSize(22); doc.text("CASA TERESA E JORGE", 105, 20, { align: "center" });
  doc.setFontSize(10); doc.text(`RELATORIO DE FECHAMENTO - ${currentDate.split('-').reverse().join('/')}`, 105, 28, { align: "center" });
  doc.line(15, 32, 195, 32);

  doc.setFontSize(12); doc.setFont(undefined, 'bold');
  doc.text("RESUMO GERAL", 15, 45);
  doc.setFontSize(10); doc.setFont(undefined, 'normal');
  doc.text(`Publico Total: ${totals.p} pessoas`, 15, 55);
  doc.text(`Valor Total Arrecadado: R$ ${totals.v.toFixed(2)}`, 15, 62);
  
  doc.setFont(undefined, 'bold');
  doc.text("DETALHAMENTO POR CATEGORIA:", 15, 75);
  let currentY = 85;
  Object.entries(stats).forEach(([k, v]) => {
    doc.setFont(undefined, 'bold');
    doc.text(`${k}:`, 20, currentY);
    doc.setFont(undefined, 'normal');
    doc.text(`${v.people} pessoas`, 60, currentY);
    doc.text(`Subtotal: R$ ${v.money.toFixed(2)}`, 110, currentY);
    currentY += 10;
  });

  doc.save(`Fechamento_${currentDate}.pdf`);
};

document.getElementById('resetDay').onclick = () => { if(confirm('Zerar hoje?')) { entries=[]; localStorage.removeItem(`ctj_final_${currentDate}`); render(); } };
const dateEl = document.getElementById('currentDate');
dateEl.value = currentDate;
dateEl.onchange = (e) => { currentDate = e.target.value; load(); };
load();
</script>
</body>
</html>
