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
  .btn { @apply rounded-xl text-white shadow-md active:scale-95 flex flex-col items-center justify-center text-center transition-all; }
  .hidden-value { filter: blur(6px); pointer-events: none; }
  .label-main { font-size: 10px; font-weight: 900; text-transform: uppercase; line-height: 1.1; }
  .label-sub { font-size: 8px; font-weight: 500; opacity: 0.85; margin-top: 2px; }
  #chartWrapper { width: 100%; max-width: 300px; margin: 0 auto; aspect-ratio: 1/1; }
</style>
</head>
<body class="bg-gray-100 min-h-screen p-2 md:p-8 text-gray-800 font-sans">

  <div class="max-w-6xl mx-auto">
    <header class="flex justify-between items-center mb-6 bg-white p-4 rounded-xl shadow-sm">
      <h1 class="text-lg font-black truncate">Casa Teresa e Jorge 🟩🩷</h1>
      <input id="currentDate" type="date" class="border-0 bg-gray-50 text-xs font-bold rounded-lg" />
    </header>

    <main class="grid grid-cols-1 lg:grid-cols-3 gap-4">
      <section class="bg-white p-4 rounded-xl shadow col-span-1">
        <h2 class="font-bold text-gray-400 text-[10px] uppercase mb-4">Lançamentos</h2>
        <div id="buttonsContainer" class="grid grid-cols-2 gap-2 mb-4"></div>
        <button id="generateReport" class="w-full py-4 bg-blue-600 rounded-xl text-white font-black text-sm shadow-lg">📄 GERAR RELATÓRIO PDF</button>
      </section>

      <section class="lg:col-span-2 space-y-4">
        <div class="grid grid-cols-2 gap-4">
          <div class="bg-white p-4 rounded-xl shadow border-l-4 border-emerald-500">
            <div class="text-[10px] font-bold text-gray-400 uppercase">Público</div>
            <div id="totalPeople" class="text-2xl font-black">0</div>
          </div>
          <div class="bg-white p-4 rounded-xl shadow border-l-4 border-blue-500 relative">
            <div class="flex justify-between items-center">
                <div class="text-[10px] font-bold text-gray-400 uppercase">Caixa</div>
                <button onclick="toggleBlur()" class="text-xs">👁️</button>
            </div>
            <div id="totalCollected" class="text-2xl font-black">R$ 0,00</div>
          </div>
        </div>

        <div class="bg-white rounded-xl shadow overflow-hidden">
          <table class="w-full text-xs text-left">
            <thead class="bg-gray-50 text-gray-400 uppercase font-bold">
              <tr>
                <th class="p-3">Hora</th>
                <th class="p-3">Tipo</th>
                <th class="p-3 text-right">Valor</th>
                <th class="p-3 text-center">✕</th>
              </tr>
            </thead>
            <tbody id="entriesBody" class="divide-y"></tbody>
          </table>
        </div>
      </section>
    </main>
  </div>

  <div id="reportModal" class="fixed inset-0 bg-black/60 hidden flex items-center justify-center p-4 z-50">
    <div class="bg-white w-full max-w-lg rounded-2xl p-6 overflow-y-auto max-h-[90vh]">
        <div class="flex justify-between items-center mb-6">
            <h2 class="font-black text-xl uppercase italic text-blue-900">Relatório de Fechamento</h2>
            <button onclick="closeModal()" class="text-gray-400 text-2xl">✕</button>
        </div>
        <div id="reportSummary" class="bg-gray-50 p-4 rounded-xl mb-6 text-sm space-y-2"></div>
        <div id="chartWrapper"><canvas id="reportChart"></canvas></div>
        <button id="downloadPdf" class="w-full py-4 bg-emerald-600 text-white font-black rounded-xl shadow-xl mt-6">💾 BAIXAR PDF OFICIAL</button>
    </div>
  </div>

<script>
const PRICE_TYPES = [
  { id: "20_din", main: "Dinheiro R$20", sub: "Individual", price: 20, people: 1, kind: "Dinheiro" },
  { id: "30_din", main: "Dinheiro R$30", sub: "Individual", price: 30, people: 1, kind: "Dinheiro" },
  { id: "50_din", main: "Dinheiro R$50", sub: "Dupla", price: 50, people: 2, kind: "Dinheiro" },
  { id: "20_car", main: "Cartão R$20", sub: "Individual", price: 20, people: 1, kind: "Cartão" },
  { id: "30_car", main: "Cartão R$30", sub: "Individual", price: 30, people: 1, kind: "Cartão" },
  { id: "50_car", main: "Cartão R$50", sub: "Dupla", price: 50, people: 2, kind: "Cartão" },
  { id: "30_pix", main: "Pix R$30", sub: "Individual", price: 30, people: 1, kind: "Pix" },
  { id: "50_pix", main: "Pix R$50", sub: "Dupla", price: 50, people: 2, kind: "Pix" },
  { id: "f100", main: "100 Pessoas", sub: "FREE", price: 0, people: 100, kind: "Gratuidade" },
  { id: "list", main: "Lista", sub: "Individual", price: 0, people: 1, kind: "Gratuidade" },
  { id: "aniv", main: "Aniversário", sub: "Isento", price: 0, people: 1, kind: "Gratuidade" },
  { id: "milt", main: "Militar", sub: "Isento", price: 0, people: 1, kind: "Gratuidade" }
];

let entries = [];
let isBlurred = true;
let chartInstance = null;
let currentDate = new Date().toISOString().slice(0,10);
document.getElementById('currentDate').value = currentDate;

// INICIAR BOTÕES
const container = document.getElementById('buttonsContainer');
PRICE_TYPES.forEach(p => {
    const b = document.createElement('button');
    b.className = `btn h-14 ${p.kind === 'Dinheiro' ? 'bg-green-600' : p.kind === 'Cartão' ? 'bg-orange-500' : p.kind === 'Pix' ? 'bg-cyan-600' : 'bg-gray-500'}`;
    b.innerHTML = `<span class="label-main">${p.main}</span><span class="label-sub">${p.sub}</span>`;
    b.onclick = () => addEntry(p);
    container.appendChild(b);
});

function addEntry(p) {
    entries.unshift({ id: Date.now(), time: new Date().toLocaleTimeString([], {hour:'2-digit', minute:'2-digit'}), label: `${p.main} (${p.sub})`, price: p.price, people: p.people, kind: p.kind });
    saveAndRender();
}

function deleteEntry(id) {
    if(confirm('Excluir?')) { entries = entries.filter(e => e.id !== id); saveAndRender(); }
}

function saveAndRender() {
    localStorage.setItem(`ctj_${currentDate}`, JSON.stringify(entries));
    const body = document.getElementById('entriesBody');
    body.innerHTML = entries.map(e => `
        <tr class="hover:bg-gray-50">
            <td class="p-3 text-gray-400 font-mono">${e.time}</td>
            <td class="p-3 font-bold text-gray-700">${e.label}</td>
            <td class="p-3 text-right font-black ${isBlurred ? 'hidden-value' : ''}">R$ ${e.price.toFixed(2)}</td>
            <td class="p-3 text-center"><button onclick="deleteEntry(${e.id})" class="text-red-300">✕</button></td>
        </tr>
    `).join('');
    
    const totals = entries.reduce((a, b) => ({ p: a.p + b.people, v: a.v + b.price }), { p: 0, v: 0 });
    document.getElementById('totalPeople').textContent = totals.p;
    document.getElementById('totalCollected').textContent = `R$ ${totals.v.toFixed(2)}`;
}

function toggleBlur() { isBlurred = !isBlurred; saveAndRender(); }

// RELATÓRIO E GRÁFICO
document.getElementById('generateReport').onclick = () => {
    document.getElementById('reportModal').classList.remove('hidden');
    const byKind = {};
    entries.forEach(e => {
        byKind[e.kind] = (byKind[e.kind] || 0) + e.price;
    });

    const totals = entries.reduce((a, b) => ({ p: a.p + b.people, v: a.v + b.price }), { p: 0, v: 0 });
    document.getElementById('reportSummary').innerHTML = `
        <div class="flex justify-between"><span>Total Público:</span><b>${totals.p} pessoas</b></div>
        <div class="flex justify-between text-lg border-t pt-2"><span>Total Caixa:</span><b class="text-emerald-600">R$ ${totals.v.toFixed(2)}</b></div>
    `;

    if(chartInstance) chartInstance.destroy();
    chartInstance = new Chart(document.getElementById('reportChart'), {
        type: 'pie',
        data: {
            labels: Object.keys(byKind),
            datasets: [{ data: Object.values(byKind), backgroundColor: ['#059669', '#d97706', '#0891b2', '#7c3aed'] }]
        },
        options: { 
            responsive: true, animation: false,
            plugins: { legend: { position: 'bottom' }, datalabels: { color: '#fff', font: {weight:'bold'} } }
        },
        plugins: [ChartDataLabels]
    });
};

function closeModal() { document.getElementById('reportModal').classList.add('hidden'); }

// DOWNLOAD PDF (O SEGREDO ESTÁ AQUI)
document.getElementById('downloadPdf').onclick = () => {
    const { jsPDF } = window.jspdf;
    const doc = new jsPDF();
    const totals = entries.reduce((a, b) => ({ p: a.p + b.people, v: a.v + b.price }), { p: 0, v: 0 });
    const chartImg = document.getElementById('reportChart').toDataURL("image/png");

    // Título e Dados
    doc.setFontSize(22); doc.text("CASA TERESA E JORGE", 105, 20, {align: "center"});
    doc.setFontSize(12); doc.text(`FECHAMENTO DE CAIXA - ${currentDate.split('-').reverse().join('/')}`, 105, 30, {align: "center"});
    
    doc.setDrawColor(200); doc.line(20, 35, 190, 35);

    doc.setFontSize(14);
    doc.text(`Público Total: ${totals.p} pessoas`, 20, 45);
    doc.setFont(undefined, 'bold');
    doc.text(`Valor Total Líquido: R$ ${totals.v.toFixed(2)}`, 20, 55);
    
    // Inserir Gráfico como Imagem (evita erros do html2canvas)
    doc.text("Distribuição por Meio de Pagamento:", 105, 75, {align: "center"});
    doc.addImage(chartImg, 'PNG', 55, 80, 100, 100);

    // Rodapé
    doc.setFontSize(10); doc.setFont(undefined, 'normal');
    doc.text(`Relatório gerado em: ${new Date().toLocaleString()}`, 105, 280, {align: "center"});

    doc.save(`Relatorio_Caixa_${currentDate}.pdf`);
};

// Carregar Dados Iniciais
const saved = localStorage.getItem(`ctj_${currentDate}`);
if(saved) { entries = JSON.parse(saved); saveAndRender(); }
document.getElementById('currentDate').onchange = (e) => {
    currentDate = e.target.value;
    const data = localStorage.getItem(`ctj_${currentDate}`);
    entries = data ? JSON.parse(data) : [];
    saveAndRender();
};
</script>
</body>
</html>
