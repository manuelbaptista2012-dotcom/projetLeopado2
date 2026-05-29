[kitanda_web_e_commerce_database.html](https://github.com/user-attachments/files/28373960/kitanda_web_e_commerce_database.html)
# projetLeopado2
projetLeopado2ANGO
<!DOCTYPE html>
<html lang="pt-AO">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Kitanda Web - A sua presença online, hospedada com confiança</title>
    <!-- Tailwind CSS para design ágil e responsivo -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Alpine.js para interações robustas de alta-fidelidade (Modais, Abas, Pesquisa de Domínios) -->
    <script src="https://cdn.jsdelivr.net/npm/alpinejs@3.x.x/dist/cdn.min.js"></script>
    <!-- Fontes Premium do Google (Montserrat e Inter) -->
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&family=Montserrat:wght@400;500;600;700;800&display=swap" rel="stylesheet">
    <!-- FontAwesome para ícones profissionais -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        kitandaRed: '#E30613',     /* Vermelho do logotipo */
                        kitandaYellow: '#FFCC00',  /* Amarelo do logotipo */
                        kitandaDark: '#121212',    /* Preto corporativo */
                        kitandaGray: '#F5F5F7',    /* Cinza claro para contrastes */
                        mcxBlue: '#005CA9',        /* Azul oficial Multicaixa */
                    },
                    fontFamily: {
                        sans: ['Inter', 'sans-serif'],
                        montserrat: ['Montserrat', 'sans-serif'],
                    }
                }
            }
        }
    </script>
    <style>
        [x-cloak] { display: none !important; }
        html {
            scroll-behavior: smooth;
        }
    </style>

    <script>
        // Declarado sincronamente no <head> para garantir que o Alpine encontre o hostingApp sem erros de tempo de execução
        window.hostingApp = function() {
            return {
                billingCycle: 'mensal',
                domainInput: '',
                domainStatus: 'idle',
                searchedDomain: '',
                checkoutOpen: false,
                selectedPlan: null,
                userId: 'A ligar...',
                authStatus: 'loading',
                ordersList: [],
                toast: { show: false, title: '', message: '' },
                checkoutForm: {
                    clientName: '',
                    clientEmail: '',
                    domain: '',
                    paymentMethod: 'multicaixa',
                    expressPhone: ''
                },

                // Inicializar a aplicação e escutar a ativação do Firebase
                init() {
                    this.showToast('Inicialização', 'A ligar à infraestrutura de nuvem...');
                    
                    if (window.FirebaseInstance) {
                        this.initAuth();
                    } else {
                        window.addEventListener('firebase-ready', () => {
                            this.initAuth();
                        });
                    }
                },

                async initAuth() {
                    const { auth, signInAnonymously, signInWithCustomToken } = window.FirebaseInstance;
                    try {
                        let userCredential;
                        if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                            userCredential = await signInWithCustomToken(auth, __initial_auth_token);
                        } else {
                            userCredential = await signInAnonymously(auth);
                        }
                        
                        this.userId = userCredential.user.uid;
                        this.authStatus = 'success';
                        this.showToast('Sessão Ativa', 'Ligado de forma segura à Base de Dados!');
                        this.loadOrders();
                    } catch (error) {
                        console.error("Erro na autenticação:", error);
                        this.authStatus = 'error';
                        this.userId = 'erro-sessao';
                        this.showToast('Erro de Sessão', 'Falha ao autenticar o utilizador.');
                    }
                },

                // Exibe as notificações customizadas no ecrã (Evita alertas no browser)
                showToast(title, message) {
                    this.toast.title = title;
                    this.toast.message = message;
                    this.toast.show = true;
                    setTimeout(() => { this.toast.show = false; }, 4000);
                },

                // Simulação inteligente de verificação de domínios nacionais DNS.AO
                checkDomain() {
                    if (!this.domainInput) return;
                    this.domainStatus = 'searching';
                    this.searchedDomain = this.domainInput;
                    
                    setTimeout(() => {
                        if (!this.searchedDomain.includes('.')) {
                            this.searchedDomain += '.ao';
                        }
                        this.domainStatus = Math.random() > 0.15 ? 'available' : 'taken';
                    }, 800);
                },

                // Abre o checkout e pré-preenche as informações
                openCheckout(planName, price, cycle, prefilledDomain = '') {
                    this.selectedPlan = { name: planName, price: price, cycle: cycle };
                    this.checkoutForm.domain = prefilledDomain;
                    this.checkoutOpen = true;
                },

                // Registar e salvar a encomenda na Base de Dados (Regra 1 de Caminhos Estruturados)
                async submitOrder() {
                    if (this.authStatus !== 'success' || !window.FirebaseInstance) {
                        this.showToast('Erro de Conexão', 'A base de dados ainda não está pronta.');
                        return;
                    }

                    const { db, collection, addDoc, appId } = window.FirebaseInstance;

                    try {
                        const randomReference = Math.floor(100000000 + Math.random() * 900000000).toString();
                        const orderData = {
                            userId: this.userId,
                            planName: this.selectedPlan.name,
                            amount: this.selectedPlan.price,
                            cycle: this.selectedPlan.cycle,
                            clientName: this.checkoutForm.clientName,
                            clientEmail: this.checkoutForm.clientEmail,
                            domain: this.checkoutForm.domain,
                            paymentMethod: this.checkoutForm.paymentMethod,
                            status: 'pendente', // Ativação pendente até simulação de pagamento
                            createdAt: new Date().toISOString(),
                            paymentDetails: {
                                reference: randomReference,
                                entity: '11223',
                                phone: this.checkoutForm.paymentMethod === 'express' ? this.checkoutForm.expressPhone : ''
                            }
                        };

                        // Salvar na coleção pública de encomendas (Regra 1)
                        const ordersPublicCollection = collection(db, 'artifacts', appId, 'public', 'data', 'orders');
                        await addDoc(ordersPublicCollection, orderData);

                        this.showToast('Sucesso', 'Fatura guardada com sucesso na Base de Dados!');
                        this.checkoutOpen = false;
                        
                        // Atualizar a lista local
                        this.loadOrders();
                        
                        // Rolar para a seção de faturas para que o usuário veja
                        this.scrollToSection('pedidos');

                        // Reset formulário
                        this.checkoutForm = {
                            clientName: '',
                            clientEmail: '',
                            domain: '',
                            paymentMethod: 'multicaixa',
                            expressPhone: ''
                        };

                    } catch (error) {
                        console.error("Erro ao guardar encomenda:", error);
                        this.showToast('Erro de Conexão', 'Não foi possível registar o seu pedido.');
                    }
                },

                // Carregar as faturas/pedidos do utilizador atual (Regra 2 - Sem consultas complexas)
                async loadOrders() {
                    if (this.authStatus !== 'success' || !window.FirebaseInstance) return;

                    const { db, collection, getDocs, appId } = window.FirebaseInstance;

                    try {
                        const ordersPublicCollection = collection(db, 'artifacts', appId, 'public', 'data', 'orders');
                        const querySnapshot = await getDocs(ordersPublicCollection);
                        
                        let list = [];
                        querySnapshot.forEach((doc) => {
                            const data = doc.data();
                            // Filtrar em memória conforme instrução da Regra 2 para evitar índices complexos
                            if (data.userId === this.userId) {
                                list.push({ id: doc.id, ...data });
                            }
                        });

                        // Ordenar por data de criação de forma manual na memória
                        list.sort((a, b) => new Date(b.createdAt) - new Date(a.createdAt));
                        this.ordersList = list;
                    } catch (error) {
                        console.error("Erro ao carregar dados:", error);
                        this.showToast('Erro de Base de Dados', 'Falha ao sincronizar as suas faturas.');
                    }
                },

                // Simula a confirmação de recebimento do pagamento local de Angola e ativa os serviços
                async confirmPaymentSimulation(orderId) {
                    if (!window.FirebaseInstance) return;
                    const { db, doc, updateDoc, appId } = window.FirebaseInstance;

                    try {
                        const orderDocRef = doc(db, 'artifacts', appId, 'public', 'data', 'orders', orderId);
                        await updateDoc(orderDocRef, {
                            status: 'pago'
                        });

                        this.showToast('Pagamento Recebido', 'O seu plano foi ativado e o cPanel está pronto!');
                        this.loadOrders();
                    } catch (error) {
                        console.error("Erro ao atualizar pagamento:", error);
                        this.showToast('Falha na Simulação', 'Não foi possível processar o pagamento.');
                    }
                },

                // Permite apagar/cancelar uma encomenda criada no sistema
                async deleteOrder(orderId) {
                    if (!window.FirebaseInstance) return;
                    const { db, doc, deleteDoc, appId } = window.FirebaseInstance;

                    try {
                        const orderDocRef = doc(db, 'artifacts', appId, 'public', 'data', 'orders', orderId);
                        await deleteDoc(orderDocRef);

                        this.showToast('Cancelado', 'A sua encomenda foi removida do sistema.');
                        this.loadOrders();
                    } catch (error) {
                        console.error("Erro ao eliminar:", error);
                    }
                },

                // Formatar valores para Kwanzas
                formatCurrency(value) {
                    if (!value) return "Kz 0,00";
                    return new Intl.NumberFormat('pt-AO', { style: 'currency', currency: 'AOA', minimumFractionDigits: 2 })
                        .format(value)
                        .replace("AOA", "Kz");
                },

                scrollToSection(id) {
                    const el = document.getElementById(id);
                    if (el) el.scrollIntoView({ behavior: 'smooth' });
                }
            };
        };
    </script>
</head>
<body class="bg-gray-50 text-kitandaDark font-sans leading-relaxed overflow-x-hidden" x-data="hostingApp()">

    <!-- BACKGROUND GRADIENT BLOBS -->
    <div class="absolute top-0 right-0 -z-10 w-96 h-96 bg-kitandaRed/5 rounded-full blur-3xl pointer-events-none"></div>
    <div class="absolute top-[600px] left-0 -z-10 w-[500px] h-[500px] bg-kitandaYellow/5 rounded-full blur-3xl pointer-events-none"></div>

    <header class="bg-white/90 backdrop-blur-md sticky top-0 z-40 border-b border-gray-100 transition-all duration-300">
        <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
            <div class="flex items-center justify-between h-20">
                <!-- Logotipo Esquerdo (Recriado com as cores exatas do arquivo 6c8c2e26-2ad8-475c-9cfc-fc12e9f928fd.png) -->
                <a href="#" class="flex items-center space-x-3 group">
                    <div class="relative flex items-center">
                        <div class="bg-white p-2 rounded-lg shadow-sm border border-gray-100">
                            <svg class="w-10 h-10" viewBox="0 0 100 80" fill="none" xmlns="http://www.w3.org/2000/svg">
                                <path d="M70 30C67.5 15 52.5 10 40 15C25 10 15 22.5 15 35C5 38 5 52.5 18 58C18 58 35 58 50 58C68 58 85 58 85 45C85 32.5 78 30 70 30Z" stroke="#E30613" stroke-width="6" stroke-linejoin="round" />
                                <rect x="30" y="32" width="40" height="8" rx="3" fill="#121212" stroke="#FFCC00" stroke-width="2" />
                                <rect x="30" y="44" width="40" height="8" rx="3" fill="#121212" stroke="#E30613" stroke-width="2" />
                                <circle cx="36" cy="36" r="2" fill="#E30613" />
                                <circle cx="36" cy="48" r="2" fill="#FFCC00" />
                                <path d="M50 52V65" stroke="#121212" stroke-width="4" stroke-linecap="round"/>
                                <circle cx="50" cy="68" r="4" fill="#121212"/>
                            </svg>
                        </div>
                        <div class="ml-3 flex flex-col">
                            <div class="flex items-baseline font-montserrat">
                                <span class="text-2xl font-black text-kitandaDark tracking-tight">K<span class="text-kitandaRed">II</span>TANDA</span>
                                <span class="text-2xl font-black text-kitandaYellow tracking-tight ml-0.5">WEB</span>
                            </div>
                            <span class="text-[9px] font-semibold text-gray-500 uppercase tracking-widest leading-none -mt-1">Hospedagem de Confiança</span>
                        </div>
                    </div>
                </a>

                <!-- Navegação Central -->
                <nav class="hidden md:flex space-x-8">
                    <div class="relative" x-data="{ open: false }" @mouseenter="open = true" @mouseleave="open = false">
                        <button class="text-gray-700 hover:text-kitandaRed font-medium flex items-center space-x-1 py-2">
                            <span>Hospedagem</span>
                            <i class="fa-solid fa-chevron-down text-xs transition-transform duration-200" :class="open ? 'rotate-180' : ''"></i>
                        </button>
                        <!-- Dropdown Menu -->
                        <div x-show="open" x-cloak x-transition class="absolute left-0 mt-1 w-60 bg-white rounded-xl shadow-xl border border-gray-100 py-2 z-50">
                            <a href="#planos" class="flex items-center space-x-3 px-4 py-3 hover:bg-gray-50 group">
                                <div class="p-2 bg-red-50 text-kitandaRed rounded-lg"><i class="fa-solid fa-server"></i></div>
                                <div>
                                    <h4 class="font-bold text-sm">Hospedagem Cloud cPanel</h4>
                                    <p class="text-xs text-gray-500">Planos de alto rendimento</p>
                                </div>
                            </a>
                            <a href="#" class="flex items-center space-x-3 px-4 py-3 hover:bg-gray-50 group">
                                <div class="p-2 bg-yellow-50 text-kitandaYellow rounded-lg"><i class="fa-solid fa-envelope"></i></div>
                                <div>
                                    <h4 class="font-bold text-sm">E-mail Profissional</h4>
                                    <p class="text-xs text-gray-500">Credibilidade para marcas</p>
                                </div>
                            </a>
                        </div>
                    </div>
                    <a href="#dominios" class="text-gray-700 hover:text-kitandaRed font-medium py-2">Domínios</a>
                    <a href="#parceiros" class="text-gray-700 hover:text-kitandaRed font-medium py-2">VPS</a>
                    <a href="#pedidos" class="text-gray-700 hover:text-kitandaRed font-medium py-2 relative">
                        Minhas Encomendas
                        <span x-show="ordersList.length > 0" class="absolute -top-1 -right-4 bg-kitandaRed text-white text-[10px] w-4 h-4 rounded-full flex items-center justify-center font-bold" x-text="ordersList.length"></span>
                    </a>
                </nav>

                <!-- Área do Cliente / Contacto -->
                <div class="flex items-center space-x-4">
                    <a href="tel:+244900000000" class="hidden lg:flex items-center space-x-2 text-sm text-gray-600 hover:text-kitandaRed">
                        <span class="bg-gray-100 p-2 rounded-full"><i class="fa-solid fa-phone"></i></span>
                        <span class="font-medium">+244 923 000 000</span>
                    </a>
                    <button @click="scrollToSection('pedidos')" class="bg-kitandaDark text-white hover:bg-kitandaRed hover:scale-105 px-5 py-2.5 rounded-xl font-semibold text-sm transition-all duration-300 shadow-sm flex items-center gap-2">
                        <i class="fa-solid fa-user-shield"></i>
                        <span>Minha Área (<span x-text="authStatus === 'success' ? 'Ligado' : 'A ligar...'"></span>)</span>
                    </button>
                </div>
            </div>
        </div>
    </header>

    <section class="relative bg-gradient-to-br from-kitandaDark via-gray-900 to-black text-white pt-16 pb-24 md:py-32 overflow-hidden">
        <div class="absolute inset-0 z-0 opacity-20">
            <svg class="w-full h-full" xmlns="http://www.w3.org/2000/svg" width="100%" height="100%">
                <defs>
                    <pattern id="grid" width="40" height="40" patternUnits="userSpaceOnUse">
                        <path d="M 40 0 L 0 0 0 40" fill="none" stroke="rgba(255, 255, 255, 0.1)" stroke-width="1"/>
                    </pattern>
                </defs>
                <rect width="100%" height="100%" fill="url(#grid)" />
            </svg>
        </div>

        <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 relative z-10 grid md:grid-cols-12 gap-12 items-center">
            <!-- Conteúdo Textual (Esquerda) -->
            <div class="md:col-span-7 space-y-6">
                <div class="inline-flex items-center space-x-2 bg-white/10 backdrop-blur-md px-3 py-1.5 rounded-full border border-white/10 text-xs sm:text-sm text-kitandaYellow font-semibold">
                    <span class="flex h-2.5 w-2.5 relative">
                        <span class="animate-ping absolute inline-flex h-full w-full rounded-full bg-kitandaYellow opacity-75"></span>
                        <span class="relative inline-flex rounded-full h-2.5 w-2.5 bg-kitandaYellow"></span>
                    </span>
                    <span>Hospedagem em Angola cPanel com Discos SSD</span>
                </div>
                
                <h1 class="text-4xl sm:text-5xl lg:text-6xl font-extrabold font-montserrat tracking-tight leading-tight">
                    Comece o seu site hoje com a <span class="text-transparent bg-clip-text bg-gradient-to-r from-kitandaYellow via-white to-kitandaRed">Kitanda Web!</span>
                </h1>

                <!-- Slogan oficial preservado do ficheiro 6c8c2e26-2ad8-475c-9cfc-fc12e9f928fd.png -->
                <p class="text-lg sm:text-xl text-gray-300 max-w-xl font-light">
                    A sua presença online, <strong class="text-white font-semibold">hospedada com confiança.</strong> Infraestrutura de alto rendimento com suporte 100% local.
                </p>

                <div class="flex flex-col sm:flex-row items-stretch sm:items-center space-y-3 sm:space-y-0 sm:space-x-4 pt-4">
                    <a href="#planos" class="bg-kitandaRed hover:bg-red-700 text-white font-bold px-8 py-4 rounded-xl shadow-lg shadow-kitandaRed/20 hover:shadow-kitandaRed/40 text-center transition-all duration-300 transform hover:-translate-y-1">
                        <i class="fa-solid fa-rocket mr-2"></i>Ver Planos de Hospedagem
                    </a>
                    <a href="#dominios" class="bg-white/10 hover:bg-white/20 border border-white/10 text-white font-semibold px-8 py-4 rounded-xl text-center transition-all duration-300">
                        Pesquisar Domínio .ao
                    </a>
                </div>

                <div class="grid grid-cols-3 gap-6 pt-8 border-t border-white/10 text-center sm:text-left">
                    <div>
                        <span class="block text-2xl sm:text-3xl font-bold text-kitandaYellow font-montserrat">99.9%</span>
                        <span class="text-xs text-gray-400">Uptime Garantido</span>
                    </div>
                    <div>
                        <span class="block text-2xl sm:text-3xl font-bold text-kitandaYellow font-montserrat">SSD</span>
                        <span class="text-xs text-gray-400">Armazenamento Ultraveloz</span>
                    </div>
                    <div>
                        <span class="block text-2xl sm:text-3xl font-bold text-kitandaYellow font-montserrat">24/7</span>
                        <span class="text-xs text-gray-400">Suporte Dedicado</span>
                    </div>
                </div>
            </div>

            <!-- Imagem Destacada (Direita - Baseado em watermarked_img_6979545780246694902.png) -->
            <div class="md:col-span-5 relative">
                <div class="relative z-10 rounded-2xl overflow-hidden border-4 border-white/10 shadow-2xl shadow-black/50 transform hover:scale-[1.02] transition-all duration-500 aspect-video md:aspect-square">
                    <img src="http://googleusercontent.com/image_collection/image_retrieval/17345397940874829456_0" alt="Empreendedora angolana trabalhando em escritório de TI moderno com laptop" class="w-full h-full object-cover">
                    <div class="absolute inset-0 bg-gradient-to-t from-kitandaDark/80 via-transparent to-transparent"></div>
                </div>
                <!-- Detalhes Flutuantes Decorativos em Amarelo/Vermelho -->
                <div class="absolute -top-6 -right-6 w-32 h-32 bg-kitandaYellow rounded-full -z-0 blur-xl opacity-30"></div>
                <div class="absolute -bottom-6 -left-6 w-32 h-32 bg-kitandaRed rounded-full -z-0 blur-xl opacity-30"></div>
                
                <!-- Card de Status Flutuante -->
                <div class="absolute bottom-6 left-6 right-6 bg-white/95 backdrop-blur-md p-4 rounded-xl shadow-lg border border-white/20 text-kitandaDark z-20 flex items-center space-x-3">
                    <div class="bg-emerald-500 text-white p-2.5 rounded-lg text-lg"><i class="fa-solid fa-circle-check"></i></div>
                    <div>
                        <p class="text-xs text-gray-500 font-bold uppercase tracking-wider">Status do Servidor</p>
                        <p class="text-sm font-bold">Todos os sistemas operacionais (Luanda Cloud)</p>
                    </div>
                </div>
            </div>
        </div>
    </section>

    <section id="dominios" class="relative z-20 -mt-12 max-w-5xl mx-auto px-4 sm:px-6">
        <div class="bg-white rounded-2xl shadow-xl p-6 sm:p-8 border border-gray-100">
            <h3 class="text-xl sm:text-2xl font-bold font-montserrat mb-2 text-center">Encontre o seu Domínio de Sucesso</h3>
            <p class="text-sm text-gray-500 mb-6 text-center">Comece a sua presença digital de forma sólida registrando o seu domínio .ao, .com ou .co.ao</p>
            
            <form @submit.prevent="checkDomain()" class="flex flex-col sm:flex-row gap-3">
                <div class="relative flex-grow">
                    <div class="absolute inset-y-0 left-0 pl-4 flex items-center pointer-events-none text-gray-400">
                        <i class="fa-solid fa-globe"></i>
                    </div>
                    <input 
                        type="text" 
                        x-model="domainInput" 
                        placeholder="Escreva o nome do domínio desejado (ex: meuprojetosupremo.ao)" 
                        class="w-full pl-11 pr-4 py-4 rounded-xl border-2 border-gray-100 focus:border-kitandaRed focus:ring-0 focus:outline-none transition-all duration-300 font-medium text-base text-kitandaDark"
                    >
                </div>
                <button 
                    type="submit" 
                    class="bg-kitandaDark text-white hover:bg-kitandaRed font-bold px-8 py-4 rounded-xl transition-all duration-300 flex items-center justify-center space-x-2"
                >
                    <i class="fa-solid fa-magnifying-glass"></i>
                    <span>Pesquisar Domínio</span>
                </button>
            </form>

            <!-- Resultados Interativos em Tempo Real -->
            <div x-show="domainStatus !== 'idle'" x-cloak x-transition class="mt-4 pt-4 border-t border-gray-100">
                <!-- Estado Carregando -->
                <div x-show="domainStatus === 'searching'" class="flex items-center justify-center space-x-3 py-4">
                    <svg class="animate-spin h-6 w-6 text-kitandaRed" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                        <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                        <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
                    </svg>
                    <span class="font-medium text-gray-600">A verificar disponibilidade na DNS.AO...</span>
                </div>

                <!-- Disponível -->
                <div x-show="domainStatus === 'available'" class="bg-green-50 rounded-xl p-4 flex flex-col sm:flex-row items-center justify-between gap-4">
                    <div class="flex items-center space-x-3 text-center sm:text-left">
                        <span class="bg-green-500 text-white p-2 rounded-full"><i class="fa-solid fa-check"></i></span>
                        <div>
                            <p class="font-bold text-green-900 text-lg">Parabéns! <span x-text="searchedDomain" class="underline decoration-green-400 font-mono"></span> está livre!</p>
                            <p class="text-xs text-green-700 font-medium">Pronto para registro nacional com a Kitanda Web.</p>
                        </div>
                    </div>
                    <div class="flex items-center space-x-4">
                        <div class="text-right">
                            <span class="text-sm text-gray-500 line-through">Kz 28.500,00</span>
                            <span class="block text-xl font-black text-green-900 font-montserrat">Kz 24.900,00<span class="text-xs font-normal">/ano</span></span>
                        </div>
                        <button @click="openCheckout('Registo de Domínio: ' + searchedDomain, 24900, 'anual', searchedDomain)" class="bg-green-600 hover:bg-green-700 text-white font-bold px-5 py-3 rounded-lg text-sm transition-colors">
                            Registrar Agora
                        </button>
                    </div>
                </div>

                <!-- Ocupado -->
                <div x-show="domainStatus === 'taken'" class="bg-red-50 rounded-xl p-4 flex flex-col sm:flex-row items-center justify-between gap-4">
                    <div class="flex items-center space-x-3 text-center sm:text-left">
                        <span class="bg-kitandaRed text-white p-2 rounded-full"><i class="fa-solid fa-times"></i></span>
                        <div>
                            <p class="font-bold text-red-900 text-lg">O domínio <span x-text="searchedDomain"></span> já está registrado.</p>
                            <p class="text-xs text-red-700 font-medium">Experimente uma terminação alternativa ou pesquise outro termo.</p>
                        </div>
                    </div>
                    <div class="flex gap-2">
                        <button @click="domainInput = searchedDomain.split('.')[0] + '.com'; checkDomain()" class="bg-white border border-red-200 text-red-900 hover:bg-red-100 font-semibold px-3 py-2 rounded-lg text-xs transition-colors">
                            Verificar .com
                        </button>
                        <button @click="domainInput = searchedDomain.split('.')[0] + '.net'; checkDomain()" class="bg-white border border-red-200 text-red-900 hover:bg-red-100 font-semibold px-3 py-2 rounded-lg text-xs transition-colors">
                            Verificar .net
                        </button>
                    </div>
                </div>
            </div>
        </div>
    </section>

    <section id="planos" class="py-20 bg-gray-50">
        <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
            <div class="text-center max-w-3xl mx-auto mb-16">
                <h2 class="text-3xl sm:text-4xl font-extrabold font-montserrat tracking-tight mb-4">
                    Nossos Planos de Hospedagem em <span class="text-kitandaRed">Kwanzas</span>
                </h2>
                <p class="text-lg text-gray-600">
                    Hospedagem robusta, compatível com os padrões de mercado da AngoWeb. Escolha o plano perfeito para o seu orçamento e escala.
                </p>

                <!-- Seletor de Ciclo de Faturamento (Mensal / Anual) -->
                <div class="inline-flex items-center bg-white border border-gray-200 rounded-full p-1.5 mt-8 shadow-sm">
                    <button 
                        @click="billingCycle = 'mensal'"
                        :class="billingCycle === 'mensal' ? 'bg-kitandaDark text-white shadow-md' : 'text-gray-600 hover:text-black'"
                        class="px-6 py-2.5 rounded-full text-sm font-bold transition-all duration-300 focus:outline-none"
                    >
                        Pagamento Mensal
                    </button>
                    <button 
                        @click="billingCycle = 'anual'"
                        :class="billingCycle === 'anual' ? 'bg-kitandaRed text-white shadow-md' : 'text-gray-600 hover:text-red-600'"
                        class="px-6 py-2.5 rounded-full text-sm font-bold transition-all duration-300 focus:outline-none flex items-center space-x-1"
                    >
                        <span>Faturação Anual</span>
                        <span class="bg-kitandaYellow text-kitandaDark text-[10px] font-black uppercase px-2 py-0.5 rounded-full tracking-wider animate-pulse">Poupa 30%</span>
                    </button>
                </div>
            </div>

            <!-- Grade de Planos de Hospedagem -->
            <div class="grid lg:grid-cols-3 gap-8 items-stretch">
                <!-- PLANO 1: PESSOAL -->
                <div class="bg-white rounded-2xl border-2 border-gray-100 hover:border-kitandaYellow p-8 shadow-sm hover:shadow-xl transition-all duration-300 flex flex-col justify-between transform hover:-translate-y-1 relative">
                    <div>
                        <div class="flex justify-between items-start mb-6">
                            <div>
                                <span class="text-xs font-bold text-gray-400 uppercase tracking-widest">Iniciante / Portfólios</span>
                                <h3 class="text-2xl font-extrabold font-montserrat text-kitandaDark mt-1">Plano Pessoal</h3>
                            </div>
                        </div>

                        <p class="text-gray-500 text-sm mb-6">Otimizado para blogs pessoais, páginas profissionais e uso moderado de tráfego.</p>

                        <!-- Preço Dinâmico (AngoWeb Match) -->
                        <div class="mb-8">
                            <span class="text-xs text-gray-500 block">A partir de</span>
                            <div class="flex items-baseline">
                                <span class="text-4xl font-black font-montserrat text-kitandaDark">
                                    <span x-text="billingCycle === 'mensal' ? 'Kz 5.250,00' : 'Kz 3.675,35'"></span>
                                </span>
                                <span class="text-gray-500 text-sm ml-2">/mês</span>
                            </div>
                            <span x-show="billingCycle === 'anual'" class="text-xs font-semibold text-emerald-600 mt-1 block">Faturado anualmente (Poupa Kz 18.895,80)</span>
                        </div>

                        <!-- Lista de Recursos -->
                        <ul class="space-y-4 border-t border-gray-100 pt-6 mb-8 text-sm">
                            <li class="flex items-center space-x-3">
                                <i class="fa-solid fa-check text-emerald-500"></i>
                                <span>Permite <strong>1 Website</strong></span>
                            </li>
                            <li class="flex items-center space-x-3">
                                <i class="fa-solid fa-check text-emerald-500"></i>
                                <span><strong>10 GB</strong> armazenamento SSD</span>
                            </li>
                            <li class="flex items-center space-x-3">
                                <i class="fa-solid fa-check text-emerald-500"></i>
                                <span>Tráfego Mensal <strong>Ilimitado</strong></span>
                            </li>
                            <li class="flex items-center space-x-3">
                                <i class="fa-solid fa-check text-emerald-500"></i>
                                <span>Certificado <strong>SSL Gratuito</strong></span>
                            </li>
                            <li class="flex items-center space-x-3">
                                <i class="fa-solid fa-check text-emerald-500"></i>
                                <span>Painel de Controle <strong>cPanel</strong></span>
                            </li>
                            <li class="flex items-center space-x-3 text-gray-400 line-through">
                                <i class="fa-solid fa-minus text-gray-300"></i>
                                <span>Domínio .ao Grátis</span>
                            </li>
                        </ul>
                    </div>

                    <button @click="openCheckout('Plano Pessoal', billingCycle === 'mensal' ? 5250 : 44104, billingCycle)" class="w-full bg-kitandaDark text-white hover:bg-kitandaRed font-bold py-4 rounded-xl transition-all duration-300 flex items-center justify-center space-x-2">
                        <span>Adicionar ao Carrinho</span>
                        <i class="fa-solid fa-chevron-right text-xs"></i>
                    </button>
                </div>

                <!-- PLANO 2: ESSENCIAL -->
                <div class="bg-white rounded-2xl border-4 border-kitandaRed p-8 shadow-xl relative flex flex-col justify-between transform hover:-translate-y-1 z-10">
                    <div class="absolute -top-5 left-1/2 -translate-x-1/2 bg-kitandaYellow text-kitandaDark text-xs font-black uppercase px-6 py-2 rounded-full tracking-widest shadow-md">
                        <i class="fa-solid fa-fire mr-1 text-kitandaRed"></i>Mais Solicitado!
                    </div>

                    <div>
                        <div class="flex justify-between items-start mb-6">
                            <div>
                                <span class="text-xs font-bold text-kitandaRed uppercase tracking-widest">Negócios & PME</span>
                                <h3 class="text-2xl font-extrabold font-montserrat text-kitandaDark mt-1">Plano Essencial</h3>
                            </div>
                        </div>

                        <p class="text-gray-500 text-sm mb-6">Ideal para marcas e negócios em crescimento que necessitam de múltiplos canais digitais.</p>

                        <!-- Preço Dinâmico (AngoWeb Match) -->
                        <div class="mb-8">
                            <span class="text-xs text-gray-500 block">A partir de</span>
                            <div class="flex items-baseline">
                                <span class="text-4xl font-black font-montserrat text-kitandaDark">
                                    <span x-text="billingCycle === 'mensal' ? 'Kz 10.501,00' : 'Kz 7.350,70'"></span>
                                </span>
                                <span class="text-gray-500 text-sm ml-2">/mês</span>
                            </div>
                            <span x-show="billingCycle === 'anual'" class="text-xs font-semibold text-emerald-600 mt-1 block">Faturado anualmente (Poupa Kz 37.803,60)</span>
                        </div>

                        <!-- Lista de Recursos -->
                        <ul class="space-y-4 border-t border-gray-100 pt-6 mb-8 text-sm">
                            <li class="flex items-center space-x-3">
                                <i class="fa-solid fa-check text-emerald-500"></i>
                                <span>Crie até <strong>5 Sites</strong></span>
                            </li>
                            <li class="flex items-center space-x-3">
                                <i class="fa-solid fa-check text-emerald-500"></i>
                                <span><strong>50 GB</strong> armazenamento SSD</span>
                            </li>
                            <li class="flex items-center space-x-3">
                                <i class="fa-solid fa-check text-emerald-500"></i>
                                <span>Tráfego Mensal <strong>Ilimitado</strong></span>
                            </li>
                            <li class="flex items-center space-x-3">
                                <i class="fa-solid fa-check text-emerald-500"></i>
                                <span>Certificado <strong>SSL Gratuito</strong></span>
                            </li>
                            <li class="flex items-center space-x-3 text-kitandaRed font-semibold">
                                <i class="fa-solid fa-gift text-kitandaRed"></i>
                                <span><strong>Domínio .AO Gratuito!</strong></span>
                            </li>
                            <li class="flex items-center space-x-3">
                                <i class="fa-solid fa-check text-emerald-500"></i>
                                <span>Painel de Controle <strong>cPanel</strong></span>
                            </li>
                        </ul>
                    </div>

                    <button @click="openCheckout('Plano Essencial', billingCycle === 'mensal' ? 10501 : 88208, billingCycle)" class="w-full bg-kitandaRed hover:bg-red-700 text-white font-bold py-4 rounded-xl shadow-lg shadow-kitandaRed/20 hover:shadow-kitandaRed/40 transition-all duration-300 flex items-center justify-center space-x-2">
                        <span>Adicionar ao Carrinho</span>
                        <i class="fa-solid fa-chevron-right text-xs"></i>
                    </button>
                </div>

                <!-- PLANO 3: PROFISSIONAL -->
                <div class="bg-white rounded-2xl border-2 border-gray-100 hover:border-kitandaRed p-8 shadow-sm hover:shadow-xl transition-all duration-300 flex flex-col justify-between transform hover:-translate-y-1 relative">
                    <div>
                        <div class="flex justify-between items-start mb-6">
                            <div>
                                <span class="text-xs font-bold text-gray-400 uppercase tracking-widest">Empresarial & Lojas Virtuais</span>
                                <h3 class="text-2xl font-extrabold font-montserrat text-kitandaDark mt-1">Plano Profissional</h3>
                            </div>
                        </div>

                        <p class="text-gray-500 text-sm mb-6">Perfeito para plataformas empresariais robustas, blogs de alto tráfego e e-commerce.</p>

                        <!-- Preço Dinâmico (AngoWeb Match) -->
                        <div class="mb-8">
                            <span class="text-xs text-gray-500 block">A partir de</span>
                            <div class="flex items-baseline">
                                <span class="text-4xl font-black font-montserrat text-kitandaDark">
                                    <span x-text="billingCycle === 'mensal' ? 'Kz 15.988,00' : 'Kz 11.191,60'"></span>
                                </span>
                                <span class="text-gray-500 text-sm ml-2">/mês</span>
                            </div>
                            <span x-show="billingCycle === 'anual'" class="text-xs font-semibold text-emerald-600 mt-1 block">Faturado anualmente (Poupa Kz 57.556,80)</span>
                        </div>

                        <!-- Lista de Recursos -->
                        <ul class="space-y-4 border-t border-gray-100 pt-6 mb-8 text-sm">
                            <li class="flex items-center space-x-3">
                                <i class="fa-solid fa-check text-emerald-500"></i>
                                <span>Crie até <strong>50 Sites</strong></span>
                            </li>
                            <li class="flex items-center space-x-3">
                                <i class="fa-solid fa-check text-emerald-500"></i>
                                <span><strong>100 GB</strong> armazenamento SSD</span>
                            </li>
                            <li class="flex items-center space-x-3">
                                <i class="fa-solid fa-check text-emerald-500"></i>
                                <span>Tráfego Mensal <strong>Ilimitado</strong></span>
                            </li>
                            <li class="flex items-center space-x-3">
                                <i class="fa-solid fa-check text-emerald-500"></i>
                                <span>Certificado <strong>SSL Gratuito</strong></span>
                            </li>
                            <li class="flex items-center space-x-3 text-kitandaRed font-semibold">
                                <i class="fa-solid fa-gift text-kitandaRed"></i>
                                <span><strong>Domínio .AO Gratuito!</strong></span>
                            </li>
                            <li class="flex items-center space-x-3">
                                <i class="fa-solid fa-check text-emerald-500"></i>
                                <span>Endereço de <strong class="text-kitandaDark">IP Dedicado</strong></span>
                            </li>
                        </ul>
                    </div>

                    <button @click="openCheckout('Plano Profissional', billingCycle === 'mensal' ? 15988 : 134299, billingCycle)" class="w-full bg-kitandaDark text-white hover:bg-kitandaRed font-bold py-4 rounded-xl transition-all duration-300 flex items-center justify-center space-x-2">
                        <span>Adicionar ao Carrinho</span>
                        <i class="fa-solid fa-chevron-right text-xs"></i>
                    </button>
                </div>
            </div>
        </div>
    </section>

    <section id="pedidos" class="py-20 bg-white border-t border-b border-gray-100">
        <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
            <div class="text-center max-w-3xl mx-auto mb-12">
                <span class="text-xs font-bold uppercase tracking-widest text-kitandaRed">Gestão da sua Presença Digital</span>
                <h2 class="text-3xl font-extrabold font-montserrat tracking-tight mt-2 text-kitandaDark">
                    Painel de Encomendas & Faturas
                </h2>
                <p class="text-sm text-gray-500 mt-2">
                    Acompanhe em tempo real a ativação do seu alojamento de websites armazenado na base de dados Kitanda.
                </p>
            </div>

            <!-- Lista de Faturas / Encomendas da Base de Dados -->
            <div class="bg-gray-50 rounded-2xl p-6 sm:p-8 border border-gray-100 shadow-inner">
                <div class="flex flex-col sm:flex-row items-center justify-between mb-6 gap-4">
                    <div class="flex items-center space-x-3">
                        <div class="bg-kitandaYellow/20 text-kitandaDark p-2.5 rounded-lg text-lg"><i class="fa-solid fa-database"></i></div>
                        <div>
                            <h3 class="font-extrabold text-sm uppercase text-gray-500 tracking-wider">Histórico Seguro de Subscrições</h3>
                            <span class="text-xs text-gray-400 font-mono">UID do Cliente: <span x-text="userId"></span></span>
                        </div>
                    </div>
                    <button @click="loadOrders()" class="bg-white border border-gray-200 hover:bg-gray-100 text-xs font-bold py-2 px-4 rounded-xl transition-all duration-200 flex items-center gap-2">
                        <i class="fa-solid fa-rotate"></i> Autorizar e Atualizar Estado
                    </button>
                </div>

                <!-- Estado Sem Pedidos -->
                <div x-show="ordersList.length === 0" x-cloak class="text-center py-12 bg-white rounded-xl border border-gray-100">
                    <div class="text-gray-300 text-5xl mb-4"><i class="fa-solid fa-folder-open"></i></div>
                    <p class="font-semibold text-gray-600">Nenhum pedido de alojamento ou domínio ativo nesta conta.</p>
                    <p class="text-xs text-gray-400 mt-1">Selecione um plano acima para simular uma transação nacional.</p>
                </div>

                <!-- Lista de Pedidos Ativos -->
                <div x-show="ordersList.length > 0" x-cloak class="space-y-4">
                    <template x-for="order in ordersList" :key="order.id">
                        <div class="bg-white border border-gray-100 rounded-xl p-5 shadow-sm hover:shadow-md transition-all duration-300 grid md:grid-cols-4 items-center gap-4">
                            <!-- Info do Plano -->
                            <div>
                                <span class="text-[10px] text-gray-400 uppercase font-bold tracking-wider">Serviço Adquirido</span>
                                <h4 class="font-bold text-base text-kitandaDark" x-text="order.planName"></h4>
                                <p class="text-xs text-gray-500" x-text="'Domínio: ' + order.domain"></p>
                            </div>
                            <!-- Informações de Fatura -->
                            <div>
                                <span class="text-[10px] text-gray-400 uppercase font-bold tracking-wider">Valor total em Kz</span>
                                <p class="font-extrabold text-kitandaRed font-montserrat" x-text="formatCurrency(order.amount)"></p>
                                <span class="text-[10px] bg-gray-100 text-gray-600 py-0.5 px-2 rounded-full font-mono uppercase" x-text="order.paymentMethod"></span>
                            </div>
                            <!-- Dados de Pagamento -->
                            <div class="text-xs space-y-1">
                                <span class="text-[10px] text-gray-400 uppercase font-bold tracking-wider block">Dados para liquidação</span>
                                <template x-if="order.paymentMethod === 'multicaixa'">
                                    <div>
                                        <p class="text-[11px]">Entidade: <strong class="font-mono">11223</strong></p>
                                        <p class="text-[11px]">Referência: <strong class="font-mono text-kitandaRed" x-text="order.paymentDetails?.reference"></strong></p>
                                    </div>
                                </template>
                                <template x-if="order.paymentMethod === 'express'">
                                    <div>
                                        <p class="text-[11px]">Telemóvel Express: <span class="font-mono font-bold" x-text="order.paymentDetails?.phone"></span></p>
                                        <p class="text-[10px] text-amber-600 animate-pulse font-medium"><i class="fa-solid fa-mobile-screen"></i> Aguardando aprovação no Express</p>
                                    </div>
                                </template>
                                <template x-if="order.paymentMethod === 'transferencia'">
                                    <div>
                                        <p class="text-[10px] font-semibold text-gray-600 truncate">IBAN BAI: AO06.0040.0000.8932.3021.1</p>
                                        <p class="text-[10px] text-gray-400">Enviar comprovativo para suporte</p>
                                    </div>
                                </template>
                            </div>
                            <!-- Status & Ações -->
                            <div class="flex flex-col items-start md:items-end gap-2">
                                <span class="text-[10px] text-gray-400 uppercase font-bold tracking-wider">Estado de Ativação</span>
                                <span :class="{
                                    'bg-amber-100 text-amber-800 border-amber-200': order.status === 'pendente',
                                    'bg-emerald-100 text-emerald-800 border-emerald-200': order.status === 'pago'
                                }" class="text-xs font-bold py-1 px-3 rounded-full border" x-text="order.status === 'pendente' ? 'Pendente de Pagamento' : 'Ativo / Servidor Pronto'"></span>
                                
                                <div class="flex items-center gap-2 mt-1">
                                    <template x-if="order.status === 'pendente'">
                                        <button @click="confirmPaymentSimulation(order.id)" class="text-[10px] bg-green-600 hover:bg-green-700 text-white font-bold py-1 px-2.5 rounded-md transition-colors shadow-sm">
                                            <i class="fa-solid fa-check-double"></i> Simular Pagamento
                                        </button>
                                    </template>
                                    <button @click="deleteOrder(order.id)" class="text-[10px] text-gray-400 hover:text-red-600 transition-colors">
                                        <i class="fa-solid fa-trash-can"></i> Cancelar
                                    </button>
                                </div>
                            </div>
                        </div>
                    </template>
                </div>
            </div>
        </div>
    </section>

    <section class="py-20 bg-gray-50 border-t border-gray-100">
        <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
            <div class="text-center max-w-3xl mx-auto mb-16">
                <span class="text-xs font-bold uppercase tracking-widest text-kitandaRed">A nossa proposta de valor</span>
                <h2 class="text-3xl sm:text-4xl font-extrabold font-montserrat tracking-tight mt-2">
                    Por que hospedar com a Kitanda Web?
                </h2>
                <p class="text-lg text-gray-600 mt-4">
                    Unimos tecnologia em nuvem global de última geração com suporte nativo focado em Angola.
                </p>
            </div>

            <div class="grid md:grid-cols-3 gap-8 text-center sm:text-left">
                <!-- Diferencial 1 -->
                <div class="p-8 rounded-2xl bg-white border border-gray-100 hover:shadow-lg transition-all duration-300">
                    <div class="w-14 h-14 bg-red-100 text-kitandaRed rounded-2xl flex items-center justify-center text-2xl mb-6 mx-auto sm:mx-0">
                        <i class="fa-solid fa-gauge-high"></i>
                    </div>
                    <h3 class="text-xl font-extrabold font-montserrat mb-3 text-kitandaDark">Velocidade Extrema</h3>
                    <p class="text-gray-500 text-sm leading-relaxed">
                        Infraestrutura baseada em nuvem SSD e aceleração LiteSpeed Web Server, garantindo que o seu website em Angola carregue em menos de 1 segundo.
                    </p>
                </div>

                <!-- Diferencial 2 -->
                <div class="p-8 rounded-2xl bg-white border border-gray-100 hover:shadow-lg transition-all duration-300">
                    <div class="w-14 h-14 bg-yellow-100 text-kitandaYellow rounded-2xl flex items-center justify-center text-2xl mb-6 mx-auto sm:mx-0">
                        <i class="fa-solid fa-shield-halved"></i>
                    </div>
                    <h3 class="text-xl font-extrabold font-montserrat mb-3 text-kitandaDark">Confiabilidade e Segurança</h3>
                    <p class="text-gray-500 text-sm leading-relaxed">
                        Contamos com blindagem de segurança Imunify360 contra ataques DDoS, malware ativo, isolamento de recursos cloud e backups automatizados diários.
                    </p>
                </div>

                <!-- Diferencial 3 -->
                <div class="p-8 rounded-2xl bg-white border border-gray-100 hover:shadow-lg transition-all duration-300">
                    <div class="w-14 h-14 bg-gray-900 text-white rounded-2xl flex items-center justify-center text-2xl mb-6 mx-auto sm:mx-0">
                        <i class="fa-solid fa-headset"></i>
                    </div>
                    <h3 class="text-xl font-extrabold font-montserrat mb-3 text-kitandaDark">Suporte Local Real</h3>
                    <p class="text-gray-500 text-sm leading-relaxed">
                        Sem barreiras de idioma ou fusos horários distantes. Equipa técnica presente em Angola, pronta para atender o seu negócio 24 horas por dia por telefone ou ticket.
                    </p>
                </div>
            </div>
        </div>
    </section>

    <footer class="bg-kitandaDark text-gray-400 pt-16 pb-12 border-t-4 border-kitandaRed">
        <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 grid md:grid-cols-4 gap-12 mb-12">
            <!-- Coluna 1: Sobre e Logo -->
            <div class="space-y-4">
                <div class="flex items-baseline font-montserrat text-white text-xl font-black">
                    <span>K<span class="text-kitandaRed">II</span>TANDA</span>
                    <span class="text-kitandaYellow ml-0.5">WEB</span>
                </div>
                <p class="text-xs text-gray-500 leading-relaxed">
                    Presença online de alto desempenho hospedada com segurança e integridade de dados absoluta. Sediados em Luanda, servindo soluções web para Angola e o mundo.
                </p>
                <!-- Redes sociais -->
                <div class="flex space-x-3">
                    <a href="#" class="w-9 h-9 bg-white/10 hover:bg-kitandaRed rounded-full flex items-center justify-center text-white transition-colors"><i class="fa-brands fa-facebook-f text-sm"></i></a>
                    <a href="#" class="w-9 h-9 bg-white/10 hover:bg-kitandaRed rounded-full flex items-center justify-center text-white transition-colors"><i class="fa-brands fa-linkedin-in text-sm"></i></a>
                    <a href="#" class="w-9 h-9 bg-white/10 hover:bg-kitandaRed rounded-full flex items-center justify-center text-white transition-colors"><i class="fa-brands fa-instagram text-sm"></i></a>
                </div>
            </div>

            <!-- Coluna 2: Serviços -->
            <div>
                <h4 class="text-white font-bold font-montserrat text-sm uppercase tracking-wider mb-4">Serviços</h4>
                <ul class="space-y-2 text-sm">
                    <li><a href="#planos" class="hover:text-white transition-colors">Alojamento de Sites</a></li>
                    <li><a href="#dominios" class="hover:text-white transition-colors">Registro de Domínio .ao</a></li>
                    <li><a href="#" class="hover:text-white transition-colors">E-mail Profissional Corporativo</a></li>
                    <li><a href="#" class="hover:text-white transition-colors">Servidores Virtuais VPS</a></li>
                </ul>
            </div>

            <!-- Coluna 3: Contacto -->
            <div>
                <h4 class="text-white font-bold font-montserrat text-sm uppercase tracking-wider mb-4">Contacto</h4>
                <ul class="space-y-3 text-sm">
                    <li class="flex items-center space-x-2">
                        <i class="fa-solid fa-phone text-kitandaYellow"></i>
                        <span>+244 923 000 000</span>
                    </li>
                    <li class="flex items-center space-x-2">
                        <i class="fa-solid fa-envelope text-kitandaRed"></i>
                        <span>suporte@kitandaweb.ao</span>
                    </li>
                    <li class="flex items-start space-x-2">
                        <i class="fa-solid fa-location-dot text-kitandaYellow mt-1"></i>
                        <span>Av. Revolução de Outubro, Luanda, Angola</span>
                    </li>
                </ul>
            </div>

            <!-- Coluna 4: Métodos de Pagamento Oficiais de Angola -->
            <div>
                <h4 class="text-white font-bold font-montserrat text-sm uppercase tracking-wider mb-4">Pagamentos Aceites</h4>
                <div class="p-4 bg-white/5 rounded-xl border border-white/5 space-y-4">
                    <p class="text-xs text-gray-300 leading-relaxed">Aceitamos pagamentos instantâneos locais em Kwanzas.</p>
                    
                    <!-- Emblemas de Métodos Angolanos com MULTICAIXA EXPRESS destacado -->
                    <div class="flex flex-wrap items-center gap-3">
                        <!-- Símbolo Multicaixa Express (MCX) -->
                        <div class="bg-white px-2 py-1.5 rounded-lg flex items-center justify-center border border-gray-100 shadow-sm select-none" title="Multicaixa Express">
                            <svg class="h-6 w-auto" viewBox="0 0 120 40" fill="none" xmlns="http://www.w3.org/2000/svg">
                                <rect width="120" height="40" rx="6" fill="#005CA9"/>
                                <!-- Representação da Seta Dupla MCX em Laranja e Branco -->
                                <path d="M15 20L25 10V16H35V24H25V30L15 20Z" fill="#F47920"/>
                                <path d="M45 20L35 12V17H18V23H35V28L45 20Z" fill="#FFFFFF"/>
                                <!-- Texto MCX Express -->
                                <text x="50" y="24" fill="#FFFFFF" font-family="Montserrat" font-size="11" font-weight="900">EXPRESS</text>
                            </svg>
                        </div>
                        
                        <!-- Logotipo da Referência Multicaixa Tradicional -->
                        <div class="bg-gray-100 p-1.5 rounded-lg flex items-center justify-center border border-gray-300 text-kitandaDark text-[10px] font-bold" title="Referência Multicaixa">
                            <i class="fa-solid fa-money-bill-transfer text-sky-700 mr-1"></i> Multicaixa
                        </div>

                        <!-- IBAN/Transferência -->
                        <div class="bg-gray-100 p-1.5 rounded-lg flex items-center justify-center border border-gray-300 text-kitandaDark text-[10px] font-bold" title="Transferência Bancária">
                            <i class="fa-solid fa-building-columns text-emerald-700 mr-1"></i> IBAN
                        </div>
                    </div>

                    <div class="flex items-center space-x-2 text-emerald-400 text-xs font-semibold">
                        <span class="flex h-2 w-2 relative">
                            <span class="animate-ping absolute inline-flex h-full w-full rounded-full bg-emerald-400 opacity-75"></span>
                            <span class="relative inline-flex rounded-full h-2 w-2 bg-emerald-400"></span>
                        </span>
                        <span>Faturação Segura por Referência</span>
                    </div>
                </div>
            </div>
        </div>

        <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 border-t border-white/10 pt-8 flex flex-col sm:flex-row items-center justify-between gap-4 text-xs">
            <p>© 2026 Kitanda Web. Slogan: "A sua presença online, hospedada com confiança." Todos os direitos reservados.</p>
            <div class="flex space-x-4">
                <a href="#" class="hover:text-white transition-colors">Política de Privacidade</a>
                <span>|</span>
                <a href="#" class="hover:text-white transition-colors">Termos de Serviço</a>
            </div>
        </div>
    </footer>

    <div x-show="checkoutOpen" x-cloak class="fixed inset-0 z-50 overflow-y-auto flex items-center justify-center p-4">
        <div class="fixed inset-0 bg-kitandaDark/80 backdrop-blur-sm transition-opacity" @click="checkoutOpen = false"></div>

        <div class="bg-white rounded-2xl max-w-lg w-full p-6 sm:p-8 relative z-10 shadow-2xl border border-gray-100 transform transition-all max-h-[90vh] overflow-y-auto">
            <button @click="checkoutOpen = false" class="absolute top-4 right-4 text-gray-400 hover:text-kitandaDark text-xl"><i class="fa-solid fa-xmark"></i></button>
            
            <div class="flex items-center space-x-3 mb-6">
                <div class="bg-red-50 text-kitandaRed p-3 rounded-xl text-xl"><i class="fa-solid fa-cart-shopping"></i></div>
                <div>
                    <h3 class="font-extrabold text-xl font-montserrat">Checkout Kitanda Web</h3>
                    <p class="text-xs text-gray-500">Adquira o seu alojamento em poucos segundos.</p>
                </div>
            </div>

            <!-- Passo 1: Seleção / Resumo do Pedido -->
            <div class="bg-gray-50 p-4 rounded-xl border border-gray-100 mb-6">
                <p class="text-xs font-bold text-gray-400 uppercase tracking-widest">Serviço Selecionado</p>
                <div class="flex justify-between items-center mt-1">
                    <div>
                        <span class="font-bold text-kitandaDark text-base" x-text="selectedPlan?.name"></span>
                        <span class="block text-[10px] text-gray-400" x-text="'Ciclo de Faturação: ' + selectedPlan?.cycle"></span>
                    </div>
                    <span class="font-black text-kitandaRed text-lg font-montserrat" x-text="formatCurrency(selectedPlan?.price)"></span>
                </div>
            </div>

            <!-- Formulário do E-commerce integrado à Base de Dados -->
            <form @submit.prevent="submitOrder()" class="space-y-4">
                <!-- Informações do Cliente -->
                <div>
                    <label class="block text-xs font-bold text-gray-500 uppercase tracking-wider mb-1.5">Seu Nome Completo</label>
                    <input type="text" x-model="checkoutForm.clientName" required placeholder="Ex: Hamilton de Almeida" class="w-full p-3.5 rounded-xl border border-gray-200 focus:border-kitandaRed focus:outline-none text-sm text-kitandaDark bg-gray-50">
                </div>
                <div>
                    <label class="block text-xs font-bold text-gray-500 uppercase tracking-wider mb-1.5">Endereço de E-mail</label>
                    <input type="email" x-model="checkoutForm.clientEmail" required placeholder="exemplo@empresa.ao" class="w-full p-3.5 rounded-xl border border-gray-200 focus:border-kitandaRed focus:outline-none text-sm text-kitandaDark bg-gray-50">
                </div>
                <div>
                    <label class="block text-xs font-bold text-gray-500 uppercase tracking-wider mb-1.5">Domínio a Associar</label>
                    <input type="text" x-model="checkoutForm.domain" required placeholder="exemplo.ao ou exemplo.com" class="w-full p-3.5 rounded-xl border border-gray-200 focus:border-kitandaRed focus:outline-none text-sm text-kitandaDark bg-gray-50 font-mono">
                </div>

                <!-- Escolha do Método de Pagamento Angolano -->
                <div>
                    <label class="block text-xs font-bold text-gray-500 uppercase tracking-wider mb-2">Método de Liquidação</label>
                    <div class="grid grid-cols-1 gap-2">
                        <!-- Referência Multicaixa -->
                        <label class="border-2 rounded-xl p-3 flex items-center justify-between cursor-pointer transition-all" :class="checkoutForm.paymentMethod === 'multicaixa' ? 'border-kitandaRed bg-red-50/20' : 'border-gray-100 hover:border-gray-200'">
                            <div class="flex items-center space-x-3">
                                <input type="radio" value="multicaixa" x-model="checkoutForm.paymentMethod" class="text-kitandaRed focus:ring-kitandaRed">
                                <div>
                                    <span class="block text-sm font-bold text-kitandaDark">Referência Multicaixa (SIM)</span>
                                    <span class="text-[10px] text-gray-500">Pague no Caixa Eletrónico ou Internet Banking.</span>
                                </div>
                            </div>
                            <span class="text-xs bg-gray-100 text-gray-600 px-2 py-1 rounded font-bold">Kz</span>
                        </label>

                        <!-- Multicaixa Express -->
                        <label class="border-2 rounded-xl p-3 flex items-center justify-between cursor-pointer transition-all" :class="checkoutForm.paymentMethod === 'express' ? 'border-mcxBlue bg-blue-50/20' : 'border-gray-100 hover:border-gray-200'">
                            <div class="flex items-center space-x-3">
                                <input type="radio" value="express" x-model="checkoutForm.paymentMethod" class="text-mcxBlue focus:ring-mcxBlue">
                                <div>
                                    <span class="block text-sm font-bold text-kitandaDark flex items-center gap-1.5">
                                        Multicaixa Express <span class="bg-mcxBlue text-white text-[9px] px-1.5 py-0.2 rounded font-black">EXPRESS</span>
                                    </span>
                                    <span class="text-[10px] text-gray-500">Receba a notificação direta no seu telemóvel associado.</span>
                                </div>
                            </div>
                        </label>

                        <!-- Campo extra para telefone Multicaixa Express -->
                        <div x-show="checkoutForm.paymentMethod === 'express'" x-cloak class="p-3 bg-blue-50/30 rounded-xl border border-blue-100 space-y-2">
                            <label class="block text-[10px] font-bold text-gray-500 uppercase">Telemóvel do Express (Angola)</label>
                            <input type="tel" x-model="checkoutForm.expressPhone" placeholder="Ex: 923000000" class="w-full p-2.5 rounded-lg border border-gray-200 text-xs text-kitandaDark">
                        </div>

                        <!-- Transferência Bancária -->
                        <label class="border-2 rounded-xl p-3 flex items-center justify-between cursor-pointer transition-all" :class="checkoutForm.paymentMethod === 'transferencia' ? 'border-kitandaYellow bg-yellow-50/20' : 'border-gray-100 hover:border-gray-200'">
                            <div class="flex items-center space-x-3">
                                <input type="radio" value="transferencia" x-model="checkoutForm.paymentMethod" class="text-kitandaYellow focus:ring-kitandaYellow">
                                <div>
                                    <span class="block text-sm font-bold text-kitandaDark">Transferência Bancária (IBAN BAI)</span>
                                    <span class="text-[10px] text-gray-500">Ativação após envio do comprovativo via e-mail.</span>
                                </div>
                            </div>
                            <span class="text-xs bg-gray-100 text-gray-600 px-2 py-1 rounded font-bold"><i class="fa-solid fa-bank"></i></span>
                        </label>
                    </div>
                </div>

                <!-- Ação de Finalizar e Enviar para Firestore -->
                <div class="pt-2">
                    <button type="submit" class="w-full bg-kitandaRed hover:bg-red-700 text-white font-bold py-4 rounded-xl text-center shadow-lg shadow-kitandaRed/20 transition-all duration-300 flex items-center justify-center space-x-2">
                        <i class="fa-solid fa-lock"></i>
                        <span>Gerar Encomenda & Pagamento</span>
                    </button>
                    <p class="text-[10px] text-gray-400 text-center mt-2.5">O processamento inicial cria o registo seguro da transação em base de dados e gera o talão para pagamento.</p>
                </div>
            </form>
        </div>
    </div>

    <!-- Mensagem Flutuante de Notificação Personalizada -->
    <div x-show="toast.show" x-cloak class="fixed bottom-6 right-6 z-50 bg-kitandaDark text-white p-4 rounded-xl shadow-2xl border border-gray-800 flex items-center space-x-3 max-w-sm" x-transition>
        <div class="bg-kitandaYellow text-kitandaDark rounded-lg p-2"><i class="fa-solid fa-bell"></i></div>
        <div>
            <h4 class="font-bold text-xs" x-text="toast.title"></h4>
            <p class="text-[11px] text-gray-300" x-text="toast.message"></p>
        </div>
    </div>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, setDoc, getDoc, addDoc, updateDoc, deleteDoc, collection, getDocs } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // Inicialização Segura das Variáveis de Contexto Firebase
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'kitanda-web-prod';
        
        // Configuração de Fallback Segura para Ambientes de Testes Locais
        const defaultFirebaseConfig = {
            apiKey: "mock-api-key-kitanda",
            authDomain: "kitanda-web-app.firebaseapp.com",
            projectId: "kitanda-web-app",
            storageBucket: "kitanda-web-app.appspot.com",
            messagingSenderId: "123456789",
            appId: "1:123456789:web:abcdef"
        };
        const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : defaultFirebaseConfig;

        // Inicializar os Serviços do Firebase
        const app = initializeApp(firebaseConfig);
        const db = getFirestore(app);
        const auth = getAuth(app);

        // Exportação global segura das dependências para os controladores síncronos da página
        window.FirebaseInstance = {
            appId,
            db,
            auth,
            signInAnonymously,
            signInWithCustomToken,
            doc,
            addDoc,
            updateDoc,
            deleteDoc,
            collection,
            getDocs
        };

        // Avisar ao Alpine que os serviços do Firebase estão totalmente configurados no window
        window.dispatchEvent(new CustomEvent('firebase-ready'));
    </script>
</body>
</html>
