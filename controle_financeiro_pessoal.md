```html
<!DOCTYPE html>
<html lang="pt-BR" class="dark">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Gestão Financeira Pessoal & Compartilhada</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- FontAwesome Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <script>
        tailwind.config = {
            darkMode: 'class',
            theme: {
                extend: {
                    colors: {
                        brand: {
                            50: '#f0f7ff',
                            100: '#e0effe',
                            500: '#3b82f6',
                            600: '#2563eb',
                            700: '#1d4ed8',
                            800: '#1e40af',
                            900: '#1e3a8a',
                        }
                    }
                }
            }
        }
    </script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap');
        body { font-family: 'Inter', sans-serif; }
        ::-webkit-scrollbar { width: 6px; height: 6px; }
        ::-webkit-scrollbar-track { background: transparent; }
        ::-webkit-scrollbar-thumb { background: #94a3b8; border-radius: 4px; }
        .dark ::-webkit-scrollbar-thumb { background: #334155; }
    </style>

    <!-- Firebase SDKs -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { 
            getAuth, 
            signInAnonymously, 
            signInWithCustomToken, 
            onAuthStateChanged,
            signOut,
            createUserWithEmailAndPassword,
            signInWithEmailAndPassword
        } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { 
            getFirestore, 
            doc, 
            getDoc, 
            setDoc, 
            onSnapshot 
        } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        const firebaseConfig = typeof __firebase_config !== 'undefined' 
            ? JSON.parse(__firebase_config) 
            : {
                apiKey: "demo-key",
                authDomain: "demo-app.firebaseapp.com",
                projectId: "demo-app",
                storageBucket: "demo-app.appspot.com",
                messagingSenderId: "000000000000",
                appId: "1:000000000000:web:000000000000"
            };

        const appId = typeof __app_id !== 'undefined' ? __app_id : 'finances-app-default';

        const app = initializeApp(firebaseConfig);
        const auth = getAuth(app);
        const db = getFirestore(app);

        window.fbAuth = auth;
        window.fbDb = db;
        window.fbAppId = appId;
        window.fbMethods = {
            signInAnonymously,
            signInWithCustomToken,
            onAuthStateChanged,
            signOut,
            createUserWithEmailAndPassword,
            signInWithEmailAndPassword,
            doc,
            getDoc,
            setDoc,
            onSnapshot
        };
    </script>
</head>
<body class="bg-slate-100 dark:bg-slate-900 text-slate-800 dark:text-slate-100 min-h-screen flex flex-col transition-colors duration-200">

    <!-- Header -->
    <header class="bg-white dark:bg-slate-800 border-b border-slate-200 dark:border-slate-700 sticky top-0 z-30 shadow-sm">
        <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 h-16 flex items-center justify-between">
            <div class="flex items-center space-x-3">
                <div class="bg-brand-600 text-white p-2.5 rounded-xl shadow-md">
                    <i class="fa-solid fa-wallet text-xl"></i>
                </div>
                <div>
                    <h1 class="font-bold text-lg leading-tight text-slate-900 dark:text-white">Gestão Financeira</h1>
                    <p class="text-xs text-slate-500 dark:text-slate-400">Nuvem & Compartilhada</p>
                </div>
            </div>

            <div class="flex items-center space-x-2 sm:space-x-3">
                <div id="user-status-container" class="flex items-center space-x-2">
                    <button onclick="openAuthModal()" id="btn-auth-trigger" class="px-3 py-1.5 text-xs font-semibold rounded-lg bg-brand-50 dark:bg-brand-950/60 text-brand-600 dark:text-brand-400 hover:bg-brand-100 border border-brand-200 dark:border-brand-800 transition flex items-center space-x-1.5">
                        <i class="fa-solid fa-user-circle text-sm"></i>
                        <span id="user-display-name">Entrar / Cadastrar</span>
                    </button>
                </div>

                <button id="themeToggle" class="p-2 rounded-lg text-slate-500 hover:text-slate-700 dark:text-slate-400 dark:hover:text-white bg-slate-100 dark:bg-slate-700/50 hover:bg-slate-200 dark:hover:bg-slate-700 transition">
                    <i class="fa-solid fa-moon dark:hidden"></i>
                    <i class="fa-solid fa-sun hidden dark:block text-amber-400"></i>
                </button>

                <button onclick="confirmResetData()" title="Restaurar dados originais da planilha" class="px-2.5 py-1.5 text-xs font-medium text-rose-600 dark:text-rose-400 bg-rose-50 dark:bg-rose-950/40 hover:bg-rose-100 rounded-lg transition border border-rose-200 dark:border-rose-800/50 flex items-center space-x-1">
                    <i class="fa-solid fa-rotate-left"></i>
                    <span class="hidden md:inline">Restaurar</span>
                </button>
            </div>
        </div>
    </header>

    <!-- Navigation Tabs -->
    <div class="bg-white dark:bg-slate-800/80 border-b border-slate-200 dark:border-slate-700/80 backdrop-blur">
        <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
            <nav class="flex space-x-2 sm:space-x-6 overflow-x-auto py-2" aria-label="Tabs">
                <button onclick="switchTab('dashboard')" id="tab-dashboard" class="tab-btn px-4 py-2.5 rounded-lg text-sm font-medium transition flex items-center space-x-2 whitespace-nowrap text-brand-600 dark:text-brand-400 bg-brand-50 dark:bg-brand-950/50 border border-brand-200 dark:border-brand-800/50">
                    <i class="fa-solid fa-chart-pie"></i>
                    <span>Dashboard & Visão Mensal</span>
                </button>
                <button onclick="switchTab('dividas')" id="tab-dividas" class="tab-btn px-4 py-2.5 rounded-lg text-sm font-medium transition flex items-center space-x-2 whitespace-nowrap text-slate-600 hover:text-slate-900 dark:text-slate-400 dark:hover:text-white">
                    <i class="fa-solid fa-file-invoice-dollar"></i>
                    <span>Dívidas & Crédito</span>
                </button>
                <button onclick="switchTab('renda-extra')" id="tab-renda-extra" class="tab-btn px-4 py-2.5 rounded-lg text-sm font-medium transition flex items-center space-x-2 whitespace-nowrap text-slate-600 hover:text-slate-900 dark:text-slate-400 dark:hover:text-white">
                    <i class="fa-solid fa-arrow-up-right-dots"></i>
                    <span>Renda Extra</span>
                </button>
                <button onclick="switchTab('gastos')" id="tab-gastos" class="tab-btn px-4 py-2.5 rounded-lg text-sm font-medium transition flex items-center space-x-2 whitespace-nowrap text-slate-600 hover:text-slate-900 dark:text-slate-400 dark:hover:text-white">
                    <i class="fa-solid fa-basket-shopping"></i>
                    <span>Pequenos Gastos</span>
                </button>
                <button onclick="switchTab('config')" id="tab-config" class="tab-btn px-4 py-2.5 rounded-lg text-sm font-medium transition flex items-center space-x-2 whitespace-nowrap text-slate-600 hover:text-slate-900 dark:text-slate-400 dark:hover:text-white">
                    <i class="fa-solid fa-sliders"></i>
                    <span>Gerenciar Meses & Renda Fixa</span>
                </button>
            </nav>
        </div>
    </div>

    <!-- Sync Indicator Banner -->
    <div id="sync-banner" class="bg-amber-500/10 border-b border-amber-500/20 py-1.5 px-4 text-center text-xs text-amber-700 dark:text-amber-400 font-medium flex items-center justify-center space-x-2">
        <i class="fa-solid fa-cloud text-amber-500 animate-pulse"></i>
        <span id="sync-status-text">Modo Local (Inicie sessão para sincronizar na nuvem e compartilhar)</span>
    </div>

    <!-- Main Content -->
    <main class="flex-grow max-w-7xl w-full mx-auto px-4 sm:px-6 lg:px-8 py-6">

        <!-- TAB 1: DASHBOARD & VISÃO MENSAL -->
        <div id="view-dashboard" class="tab-content space-y-6">
            
            <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-5 gap-4">
                <div class="bg-white dark:bg-slate-800 p-5 rounded-2xl border border-slate-200 dark:border-slate-700 shadow-sm">
                    <div class="flex items-center justify-between">
                        <span class="text-xs font-semibold uppercase tracking-wider text-slate-500 dark:text-slate-400">Total Dívidas Restantes</span>
                        <div class="p-2 rounded-lg bg-rose-100 dark:bg-rose-950/50 text-rose-600 dark:text-rose-400">
                            <i class="fa-solid fa-hand-holding-dollar"></i>
                        </div>
                    </div>
                    <div class="mt-3">
                        <div id="kpi-total-dividas-restantes" class="text-2xl font-bold text-rose-600 dark:text-rose-400">R$ 0,00</div>
                        <p class="text-xs text-slate-500 dark:text-slate-400 mt-1" id="kpi-total-dividas-original">Original: R$ 0,00</p>
                    </div>
                </div>

                <div class="bg-white dark:bg-slate-800 p-5 rounded-2xl border border-slate-200 dark:border-slate-700 shadow-sm">
                    <div class="flex items-center justify-between">
                        <span class="text-xs font-semibold uppercase tracking-wider text-slate-500 dark:text-slate-400">Total Amortizado</span>
                        <div class="p-2 rounded-lg bg-emerald-100 dark:bg-emerald-950/50 text-emerald-600 dark:text-emerald-400">
                            <i class="fa-solid fa-circle-check"></i>
                        </div>
                    </div>
                    <div class="mt-3">
                        <div id="kpi-total-pago" class="text-2xl font-bold text-emerald-600 dark:text-emerald-400">R$ 0,00</div>
                        <p class="text-xs text-slate-500 dark:text-slate-400 mt-1">Dívidas quitadas/parciais</p>
                    </div>
                </div>

                <div class="bg-white dark:bg-slate-800 p-5 rounded-2xl border border-slate-200 dark:border-slate-700 shadow-sm">
                    <div class="flex items-center justify-between">
                        <span class="text-xs font-semibold uppercase tracking-wider text-slate-500 dark:text-slate-400">Renda Extra Registrada</span>
                        <div class="p-2 rounded-lg bg-teal-100 dark:bg-teal-950/50 text-teal-600 dark:text-teal-400">
                            <i class="fa-solid fa-arrow-up-right-dots"></i>
                        </div>
                    </div>
                    <div class="mt-3">
                        <div id="kpi-renda-extra" class="text-2xl font-bold text-teal-600 dark:text-teal-400">R$ 0,00</div>
                        <p class="text-xs text-slate-500 dark:text-slate-400 mt-1">Ganhos extras no período</p>
                    </div>
                </div>

                <div class="bg-white dark:bg-slate-800 p-5 rounded-2xl border border-slate-200 dark:border-slate-700 shadow-sm">
                    <div class="flex items-center justify-between">
                        <span class="text-xs font-semibold uppercase tracking-wider text-slate-500 dark:text-slate-400">Pequenos Gastos</span>
                        <div class="p-2 rounded-lg bg-amber-100 dark:bg-amber-950/50 text-amber-600 dark:text-amber-400">
                            <i class="fa-solid fa-basket-shopping"></i>
                        </div>
                    </div>
                    <div class="mt-3">
                        <div id="kpi-pequenos-gastos" class="text-2xl font-bold text-amber-600 dark:text-amber-400">R$ 0,00</div>
                        <p class="text-xs text-slate-500 dark:text-slate-400 mt-1">Despesas do dia a dia</p>
                    </div>
                </div>

                <div class="bg-white dark:bg-slate-800 p-5 rounded-2xl border border-slate-200 dark:border-slate-700 shadow-sm border-l-4 border-l-brand-500">
                    <div class="flex items-center justify-between">
                        <span class="text-xs font-semibold uppercase tracking-wider text-slate-500 dark:text-slate-400">Saldo Líquido Período</span>
                        <div class="p-2 rounded-lg bg-brand-100 dark:bg-brand-950/50 text-brand-600 dark:text-brand-400">
                            <i class="fa-solid fa-piggy-bank"></i>
                        </div>
                    </div>
                    <div class="mt-3">
                        <div id="kpi-saldo-liquido" class="text-2xl font-bold text-brand-600 dark:text-brand-400">R$ 0,00</div>
                        <p class="text-xs text-slate-500 dark:text-slate-400 mt-1">(Renda Fixa + Extra) - (Dívidas + Gastos)</p>
                    </div>
                </div>
            </div>

            <!-- Visão Mensal Grid -->
            <div class="bg-white dark:bg-slate-800 rounded-2xl p-6 border border-slate-200 dark:border-slate-700 shadow-sm">
                <div class="flex flex-col sm:flex-row sm:items-center justify-between gap-4 mb-6">
                    <div>
                        <h2 class="text-lg font-bold text-slate-900 dark:text-white flex items-center space-x-2">
                            <i class="fa-regular fa-calendar-days text-brand-500"></i>
                            <span>Fluxo de Caixa Mensal</span>
                        </h2>
                        <p class="text-xs text-slate-500 dark:text-slate-400">Resumo mês a mês comparando dívidas programadas, renda fixa, extras e pequenos gastos.</p>
                    </div>
                    
                    <div class="flex items-center space-x-2">
                        <button onclick="switchTab('config')" class="px-3 py-1.5 bg-brand-50 dark:bg-brand-950/50 border border-brand-200 dark:border-brand-800/50 text-brand-600 dark:text-brand-400 text-xs font-semibold rounded-lg hover:bg-brand-100 transition flex items-center space-x-1">
                            <i class="fa-solid fa-plus text-xs"></i>
                            <span>Adicionar Mês</span>
                        </button>
                        <select id="filter-year" onchange="renderMonthlyCards()" class="bg-slate-100 dark:bg-slate-700 text-slate-800 dark:text-slate-100 text-xs font-semibold rounded-lg px-3 py-1.5 border border-slate-200 dark:border-slate-600 focus:outline-none focus:ring-2 focus:ring-brand-500">
                            <option value="ALL">Todos os Anos</option>
                        </select>
                    </div>
                </div>

                <!-- Monthly Cards Grid -->
                <div id="monthly-cards-grid" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-4">
                </div>
            </div>
        </div>

        <!-- TAB 2: DÍVIDAS E CRÉDITO -->
        <div id="view-dividas" class="tab-content hidden space-y-6">
            <div class="bg-slate-900 text-white rounded-2xl p-6 shadow-lg border border-slate-800 flex flex-col md:flex-row items-center justify-between gap-6">
                <div>
                    <span class="text-xs uppercase tracking-wider text-slate-400 font-medium">Situação Geral de Dívidas</span>
                    <h2 class="text-3xl font-extrabold text-rose-500 mt-1" id="total-dividas-header">TOTAL RESTANTE: R$ 0,00</h2>
                    <p class="text-xs text-slate-400 mt-1">Soma de todos os contratos e parcelamentos em aberto.</p>
                </div>
                <div class="flex items-center space-x-6 text-center">
                    <div>
                        <span class="text-xs text-slate-400 block">Total Original</span>
                        <span class="text-lg font-bold text-slate-200" id="total-dividas-original-header">R$ 0,00</span>
                    </div>
                    <div class="h-8 w-px bg-slate-700"></div>
                    <div>
                        <span class="text-xs text-slate-400 block">Amortizado</span>
                        <span class="text-lg font-bold text-emerald-400" id="total-dividas-pago-header">R$ 0,00</span>
                    </div>
                </div>
            </div>

            <div id="contracts-container" class="space-y-6">
            </div>
        </div>

        <!-- TAB 3: RENDA EXTRA -->
        <div id="view-renda-extra" class="tab-content hidden space-y-6">
            <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
                <!-- Formulário de Renda Extra -->
                <div class="bg-white dark:bg-slate-800 p-6 rounded-2xl border border-slate-200 dark:border-slate-700 shadow-sm h-fit">
                    <h3 class="text-md font-bold text-slate-900 dark:text-white mb-4 flex items-center space-x-2">
                        <i class="fa-solid fa-plus-circle text-teal-500"></i>
                        <span id="form-renda-title">Registrar Renda Extra</span>
                    </h3>

                    <form id="form-renda-extra" onsubmit="handleSaveRendaExtra(event)" class="space-y-4">
                        <input type="hidden" id="edit-renda-id" value="">

                        <div>
                            <label class="block text-xs font-medium text-slate-700 dark:text-slate-300 mb-1">Descrição do Ganho Extra</label>
                            <input type="text" id="input-renda-desc" required placeholder="Ex: Venda de Doces, Freelance, Bico" class="w-full text-xs p-2.5 rounded-lg bg-slate-50 dark:bg-slate-700 border border-slate-200 dark:border-slate-600 text-slate-900 dark:text-white focus:outline-none focus:ring-2 focus:ring-teal-500">
                        </div>

                        <div class="grid grid-cols-2 gap-3">
                            <div>
                                <label class="block text-xs font-medium text-slate-700 dark:text-slate-300 mb-1">Valor (R$)</label>
                                <input type="number" step="0.01" id="input-renda-valor" required placeholder="0,00" class="w-full text-xs p-2.5 rounded-lg bg-slate-50 dark:bg-slate-700 border border-slate-200 dark:border-slate-600 text-slate-900 dark:text-white focus:outline-none focus:ring-2 focus:ring-teal-500">
                            </div>
                            <div>
                                <label class="block text-xs font-medium text-slate-700 dark:text-slate-300 mb-1">Mês de Referência</label>
                                <select id="input-renda-mes" required class="w-full text-xs p-2.5 rounded-lg bg-slate-50 dark:bg-slate-700 border border-slate-200 dark:border-slate-600 text-slate-900 dark:text-white focus:outline-none focus:ring-2 focus:ring-teal-500">
                                </select>
                            </div>
                        </div>

                        <div>
                            <label class="block text-xs font-medium text-slate-700 dark:text-slate-300 mb-1">Categoria</label>
                            <select id="input-renda-categoria" class="w-full text-xs p-2.5 rounded-lg bg-slate-50 dark:bg-slate-700 border border-slate-200 dark:border-slate-600 text-slate-900 dark:text-white focus:outline-none focus:ring-2 focus:ring-teal-500">
                                <option value="Venda de Produtos">Venda de Produtos (Alimentos, Roupas, etc)</option>
                                <option value="Serviço / Bico">Serviço / Freelance / Bico</option>
                                <option value="Comissão / Bonificação">Comissão / Bonificação</option>
                                <option value="Outros">Outros Ganhos</option>
                            </select>
                        </div>

                        <div class="flex space-x-2 pt-2">
                            <button type="submit" class="flex-1 py-2.5 px-4 bg-teal-600 hover:bg-teal-700 text-white font-medium text-xs rounded-lg transition shadow-md flex items-center justify-center space-x-2">
                                <i class="fa-solid fa-save"></i>
                                <span id="btn-renda-submit-text">Salvar Extra</span>
                            </button>
                            <button type="button" id="btn-cancel-renda-edit" onclick="cancelRendaExtraEdit()" class="hidden py-2.5 px-3 bg-slate-200 dark:bg-slate-700 text-slate-700 dark:text-slate-200 text-xs font-medium rounded-lg hover:bg-slate-300 transition">
                                Cancelar
                            </button>
                        </div>
                    </form>
                </div>

                <!-- Tabela de Renda Extra com Edição e Exclusão -->
                <div class="lg:col-span-2 bg-white dark:bg-slate-800 p-6 rounded-2xl border border-slate-200 dark:border-slate-700 shadow-sm">
                    <div class="flex flex-col sm:flex-row items-start sm:items-center justify-between gap-4 mb-4">
                        <div>
                            <h3 class="text-md font-bold text-slate-900 dark:text-white flex items-center space-x-2">
                                <i class="fa-solid fa-list text-teal-500"></i>
                                <span>Histórico de Rendas Extras</span>
                            </h3>
                            <p class="text-xs text-slate-500 dark:text-slate-400">Edite ou exclua rapidamente qualquer lançamento extra cadastrado.</p>
                        </div>

                        <div class="flex items-center space-x-2 w-full sm:w-auto">
                            <select id="filter-mes-renda" onchange="renderRendaExtraTable()" class="text-xs bg-slate-100 dark:bg-slate-700 border border-slate-200 dark:border-slate-600 text-slate-800 dark:text-slate-100 rounded-lg p-2 focus:outline-none">
                                <option value="ALL">Todos os Meses</option>
                            </select>
                        </div>
                    </div>

                    <div class="overflow-x-auto">
                        <table class="w-full text-left text-xs">
                            <thead class="bg-slate-50 dark:bg-slate-700/50 text-slate-500 dark:text-slate-400 uppercase font-semibold">
                                <tr>
                                    <th class="p-3 rounded-l-lg">Descrição</th>
                                    <th class="p-3">Mês</th>
                                    <th class="p-3">Categoria</th>
                                    <th class="p-3 text-right">Valor</th>
                                    <th class="p-3 text-center rounded-r-lg">Ações</th>
                                </tr>
                            </thead>
                            <tbody id="tbody-renda-extra" class="divide-y divide-slate-100 dark:divide-slate-700/50 text-slate-700 dark:text-slate-300">
                            </tbody>
                        </table>
                        <div id="empty-renda-extra" class="hidden py-8 text-center text-slate-400 text-xs">
                            Nenhuma renda extra cadastrada para o mês selecionado.
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <!-- TAB 4: PEQUENOS GASTOS -->
        <div id="view-gastos" class="tab-content hidden space-y-6">
            <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
                <!-- Formulário de Pequenos Gastos -->
                <div class="bg-white dark:bg-slate-800 p-6 rounded-2xl border border-slate-200 dark:border-slate-700 shadow-sm h-fit">
                    <h3 class="text-md font-bold text-slate-900 dark:text-white mb-4 flex items-center space-x-2">
                        <i class="fa-solid fa-minus-circle text-amber-500"></i>
                        <span id="form-gasto-title">Novo Pequeno Gasto</span>
                    </h3>

                    <form id="form-gastos" onsubmit="handleSaveGasto(event)" class="space-y-4">
                        <input type="hidden" id="edit-gasto-id" value="">

                        <div>
                            <label class="block text-xs font-medium text-slate-700 dark:text-slate-300 mb-1">Descrição</label>
                            <input type="text" id="input-gasto-desc" required placeholder="Ex: Lanche, Uber, Café, Mercado pequeno" class="w-full text-xs p-2.5 rounded-lg bg-slate-50 dark:bg-slate-700 border border-slate-200 dark:border-slate-600 text-slate-900 dark:text-white focus:outline-none focus:ring-2 focus:ring-amber-500">
                        </div>

                        <div class="grid grid-cols-2 gap-3">
                            <div>
                                <label class="block text-xs font-medium text-slate-700 dark:text-slate-300 mb-1">Valor (R$)</label>
                                <input type="number" step="0.01" id="input-gasto-valor" required placeholder="0,00" class="w-full text-xs p-2.5 rounded-lg bg-slate-50 dark:bg-slate-700 border border-slate-200 dark:border-slate-600 text-slate-900 dark:text-white focus:outline-none focus:ring-2 focus:ring-amber-500">
                            </div>
                            <div>
                                <label class="block text-xs font-medium text-slate-700 dark:text-slate-300 mb-1">Mês de Referência</label>
                                <select id="input-gasto-mes" required class="w-full text-xs p-2.5 rounded-lg bg-slate-50 dark:bg-slate-700 border border-slate-200 dark:border-slate-600 text-slate-900 dark:text-white focus:outline-none focus:ring-2 focus:ring-amber-500">
                                </select>
                            </div>
                        </div>

                        <div>
                            <label class="block text-xs font-medium text-slate-700 dark:text-slate-300 mb-1">Categoria</label>
                            <select id="input-gasto-categoria" class="w-full text-xs p-2.5 rounded-lg bg-slate-50 dark:bg-slate-700 border border-slate-200 dark:border-slate-600 text-slate-900 dark:text-white focus:outline-none focus:ring-2 focus:ring-amber-500">
                                <option value="Alimentação / Lanches">Alimentação / Lanches</option>
                                <option value="Transporte / Uber">Transporte / Uber / Combustível</option>
                                <option value="Lazer / Entretenimento">Lazer / Entretenimento</option>
                                <option value="Compras do Dia a Dia">Compras do Dia a Dia</option>
                                <option value="Outros">Outros</option>
                            </select>
                        </div>

                        <div class="flex space-x-2 pt-2">
                            <button type="submit" class="flex-1 py-2.5 px-4 bg-amber-600 hover:bg-amber-700 text-white font-medium text-xs rounded-lg transition shadow-md flex items-center justify-center space-x-2">
                                <i class="fa-solid fa-save"></i>
                                <span id="btn-gasto-submit-text">Salvar Gasto</span>
                            </button>
                            <button type="button" id="btn-cancel-gasto-edit" onclick="cancelGastoEdit()" class="hidden py-2.5 px-3 bg-slate-200 dark:bg-slate-700 text-slate-700 dark:text-slate-200 text-xs font-medium rounded-lg hover:bg-slate-300 transition">
                                Cancelar
                            </button>
                        </div>
                    </form>
                </div>

                <!-- Tabela de Pequenos Gastos -->
                <div class="lg:col-span-2 bg-white dark:bg-slate-800 p-6 rounded-2xl border border-slate-200 dark:border-slate-700 shadow-sm">
                    <div class="flex flex-col sm:flex-row items-start sm:items-center justify-between gap-4 mb-4">
                        <div>
                            <h3 class="text-md font-bold text-slate-900 dark:text-white flex items-center space-x-2">
                                <i class="fa-solid fa-receipt text-amber-500"></i>
                                <span>Lista de Pequenos Gastos</span>
                            </h3>
                            <p class="text-xs text-slate-500 dark:text-slate-400">Controle rápido de pequenas despesas diárias.</p>
                        </div>

                        <div class="flex items-center space-x-2 w-full sm:w-auto">
                            <select id="filter-mes-gastos" onchange="renderGastosTable()" class="text-xs bg-slate-100 dark:bg-slate-700 border border-slate-200 dark:border-slate-600 text-slate-800 dark:text-slate-100 rounded-lg p-2 focus:outline-none">
                                <option value="ALL">Todos os Meses</option>
                            </select>
                        </div>
                    </div>

                    <div class="overflow-x-auto">
                        <table class="w-full text-left text-xs">
                            <thead class="bg-slate-50 dark:bg-slate-700/50 text-slate-500 dark:text-slate-400 uppercase font-semibold">
                                <tr>
                                    <th class="p-3 rounded-l-lg">Descrição</th>
                                    <th class="p-3">Mês</th>
                                    <th class="p-3">Categoria</th>
                                    <th class="p-3 text-right">Valor</th>
                                    <th class="p-3 text-center rounded-r-lg">Ações</th>
                                </tr>
                            </thead>
                            <tbody id="tbody-gastos" class="divide-y divide-slate-100 dark:divide-slate-700/50 text-slate-700 dark:text-slate-300">
                            </tbody>
                        </table>
                        <div id="empty-gastos" class="hidden py-8 text-center text-slate-400 text-xs">
                            Nenhum pequeno gasto registrado para o mês selecionado.
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <!-- TAB 5: GERENCIAR MESES E RENDA FIXA -->
        <div id="view-config" class="tab-content hidden space-y-6">
            <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
                <!-- Adicionar Mês -->
                <div class="bg-white dark:bg-slate-800 p-6 rounded-2xl border border-slate-200 dark:border-slate-700 shadow-sm">
                    <h3 class="text-md font-bold text-slate-900 dark:text-white mb-4 flex items-center space-x-2">
                        <i class="fa-solid fa-calendar-plus text-brand-500"></i>
                        <span>Adicionar Novo Mês ao Planejamento</span>
                    </h3>

                    <form onsubmit="handleAddMonth(event)" class="space-y-4">
                        <div class="grid grid-cols-2 gap-3">
                            <div>
                                <label class="block text-xs font-medium text-slate-700 dark:text-slate-300 mb-1">Nome do Mês</label>
                                <input type="text" id="new-month-label" placeholder="Ex: Setembro 2027" required class="w-full text-xs p-2.5 rounded-lg bg-slate-50 dark:bg-slate-700 border border-slate-200 dark:border-slate-600 text-slate-900 dark:text-white">
                            </div>
                            <div>
                                <label class="block text-xs font-medium text-slate-700 dark:text-slate-300 mb-1">Ano</label>
                                <input type="number" id="new-month-year" value="2027" required class="w-full text-xs p-2.5 rounded-lg bg-slate-50 dark:bg-slate-700 border border-slate-200 dark:border-slate-600 text-slate-900 dark:text-white">
                            </div>
                        </div>

                        <div>
                            <label class="block text-xs font-medium text-slate-700 dark:text-slate-300 mb-1">Renda Fixa para este Mês (R$)</label>
                            <input type="number" step="0.01" id="new-month-fixed" value="950.00" required class="w-full text-xs p-2.5 rounded-lg bg-slate-50 dark:bg-slate-700 border border-slate-200 dark:border-slate-600 text-slate-900 dark:text-white">
                        </div>

                        <button type="submit" class="w-full py-2.5 bg-brand-600 hover:bg-brand-700 text-white font-medium text-xs rounded-lg transition shadow flex items-center justify-center space-x-1.5">
                            <i class="fa-solid fa-plus"></i>
                            <span>Incluir Mês</span>
                        </button>
                    </form>
                </div>

                <!-- Alterar Renda Fixa Global -->
                <div class="bg-white dark:bg-slate-800 p-6 rounded-2xl border border-slate-200 dark:border-slate-700 shadow-sm">
                    <h3 class="text-md font-bold text-slate-900 dark:text-white mb-4 flex items-center space-x-2">
                        <i class="fa-solid fa-sack-dollar text-emerald-500"></i>
                        <span>Ajuste Global de Renda Fixa</span>
                    </h3>
                    <p class="text-xs text-slate-500 dark:text-slate-400 mb-4">Atualize o valor de Renda Fixa para TODOS os meses cadastrados de uma só vez.</p>

                    <form onsubmit="handleUpdateAllFixedIncome(event)" class="space-y-4">
                        <div>
                            <label class="block text-xs font-medium text-slate-700 dark:text-slate-300 mb-1">Novo Valor Global de Renda Fixa (R$)</label>
                            <input type="number" step="0.01" id="global-fixed-income" placeholder="Ex: 1200.00" required class="w-full text-xs p-2.5 rounded-lg bg-slate-50 dark:bg-slate-700 border border-slate-200 dark:border-slate-600 text-slate-900 dark:text-white">
                        </div>

                        <button type="submit" class="w-full py-2.5 bg-emerald-600 hover:bg-emerald-700 text-white font-medium text-xs rounded-lg transition shadow flex items-center justify-center space-x-1.5">
                            <i class="fa-solid fa-arrows-rotate"></i>
                            <span>Aplicar a Todos os Meses</span>
                        </button>
                    </form>
                </div>
            </div>

            <!-- Lista de Meses Cadastrados -->
            <div class="bg-white dark:bg-slate-800 p-6 rounded-2xl border border-slate-200 dark:border-slate-700 shadow-sm">
                <h3 class="text-md font-bold text-slate-900 dark:text-white mb-4 flex items-center space-x-2">
                    <i class="fa-solid fa-calendar-days text-brand-500"></i>
                    <span>Gerenciar Meses Existentes</span>
                </h3>

                <div class="overflow-x-auto">
                    <table class="w-full text-left text-xs">
                        <thead class="bg-slate-50 dark:bg-slate-700/50 text-slate-500 dark:text-slate-400 uppercase font-semibold">
                            <tr>
                                <th class="p-3">Mês / Ano</th>
                                <th class="p-3">Renda Fixa do Mês</th>
                                <th class="p-3 text-center">Ação</th>
                            </tr>
                        </thead>
                        <tbody id="tbody-months-config" class="divide-y divide-slate-100 dark:divide-slate-700/50 text-slate-700 dark:text-slate-300">
                        </tbody>
                    </table>
                </div>
            </div>
        </div>

    </main>

    <!-- Modal de Auth / Login -->
    <div id="auth-modal" class="fixed inset-0 z-50 bg-slate-900/60 backdrop-blur-sm hidden flex items-center justify-center p-4">
        <div class="bg-white dark:bg-slate-800 w-full max-w-md rounded-2xl border border-slate-200 dark:border-slate-700 shadow-2xl p-6 relative">
            <button onclick="closeAuthModal()" class="absolute top-4 right-4 text-slate-400 hover:text-slate-600 dark:hover:text-white">
                <i class="fa-solid fa-xmark text-lg"></i>
            </button>

            <div class="text-center mb-6">
                <div class="w-12 h-12 bg-brand-100 dark:bg-brand-950/60 text-brand-600 dark:text-brand-400 rounded-2xl flex items-center justify-center mx-auto mb-2 text-xl">
                    <i class="fa-solid fa-cloud-arrow-up"></i>
                </div>
                <h3 class="text-lg font-bold text-slate-900 dark:text-white">Sincronização & Login</h3>
                <p class="text-xs text-slate-500 dark:text-slate-400 mt-1">Conecte sua conta para sincronizar e compartilhar dados entre múltiplos dispositivos.</p>
            </div>

            <form id="form-auth" onsubmit="handleAuthSubmit(event)" class="space-y-4">
                <div>
                    <label class="block text-xs font-medium text-slate-700 dark:text-slate-300 mb-1">E-mail</label>
                    <input type="email" id="auth-email" required placeholder="seu@email.com" class="w-full text-xs p-2.5 rounded-lg bg-slate-50 dark:bg-slate-700 border border-slate-200 dark:border-slate-600 text-slate-900 dark:text-white">
                </div>
                <div>
                    <label class="block text-xs font-medium text-slate-700 dark:text-slate-300 mb-1">Senha</label>
                    <input type="password" id="auth-password" required placeholder="••••••••" class="w-full text-xs p-2.5 rounded-lg bg-slate-50 dark:bg-slate-700 border border-slate-200 dark:border-slate-600 text-slate-900 dark:text-white">
                </div>

                <div class="flex items-center space-x-2 pt-2">
                    <button type="submit" onclick="window.authAction='login'" class="flex-1 py-2.5 bg-brand-600 hover:bg-brand-700 text-white font-semibold text-xs rounded-lg transition shadow">
                        Entrar
                    </button>
                    <button type="submit" onclick="window.authAction='signup'" class="flex-1 py-2.5 bg-slate-200 dark:bg-slate-700 hover:bg-slate-300 dark:hover:bg-slate-600 text-slate-800 dark:text-slate-200 font-semibold text-xs rounded-lg transition">
                        Criar Conta
                    </button>
                </div>
            </form>

            <div id="auth-user-info" class="hidden text-center space-y-4 pt-2">
                <p class="text-xs text-emerald-600 dark:text-emerald-400 font-semibold">
                    <i class="fa-solid fa-circle-check mr-1"></i> Sincronizado como: <span id="auth-active-email"></span>
                </p>
                <div class="p-3 bg-slate-50 dark:bg-slate-700/50 rounded-lg text-xs text-slate-500 text-left space-y-1">
                    <div class="font-bold text-slate-700 dark:text-slate-300">ID do Usuário:</div>
                    <div id="auth-user-id" class="font-mono text-[10px] break-all select-all bg-white dark:bg-slate-800 p-1.5 rounded border border-slate-200 dark:border-slate-600"></div>
                </div>
                <button onclick="handleLogout()" class="w-full py-2 bg-rose-600 hover:bg-rose-700 text-white font-semibold text-xs rounded-lg transition">
                    Sair da Conta
                </button>
            </div>
        </div>
    </div>

    <!-- Toast Notification -->
    <div id="toast" class="fixed bottom-5 right-5 z-50 transform translate-y-20 opacity-0 transition-all duration-300 pointer-events-none bg-slate-900 text-white text-xs px-4 py-3 rounded-xl shadow-xl flex items-center space-x-2 border border-slate-700">
        <i id="toast-icon" class="fa-solid fa-circle-check text-emerald-400"></i>
        <span id="toast-message">Operação realizada com sucesso.</span>
    </div>

    <!-- JS Application Engine -->
    <script>
        // Initial dataset preserving original spreadsheet values as FIXED INCOMES
        const INITIAL_MONTHS = [
            { id: "out26", label: "Outubro 2026", year: 2026, fixedIncome: 1650.85 },
            { id: "nov26", label: "Novembro 2026", year: 2026, fixedIncome: 1725.85 },
            { id: "dez26", label: "Dezembro 2026", year: 2026, fixedIncome: 1725.85 },
            { id: "jan27", label: "Janeiro 2027", year: 2027, fixedIncome: 1337.03 },
            { id: "fev27", label: "Fevereiro 2027", year: 2027, fixedIncome: 1337.03 },
            { id: "mar27", label: "Março 2027", year: 2027, fixedIncome: 1725.85 },
            { id: "abr27", label: "Abril 2027", year: 2027, fixedIncome: 1725.85 },
            { id: "mai27", label: "Maio 2027", year: 2027, fixedIncome: 1725.85 },
            { id: "jun27", label: "Junho 2027", year: 2027, fixedIncome: 1725.85 },
            { id: "jul27", label: "Julho 2027", year: 2027, fixedIncome: 1725.85 },
            { id: "ago27", label: "Agosto 2027", year: 2027, fixedIncome: 1725.85 }
        ];

        const INITIAL_CONTRACTS = [
            {
                id: "vectra",
                title: "VECTRA",
                installments: [
                    { id: "v_0", name: "Entrada", monthId: "ago26", vencimento: "ago/26", originalVal: 1500.00, p1: 800.00, p2: 450.00, p3: 150.00, p4: 100.00, status: "Pago (x)" },
                    { id: "v_1", name: "1ª Parcela", monthId: "set26", vencimento: "set/26", originalVal: 1000.00, p1: 600.00, p2: 320.00, p3: 0, p4: 0, status: "Pago ( )" },
                    { id: "v_2", name: "2ª Parcela", monthId: "out26", vencimento: "out/26", originalVal: 1000.00, p1: 0, p2: 0, p3: 0, p4: 0, status: "Pago ( )" },
                    { id: "v_3", name: "3ª Parcela", monthId: "nov26", vencimento: "nov/26", originalVal: 1000.00, p1: 0, p2: 0, p3: 0, p4: 0, status: "Pago ( )" },
                    { id: "v_4", name: "4ª Parcela", monthId: "dez26", vencimento: "dez/26", originalVal: 1000.00, p1: 0, p2: 0, p3: 0, p4: 0, status: "Pago ( )" },
                    { id: "v_5", name: "5ª Parcela", monthId: "jan27", vencimento: "jan/27", originalVal: 1000.00, p1: 0, p2: 0, p3: 0, p4: 0, status: "Pago ( )" },
                    { id: "v_6", name: "6ª Parcela", monthId: "fev27", vencimento: "fev/27", originalVal: 1000.00, p1: 0, p2: 0, p3: 0, p4: 0, status: "Pago ( )" },
                    { id: "v_7", name: "7ª Parcela", monthId: "mar27", vencimento: "mar/27", originalVal: 1000.00, p1: 0, p2: 0, p3: 0, p4: 0, status: "Pago ( )" },
                    { id: "v_8", name: "8ª Parcela", monthId: "abr27", vencimento: "abr/27", originalVal: 1000.00, p1: 0, p2: 0, p3: 0, p4: 0, status: "Pago ( )" },
                    { id: "v_9", name: "9ª Parcela", monthId: "mai27", vencimento: "mai/27", originalVal: 980.00,  p1: 0, p2: 0, p3: 0, p4: 0, status: "Pago ( )" },
                    { id: "v_10", name: "10ª Parcela", monthId: "jun27", vencimento: "jun/27", originalVal: 1000.00, p1: 0, p2: 0, p3: 0, p4: 0, status: "Pago ( )" }
                ]
            },
            {
                id: "simone",
                title: "SIMONE",
                installments: [
                    { id: "s_1", name: "1ª Parcela", monthId: "out26", vencimento: "out/26", originalVal: 50.00, p1: 50.00, p2: 0, p3: 0, p4: 0, status: "Pago ( )" },
                    { id: "s_2", name: "2ª Parcela", monthId: "nov26", vencimento: "nov/26", originalVal: 50.00, p1: 0, p2: 0, p3: 0, p4: 0, status: "Pago ( )" },
                    { id: "s_3", name: "3ª Parcela", monthId: "dez26", vencimento: "dez/26", originalVal: 50.00, p1: 0, p2: 0, p3: 0, p4: 0, status: "Pago ( )" },
                    { id: "s_4", name: "4ª Parcela", monthId: "jan27", vencimento: "jan/27", originalVal: 100.00, p1: 0, p2: 0, p3: 0, p4: 0, status: "Pago ( )" },
                    { id: "s_5", name: "5ª Parcela", monthId: "fev27", vencimento: "fev/27", originalVal: 100.00, p1: 0, p2: 0, p3: 0, p4: 0, status: "Pago ( )" },
                    { id: "s_6", name: "6ª Parcela", monthId: "mar27", vencimento: "mar/27", originalVal: 100.00, p1: 0, p2: 0, p3: 0, p4: 0, status: "Pago ( )" },
                    { id: "s_7", name: "7ª Parcela", monthId: "abr27", vencimento: "abr/27", originalVal: 100.00, p1: 0, p2: 0, p3: 0, p4: 0, status: "Pago ( )" },
                    { id: "s_8", name: "8ª Parcela", monthId: "mai27", vencimento: "mai/27", originalVal: 80.00, p1: 0, p2: 0, p3: 0, p4: 0, status: "Pago ( )" },
                    { id: "s_9", name: "9ª Parcela", monthId: "jun27", vencimento: "jun/27", originalVal: 50.00, p1: 0, p2: 0, p3: 0, p4: 0, status: "Pago ( )" },
                    { id: "s_10", name: "10ª Parcela", monthId: "jul27", vencimento: "jul/27", originalVal: 40.00, p1: 0, p2: 0, p3: 0, p4: 0, status: "Pago ( )" }
                ]
            },
            {
                id: "adriana",
                title: "ADRIANA",
                installments: [
                    { id: "a_1", name: "1ª Parcela", monthId: "out26", vencimento: "out/26", originalVal: 122.42, p1: 122.42, p2: 0, p3: 0, p4: 0, status: "Pago ( )" },
                    { id: "a_2", name: "2ª Parcela", monthId: "nov26", vencimento: "nov/26", originalVal: 122.42, p1: 0, p2: 0, p3: 0, p4: 0, status: "Pago ( )" },
                    { id: "a_3", name: "3ª Parcela", monthId: "dez26", vencimento: "dez/26", originalVal: 122.42, p1: 0, p2: 0, p3: 0, p4: 0, status: "Pago ( )" },
                    { id: "a_4", name: "4ª Parcela", monthId: "jan27", vencimento: "jan/27", originalVal: 122.42, p1: 0, p2: 0, p3: 0, p4: 0, status: "Pago ( )" }
                ]
            }
        ];

        // Global State
        let monthsData = [];
        let contractsData = [];
        let rendaExtraData = [];
        let pequenosGastosData = [];
        let currentUser = null;
        let unsubscribeFirestore = null;

        window.addEventListener('load', () => {
            initTheme();
            loadStateLocal();
            initFirebaseListener();
            renderAllViews();
        });

        function initTheme() {
            const themeToggleBtn = document.getElementById('themeToggle');
            themeToggleBtn.addEventListener('click', () => {
                document.documentElement.classList.toggle('dark');
                localStorage.setItem('theme', document.documentElement.classList.contains('dark') ? 'dark' : 'light');
            });
            if (localStorage.getItem('theme') === 'light') {
                document.documentElement.classList.remove('dark');
            }
        }

        function initFirebaseListener() {
            if (!window.fbAuth) return;
            const { onAuthStateChanged, doc, onSnapshot } = window.fbMethods;

            onAuthStateChanged(window.fbAuth, (user) => {
                currentUser = user;
                const statusText = document.getElementById('sync-status-text');
                const userDisplayName = document.getElementById('user-display-name');

                if (user && !user.isAnonymous) {
                    statusText.textContent = `Sincronizado na Nuvem (${user.email})`;
                    userDisplayName.textContent = user.email.split('@')[0];

                    document.getElementById('form-auth').classList.add('hidden');
                    document.getElementById('auth-user-info').classList.remove('hidden');
                    document.getElementById('auth-active-email').textContent = user.email;
                    document.getElementById('auth-user-id').textContent = user.uid;

                    const docRef = doc(window.fbDb, 'artifacts', window.fbAppId, 'users', user.uid, 'data', 'financeData');
                    if (unsubscribeFirestore) unsubscribeFirestore();

                    unsubscribeFirestore = onSnapshot(docRef, (docSnap) => {
                        if (docSnap.exists()) {
                            const data = docSnap.data();
                            if (data.monthsData) monthsData = data.monthsData;
                            if (data.contractsData) contractsData = data.contractsData;
                            if (data.rendaExtraData) rendaExtraData = data.rendaExtraData;
                            if (data.pequenosGastosData) pequenosGastosData = data.pequenosGastosData;
                            renderAllViews();
                        } else {
                            saveState();
                        }
                    }, (err) => console.error("Firestore sync error:", err));

                } else {
                    statusText.textContent = "Modo Local (Inicie sessão para sincronizar na nuvem e compartilhar)";
                    userDisplayName.textContent = "Entrar / Cadastrar";

                    document.getElementById('form-auth').classList.remove('hidden');
                    document.getElementById('auth-user-info').classList.add('hidden');

                    if (unsubscribeFirestore) unsubscribeFirestore();
                }
            });
        }

        function loadStateLocal() {
            const savedMonths = localStorage.getItem('fin_months_v2');
            const savedContracts = localStorage.getItem('fin_contracts_v2');
            const savedRenda = localStorage.getItem('fin_renda_v2');
            const savedGastos = localStorage.getItem('fin_gastos_v2');

            monthsData = savedMonths ? JSON.parse(savedMonths) : JSON.parse(JSON.stringify(INITIAL_MONTHS));
            contractsData = savedContracts ? JSON.parse(savedContracts) : JSON.parse(JSON.stringify(INITIAL_CONTRACTS));
            rendaExtraData = savedRenda ? JSON.parse(savedRenda) : [];
            pequenosGastosData = savedGastos ? JSON.parse(savedGastos) : [];
        }

        function saveState() {
            localStorage.setItem('fin_months_v2', JSON.stringify(monthsData));
            localStorage.setItem('fin_contracts_v2', JSON.stringify(contractsData));
            localStorage.setItem('fin_renda_v2', JSON.stringify(rendaExtraData));
            localStorage.setItem('fin_gastos_v2', JSON.stringify(pequenosGastosData));

            if (currentUser && !currentUser.isAnonymous && window.fbDb) {
                const { doc, setDoc } = window.fbMethods;
                const docRef = doc(window.fbDb, 'artifacts', window.fbAppId, 'users', currentUser.uid, 'data', 'financeData');
                setDoc(docRef, {
                    monthsData,
                    contractsData,
                    rendaExtraData,
                    pequenosGastosData,
                    updatedAt: new Date().toISOString()
                }, { merge: true }).catch(err => console.error("Error saving cloud data:", err));
            }

            renderAllViews();
        }

        function confirmResetData() {
            if (confirm("Deseja restaurar os dados originais da planilha? Todos os dados serão resetados.")) {
                monthsData = JSON.parse(JSON.stringify(INITIAL_MONTHS));
                contractsData = JSON.parse(JSON.stringify(INITIAL_CONTRACTS));
                rendaExtraData = [];
                pequenosGastosData = [];
                saveState();
                showToast("Dados restaurados com sucesso!", "success");
            }
        }

        function switchTab(tabId) {
            document.querySelectorAll('.tab-btn').forEach(btn => {
                btn.classList.remove('text-brand-600', 'dark:text-brand-400', 'bg-brand-50', 'dark:bg-brand-950/50', 'border', 'border-brand-200', 'dark:border-brand-800/50');
                btn.classList.add('text-slate-600', 'dark:text-slate-400');
            });
            document.querySelectorAll('.tab-content').forEach(content => content.classList.add('hidden'));

            const selectedBtn = document.getElementById(`tab-${tabId}`);
            const selectedContent = document.getElementById(`view-${tabId}`);

            if (selectedBtn && selectedContent) {
                selectedBtn.classList.remove('text-slate-600', 'dark:text-slate-400');
                selectedBtn.classList.add('text-brand-600', 'dark:text-brand-400', 'bg-brand-50', 'dark:bg-brand-950/50', 'border', 'border-brand-200', 'dark:border-brand-800/50');
                selectedContent.classList.remove('hidden');
            }
        }

        function renderAllViews() {
            populateMonthDropdowns();
            renderYearFilterOptions();
            renderDashboardKPIs();
            renderMonthlyCards();
            renderContractsTable();
            renderRendaExtraTable();
            renderGastosTable();
            renderMonthsConfigTable();
        }

        function formatBRL(val) {
            return new Intl.NumberFormat('pt-BR', { style: 'currency', currency: 'BRL' }).format(val || 0);
        }

        function populateMonthDropdowns() {
            const dropdowns = ['input-renda-mes', 'filter-mes-renda', 'input-gasto-mes', 'filter-mes-gastos'];
            dropdowns.forEach(id => {
                const el = document.getElementById(id);
                if (!el) return;
                const currentVal = el.value;
                el.innerHTML = id.startsWith('filter') ? '<option value="ALL">Todos os Meses</option>' : '';

                monthsData.forEach(m => {
                    const opt = document.createElement('option');
                    opt.value = m.id;
                    opt.textContent = m.label;
                    el.appendChild(opt);
                });

                if (currentVal) el.value = currentVal;
            });
        }

        function renderYearFilterOptions() {
            const yearSelect = document.getElementById('filter-year');
            const currentYearVal = yearSelect.value;
            const years = [...new Set(monthsData.map(m => m.year))].sort();

            yearSelect.innerHTML = '<option value="ALL">Todos os Anos</option>';
            years.forEach(y => {
                const opt = document.createElement('option');
                opt.value = y;
                opt.textContent = y;
                yearSelect.appendChild(opt);
            });
            if (currentYearVal) yearSelect.value = currentYearVal;
        }

        function calculateContractTotals(contract) {
            let totalOriginal = 0;
            let totalPago = 0;
            contract.installments.forEach(inst => {
                totalOriginal += (inst.originalVal || 0);
                totalPago += (inst.p1 || 0) + (inst.p2 || 0) + (inst.p3 || 0) + (inst.p4 || 0);
            });
            return { totalOriginal, totalPago, totalRestante: Math.max(0, totalOriginal - totalPago) };
        }

        function calculateGlobalTotals() {
            let grandOriginal = 0;
            let grandPago = 0;

            contractsData.forEach(c => {
                const { totalOriginal, totalPago } = calculateContractTotals(c);
                grandOriginal += totalOriginal;
                grandPago += totalPago;
            });

            let totalPequenosGastos = pequenosGastosData.reduce((acc, g) => acc + g.valor, 0);
            let totalRendaExtra = rendaExtraData.reduce((acc, r) => acc + r.valor, 0);

            let totalRendaFixa = monthsData.reduce((acc, m) => acc + (m.fixedIncome || 0), 0);

            const saldoLiquidoGlobal = (totalRendaFixa + totalRendaExtra) - (grandOriginal + totalPequenosGastos);

            return {
                grandOriginal,
                grandPago,
                grandRestante: Math.max(0, grandOriginal - grandPago),
                totalPequenosGastos,
                totalRendaExtra,
                saldoLiquidoGlobal
            };
        }

        function renderDashboardKPIs() {
            const totals = calculateGlobalTotals();

            document.getElementById('kpi-total-dividas-restantes').textContent = formatBRL(totals.grandRestante);
            document.getElementById('kpi-total-dividas-original').textContent = `Original Total: ${formatBRL(totals.grandOriginal)}`;
            document.getElementById('kpi-total-pago').textContent = formatBRL(totals.grandPago);
            document.getElementById('kpi-renda-extra').textContent = formatBRL(totals.totalRendaExtra);
            document.getElementById('kpi-pequenos-gastos').textContent = formatBRL(totals.totalPequenosGastos);
            document.getElementById('kpi-saldo-liquido').textContent = formatBRL(totals.saldoLiquidoGlobal);

            document.getElementById('total-dividas-header').textContent = `TOTAL RESTANTE: ${formatBRL(totals.grandRestante)}`;
            document.getElementById('total-dividas-original-header').textContent = formatBRL(totals.grandOriginal);
            document.getElementById('total-dividas-pago-header').textContent = formatBRL(totals.grandPago);
        }

        function renderMonthlyCards() {
            const container = document.getElementById('monthly-cards-grid');
            const selectedYear = document.getElementById('filter-year').value;
            container.innerHTML = '';

            monthsData.forEach(month => {
                if (selectedYear !== 'ALL' && month.year.toString() !== selectedYear) return;

                let monthDividasOriginal = 0;
                let monthDividasPagas = 0;

                contractsData.forEach(contract => {
                    contract.installments.forEach(inst => {
                        if (inst.monthId === month.id) {
                            monthDividasOriginal += (inst.originalVal || 0);
                            monthDividasPagas += (inst.p1 || 0) + (inst.p2 || 0) + (inst.p3 || 0) + (inst.p4 || 0);
                        }
                    });
                });

                let monthPequenosGastos = pequenosGastosData
                    .filter(g => g.mesId === month.id)
                    .reduce((acc, g) => acc + g.valor, 0);

                let monthRendaExtra = rendaExtraData
                    .filter(r => r.mesId === month.id)
                    .reduce((acc, r) => acc + r.valor, 0);

                const totalReceitaMes = (month.fixedIncome || 0) + monthRendaExtra;
                const totalDespesasMes = monthDividasOriginal + monthPequenosGastos;
                const saldoMes = totalReceitaMes - totalDespesasMes;

                const cardHtml = `
                    <div class="bg-slate-50 dark:bg-slate-800/60 rounded-xl p-4 border border-slate-200 dark:border-slate-700/80 shadow-sm flex flex-col justify-between space-y-3">
                        <div>
                            <div class="flex items-center justify-between border-b border-slate-200 dark:border-slate-700 pb-2 mb-3">
                                <span class="font-extrabold text-sm text-slate-800 dark:text-white uppercase tracking-wider">${month.label}</span>
                                <span class="text-[10px] px-2 py-0.5 rounded-full font-bold ${saldoMes >= 0 ? 'bg-emerald-100 text-emerald-700 dark:bg-emerald-950/60 dark:text-emerald-300' : 'bg-rose-100 text-rose-700 dark:bg-rose-950/60 dark:text-rose-300'}">
                                    ${saldoMes >= 0 ? 'Positivo' : 'Deficitário'}
                                </span>
                            </div>

                            <div class="space-y-2 text-xs">
                                <div class="bg-white dark:bg-slate-700/40 p-2 rounded-lg space-y-1">
                                    <div class="text-[11px] font-bold text-slate-500 dark:text-slate-400 uppercase">Dívidas Programadas</div>
                                    <div class="flex justify-between font-medium">
                                        <span class="text-slate-600 dark:text-slate-300">Previsto:</span>
                                        <span class="text-rose-600 dark:text-rose-400 font-bold">${formatBRL(monthDividasOriginal)}</span>
                                    </div>
                                    <div class="flex justify-between text-[11px] text-slate-500">
                                        <span>Amortizado:</span>
                                        <span class="text-emerald-600 dark:text-emerald-400 font-semibold">${formatBRL(monthDividasPagas)}</span>
                                    </div>
                                </div>

                                <div class="bg-white dark:bg-slate-700/40 p-2 rounded-lg space-y-1">
                                    <div class="text-[11px] font-bold text-slate-500 dark:text-slate-400 uppercase">Receitas</div>
                                    <div class="flex justify-between items-center">
                                        <span class="text-slate-600 dark:text-slate-300">Renda Fixa:</span>
                                        <input type="number" step="0.01" value="${month.fixedIncome || 0}" onchange="updateMonthFixedIncome('${month.id}', this.value)" class="w-24 text-right text-xs font-bold bg-slate-100 dark:bg-slate-800 border border-slate-200 dark:border-slate-600 rounded px-1.5 py-0.5">
                                    </div>
                                    <div class="flex justify-between items-center text-teal-600 dark:text-teal-400 font-medium pt-1">
                                        <span>Renda Extra:</span>
                                        <div class="flex items-center space-x-1">
                                            <span class="font-bold">+ ${formatBRL(monthRendaExtra)}</span>
                                            <button onclick="quickAddRendaExtra('${month.id}')" title="Adicionar Renda Extra para este mês" class="px-1.5 py-0.5 bg-teal-100 dark:bg-teal-950 text-teal-700 dark:text-teal-300 rounded text-[10px] font-bold hover:bg-teal-200">
                                                + Extra
                                            </button>
                                        </div>
                                    </div>
                                </div>

                                ${monthPequenosGastos > 0 ? `
                                <div class="bg-amber-50 dark:bg-amber-950/20 border border-amber-200 dark:border-amber-800/40 p-2 rounded-lg flex justify-between text-amber-800 dark:text-amber-300">
                                    <span>Pequenos Gastos:</span>
                                    <span class="font-bold">- ${formatBRL(monthPequenosGastos)}</span>
                                </div>` : ''}
                            </div>
                        </div>

                        <div class="pt-2 border-t border-slate-200 dark:border-slate-700 flex justify-between items-center text-xs">
                            <span class="font-bold text-slate-600 dark:text-slate-400">Saldo do Mês:</span>
                            <span class="font-extrabold text-sm ${saldoMes >= 0 ? 'text-brand-600 dark:text-brand-400' : 'text-rose-600 dark:text-rose-400'}">
                                ${formatBRL(saldoMes)}
                            </span>
                        </div>
                    </div>
                `;
                container.insertAdjacentHTML('beforeend', cardHtml);
            });
        }

        function renderContractsTable() {
            const container = document.getElementById('contracts-container');
            container.innerHTML = '';

            contractsData.forEach(contract => {
                const { totalOriginal, totalPago, totalRestante } = calculateContractTotals(contract);

                let rowsHtml = '';
                contract.installments.forEach(inst => {
                    const totalPagoInst = (inst.p1 || 0) + (inst.p2 || 0) + (inst.p3 || 0) + (inst.p4 || 0);
                    const isFullyPaid = totalPagoInst >= inst.originalVal;

                    rowsHtml += `
                        <tr class="hover:bg-slate-50 dark:hover:bg-slate-700/30 transition text-xs">
                            <td class="p-3 font-semibold text-slate-800 dark:text-slate-200">${inst.name}</td>
                            <td class="p-3 text-slate-500 dark:text-slate-400">${inst.vencimento}</td>
                            <td class="p-3 font-medium text-slate-700 dark:text-slate-300">${formatBRL(inst.originalVal)}</td>
                            <td class="p-2"><input type="number" step="0.01" value="${inst.p1 || ''}" onchange="updatePayment('${contract.id}', '${inst.id}', 'p1', this.value)" placeholder="0.00" class="w-20 p-1 text-xs rounded border border-slate-200 dark:border-slate-600 bg-white dark:bg-slate-700"></td>
                            <td class="p-2"><input type="number" step="0.01" value="${inst.p2 || ''}" onchange="updatePayment('${contract.id}', '${inst.id}', 'p2', this.value)" placeholder="0.00" class="w-20 p-1 text-xs rounded border border-slate-200 dark:border-slate-600 bg-white dark:bg-slate-700"></td>
                            <td class="p-2"><input type="number" step="0.01" value="${inst.p3 || ''}" onchange="updatePayment('${contract.id}', '${inst.id}', 'p3', this.value)" placeholder="0.00" class="w-20 p-1 text-xs rounded border border-slate-200 dark:border-slate-600 bg-white dark:bg-slate-700"></td>
                            <td class="p-2"><input type="number" step="0.01" value="${inst.p4 || ''}" onchange="updatePayment('${contract.id}', '${inst.id}', 'p4', this.value)" placeholder="0.00" class="w-20 p-1 text-xs rounded border border-slate-200 dark:border-slate-600 bg-white dark:bg-slate-700"></td>
                            <td class="p-3 font-bold ${isFullyPaid ? 'text-emerald-600 dark:text-emerald-400' : 'text-slate-700 dark:text-slate-300'}">${formatBRL(totalPagoInst)}</td>
                            <td class="p-3">
                                <select onchange="updateStatus('${contract.id}', '${inst.id}', this.value)" class="p-1 rounded text-xs font-semibold bg-slate-100 dark:bg-slate-700 border border-slate-200 dark:border-slate-600">
                                    <option value="Pago (x)" ${inst.status === 'Pago (x)' ? 'selected' : ''}>Pago (x)</option>
                                    <option value="Pago ( )" ${inst.status === 'Pago ( )' ? 'selected' : ''}>Pendente ( )</option>
                                </select>
                            </td>
                        </tr>
                    `;
                });

                container.insertAdjacentHTML('beforeend', `
                    <div class="bg-white dark:bg-slate-800 rounded-2xl border border-slate-200 dark:border-slate-700 shadow-sm overflow-hidden">
                        <div class="bg-slate-800 text-white p-4 flex flex-col sm:flex-row items-start sm:items-center justify-between gap-4">
                            <div>
                                <h3 class="text-lg font-bold tracking-wide">${contract.title}</h3>
                                <p class="text-xs text-slate-400">Controle individual de liquidação.</p>
                            </div>
                            <div class="flex items-center space-x-4 text-xs">
                                <div><span class="text-slate-400 block">Original:</span><span class="font-bold text-white text-sm">${formatBRL(totalOriginal)}</span></div>
                                <div><span class="text-slate-400 block">Pago:</span><span class="font-bold text-emerald-400 text-sm">${formatBRL(totalPago)}</span></div>
                                <div><span class="text-slate-400 block">Restante:</span><span class="font-bold text-rose-400 text-sm">${formatBRL(totalRestante)}</span></div>
                            </div>
                        </div>
                        <div class="overflow-x-auto">
                            <table class="w-full text-left">
                                <thead class="bg-slate-50 dark:bg-slate-700/50 text-slate-500 dark:text-slate-400 uppercase text-[11px] font-bold">
                                    <tr>
                                        <th class="p-3">Parcela</th><th class="p-3">Vencimento</th><th class="p-3">Valor Orig.</th>
                                        <th class="p-3">P1</th><th class="p-3">P2</th><th class="p-3">P3</th><th class="p-3">P4</th>
                                        <th class="p-3">Total Pago</th><th class="p-3">Status</th>
                                    </tr>
                                </thead>
                                <tbody class="divide-y divide-slate-100 dark:divide-slate-700/50">${rowsHtml}</tbody>
                            </table>
                        </div>
                    </div>
                `);
            });
        }

        function updatePayment(contractId, instId, field, value) {
            const numVal = parseFloat(value) || 0;
            const contract = contractsData.find(c => c.id === contractId);
            if (contract) {
                const inst = contract.installments.find(i => i.id === instId);
                if (inst) {
                    inst[field] = numVal;
                    const sum = (inst.p1 || 0) + (inst.p2 || 0) + (inst.p3 || 0) + (inst.p4 || 0);
                    inst.status = sum >= inst.originalVal ? "Pago (x)" : "Pago ( )";
                    saveState();
                    showToast("Pagamento atualizado!", "success");
                }
            }
        }

        function updateStatus(contractId, instId, statusVal) {
            const contract = contractsData.find(c => c.id === contractId);
            if (contract) {
                const inst = contract.installments.find(i => i.id === instId);
                if (inst) {
                    inst.status = statusVal;
                    saveState();
                    showToast("Status alterado!", "info");
                }
            }
        }

        // RENDA EXTRA LOGIC
        function quickAddRendaExtra(mesId) {
            switchTab('renda-extra');
            document.getElementById('input-renda-mes').value = mesId;
            document.getElementById('input-renda-desc').focus();
        }

        function handleSaveRendaExtra(e) {
            e.preventDefault();
            const editId = document.getElementById('edit-renda-id').value;
            const descricao = document.getElementById('input-renda-desc').value.trim();
            const valor = parseFloat(document.getElementById('input-renda-valor').value);
            const mesId = document.getElementById('input-renda-mes').value;
            const categoria = document.getElementById('input-renda-categoria').value;

            if (!descricao || isNaN(valor) || valor <= 0) return;

            if (editId) {
                const item = rendaExtraData.find(r => r.id === editId);
                if (item) {
                    item.descricao = descricao;
                    item.valor = valor;
                    item.mesId = mesId;
                    item.categoria = categoria;
                    showToast("Renda Extra atualizada!", "success");
                }
            } else {
                rendaExtraData.unshift({
                    id: 'r_' + Date.now(),
                    descricao,
                    valor,
                    mesId,
                    categoria
                });
                showToast("Renda Extra registrada!", "success");
            }

            saveState();
            cancelRendaExtraEdit();
        }

        function editRendaExtra(id) {
            const item = rendaExtraData.find(r => r.id === id);
            if (!item) return;

            document.getElementById('edit-renda-id').value = item.id;
            document.getElementById('input-renda-desc').value = item.descricao;
            document.getElementById('input-renda-valor').value = item.valor;
            document.getElementById('input-renda-mes').value = item.mesId;
            document.getElementById('input-renda-categoria').value = item.categoria || 'Venda de Produtos';

            document.getElementById('form-renda-title').textContent = "Editar Renda Extra";
            document.getElementById('btn-renda-submit-text').textContent = "Atualizar";
            document.getElementById('btn-cancel-renda-edit').classList.remove('hidden');

            window.scrollTo({ top: 0, behavior: 'smooth' });
        }

        function cancelRendaExtraEdit() {
            document.getElementById('form-renda-extra').reset();
            document.getElementById('edit-renda-id').value = '';
            document.getElementById('form-renda-title').textContent = "Registrar Renda Extra";
            document.getElementById('btn-renda-submit-text').textContent = "Salvar Extra";
            document.getElementById('btn-cancel-renda-edit').classList.add('hidden');
        }

        function deleteRendaExtra(id) {
            if (confirm("Excluir este lançamento de Renda Extra?")) {
                rendaExtraData = rendaExtraData.filter(r => r.id !== id);
                saveState();
                showToast("Renda extra removida.", "info");
            }
        }

        function renderRendaExtraTable() {
            const tbody = document.getElementById('tbody-renda-extra');
            const filterMes = document.getElementById('filter-mes-renda').value;
            const emptyMsg = document.getElementById('empty-renda-extra');

            tbody.innerHTML = '';
            const filtered = rendaExtraData.filter(r => filterMes === 'ALL' || r.mesId === filterMes);

            if (filtered.length === 0) {
                emptyMsg.classList.remove('hidden');
                return;
            }
            emptyMsg.classList.add('hidden');

            filtered.forEach(item => {
                const monthObj = monthsData.find(m => m.id === item.mesId);
                const tr = document.createElement('tr');
                tr.className = 'hover:bg-slate-50 dark:hover:bg-slate-700/30 transition';
                tr.innerHTML = `
                    <td class="p-3 font-semibold text-slate-800 dark:text-slate-200">${item.descricao}</td>
                    <td class="p-3 text-slate-500 dark:text-slate-400">${monthObj ? monthObj.label : item.mesId}</td>
                    <td class="p-3 text-slate-500 dark:text-slate-400">${item.categoria || 'Geral'}</td>
                    <td class="p-3 text-right font-bold text-teal-600 dark:text-teal-400">+ ${formatBRL(item.valor)}</td>
                    <td class="p-3 text-center flex items-center justify-center space-x-2">
                        <button onclick="editRendaExtra('${item.id}')" title="Editar" class="text-slate-400 hover:text-teal-600 p-1"><i class="fa-solid fa-pen"></i></button>
                        <button onclick="deleteRendaExtra('${item.id}')" title="Excluir" class="text-slate-400 hover:text-rose-600 p-1"><i class="fa-solid fa-trash-can"></i></button>
                    </td>
                `;
                tbody.appendChild(tr);
            });
        }

        // PEQUENOS GASTOS LOGIC
        function handleSaveGasto(e) {
            e.preventDefault();
            const editId = document.getElementById('edit-gasto-id').value;
            const descricao = document.getElementById('input-gasto-desc').value.trim();
            const valor = parseFloat(document.getElementById('input-gasto-valor').value);
            const mesId = document.getElementById('input-gasto-mes').value;
            const categoria = document.getElementById('input-gasto-categoria').value;

            if (!descricao || isNaN(valor) || valor <= 0) return;

            if (editId) {
                const item = pequenosGastosData.find(g => g.id === editId);
                if (item) {
                    item.descricao = descricao;
                    item.valor = valor;
                    item.mesId = mesId;
                    item.categoria = categoria;
                    showToast("Pequeno gasto atualizado!", "success");
                }
            } else {
                pequenosGastosData.unshift({
                    id: 'g_' + Date.now(),
                    descricao,
                    valor,
                    mesId,
                    categoria
                });
                showToast("Pequeno gasto registrado!", "success");
            }

            saveState();
            cancelGastoEdit();
        }

        function editGasto(id) {
            const item = pequenosGastosData.find(g => g.id === id);
            if (!item) return;

            document.getElementById('edit-gasto-id').value = item.id;
            document.getElementById('input-gasto-desc').value = item.descricao;
            document.getElementById('input-gasto-valor').value = item.valor;
            document.getElementById('input-gasto-mes').value = item.mesId;
            document.getElementById('input-gasto-categoria').value = item.categoria || 'Alimentação / Lanches';

            document.getElementById('form-gasto-title').textContent = "Editar Pequeno Gasto";
            document.getElementById('btn-gasto-submit-text').textContent = "Atualizar";
            document.getElementById('btn-cancel-gasto-edit').classList.remove('hidden');

            window.scrollTo({ top: 0, behavior: 'smooth' });
        }

        function cancelGastoEdit() {
            document.getElementById('form-gastos').reset();
            document.getElementById('edit-gasto-id').value = '';
            document.getElementById('form-gasto-title').textContent = "Novo Pequeno Gasto";
            document.getElementById('btn-gasto-submit-text').textContent = "Salvar Gasto";
            document.getElementById('btn-cancel-gasto-edit').classList.add('hidden');
        }

        function deleteGasto(id) {
            if (confirm("Excluir este pequeno gasto?")) {
                pequenosGastosData = pequenosGastosData.filter(g => g.id !== id);
                saveState();
                showToast("Pequeno gasto removido.", "info");
            }
        }

        function renderGastosTable() {
            const tbody = document.getElementById('tbody-gastos');
            const filterMes = document.getElementById('filter-mes-gastos').value;
            const emptyMsg = document.getElementById('empty-gastos');

            tbody.innerHTML = '';
            const filtered = pequenosGastosData.filter(g => filterMes === 'ALL' || g.mesId === filterMes);

            if (filtered.length === 0) {
                emptyMsg.classList.remove('hidden');
                return;
            }
            emptyMsg.classList.add('hidden');

            filtered.forEach(item => {
                const monthObj = monthsData.find(m => m.id === item.mesId);
                const tr = document.createElement('tr');
                tr.className = 'hover:bg-slate-50 dark:hover:bg-slate-700/30 transition';
                tr.innerHTML = `
                    <td class="p-3 font-semibold text-slate-800 dark:text-slate-200">${item.descricao}</td>
                    <td class="p-3 text-slate-500 dark:text-slate-400">${monthObj ? monthObj.label : item.mesId}</td>
                    <td class="p-3 text-slate-500 dark:text-slate-400">${item.categoria || 'Geral'}</td>
                    <td class="p-3 text-right font-bold text-amber-600 dark:text-amber-400">- ${formatBRL(item.valor)}</td>
                    <td class="p-3 text-center flex items-center justify-center space-x-2">
                        <button onclick="editGasto('${item.id}')" title="Editar" class="text-slate-400 hover:text-amber-600 p-1"><i class="fa-solid fa-pen"></i></button>
                        <button onclick="deleteGasto('${item.id}')" title="Excluir" class="text-slate-400 hover:text-rose-600 p-1"><i class="fa-solid fa-trash-can"></i></button>
                    </td>
                `;
                tbody.appendChild(tr);
            });
        }

        // MONTH & FIXED INCOME MANAGEMENT
        function handleAddMonth(e) {
            e.preventDefault();
            const label = document.getElementById('new-month-label').value.trim();
            const year = parseInt(document.getElementById('new-month-year').value);
            const fixedIncome = parseFloat(document.getElementById('new-month-fixed').value) || 0;

            if (!label || isNaN(year)) return;

            const id = 'm_' + Date.now();
            monthsData.push({ id, label, year, fixedIncome });

            saveState();
            document.getElementById('new-month-label').value = '';
            showToast(`Mês "${label}" adicionado!`, "success");
        }

        function deleteMonth(id) {
            const m = monthsData.find(x => x.id === id);
            if (m && confirm(`Tem certeza que deseja excluir o mês "${m.label}"?`)) {
                monthsData = monthsData.filter(x => x.id !== id);
                saveState();
                showToast("Mês removido com sucesso.", "info");
            }
        }

        function updateMonthFixedIncome(id, value) {
            const m = monthsData.find(x => x.id === id);
            if (m) {
                m.fixedIncome = parseFloat(value) || 0;
                saveState();
                showToast(`Renda fixa de ${m.label} atualizada.`, "success");
            }
        }

        function handleUpdateAllFixedIncome(e) {
            e.preventDefault();
            const val = parseFloat(document.getElementById('global-fixed-income').value);
            if (isNaN(val)) return;

            monthsData.forEach(m => m.fixedIncome = val);
            saveState();
            document.getElementById('global-fixed-income').value = '';
            showToast(`Renda fixa de todos os meses alterada para ${formatBRL(val)}!`, "success");
        }

        function renderMonthsConfigTable() {
            const tbody = document.getElementById('tbody-months-config');
            tbody.innerHTML = '';

            monthsData.forEach(m => {
                const tr = document.createElement('tr');
                tr.className = 'hover:bg-slate-50 dark:hover:bg-slate-700/30 transition text-xs';
                tr.innerHTML = `
                    <td class="p-3 font-semibold text-slate-800 dark:text-slate-200">${m.label} (${m.year})</td>
                    <td class="p-3">
                        <input type="number" step="0.01" value="${m.fixedIncome || 0}" onchange="updateMonthFixedIncome('${m.id}', this.value)" class="w-32 p-1.5 text-xs font-bold border rounded bg-white dark:bg-slate-700 text-slate-900 dark:text-white">
                    </td>
                    <td class="p-3 text-center">
                        <button onclick="deleteMonth('${m.id}')" class="text-rose-500 hover:text-rose-700 font-semibold p-1">
                            <i class="fa-solid fa-trash mr-1"></i>Excluir
                        </button>
                    </td>
                `;
                tbody.appendChild(tr);
            });
        }

        // AUTH MODAL LOGIC
        function openAuthModal() {
            document.getElementById('auth-modal').classList.remove('hidden');
        }

        function closeAuthModal() {
            document.getElementById('auth-modal').classList.add('hidden');
        }

        async function handleAuthSubmit(e) {
            e.preventDefault();
            const email = document.getElementById('auth-email').value;
            const password = document.getElementById('auth-password').value;

            const { createUserWithEmailAndPassword, signInWithEmailAndPassword } = window.fbMethods;

            try {
                if (window.authAction === 'signup') {
                    await createUserWithEmailAndPassword(window.fbAuth, email, password);
                    showToast("Conta criada com sucesso!", "success");
                } else {
                    await signInWithEmailAndPassword(window.fbAuth, email, password);
                    showToast("Login efetuado com sucesso!", "success");
                }
                closeAuthModal();
            } catch (err) {
                showToast(err.message || "Erro na autenticação.", "error");
            }
        }

        async function handleLogout() {
            if (window.fbAuth) {
                await window.fbMethods.signOut(window.fbAuth);
                closeAuthModal();
                showToast("Sessão encerrada.", "info");
            }
        }

        function showToast(message, type = "info") {
            const toast = document.getElementById('toast');
            document.getElementById('toast-message').textContent = message;
            const toastIcon = document.getElementById('toast-icon');

            toastIcon.className = type === 'success' 
                ? "fa-solid fa-circle-check text-emerald-400" 
                : type === 'error' ? "fa-solid fa-circle-exclamation text-rose-400" : "fa-solid fa-circle-info text-sky-400";

            toast.classList.remove('translate-y-20', 'opacity-0');
            toast.classList.add('translate-y-0', 'opacity-100');

            setTimeout(() => {
                toast.classList.remove('translate-y-0', 'opacity-100');
                toast.classList.add('translate-y-20', 'opacity-0');
            }, 3000);
        }
    </script>
</body>
</html>