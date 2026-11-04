```html
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
                <!-- Level Badge -->
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

                    <!-- Dining Counter Divider -->
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
                                <div id="burger-plate" class="burger-stack z-10 mb-2 transition-all">
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
                                        Налить 🥤
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

    <!-- CART MODAL -->
    <div id="cart-modal" class="fixed inset-0 bg-black/75 z-50 backdrop-blur-sm hidden flex items-center justify-center p-4">
        <div class="bg-slate-900 border-2 border-amber-500/50 rounded-3xl max-w-md w-full p-6 shadow-2xl relative pop-in">
            <button onclick="toggleCartModal()" class="absolute top-4 right-4 text-slate-400 hover:text-white text-xl">
                <i class="fa-solid fa-xmark"></i>
            </button>
            <h3 class="font-game text-2xl text-amber-400 mb-4 flex items-center gap-2"
