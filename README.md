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
<script src="https://cdn.jsdelivr.net/npm/html2canvas@1.4.1/dist/html2canvas.min.js"></script>

<style>
  .btn { @apply rounded-xl text-white shadow-md transition-all duration-200 active:scale-95 flex flex-col items-center justify-center text-center; }
  .hidden-value { filter: blur(6px); pointer-events: none; user-select: none; }
  
  /* Ajuste do título para não quebrar */
  .title-nowrap { white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }

  /* Estilo dos textos dentro do botão (2 linhas) */
  .label-main { font-size: 10px; font-weight: 900; text-transform: uppercase; white-space: nowrap; line-height: 1; }
  .label-sub { font-size: 8px; font-weight: 500; opacity: 0.85; white-space: nowrap; margin-top: 2px; }

  @media (min-width: 640px) {
    .label-main { font-size: 12px; }
    .label-sub { font-size: 9px; }
  }

  /* Container do gráfico para o PDF */
  .chart-container { position: relative; height: 280px; width: 100%; background: white; }
</style>
</head>
<body class="bg-gray-50 min-h-screen p-2 md:p-8 text-gray-800">

  <div class="max-w-6xl mx-auto">
    <header class="flex flex-col md:flex-row md:items-center md:justify-between gap-3 mb-6 bg-white p-4 rounded-xl shadow-sm">
      <div class="overflow-hidden">
        <h1 class="text-lg sm:text-xl md:text-2xl font-black title-nowrap">
          Casa Teresa e Jorge <span class="inline-block">🟩🩷</span>
        </h1>
        <div class="text-[10px] text-gray-400 font-bold uppercase tracking-widest">Gestão de Portaria</div>
      </div>

      <div class="flex gap-2 items-center bg-gray-50 p-1.5 rounded-lg border border-gray-100 self-end md:self-auto">
        <input id="currentDate" type="date" class="border-0 bg-transparent text-xs font-bold focus:ring-0" />
        <button id="resetDay" class="text-red-400 hover:text-red-600 px-2">🗑️</button>
      </div>
    </header>

    <main class="grid grid-cols-1 lg:grid-cols-3 gap-6">
      
      <section class="bg-white p-4 rounded-xl shadow col-span-1 border-t-4 border-blue-500">
        <h2 class="font-bold text-gray-700 mb-4 text-xs uppercase tracking-wider">📝 Lançar Entrada</h2>
        <div id="buttonsContainer" class="grid grid-cols-2 gap-2 mb-6"></div>
        <div class="pt-4 border-t">
          <button id="generateReport" class="w-full py-3 bg-blue-600 rounded-xl text-white font-bold text-sm shadow-lg active:scale-95 transition-all">📄 GERAR RELATÓRIO PDF</button>
        </div>
      </section>

      <section class="bg-white p-4 rounded-xl shadow col-span-2 border-t-4 border-emerald-500">
        <div class="grid grid-cols-2 gap-4 mb-6">
          <div class="bg-emerald-50 p-3 rounded-xl border border-emerald-100 text-center sm:text-left">
            <div class="text-[10px] text-emerald-600 font-bold uppercase">Público Total</div>
            <div id="totalPeople" class="text-2xl font-black text-emerald-900">0</div>
          </div>
          <div class="bg-blue-50 p-3 rounded-xl border border-blue-100 relative text-center sm:text-left">
            <div class="flex justify-between items-center">
              <div class="text-[10px] text-blue-600 font-bold uppercase">Caixa</div>
              <button id="toggleValue" class="text-lg leading-none"> <span id="eyeIcon">👁️</span> </button>
            </div>
            <div id="totalCollected" class="text-2xl font-black text-blue-900">R$ 0,00</div>
          </div>
        </div>

        <div id="tableWrapper" class="overflow-x-auto rounded-lg">
          <table class="min-w-full text-xs">
            <thead class="bg-gray-50 text-gray-400 uppercase">
              <tr>
                <th class="p-2 text-left">Hora</th>
                <th class="p-2 text-left">Tipo</th>
                <th class="p-2 text-right">Valor</th>
                <th class="p-2 text-center">✕</th>
              </tr>
            </thead>
            <tbody id="entriesBody" class="divide-y divide-gray-50"></tbody>
          </table>
        </div>
      </section>

      <section id="reportPanel" class="bg-white p-6 rounded-xl shadow-2xl col-span-3 hidden border-b-8 border-emerald-500">
        <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center gap-4 mb-8 border-b pb-4">
          <div>
            <h2 class="text-xl font-black text-gray-800 uppercase italic">Fechamento de Caixa</h2>
            <p class="text-xs text-gray-400 font-bold uppercase">Casa Teresa e Jorge</p>
          </div>
          <div class="flex gap-2 w-full sm:w-auto">
            <button id="downloadPdf" class="flex-1 px-4 py-2 bg-emerald-600 text-white rounded-lg font-bold text-xs">💾 BAIXAR PDF</button>
            <button id="closeReport" class="px-4 py-2 bg-gray-400 text-white rounded-lg text-xs">FECHAR</button>
          </div>
        </div>

        <div class="grid grid-cols-1 md:grid-cols-2 gap-8">
          <div id="reportSummary" class="space-y-2 text-sm bg-gray-50 p-4 rounded-xl border"></div>
          <div id="reportTotals" class="space-y-1"></div>
          <div class="col-span-1 md:col-span-2 flex justify-center py-4">
            <div class="chart-container">
              <canvas id="reportChart"></canvas>
            </div>
          </div>
        </div>
      </section>
    </main>
  </div>

<script>
const PRICE_TYPES = [
  { id: "20_din", main: "Dinheiro R$20", sub: "Individual", price: 20, people: 1, kind: "Dinheiro" },
  { id: "30_din", main: "Dinheiro R$30", sub: "Individual", price: 30, people: 1, kind: "Dinheiro" },
  { id: "50_din", main: "Dinheiro R$50", sub: "Dupla", price: 50, people: 2, kind: "Dinheiro" },
  { id: "20_car", main: "Cartão R$20", sub: "Individual", price: 20, people: 1, kind: "Cartão" },
  { id: "30_car", main: "Cartão R$30", sub: "Individual", price: 30, people: 1, kind: "Cartão" },
  { id: "50_car", main: "Cartão R$50", sub: "Dupla", price: 50, people: 2, kind: "Cartão" },
  { id: "20_pix", main: "Pix R$20", sub: "Individual", price: 20, people: 1, kind: "Pix" },
  { id: "30_pix", main: "Pix R$30", sub: "Individual", price: 30, people: 1, kind: "Pix" },
  { id: "50_pix", main: "Pix R$50", sub: "Dupla", price: 50, people: 2, kind: "Pix" },
  { id: "f100", main: "100 Pessoas", sub: "FREE", price: 0, people: 100, kind: "Gratuidade" },
  { id: "list", main: "Lista", sub: "Individual", price: 0, people: 1, kind: "Gratuidade" },
  { id: "aniv", main: "Aniversário", sub: "Isento", price: 0, people: 1, kind: "Gratuidade" },
  { id: "milt", main: "Militar", sub: "Isento", price: 0, people: 1, kind: "Gratuidade" }
];

const buttonsContainer = document.getElementById('buttonsContainer');
PRICE_TYPES.forEach(p => {
  const b = document.createElement('button');
  b.className = 'btn h-14'; 
  if (p.kind === "Dinheiro") b.classList.add("bg-green-600");
  else if (p.kind === "Cartão") b.classList.add("bg-amber-500");
  else if (p.kind === "Pix") b.classList.add("bg-cyan-600");
  else if (p.id === "f100") b.classList.add("bg-purple-600");
  else b.classList.add("bg-gray-400");
  
  b.innerHTML = `<span class="label-main">${p.main}</span><span class="label-sub">${p.sub}</span>`;
  b.onclick = () => addEntry(p.id);
  buttonsContainer.appendChild(b);
});

let currentDate = new Date().toISOString().slice(0,10);
let entries = [];
let isValueVisible = true;
let reportChart = null;

const currentDateEl = document.getElementById('currentDate');
currentDateEl.value = currentDate;

document.getElementById('toggleValue').onclick = () => {
  isValueVisible = !isValueVisible;
  document.getElementById('eyeIcon').textContent = isValueVisible ? '👁️' : '🙈';
  document.getElementById('totalCollected').classList.toggle('hidden-value', !isValueVisible);
};

function load() {
  const data = localStorage.getItem(`ctj_final_${currentDate}`);
  entries = data ? JSON.parse(data) : [];
  render();
}

function render() {
  const body = document.getElementById('entriesBody');
  const blur = isValueVisible ? '' : 'hidden-value';
  body.innerHTML = entries.map(e => `
    <tr>
      <td class="p-2 text-gray-400 font-mono">${new Date(e.ts).toLocaleTimeString([], {hour:'2-digit', minute:'2-digit'})}</td>
      <td class="p-2 font-bold text-gray-600 text-[10px] uppercase">${e.type}</td>
      <td class="p-2 text-right font-black ${blur}">R$ ${e.price.toFixed(2)}</td>
      <td class="p-2 text-center"><button onclick="deleteEntry(${e.id})" class="text-red-300 px-2">✕</button></td>
    </tr>
  `).join('');

  const totals = entries.reduce((a, b) => ({ p: a.p + b.people, v: a.v + b.price }), { p: 0, v: 0 });
  document.getElementById('totalPeople').textContent = totals.p;
  document.getElementById('totalCollected').textContent = `R$ ${totals.v.toFixed(2)}`;
}

function addEntry(id) {
  const t = PRICE_TYPES.find(x => x.id === id);
  entries.unshift({ id: Date.now(), ts: new Date().toISOString(), type: t.main + ' ' + t.sub, price: t.price, people: t.people, kind: t.kind });
  localStorage.setItem(`ctj_final_${currentDate}`, JSON.stringify(entries));
  render();
}

window.deleteEntry = (id) => {
  if(!confirm('Excluir registro?')) return;
  entries = entries.filter(e => e.id !== id);
  localStorage.setItem(`ctj_final_${currentDate}`, JSON.stringify(entries));
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
    <h3 class="font-black text-emerald-700 uppercase border-b mb-1 text-xs">Resumo Geral</h3>
    <p class="flex justify-between">Público: <b>${totals.p}p</b></p>
    <p class="flex justify-between text-base">Caixa: <b>R$ ${totals.v.toFixed(2)}</b></p>
  `;

  document.getElementById('reportTotals').innerHTML = Object.entries(byKind).map(([k, v]) => `
    <div class="flex justify-between border-b py-2 text-[10px]">
      <span class="text-gray-400 font-bold uppercase">${k}</span>
      <span class="font-black text-gray-700">R$ ${v.v.toFixed(2)} (${v.p}p)</span>
    </div>
  `).join('');

  if(reportChart) reportChart.destroy();
  
  // Ajuste para celular renderizar o canvas corretamente para captura
  const ctx = document.getElementById('reportChart').getContext('2d', { willReadFrequently: true });
  
  reportChart = new Chart(ctx, {
    type: 'doughnut',
    data: {
      labels: Object.keys(byKind),
      datasets: [{ 
        data: Object.values(byKind).map(x => x.v), 
        backgroundColor: ['#10b981', '#f59e0b', '#06b6d4', '#8b5cf6', '#64748b'] 
      }]
    },
    plugins: [ChartDataLabels],
    options: { 
      responsive: true,
      maintainAspectRatio: false,
      animation: false, // Desliga animação para o PDF não sair branco
      plugins: { 
        legend: { position: 'bottom', labels: { boxWidth: 8, font: { size: 9 } } },
        datalabels: { 
          color: '#fff', 
          font: { weight: 'bold', size: 10 }, 
          formatter: (v) => v > 0 ? `R$${v.toFixed(0)}` : '' 
        }
      }
    }
  });
  reportChart.update(); // Força desenho síncrono
}

document.getElementById('downloadPdf').onclick = async () => {
  const btnPdf = document.getElementById('downloadPdf');
  const btnClose = document.getElementById('closeReport');
  
  btnPdf.style.visibility = 'hidden';
  btnClose.style.visibility = 'hidden';

  // Pausa essencial para navegadores mobile terminarem o render
  await new Promise(r => setTimeout(r, 800));

  const panel = document.getElementById('reportPanel');
  
  try {
    const canvas = await html2canvas(panel, { 
      scale: 1.5, 
      useCORS: true,
      backgroundColor: "#ffffff",
      logging: false 
    });
    
    const imgData = canvas.toDataURL('image/png', 1.0);
    const { jsPDF } = window.jspdf;
    const pdf = new jsPDF('p', 'mm', 'a4');
    
    const imgW = 190;
    const imgH = (canvas.height * imgW) / canvas.width;

    pdf.setFontSize(14);
    pdf.text(`RELATORIO DE CAIXA - ${currentDate.split('-').reverse().join('/')}`, 10, 15);
    pdf.addImage(imgData, 'PNG', 10, 20, imgW, imgH);
    pdf.save(`caixa_${currentDate}.pdf`);
  } catch (err) {
    alert("Erro ao gerar PDF. Tente novamente.");
  } finally {
    btnPdf.style.visibility = 'visible';
    btnClose.style.visibility = 'visible';
  }
};

document.getElementById('closeReport').onclick = () => document.getElementById('reportPanel').classList.add('hidden');
document.getElementById('resetDay').onclick = () => { if(confirm('Zerar hoje?')) { entries=[]; localStorage.removeItem(`ctj_final_${currentDate}`); render(); } };
currentDateEl.onchange = (e) => { currentDate = e.target.value; load(); };

load();
</script>
</body>
</html>
