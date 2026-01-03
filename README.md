<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Ninja Mastery 365</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=VT323&display=swap" rel="stylesheet">
    <style>
        body { background-color: #050505; color: #e5e5e5; font-family: 'Inter', sans-serif; }
        .ninja-card { background: #111; border: 1px solid #222; border-radius: 16px; }
        .accent-green { color: #00ff88; }
        .bg-accent { background-color: #00ff88; }
        .rank-glow { text-shadow: 0 0 12px rgba(0, 255, 136, 0.7); font-family: 'VT323', monospace; }
        .streak-fire { filter: drop-shadow(0 0 5px #ff6600); }
    </style>
</head>
<body class="p-4 max-w-4xl mx-auto pb-10">

    <audio id="slashSound" src="https://assets.mixkit.co/sfx/preview/mixkit-fast-sword-whoosh-1507.mp3" preload="auto"></audio>
    <audio id="submitSound" src="https://assets.mixkit.co/sfx/preview/mixkit-medieval-fantasy-victory-2015.mp3" preload="auto"></audio>

    <header class="mb-8 flex justify-between items-center">
        <div>
            <h1 class="text-3xl font-black uppercase tracking-tighter">Ninja Protocol</h1>
            <div class="flex items-center gap-3">
                <p id="rankTitle" class="accent-green font-bold uppercase tracking-widest text-xl rank-glow underline">Recruit</p>
                <div class="flex items-center bg-orange-900/30 px-2 py-0.5 rounded-full border border-orange-700/50">
                    <span class="streak-fire text-sm">🔥</span>
                    <span id="streakCount" class="text-orange-400 font-bold text-xs ml-1">0 Day Streak</span>
                </div>
            </div>
        </div>
        <div class="text-right">
            <p id="masteryPercentage" class="text-xs text-gray-400 mb-1">Mastery: 0%</p>
            <div class="w-32 h-2 bg-gray-900 rounded-full overflow-hidden border border-gray-800">
                <div id="masteryBar" class="h-full bg-accent transition-all duration-500" style="width: 0%;"></div>
            </div>
        </div>
    </header>

    <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
        
        <section class="ninja-card p-6 md:col-span-1 border-t-4 border-t-green-500">
            <div class="flex justify-between items-center mb-4">
                <h2 class="text-sm font-bold text-gray-400 uppercase tracking-widest">Protocol</h2>
                <button onclick="toggleEditMode()" class="text-[10px] bg-gray-800 px-2 py-1 rounded hover:text-green-400 uppercase font-bold">Edit</button>
            </div>
            <div id="editControls" class="hidden mb-4 space-y-2">
                <input type="text" id="newTaskInput" placeholder="New Task..." class="w-full bg-black border border-gray-700 p-2 rounded text-xs outline-none focus:border-green-500">
                <button onclick="addTask()" class="w-full bg-green-900 text-green-400 text-[10px] font-bold py-1 rounded uppercase tracking-widest">Add</button>
            </div>
            <div id="habitList" class="space-y-3"></div>
        </section>

        <section class="ninja-card p-6 md:col-span-2 space-y-6">
            <div class="grid grid-cols-2 gap-4">
                <div>
                    <label class="block text-[10px] text-gray-500 mb-1 uppercase font-bold tracking-widest text-center">Screen Time</label>
                    <input type="number" id="screenTimeInput" placeholder="Hrs" class="w-full bg-black border border-gray-800 p-4 rounded-xl focus:border-green-500 outline-none text-2xl font-bold text-center">
                </div>
                <div class="bg-black/40 rounded-xl p-4 border border-gray-800 text-center flex flex-col justify-center">
                    <p class="text-[10px] text-gray-500 uppercase font-bold">Today's Score</p>
                    <p id="todayScore" class="text-4xl font-black accent-green rank-glow">0</p>
                </div>
            </div>

            <div>
                <label class="block text-[10px] text-gray-500 mb-1 uppercase font-bold tracking-widest">Daily Scroll (Notes)</label>
                <textarea id="dailyJournal" placeholder="What did you learn today, Ninja?" class="w-full bg-black border border-gray-800 p-3 rounded-xl focus:border-green-500 outline-none text-sm h-20 resize-none"></textarea>
            </div>

            <button onclick="saveDailyLog()" class="w-full bg-accent text-black font-black py-5 rounded-2xl hover:brightness-110 active:scale-95 transition uppercase shadow-lg shadow-green-900/20 text-lg">
                Execute Daily Entry
            </button>
        </section>

        <section class="ninja-card p-6 md:col-span-3">
            <h2 class="text-sm font-bold mb-4 text-gray-400 uppercase tracking-widest">Combat Effectiveness</h2>
            <div class="h-64"><canvas id="progressChart"></canvas></div>
        </section>

        <section class="ninja-card p-4 md:col-span-3 flex justify-between items-center bg-gray-900/20 border-dashed">
            <p class="text-[10px] text-gray-600 uppercase font-bold">1-Year Data Protection</p>
            <div class="flex gap-2">
                <button onclick="exportData()" class="text-[10px] bg-gray-800 px-3 py-1 rounded text-gray-400 hover:text-white uppercase font-bold">Backup</button>
                <label class="text-[10px] bg-gray-800 px-3 py-1 rounded text-gray-400 hover:text-white uppercase font-bold cursor-pointer">
                    Import <input type="file" id="importFile" class="hidden" onchange="importData(event)">
                </label>
            </div>
        </section>
    </div>

    <script>
        let habits = JSON.parse(localStorage.getItem('ninjaHabits')) || ["Gym", "Reading", "Deep Work"];
        let history = JSON.parse(localStorage.getItem('ninjaHistory')) || [];
        let editMode = false;

        const slashSound = document.getElementById('slashSound');
        const submitSound = document.getElementById('submitSound');

        function toggleEditMode() {
            editMode = !editMode;
            document.getElementById('editControls').classList.toggle('hidden');
            renderHabits();
        }

        function addTask() {
            const input = document.getElementById('newTaskInput');
            if (input.value.trim() !== "") {
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
                    <div class="flex items-center justify-between bg-black/50 p-3 rounded-xl border border-gray-900">
                        <span class="text-xs font-bold uppercase tracking-tight">${h}</span>
                        <div class="flex items-center gap-2">
                            ${editMode ? `<button onclick="deleteTask(${i})" class="text-red-500 font-bold">✕</button>` : `<input type="checkbox" onchange="slashSound.play()" class="habit-check w-5 h-5 accent-green-500 cursor-pointer">`}
                        </div>
                    </div>`;
            });
        }

        function saveDailyLog() {
            const screenTime = parseFloat(document.getElementById('screenTimeInput').value) || 0;
            const checks = document.querySelectorAll('.habit-check:checked').length;
            const journalText = document.getElementById('dailyJournal').value;
            
            let baseScore = habits.length > 0 ? (checks / habits.length) * 100 : 0;
            let penalty = screenTime > 3 ? (screenTime - 3) * 15 : 0;
            let finalScore = Math.max(0, Math.round(baseScore - penalty));

            document.getElementById('todayScore').innerText = finalScore;
            
            const entry = {
                date: new Date().toLocaleDateString(),
                score: finalScore,
                note: journalText,
                screen: screenTime
            };
            
            history.push(entry);
            localStorage.setItem('ninjaHistory', JSON.stringify(history));
            submitSound.play();
            updateDashboard();
            alert("Protocol Executed.");
        }

        function updateDashboard() {
            const avg = history.length ? history.reduce((a, b) => a + b.score, 0) / history.length : 0;
            
            let rank = "Recruit";
            if (avg >= 90) rank = "Shadow Grandmaster";
            else if (avg >= 75) rank = "Elite Assassin";
            else if (avg >= 50) rank = "Chunin";
            else if (avg >= 25) rank = "Genin";
            
            document.getElementById('rankTitle').innerText = rank;
            document.getElementById('masteryBar').style.width = `${avg}%`;
            document.getElementById('masteryPercentage').innerText = `Mastery: ${Math.round(avg)}%`;
            document.getElementById('streakCount').innerText = `${history.length} Day Streak`;

            const ctx = document.getElementById('progressChart').getContext('2d');
            if (window.myChart) window.myChart.destroy();
            window.myChart = new Chart(ctx, {
                type: 'line',
                data: {
                    labels: history.map(h => h.date).slice(-30),
                    datasets: [{
                        label: 'Mastery',
                        data: history.map(h => h.score).slice(-30),
                        borderColor: '#00ff88',
                        backgroundColor: 'rgba(0, 255, 136, 0.1)',
                        fill: true,
                        tension: 0.4
                    }]
                },
                options: { responsive: true, maintainAspectRatio: false, scales: { y: { beginAtZero: true, max: 100 } } }
            });
        }

        function exportData() {
            const data = { habits, history };
            const blob = new Blob([JSON.stringify(data)], {type: 'application/json'});
            const a = document.createElement('a');
            a.href = URL.createObjectURL(blob);
            a.download = `ninja_backup.json`;
            a.click();
        }

        function importData(event) {
            const reader = new FileReader();
            reader.onload = (e) => {
                const imported = JSON.parse(e.target.result);
                localStorage.setItem('ninjaHabits', JSON.stringify(imported.habits));
                localStorage.setItem('ninjaHistory', JSON.stringify(imported.history));
                location.reload();
            };
            reader.readAsText(event.target.files[0]);
        }

        renderHabits();
        updateDashboard();
    </script>
</body>
</html>
