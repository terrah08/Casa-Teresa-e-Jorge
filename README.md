<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Portaria - Casa Teresa e Jorge 🟩🩷</title>

<script src="https://cdn.tailwindcss.com"></script>
<script src="https://cdn.jsdelivr.net/npm/jspdf@2.5.1/dist/jspdf.umd.min.js"></script>

<style>
  .btn { @apply px-3 py-2 rounded-xl text-white shadow-md transition-all duration-200 active:scale-95; }
  .hidden-value { filter: blur(6px); pointer-events: none; user-select: none; }
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
        <input id="currentDate" type="date" class="border rounded px-2 py-1 text-sm font-bold" />
        <button id="resetDay" class="btn bg-red-500 hover:bg-red-600 p-2">🗑️</button>
      </div>
    </header>

    <main class="grid grid-cols-1 lg:grid-cols-3 gap-6">
      <section class="bg-white p-4 rounded-xl shadow col-span-1">
        <h2 class="font-bold text-gray-700 mb-4 uppercase text-xs tracking-widest">📝 Registrar</h2>
        <div id="buttonsContainer" class="grid grid-cols-2 gap-2 mb-6"></div>
        <button id="downloadPdf" class="w-full btn bg-blue-600 font-bold flex items-center justify-center gap-2">
          <span>💾 BAIXAR RELATÓRIO PDF</span>
        </button>
      </section>

      <section class="bg-white p-4 rounded-xl shadow col-span-2">
        <div class="grid grid-cols-2 gap-4 mb-6">
          <div class="bg-gray-50 p-3 rounded-xl border border-gray-100">
            <div class="text-[10px] text-gray-400 font-bold uppercase">Público</div>
            <div id="totalPeople" class="text-2xl font-black text-gray-800">0</div>
          </div>
          <div class="bg-gray-50 p-3 rounded-xl border border-gray-100 relative">
            <div class="flex justify-between items-center">
              <div class="text-[10px] text-gray-400 font-bold uppercase">Caixa</div>
              <button id="toggleValue" class="text-lg"> <span id="eyeIcon">👁️</span> </button>
            </div>
            <div id="totalCollected" class="text-2xl font-black text-emerald-600">R$ 0,00</div>
          </div>
        </div>

        <div id="tableWrapper" class="overflow-x-auto rounded-lg border border-gray-100">
          <table class="min-w-full text-sm">
            <thead class="bg-gray-50 text-gray-500 uppercase text-[10px]">
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
  { id: "free", label: "Lista (Individual)", price: 0, people: 1, kind: "Gratuidade" },
  { id: "Aniv", label: "Aniversário", price: 0, people: 1, kind: "Gratuidade" },
  { id: "militar", label: "Militar", price: 0, people: 1, kind: "Gratuidade" }
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
  const data = localStorage.getItem(`ctj_p_${currentDate}`);
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
  localStorage.setItem(`ctj_p_${currentDate}`, JSON.stringify(entries));
  render();
}

window.deleteEntry = (id) => {
  if(confirm('Excluir?')) {
    entries = entries.filter(e => e.id !== id);
    localStorage.setItem(`ctj_p_${currentDate}`, JSON.stringify(entries));
    render();
  }
};

// DOWNLOAD PDF - GERAÇÃO DIRETA SEM CORTES
document.getElementById('downloadPdf').onclick = () => {
  if (entries.length === 0) return alert("Nenhum dado para exportar.");

  const { jsPDF } = window.jspdf;
  const doc = new jsPDF();
  const totals = entries.reduce((a, b) => ({ p: a.p + b.people, v: a.v + b.price }), { p: 0, v: 0 });
  
  // Cabeçalho
  doc.setFontSize(20);
  doc.setTextColor(40);
  doc.text("CASA TERESA E JORGE", 105, 20, { align: "center" });
  
  doc.setFontSize(12);
  doc.text(`Relatório de Portaria - ${currentDate.split('-').reverse().join('/')}`, 105, 28, { align: "center" });
  
  doc.setDrawColor(200);
  doc.line(10, 35, 200, 35);

  // Resumo
  doc.setFontSize(14);
  doc.setFont(undefined, 'bold');
  doc.text("RESUMO DO DIA", 10, 45);
  
  doc.setFont(undefined, 'normal');
  doc.setFontSize(11);
  doc.text(`Público Total: ${totals.p} pessoas`, 10, 52);
  doc.text(`Arrecadação Bruta: R$ ${totals.v.toFixed(2)}`, 10, 59);
  doc.text(`Ticket Médio: R$ ${totals.p ? (totals.v / totals.p).toFixed(2) : '0.00'}`, 10, 66);

  // Totais por Tipo
  const byKind = {};
  entries.forEach(e => {
    byKind[e.kind] = (byKind[e.kind] || 0) + e.price;
  });

  doc.setFont(undefined, 'bold');
  doc.text("TOTAIS POR CATEGORIA:", 10, 78);
  doc.setFont(undefined, 'normal');
  
  let currentY = 85;
  Object.entries(byKind).forEach(([kind, val]) => {
    doc.text(`${kind}: R$ ${val.toFixed(2)}`, 15, currentY);
    currentY += 7;
  });

  // Lista de Entradas
  doc.setDrawColor(230);
  doc.line(10, currentY + 5, 200, currentY + 5);
  currentY += 15;
  
  doc.setFont(undefined, 'bold');
  doc.text("DETALHAMENTO DE ENTRADAS", 10, currentY);
  doc.setFontSize(9);
  doc.setFont(undefined, 'normal');
  currentY += 8;

  // Cabeçalho Tabela
  doc.setFillColor(240);
  doc.rect(10, currentY - 5, 190, 7, 'F');
  doc.text("HORA", 12, currentY);
  doc.text("DESCRIÇÃO", 40, currentY);
  doc.text("VALOR", 180, currentY);
  currentY += 8;

  // Itens da Tabela (com paginação simples)
  entries.forEach((e) => {
    if (currentY > 280) {
      doc.addPage();
      currentY = 20;
    }
    const time = new Date(e.ts).toLocaleTimeString([], {hour:'2-digit', minute:'2-digit'});
    doc.text(time, 12, currentY);
    doc.text(e.type.substring(0, 45), 40, currentY);
    doc.text(`R$ ${e.price.toFixed(2)}`, 180, currentY);
    currentY += 6;
  });

  doc.save(`relatorio_${currentDate}.pdf`);
};

document.getElementById('resetDay').onclick = () => { if(confirm('Zerar hoje?')) { entries=[]; localStorage.removeItem(`ctj_p_${currentDate}`); render(); } };
currentDateEl.onchange = (e) => { currentDate = e.target.value; load(); };

load();
</script>
</body>
</html>
