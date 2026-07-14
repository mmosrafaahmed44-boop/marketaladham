<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>أسواق الأدهم</title>
    <!-- تحميل مكتبة Tailwind CSS للتصميم العصري والسريع -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Cairo:wght@400;600;700&display=swap');
        body { 
            background-color: #f0fdf4; 
            color: #1e293b; 
            font-family: 'Cairo', sans-serif; 
            -webkit-tap-highlight-color: transparent;
        }

        /* حركات التنبيه والنبض لعروض اليوم المميزة */
        @keyframes pulse-red {
            0%, 100% { transform: scale(1); box-shadow: 0 0 0 0 rgba(220, 38, 38, 0.4); }
            50% { transform: scale(1.03); box-shadow: 0 0 10px 4px rgba(220, 38, 38, 0.2); }
        }
        .offer-btn-active {
            animation: pulse-red 2s ease-in-out infinite;
        }

        /* حركة شريط "وصل حديثاً" الدوار بشكل انسيابي من اليسار إلى اليمين */
        @keyframes marquee-scroll {
            0% { transform: translateX(-50%); }
            100% { transform: translateX(0%); }
        }
        .marquee-container {
            overflow: hidden;
            white-space: nowrap;
            position: relative;
            direction: ltr; /* لضمان دوران الحركة من اليسار لليمين */
        }
        .marquee-track {
            display: inline-flex;
            gap: 1.5rem;
            animation: marquee-scroll 25s linear infinite;
            width: max-content;
        }
        .marquee-track:hover {
            animation-play-state: paused; /* يقف مؤقتاً عند مرور الماوس لراحة الزبون */
        }

        ::-webkit-scrollbar { width: 4px; }
        ::-webkit-scrollbar-thumb { background: #10b981; border-radius: 10px; }
        .nav-icon { width: 22px; height: 22px; stroke: currentColor; stroke-width: 2; fill: none; }
    </style>
</head>
<body class="pb-28">

    <!-- -->
    <!-- حاوية التنبيهات السريعة العلوية -->
    <div id="toast" class="fixed top-4 left-1/2 -translate-x-1/2 px-6 py-3 rounded-2xl shadow-2xl z-[100] transition-all duration-500 text-center font-bold text-sm w-11/12 max-w-sm opacity-0 -translate-y-10 pointer-events-none flex items-center justify-center gap-2 text-white">
        <span id="toast-msg"></span>
    </div>

    <!-- نافذة تأكيد الخروج لمنع الخروج بالخطأ -->
    <div id="exit-modal" class="fixed inset-0 bg-black/60 backdrop-blur-sm z-[150] flex items-center justify-center p-4 opacity-0 pointer-events-none transition-all duration-300">
        <div class="bg-white rounded-3xl p-6 max-w-sm w-full text-center shadow-2xl border border-emerald-100 transform scale-95 transition-all duration-300" id="exit-modal-content">
            <div class="w-16 h-16 bg-red-50 rounded-full flex items-center justify-center mx-auto text-3xl mb-4 animate-bounce">👋</div>
            <h3 class="text-lg font-bold text-emerald-950 mb-2">هل تريد الخروج من التطبيق؟</h3>
            <p class="text-xs text-emerald-600 mb-6 leading-relaxed">يسعدنا دائماً تسوقك معنا في أسواق الأدهم، هل أنت متأكد من رغبتك في المغادرة الآن؟</p>
            <div class="grid grid-cols-2 gap-3">
                <button onclick="closeExitModal()" class="bg-emerald-600 hover:bg-emerald-700 text-white py-3 rounded-xl font-bold text-sm shadow-md active:scale-95 transition-all">لا، البقاء 💚</button>
                <button onclick="exitApp()" class="bg-red-50 hover:bg-red-100 text-red-600 py-3 rounded-xl font-bold text-sm border border-red-200 active:scale-95 transition-all">نعم، خروج 🚪</button>
            </div>
        </div>
    </div>

    <!-- ترويسة التطبيق الثابتة بالأعلى مع شعار البانر الرسمي الفخم للمتجر -->
    <header class="p-4 text-center sticky top-0 z-40 bg-emerald-50/90 backdrop-blur-md shadow-sm rounded-b-3xl">
        <div class="rounded-2xl shadow-lg border border-emerald-700 overflow-hidden">
            <img src="https://i.ibb.co/0R11835H/IMG-20260712-WA0006.jpg" alt="أسواق الأدهم" class="w-full h-auto object-cover">
        </div>
    </header>

    <!-- شريط "وصل حديثاً" الدوار المميز والنشط التفاعلي تحت الترويسة مباشرة -->
    <div id="new-arrivals-bar" onclick="openNewArrivalsCategory()" class="bg-emerald-950 text-white py-2.5 shadow-inner border-y border-emerald-800 cursor-pointer hover:bg-emerald-900 transition-colors duration-300">
        <div class="max-w-lg mx-auto flex items-center px-4">
            <span class="bg-red-600 text-white text-[11px] font-bold px-2.5 py-1 rounded-lg ml-3 shadow-md flex-shrink-0 animate-pulse">🆕 وصل حديثاً</span>
            <div class="marquee-container w-full" id="marquee-container">
                <div class="marquee-track" id="marquee-track">
                    <span class="text-sm font-semibold opacity-75">جاري تحميل المنتجات الجديدة...</span>
                </div>
            </div>
        </div>
    </div>

    <!-- -->
    <!-- حاوية الصفحات الرئيسية للتطبيق -->
    <main id="app" class="p-4 max-w-lg mx-auto">
        
        <!-- لوحة تشخيص الأخطاء الذكية على الشاشة (تظهر فقط عند حدوث خطأ في جلب البيانات) -->
        <div id="debug-error-panel" class="hidden bg-red-50 border-2 border-red-200 rounded-3xl p-5 mb-6 text-right shadow-lg">
            <div class="flex items-center gap-2 text-red-600 font-bold mb-2">
                <span class="text-xl">⚠️</span>
                <h3 class="text-base">خطأ في قراءة جدول البيانات</h3>
            </div>
            <p class="text-xs text-red-700 leading-relaxed mb-4">
                لم يتمكن الموقع من جلب المنتجات من جدول جوجل شيتس الخاص بك. يرجى مراجعة الخطوات التالية لحل المشكلة فوراً:
            </p>
            <ol class="text-xs text-red-800 list-decimal list-inside space-y-2 mb-4 leading-relaxed">
                <li>تأكد من قيامك بـ <b>"النشر على الويب"</b> كـ ملف <b>CSV</b> وليس مشاركة عادية.</li>
                <li>تأكد من كتابة أسماء الأعمدة بدقة بالإنجليزية في الصف الأول: <code class="bg-red-100 text-red-950 px-1 py-0.5 rounded font-mono">id, name, price, cat, img</code>.</li>
                <li>تأكد أن عمود السعر (price) يحتوي على <b>أرقام فقط</b> (مثال: <code class="bg-red-100 text-red-950 px-1 py-0.5 rounded font-mono">15</code> وليس "15 جنيه").</li>
            </ol>
            <div class="bg-red-100 p-3 rounded-xl text-[11px] font-mono text-red-950 break-all leading-relaxed">
                <b>تفاصيل الخطأ الفني:</b> <span id="debug-error-message" class="font-bold text-red-600"></span>
            </div>
        </div>

        <!-- صفحة الأقسام الرئيسية -->
        <div id="categories-page">
            <h2 class="text-lg font-bold mb-4 text-emerald-900 flex items-center gap-2">
                <span>📁 الأقسام المتاحة</span>
            </h2>
            <div id="categories-grid" class="grid grid-cols-2 gap-4">
                <div class="col-span-2 text-center py-12 text-emerald-600 font-semibold animate-pulse">جاري تحميل الأقسام...</div>
            </div>
        </div>

        <!-- صفحة المنتجات داخل القسم المحدد -->
        <div id="products-page" class="hidden">
            <button onclick="goBack()" class="text-emerald-700 hover:text-emerald-900 mb-6 font-bold flex items-center gap-1 transition-colors">
                <span class="text-lg">←</span> عودة للأقسام
            </button>
            <h2 id="cat-title" class="text-2xl font-bold mb-6 text-emerald-950 text-center relative after:content-[''] after:block after:w-16 after:h-1 after:bg-emerald-600 after:mx-auto after:mt-2 after:rounded"></h2>
            <div id="products-grid" class="grid grid-cols-2 gap-4"></div>
        </div>

        <!-- صفحة سلة المشتريات -->
        <div id="cart-page" class="hidden">
            <h2 class="text-2xl font-bold mb-6 text-emerald-950 text-center">🛒 السلة</h2>
            <div id="cart-items" class="space-y-4 mb-6"></div>
            
            <!-- -->
            <!-- نموذج إدخال البيانات الإجباري عدا الملاحظات -->
            <div class="bg-white p-5 rounded-3xl border border-emerald-100 shadow-lg space-y-4 mb-6">
                <h3 class="font-bold text-emerald-950 text-base mb-2 border-b pb-2">📋 بيانات العميل لتوصيل الطلب</h3>
                <div>
                    <label class="block text-xs font-bold text-emerald-900 mb-1">الاسم الكامل <span class="text-red-500">*</span></label>
                    <input type="text" id="cust-name" placeholder="اكتب اسمك الثلاثي" class="w-full p-3 border border-emerald-200 rounded-xl focus:ring-2 focus:ring-emerald-500 text-sm">
                </div>
                <div>
                    <label class="block text-xs font-bold text-emerald-900 mb-1">رقم التليفون (11 رقم) <span class="text-red-500">*</span></label>
                    <input type="tel" id="cust-phone" placeholder="يبدأ بـ 01..." class="w-full p-3 border border-emerald-200 rounded-xl focus:ring-2 focus:ring-emerald-500 text-sm" dir="ltr">
                </div>
                <div>
                    <label class="block text-xs font-bold text-emerald-900 mb-1">العنوان بالتفصيل <span class="text-red-500">*</span></label>
                    <input type="text" id="cust-addr" placeholder="المنطقة، الشارع، علامة مميزة" class="w-full p-3 border border-emerald-200 rounded-xl focus:ring-2 focus:ring-emerald-500 text-sm">
                </div>
                <div>
                    <label class="block text-xs font-bold text-emerald-900 mb-1">ملاحظات على الطلب <span class="text-gray-400">(اختياري)</span></label>
                    <textarea id="cust-notes" placeholder="أي ملاحظات تود إضافتها للطلب..." class="w-full p-3 border border-emerald-200 rounded-xl focus:ring-2 focus:ring-emerald-500 text-sm h-20"></textarea>
                </div>
            </div>
            
            <!-- لوحة إتمام الطلب الفوري والمباشر -->
            <div class="bg-white p-6 rounded-3xl border border-emerald-100 shadow-lg text-center">
                <div class="w-12 h-12 bg-emerald-100 rounded-full flex items-center justify-center mx-auto text-xl mb-3">🛒</div>
                <h3 class="font-bold text-emerald-950 text-lg">جاهز لإرسال طلبك?</h3>
                <p class="text-xs text-emerald-700 leading-relaxed mb-4">يرجى ملء بياناتك أعلاه بدقة، وسيتم إرسال طلبك مباشرة إلى واتساب بضغطة زر واحدة مجهزاً بالكامل!</p>
                <button onclick="confirmOrder()" class="w-full bg-emerald-600 hover:bg-emerald-700 text-white py-4 rounded-xl font-bold shadow-lg active:scale-95 transition-all flex items-center justify-center gap-2">
                    <span>💬 إرسال الطلب المباشر عبر واتساب</span>
                </button>
            </div>
        </div>

        <!-- صفحة أرشيف المشتريات السابقة (مشترياتي) -->
        <div id="purchases-page" class="hidden">
            <h2 class="text-2xl font-bold mb-6 text-emerald-950 text-center">📜 مشترياتي السابقة</h2>
            <div id="purchases-items" class="space-y-4"></div>
        </div>

        <!-- صفحة مساعدة ومعلومات المتجر -->
        <div id="about-page" class="hidden">
            <h2 class="text-2xl font-bold mb-6 text-emerald-950 text-center">🆘 مساعدة ومعلومات</h2>
            <div class="bg-white p-6 rounded-3xl shadow-sm border border-emerald-100 space-y-6 text-center">
                <div class="w-20 h-20 bg-emerald-100 rounded-full flex items-center justify-center mx-auto text-4xl">💚</div>
                <div>
                    <h3 class="font-bold text-xl text-emerald-950">أسواق الأدهم</h3>
                    <p class="text-emerald-600 mt-1 text-sm">نسعد دائماً بخدمتكم وتوفير أفضل المنتجات الطازجة واليومية</p>
                </div>
                <div class="border-t border-emerald-50 pt-4 text-right space-y-3">
                    <p class="text-emerald-950 font-bold text-sm">💡 طريقة الطلب والشراء السريعة:</p>
                    <ol class="text-xs text-emerald-700 list-decimal list-inside space-y-2.5 leading-relaxed">
                        <li>تصفح الأقسام والمنتجات المتوفرة.</li>
                        <li>أضف الكميات المناسبة التي ترغب بها إلى سلتك.</li>
                        <li>توجه إلى صفحة السلة واكتب اسمك، عنوانك ورقم تليفونك.</li>
                        <li>اضغط على "إرسال الطلب عبر واتساب" لتفتح محادثة مجهزة بطلبك بالكامل!</li>
                        <li class="border-t border-emerald-50 pt-2 font-bold text-emerald-950">
                            العنوان بالتفصيل: <span class="text-emerald-700 font-extrabold">بجوار مسجد الزاوية</span>
                        </li>
                        <li class="font-bold text-emerald-950">
                            مواعيد العمل: <span class="text-emerald-700 font-extrabold">من 7 صباحاً حتى 12 مساءً</span>
                        </li>
                        <li class="font-bold text-emerald-950">
                            رقم الهاتف للتواصل المباشر: 
                            <a href="tel:01093732952" dir="ltr" class="text-emerald-600 hover:underline font-extrabold mr-1 inline-block">01093732952</a>
                        </li>
                    </ol>
                </div>
            </div>
        </div>
    </main>

    <!-- -->
    <!-- شريط التنقل السفلي الثابت المطور -->
    <nav id="bottom-nav" class="fixed bottom-4 left-4 right-4 max-w-lg mx-auto bg-emerald-700 text-white p-2.5 rounded-3xl flex justify-between items-center z-50 shadow-2xl border border-emerald-600">
        
        <!-- 1. الرئيسية (يمين خالص) -->
        <button onclick="showPage('categories-page')" class="flex flex-col items-center gap-1 flex-1 py-1 text-emerald-100 hover:text-white transition-all">
            <svg class="nav-icon" viewBox="0 0 24 24"><path d="M3 12l2-2m0 0l7-7 7 7M5 10v10a1 1 0 001 1h3m10-11l2 2m-2-2v10a1 1 0 01-1 1h-3m-6 0a1 1 0 001-1v-4a1 1 0 011-1h2a1 1 0 011 1v4a1 1 0 001 1m-6 0h6"/></svg>
            <span class="text-[10px] font-semibold">الرئيسية</span>
        </button>
        
        <!-- 2. مشترياتي -->
        <button onclick="showPage('purchases-page')" class="flex flex-col items-center gap-1 flex-1 py-1 text-emerald-100 hover:text-white transition-all">
            <svg class="nav-icon" viewBox="0 0 24 24"><path d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2"/></svg>
            <span class="text-[10px] font-semibold">مشترياتي</span>
        </button>

        <!-- 3. عروض اليوم (الزر الأحمر المميز بالمنتصف - مع تفعيل التغيير التلقائي للاسم من الجدول) -->
        <button onclick="openOffersCategory()" class="flex flex-col items-center justify-center gap-1 bg-red-600 hover:bg-red-700 text-white rounded-2xl py-1.5 px-3 mx-1 flex-1 transition-all offer-btn-active">
            <span class="text-xs">🔥</span>
            <span id="offers-btn-text" class="text-[10px] font-bold">عروض اليوم</span>
        </button>

        <!-- 4. السلة -->
        <button onclick="showPage('cart-page')" class="flex flex-col items-center gap-1 flex-1 py-1 text-emerald-100 hover:text-white transition-all relative">
            <svg class="nav-icon" viewBox="0 0 24 24"><path d="M3 3h2l.4 2M7 13h10l4-8H5.4M7 13L5.4 5M7 13l-2.293 2.293c-.63.63-.184 1.707.707 1.707H17m0 0a2 2 0 100 4 2 2 0 000-4zm-8 2a2 2 0 11-4 0 2 2 0 014 0z"/></svg>
            <span id="cart-badge" class="absolute -top-1.5 right-4 bg-red-600 text-white text-[9px] font-bold rounded-full w-4.5 h-4.5 flex items-center justify-center scale-0 transition-transform">0</span>
            <span class="text-[10px] font-semibold">السلة</span>
        </button>
        
        <!-- 5. مساعدة (يسار خالص) -->
        <button onclick="showPage('about-page')" class="flex flex-col items-center gap-1 flex-1 py-1 text-emerald-100 hover:text-white transition-all">
            <svg class="nav-icon" viewBox="0 0 24 24"><path d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"/></svg>
            <span class="text-[10px] font-semibold">مساعدة</span>
        </button>
    </nav>

    <!-- -->
    <script>
        // إعدادات المتغيرات العامة وإدارة الحالة
        let allData = [];
        let cart = {};
        let purchases = JSON.parse(localStorage.getItem('purchases') || '[]');
        let toastTimeout;
        let offersCategoryName = "عروض اليوم"; // افتراضي ويتحدث تلقائياً من الجدول

        // إدارة زر الرجوع الذكي لمنع خروج المستخدم بطريق الخطأ
        window.history.pushState({page: 'home'}, '', '');
        window.addEventListener('popstate', function(event) {
            if(!document.getElementById('categories-page').classList.contains('hidden')) {
                showExitModal();
            } else {
                goBack();
                window.history.pushState({page: 'home'}, '', '');
            }
        });

        // دوال التحكم بنافذة الخروج
        function showExitModal() {
            const modal = document.getElementById('exit-modal');
            const content = document.getElementById('exit-modal-content');
            modal.classList.remove('opacity-0', 'pointer-events-none');
            content.classList.remove('scale-95');
            content.classList.add('scale-100');
        }

        function closeExitModal() {
            const modal = document.getElementById('exit-modal');
            const content = document.getElementById('exit-modal-content');
            modal.classList.add('opacity-0', 'pointer-events-none');
            content.classList.remove('scale-100');
            content.classList.add('scale-95');
            window.history.pushState({page: 'home'}, '', '');
        }

        function exitApp() {
            if (navigator.app && navigator.app.exitApp) {
                navigator.app.exitApp();
            } else if (navigator.device && navigator.device.exitApp) {
                navigator.device.exitApp();
            } else {
                window.location.href = "about:blank";
            }
        }

        // إظهار التنبيهات السريعة (تلقائياً لـ 4 ثوانٍ)
        function showToast(message, type = 'success', duration = 4000) {
            const toast = document.getElementById('toast');
            const toastMsg = document.getElementById('toast-msg');
            clearTimeout(toastTimeout);
            toastMsg.innerText = message;
            toast.className = `fixed top-4 left-1/2 -translate-x-1/2 px-6 py-3 rounded-2xl shadow-2xl z-[100] transition-all duration-500 text-center font-bold text-sm w-11/12 max-w-sm flex items-center justify-center gap-2 text-white ${type === 'error' ? 'bg-red-600' : 'bg-emerald-600'}`;
            toast.classList.remove('opacity-0', '-translate-y-10');
            toast.classList.add('opacity-100', 'translate-y-0');
            toastTimeout = setTimeout(() => {
                toast.classList.remove('opacity-100', 'translate-y-0');
                toast.classList.add('opacity-0', '-translate-y-10');
            }, duration);
        }

        // قائمة السلع الافتراضية الذكية جداً لتشغيل المتجر فوراً في حال انقطاع النت أو حدوث خطأ فني بالجدول
        const defaultFallbackProducts = [
            { id: "fallback-1", name: "أرز مصري فاخر معبأ 1 كيلو", price: 32, cat: "الأرز والمكرونة", img: "https://images.unsplash.com/photo-1586201375761-83865001e31c?auto=format&fit=crop&q=80&w=200" },
            { id: "fallback-2", name: "مكرونة الملكة 400 جرام", price: 12, cat: "الأرز والمكرونة", img: "https://images.unsplash.com/photo-1621961411198-e40a048a72c1?auto=format&fit=crop&q=80&w=200" },
            { id: "fallback-3", name: "جبنة عبور لاند فيتا نصف دسم", price: 38, cat: "الألبان والجبن", img: "https://images.unsplash.com/photo-1528750901443-e9c17cc97604?auto=format&fit=crop&q=80&w=200" },
            { id: "fallback-4", name: "حليب جهينة كامل الدسم 1 لتر", price: 44, cat: "الألبان والجبن", img: "https://images.unsplash.com/photo-1550583724-b2692b85b150?auto=format&fit=crop&q=80&w=200" },
            { id: "fallback-5", name: "زيت عافية ذرة 800 مل", price: 95, cat: "الزيوت والخل", img: "https://images.unsplash.com/photo-1474979266404-7eaacbcd87c5?auto=format&fit=crop&q=80&w=200" },
            { id: "fallback-6", name: "صابون ديتول حماية أصلية", price: 20, cat: "المنظفات والصابون", img: "https://images.unsplash.com/photo-1607006342445-360f141b02db?auto=format&fit=crop&q=80&w=200" },
            { id: "fallback-7", name: "تونة دولفين سهلة الفتح", price: 45, cat: "المعلبات", img: "https://images.unsplash.com/photo-1599059021750-8e940e2b63bc?auto=format&fit=crop&q=80&w=200" },
            { id: "fallback-8", name: "شيبس عائلي ملح وخل", price: 10, cat: "عروض اليوم", img: "https://images.unsplash.com/photo-1566478989037-eec170784d0b?auto=format&fit=crop&q=80&w=200" }
        ];

        // مفسر CSV ذكي ومقاوم للأعطال والمسافات الزائدة
        function parseCSVLine(line, delimiter) {
            const result = [];
            let current = '';
            let inQuotes = false;
            for (let i = 0; i < line.length; i++) {
                const char = line[i];
                if (char === '"') {
                    inQuotes = !inQuotes;
                } else if (char === delimiter && !inQuotes) {
                    result.push(current);
                    current = '';
                } else {
                    current += char;
                }
            }
            result.push(current);
            return result;
        }

        /* */
        // تحميل البيانات الفورية من جوجل شيتس كملف CSV مباشر وتنقيتها لمنع التكرار ومنع الأخطاء نهائياً
        async function loadData() {
            const errorPanel = document.getElementById('debug-error-panel');
            const errorMsg = document.getElementById('debug-error-message');
            
            try {
                // رابطك الحقيقي والسليم والمباشر من جوجل شيتس
                const sheetUrl = 'https://docs.google.com/spreadsheets/d/e/2PACX-1vRaS65gcu4u7LSt9qfn6AVtJgqRoHQ_P8iQC4GmOEsqE21fNZ4Wl-EXBuK0sZMvj8a1KXmXZtM3EEZi/pub?output=csv';
                
                const response = await fetch(sheetUrl);
                if (!response.ok) throw new Error(`فشل الاتصال بجوجل شيتس (كود الاستجابة: ${response.status})`);
                
                const text = await response.text();
                
                // تنظيف الملف بالكامل لمنع تكرار الأقسام والمنتجات
                const cleanText = text.replace(/\r/g, "");
                const lines = cleanText.split('\n');
                
                if (lines.length === 0 || (lines.length === 1 && lines[0].trim() === "")) {
                    throw new Error("الجدول فارغ تماماً ولا يحتوي على أي بيانات!");
                }

                // كاشف الفواصل الذكي للغاية والمحدث:
                // نكتشف الفاصلة المستخدمة بناءً على فحص دقيق للسطر الأول من جدولك
                const firstLine = lines[0];
                const commaCount = (firstLine.match(/,/g) || []).length;
                const semicolonCount = (firstLine.match(/;/g) || []).length;
                
                const delimiter = semicolonCount > commaCount ? ';' : ',';
                console.log(`Detected CSV Delimiter: [${delimiter}] (Commas: ${commaCount}, Semicolons: ${semicolonCount})`);
                
                allData = lines.slice(1).filter(r => r.trim() !== "").map((row, index) => {
                    const columns = parseCSVLine(row, delimiter);
                    if (columns.length < 2) return null; // تخطي الأسطر غير المكتملة
                    
                    const id = columns[0] ? columns[0].trim() : "prod-" + index;
                    const name = columns[1] ? columns[1].trim() : "";
                    
                    // تحويل السعر بأمان وحذف أي علامات أو مسافات
                    const rawPrice = columns[2] ? columns[2].replace(/[^\d.]/g, '') : "0";
                    const price = parseFloat(rawPrice) || 0;
                    
                    // حل ذكي: إذا كان القسم فارغاً نضعه في قسم افتراضي فخم بدلاً من حذفه
                    let cat = columns[3] ? columns[3].replace(/\s+/g, " ").trim() : "";
                    if (!cat) {
                        cat = "منتجات متنوعة";
                    }
                    
                    // رابط الصورة، وإذا كان فارغاً نستخدم صورة بديلة
                    let img = columns[4] ? columns[4].trim() : "";
                    if (!img || img === "") {
                        img = "https://placehold.co/150x150/e0f2f1/064e3b?text=" + encodeURIComponent(name);
                    }
                    
                    return { id, name, price, cat, img };
                }).filter(p => p && p.name && p.name.trim() !== ""); // نفلتر فقط المنتجات التي لها اسم فعلي
                
                if (allData.length === 0) {
                    throw new Error("لم نتمكن من العثور على أي منتجات صالحة في جدولك، يرجى التحقق من تسمية الأعمدة في الصف الأول.");
                }

                // إخفاء لوحة الأخطاء إذا كانت معروضة ونجحت القراءة الحقيقية
                errorPanel.classList.add('hidden');

                // البحث التلقائي عن أي فئة تحتوي على كلمات عروض/خصومات في الجدول
                const foundOffer = allData.find(p => p.cat.includes('عروض') || p.cat.includes('عرض') || p.cat.includes('خصم') || p.cat.includes('خصومات'));
                if(foundOffer) {
                    offersCategoryName = foundOffer.cat;
                } else {
                    offersCategoryName = "عروض اليوم";
                }
                
                // تحديث اسم الزر الأحمر السفلي ليتطابق فورياً مع الاسم المكتوب بجدولك
                document.getElementById('offers-btn-text').innerText = offersCategoryName;

                renderCategories();
                renderMarquee();
            } catch (e) { 
                console.error("Error loading data, using fallback: ", e); 
                // إظهار لوحة فحص وتشخيص الأخطاء المباشرة على الشاشة
                errorPanel.classList.remove('hidden');
                errorMsg.innerText = e.message;

                // تفعيل خط الدفاع الأخير لتشغيل المتجر بالمنتجات الافتراضية لكي لا يتوقف أبداً
                allData = defaultFallbackProducts;
                offersCategoryName = "عروض اليوم";
                document.getElementById('offers-btn-text').innerText = "عروض اليوم";
                
                renderCategories();
                renderMarquee();
            }
        }

        // بناء شريط "وصل حديثاً" الدوار بالتناخم مع السلع ليلف للأبد (من الشمال لليمين)
        function renderMarquee() {
            const track = document.getElementById('marquee-track');
            const newArrivals = allData.filter(p => p.cat.includes('حديث') || p.cat.includes('جديد'));
            let content = '';
            
            if (newArrivals.length === 0) {
                const placeholders = [
                    { name: "كفتة كبير فاخرة", price: 78 },
                    { name: "كيمو كونو فانيليا طازج", price: 10 },
                    { name: "أرز الشعلة كيس 1 ك", price: 30 }
                ];
                content = placeholders.map(p => `
                    <span class="text-xs bg-emerald-900 border border-emerald-800 text-emerald-100 px-3 py-1.5 rounded-full flex items-center gap-1.5 shadow-sm font-semibold whitespace-nowrap">
                        <span class="w-1.5 h-1.5 rounded-full bg-emerald-400"></span>
                        ${p.name} - ${p.price} ج.م
                    </span>
                `).join('');
            } else {
                content = newArrivals.map(p => `
                    <div onclick="event.stopPropagation(); addDirectFromMarquee('${p.name.replace(/'/g, "\\'")}', ${p.price})" class="text-xs bg-emerald-900 hover:bg-emerald-800 border border-emerald-800 text-emerald-100 px-3 py-1.5 rounded-full flex items-center gap-2 cursor-pointer transition-colors shadow-sm font-semibold whitespace-nowrap">
                        <img src="${p.img}" onerror="this.src='https://placehold.co/50x50/e0f2f1/064e3b?text=🛍️'" class="w-4 h-4 rounded-full object-cover">
                        <span>${p.name}</span>
                        <span class="text-amber-300 font-bold">${p.price} ج.م</span>
                    </div>
                `).join('');
            }

            track.innerHTML = `
                <div class="flex gap-6 shrink-0">${content}</div>
                <div class="flex gap-6 shrink-0">${content}</div>
            `;
            
            track.style.animation = 'none';
            track.offsetHeight;
            track.style.animation = '';
        }

        function addDirectFromMarquee(name, price) {
            updateCart(name, price, 1);
        }

        function openNewArrivalsCategory() {
            const foundCat = allData.find(p => p.cat.includes('حديث') || p.cat.includes('جديد'));
            if(foundCat) {
                renderProducts(foundCat.cat);
            } else {
                document.getElementById('cat-title').innerText = "وصل حديثاً";
                const grid = document.getElementById('products-grid');
                grid.innerHTML = '';
                
                const placeholders = [
                    { name: "كفتة كبير فاخرة", price: 78, img: 'https://placehold.co/150x150/e0f2f1/064e3b?text=كفتة' },
                    { name: "كيمو كونو فانيليا", price: 10, img: 'https://placehold.co/150x150/e0f2f1/064e3b?text=آيس+كريم' },
                    { name: "أرز الشعلة", price: 30, img: 'https://placehold.co/150x150/e0f2f1/064e3b?text=أرز' }
                ];
                
                placeholders.forEach((p, idx) => {
                    const uniqueId = `ph-${idx}`;
                    const qty = cart[p.name]?.qty || 0;
                    grid.innerHTML += `
                        <div class="bg-white p-3 rounded-2xl border border-emerald-100 shadow-md flex flex-col justify-between">
                            <div>
                                <img src="${p.img}" onerror="this.src='https://placehold.co/150x150/e0f2f1/064e3b?text=منتج'" class="w-full h-28 object-cover rounded-xl mb-2 shadow-sm">
                                <h3 class="font-bold text-xs text-emerald-950 min-h-[32px] line-clamp-2 leading-relaxed mb-1">${p.name}</h3>
                                <p class="text-emerald-600 font-bold text-sm mb-3">${p.price} ج.م</p>
                            </div>
                            <div class="flex items-center justify-between bg-emerald-50 p-1 rounded-xl">
                                <button onclick="updateCart('${p.name.replace(/'/g, "\\'")}', ${p.price}, -1, '${uniqueId}')" class="bg-white w-8 h-8 rounded-lg shadow text-emerald-950 font-bold hover:bg-emerald-100 active:scale-90 transition-all">-</button>
                                <span id="qty-${uniqueId}" class="font-bold text-emerald-950 text-sm px-1">${qty}</span>
                                <button onclick="updateCart('${p.name.replace(/'/g, "\\'")}', ${p.price}, 1, '${uniqueId}')" class="bg-emerald-600 w-8 h-8 rounded-lg text-white font-bold hover:bg-emerald-700 active:scale-90 transition-all">+</button>
                            </div>
                        </div>`;
                });
                showPage('products-page');
            }
        }

        // دالة إرجاع الأيقونة المناسبة لاسم القسم
        function getCategoryIcon(cat) {
            const name = cat.toLowerCase();
            if (name.includes('أرز') || name.includes('ارز') || name.includes('مكرون') || name.includes('حبوب') || name.includes('رز') || name.includes('دقيق')) return '🌾'; 
            if (name.includes('جبن') || name.includes('ألبان') || name.includes('البان') || name.includes('زبادي') || name.includes('سمن') || name.includes('زبد') || name.includes('قشطة')) return '🧀'; 
            if (name.includes('لبن') || name.includes('حليب') || name.includes('مشروب')) return '🥛'; 
            if (name.includes('آيس') || name.includes('ايس') || name.includes('كريم') || name.includes('جيلاتو') || name.includes('كيمو') || name.includes('ميجا')) return '🍦';
            if (name.includes('زيت') || name.includes('زيوت') || name.includes('خل')) return '🍾'; 
            if (name.includes('منظف') || name.includes('غسيل') || name.includes('صابون') || name.includes('شامبو')) return '🧼'; 
            if (name.includes('تونة') || name.includes('تونه') || name.includes('سمك') || name.includes('معلبات') || name.includes('فول')) return '🥫'; 
            if (name.includes('شيبس') || name.includes('مقرمشات') || name.includes('كراتيه') || name.includes('بسكويت') || name.includes('حلويات') || name.includes('شوكول')) return '🍟'; 
            if (name.includes('خضار') || name.includes('خضروات') || name.includes('فاكهة') || name.includes('طماطم') || name.includes('بطاطس')) return '🥦'; 
            if (name.includes('لحوم') || name.includes('فراخ') || name.includes('دجاج') || name.includes('مجمدا') || name.includes('كفتة') || name.includes('برجر') || name.includes('سجق')) return '🥩'; 
            if (name.includes('توابل') || name.includes('بهارات') || name.includes('ملح') || name.includes('سكر') || name.includes('شاي')) return '🧂'; 
            if (name.includes('غازية') || name.includes('كولا') || name.includes('بيبسي') || name.includes('عصير') || name.includes('عصائر')) return '🥤';
            if (name.includes('عروض') || name.includes('عرض') || name.includes('خصم')) return '🔥'; 
            return '🛍️'; 
        }

        /* */
        // عرض قائمة الأقسام المتاحة بالمتجر بدون تكرار
        function renderCategories() {
            const grid = document.getElementById('categories-grid');
            const uniqueCats = [...new Set(allData.map(p => p.cat))];
            grid.innerHTML = '';
            
            // تصفية الأقسام: إخفاء الأقسام الجديدة وتصفية وإخفاء عروض اليوم بالكامل لتكون بالأسفل فقط!
            const filteredCats = uniqueCats.filter(c => 
                !c.includes('حديث') && 
                !c.includes('جديد') &&
                !c.includes('عروض') &&
                !c.includes('عرض') &&
                !c.includes('خصم') &&
                !c.includes('خصومات')
            );

            // ترتيب الأقسام أبجدياً
            filteredCats.sort((a, b) => a.localeCompare(b, 'ar'));

            if(filteredCats.length === 0) {
                grid.innerHTML = `<div class="col-span-2 text-center text-emerald-600 font-semibold py-8">لا توجد أقسام عادية متوفرة حالياً بالجدول.</div>`;
                return;
            }

            filteredCats.forEach(c => {
                const icon = getCategoryIcon(c);
                grid.innerHTML += `
                    <div onclick="renderProducts('${c}')" class="bg-white p-6 rounded-2xl text-center shadow-md border border-emerald-100 hover:border-emerald-300 active:scale-95 hover:shadow-lg transition-all duration-300 cursor-pointer flex flex-col justify-center items-center min-h-[110px]">
                        <span class="text-3xl mb-1.5">${icon}</span>
                        <h3 class="font-bold text-emerald-950 text-xs leading-snug">${c}</h3>
                    </div>`;
            });
        }

        function openOffersCategory() {
            renderProducts(offersCategoryName);
        }

        /* */
        // عرض قائمة المنتجات داخل قسم محدد
        function renderProducts(cat) {
            document.getElementById('cat-title').innerText = cat;
            const grid = document.getElementById('products-grid');
            grid.innerHTML = '';
            
            const products = allData.filter(p => p.cat === cat);
            
            if(products.length === 0) {
                grid.innerHTML = `<p class="col-span-2 text-center text-emerald-600 py-12 font-bold">لا توجد منتجات متوفرة حالياً في هذا القسم.</p>`;
                if (document.getElementById('products-page').classList.contains('hidden')) {
                    showPage('products-page');
                }
                return;
            }

            products.forEach((p, idx) => {
                const uniqueId = p.id ? p.id.toString().replace(/[^a-zA-Z0-9]/g, "-") : `idx-${idx}`;
                const qty = cart[p.name]?.qty || 0;
                grid.innerHTML += `
                    <div class="bg-white p-3 rounded-2xl border border-emerald-100 shadow-md flex flex-col justify-between">
                        <div>
                            <img src="${p.img}" onerror="this.src='https://placehold.co/150x150/e0f2f1/064e3b?text=منتج'" class="w-full h-28 object-cover rounded-xl mb-2 shadow-sm">
                            <h3 class="font-bold text-xs text-emerald-950 min-h-[32px] line-clamp-2 leading-relaxed mb-1">${p.name}</h3>
                            <p class="text-emerald-600 font-bold text-sm mb-3">${p.price} ج.م</p>
                        </div>
                        <div class="flex items-center justify-between bg-emerald-50 p-1 rounded-xl">
                            <button onclick="updateCart('${p.name.replace(/'/g, "\\'")}', ${p.price}, -1, '${uniqueId}')" class="bg-white w-8 h-8 rounded-lg shadow text-emerald-950 font-bold hover:bg-emerald-100 active:scale-90 transition-all">-</button>
                            <span id="qty-${uniqueId}" class="font-bold text-emerald-950 text-sm px-1">${qty}</span>
                            <button onclick="updateCart('${p.name.replace(/'/g, "\\'")}', ${p.price}, 1, '${uniqueId}')" class="bg-emerald-600 w-8 h-8 rounded-lg text-white font-bold hover:bg-emerald-700 active:scale-90 transition-all">+</button>
                        </div>
                    </div>`;
            });

            if (document.getElementById('products-page').classList.contains('hidden')) {
                showPage('products-page');
            }
        }

        /* */
        // إدارة السلة وتحديث الكميات بشكل لحظي ومثالي
        function updateCart(name, price, delta, domId) {
            if(!cart[name]) cart[name] = { price, qty: 0 };
            cart[name].qty = Math.max(0, cart[name].qty + delta);
            const currentQty = cart[name].qty;
            if(cart[name].qty === 0) delete cart[name];
            
            // إرسال رسالة "تم الإضافة بنجاح" تظل لمدة 4 ثوانٍ كاملة عند الإضافة (+)
            if (delta > 0) {
                showToast("تم الإضافة بنجاح 🛒", "success", 4000);
            }
            
            // تحديث DOM للعنصر المحدد فقط لمنع اهتزاز الصفحة أو إعادة جلب الصور
            if (domId) {
                const qtySpan = document.getElementById(`qty-${domId}`);
                if (qtySpan) {
                    qtySpan.innerText = currentQty;
                }
            } else {
                if(!document.getElementById('products-page').classList.contains('hidden')) {
                    renderProducts(document.getElementById('cat-title').innerText);
                }
            }
            
            renderCart();
            updateCartBadge();
        }

        function updateCartBadge() {
            const badge = document.getElementById('cart-badge');
            const totalItems = Object.values(cart).reduce((sum, item) => sum + item.qty, 0);
            if (totalItems > 0) {
                badge.innerText = totalItems;
                badge.classList.remove('scale-0');
                badge.classList.add('scale-100');
            } else {
                badge.classList.remove('scale-100');
                badge.classList.add('scale-0');
            }
        }

        function renderCart() {
            const itemsDiv = document.getElementById('cart-items');
            const entries = Object.entries(cart);
            
            if(entries.length === 0) {
                itemsDiv.innerHTML = `<div class="text-center py-12 text-emerald-600 font-bold">🛒 سلتك فارغة حالياً. ابدأ بإضافة المنتجات!</div>`;
                return;
            }

            let total = 0;
            itemsDiv.innerHTML = entries.map(([name, item]) => {
                const subtotal = item.price * item.qty;
                total += subtotal;
                return `
                    <div class="flex justify-between items-center bg-white p-4 rounded-2xl border border-emerald-50 shadow-sm">
                        <div class="flex-1 ml-3">
                            <h4 class="font-bold text-xs text-emerald-950">${name}</h4>
                            <p class="text-xs text-emerald-600 font-semibold mt-1">${item.price} ج.م × ${item.qty}</p>
                        </div>
                        <div class="flex items-center gap-2">
                            <div class="flex items-center bg-emerald-50 p-1 rounded-xl">
                                <button onclick="updateCart('${name.replace(/'/g, "\\'")}', ${item.price}, -1)" class="bg-white w-7 h-7 rounded-lg shadow text-emerald-950 font-bold text-xs">-</button>
                                <span class="font-bold text-emerald-950 text-xs px-2">${item.qty}</span>
                                <button onclick="updateCart('${name.replace(/'/g, "\\'")}', ${item.price}, 1)" class="bg-emerald-600 w-7 h-7 rounded-lg text-white font-bold text-xs">+</button>
                            </div>
                            <span class="font-bold text-emerald-950 text-xs min-w-[55px] text-left">${subtotal} ج.م</span>
                        </div>
                    </div>`;
            }).join('');

            itemsDiv.innerHTML += `
                <div class="flex justify-between items-center bg-emerald-100/50 p-4 rounded-2xl border border-emerald-200 mt-4">
                    <span class="font-bold text-emerald-950 text-sm">💰 الحساب الإجمالي:</span>
                    <span class="font-extrabold text-emerald-800 text-lg">${total} ج.م</span>
                </div>`;
        }

        // دالة إرسال الطلب المطورّة التي ترسل فاتورة رقمية منسقة للواتساب
        function confirmOrder() {
            const name = document.getElementById('cust-name').value.trim();
            const phone = document.getElementById('cust-phone').value.trim();
            const addr = document.getElementById('cust-addr').value.trim();
            const notes = document.getElementById('cust-notes').value.trim();
            
            const entries = Object.entries(cart);
            if(entries.length === 0) {
                showToast("🔴 سلتك فارغة! أضف بعض السلع أولاً", "error", 4000);
                return;
            }

            if(!name || name.length < 5) {
                showToast("🔴 يرجى إدخال اسمك الكامل بشكل صحيح", "error", 4000);
                document.getElementById('cust-name').focus();
                return;
            }

            if(!phone || !/^01[0125][0-9]{8}$/.test(phone)) {
                showToast("🔴 يرجى إدخال رقم تليفون صحيح مكون من 11 رقم ويبدأ بـ 01", "error", 4000);
                document.getElementById('cust-phone').focus();
                return;
            }

            if(!addr || addr.length < 8) {
                showToast("🔴 يرجى كتابة العنوان بالتفصيل لضمان سرعة التوصيل", "error", 4000);
                document.getElementById('cust-addr').focus();
                return;
            }

            let total = 0;
            let totalQty = 0; // لحساب إجمالي عدد القطع في الطلبية
            
            // بناء رسالة الفاتورة المطورّة والمريحة في القراءة
            let msg = `🧾 *فـاتـورة طـلـب جـديـد | أسواق الأدهم* 🧾%0A`;
            msg += `============================%0A%0A`;
            msg += `👤 *بـيـانـات الـعـمـيـل:*%0A`;
            msg += ` 🧑‍💼 *الاسم:* ${name}%0A`;
            msg += ` 📞 *الهاتف:* ${phone}%0A`;
            msg += ` 📍 *العنوان:* ${addr}%0A`;
            if(notes) msg += ` 📝 *ملاحظات:* ${notes}%0A`;
            msg += `%0A============================%0A%0A`;
            msg += `🛒 *الـمـنـتـجـات الـمـطـلـوبـة:*%0A%0A`;

            entries.forEach(([n, item]) => {
                const subtotal = item.price * item.qty;
                total += subtotal;
                totalQty += item.qty;
                msg += `📦 *${n}*%0A`;
                msg += ` 🔢 العدد: ${item.qty} × 💰 S.P: ${item.price} ج.م%0A`;
                msg += ` 💵 الحساب: ${subtotal} ج.م%0A`;
                msg += `----------------------------%0A`;
            });

            msg += `%0A📊 *مـلـخـص الـطـلـبـيـة:*%0A`;
            msg += `🧮 *إجمالي عدد السلع:* ${totalQty} قطعة%0A`;
            msg += `💵 *الحساب الإجمالي:* *${total} ج.م*%0A%0A`;
            msg += `============================%0A`;
            msg += `⏱️ *وقت إرسال الطلب:* ${new Date().toLocaleString('ar-EG')}%0A%0A`;
            msg += `✨ *شكراً لتسوقكم معنا في أسواق الأدهم* ✨`;

            const newPurchase = {
                id: Date.now(),
                date: new Date().toLocaleDateString('ar-EG'),
                time: new Date().toLocaleTimeString('ar-EG'),
                total: total,
                items: entries.map(([n, item]) => ({
                    name: n,
                    price: item.price,
                    qty: item.qty
                }))
            };
            purchases.unshift(newPurchase);
            localStorage.setItem('purchases', JSON.stringify(purchases));

            cart = {};
            localStorage.removeItem('cart');
            updateCartBadge();
            renderCart();
            
            window.open(`https://wa.me/201093732952?text=${msg}`);
        }

        function renderPurchases() {
            const container = document.getElementById('purchases-items');
            if(purchases.length === 0) {
                container.innerHTML = `<div class="text-center py-12 text-emerald-600 font-bold">📜 ليس لديك أي مشتريات سابقة حالياً.</div>`;
                return;
            }

            container.innerHTML = purchases.map(p => `
                <div class="bg-white p-5 rounded-3xl border border-emerald-50 shadow-md relative">
                    <div class="flex justify-between items-center mb-3">
                        <div class="text-xs text-gray-400">${p.date} - ${p.time}</div>
                        <div class="flex gap-2">
                            <button onclick="reorderPurchase(${p.id})" class="text-white hover:bg-emerald-700 text-xs font-bold bg-emerald-600 px-3 py-1.5 rounded-xl transition-all">الطلب من جديد 🔁</button>
                            <button onclick="deletePurchase(${p.id})" class="text-red-500 hover:text-red-750 text-xs font-bold bg-red-50 hover:bg-red-100 px-3 py-1.5 rounded-xl transition-all">حذف 🗑️</button>
                        </div>
                    </div>
                    <div class="space-y-1 mb-3">
                        ${p.items.map(item => `<div class="text-emerald-950 font-bold text-sm">• ${item.name} (${item.qty})</div>`).join('')}
                    </div>
                    <div class="border-t border-emerald-50 pt-2 flex justify-between items-center">
                        <span class="text-xs font-bold text-emerald-900">المبلغ المدفوع:</span>
                        <span class="text-sm font-extrabold text-emerald-700">${p.total} ج.م</span>
                    </div>
                </div>`).join('');
        }

        function reorderPurchase(id) {
            const purchase = purchases.find(p => p.id === id);
            if (!purchase) return;

            purchase.items.forEach(item => {
                if (!cart[item.name]) {
                    cart[item.name] = { price: item.price, qty: 0 };
                }
                cart[item.name].qty += item.qty;
            });

            updateCartBadge();
            renderCart();
            showPage('cart-page');
            showToast("🛒 تم تكرار طلبك السابق وإضافته للسلة بنجاح!", "success", 4000);
        }

        function deletePurchase(id) {
            purchases = purchases.filter(p => p.id !== id);
            localStorage.setItem('purchases', JSON.stringify(purchases));
            renderPurchases();
            showToast("تم حذف الطلب من الأرشيف بنجاح", "success", 4000);
        }

        function showPage(pageId) {
            document.querySelectorAll('main > div').forEach(div => div.classList.add('hidden'));
            document.getElementById(pageId).classList.remove('hidden');
            
            document.querySelectorAll('#bottom-nav button').forEach(btn => {
                btn.classList.add('text-emerald-100');
                btn.classList.remove('text-white');
            });
            
            if(pageId === 'cart-page') renderCart();
            if(pageId === 'purchases-page') renderPurchases();
            
            window.scrollTo({ top: 0, behavior: 'smooth' });
        }

        function goBack() {
            showPage('categories-page');
        }

        // بدء تحميل البيانات مباشرة عند تشغيل الصفحة
        loadData();
    </script>
</body>
</html>
