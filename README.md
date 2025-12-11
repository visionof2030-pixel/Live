<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>مباريات اليوم - كرة القدم</title>
    <style>
        /* تنسيق عام للصفحة */
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }

        body {
            background-color: #f5f5f5; /* خلفية رمادية فاتحة */
            color: #333;
            padding: 20px;
            line-height: 1.6;
            max-width: 1200px;
            margin: 0 auto;
        }

        .header {
            text-align: center;
            margin-bottom: 30px;
            padding-bottom: 20px;
            border-bottom: 1px solid #ddd;
        }

        h1 {
            color: #2c3e50;
            margin-bottom: 15px;
        }

        .date-display {
            color: #7f8c8d;
            font-size: 1.1em;
            margin-bottom: 20px;
        }

        /* تنسيق زر التحديث */
        .refresh-btn {
            background-color: #3498db;
            color: white;
            border: none;
            padding: 12px 25px;
            font-size: 1em;
            border-radius: 6px;
            cursor: pointer;
            transition: background-color 0.3s;
            margin: 10px 0;
        }

        .refresh-btn:hover {
            background-color: #2980b9;
        }

        /* منطقة الرسائل */
        .message {
            padding: 15px;
            margin: 20px 0;
            border-radius: 6px;
            text-align: center;
            display: none; /* مخفية افتراضياً */
        }

        .error {
            background-color: #ffebee;
            color: #c62828;
            border: 1px solid #ef9a9a;
        }

        .loading {
            background-color: #e3f2fd;
            color: #1565c0;
            border: 1px solid #90caf9;
        }

        /* تنسيق منطقة المباريات */
        .fixtures-container {
            display: flex;
            flex-direction: column;
            gap: 20px;
        }

        /* تنسيق بطاقة المباراة */
        .fixture-card {
            background-color: white;
            border-radius: 10px;
            padding: 20px;
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.05);
            display: flex;
            flex-direction: column;
            transition: transform 0.3s, box-shadow 0.3s;
        }

        .fixture-card:hover {
            transform: translateY(-5px);
            box-shadow: 0 6px 12px rgba(0, 0, 0, 0.1);
        }

        /* بطاقات الدوريات المهمة - لون ذهبي خفيف */
        .fixture-card.important {
            background-color: #fffbf0;
            border-right: 4px solid #ffd700;
        }

        /* رأس البطاقة - اسم الدوري */
        .fixture-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 15px;
            padding-bottom: 10px;
            border-bottom: 1px dashed #eee;
        }

        .league-name {
            font-weight: bold;
            color: #2c3e50;
            font-size: 1.2em;
        }

        .match-status {
            padding: 5px 10px;
            border-radius: 20px;
            font-size: 0.85em;
            font-weight: bold;
        }

        .status-not-started {
            background-color: #ecf0f1;
            color: #7f8c8d;
        }

        .status-live {
            background-color: #e8f5e9;
            color: #2e7d32;
            animation: pulse 1.5s infinite;
        }

        .status-finished {
            background-color: #f3e5f5;
            color: #6a1b9a;
        }

        @keyframes pulse {
            0% { opacity: 1; }
            50% { opacity: 0.7; }
            100% { opacity: 1; }
        }

        /* جسم البطاقة - الفريقان والوقت */
        .fixture-body {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 10px 0;
        }

        .team {
            display: flex;
            flex-direction: column;
            align-items: center;
            width: 40%;
        }

        .team-name {
            font-weight: bold;
            margin-top: 8px;
            text-align: center;
        }

        .team-logo {
            width: 40px;
            height: 40px;
            object-fit: contain;
        }

        .vs-section {
            display: flex;
            flex-direction: column;
            align-items: center;
            width: 20%;
        }

        .match-time {
            font-weight: bold;
            color: #2c3e50;
            font-size: 1.1em;
            margin-bottom: 5px;
        }

        .vs-text {
            color: #95a5a6;
            font-size: 0.9em;
        }

        /* تذييل البطاقة */
        .fixture-footer {
            margin-top: 15px;
            padding-top: 10px;
            border-top: 1px dashed #eee;
            color: #7f8c8d;
            font-size: 0.9em;
            text-align: center;
        }

        /* تصميم متجاوب للهواتف */
        @media (max-width: 768px) {
            body {
                padding: 15px;
            }
            
            .fixture-body {
                flex-direction: column;
                gap: 15px;
            }
            
            .team {
                width: 100%;
                flex-direction: row;
                justify-content: center;
                gap: 15px;
            }
            
            .team-name {
                margin-top: 0;
            }
            
            .vs-section {
                width: 100%;
                margin: 10px 0;
            }
        }
    </style>
</head>
<body>
    <div class="header">
        <h1>⚽ مباريات اليوم</h1>
        <div class="date-display" id="currentDate">جاري تحميل التاريخ...</div>
        <button class="refresh-btn" id="refreshBtn">🔄 تحديث البيانات</button>
    </div>

    <!-- منطقة عرض رسائل الحالة -->
    <div class="message loading" id="loadingMessage">
        جاري جلب بيانات المباريات...
    </div>
    
    <div class="message error" id="errorMessage">
        <!-- سيتم تعبئة رسالة الخطأ هنا بشكل ديناميكي -->
    </div>

    <!-- منطقة عرض المباريات -->
    <div class="fixtures-container" id="fixturesContainer">
        <!-- سيتم تعبئة المباريات هنا بشكل ديناميكي -->
    </div>

    <script>
        // عناصر DOM الرئيسية
        const fixturesContainer = document.getElementById('fixturesContainer');
        const loadingMessage = document.getElementById('loadingMessage');
        const errorMessage = document.getElementById('errorMessage');
        const refreshBtn = document.getElementById('refreshBtn');
        const currentDateElement = document.getElementById('currentDate');
        
        // متغيرات التطبيق
        let currentDate = new Date();
        const importantLeagues = ["Premier League", "La Liga", "Serie A", "Bundesliga", "Ligue 1"];
        
        // ========== دالة التهيئة الرئيسية ==========
        function init() {
            // عرض تاريخ اليوم
            displayCurrentDate();
            
            // إضافة مستمع حدث لزر التحديث
            refreshBtn.addEventListener('click', fetchFixtures);
            
            // جلب المباريات عند تحميل الصفحة
            fetchFixtures();
        }
        
        // ========== عرض التاريخ الحالي ==========
        function displayCurrentDate() {
            const options = { 
                weekday: 'long', 
                year: 'numeric', 
                month: 'long', 
                day: 'numeric' 
            };
            currentDateElement.textContent = currentDate.toLocaleDateString('ar-SA', options);
        }
        
        // ========== دالة جلب بيانات المباريات من API ==========
        async function fetchFixtures() {
            // إظهار رسالة التحميل وإخفاء رسالة الخطأ
            showLoading();
            hideError();
            
            // تهيئة تاريخ اليوم بالتنسيق YYYY-MM-DD
            const year = currentDate.getFullYear();
            const month = String(currentDate.getMonth() + 1).padStart(2, '0');
            const day = String(currentDate.getDate()).padStart(2, '0');
            const today = `${year}-${month}-${day}`;
            
            // بناء URL الخاص بـ API مع تاريخ اليوم
            const apiUrl = `https://api-football-v1.p.rapidapi.com/v3/fixtures?date=${today}`;
            
            try {
                // إعداد رؤوس الطلب كما هو مطلوب من RapidAPI
                const headers = {
                    'X-RapidAPI-Key': '', // ضع هنا rapidapi key
                    'X-RapidAPI-Host': 'api-football-v1.p.rapidapi.com'
                };
                
                // إجراء طلب fetch إلى API
                const response = await fetch(apiUrl, { method: 'GET', headers: headers });
                
                // التحقق من نجاح الاستجابة
                if (!response.ok) {
                    throw new Error(`خطأ في الشبكة: ${response.status}`);
                }
                
                // تحويل الاستجابة إلى JSON
                const data = await response.json();
                
                // التحقق من وجود مباريات
                if (data.response && data.response.length > 0) {
                    displayFixtures(data.response);
                } else {
                    throw new Error('لا توجد مباريات اليوم');
                }
                
                // إخفاء رسالة التحميل
                hideLoading();
                
            } catch (error) {
                // معالجة الأخطاء وعرض رسالة مناسبة
                hideLoading();
                showError(`حدث خطأ: ${error.message}`);
                console.error('تفاصيل الخطأ:', error);
            }
        }
        
        // ========== دالة عرض المباريات في الصفحة ==========
        function displayFixtures(fixtures) {
            // مسح المحتوى القديم
            fixturesContainer.innerHTML = '';
            
            // ترتيب المباريات من الأقدم إلى الأحدث حسب الوقت
            fixtures.sort((a, b) => {
                const timeA = new Date(a.fixture.date);
                const timeB = new Date(b.fixture.date);
                return timeA - timeB;
            });
            
            // إنشاء بطاقة لكل مباراة
            fixtures.forEach(fixture => {
                const fixtureCard = createFixtureCard(fixture);
                fixturesContainer.appendChild(fixtureCard);
            });
        }
        
        // ========== دالة إنشاء بطاقة مباراة ==========
        function createFixtureCard(fixtureData) {
            // استخراج البيانات من الاستجابة
            const leagueName = fixtureData.league.name;
            const homeTeam = fixtureData.teams.home;
            const awayTeam = fixtureData.teams.away;
            const fixture = fixtureData.fixture;
            
            // إنشاء عنصر البطاقة
            const card = document.createElement('div');
            card.className = 'fixture-card';
            
            // التحقق إذا كانت الدوري من الدوريات المهمة
            if (importantLeagues.includes(leagueName)) {
                card.classList.add('important');
            }
            
            // تحديد حالة المباراة
            let statusText, statusClass;
            if (fixture.status.short === 'NS') {
                statusText = 'لم تبدأ';
                statusClass = 'status-not-started';
            } else if (fixture.status.short === 'LIVE' || fixture.status.short === 'HT' || fixture.status.short === '1H' || fixture.status.short === '2H') {
                statusText = 'مباشر';
                statusClass = 'status-live';
            } else if (fixture.status.short === 'FT' || fixture.status.short === 'AET' || fixture.status.short === 'PEN') {
                statusText = 'انتهت';
                statusClass = 'status-finished';
            } else {
                statusText = fixture.status.long;
                statusClass = 'status-not-started';
            }
            
            // تنسيق الوقت
            const matchDate = new Date(fixture.date);
            const timeString = matchDate.toLocaleTimeString('ar-SA', { 
                hour: '2-digit', 
                minute: '2-digit',
                hour12: true 
            });
            
            // بناء HTML للبطاقة
            card.innerHTML = `
                <div class="fixture-header">
                    <div class="league-name">${leagueName}</div>
                    <div class="match-status ${statusClass}">${statusText}</div>
                </div>
                <div class="fixture-body">
                    <div class="team">
                        <img src="${homeTeam.logo}" alt="${homeTeam.name}" class="team-logo" onerror="this.src='https://via.placeholder.com/40?text=H'">
                        <div class="team-name">${homeTeam.name}</div>
                    </div>
                    <div class="vs-section">
                        <div class="match-time">${timeString}</div>
                        <div class="vs-text">vs</div>
                    </div>
                    <div class="team">
                        <img src="${awayTeam.logo}" alt="${awayTeam.name}" class="team-logo" onerror="this.src='https://via.placeholder.com/40?text=A'">
                        <div class="team-name">${awayTeam.name}</div>
                    </div>
                </div>
                <div class="fixture-footer">
                    ${fixture.venue.name ? `📍 ${fixture.venue.name}` : ''}
                </div>
            `;
            
            return card;
        }
        
        // ========== دوال التحكم بالرسائل ==========
        function showLoading() {
            loadingMessage.style.display = 'block';
        }
        
        function hideLoading() {
            loadingMessage.style.display = 'none';
        }
        
        function showError(message) {
            errorMessage.textContent = message;
            errorMessage.style.display = 'block';
        }
        
        function hideError() {
            errorMessage.style.display = 'none';
        }
        
        // بدء التطبيق عند تحميل الصفحة
        document.addEventListener('DOMContentLoaded', init);
    </script>
</body>
</html>
