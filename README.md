<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Portaria - Casa Teresa e Jorge 🟩🩷</title>

<script src="https://cdn.tailwindcss.com"></script>
<script src="https://cdn.jsdelivr.net/npm/jspdf@2.5.1/dist/jspdf.umd.min.js"></script>

<style>
  .btn { @apply px-3 py-2 rounded-xl text-white shadow-md transition-all duration-200 active:scale-95 flex items-center justify-center gap-2; }
  .hidden-value { filter: blur(6px); pointer-events: none; user-select: none; }
  .table-resumo td { @apply py-1 border-b border-gray-100; }
</style>
</head>
<body class="bg-gray-50 min-h-screen p-2 md:p-8 text-gray-800">

  <div class="max-w-6xl mx-auto">
    <header class="flex flex-col md:flex-row md:items-center md:justify-between gap-4 mb-6 bg-white p-4 rounded-xl shadow-sm">
      <div>
        <h1 class="text-xl md:text-3xl font-extrabold text-gray-800">Casa Teresa e Jorge 🟩🩷</h1>
        <div class="text-xs text-gray-500 font-bold uppercase tracking-wider">Gestão de Portaria Profissional</div>
      </div>
      <div class="flex gap-2 items-center bg-gray-100 p-2 rounded-lg border border-gray-200">
        <input id="currentDate" type="date" class="border rounded px-2 py-1 text-sm font-bold bg-white" />
        <button id="resetDay" class="btn bg-red-500 hover:bg-red-600 p-2" title="Zerar Tudo">🗑️</button>
      </div>
    </header>

    <main class="grid grid-cols-1 lg:grid-cols-3 gap-6">
      
      <section class="bg-white p-4 rounded-xl shadow col-span-1 border-t-4 border-blue-500">
        <h2 class="font-bold text-gray-400 text-[10px] uppercase mb-4 tracking-widest">📝 Registrar Entrada</h2>
        <div id="buttonsContainer" class="grid grid-cols-2 gap-2 mb-6"></div>
        <button id="btnOpenReport" class="w-full btn bg-blue-600 font-bold py-3">
          <span>📄 GERAR RELATÓRIO PDF</span>
        </button>
      </section>

      <section class="bg-white p-4 rounded-xl shadow col-span-2 border-t-4 border-emerald-500">
        <div class="grid grid-cols-2 gap-4 mb-6">
          <div class="bg-emerald-50 p-3 rounded-xl border border-emerald-100 text-center sm:text-left">
            <div class="text-[10px] text-emerald-600 font-bold uppercase">Público Atual</div>
            <div id="totalPeople" class="text-2xl font-black text-emerald-900">0</div>
          </div>
          <div class="bg-blue-50 p-3 rounded-xl border border-blue-100 relative text-center sm:text-left">
            <div class="flex justify-between items-center">
              <div class="text-[10px] text-blue-600 font-bold uppercase">Caixa Total</div>
              <button id="toggleValue" class="text-lg"> <span id="eyeIcon">👁️</span> </button>
            </div>
            <div id="totalCollected" class="text-2xl font-black text-blue-900">R$ 0,00</div>
          </div>
        </div>

        <div class="overflow-x-auto rounded-lg border border-gray-50">
          <table class="min-w-full text-sm">
            <thead class="bg-gray-50 text-gray-400 uppercase text-[10px] font-bold">
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

      <section id="reportPanel" class="bg-white p-6 rounded-xl shadow-2xl col-span-3 hidden border-b-8 border-emerald-600 animate-in fade-in duration-300">
        <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center mb-6 border-b pb-4 gap-4">
          <div>
            <h2 class="text-xl font-black text-gray-800 uppercase italic">Conferência de Fechamento</h2>
            <p id="reportDateDisplay" class="text-xs text-gray-400 font-bold uppercase"></p>
          </div>
          <div class="flex gap-2 w-full sm:w-auto">
            <button id="downloadPdf" class="btn bg-emerald-600 font-bold flex-1 sm:flex-none px-6">💾 BAIXAR PDF</button>
            <button id="closeReport" class="btn bg-gray-400 text-sm px-4">FECHAR</button>
          </div>
        </div>

        <div class="grid grid-cols-1 md:grid-cols-2 gap-8">
          <div class="bg-gray-50 p-6 rounded-2xl border border-gray-100">
             <h3 class="text-emerald-700 font-black text-sm uppercase mb-4 border-b pb-2">Resumo Geral</h3>
             <div id="reportSummary" class="space-y-3 text-gray-700"></div>
          </div>
          <div class="bg-white p-6 rounded-2xl border border-gray-100">
             <h3 class="text-gray-400 font-black text-sm uppercase mb-4 border-b pb-2">Por Meio de Pagamento</h3>
             <div id="reportTotals" class="space-y-1"></div>
          </div>
        </div>
      </section>

    </main>
  </div>

<script>
const PRICE_TYPES = [
  { id: "20_din", label: "Dinheiro R$20 - Indiv.", price: 20, people: 1, kind: "Dinheiro" },
  { id: "30_din", label: "Dinheiro R$30 - Indiv.", price: 30, people: 1, kind: "Dinheiro" },
  { id: "50_din", label: "Dinheiro R$50 - Dupla", price: 50, people: 2, kind: "Dinheiro" },
  { id: "20_cart", label: "Cartão R$20 - Indiv.", price: 20, people: 1, kind: "Cartão" },
  { id: "30_cart", label: "Cartão R$30 - Indiv.", price: 30, people: 1, kind: "Cartão" },
  { id: "50_cart", label: "Cartão R$50 - Dupla", price: 50, people: 2, kind: "Cartão" },
  { id: "20_pix", label: "Pix R$20 - Indiv.", price: 20, people: 1, kind: "Pix" },
  { id: "30_pix", label: "Pix R$30 - Indiv.", price: 30, people: 1, kind: "Pix" },
  { id: "50_pix", label: "Pix R$50 - Dupla", price: 50, people: 2, kind: "Pix" },
  { id: "free100", label: "100 Pessoas FREE", price: 0, people: 100, kind: "Gratuidade" },
  { id: "free", label: "Lista Individual", price: 0, people: 1, kind: "Gratuidade" },
  { id: "Aniv", label: "Aniversário", price: 0, people: 1, kind: "Gratuidade" },
  { id: "milt", label: "Militar", price: 0, people: 1, kind: "Gratuidade" }
];

let currentDate = new Date().toISOString().slice(0,10);
let entries = [];
let isValueVisible = true;

const container = document.getElementById('buttonsContainer');
PRICE_TYPES.forEach(p => {
  const b = document.createElement('button');
  b.className = `p-2 rounded-lg text-white text-[10px] font-bold uppercase ${p.kind === 'Dinheiro' ? 'bg-green-600' : p.kind === 'Cartão' ? 'bg-amber-500' : p.kind === 'Pix' ? 'bg-cyan-600' : 'bg-gray-400'}`;
  b.textContent = p.label;
  b.onclick = () => addEntry(p.id);
  container.appendChild(b);
});

const currentDateEl = document.getElementById('currentDate');
currentDateEl.value = currentDate;

document.getElementById('toggleValue').onclick = () => {
  isValueVisible = !isValueVisible;
  document.getElementById('eyeIcon').textContent = isValueVisible ? '👁️' : '🙈';
  render();
};

function load() {
  const data = localStorage.getItem(`ctj_v3_${currentDate}`);
  entries = data ? JSON.parse(data) : [];
  render();
}

function render() {
  const body = document.getElementById('entriesBody');
  const blur = isValueVisible ? '' : 'hidden-value';
  body.innerHTML = entries.map(e => `
    <tr>
      <td class="p-2 text-gray-400 font-mono">${new Date(e.ts).toLocaleTimeString([], {hour:'2-digit', minute:'2-digit'})}</td>
      <td class="p-2 font-bold text-gray-700 uppercase text-[10px]">${e.type}</td>
      <td class="p-2 text-right font-mono ${blur}">R$ ${e.price.toFixed(2)}</td>
      <td class="p-2 text-center"><button onclick="deleteEntry(${e.id})" class="text-red-300">✕</button></td>
    </tr>
  `).join('');

  const totals = entries.reduce((a, b) => ({ p: a.p + b.people, v: a.v + b.price }), { p: 0, v: 0 });
  document.getElementById('totalPeople').textContent = totals.p;
  document.getElementById('totalCollected').textContent = `R$ ${totals.v.toFixed(2)}`;
  if(!isValueVisible) document.getElementById('totalCollected').classList.add('hidden-value');
  else document.getElementById('totalCollected').classList.remove('hidden-value');
}

function addEntry(id) {
  const t = PRICE_TYPES.find(x => x.id === id);
  entries.unshift({ id: Date.now(), ts: new Date().toISOString(), type: t.label, price: t.price, people: t.people, kind: t.kind });
  localStorage.setItem(`ctj_v3_${currentDate}`, JSON.stringify(entries));
  render();
}

window.deleteEntry = (id) => {
  if(confirm('Excluir este registro?')) {
    entries = entries.filter(e => e.id !== id);
    localStorage.setItem(`ctj_v3_${currentDate}`, JSON.stringify(entries));
    render();
  }
};

// BOTÃO GERAR (ABRE O PAINEL NA PÁGINA)
document.getElementById('btnOpenReport').onclick = () => {
  if(entries.length === 0) return alert("Nenhum dado registrado hoje.");
  
  const panel = document.getElementById('reportPanel');
  panel.classList.remove('hidden');
  document.getElementById('reportDateDisplay').textContent = `Portaria de ${currentDate.split('-').reverse().join('/')}`;

  const totals = entries.reduce((a, b) => ({ p: a.p + b.people, v: a.v + b.price }), { p: 0, v: 0 });
  const byKind = {};
  entries.forEach(e => {
    byKind[e.kind] = (byKind[e.kind] || 0) + e.price;
  });

  document.getElementById('reportSummary').innerHTML = `
    <div class="flex justify-between"><span>Público Total:</span><b class="text-lg">${totals.p}p</b></div>
    <div class="flex justify-between border-t pt-2 mt-2"><span>Arrecadação:</span><b class="text-xl text-emerald-600 font-black">R$ ${totals.v.toFixed(2)}</b></div>
    <div class="flex justify-between text-xs text-gray-400 italic"><span>Ticket Médio:</span><span>R$ ${totals.p ? (totals.v/totals.p).toFixed(2) : '0.00'}</span></div>
  `;

  document.getElementById('reportTotals').innerHTML = Object.entries(byKind).map(([k, v]) => `
    <div class="flex justify-between border-b py-2 text-sm uppercase">
      <span class="font-bold text-gray-500">${k}:</span>
      <span class="font-black text-gray-700">R$ ${v.toFixed(2)}</span>
    </div>
  `).join('');

  panel.scrollIntoView({ behavior: 'smooth' });
};

// BOTÃO BAIXAR (GERA O ARQUIVO PDF REAL)
document.getElementById('downloadPdf').onclick = () => {
  const { jsPDF } = window.jspdf;
  const doc = new jsPDF();
  const totals = entries.reduce((a, b) => ({ p: a.p + b.people, v: a.v + b.price }), { p: 0, v: 0 });

  doc.setFontSize(22);
  doc.text("CASA TERESA E JORGE", 105, 20, { align: "center" });
  doc.setFontSize(12);
  doc.text(`FECHAMENTO DE CAIXA - ${currentDate.split('-').reverse().join('/')}`, 105, 30, { align: "center" });
  
  doc.line(15, 35, 195, 35);

  doc.setFontSize(14);
  doc.setFont(undefined, 'bold');
  doc.text("DADOS GERAIS", 15, 45);
  doc.setFontSize(11);
  doc.setFont(undefined, 'normal');
  doc.text(`Público Total: ${totals.p} pessoas`, 15, 52);
  doc.text(`Valor Total Bruto: R$ ${totals.v.toFixed(2)}`, 15, 59);

  const byKind = {};
  entries.forEach(e => byKind[e.kind] = (byKind[e.kind] || 0) + e.price);

  doc.setFont(undefined, 'bold');
  doc.text("POR MEIO DE PAGAMENTO:", 15, 75);
  doc.setFont(undefined, 'normal');
  let currentY = 82;
  Object.entries(byKind).forEach(([k, v]) => {
    doc.text(`${k}: R$ ${v.toFixed(2)}`, 20, currentY);
    currentY += 7;
  });

  doc.line(15, currentY + 5, 195, currentY + 5);
  currentY += 15;

  doc.setFont(undefined, 'bold');
  doc.text("LISTA DE ENTRADAS", 15, currentY);
  doc.setFontSize(8);
  doc.setFont(undefined, 'normal');
  currentY += 8;

  entries.forEach(e => {
    if(currentY > 280) { doc.addPage(); currentY = 20; }
    const time = new Date(e.ts).toLocaleTimeString([], {hour:'2-digit', minute:'2-digit'});
    doc.text(time, 15, currentY);
    doc.text(e.type.substring(0, 50), 40, currentY);
    doc.text(`R$ ${e.price.toFixed(2)}`, 170, currentY);
    currentY += 6;
  });

  doc.save(`Fechamento_${currentDate}.pdf`);
};

document.getElementById('closeReport').onclick = () => document.getElementById('reportPanel').classList.add('hidden');
document.getElementById('resetDay').onclick = () => { if(confirm('Limpar todos os dados de hoje?')) { entries=[]; localStorage.removeItem(`ctj_v3_${currentDate}`); render(); } };
currentDateEl.onchange = (e) => { currentDate = e.target.value; load(); };

load();
</script>
</body>
</html>
