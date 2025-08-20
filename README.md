<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Game Pembangunan Kota</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f0f4f8;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            color: #334155;
            flex-direction: column;
            gap: 1rem;
        }
        canvas {
            background-color: #e2e8f0;
            border: 2px solid #94a3b8;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
            border-radius: 0.5rem;
            cursor: crosshair;
        }
        .info-box {
            position: absolute;
            background-color: rgba(255, 255, 255, 0.9);
            border: 1px solid #cbd5e1;
            padding: 8px;
            border-radius: 0.5rem;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
            pointer-events: none;
            z-index: 1000;
            min-width: 150px;
            transition: opacity 0.2s ease-in-out;
        }
        .mode-active {
            box-shadow: 0 0 0 3px #1e40af, 0 2px 4px rgba(0, 0, 0, 0.1);
        }
        .control-button {
            background-color: rgba(59, 130, 246, 0.8);
            color: white;
            padding: 0.75rem 1rem;
            border-radius: 9999px;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
            transition: background-color 0.2s;
        }
        .control-button:active {
            background-color: rgb(59, 130, 246);
        }
    </style>
</head>
<body class="bg-gray-100 p-4 flex flex-col items-center justify-center">

    <div class="text-center mb-4">
        <h1 class="text-3xl font-bold mb-2">Simulasi Kota 2D Sederhana</h1>
        <p class="text-gray-600">Gunakan tombol di bawah untuk berinteraksi.</p>
    </div>

    <div class="relative w-full max-w-4xl flex justify-center">
        <canvas id="gameCanvas" width="800" height="600" class="w-full h-auto max-w-full"></canvas>
        <div id="infoBox" class="info-box hidden opacity-0"></div>
    </div>

    <div id="statusPanel" class="w-full max-w-4xl text-lg text-center font-bold my-4 p-2 bg-slate-200 rounded-lg shadow-inner flex flex-col md:flex-row justify-around">
        <div>Uang: <span id="moneyDisplay">1000</span></div>
        <div>Populasi: <span id="populationDisplay">0</span></div>
    </div>

    <!-- Kontrol Gerak (Dibuat agar selalu terlihat di HP, tapi disembunyikan di desktop) -->
    <div class="md:hidden w-full max-w-4xl p-2 bg-slate-200 rounded-lg shadow-inner mt-2 flex justify-center mb-4">
        <div class="grid grid-cols-3 grid-rows-3 gap-2 w-48 h-48">
            <div></div>
            <button id="upButton" class="control-button text-2xl">▲</button>
            <div></div>
            <button id="leftButton" class="control-button text-2xl">◀</button>
            <div></div>
            <button id="rightButton" class="control-button text-2xl">►</button>
            <div></div>
            <button id="downButton" class="control-button text-2xl">▼</button>
            <div></div>
        </div>
    </div>

    <div class="w-full max-w-4xl p-2 bg-slate-200 rounded-lg shadow-inner mt-2">
        <label for="taxRateSlider" class="block text-center font-bold">Tingkat Pajak: <span id="taxRateDisplay">5</span>%</label>
        <input type="range" id="taxRateSlider" min="0" max="50" value="5" class="w-full mt-1 accent-blue-500">
    </div>

    <div class="mt-4 flex flex-col sm:flex-row gap-2 flex-wrap justify-center">
        <button id="moveButton" class="px-4 py-2 text-white font-bold rounded-lg shadow-md hover:bg-blue-600 transition-colors">Mode Pindah</button>
        <button id="houseButton" class="px-4 py-2 text-white font-bold rounded-lg shadow-md transition-colors">Bangun Rumah</button>
        <button id="parkButton" class="px-4 py-2 text-white font-bold rounded-lg shadow-md transition-colors">Bangun Taman</button>
        <button id="storeButton" class="px-4 py-2 text-white font-bold rounded-lg shadow-md transition-colors">Bangun Toko</button>
        <button id="industrialButton" class="px-4 py-2 text-white font-bold rounded-lg shadow-md transition-colors">Bangun Industri</button>
        <button id="roadButton" class="px-4 py-2 text-white font-bold rounded-lg shadow-md transition-colors">Bangun Jalan</button>
        <button id="destroyButton" class="px-4 py-2 text-white font-bold rounded-lg shadow-md hover:bg-red-600 transition-colors">Hancurkan</button>
    </div>

    <script>
        // Mendapatkan elemen canvas dan konteks gambar 2D
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const infoBox = document.getElementById('infoBox');
        const moneyDisplay = document.getElementById('moneyDisplay');
        const populationDisplay = document.getElementById('populationDisplay');
        const taxRateSlider = document.getElementById('taxRateSlider');
        const taxRateDisplay = document.getElementById('taxRateDisplay');

        // Mendapatkan tombol kontrol gerak baru
        const upButton = document.getElementById('upButton');
        const downButton = document.getElementById('downButton');
        const leftButton = document.getElementById('leftButton');
        const rightButton = document.getElementById('rightButton');

        // Menyesuaikan ukuran canvas agar responsif
        function resizeCanvas() {
            canvas.width = Math.min(800, window.innerWidth - 40);
            canvas.height = canvas.width * 0.75;
        }
        window.addEventListener('resize', resizeCanvas);
        resizeCanvas();

        // Variabel-variabel game
        const gridSize = 40;
        let mapOffset = { x: 0, y: 0 };
        const buildings = [];
        let mode = 'move'; // 'move', 'build', atau 'destroy'
        let buildingType = 'house';

        // Sistem mata uang dan populasi
        let money = 1000.00; // Mulai dengan float untuk menjaga akurasi
        let population = 0;
        let lastIncomeTime = Date.now();
        const incomeInterval = 1000; // Pendapatan setiap 1 detik
        let taxRate = parseInt(taxRateSlider.value);
        const incomePerPersonPerSecond = 10;

        // Biaya dan Pendapatan Bangunan
        const buildingStats = {
            house: { cost: 100, population: 5, name: 'Rumah', color: '#fde047' },
            park: { cost: 50, name: 'Taman', color: '#22c55e' },
            store: { cost: 200, income: 0, workers: 10, name: 'Toko', color: '#f59e0b' },
            industrial: { cost: 300, income: 0, workers: 20, name: 'Industri', color: '#1f2937' },
            road: { cost: 20, name: 'Jalan', color: '#64748b' }
        };
        const moveButtonColor = '#3b82f6';
        const destroyButtonColor = '#ef4444';

        // Objek pemain
        const player = {
            x: 0,
            y: 0,
            width: gridSize * 0.7,
            height: gridSize * 0.7,
            speed: 5,
            color: '#ef4444' // Merah
        };

        /**
         * Mengubah angka menjadi format mata uang Rupiah dengan desimal
         * @param {number} amount - Jumlah uang
         * @returns {string} - Teks dengan format Rp.
         */
        function formatRupiah(amount) {
            return new Intl.NumberFormat('id-ID', {
                style: 'currency',
                currency: 'IDR',
                minimumFractionDigits: 2,
                maximumFractionDigits: 2
            }).format(amount);
        }

        // Fungsi untuk memperbarui tampilan uang dan populasi
        function updateUI() {
            moneyDisplay.textContent = formatRupiah(money);
            populationDisplay.textContent = population;
            taxRateDisplay.textContent = taxRate;
        }

        // Fungsi untuk menggambar kotak bergaris (grid)
        function drawGrid() {
            ctx.strokeStyle = '#94a3b8';
            ctx.lineWidth = 1;
            const screenGridSizeX = Math.ceil(canvas.width / gridSize) + 1;
            const screenGridSizeY = Math.ceil(canvas.height / gridSize) + 1;

            // Menggambar garis vertikal
            for (let x = 0; x < screenGridSizeX; x++) {
                const drawX = (x * gridSize) - (mapOffset.x % gridSize);
                ctx.beginPath();
                ctx.moveTo(drawX, 0);
                ctx.lineTo(drawX, canvas.height);
                ctx.stroke();
            }

            // Menggambar garis horizontal
            for (let y = 0; y < screenGridSizeY; y++) {
                const drawY = (y * gridSize) - (mapOffset.y % gridSize);
                ctx.beginPath();
                ctx.moveTo(0, drawY);
                ctx.lineTo(canvas.width, drawY);
                ctx.stroke();
            }
        }

        // Fungsi untuk menggambar bangunan
        function drawBuildings() {
            buildings.forEach(building => {
                const drawX = building.x - mapOffset.x;
                const drawY = building.y - mapOffset.y;

                if (drawX + gridSize < 0 || drawX > canvas.width || drawY + gridSize < 0 || drawY > canvas.height) {
                    return; // Lewati jika di luar layar
                }
                
                ctx.fillStyle = building.color;
                ctx.fillRect(drawX, drawY, gridSize, gridSize);
                
                // Tambahkan garis luar kecuali untuk jalan
                if (building.type !== 'road') {
                    ctx.strokeStyle = '#334155';
                    ctx.strokeRect(drawX, drawY, gridSize, gridSize);
                }
            });
        }
        
        // Fungsi untuk mengecek apakah sebuah ubin terhubung ke jalan
        function isConnectedToRoad(tileX, tileY) {
            // Periksa 4 arah (atas, bawah, kiri, kanan)
            const adjacentTiles = [
                { x: tileX, y: tileY - 1 },
                { x: tileX, y: tileY + 1 },
                { x: tileX - 1, y: tileY },
                { x: tileX + 1, y: tileY }
            ];

            for (const tile of adjacentTiles) {
                const road = buildings.find(b =>
                    Math.floor(b.x / gridSize) === tile.x &&
                    Math.floor(b.y / gridSize) === tile.y &&
                    b.type === 'road'
                );
                if (road) {
                    return true;
                }
            }
            return false;
        }

        // Fungsi untuk menghitung kebutuhan warga berdasarkan bangunan di sekitarnya
        function calculateNeeds() {
            buildings.forEach(building => {
                const tileX = Math.floor(building.x / gridSize);
                const tileY = Math.floor(building.y / gridSize);
                const isConnected = isConnectedToRoad(tileX, tileY);
                
                const nearbyBuildings = buildings.filter(b => {
                    const distanceX = Math.abs(building.x - b.x);
                    const distanceY = Math.abs(building.y - b.y);
                    return b.id !== building.id && distanceX <= gridSize * 3 && distanceY <= gridSize * 3;
                });
                
                // Logika untuk rumah
                if (building.type === 'house') {
                    const nearbyParks = nearbyBuildings.filter(b => b.type === 'park').length;
                    let happinessBonus = nearbyParks * 15;
                    
                    if (isConnected) {
                        happinessBonus += 30; // Bonus besar untuk koneksi jalan
                    }

                    // Pengurangan kebahagiaan karena pajak
                    let taxPenalty = 0;
                    if (taxRate > 10) {
                        taxPenalty = (taxRate - 10) * 2;
                    }
                    happinessBonus -= taxPenalty;

                    building.needs.happiness = Math.max(0, Math.min(100, Math.floor(happinessBonus + 50)));
                }
                
                // Logika untuk toko
                if (building.type === 'store') {
                    const nearbyHouses = nearbyBuildings.filter(b => b.type === 'house').length;
                    let profitabilityBonus = nearbyHouses * 20;

                    if (isConnected) {
                        profitabilityBonus += 40; // Bonus besar untuk koneksi jalan
                    }
                    
                    if (population === 0) {
                        profitabilityBonus = 0;
                    }

                    building.needs.profitability = Math.min(100, profitabilityBonus);
                }

                // Logika untuk industri
                if (building.type === 'industrial') {
                     const nearbyHouses = nearbyBuildings.filter(b => b.type === 'house').length;
                    let profitabilityBonus = nearbyHouses * 20;

                    if (isConnected) {
                        profitabilityBonus += 40; // Bonus besar untuk koneksi jalan
                    }

                    if (population === 0) {
                        profitabilityBonus = 0;
                    }

                    building.needs.profitability = Math.min(100, profitabilityBonus);
                }
            });
        }

        // Fungsi untuk menggambar pemain
        function drawPlayer() {
            const playerScreenX = player.x - mapOffset.x;
            const playerScreenY = player.y - mapOffset.y;
            ctx.fillStyle = player.color;
            ctx.fillRect(playerScreenX, playerScreenY, player.width, player.height);
        }

        // Variabel untuk melacak status kontrol gerak
        let keys = {};
        let touchControls = {
            up: false,
            down: false,
            left: false,
            right: false
        };

        // Fungsi untuk menangani input keyboard
        document.addEventListener('keydown', (e) => {
            keys[e.key.toLowerCase()] = true;
            const key = e.key.toLowerCase();
            if (key === 'm') {
                mode = 'move';
            } else if (key === 'h') {
                mode = 'build';
                buildingType = 'house';
            } else if (key === 'p') {
                mode = 'build';
                buildingType = 'park';
            } else if (key === 't') {
                mode = 'build';
                buildingType = 'store';
            } else if (key === 'i') {
                mode = 'build';
                buildingType = 'industrial';
            } else if (key === 'r') {
                mode = 'build';
                buildingType = 'road';
            } else if (key === 'x') {
                mode = 'destroy';
            }
            updateAllButtons();
        });
        document.addEventListener('keyup', (e) => {
            keys[e.key.toLowerCase()] = false;
        });

        // Event listener untuk penggeser pajak
        taxRateSlider.addEventListener('input', (e) => {
            taxRate = parseInt(e.target.value);
            taxRateDisplay.textContent = taxRate;
            calculateNeeds(); // Perbarui kebahagiaan secara real-time saat pajak diubah
        });

        // Tangani event sentuh pada tombol gerak
        function setupTouchControls() {
            const controls = {
                up: upButton, down: downButton, left: leftButton, right: rightButton
            };

            for (const direction in controls) {
                const button = controls[direction];
                button.addEventListener('touchstart', (e) => {
                    e.preventDefault();
                    touchControls[direction] = true;
                }, { passive: false });
                button.addEventListener('touchend', (e) => {
                    e.preventDefault();
                    touchControls[direction] = false;
                });
            }
        }
        setupTouchControls();
        
        // --- LOGIKA POPULASI YANG SANGAT JELAS BERDASARKAN ATURAN PENGGUNA ---
        function checkPopulationChange() {
            const houseBuildings = buildings.filter(b => b.type === 'house');
            
            houseBuildings.forEach(house => {
                let changeAmount = 0;
                let logMessage = '';

                // Logika berdasarkan Zona Pajak
                if (taxRate <= 20) {
                    // Zona 0-20%: Populasi pasti bertambah
                    changeAmount = Math.floor(Math.random() * 2) + 1; // Bertambah 1 atau 2
                    logMessage = `populasi bertambah karena pajak rendah`;
                } else if (taxRate <= 45) {
                    // Zona 21-45%: Zona Abu-abu
                    const increaseChance = (45 - taxRate) / 25; // Peluang menurun dari 100% (pajak 20) ke 0% (pajak 45)
                    const decreaseChance = (taxRate - 20) / 25; // Peluang meningkat dari 0% (pajak 20) ke 100% (pajak 45)
                    
                    if (Math.random() < increaseChance) {
                        changeAmount += Math.floor(Math.random() * 2) + 1;
                        logMessage = `mendapatkan ${changeAmount} penghuni baru`;
                    }
                    
                    if (Math.random() < decreaseChance) {
                        changeAmount -= Math.floor(Math.random() * 2) + 1;
                        logMessage = `kehilangan ${Math.abs(changeAmount)} penghuni`;
                    }
                } else {
                    // Zona 46-50%: Populasi pasti berkurang
                    changeAmount = -(Math.floor(Math.random() * 2) + 1); // Berkurang 1 atau 2
                    logMessage = `populasi berkurang karena pajak sangat tinggi`;
                }
                
                const oldPopulation = house.population;
                let newPopulation = oldPopulation + changeAmount;
                
                // Aturan khusus: Rumah kosong hanya jika pajak > 40%
                if (taxRate > 40 && newPopulation < 0) {
                    newPopulation = 0;
                } else if (newPopulation < 1) {
                    // Jika pajak <= 40%, populasi tidak boleh 0
                    newPopulation = 1;
                }

                house.population = Math.min(buildingStats.house.population, newPopulation);
                
                if (house.population !== oldPopulation) {
                    console.log(`Rumah di (${Math.floor(house.x/gridSize)}, ${Math.floor(house.y/gridSize)}) berubah populasi dari ${oldPopulation} menjadi ${house.population}`);
                }
            });
        }

        // Set interval untuk mengecek perubahan populasi
        setInterval(checkPopulationChange, 5000); 

        // Fungsi untuk memperbarui posisi pemain, peta, dan data game
        function update() {
            if (mode === 'move') {
                let moveX = 0;
                let moveY = 0;
                // Gerak dari keyboard
                if (keys['arrowup'] || keys['w']) moveY -= player.speed;
                if (keys['arrowdown'] || keys['s']) moveY += player.speed;
                if (keys['arrowleft'] || keys['a']) moveX -= player.speed;
                if (keys['arrowright'] || keys['d']) moveX += player.speed;
                // Gerak dari tombol sentuh
                if (touchControls.up) moveY -= player.speed;
                if (touchControls.down) moveY += player.speed;
                if (touchControls.left) moveX -= player.speed;
                if (touchControls.right) moveX += player.speed;

                const playerScreenX = player.x - mapOffset.x;
                const playerScreenY = player.y - mapOffset.y;
                const edgeBuffer = 200;

                if (playerScreenX + moveX < edgeBuffer) mapOffset.x -= player.speed;
                if (playerScreenX + moveX > canvas.width - edgeBuffer) mapOffset.x += player.speed;
                if (playerScreenY + moveY < edgeBuffer) mapOffset.y -= player.speed;
                if (playerScreenY + moveY > canvas.height - edgeBuffer) mapOffset.y += player.speed;
                
                player.x += moveX;
                player.y += moveY;

                const worldSize = 5000;
                player.x = Math.max(0, Math.min(worldSize - player.width, player.x));
                player.y = Math.max(0, Math.min(worldSize - player.height, player.y));
            }

            // Hitung ulang populasi total
            population = 0;
            buildings.forEach(b => {
                if (b.type === 'house') {
                    population += b.population;
                }
            });

            const now = Date.now();
            if (now - lastIncomeTime > incomeInterval) {
                let totalIncome = 0;
                
                // PENDAPATAN DARI PAJAK
                totalIncome += population * incomePerPersonPerSecond * (taxRate / 100);

                // PENDAPATAN DARI KEUNTUNGAN TOKO DAN INDUSTRI
                buildings.forEach(b => {
                    if (population > 0 && (b.type === 'store' || b.type === 'industrial')) {
                        totalIncome += buildingStats[b.type].cost * (b.needs.profitability / 100);
                    }
                });

                money += totalIncome;
                lastIncomeTime = now;
                updateUI();
            }
        }

        // Fungsi untuk menggambar ulang semua elemen game
        function draw() {
            ctx.fillStyle = '#f8fafc';
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            drawGrid();
            drawBuildings();
            drawPlayer();

            const playerTileX = Math.floor(player.x / gridSize);
            const playerTileY = Math.floor(player.y / gridSize);
            let buildingFound = null;
            buildings.forEach(b => {
                if (Math.floor(b.x / gridSize) === playerTileX && Math.floor(b.y / gridSize) === playerTileY) {
                    buildingFound = b;
                }
            });

            if (buildingFound) {
                let infoText = `
                    <h3 class="font-bold text-lg mb-1">${buildingStats[buildingFound.type].name}</h3>
                    <p>Posisi: (${Math.floor(buildingFound.x/gridSize)}, ${Math.floor(buildingFound.y/gridSize)})</p>
                `;
                if (buildingFound.type === 'house') {
                    infoText += `<p>Populasi: ${buildingFound.population} orang</p>`;
                    infoText += `<p>Kebahagiaan Warga: ${buildingFound.needs.happiness}%</p>`;
                    // Menambahkan informasi pendapatan per rumah
                    const taxPerHouse = (buildingFound.population * incomePerPersonPerSecond) * (taxRate / 100);
                    infoText += `<p>Pendapatan Pajak: ${formatRupiah(taxPerHouse)}/detik</p>`;
                } else if (buildingFound.type === 'store' || buildingFound.type === 'industrial') {
                    infoText += `<p>Profitabilitas: ${buildingFound.needs.profitability}%</p>`;
                }

                infoBox.innerHTML = infoText;
                infoBox.style.left = `${(player.x - mapOffset.x) + player.width + 10}px`;
                infoBox.style.top = `${(player.y - mapOffset.y) - infoBox.offsetHeight}px`;
                infoBox.classList.remove('hidden');
                infoBox.classList.add('opacity-100');
            } else {
                infoBox.classList.remove('opacity-100');
                infoBox.classList.add('hidden');
            }
        }

        // Loop utama game
        function gameLoop() {
            update();
            draw();
            requestAnimationFrame(gameLoop);
        }
        gameLoop();

        // Tangani klik mouse dan sentuhan untuk mode 'build' dan 'destroy'
        canvas.addEventListener('click', handleCanvasClick);
        canvas.addEventListener('touchstart', (e) => {
            e.preventDefault(); // Mencegah scrolling default
            const touch = e.touches[0];
            const mouseEvent = new MouseEvent("click", {
                clientX: touch.clientX,
                clientY: touch.clientY
            });
            canvas.dispatchEvent(mouseEvent);
        });

        function handleCanvasClick(e) {
            const rect = canvas.getBoundingClientRect();
            const scaleX = canvas.width / rect.width;
            const scaleY = canvas.height / rect.height;
            const mouseX = (e.clientX - rect.left) * scaleX;
            const mouseY = (e.clientY - rect.top) * scaleY;
            const tileX = Math.floor((mouseX + mapOffset.x) / gridSize);
            const tileY = Math.floor((mouseY + mapOffset.y) / gridSize);

            if (mode === 'build') {
                const existingBuilding = buildings.find(b =>
                    Math.floor(b.x / gridSize) === tileX && Math.floor(b.y / gridSize) === tileY
                );

                const stats = buildingStats[buildingType];
                const cost = stats.cost;

                if (!existingBuilding && money >= cost) {
                    const newBuilding = {
                        id: Date.now(),
                        x: tileX * gridSize,
                        y: tileY * gridSize,
                        type: buildingType,
                        color: stats.color,
                        population: stats.population || 0,
                        needs: {
                            happiness: 0,
                            profitability: 0
                        }
                    };
                    
                    money -= cost;
                    buildings.push(newBuilding);
                    calculateNeeds();
                    updateUI();
                } else if (existingBuilding) {
                    infoBox.innerHTML = `<p class="text-red-500">Sudah ada bangunan di sini!</p>`;
                    infoBox.classList.remove('hidden');
                    infoBox.classList.add('opacity-100');
                    setTimeout(() => infoBox.classList.remove('opacity-100'), 1000);
                } else if (money < cost) {
                    infoBox.innerHTML = `<p class="text-red-500">Uang tidak cukup! Biaya: ${formatRupiah(cost)}</p>`;
                    infoBox.classList.remove('hidden');
                    infoBox.classList.add('opacity-100');
                    setTimeout(() => infoBox.classList.remove('opacity-100'), 1000);
                }
            } else if (mode === 'destroy') {
                const buildingIndex = buildings.findIndex(b =>
                    Math.floor(b.x / gridSize) === tileX && Math.floor(b.y / gridSize) === tileY
                );

                if (buildingIndex !== -1) {
                    const destroyedBuilding = buildings.splice(buildingIndex, 1)[0];
                    money += buildingStats[destroyedBuilding.type].cost * 0.5;
                    calculateNeeds();
                    updateUI();
                }
            }
        }

        // Tangani tombol UI
        const moveButton = document.getElementById('moveButton');
        const houseButton = document.getElementById('houseButton');
        const parkButton = document.getElementById('parkButton');
        const storeButton = document.getElementById('storeButton');
        const industrialButton = document.getElementById('industrialButton');
        const roadButton = document.getElementById('roadButton');
        const destroyButton = document.getElementById('destroyButton');

        // Fungsi untuk mengupdate tampilan semua tombol
        function updateAllButtons() {
            const allButtons = [
                { id: moveButton, mode: 'move', type: 'move' },
                { id: houseButton, mode: 'build', type: 'house' },
                { id: parkButton, mode: 'build', type: 'park' },
                { id: storeButton, mode: 'build', type: 'store' },
                { id: industrialButton, mode: 'build', type: 'industrial' },
                { id: roadButton, mode: 'build', type: 'road' },
                { id: destroyButton, mode: 'destroy', type: 'destroy' }
            ];
            
            allButtons.forEach(btn => {
                btn.id.classList.remove('mode-active');
                if (btn.type === 'move') {
                    btn.id.style.backgroundColor = moveButtonColor;
                } else if (btn.type === 'destroy') {
                    btn.id.style.backgroundColor = destroyButtonColor;
                } else {
                    btn.id.style.backgroundColor = buildingStats[btn.type].color;
                }
            });

            if (mode === 'move') {
                moveButton.classList.add('mode-active');
            } else if (mode === 'destroy') {
                destroyButton.classList.add('mode-active');
            } else if (mode === 'build') {
                if (buildingType === 'house') {
                    houseButton.classList.add('mode-active');
                } else if (buildingType === 'park') {
                    parkButton.classList.add('mode-active');
                } else if (buildingType === 'store') {
                    storeButton.classList.add('mode-active');
                } else if (buildingType === 'industrial') {
                    industrialButton.classList.add('mode-active');
                } else if (buildingType === 'road') {
                    roadButton.classList.add('mode-active');
                }
            }
        }

        // Event listeners untuk tombol
        moveButton.addEventListener('click', () => {
            mode = 'move';
            updateAllButtons();
        });

        houseButton.addEventListener('click', () => {
            mode = 'build';
            buildingType = 'house';
            updateAllButtons();
        });
        parkButton.addEventListener('click', () => {
            mode = 'build';
            buildingType = 'park';
            updateAllButtons();
        });
        storeButton.addEventListener('click', () => {
            mode = 'build';
            buildingType = 'store';
            updateAllButtons();
        });
        industrialButton.addEventListener('click', () => {
            mode = 'build';
            buildingType = 'industrial';
            updateAllButtons();
        });
        roadButton.addEventListener('click', () => {
            mode = 'build';
            buildingType = 'road';
            updateAllButtons();
        });
        destroyButton.addEventListener('click', () => {
            mode = 'destroy';
            updateAllButtons();
        });

        updateUI();
        updateAllButtons();
    </script>
</body>
</html>
