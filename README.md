<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Simulasi Kota 2D Sederhana</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        /* Perbaikan: Wadah fleksibel untuk konten yang dapat digulir */
        body {
            font-family: 'Inter', sans-serif;
            margin: 0;
            background-color: #f3f4f6;
            min-height: 100vh;
            display: flex;
            justify-content: center;
            align-items: flex-start; /* Ganti dari 'center' ke 'flex-start' */
        }
        .scroll-container {
            width: 100%;
            max-width: 800px;
            padding: 1rem;
            display: flex;
            flex-direction: column;
            align-items: center;
            overflow-y: auto; /* Memungkinkan scrolling di wadah ini */
            -webkit-overflow-scrolling: touch; /* Perilaku scrolling yang lebih baik di iOS */
        }
        canvas {
            border-radius: 0.5rem;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
            touch-action: none;
            width: 100%;
            height: auto;
            max-width: 800px;
        }
        .mode-active {
            box-shadow: 0 0 0 4px #60a5fa;
            transform: scale(1.05);
        }
        .info-box {
            position: absolute;
            background-color: rgba(255, 255, 255, 0.9);
            padding: 0.5rem;
            border-radius: 0.5rem;
            box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
            transition: opacity 0.2s ease-in-out;
            pointer-events: none;
            z-index: 10;
        }
        .control-button {
            background-color: rgba(209, 213, 219, 0.7);
            color: #4b5563;
            border-radius: 0.5rem;
            display: flex;
            align-items: center;
            justify-content: center;
            font-weight: bold;
            box-shadow: inset 0 2px 4px rgba(0, 0, 0, 0.1);
            transition: transform 0.1s ease-in-out;
            backdrop-filter: blur(2px);
            width: 50px;
            height: 50px;
        }
        .control-button:active {
            transform: scale(0.95);
        }
        .action-button {
            width: 100%;
            padding: 0.5rem 1rem;
            color: white;
            font-weight: bold;
            border-radius: 0.5rem;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
            transition-property: background-color, transform, box-shadow;
            transition-duration: 0.15s;
            transition-timing-function: ease-in-out;
        }
        .modal {
            position: fixed;
            z-index: 100;
            left: 0;
            top: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.6);
            display: flex;
            justify-content: center;
            align-items: center;
            visibility: hidden;
            opacity: 0;
            transition: visibility 0s, opacity 0.3s linear;
        }
        .modal-show {
            visibility: visible;
            opacity: 1;
        }
        .modal-content {
            background-color: #fff;
            padding: 2rem;
            border-radius: 0.75rem;
            max-width: 90%;
            width: 500px;
            box-shadow: 0 8px 16px rgba(0, 0, 0, 0.2);
            position: relative;
        }
        .modal-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 1rem;
        }
        .modal-close {
            font-size: 2rem;
            font-weight: bold;
            color: #9ca3af;
            cursor: pointer;
            border: none;
            background: none;
        }
        .modal-close:hover {
            color: #6b7280;
        }
        .guide-list li {
            margin-bottom: 0.5rem;
            line-height: 1.5;
        }
        .popup-menu {
            position: absolute;
            bottom: 100%;
            left: 50%;
            transform: translateX(-50%);
            margin-bottom: 1rem;
            background-color: #fff;
            padding: 1rem;
            border-radius: 0.75rem;
            box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
            display: flex;
            flex-direction: column;
            gap: 0.5rem;
            z-index: 50;
            visibility: hidden;
            opacity: 0;
            transition: visibility 0s, opacity 0.2s linear;
        }
        .popup-menu.show {
            visibility: visible;
            opacity: 1;
        }
        #landscape-controls {
            display: none;
            position: absolute;
            bottom: 1rem;
            right: 1rem;
            width: 150px;
            height: 150px;
            grid-template-columns: repeat(3, 1fr);
            grid-template-rows: repeat(3, 1fr);
            gap: 0.5rem;
            z-index: 20;
        }
        @media (max-width: 768px) and (orientation: landscape) {
            #landscape-controls {
                display: grid;
            }
        }
        #portrait-controls {
            display: none;
            flex-direction: row;
            justify-content: center;
            gap: 4px;
            margin-top: 1rem;
            width: 100%;
            z-index: 20;
        }
        @media (max-width: 768px) and (orientation: portrait) {
            #portrait-controls {
                display: flex;
            }
        }
    </style>
</head>
<body>

<div class="scroll-container">
    <div class="main-container flex flex-col items-center w-full max-w-full md:max-w-4xl">
        <div class="text-center mb-4">
            <h1 class="text-3xl font-bold mb-2">Simulasi Kota 2D Sederhana</h1>
            <p class="text-gray-600">Gunakan tombol di bawah untuk berinteraksi.</p>
        </div>

        <div class="relative w-full flex justify-center">
            <canvas id="gameCanvas" class="w-full h-auto max-w-full"></canvas>
            <div id="infoBox" class="info-box opacity-0 hidden"></div>
            
            <div id="landscape-controls">
                <div></div>
                <button id="landscape-up-btn" class="control-button text-2xl">▲</button>
                <div></div>
                <button id="landscape-left-btn" class="control-button text-2xl">◀</button>
                <div></div>
                <button id="landscape-right-btn" class="control-button text-2xl">►</button>
                <div></div>
                <button id="landscape-down-btn" class="control-button text-2xl">▼</button>
                <div></div>
            </div>
        </div>
        
        <div id="portrait-controls" class="flex flex-row justify-center gap-4 mt-4 w-full">
            <button id="portrait-up-btn" class="control-button text-2xl">▲</button>
            <button id="portrait-down-btn" class="control-button text-2xl">▼</button>
            <button id="portrait-left-btn" class="control-button text-2xl">◀</button>
            <button id="portrait-right-btn" class="control-button text-2xl">►</button>
        </div>

        <div class="w-full text-lg text-center font-bold my-4 p-2 bg-slate-200 rounded-lg shadow-inner flex flex-col md:flex-row justify-around">
            <div>Uang: <span id="moneyDisplay"></span></div>
            <div>Populasi: <span id="populationDisplay"></span></div>
        </div>
        
        <div class="w-full p-2 bg-slate-200 rounded-lg shadow-inner mt-2">
            <label for="taxRateSlider" class="block text-center font-bold">Tingkat Pajak: <span id="taxRateDisplay"></span>%</label>
            <input type="range" id="taxRateSlider" min="0" max="50" value="5" class="w-full mt-1 accent-blue-500" />
        </div>

        <div class="mt-4 w-full flex flex-col items-center">
            <div class="relative w-full flex justify-center">
                <div id="popupMenu" class="popup-menu grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-2 w-full max-w-md">
                    <button id="houseButton" class="action-button" style="background-color: #fde047;">Bangun Rumah</button>
                    <button id="parkButton" class="action-button" style="background-color: #22c55e;">Bangun Taman</button>
                    <button id="storeButton" class="action-button" style="background-color: #f59e0b;">Bangun Toko</button>
                    <button id="industrialButton" class="action-button" style="background-color: #1f2937;">Bangun Industri</button>
                    <button id="roadButton" class="action-button" style="background-color: #64748b;">Bangun Jalan</button>
                    <button id="hospitalButton" class="action-button" style="background-color: #7b241c;">Bangun Rumah Sakit</button>
                </div>
            </div>
            <div class="w-full grid grid-cols-2 md:grid-cols-4 lg:grid-cols-5 gap-2 max-w-full">
                <button id="moveModeButton" class="action-button bg-blue-500 hover:bg-blue-600 transition-colors">
                    Mode Pindah
                </button>
                <button id="destroyModeButton" class="action-button bg-red-500 hover:bg-red-600 transition-colors">
                    Hancurkan
                </button>
                <button id="buildMenuButton" class="action-button bg-gray-600 hover:bg-gray-700 transition-colors">
                    Bangun
                </button>
                <button id="guideButton" class="action-button bg-gray-400 hover:bg-gray-500">Panduan</button>
                <button id="restartButton" class="action-button bg-yellow-500 hover:bg-yellow-600">Mulai Ulang</button>
            </div>
        </div>

    </div>
</div>

<div id="guideModal" class="modal">
    <div class="modal-content">
        <div class="modal-header">
            <h2>Panduan Permainan</h2>
            <button id="modalCloseButton" class="modal-close">&times;</button>
        </div>
        <p>Selamat datang! Ini adalah simulasi pembangunan kota sederhana. Berikut panduan dasar untuk memulai:</p>
        <ul class="guide-list mt-4">
            <li><strong>Mode Pindah:</strong> Gunakan tombol panah di keyboard, atau tombol panah di layar sentuh, untuk menggerakkan pemain (kotak merah) dan menjelajahi peta.</li>
            <li><strong>Membangun Bangunan:</strong> Pilih salah satu tombol bangunan (Rumah, Taman, Toko, Industri, Jalan, Rumah Sakit) lalu klik di kanvas untuk membangunnya. Pastikan Anda memiliki cukup uang!</li>
            <li><strong>Mode Hancurkan:</strong> Pilih tombol Hancurkan, lalu klik di bangunan yang ingin Anda hancurkan. Anda akan mendapatkan setengah dari biaya bangunan kembali.</li>
            <li><strong>Tingkat Pajak:</strong> Sesuaikan tingkat pajak dengan penggeser di bawah kanvas. Tingkat pajak yang lebih tinggi akan meningkatkan uang Anda, tetapi bisa membuat populasi turun.</li>
            <li><strong>Uang dan Populasi:</strong> Perhatikan panel di atas kanvas untuk melihat uang dan populasi Anda saat ini. Bangun rumah untuk meningkatkan populasi. Bangunan seperti Toko, Industri, dan Rumah Sakit akan memberikan keuntungan.</li>
            <li><strong>Koneksi Jalan:</strong> Pastikan bangunan Anda terhubung ke jalan agar warga dan bisnis lebih bahagia dan menguntungkan.</li>
            <li><strong>Mulai Ulang:</strong> Tombol ini akan mereset semua uang, populasi, dan bangunan ke awal permainan. Gunakan jika Anda ingin memulai dari nol.</li>
        </ul>
    </div>
</div>

<script>
    // Game state variables
    let money = 1000.00;
    let population = 0;
    let buildings = [];
    let mapOffset = { x: 0, y: 0 };
    let player = { x: 0, y: 0, width: 28, height: 28, speed: 1.5, color: '#ef4444' };
    let activeMode = 'move';
    let buildingType = null;
    let taxRate = 5;
    let infoBox = { visible: false, x: 0, y: 0, content: '' };
    let isPopupMenuOpen = false;
    
    // Game constants
    const gridSize = 40;
    const incomeInterval = 1000;
    const incomePerPersonPerSecond = 10;
    const influenceRadiusInBlocks = 7;
    let lastIncomeTime = Date.now();
    const keys = {};
    const touchControls = { up: false, down: false, left: false, right: false };

    const buildingStats = {
        house: { cost: 100, population: 5, name: 'Rumah', color: '#fde047' },
        park: { cost: 50, name: 'Taman', color: '#22c55e', maintenance: 10, influenceRadius: 5 },
        store: { cost: 200, name: 'Toko', color: '#f59e0b', baseIncome: 250 },
        industrial: { cost: 300, name: 'Industri', color: '#1f2937', baseIncome: 400 },
        road: { cost: 20, name: 'Jalan', color: '#64748b', maintenance: 1.5 },
        hospital: {
            cost: 500,
            name: 'Rumah Sakit',
            color: '#7b241c',
            maintenance: 30,
            patientCapacity: 10, // New property for patient capacity
            treatmentCost: 5, // New property for treatment cost
            influenceRadius: 10 // Used for happiness calculation only
        }
    };
    
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');
    const moneyDisplay = document.getElementById('moneyDisplay');
    const populationDisplay = document.getElementById('populationDisplay');
    const taxRateDisplay = document.getElementById('taxRateDisplay');
    const taxRateSlider = document.getElementById('taxRateSlider');
    const infoBoxEl = document.getElementById('infoBox');
    const modal = document.getElementById('guideModal');
    const popupMenu = document.getElementById('popupMenu');
    const buildMenuButton = document.getElementById('buildMenuButton');
    const moveModeButton = document.getElementById('moveModeButton');
    const destroyModeButton = document.getElementById('destroyModeButton');

    const buildingButtons = {
        house: document.getElementById('houseButton'),
        park: document.getElementById('parkButton'),
        store: document.getElementById('storeButton'),
        industrial: document.getElementById('industrialButton'),
        road: document.getElementById('roadButton'),
        hospital: document.getElementById('hospitalButton'),
    };
    
    const guideButton = document.getElementById('guideButton');
    const restartButton = document.getElementById('restartButton');
    const modalCloseButton = document.getElementById('modalCloseButton');

    const landscapeControls = {
        up: document.getElementById('landscape-up-btn'),
        down: document.getElementById('landscape-down-btn'),
        left: document.getElementById('landscape-left-btn'),
        right: document.getElementById('landscape-right-btn')
    };

    const portraitControls = {
        up: document.getElementById('portrait-up-btn'),
        down: document.getElementById('portrait-down-btn'),
        left: document.getElementById('portrait-left-btn'),
        right: document.getElementById('portrait-right-btn')
    };
    
    function formatRupiah(amount) {
        return new Intl.NumberFormat('id-ID', {
            style: 'currency',
            currency: 'IDR',
            minimumFractionDigits: 2,
            maximumFractionDigits: 2
        }).format(amount);
    }

    function findBuilding(tileX, tileY) {
        return buildings.find(b =>
            Math.floor(b.x / gridSize) === tileX && Math.floor(b.y / gridSize) === tileY
        );
    }
    
    function isConnectedToRoad(tileX, tileY) {
        const adjacentTiles = [
            { x: tileX, y: tileY - 1 },
            { x: tileX, y: tileY + 1 },
            { x: tileX - 1, y: tileY },
            { x: tileX + 1, y: tileY }
        ];
        return !!adjacentTiles.find(tile => {
            const building = findBuilding(tile.x, tile.y);
            return building && building.type === 'road';
        });
    }

    function calculateNeeds() {
        const influentialBuildings = buildings.filter(b => buildingStats[b.type].influenceRadius);

        buildings.forEach(building => {
            const tileX = Math.floor(building.x / gridSize);
            const tileY = Math.floor(building.y / gridSize);
            const isConnected = isConnectedToRoad(tileX, tileY);

            if (building.type === 'house') {
                let happinessBonus = 0;
                
                influentialBuildings.forEach(influencer => {
                    const distanceX = Math.abs(building.x - influencer.x);
                    const distanceY = Math.abs(building.y - influencer.y);
                    const distanceInBlocks = Math.sqrt(Math.pow(distanceX, 2) + Math.pow(distanceY, 2)) / gridSize;
                    const influenceRadius = buildingStats[influencer.type].influenceRadius;

                    if (distanceInBlocks <= influenceRadius) {
                        if (influencer.type === 'park') {
                            happinessBonus += 15;
                        } else if (influencer.type === 'hospital') {
                            happinessBonus += 25; 
                        }
                    }
                });

                if (isConnected) happinessBonus += 30;
                let taxPenalty = taxRate > 10 ? (taxRate - 10) * 2 : 0;
                happinessBonus -= taxPenalty;
                
                building.needs.happiness = Math.max(0, Math.min(100, Math.floor(happinessBonus + 50)));
            } else if (building.type === 'store' || building.type === 'industrial') {
                const nearbyHouses = buildings.filter(b => {
                    const distanceX = Math.abs(building.x - b.x);
                    const distanceY = Math.abs(building.y - b.y);
                    const distanceInBlocks = Math.sqrt(Math.pow(distanceX, 2) + Math.pow(distanceY, 2)) / gridSize;
                    return b.type === 'house' && distanceInBlocks <= influenceRadiusInBlocks;
                }).length;
                
                let profitabilityBonus = nearbyHouses * 20;
                if (isConnected) profitabilityBonus += 40;
                if (population === 0) profitabilityBonus = 0;
                building.needs.profitability = Math.min(100, profitabilityBonus);
            }
        });
    }

    function gameLoop() {
        if (activeMode === 'move') {
            let moveX = 0, moveY = 0;
            if (keys['arrowup'] || keys['w'] || touchControls.up) moveY -= player.speed;
            if (keys['arrowdown'] || keys['s'] || touchControls.down) moveY += player.speed;
            if (keys['arrowleft'] || keys['a'] || touchControls.left) moveX -= player.speed;
            if (keys['arrowright'] || keys['d'] || touchControls.right) moveX += player.speed;
            
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

        if (Date.now() - lastIncomeTime > incomeInterval) {
            let totalIncome = 0;
            let totalExpenditure = 0;
            
            buildings.forEach(b => {
                const stats = buildingStats[b.type];
                
                if (b.type === 'house') {
                    totalIncome += b.population * incomePerPersonPerSecond * (taxRate / 100);
                } else if (b.type === 'store' || b.type === 'industrial') {
                    totalIncome += stats.baseIncome * (b.needs.profitability / 100) * (taxRate / 100);
                } else if (b.type === 'hospital') {
                    // FIX BUG #2: Implementasi formula baru untuk pendapatan rumah sakit
                    totalIncome += b.currentPatients * stats.treatmentCost * (taxRate / 100);
                }
                
                // Biaya pemeliharaan adalah pengeluaran terpisah
                if (stats.maintenance) {
                    totalExpenditure += stats.maintenance;
                }
            });
            money += totalIncome;
            money -= totalExpenditure;
            lastIncomeTime = Date.now();
        }

        moneyDisplay.textContent = formatRupiah(money);
        populationDisplay.textContent = population;

        ctx.fillStyle = '#f8fafc';
        ctx.fillRect(0, 0, canvas.width, canvas.height);

        ctx.strokeStyle = '#94a3b8';
        ctx.lineWidth = 1;
        const screenGridSizeX = Math.ceil(canvas.width / gridSize) + 1;
        const screenGridSizeY = Math.ceil(canvas.height / gridSize) + 1;
        for (let x = 0; x < screenGridSizeX; x++) {
            const drawX = (x * gridSize) - (mapOffset.x % gridSize);
            ctx.beginPath();
            ctx.moveTo(drawX, 0);
            ctx.lineTo(drawX, canvas.height);
            ctx.stroke();
        }
        for (let y = 0; y < screenGridSizeY; y++) {
            const drawY = (y * gridSize) - (mapOffset.y % gridSize);
            ctx.beginPath();
            ctx.moveTo(0, drawY);
            ctx.lineTo(canvas.width, drawY);
            ctx.stroke();
        }

        buildings.forEach(building => {
            const drawX = building.x - mapOffset.x;
            const drawY = building.y - mapOffset.y;
            if (drawX + gridSize < 0 || drawX > canvas.width || drawY + gridSize < 0 || drawY > canvas.height) return;
            ctx.fillStyle = building.color;
            ctx.fillRect(drawX, drawY, gridSize, gridSize);
            if (building.type !== 'road') {
                ctx.strokeStyle = '#334155';
                ctx.strokeRect(drawX, drawY, gridSize, gridSize);
            }
        });

        const playerScreenX = player.x - mapOffset.x;
        const playerScreenY = player.y - mapOffset.y;
        ctx.fillStyle = player.color;
        ctx.fillRect(playerScreenX, playerScreenY, player.width, player.height);

        const playerTileX = Math.floor(player.x / gridSize);
        const playerTileY = Math.floor(player.y / gridSize);
        const buildingFound = findBuilding(playerTileX, playerTileY);

        if (buildingFound) {
            let infoText = `
                <h3 class="font-bold text-lg mb-1">${buildingStats[buildingFound.type].name}</h3>
                <p>Posisi: (${Math.floor(buildingFound.x/gridSize)}, ${Math.floor(buildingFound.y/gridSize)})</p>
            `;
            const stats = buildingStats[buildingFound.type];
            if (buildingFound.type === 'house') {
                infoText += `<p>Populasi: ${buildingFound.population} orang</p>`;
                infoText += `<p>Kebahagiaan Warga: ${buildingFound.needs.happiness}%</p>`;
                const taxPerHouse = (buildingFound.population * incomePerPersonPerSecond) * (taxRate / 100);
                infoText += `<p>Pajak Bangunan: ${formatRupiah(taxPerHouse)}/detik</p>`;
            } else if (buildingFound.type === 'store' || buildingFound.type === 'industrial') {
                const profitPerBuilding = stats.baseIncome * (buildingFound.needs.profitability / 100) * (taxRate / 100);
                infoText += `<p>Profitabilitas: ${buildingFound.needs.profitability}%</p>`;
                infoText += `<p>Pajak Bangunan: ${formatRupiah(profitPerBuilding)}/detik</p>`;
            } else if (buildingFound.type === 'hospital') {
                // FIX BUG #1: Menampilkan kapasitas pasien, bukan radius pengaruh
                infoText += `<p>Kapasitas Pasien: ${stats.patientCapacity} orang</p>`;
                infoText += `<p>Pasien Saat Ini: ${buildingFound.currentPatients} orang</p>`;
                
                // FIX BUG #1 & #2: Menampilkan pajak bangunan dan biaya perawatan secara terpisah
                const hospitalTax = buildingFound.currentPatients * stats.treatmentCost * (taxRate / 100);
                const hospitalMaintenance = stats.maintenance;
                infoText += `<p>Pajak Bangunan: ${formatRupiah(hospitalTax)}/detik</p>`;
                infoText += `<p>Biaya Perawatan: ${formatRupiah(hospitalMaintenance)}/detik</p>`;
            }
            if (stats.maintenance && buildingFound.type !== 'hospital') {
                infoText += `<p>Biaya Perawatan: ${formatRupiah(stats.maintenance)}/detik</p>`;
            }

            infoBoxEl.innerHTML = infoText;
            infoBoxEl.style.left = `${playerScreenX + player.width + 10}px`;
            infoBoxEl.style.top = `${playerScreenY}px`;
            infoBoxEl.classList.remove('opacity-0', 'hidden');
        } else {
            infoBoxEl.classList.add('opacity-0', 'hidden');
        }

        requestAnimationFrame(gameLoop);
    }

    function togglePopupMenu() {
        isPopupMenuOpen = !isPopupMenuOpen;
        if (isPopupMenuOpen) {
            popupMenu.classList.add('show');
        } else {
            popupMenu.classList.remove('show');
        }
    }

    function setMode(newMode, newType) {
        activeMode = newMode;
        if (newType) {
            buildingType = newType;
        }
        updateButtonStyles();
        if (newMode === 'build') {
            togglePopupMenu();
        }
    }
    
    function updateButtonStyles() {
        moveModeButton.classList.remove('mode-active');
        destroyModeButton.classList.remove('mode-active');
        buildMenuButton.classList.remove('mode-active');
        
        for (const btn in buildingButtons) {
            const buttonEl = buildingButtons[btn];
            if (buttonEl) {
                buttonEl.classList.remove('mode-active');
            }
        }

        if (activeMode === 'move') {
            moveModeButton.classList.add('mode-active');
        } else if (activeMode === 'destroy') {
            destroyModeButton.classList.add('mode-active');
        } else if (activeMode === 'build' && buildingType) {
            buildMenuButton.classList.add('mode-active');
            if (buildingButtons[buildingType]) {
                buildingButtons[buildingType].classList.add('mode-active');
            }
        }
    }

    function restartGame() {
        money = 1000.00;
        population = 0;
        buildings = [];
        mapOffset = { x: 0, y: 0 };
        player = { x: 0, y: 0, width: 28, height: 28, speed: 1.5, color: '#ef4444' };
        activeMode = 'move';
        buildingType = null;
        taxRate = 5;
        infoBox = { visible: false, x: 0, y: 0, content: '' };
        taxRateDisplay.textContent = taxRate;
        taxRateSlider.value = taxRate;
        updateButtonStyles();
    }

    function init() {
        window.addEventListener('keydown', (e) => {
            keys[e.key.toLowerCase()] = true;
            if (e.key.toLowerCase() === 'm') setMode('move', null);
            else if (e.key.toLowerCase() === 'h') setMode('build', 'house');
            else if (e.key.toLowerCase() === 'p') setMode('build', 'park');
            else if (e.key.toLowerCase() === 't') setMode('build', 'store');
            else if (e.key.toLowerCase() === 'i') setMode('build', 'industrial');
            else if (e.key.toLowerCase() === 'r') setMode('build', 'road');
            else if (e.key.toLowerCase() === 'x') setMode('destroy', null);
            else if (e.key.toLowerCase() === 'o') setMode('build', 'hospital');
        });

        window.addEventListener('keyup', (e) => {
            keys[e.key.toLowerCase()] = false;
        });

        const allMobileButtons = [
            landscapeControls.up, landscapeControls.down, landscapeControls.left, landscapeControls.right,
            portraitControls.up, portraitControls.down, portraitControls.left, portraitControls.right
        ];

        allMobileButtons.forEach(btn => {
            if (btn) {
                btn.addEventListener('touchstart', (e) => {
                    e.preventDefault();
                    if (e.target.id.includes('up')) touchControls.up = true;
                    if (e.target.id.includes('down')) touchControls.down = true;
                    if (e.target.id.includes('left')) touchControls.left = true;
                    if (e.target.id.includes('right')) touchControls.right = true;
                });
                btn.addEventListener('touchend', () => {
                    touchControls.up = false;
                    touchControls.down = false;
                    touchControls.left = false;
                    touchControls.right = false;
                });
            }
        });

        canvas.addEventListener('click', (e) => {
            const rect = canvas.getBoundingClientRect();
            const scaleX = canvas.width / rect.width;
            const scaleY = canvas.height / rect.height;
            const mouseX = (e.clientX - rect.left) * scaleX;
            const mouseY = (e.clientY - rect.top) * scaleY;
            const tileX = Math.floor((mouseX + mapOffset.x) / gridSize);
            const tileY = Math.floor((mouseY + mapOffset.y) / gridSize);

            if (activeMode === 'build') {
                const existingBuilding = findBuilding(tileX, tileY);
                const stats = buildingStats[buildingType];
                const cost = stats.cost;
                if (!existingBuilding && money >= cost) {
                    const newBuilding = {
                        id: Date.now(), x: tileX * gridSize, y: tileY * gridSize, type: buildingType, color: stats.color,
                        population: stats.population || 0, 
                        currentPatients: buildingType === 'hospital' ? 0 : null, // Initialize currentPatients for hospital
                        needs: { happiness: 0, profitability: 0 }
                    };
                    buildings.push(newBuilding);
                    money -= cost;
                    calculateNeeds();
                } else if (existingBuilding) {
                    infoBoxEl.innerHTML = `<p class="text-red-500">Sudah ada bangunan di sini!</p>`;
                    infoBoxEl.classList.remove('opacity-0', 'hidden');
                    infoBoxEl.style.left = `${mouseX + 10}px`;
                    infoBoxEl.style.top = `${mouseY}px`;
                    setTimeout(() => infoBoxEl.classList.add('opacity-0', 'hidden'), 1000);
                } else if (money < cost) {
                    infoBoxEl.innerHTML = `<p class="text-red-500">Uang tidak cukup! Biaya: ${formatRupiah(cost)}</p>`;
                    infoBoxEl.classList.remove('opacity-0', 'hidden');
                    infoBoxEl.style.left = `${mouseX + 10}px`;
                    infoBoxEl.style.top = `${mouseY}px`;
                    setTimeout(() => infoBoxEl.classList.add('opacity-0', 'hidden'), 1000);
                }
            } else if (activeMode === 'destroy') {
                const buildingIndex = buildings.findIndex(b =>
                    Math.floor(b.x / gridSize) === tileX && Math.floor(b.y / gridSize) === tileY
                );
                if (buildingIndex !== -1) {
                    const destroyedBuilding = buildings.splice(buildingIndex, 1)[0];
                    const refund = buildingStats[destroyedBuilding.type].cost * 0.5;
                    money += refund;
                    calculateNeeds();
                }
            }
        });

        taxRateSlider.addEventListener('input', (e) => {
            taxRate = parseInt(e.target.value);
            taxRateDisplay.textContent = taxRate;
            calculateNeeds();
        });

        buildMenuButton.addEventListener('click', togglePopupMenu);
        moveModeButton.addEventListener('click', () => setMode('move', null));
        destroyModeButton.addEventListener('click', () => setMode('destroy', null));
        restartButton.addEventListener('click', restartGame);
        guideButton.addEventListener('click', () => { modal.classList.add('modal-show'); });
        modalCloseButton.addEventListener('click', () => { modal.classList.remove('modal-show'); });
        window.addEventListener('click', (event) => {
            if (event.target === modal) {
                modal.classList.remove('modal-show');
            }
            if (isPopupMenuOpen && !popupMenu.contains(event.target) && !buildMenuButton.contains(event.target)) {
                togglePopupMenu();
            }
        });
        
        for (const type in buildingButtons) {
            buildingButtons[type].addEventListener('click', () => setMode('build', type));
        }

        const resizeCanvas = () => {
            canvas.width = Math.min(800, window.innerWidth - 40);
            canvas.height = canvas.width * 0.75;
            gameLoop();
        };
        window.addEventListener('resize', resizeCanvas);
        resizeCanvas();

        restartGame();

        setInterval(() => {
            buildings.filter(b => b.type === 'house').forEach(house => {
                let changeAmount = 0;
                // Logika perubahan populasi berdasarkan kebahagiaan
                if (house.needs.happiness >= 80) {
                    changeAmount = Math.floor(Math.random() * 2) + 1; // Populasi meningkat
                } else if (house.needs.happiness < 50) {
                    changeAmount = -(Math.floor(Math.random() * 2) + 1); // Populasi menurun
                }
                
                let newPopulation = house.population + changeAmount;
                if (taxRate > 40 && newPopulation < 0) newPopulation = 0;
                else if (newPopulation < 1) newPopulation = 1;
                house.population = Math.min(buildingStats.house.population, newPopulation);
            });
            let totalPopulation = 0;
            buildings.forEach(b => {
                if (b.type === 'house') totalPopulation += b.population;
            });
            population = totalPopulation;

            // FIX BUG #2: Logic to dynamically change the number of patients
            buildings.filter(b => b.type === 'hospital').forEach(hospital => {
                const stats = buildingStats.hospital;
                // Randomly add or remove patients
                const change = Math.random() < 0.5 ? -1 : 1;
                let newPatients = hospital.currentPatients + change;
                // Clamp the number of patients to be within the capacity and not negative
                hospital.currentPatients = Math.max(0, Math.min(stats.patientCapacity, newPatients));
            });

        }, 5000);
        setInterval(calculateNeeds, 2000);

        gameLoop();
    }

    window.onload = init;
</script>

</body>
</html>
