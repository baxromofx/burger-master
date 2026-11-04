
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Burger Master - Игра и Заказ Бургеров</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- FontAwesome icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Google Fonts -->
    <link href="https://fonts.googleapis.com/css2?family=Fredoka+One&family=Nunito:wght@600;700;800;900&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Nunito', sans-serif;
            user-select: none;
            background: #111827;
        }
        .font-game {
            font-family: 'Fredoka One', cursive;
        }
        /* Custom game styling */
        .bubble-tail::after {
            content: '';
            position: absolute;
            bottom: -12px;
            left: 50%;
            transform: translateX(-50%);
            border-width: 12px 10px 0;
            border-style: solid;
            border-color: #ffffff transparent;
            display: block;
            width: 0;
        }
        .wood-texture {
            background: #8b4513;
            background-image: repeating-linear-gradient(45deg, rgba(255,255,255,0.05) 0px, rgba(255,255,255,0.05) 2px, transparent 2px, transparent 4px);
        }
        .metal-grill {
            background: linear-gradient(180deg, #374151 0%, #1f2937 100%);
            border-top: 4px solid #4b5563;
        }
        /* Ingredient layering on burger build */
        .burger-stack {
            display: flex;
            flex-direction: column-reverse;
            align-items: center;
            justify-content: flex-end;
        }
        @keyframes sizzle {
            0% { transform: translateY(0px) scale(1); }
            50% { transform: translateY(-1px) scale(1.02); }
            100% { transform: translateY(0px) scale(1); }
        }
        .sizzling {
            animation: sizzle 0.3s infinite ease-in-out;
        }
        @keyframes popIn {
            0% { transform: scale(0.5); opacity: 0; }
            80% { transform: scale(1.1); }
            100% { transform: scale(1); opacity: 1; }
        }
        .pop-in {
            animation: popIn 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275) forwards;
        }
        /* Custom Scrollbars */
        ::-webkit-scrollbar {
            width: 8px;
        }
        ::-webkit-scrollbar-track {
            background: #1f2937;
        }
        ::-webkit-scrollbar-thumb {
            background: #f59e0b;
            border-radius: 4px;
        }
    </style>
</head>
<body class="text-white min-h-screen flex flex-col justify-between overflow-x-hidden">

    <!-- GAME TOP HUD / HEADER -->
    <header class="sticky top-0 z-50 bg-slate-900 border-b-4 border-amber-500 shadow-2xl px-4 py-2">
        <div class="max-w-7xl mx-auto flex flex-wrap justify-between items-center gap-3">
            
            <!-- Logo & Brand -->
            <div class="flex items-center space-x-3">
                <div class="bg-amber-500 p-2 rounded-2xl shadow-lg transform -rotate-6">
                    <i class="fa-solid fa-hamburger text-3xl text-slate-900"></i>
                </div>
                <div>
                    <h1 class="font-game text-2xl text-amber-400 tracking-wide leading-none">БУРГЕР МАСТЕР</h1>
                    <span class="text-xs text-slate-400 font-bold tracking-wider">SHEF RESTAURANT</span>
                </div>
            </div>

            <!-- Game Stats Bar inspired by Arcade Games -->
            <div class="flex items-center space-x-3 bg-slate-800/90 border-2 border-slate-700 px-4 py-1.5 rounded-full shadow-inner">
[04.11.2026 13:40] Shohrux: <!-- Level Badge -->
                <div class="flex items-center space-x-1">
                    <span class="bg-gradient-to-r from-blue-600 to-indigo-600 font-game text-xs px-2.5 py-1 rounded-full text-white shadow">
                        УР. <span id="level-val">1</span>
                    </span>
                </div>

                <!-- XP Bar -->
                <div class="w-24 md:w-32 bg-slate-700 rounded-full h-4 overflow-hidden border border-slate-600 relative">
                    <div id="xp-bar" class="bg-gradient-to-r from-emerald-500 to-green-400 h-full w-1/4 transition-all duration-300"></div>
                    <span class="absolute inset-0 text-[10px] flex items-center justify-center font-bold text-white drop-shadow" id="xp-text">25/100</span>
                </div>

                <!-- Coins -->
                <div class="flex items-center space-x-1 bg-amber-500/20 px-3 py-0.5 rounded-full border border-amber-500/40">
                    <i class="fa-solid fa-coins text-amber-400 text-sm animate-bounce"></i>
                    <span id="coins-val" class="font-game text-amber-300 text-sm">150</span>
                </div>

                <!-- Stars -->
                <div class="flex items-center space-x-1 bg-yellow-500/20 px-2 py-0.5 rounded-full border border-yellow-500/40">
                    <i class="fa-solid fa-star text-yellow-400 text-sm"></i>
                    <span id="stars-val" class="font-game text-yellow-300 text-sm">5</span>
                </div>
            </div>

            <!-- Navigation & Cart -->
            <div class="flex items-center space-x-3">
                <button onclick="switchTab('game')" id="btn-tab-game" class="px-4 py-2 rounded-xl font-bold bg-amber-500 text-slate-900 shadow-md hover:bg-amber-400 transition flex items-center gap-2">
                    <i class="fa-solid fa-gamepad"></i> Кухня
                </button>
                <button onclick="switchTab('menu')" id="btn-tab-menu" class="px-4 py-2 rounded-xl font-bold bg-slate-800 text-slate-200 border border-slate-700 hover:bg-slate-700 transition flex items-center gap-2">
                    <i class="fa-solid fa-utensils"></i> Меню & Заказ
                </button>
                
                <!-- Cart Button -->
                <button onclick="toggleCartModal()" class="relative p-2.5 bg-indigo-600 rounded-xl text-white shadow-lg hover:bg-indigo-500 transition">
                    <i class="fa-solid fa-shopping-cart text-lg"></i>
                    <span id="cart-count" class="absolute -top-2 -right-2 bg-red-500 text-white text-xs font-bold rounded-full h-5 w-5 flex items-center justify-center border-2 border-slate-900">0</span>
                </button>
            </div>
        </div>
    </header>

    <!-- MAIN VIEW AREA -->
    <main class="flex-grow container mx-auto px-2 py-4 max-w-6xl">

        <!-- ==================== GAME / SIMULATOR SECTION ==================== -->
        <section id="section-game" class="block">
            <!-- Restaurant Background Frame -->
            <div class="relative bg-slate-800 border-4 border-slate-700 rounded-3xl shadow-2xl overflow-hidden flex flex-col min-h-[560px]">
                
                <!-- Top Dining Area & Customers -->
                <div class="relative h-64 bg-gradient-to-b from-sky-900 via-indigo-950 to-slate-900 px-6 pt-4 flex flex-col justify-between overflow-hidden">
                    <!-- Restaurant Decor / Window View -->
                    <div class="absolute inset-0 opacity-20 bg-[radial-gradient(#e0e7ff_1px,transparent_1px)] [background-size:16px_16px]"></div>
                    
                    <!-- Customer Speech Bubbles & Avatars -->
                    <div id="customers-container" class="relative z-10 flex justify-around items-end h-full pb-2">
                        <!-- Customers dynamically inserted here -->
                    </div>
[04.11.2026 13:40] Shohrux: <!-- Dining Counter Divider -->
                    <div class="w-full h-4 bg-amber-900 border-t-2 border-amber-700 shadow-md z-20"></div>
                </div>

                <!-- Kitchen Counter Assembly Area -->
                <div class="flex-1 wood-texture p-4 flex flex-col justify-between border-t-4 border-amber-950 relative">
                    
                    <!-- Kitchen Equipment Layout -->
                    <div class="grid grid-cols-1 md:grid-cols-3 gap-4 h-full">
                        
                        <!-- LEFT: GRILL STATION -->
                        <div class="metal-grill rounded-2xl p-3 flex flex-col justify-between shadow-inner border border-slate-600">
                            <div class="flex justify-between items-center mb-1">
                                <span class="font-game text-xs text-amber-400 tracking-wider flex items-center gap-1">
                                    <i class="fa-solid fa-fire text-red-500"></i> ЖАРКА КОТЛЕТ
                                </span>
                                <span class="text-[10px] text-slate-400">Нажмите на сырую котлету</span>
                            </div>

                            <!-- Grill Slots -->
                            <div class="grid grid-cols-2 gap-3 my-auto">
                                <div id="grill-0" onclick="cookPatty(0)" class="h-20 bg-slate-900/80 rounded-xl border-2 border-dashed border-slate-700 flex items-center justify-center cursor-pointer hover:border-amber-500 transition relative">
                                    <span class="text-xs text-slate-600 font-bold">Гриль 1</span>
                                </div>
                                <div id="grill-1" onclick="cookPatty(1)" class="h-20 bg-slate-900/80 rounded-xl border-2 border-dashed border-slate-700 flex items-center justify-center cursor-pointer hover:border-amber-500 transition relative">
                                    <span class="text-xs text-slate-600 font-bold">Гриль 2</span>
                                </div>
                            </div>

                            <!-- Raw Patty Bin Button -->
                            <button onclick="startGrilling()" class="w-full bg-red-800 hover:bg-red-700 border-2 border-red-600 py-2 rounded-xl text-xs font-bold shadow flex items-center justify-center gap-2 transition active:scale-95">
                                <i class="fa-solid fa-circle text-amber-900"></i> Положить котлету на гриль
                            </button>
                        </div>

                        <!-- CENTER: BURGER ASSEMBLY TABLE -->
                        <div class="bg-slate-900/90 rounded-2xl p-3 flex flex-col justify-between border-2 border-amber-600/40 shadow-xl">
                            <div class="flex justify-between items-center">
                                <span class="font-game text-xs text-amber-400 tracking-wider flex items-center gap-1">
                                    <i class="fa-solid fa-layer-group text-amber-500"></i> СБОРКА БУРГЕРА
                                </span>
                                <button onclick="clearCurrentBurger()" class="text-[10px] text-red-400 hover:underline">Сброс</button>
                            </div>

                            <!-- Plate / Burger Stack Preview -->
                            <div class="h-44 my-2 bg-slate-950/60 rounded-xl border border-slate-800 flex flex-col justify-end items-center p-2 relative overflow-hidden">
                                <!-- Plate Graphic -->
                                <div class="absolute bottom-1 w-36 h-4 bg-slate-300 rounded-full border-2 border-slate-400 shadow-lg"></div>
                                
                                <!-- Active Burger Stack -->
[04.11.2026 13:40] Shohrux: <div id="burger-plate" class="burger-stack z-10 mb-2 transition-all">
                                    <span class="text-slate-600 text-xs my-auto">Тарелка пуста</span>
                                </div>
                            </div>

                            <!-- Ingredient Buttons -->
                            <div class="grid grid-cols-4 gap-1.5">
                                <button onclick="addIngredient('bottomBun')" class="bg-amber-700 hover:bg-amber-600 p-1.5 rounded-lg text-[11px] font-bold text-center truncate shadow border border-amber-600">
                                    🍞 Низ
                                </button>
                                <button onclick="addIngredient('patty')" class="bg-amber-900 hover:bg-amber-800 p-1.5 rounded-lg text-[11px] font-bold text-center truncate shadow border border-amber-950">
                                    🥩 Котлета
                                </button>
                                <button onclick="addIngredient('cheese')" class="bg-yellow-600 hover:bg-yellow-500 p-1.5 rounded-lg text-[11px] font-bold text-center truncate shadow border border-yellow-400">
                                    🧀 Сыр
                                </button>
                                <button onclick="addIngredient('lettuce')" class="bg-emerald-700 hover:bg-emerald-600 p-1.5 rounded-lg text-[11px] font-bold text-center truncate shadow border border-emerald-500">
                                    🥬 Салат
                                </button>
                                <button onclick="addIngredient('tomato')" class="bg-red-600 hover:bg-red-500 p-1.5 rounded-lg text-[11px] font-bold text-center truncate shadow border border-red-400">
                                    🍅 Томат
                                </button>
                                <button onclick="addIngredient('sauce')" class="bg-orange-600 hover:bg-orange-500 p-1.5 rounded-lg text-[11px] font-bold text-center truncate shadow border border-orange-400">
                                    🥫 Соус
                                </button>
                                <button onclick="addIngredient('topBun')" class="bg-amber-600 hover:bg-amber-500 p-1.5 rounded-lg text-[11px] font-bold text-center truncate shadow border border-amber-400 col-span-2">
                                    🍔 Верхняя булочка
                                </button>
                            </div>
                        </div>

                        <!-- RIGHT: DRINKS & SIDE DISHES -->
                        <div class="metal-grill rounded-2xl p-3 flex flex-col justify-between shadow-inner border border-slate-600">
                            <div class="flex justify-between items-center mb-1">
                                <span class="font-game text-xs text-amber-400 tracking-wider flex items-center gap-1">
                                    <i class="fa-solid fa-glass-water text-cyan-400"></i> НАПИТКИ И ФРИ
                                </span>
                            </div>

                            <!-- Station Items -->
                            <div class="grid grid-cols-2 gap-2 my-auto">
                                <!-- Cola Station -->
                                <div class="bg-slate-900/90 p-2 rounded-xl border border-slate-700 flex flex-col items-center justify-between">
                                    <i class="fa-solid fa-bottle-water text-2xl text-red-500"></i>
                                    <span class="text-[11px] font-bold mt-1">Кола</span>
                                    <button onclick="prepareDrink('cola')" class="mt-2 w-full bg-cyan-700 hover:bg-cyan-600 text-[10px] font-bold py-1.5 rounded-lg shadow">
[04.11.2026 13:40] Shohrux: Налить 🥤
                                    </button>
                                </div>
                                <!-- Fries Station -->
                                <div class="bg-slate-900/90 p-2 rounded-xl border border-slate-700 flex flex-col items-center justify-between">
                                    <i class="fa-solid fa-box-archive text-2xl text-yellow-500"></i>
                                    <span class="text-[11px] font-bold mt-1">Фри</span>
                                    <button onclick="prepareFries()" class="mt-2 w-full bg-amber-600 hover:bg-amber-500 text-[10px] font-bold py-1.5 rounded-lg shadow">
                                        Пожарить 🍟
                                    </button>
                                </div>
                            </div>

                            <!-- Serving Tray / Ready Items -->
                            <div class="bg-slate-950/70 rounded-xl p-2 border border-slate-800">
                                <span class="text-[10px] text-slate-400 font-bold block mb-1">ГОТОВО К ВЫДАЧЕ:</span>
                                <div id="ready-tray" class="flex gap-2 min-h-[36px] items-center overflow-x-auto">
                                    <span class="text-[10px] text-slate-600 italic">Поднос пуст</span>
                                </div>
                            </div>
                        </div>

                    </div>
                </div>

            </div>
        </section>


        <!-- ==================== MENU CATALOG & STORE SECTION ==================== -->
        <section id="section-menu" class="hidden">
            <div class="bg-slate-800/90 border-2 border-slate-700 rounded-3xl p-6 shadow-2xl">
                <!-- Header Title -->
                <div class="flex flex-col md:flex-row justify-between items-center mb-6 pb-4 border-b border-slate-700">
                    <div>
                        <h2 class="font-game text-3xl text-amber-400">МЕНЮ РЕСТОРАНА</h2>
                        <p class="text-slate-400 text-sm">Заказывайте самые вкусные бургеры и напитки с доставкой!</p>
                    </div>
                    <!-- Menu Filter Tabs -->
                    <div class="flex gap-2 mt-4 md:mt-0">
                        <button onclick="filterMenu('all')" class="px-3 py-1.5 rounded-xl bg-amber-500 text-slate-900 font-bold text-xs shadow hover:bg-amber-400 transition">Все</button>
                        <button onclick="filterMenu('burger')" class="px-3 py-1.5 rounded-xl bg-slate-700 text-slate-200 font-bold text-xs hover:bg-slate-600 transition">Бургеры</button>
                        <button onclick="filterMenu('sides')" class="px-3 py-1.5 rounded-xl bg-slate-700 text-slate-200 font-bold text-xs hover:bg-slate-600 transition">Закуски</button>
                        <button onclick="filterMenu('drinks')" class="px-3 py-1.5 rounded-xl bg-slate-700 text-slate-200 font-bold text-xs hover:bg-slate-600 transition">Напитки</button>
                    </div>
                </div>

                <!-- Products Grid -->
                <div id="product-grid" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
                    <!-- Products will be dynamically populated -->
                </div>
            </div>
        </section>

    </main>

    <!-- FOOTER -->
    <footer class="bg-slate-950 border-t border-slate-800 py-4 px-4 text-center text-xs text-slate-500">
        <div class="max-w-6xl mx-auto flex flex-col md:flex-row justify-between items-center gap-2">
            <div>© 2026 Бургер Мастер Ресторан. Все права защищены.</div>
            <div class="flex gap-4">
                <span class="hover:text-amber-400 cursor-pointer">Быстрая Доставка</span>
                <span class="hover:text-amber-400 cursor-pointer">Свежие Ингредиенты</span>
                <span class="hover:text-amber-400 cursor-pointer">Поддержка</span>
            </div>
        </div>
    </footer>
[04.11.2026 13:40] Shohrux: <!-- CART MODAL -->
    <div id="cart-modal" class="fixed inset-0 bg-black/75 z-50 backdrop-blur-sm hidden flex items-center justify-center p-4">
        <div class="bg-slate-900 border-2 border-amber-500/50 rounded-3xl max-w-md w-full p-6 shadow-2xl relative pop-in">
            <button onclick="toggleCartModal()" class="absolute top-4 right-4 text-slate-400 hover:text-white text-xl">
                <i class="fa-solid fa-xmark"></i>
            </button>
            <h3 class="font-game text-2xl text-amber-400 mb-4 flex items-center gap-2">
                <i class="fa-solid fa-cart-shopping"></i> Ваша Корзина
            </h3>

            <!-- Cart Items List -->
            <div id="cart-items-container" class="max-h-60 overflow-y-auto pr-2 space-y-3 mb-4">
                <!-- Cart items dynamically placed here -->
            </div>

            <!-- Total Price & Checkout -->
            <div class="border-t border-slate-800 pt-4">
                <div class="flex justify-between items-center text-lg font-bold mb-4">
                    <span>Итого:</span>
                    <span id="cart-total-price" class="text-amber-400 font-game text-xl">0 ₽</span>
                </div>
                <button onclick="checkout()" class="w-full bg-gradient-to-r from-emerald-600 to-green-500 hover:from-emerald-500 hover:to-green-400 text-white font-game py-3 rounded-2xl shadow-lg transition text-lg tracking-wider">
                    ОФОРМИТЬ ЗАКАЗ
                </button>
            </div>
        </div>
    </div>

    <!-- NOTIFICATION TOAST CONTAINER -->
    <div id="toast-container" class="fixed bottom-5 right-5 z-50 flex flex-col gap-2 pointer-events-none"></div>


    <!-- ==================== GAME LOGIC & INTERACTION SCRIPT ==================== -->
    <script>
        /* ---------------- AUDIO SYNTHESIZER (WEB AUDIO API) ---------------- */
        const AudioCtx = window.AudioContext || window.webkitAudioContext;
        let audioCtx = null;

        function initAudio() {
            if (!audioCtx) audioCtx = new AudioCtx();
        }

        function playSound(type) {
            try {
                initAudio();
                if (!audioCtx) return;

                const osc = audioCtx.createOscillator();
                const gain = audioCtx.createGain();
                osc.connect(gain);
                gain.connect(audioCtx.destination);

                if (type === 'coin') {
                    osc.type = 'sine';
                    osc.frequency.setValueAtTime(800, audioCtx.currentTime);
                    osc.frequency.exponentialRampToValueAtTime(1200, audioCtx.currentTime + 0.15);
                    gain.gain.setValueAtTime(0.3, audioCtx.currentTime);
                    gain.gain.linearRampToValueAtTime(0.01, audioCtx.currentTime + 0.15);
                    osc.start();
                    osc.stop(audioCtx.currentTime + 0.15);
                } else if (type === 'click') {
                    osc.type = 'triangle';
                    osc.frequency.setValueAtTime(300, audioCtx.currentTime);
                    gain.gain.setValueAtTime(0.2, audioCtx.currentTime);
                    gain.gain.linearRampToValueAtTime(0.01, audioCtx.currentTime + 0.05);
                    osc.start();
                    osc.stop(audioCtx.currentTime + 0.05);
                } else if (type === 'sizzle') {
                    // White noise sound for sizzling
                    const bufferSize = audioCtx.sampleRate * 0.2;
                    const buffer = audioCtx.createBuffer(1, bufferSize, audioCtx.sampleRate);
                    const data = buffer.getChannelData(0);
                    for (let i = 0; i < bufferSize; i++) {
                        data[i] = Math.random() * 2 - 1;
                    }
                    const noise = audioCtx.createBufferSource();
[04.11.2026 13:40] Shohrux: noise.buffer = buffer;
                    const noiseGain = audioCtx.createGain();
                    noiseGain.gain.setValueAtTime(0.15, audioCtx.currentTime);
                    noiseGain.gain.linearRampToValueAtTime(0.01, audioCtx.currentTime + 0.2);
                    noise.connect(noiseGain);
                    noiseGain.connect(audioCtx.destination);
                    noise.start();
                } else if (type === 'win') {
                    const now = audioCtx.currentTime;
                    [523.25, 659.25, 783.99, 1046.50].forEach((freq, idx) => {
                        const o = audioCtx.createOscillator();
                        const g = audioCtx.createGain();
                        o.frequency.value = freq;
                        g.gain.setValueAtTime(0.15, now + idx * 0.08);
                        g.gain.linearRampToValueAtTime(0.01, now + idx * 0.08 + 0.2);
                        o.connect(g);
                        g.connect(audioCtx.destination);
                        o.start(now + idx * 0.08);
                        o.stop(now + idx * 0.08 + 0.2);
                    });
                }
            } catch(e) { console.log(e); }
        }

        /* ---------------- GAME STATE ---------------- */
        let gameState = {
            coins: 150,
            xp: 25,
            maxXp: 100,
            level: 1,
            stars: 5,
            grillSlots: [
                { state: 'empty', timer: null, progress: 0 },
                { state: 'empty', timer: null, progress: 0 }
            ],
            currentBurger: [], // ingredients list
            readyTray: [], // cooked items ready to serve
            customers: []
        };

        const MENU_PRODUCTS = [
            { id: 1, name: 'Классик Бургер', price: 350, type: 'burger', icon: '🍔', desc: 'Сочная говяжья котлета, сыр чеддер, салат, свежий томат и соус.' },
            { id: 2, name: 'Двойной Чизбургер', price: 490, type: 'burger', icon: '🍔', desc: 'Две котлеты, двойная порция сыра и хрустящие огурчики.' },
            { id: 3, name: 'BBQ Бекон Бургер', price: 540, type: 'burger', icon: 'burger', icon: '🍔', desc: 'Котлета на гриле, дымный бекон, луковые кольца и BBQ соус.' },
            { id: 4, name: 'Картофель Фри', price: 180, type: 'sides', icon: '🍟', desc: 'Золотистый и хрустящий картофель с солью.' },
            { id: 5, name: 'Наггетсы (6 шт)', price: 220, type: 'sides', icon: '🍗', desc: 'Сочное куриное филе в хрустящей панировке.' },
            { id: 6, name: 'Кола 0.5л', price: 120, type: 'drinks', icon: '🥤', desc: 'Освежающий газированный напиток со льдом.' },
            { id: 7, name: 'Лимонад Домашний', price: 150, type: 'drinks', icon: '🍹', desc: 'Натуральный лимонад с цитрусами и мятой.' }
        ];

        let cart = [];

        /* ---------------- CUSTOMERS CONFIG ---------------- */
        const AVATARS = [
            { name: 'Алексей', img: '👨‍💼', color: 'from-blue-500 to-indigo-600' },
            { name: 'Мария', img: '👩', color: 'from-pink-500 to-rose-600' },
            { name: 'Иван', img: '👦', color: 'from-amber-500 to-orange-600' },
            { name: 'Ольга', img: '👩‍🦰', color: 'from-emerald-500 to-teal-600' }
        ];

        const POSSIBLE_ORDERS = [
            { name: 'Бургер', items: ['bottomBun', 'patty', 'cheese', 'topBun'], icon: '🍔', reward: 80, xp: 20 },
            { name: 'Бургер с салатом', items: ['bottomBun', 'patty', 'cheese', 'lettuce', 'tomato', 'topBun'], icon: '🍔', reward: 120, xp: 35 },
            { name: 'Кола', items: ['cola'], icon: '🥤', reward: 40, xp: 10 },
            { name: 'Фри', items: ['fries'], icon: '🍟', reward: 50, xp: 15 },
            { name: 'Комбо Сет', items: ['bottomBun', 'patty', 'topBun', 'cola'], icon: '🍱', reward: 140, xp: 45 }
        ];

        /* ---------------- INITIALIZATION ---------------- */
        window.onload = function() {
            renderMenuCatalog();
            spawnCustomer(0);
            spawnCustomer(1);
            spawnCustomer(2);
            updateHUD();
[04.11.2026 13:40] Shohrux: // Customer patience tick loop
            setInterval(updateCustomersPatience, 1000);
        };

        function switchTab(tab) {
            playSound('click');
            const gameSec = document.getElementById('section-game');
            const menuSec = document.getElementById('section-menu');
            const btnGame = document.getElementById('btn-tab-game');
            const btnMenu = document.getElementById('btn-tab-menu');

            if (tab === 'game') {
                gameSec.classList.remove('hidden');
                menuSec.classList.add('hidden');
                btnGame.className = "px-4 py-2 rounded-xl font-bold bg-amber-500 text-slate-900 shadow-md hover:bg-amber-400 transition flex items-center gap-2";
                btnMenu.className = "px-4 py-2 rounded-xl font-bold bg-slate-800 text-slate-200 border border-slate-700 hover:bg-slate-700 transition flex items-center gap-2";
            } else {
                gameSec.classList.add('hidden');
                menuSec.classList.remove('hidden');
                btnMenu.className = "px-4 py-2 rounded-xl font-bold bg-amber-500 text-slate-900 shadow-md hover:bg-amber-400 transition flex items-center gap-2";
                btnGame.className = "px-4 py-2 rounded-xl font-bold bg-slate-800 text-slate-200 border border-slate-700 hover:bg-slate-700 transition flex items-center gap-2";
            }
        }

        /* ---------------- HUD & STATS ---------------- */
        function updateHUD() {
            document.getElementById('coins-val').innerText = gameState.coins;
            document.getElementById('stars-val').innerText = gameState.stars;
            document.getElementById('level-val').innerText = gameState.level;
            document.getElementById('xp-bar').style.width = ${(gameState.xp / gameState.maxXp) * 100}%;
            document.getElementById('xp-text').innerText = ${gameState.xp}/${gameState.maxXp};
        }

        function addReward(coins, xp) {
            gameState.coins += coins;
            gameState.xp += xp;

            if (gameState.xp >= gameState.maxXp) {
                gameState.level += 1;
                gameState.xp -= gameState.maxXp;
                gameState.maxXp = Math.floor(gameState.maxXp * 1.5);
                gameState.stars += 1;
                playSound('win');
                showToast(🎉 Уровень повышен! Новый уровень: ${gameState.level}, 'success');
            } else {
                playSound('coin');
            }
            updateHUD();
        }

        /* ---------------- KITCHEN GAMEPLAY LOGIC ---------------- */
        
        // 1. Grill Station Logic
        function startGrilling() {
            // Find empty grill slot
            const slotIndex = gameState.grillSlots.findIndex(s => s.state === 'empty');
            if (slotIndex === -1) {
                showToast('Все грили заняты!', 'warning');
                return;
            }

            playSound('sizzle');
            const slot = gameState.grillSlots[slotIndex];
            slot.state = 'cooking';
            slot.progress = 0;

            const slotEl = document.getElementById(grill-${slotIndex});
            slotEl.innerHTML = 
                <div class="flex flex-col items-center">
                    <span class="text-2xl sizzling">🥩</span>
                    <div class="w-12 bg-slate-700 h-1.5 rounded-full mt-1 overflow-hidden">
                        <div id="grill-prog-${slotIndex}" class="bg-amber-500 h-full w-0 transition-all duration-300"></div>
                    </div>
                </div>
            ;

            slot.timer = setInterval(() => {
                slot.progress += 25;
                const progBar = document.getElementById(grill-prog-${slotIndex});
                if (progBar) progBar.style.width = ${slot.progress}%;
[04.11.2026 13:40] Shohrux: if (slot.progress >= 100) {
                    clearInterval(slot.timer);
                    slot.state = 'ready';
                    slotEl.innerHTML = 
                        <div class="flex flex-col items-center pop-in">
                            <span class="text-2xl">🍔</span>
                            <span class="text-[9px] font-bold text-green-400 bg-slate-950 px-1 rounded">Готово! (клик)</span>
                        </div>
                    ;
                }
            }, 750);
        }

        function cookPatty(slotIndex) {
            const slot = gameState.grillSlots[slotIndex];
            if (slot.state === 'ready') {
                playSound('click');
                // Move cooked patty to assembly or ready items
                addIngredient('patty');
                slot.state = 'empty';
                document.getElementById(grill-${slotIndex}).innerHTML = 
                    <span class="text-xs text-slate-600 font-bold">Гриль ${slotIndex + 1}</span>
                ;
            }
        }

        // 2. Burger Assembly Logic
        function addIngredient(type) {
            playSound('click');
            const plate = document.getElementById('burger-plate');
            
            if (gameState.currentBurger.length === 0) {
                plate.innerHTML = ''; // clear "empty plate" message
            }

            gameState.currentBurger.push(type);
            renderBurgerStack();
        }

        function clearCurrentBurger() {
            playSound('click');
            gameState.currentBurger = [];
            const plate = document.getElementById('burger-plate');
            plate.innerHTML = '<span class="text-slate-600 text-xs my-auto">Тарелка пуста</span>';
        }

        function renderBurgerStack() {
            const plate = document.getElementById('burger-plate');
            plate.innerHTML = '';

            const ingredientVisuals = {
                bottomBun: { label: '🍞 Нижняя булка', color: 'bg-amber-700 border-amber-800 text-amber-100', height: 'h-4' },
                patty: { label: '🥩 Котлета', color: 'bg-amber-950 border-black text-amber-200', height: 'h-4' },
                cheese: { label: '🧀 Сыр Чеддер', color: 'bg-yellow-400 border-yellow-500 text-yellow-950', height: 'h-2' },
                lettuce: { label: '🥬 Свежий салат', color: 'bg-emerald-500 border-emerald-600 text-emerald-950', height: 'h-2.5' },
                tomato: { label: '🍅 Томат', color: 'bg-red-500 border-red-600 text-white', height: 'h-2.5' },
                sauce: { label: '🥫 Фирменный Соус', color: 'bg-orange-500 border-orange-600 text-white', height: 'h-1.5' },
                topBun: { label: '🍔 Верхняя булка', color: 'bg-amber-600 border-amber-700 text-amber-100 rounded-t-xl', height: 'h-5' }
            };

            gameState.currentBurger.forEach(ing => {
                const item = ingredientVisuals[ing] || { label: ing, color: 'bg-gray-500', height: 'h-3' };
                const div = document.createElement('div');
                div.className = w-28 ${item.height} ${item.color} border flex items-center justify-center text-[9px] font-bold shadow-sm rounded-sm pop-in -mb-0.5;
                div.innerText = item.label;
                plate.appendChild(div);
            });

            // Check if burger is finished (top bun placed)
            if (gameState.currentBurger.includes('topBun') && gameState.currentBurger.includes('bottomBun')) {
                setTimeout(finishBurgerAssembly, 300);
            }
        }

        function finishBurgerAssembly() {
            // Add assembled burger to Ready Tray
            const burgerData = {
                type: 'burger',
                recipe: [...gameState.currentBurger]
[04.11.2026 13:40] Shohrux: };
            gameState.readyTray.push(burgerData);
            gameState.currentBurger = [];
            renderBurgerStack();
            document.getElementById('burger-plate').innerHTML = '<span class="text-slate-600 text-xs my-auto">Тарелка пуста</span>';
            renderReadyTray();
            showToast('🍔 Бургер собран и перенесен на поднос!', 'info');
        }

        // 3. Drinks & Sides Logic
        function prepareDrink(type) {
            playSound('click');
            gameState.readyTray.push({ type: 'cola' });
            renderReadyTray();
            showToast('🥤 Кола готова!', 'info');
        }

        function prepareFries() {
            playSound('click');
            gameState.readyTray.push({ type: 'fries' });
            renderReadyTray();
            showToast('🍟 Картофель фри готов!', 'info');
        }

        function renderReadyTray() {
            const container = document.getElementById('ready-tray');
            container.innerHTML = '';

            if (gameState.readyTray.length === 0) {
                container.innerHTML = '<span class="text-[10px] text-slate-600 italic">Поднос пуст</span>';
                return;
            }

            gameState.readyTray.forEach((item, index) => {
                const badge = document.createElement('div');
                badge.className = 'bg-slate-800 border border-slate-600 rounded-lg p-1 px-2 flex items-center gap-1 cursor-pointer hover:border-red-400 transition';
                badge.title = 'Нажмите чтобы отменить';
                badge.onclick = () => {
                    gameState.readyTray.splice(index, 1);
                    renderReadyTray();
                };

                let icon = '🍔';
                if (item.type === 'cola') icon = '🥤';
                if (item.type === 'fries') icon = '🍟';

                badge.innerHTML = <span class="text-sm">${icon}</span> <i class="fa-solid fa-xmark text-[9px] text-slate-500"></i>;
                container.appendChild(badge);
            });
        }


        /* ---------------- CUSTOMER ORDERS & AI ---------------- */
        function spawnCustomer(slotIndex) {
            const avatar = AVATARS[Math.floor(Math.random() * AVATARS.length)];
            const order = POSSIBLE_ORDERS[Math.floor(Math.random() * POSSIBLE_ORDERS.length)];

            gameState.customers[slotIndex] = {
                avatar: avatar,
                order: order,
                patience: 100,
                maxPatience: 100
            };

            renderCustomers();
        }

        function renderCustomers() {
            const container = document.getElementById('customers-container');
            container.innerHTML = '';

            gameState.customers.forEach((cust, index) => {
                if (!cust) return;

                const custDiv = document.createElement('div');
                custDiv.className = 'flex flex-col items-center relative w-1/3 max-w-[150px] pop-in';

                // Order Speech Bubble
                const bubble = document.createElement('div');
                bubble.className = 'bg-white text-slate-900 rounded-2xl p-2.5 shadow-xl border-2 border-amber-400 mb-3 relative bubble-tail text-center w-full cursor-pointer hover:scale-105 transition';
                bubble.onclick = () => serveCustomer(index);

                bubble.innerHTML = `
                    <div class="text-[10px] font-bold text-slate-500 uppercase tracking-wider mb-0.5">Хочу заказать:</div>
                    <div class="font-game text-sm flex items-center justify-center gap-1 text-slate-800">
                        <span>${cust.order.icon}</span> ${cust.order.name}
                    </div>
                    <!-- Patience Progress Bar -->
                    <div class="w-full bg-gray-200 h-1.5 rounded-full mt-1.5 overflow-hidden">
[04.11.2026 13:40] Shohrux: <div class="bg-gradient-to-r from-emerald-500 to-amber-500 h-full transition-all duration-500" style="width: ${cust.patience}%"></div>
                    </div>
                    <div class="text-[9px] font-bold text-amber-600 mt-1">Отдать заказ ➔</div>
                ;

                // Avatar Graphic
                const avatarEl = document.createElement('div');
                avatarEl.className = w-16 h-16 rounded-full bg-gradient-to-tr ${cust.avatar.color} flex items-center justify-center text-3xl shadow-lg border-2 border-white/20;
                avatarEl.innerText = cust.avatar.img;

                const nameLabel = document.createElement('span');
                nameLabel.className = 'text-[11px] font-bold text-slate-300 mt-1';
                nameLabel.innerText = cust.avatar.name;

                custDiv.appendChild(bubble);
                custDiv.appendChild(avatarEl);
                custDiv.appendChild(nameLabel);
                container.appendChild(custDiv);
            });
        }

        function updateCustomersPatience() {
            gameState.customers.forEach((cust, idx) => {
                if (cust) {
                    cust.patience -= 3;
                    if (cust.patience <= 0) {
                        showToast(😞 ${cust.avatar.name} ушел не дождавшись заказа!, 'error');
                        spawnCustomer(idx);
                    }
                }
            });
            renderCustomers();
        }

        function serveCustomer(slotIndex) {
            const cust = gameState.customers[slotIndex];
            if (!cust) return;

            // Check if ready tray contains matching order items
            const requiredItems = cust.order.items;
            let matchIndex = -1;

            if (cust.order.name.includes('Бургер')) {
                matchIndex = gameState.readyTray.findIndex(item => item.type === 'burger');
            } else if (cust.order.name === 'Кола') {
                matchIndex = gameState.readyTray.findIndex(item => item.type === 'cola');
            } else if (cust.order.name === 'Фри') {
                matchIndex = gameState.readyTray.findIndex(item => item.type === 'fries');
            } else if (cust.order.name === 'Комбо Сет') {
                matchIndex = gameState.readyTray.findIndex(item => item.type === 'burger');
            }

            if (matchIndex !== -1) {
                // Order fulfilled!
                gameState.readyTray.splice(matchIndex, 1);
                renderReadyTray();

                addReward(cust.order.reward, cust.order.xp);
                showToast(✅ Заказ выполнен! +${cust.order.reward} монет, 'success');

                // Respawn customer after delay
                delete gameState.customers[slotIndex];
                renderCustomers();
                setTimeout(() => spawnCustomer(slotIndex), 2000);
            } else {
                showToast('❌ На подносе нет подходящего блюда для этого заказа!', 'error');
            }
        }


        /* ---------------- ONLINE STORE CATALOG & CART ---------------- */
        function renderMenuCatalog(category = 'all') {
            const grid = document.getElementById('product-grid');
            grid.innerHTML = '';

            const filtered = category === 'all' ? MENU_PRODUCTS : MENU_PRODUCTS.filter(p => p.type === category);

            filtered.forEach(product => {
                const card = document.createElement('div');
                card.className = 'bg-slate-900 border border-slate-700 rounded-2xl p-4 flex flex-col justify-between hover:border-amber-500/50 transition group shadow-lg';
                card.innerHTML = 
                    <div>
                        <div class="h-32 bg-slate-950/80 rounded-xl flex items-center justify-center text-6xl mb-3 group-hover:scale-105 transition transform">
[04.11.2026 13:40] Shohrux: ${product.icon}
                        </div>
                        <h3 class="font-game text-lg text-white mb-1">${product.name}</h3>
                        <p class="text-xs text-slate-400 mb-3 leading-relaxed">${product.desc}</p>
                    </div>
                    <div class="flex justify-between items-center pt-2 border-t border-slate-800">
                        <span class="font-game text-xl text-amber-400">${product.price} ₽</span>
                        <button onclick="addToCart(${product.id})" class="bg-amber-500 hover:bg-amber-400 text-slate-950 font-bold px-3 py-1.5 rounded-xl text-xs flex items-center gap-1.5 transition shadow">
                            <i class="fa-solid fa-plus"></i> В корзину
                        </button>
                    </div>
                ;
                grid.appendChild(card);
            });
        }

        function filterMenu(cat) {
            playSound('click');
            renderMenuCatalog(cat);
        }

        function addToCart(productId) {
            playSound('click');
            const item = MENU_PRODUCTS.find(p => p.id === productId);
            const inCart = cart.find(c => c.id === productId);

            if (inCart) {
                inCart.qty += 1;
            } else {
                cart.push({ ...item, qty: 1 });
            }

            updateCartUI();
            showToast(🛒 ${item.name} добавлен в корзину, 'info');
        }

        function updateCartUI() {
            const countEl = document.getElementById('cart-count');
            const totalQty = cart.reduce((acc, i) => acc + i.qty, 0);
            countEl.innerText = totalQty;

            const container = document.getElementById('cart-items-container');
            const totalPriceEl = document.getElementById('cart-total-price');
            container.innerHTML = '';

            let total = 0;

            if (cart.length === 0) {
                container.innerHTML = '<div class="text-center text-slate-500 py-6 text-sm">Ваша корзина пуста</div>';
                totalPriceEl.innerText = '0 ₽';
                return;
            }

            cart.forEach(item => {
                total += item.price * item.qty;
                const row = document.createElement('div');
                row.className = 'flex justify-between items-center bg-slate-800 p-3 rounded-xl border border-slate-700';
                row.innerHTML = 
                    <div class="flex items-center gap-3">
                        <span class="text-2xl">${item.icon}</span>
                        <div>
                            <div class="font-bold text-sm text-white">${item.name}</div>
                            <div class="text-xs text-amber-400 font-bold">${item.price} ₽</div>
                        </div>
                    </div>
                    <div class="flex items-center gap-2">
                        <button onclick="changeQty(${item.id}, -1)" class="w-6 h-6 bg-slate-700 rounded-lg text-xs font-bold text-white hover:bg-slate-600">-</button>
                        <span class="text-xs font-bold w-4 text-center">${item.qty}</span>
                        <button onclick="changeQty(${item.id}, 1)" class="w-6 h-6 bg-slate-700 rounded-lg text-xs font-bold text-white hover:bg-slate-600">+</button>
                    </div>
                ;
                container.appendChild(row);
            });

            totalPriceEl.innerText = ${total} ₽`;
        }

        function changeQty(id, delta) {
            const item = cart.find(c => c.id === id);
            if (!item) return;

            item.qty += delta;
            if (item.qty <= 0) {
                cart = cart.filter(c => c.id !== id);
            }
            updateCartUI();
        }

        function toggleCartModal() {
            playSound('click');
            const modal = document.getElementById('cart-modal');
            modal.classList.toggle('hidden');
        }
[04.11.2026 13:40] Shohrux: function checkout() {
            if (cart.length === 0) {
                showToast('Ваша корзина пуста!', 'warning');
                return;
            }
            playSound('win');
            showToast('🎉 Заказ успешно оформлен! Курьер уже мчит к вам!', 'success');
            cart = [];
            updateCartUI();
            toggleCartModal();
        }

        /* ---------------- TOAST NOTIFICATIONS ---------------- */
        function showToast(msg, type = 'info') {
            const container = document.getElementById('toast-container');
            const toast = document.createElement('div');
            
            let bg = 'bg-slate-800 border-amber-500 text-amber-200';
            if (type === 'success') bg = 'bg-emerald-900 border-emerald-500 text-emerald-200';
            if (type === 'error') bg = 'bg-red-900 border-red-500 text-red-200';
            if (type === 'warning') bg = 'bg-amber-900 border-amber-500 text-amber-200';

            toast.className = p-3 px-4 rounded-2xl border-2 ${bg} text-xs font-bold shadow-xl flex items-center gap-2 pop-in pointer-events-auto;
            toast.innerText = msg;

            container.appendChild(toast);

            setTimeout(() => {
                toast.style.opacity = '0';
                toast.style.transition = 'opacity 0.3s';
                setTimeout(() => toast.remove(), 300);
            }, 3000);
        }
    </script>
</body>
</html>
`
