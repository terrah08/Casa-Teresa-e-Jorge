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
  
  .label-main { font-size: 10px; font-weight: 900; text-transform: uppercase; white-space: nowrap; line-height: 1.1; }
  .label-sub { font-size: 8px; font-weight: 500; opacity: 0.85; white-space: nowrap; margin-top: 2px; }

  @media (min-width: 640px) {
    .label-main { font-size: 12px; }
    .label-sub { font-size: 9px; }
  }

  /* Ajuste crucial para o gráfico não cortar no PDF */
  #chartWrapper { 
    width: 100%; 
    max-width: 320px; 
    margin: 0 auto; 
    aspect-ratio: 1/1;
    display: flex;
    align-items: center;
    justify-content: center;
  }
</style>
</head>
<body class="bg-gray-50 min-h-screen p-2 md:p-8 text-gray-800">

  <div class="max-w-6xl mx-auto">
    <header class="flex flex-col md:flex-row md:items-center md:justify-between gap-3 mb-6 bg-white p-4 rounded-xl shadow-sm">
      <div class="overflow-hidden">
        <h1 class="text-lg sm:text-xl md:text-2xl font-black truncate">Casa Teresa e Jorge 🟩🩷</h1>
        <div class="text-[10px] text-gray-400 font-bold uppercase tracking-widest">Portaria Profissional</div>
      </div>
      <div class="flex gap-2 items-center bg-gray-50 p-1.5 rounded-lg border border-gray-100 self-end md:self-auto">
        <input id="currentDate" type="date" class="border-0 bg-transparent text-xs font-bold focus:ring-0" />
        <button id="resetDay" class="text-red-400 px-2">🗑️</button>
      </div>
    </header>

    <main class="grid grid-cols-1 lg:grid-cols-3 gap-6">
      <section class="bg-white p-4 rounded-xl shadow col-span-1 border-t-4 border-blue-500">
        <h2 class="font-bold text-gray-700 mb-4 text-xs uppercase">📝 Entradas</h2>
        <div id="buttonsContainer" class="grid grid-cols-2 gap-2 mb-6"></div>
        <button id="generateReport" class="w-full py-3 bg-blue-600 rounded-xl text-white font-bold text-sm">📄 GERAR RELATÓRIO PDF</button>
      </section>

      <section class="bg-white p-4 rounded-xl shadow col-span-2 border-t-4 border-emerald-500">
        <div class="grid grid-cols-2 gap-4 mb-6 text-center sm:text-left">
          <div class="bg-emerald-50 p-3 rounded-xl border border-emerald-100">
            <div class="text-[10px] text-emerald-600 font-bold uppercase">Público</div>
            <div id="totalPeople" class="text-2xl font-black text-emerald-900">0</div>
          </div>
          <div class="bg-blue-50 p-3 rounded-xl border border-blue-100 relative">
            <div class="flex justify-between items-center">
              <div class="text-[10px] text-blue-600 font-bold uppercase">Caixa</div>
              <button id="toggleValue" class="text-lg"><span id="eyeIcon">👁️</span></button>
            </div>
            <div id="totalCollected" class="text-2xl font-black text-blue-900">R$ 0,00</div>
          </div>
        </div>
        <div class="overflow-x-auto rounded-lg">
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
        <div class="flex justify-between items-center mb-6 border-b pb-4">
          <h2 class="text-lg font-black uppercase italic">Fechamento de Caixa</h2>
          <div class="flex gap-2">
            <button id="downloadPdf" class="px-4 py-2 bg-emerald-600 text-white rounded-lg font-bold text-xs">BAIXAR PDF</button>
            <button id="closeReport" class="px-4 py-2 bg-gray-400 text-white rounded-lg text-xs">FECHAR</button>
          </div>
        </div>
        <div class="grid grid-cols-1 md:grid-cols-2 gap-8">
          <div id="reportSummary" class="space-y-2 text-sm bg-gray-50 p-4 rounded-xl border"></div>
          <div id="reportTotals" class="space-y-1"></div>
          <div class="col-span-1 md:col-span-2 py-4 flex flex-col items-center">
            <div id="chartWrapper">
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
  { id: "aniv", main: "Aniversário", sub
