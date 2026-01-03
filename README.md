<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Ninja Mastery 365</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.6.0/dist/confetti.browser.min.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=VT323&family=Inter:wght@400;900&display=swap" rel="stylesheet">
    <style>
        body { background-color: #050505; color: #e5e5e5; font-family: 'Inter', sans-serif; overflow-x: hidden; }
        .ninja-card { background: #111; border: 1px solid #222; border-radius: 16px; }
        .accent-green { color: #00ff88; }
        .rank-glow { text-shadow: 0 0 15px currentColor; font-family: 'VT323', monospace; }
        .xp-bar-fill { height: 100%; background: linear-gradient(90deg, #00ff88, #0088ff); transition: width 1s ease; }
        .mood-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(10px, 1fr)); gap: 3px; }
        .mood-pixel { aspect-ratio: 1; border-radius: 2px; background: #1a1a1a; border: 1px solid #222; }
        .penalty-red { color: #ff4444; animation: pulse 1s infinite; }
        @keyframes pulse { 0%, 100% { opacity: 1; } 50% { opacity: 0.5; } }
    </style>
</head>
<body class="p-4 max-w-4xl mx-auto pb-10">

    <header class="mb-8 flex flex-col md:flex-row justify-between items-start md:items-center gap-4">
        <div>
            <h1 class="text-3xl font-black uppercase tracking-tighter">Ninja Protocol</h1>
            <p id="rankTitle" class="font-bold uppercase tracking-widest text-3xl rank-glow">Recruit</p>
        </div>
        <div class="w-full md:w-64">
            <div class="flex justify-between text-[10px] uppercase font-black mb-1 text-gray-400">
                <span>Total XP: <span id="totalXpDisplay">0</span></span>
                <span id="xpText">Next Rank: 200</span>
            </div>
            <div class="h-3 w-full bg-black border border-gray-800 rounded-full overflow-hidden">
                <div id="masteryBar" class="xp-bar-fill" style="width: 0%;"></div>
            </div>
        </div>
    </header>

    <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
        
        <section class="ninja-card p-6 border-t-4 border-t-green-500">
            <div class="flex justify-between items-center mb-4">
                <h2 class="text-xs font-black text-gray-500 uppercase tracking-widest">Protocol</h2>
                <button onclick="toggleEditMode()" id="editBtn" class="text-[10px] bg-gray-800 px-2 py-1 rounded uppercase font-bold text-green-400">Edit Tasks</button>
            </div>

            <div id="editControls" class="hidden mb-4 space-y-2">
                <input type="text" id="newTaskInput" placeholder="Add Task Name..." class="w-full bg-black border border-gray-700 p-2 rounded text-xs outline-none focus:border-green-500">
                <button onclick="addTask()" class="w-full bg-green-900 text-green-400 text-[10px] font-bold py-1 rounded uppercase">Confirm Add</button>
            </div>

            <div id="habitList" class="space-y-3"></div>
        </section>

        <section class="ninja-card p-6 md:col-span-2 space-y-6">
            <div class="grid grid-cols-2 gap-4">
                <div class="bg-black/40 rounded-xl p-4 border border-gray-800">
                    <label class="block text-[10px] text-gray-500 mb-1 uppercase font-black text-center">Screen Time (1hr = -5xp)</label>
                    <input type="number" id="screenTimeInput" step="1" value="0" class="w-full bg-transparent p-2 text-3xl font-bold text-center outline-none">
                    <p id="penaltyDisplay" class="text-[9px] text-center mt-1 font-bold uppercase">Safe</p>
                </div>
                <div class="bg-black/40 rounded-xl p-4 border border-gray-800 text-center flex flex-col justify-center">
                    <p class="text-[10px] text-gray-500 uppercase font-black">Score Impact</p>
                    <p id="liveScore" class="text-4xl font-black accent-green rank-glow">0</p>
                </div>
            </div>

            <button onclick="saveDailyLog()" class="w-full bg-[#00ff88] text-black font-black py-5 rounded-2xl transition-all uppercase text-xl shadow-lg active:scale-95">
                Execute Daily Entry
            </button>
        </section>

        <section class="ninja-card p-6 md:col-span-3">
            <h2 class="text-xs font-black text-gray-500 uppercase tracking-widest mb-4">365 Day Spirit Scroll</h2>
            <div id="moodGrid" class="mood-grid"></div>
        </section>
    </div>

    <script>
        let habits = JSON.parse(localStorage.getItem('ninjaHabits')) || ["Gym", "Reading", "Deep Work"];
        let history = JSON.parse(localStorage.getItem('ninjaHistory')) || [];
        let totalXP = JSON.parse(localStorage.getItem('ninjaXP')) || 0;
        let isEditMode = false;

        function toggleEditMode() {
            isEditMode = !isEditMode;
            document.getElementById('editControls').classList.toggle('hidden');
            document.getElementById('editBtn').innerText = isEditMode ? "Done Editing" : "Edit Tasks";
            document.getElementById('editBtn').classList.toggle('bg-red-900');
            renderHabits();
        }

        function addTask() {
            const input = document.getElementById('newTaskInput');
            const val = input.value.trim();
            if (val) {
                habits.push(val);
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
            container.innerHTML = habits.map((h, i) => `
                <div class="flex items-center justify-between bg-black/50 p-3 rounded-xl border border-gray-900">
                    <span class="text-xs font-bold uppercase tracking-widest">${h}</span>
                    ${isEditMode ? 
                        `<button onclick="deleteTask(${i})" class="text-red-500 font-bold text-xs px-2">DELETE</button>` : 
                        `<input type="checkbox" id="check-${i}" onchange="calculateScore()" class="habit-check w-6 h-6 accent-green-500">`
                    }
                </div>`).join('');
            calculateScore();
        }

        function calculateScore() {
            const screenTime = parseFloat(document.getElementById('screenTimeInput').value) || 0;
            const checkedCount = document.querySelectorAll('.habit-check:checked').length;
            
            // Gain 20 per habit
            let gain = checkedCount * 20;

            // Laziness Penalty (-5 if 0 habits checked)
            if (checkedCount === 0 && habits.length > 0) gain = -5;

            // Screen Penalty (-5 per hour)
            let screenPenalty = screenTime * 5;
            
            let finalDailyScore = gain - screenPenalty;

            const scoreDisplay = document.getElementById('liveScore');
            scoreDisplay.innerText = (finalDailyScore > 0 ? "+" : "") + finalDailyScore;
            scoreDisplay.style.color = finalDailyScore >= 0 ? "#00ff88" : "#ff4444";

            document.getElementById('penaltyDisplay').innerText = screenPenalty > 0 ? `-${screenPenalty} XP SCREEN TIME` : "SAFE";
            document.getElementById('penaltyDisplay').className = screenPenalty > 0 ? "text-[9px] text-center mt-1 font-bold penalty-red" : "text-[9px] text-center mt-1 font-bold text-green-600";

            return finalDailyScore;
        }

        function updateLevelingUI() {
            let rank = "Recruit", color = "#9ca3af", nextXP = 200;
            if (totalXP >= 5000) { rank = "Shadow Master"; color = "#facc15"; nextXP = 10000; }
            else if (totalXP >= 2000) { rank = "Elite Chunin"; color = "#a855f7"; nextXP = 5000; }
            else if (totalXP >= 800) { rank = "Chunin"; color = "#3b82f6"; nextXP = 2000; }
            else if (totalXP >= 200) { rank = "Genin"; color = "#00ff88"; nextXP = 800; }

            document.getElementById('rankTitle').innerText = rank;
            document.getElementById('rankTitle').style.color = color;
            document.getElementById('totalXpDisplay').innerText = Math.round(totalXP);
            document.getElementById('masteryBar').style.width = `${Math.min((totalXP / nextXP) * 100, 100)}%`;
            document.getElementById('xpText').innerText = `Next Rank: ${nextXP}`;
        }

        function saveDailyLog() {
            const dailyScore = calculateScore();
            totalXP += dailyScore;
            if (totalXP < 0) totalXP = 0;
            
            localStorage.setItem('ninjaXP', JSON.stringify(totalXP));
            history.push({ date: new Date().toLocaleDateString(), score: dailyScore });
            localStorage.setItem('ninjaHistory', JSON.stringify(history));
            
            if (dailyScore > 0) confetti({ particleCount: 100, spread: 70, origin: { y: 0.6 } });
            
            updateLevelingUI();
            renderMoodGrid();
            alert(`Log Saved! Change: ${dailyScore} XP`);
        }

        function renderMoodGrid() {
            const grid = document.getElementById('moodGrid');
            grid.innerHTML = "";
            for (let i = 0; i < 365; i++) {
                const pixel = document.createElement('div');
                pixel.className = 'mood-pixel';
                if (history[i]) pixel.style.background = history[i].score > 0 ? "#00ff88" : "#ff4444";
                grid.appendChild(pixel);
            }
        }

        document.getElementById('screenTimeInput').addEventListener('input', calculateScore);
        renderHabits();
        updateLevelingUI();
        renderMoodGrid();
    </script>
</body>
</html>
