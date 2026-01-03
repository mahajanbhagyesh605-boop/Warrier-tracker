<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Ninja Mastery 365</title>
    <link rel="manifest" href="manifest.json">
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=VT323&family=Inter:wght@400;900&display=swap" rel="stylesheet">
    <style>
        body { background-color: #050505; color: #e5e5e5; font-family: 'Inter', sans-serif; overflow-x: hidden; }
        .ninja-card { background: #111; border: 1px solid #222; border-radius: 16px; transition: all 0.3s ease; }
        .accent-green { color: #00ff88; }
        .bg-accent { background-color: #00ff88; }
        .rank-glow { text-shadow: 0 0 12px rgba(0, 255, 136, 0.7); font-family: 'VT323', monospace; }
        .xp-bar-container { background: #000; border: 1px solid #333; height: 10px; border-radius: 5px; overflow: hidden; }
        .xp-bar-fill { height: 100%; background: linear-gradient(90deg, #00ff88, #0088ff); transition: width 0.8s cubic-bezier(0.17, 0.67, 0.83, 0.67); }
        .penalty-red { color: #ff4444; }
    </style>
</head>
<body class="p-4 max-w-4xl mx-auto pb-10">

    <audio id="slashSound" src="https://assets.mixkit.co/sfx/preview/mixkit-fast-sword-whoosh-1507.mp3" preload="auto"></audio>
    <audio id="submitSound" src="https://assets.mixkit.co/sfx/preview/mixkit-medieval-fantasy-victory-2015.mp3" preload="auto"></audio>

    <header class="mb-8 flex flex-col md:flex-row justify-between items-start md:items-center gap-4">
        <div>
            <h1 class="text-3xl font-black uppercase tracking-tighter">Ninja Protocol</h1>
            <div class="flex items-center gap-3">
                <p id="rankTitle" class="accent-green font-bold uppercase tracking-widest text-2xl rank-glow">Loading...</p>
                <span id="streakCount" class="bg-orange-900/30 text-orange-400 px-3 py-1 rounded-full text-xs font-bold border border-orange-700/50">🔥 0 Days</span>
            </div>
        </div>
        <div class="w-full md:w-64">
            <div class="flex justify-between text-[10px] uppercase font-black mb-1 tracking-widest text-gray-400">
                <span>Mastery Progress</span>
                <span id="xpText">0/0 XP</span>
            </div>
            <div class="xp-bar-container">
                <div id="masteryBar" class="xp-bar-fill" style="width: 0%;"></div>
            </div>
        </div>
    </header>

    <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
        
        <section class="ninja-card p-6 border-t-4 border-t-green-500">
            <div class="flex justify-between items-center mb-4">
                <h2 class="text-sm font-black text-gray-500 uppercase tracking-widest">The Protocol</h2>
                <button onclick="toggleEditMode()" class="text-[10px] bg-gray-800 px-2 py-1 rounded hover:text-green-400 uppercase font-bold">Edit List</button>
            </div>
            <div id="editControls" class="hidden mb-4 space-y-2">
                <input type="text" id="newTaskInput" placeholder="Add custom habit..." class="w-full bg-black border border-gray-700 p-2 rounded text-xs outline-none focus:border-green-500">
                <button onclick="addTask()" class="w-full bg-green-900 text-green-400 text-[10px] font-bold py-1 rounded uppercase">Confirm Addition</button>
            </div>
            <div id="habitList" class="space-y-3"></div>
        </section>

        <section class="ninja-card p-6 md:col-span-2 space-y-6">
            <div class="grid grid-cols-2 gap-4">
                <div>
                    <label class="block text-[10px] text-gray-500 mb-1 uppercase font-black tracking-widest text-center">Screen Time (Limit: 3h)</label>
                    <input type="number" id="screenTimeInput" step="0.1" placeholder="Hrs" class="w-full bg-black border border-gray-800 p-4 rounded-xl focus:border-green-500 outline-none text-2xl font-bold text-center">
                    <p id="penaltyDisplay" class="text-[9px] text-center mt-1 uppercase font-bold"></p>
                </div>
                <div class="bg-black/40 rounded-xl p-4 border border-gray-800 text-center flex flex-col justify-center">
                    <p class="text-[10px] text-gray-500 uppercase font-black">Est. XP Gain</p>
                    <p id="liveScore" class="text-4xl font-black accent-green rank-glow">0</p>
                </div>
            </div>

            <button onclick="saveDailyLog()" class="w-full bg-accent text-black font-black py-5 rounded-2xl hover:brightness-110 active:scale-95 transition-all uppercase shadow-lg shadow-green-900/20 text-xl">
                Execute Daily Entry
            </button>
        </section>

        <section class="ninja-card p-6 md:col-span-3">
            <div class="flex justify-between items-center mb-6">
                <h2 class="text-sm font-black text-gray-500 uppercase tracking-widest">Combat Effectiveness (Last 7 Days)</h2>
                <div class="flex gap-2">
                     <span class="text-[10px] text-green-400 font-bold uppercase">Progress Tracker</span>
                </div>
            </div>
            <div class="h-64"><canvas id="progressChart"></canvas></div>
        </section>

    </div>

    <script>
        // --- DATABASE SETUP ---
        let habits = JSON.parse(localStorage.getItem('ninjaHabits')) || ["Gym Training", "Reading", "Deep Work", "Meditation"];
        let history = JSON.parse(localStorage.getItem('ninjaHistory')) || [];
        let totalXP = JSON.parse(localStorage.getItem('ninjaXP')) || 0;
        let editMode = false;

        const slashSound = document.getElementById('slashSound');
        const submitSound = document.getElementById('submitSound');

        // --- LEVEL SYSTEM (The XP Logic) ---
        // Every level takes progressively more XP
        const getLevel = (xp) => Math.floor(0.1 * Math.sqrt(xp)) + 1;
        const getXPForLevel = (lvl) => Math.pow((lvl) / 0.1, 2);

        function updateLevelingUI() {
            const currentLevel = getLevel(totalXP);
            const nextLevelXP = Math.round(getXPForLevel(currentLevel + 1));
            const prevLevelXP = Math.round(getXPForLevel(currentLevel));
            
            const progressInLevel = ((totalXP - prevLevelXP) / (nextLevelXP - prevLevelXP)) * 100;
            
            document.getElementById('rankTitle').innerText = `Level ${currentLevel} Ninja`;
            document.getElementById('masteryBar').style.width = `${progressInLevel}%`;
            document.getElementById('xpText').innerText = `${Math.round(totalXP)} / ${nextLevelXP} XP`;
        }

        // --- HABIT MANAGEMENT ---
        function toggleEditMode() {
            editMode = !editMode;
            document.getElementById('editControls').classList.toggle('hidden');
            renderHabits();
        }

        function addTask() {
            const input = document.getElementById('newTaskInput');
            if (input.value.trim()) {
                habits.push(input.value.trim());
                localStorage.setItem('ninjaHabits', JSON.stringify(habits));
                input.value = "";
                renderHabits();
            }
        }

        function deleteTask(index) {
            habits.splice(index, 1);
            localStorage.setItem('ninjaHabits', JSON.stringify(habits));
            renderHabits();
        }

        function renderHabits() {
            const container = document.getElementById('habitList');
            container.innerHTML = "";
            habits.forEach((h, i) => {
                container.innerHTML += `
                    <div class="flex items-center justify-between bg-black/50 p-4 rounded-xl border border-gray-900">
                        <span class="text-xs font-black uppercase tracking-widest">${h}</span>
                        <div>
                            ${editMode ? 
                                `<button onclick="deleteTask(${i})" class="text-red-500 font-bold px-2">REMOVE</button>` : 
                                `<input type="checkbox" onchange="calculateLiveScore(); slashSound.play()" class="habit-check w-6 h-6 accent-green-500 cursor-pointer">`}
                        </div>
                    </div>`;
            });
            calculateLiveScore();
        }

        // --- CALCULATION LOGIC ---
        function calculateLiveScore() {
            const screenTime = parseFloat(document.getElementById('screenTimeInput').value) || 0;
            const checks = document.querySelectorAll('.habit-check:checked').length;
            
            let baseXP = habits.length > 0 ? (checks / habits.length) * 100 : 0;
            
            // Screen Time Penalty Logic
            let penalty = 0;
            if (screenTime > 3) {
                penalty = (screenTime - 3) * 25; // 25 XP penalty per hour over 3
                document.getElementById('penaltyDisplay').innerText = `Penalty: -${Math.round(penalty)} XP`;
                document.getElementById('penaltyDisplay').className = "text-[9px] text-center mt-1 uppercase font-bold penalty-red";
            } else {
                document.getElementById('penaltyDisplay').innerText = "Safe Zone";
                document.getElementById('penaltyDisplay').className = "text-[9px] text-center mt-1 uppercase font-bold text-green-600";
            }

            const finalDailyXP = Math.max(0, Math.round(baseXP - penalty));
            document.getElementById('liveScore').innerText = finalDailyXP;
            return finalDailyXP;
        }

        function saveDailyLog() {
            const finalScore = calculateLiveScore();
            const screenTime = parseFloat(document.getElementById('screenTimeInput').value) || 0;
            
            totalXP += finalScore;
            localStorage.setItem('ninjaXP', JSON.stringify(totalXP));
            
            const entry = {
                date: new Date().toLocaleDateString('en-US', { weekday: 'short' }),
                score: finalScore
            };
            
            history.push(entry);
            localStorage.setItem('ninjaHistory', JSON.stringify(history));
            
            submitSound.play();
            updateDashboard();
            alert(`Mission Accomplished. +${finalScore} XP`);
        }

        // --- CHART & UI UPDATES ---
        function updateDashboard() {
            updateLevelingUI();
            document.getElementById('streakCount').innerText = `🔥 ${history.length} Day Streak`;

            const ctx = document.getElementById('progressChart').getContext('2d');
            if (window.myChart) window.myChart.destroy();
            
            window.myChart = new Chart(ctx, {
                type: 'line',
                data: {
                    labels: history.map(h => h.date).slice(-7),
                    datasets: [{
                        label: 'Daily XP',
                        data: history.map(h => h.score).slice(-7),
                        borderColor: '#00ff88',
                        backgroundColor: 'rgba(0, 255, 136, 0.1)',
                        borderWidth: 4,
                        fill: true,
                        tension: 0.4,
                        pointBackgroundColor: '#00ff88'
                    }]
                },
                options: { 
                    responsive: true, 
                    maintainAspectRatio: false,
                    scales: { 
                        y: { beginAtZero: true, max: 120, grid: { color: '#222' }, ticks: { color: '#555' } },
                        x: { grid: { display: false }, ticks: { color: '#555' } }
                    },
                    plugins: { legend: { display: false } }
                }
            });
        }

        // Listen for screen time changes to update score live
        document.getElementById('screenTimeInput').addEventListener('input', calculateLiveScore);

        // --- PWA SERVICE WORKER REGISTRATION ---
        if ('serviceWorker' in navigator) {
            navigator.serviceWorker.register('sw.js').catch(err => console.log("SW error", err));
        }
        renderHabits();
        updateDashboard();
    </script>
</body>
</html>
