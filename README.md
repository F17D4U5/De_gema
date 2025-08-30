<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Simulasi Kota 2D Sederhana</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        /* General CSS for layout and styling */
        body {
            font-family: 'Inter', sans-serif;
            margin: 0;
            background-color: #f3f4f6;
            min-height: 100vh;
            display: flex;
            justify-content: center;
            align-items: flex-start;
        }
        .scroll-container {
            width: 100%;
            max-width: 800px;
            padding: 1rem;
            display: flex;
            flex-direction: column;
            align-items: center;
            overflow-y: auto;
            -webkit-overflow-scrolling: touch;
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
            max-height: 80vh;
            overflow-y: auto;
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
        /* Style for the temporary message box */
        .message-box {
            position: fixed;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background-color: #333;
            color: white;
            padding: 1rem 2rem;
            border-radius: 0.5rem;
            box-shadow: 0 4px 12px rgba(0, 0, 0, 0.3);
            z-index: 101;
            opacity: 0;
            visibility: hidden;
            transition: opacity 0.3s ease-in-out, visibility 0.3s ease-in-out;
        }
        .message-box.show {
            opacity: 1;
            visibility: visible;
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
            <div>Pekerja Tersedia: <span id="availableWorkersDisplay"></span></div>
            <div>Daya Tersedia: <span id="powerDisplay"></span></div>
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
                    <button id="windTurbineButton" class="action-button" style="background-color: #3b82f6;">Bangun Kincir Angin</button>
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
            <button id="guideModalCloseButton" class="modal-close">&times;</button>
        </div>
        <p>Selamat datang! Ini adalah simulasi pembangunan kota sederhana. Berikut panduan dasar untuk memulai:</p>
        <ul class="guide-list mt-4">
            <li><strong>Pergerakan (Tombol Panah):</strong> Gunakan tombol panah di keyboard, atau tombol panah di layar sentuh, untuk menggerakkan pemain (kotak merah) dan menjelajahi peta. Dalam mode ini, klik bangunan untuk melihat informasinya.</li>
            <li><strong>Pintasan Keyboard:</strong>
                <ul>
                    <li>'M' untuk <strong>Mode Pindah</strong></li>
                    <li>'H' untuk <strong>Bangun Rumah</strong></li>
                    <li>'P' untuk <strong>Bangun Taman</strong></li>
                    <li>'T' untuk <strong>Bangun Toko</strong></li>
                    <li>'I' untuk <strong>Bangun Industri</strong></li>
                    <li>'J' untuk <strong>Bangun Jalan</strong></li>
                    <li>'O' untuk <strong>Bangun Rumah Sakit</strong></li>
                    <li>'L' untuk <strong>Bangun Kincir Angin</strong></li>
                    <li>'X' untuk <strong>Mode Hancurkan</strong></li>
                    <li>'G' untuk menampilkan <strong>Panduan</strong> ini</li>
                    <li>'R' untuk <strong>Mulai Ulang</strong> permainan</li>
                </ul>
            </li>
            <li><strong>Membangun Bangunan:</strong> Pilih salah satu tombol bangunan lalu klik di kanvas untuk membangunnya. Pastikan Anda memiliki cukup uang!</li>
            <li><strong>Mode Hancurkan:</strong> Pilih tombol Hancurkan, lalu klik di bangunan yang ingin Anda hancurkan. Anda akan mendapatkan setengah dari biaya bangunan kembali.</li>
            <li><strong>Tingkat Pajak:</strong> Sesuaikan tingkat pajak dengan penggeser di bawah kanvas. Tingkat pajak yang lebih tinggi akan meningkatkan uang Anda, tetapi bisa membuat populasi turun.</li>
            <li><strong>Uang dan Populasi:</strong> Perhatikan panel di atas kanvas untuk melihat uang dan populasi Anda saat ini. Bangun rumah untuk meningkatkan populasi, dan bangun toko atau industri untuk memberikan lapangan pekerjaan bagi populasi.</li>
            <li><strong>Pekerja:</strong> Bangunan bisnis hanya akan menghasilkan uang jika Anda memiliki cukup populasi untuk mengisi semua posisi pekerjaan yang tersedia.</li>
            <li><strong>Koneksi Jalan:</strong> Pastikan bangunan Anda terhubung ke jalan agar warga dan bisnis lebih bahagia dan menguntungkan. Koneksi ini juga penting untuk mendapatkan akses ke pasokan listrik dari Pembangkit Listrik.</li>
            <li><strong>Kebutuhan Listrik:</strong> Bangunan seperti Toko, Industri, dan Rumah Sakit membutuhkan listrik untuk beroperasi. Anda harus membangun **Pembangkit Listrik** untuk memenuhi kebutuhan ini. Pendapatan dari pembangkit listrik berasal dari penjualan listrik ke bangunan lain.</li>
            <li><strong>Mulai Ulang:</strong> Tombol ini akan mereset semua uang, populasi, dan bangunan ke awal permainan. Gunakan jika Anda ingin memulai dari nol.</li>
        </ul>
    </div>
</div>

<!-- New modal to display building info -->
<div id="infoModal" class="modal">
    <div class="modal-content">
        <div class="modal-header">
            <h2 id="infoModalTitle">Informasi Bangunan</h2>
            <button id="infoModalCloseButton" class="modal-close">&times;</button>
        </div>
        <div id="infoModalContent">
            <!-- Info content will be inserted here by JavaScript -->
        </div>
    </div>
</div>

<!-- Message box for temporary messages -->
<div id="messageBox" class="message-box"></div>

<script>
    // Game state variables
    let money = 1000.00;
    let population = 0;
    let buildings = [];
    let mapOffset = { x: 0, y: 0 };
    let player = { x: 0, y: 0, width: 28, height: 28, speed: 1.0, color: '#ef4444' };
    let activeMode = 'move';
    let buildingType = null;
    let taxRate = 5;
    let isPopupMenuOpen = false;
    let selectedBuilding = null; 
    let totalPowerOutput = 0;
    let totalPowerUsage = 0;

    // Game constants
    const gridSize = 40;
    const incomeInterval = 1000;
    const incomePerPersonPerSecond = 10;
    const pricePerUnitPower = 0.5;
    const influenceRadiusInBlocks = 7;
    let lastIncomeTime = Date.now();
    const keys = {};
    const touchControls = { up: false, down: false, left: false, right: false };

    // Base income values per worker
    const baseIncomePerWorker = {
        store: 16.67,
        industrial: 25.00,
        windTurbine: 0
    };

    const buildingStats = {
        house: { cost: 100, populationCapacity: 5, name: 'Rumah', color: '#fde047' },
        park: { cost: 50, name: 'Taman', color: '#22c55e', maintenance: 10, influenceRadius: 5 },
        store: { cost: 200, name: 'Toko', color: '#f59e0b', workersRequired: 3, powerRequired: 10 },
        industrial: { cost: 300, name: 'Industri', color: '#1f2937', workersRequired: 8, powerRequired: 25 },
        road: { cost: 20, name: 'Jalan', color: '#64748b', maintenance: 1.5 },
        hospital: {
            cost: 500,
            name: 'Rumah Sakit',
            color: '#7b241c',
            maintenance: 30,
            patientCapacity: 100,
            treatmentCost: 10,
            influenceRadius: 10,
            powerRequired: 20
        },
        windTurbine: { cost: 400, name: 'Kincir Angin', color: '#3b82f6', maintenance: 10, powerOutput: 40 }
    };
    
    // UI elements
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');
    const moneyDisplay = document.getElementById('moneyDisplay');
    const populationDisplay = document.getElementById('populationDisplay');
    const availableWorkersDisplay = document.getElementById('availableWorkersDisplay');
    const powerDisplay = document.getElementById('powerDisplay');
    const taxRateDisplay = document.getElementById('taxRateDisplay');
    const taxRateSlider = document.getElementById('taxRateSlider');
    const infoModal = document.getElementById('infoModal');
    const infoModalCloseButton = document.getElementById('infoModalCloseButton');
    const infoModalContent = document.getElementById('infoModalContent');
    const guideModal = document.getElementById('guideModal');
    const guideModalCloseButton = document.getElementById('guideModalCloseButton');
    const popupMenu = document.getElementById('popupMenu');
    const buildMenuButton = document.getElementById('buildMenuButton');
    const moveModeButton = document.getElementById('moveModeButton');
    const destroyModeButton = document.getElementById('destroyModeButton');
    const messageBox = document.getElementById('messageBox');

    const buildingButtons = {
        house: document.getElementById('houseButton'),
        park: document.getElementById('parkButton'),
        store: document.getElementById('storeButton'),
        industrial: document.getElementById('industrialButton'),
        road: document.getElementById('roadButton'),
        hospital: document.getElementById('hospitalButton'),
        windTurbine: document.getElementById('windTurbineButton'),
    };
    
    const guideButton = document.getElementById('guideButton');
    const restartButton = document.getElementById('restartButton');

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
    
    // Function to format money to IDR
    function formatRupiah(amount) {
        return new Intl.NumberFormat('id-ID', {
            style: 'currency',
            currency: 'IDR',
            minimumFractionDigits: 2,
            maximumFractionDigits: 2
        }).format(amount);
    }

    // Function to show a temporary message
    function showMessage(text) {
        messageBox.textContent = text;
        messageBox.classList.add('show');
        setTimeout(() => {
            messageBox.classList.remove('show');
        }, 3000);
    }

    // Function to find a building at a specific tile
    function findBuilding(tileX, tileY) {
        return buildings.find(b =>
            Math.floor(b.x / gridSize) === tileX && Math.floor(b.y / gridSize) === tileY
        );
    }
    
    // Function to check for road connection
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

    // Function to calculate building needs and bonuses
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
            }
        });
    }

    // --- FUNCTION TO DISPLAY THE INFO MODAL ---
    function showInfoModal(building) {
        let infoText = `
            <h3 class="font-bold text-lg mb-1">${buildingStats[building.type].name}</h3>
            <p>Posisi: (${Math.floor(building.x/gridSize)}, ${Math.floor(building.y/gridSize)})</p>
        `;
        const stats = buildingStats[building.type];
        const isPowered = building.isPowered;

        if (building.type === 'house') {
            infoText += `<p>Populasi: ${building.population} orang</p>`;
            infoText += `<p>Kapasitas Maks: ${stats.populationCapacity} orang</p>`;
            infoText += `<p>Kebahagiaan Warga: ${building.needs.happiness}%</p>`;
            const taxPerHouse = (building.population * incomePerPersonPerSecond) * (taxRate / 100);
            infoText += `<p>Pajak Bangunan: ${formatRupiah(taxPerHouse)}/detik</p>`;
        } else if (stats.workersRequired) {
            infoText += `<p>Status Listrik: ${isPowered ? 'Tersedia' : 'Tidak Tersedia'}</p>`;
            infoText += `<p>Pekerja Dibutuhkan: ${stats.workersRequired}</p>`;
            infoText += `<p>Pekerja Ditugaskan: ${building.workersAssigned || 0}</p>`;
            const taxGain = (baseIncomePerWorker[building.type] * (building.workersAssigned || 0)) * (taxRate / 100);
            infoText += `<p>Pajak Bangunan: ${formatRupiah(taxGain)}/detik</p>`;
        } else if (building.type === 'hospital') {
            infoText += `<p>Status Listrik: ${isPowered ? 'Tersedia' : 'Tidak Tersedia'}</p>`;
            infoText += `<p>Kapasitas Pasien: ${stats.patientCapacity} orang</p>`;
            infoText += `<p>Pasien Saat Ini: ${building.currentPatients} orang</p>`;
            const taxGain = (building.currentPatients || 0) * stats.treatmentCost;
            infoText += `<p>Pajak Pengobatan: ${formatRupiah(taxGain)}/detik</p>`;
            infoText += `<p>Biaya Perawatan: ${formatRupiah(stats.maintenance)}/detik</p>`;
        } else if (building.type === 'windTurbine') {
            const powerIncome = (building.powerSold || 0) * pricePerUnitPower;
            infoText += `<p>Kapasitas Daya: ${stats.powerOutput}</p>`;
            infoText += `<p>Pendapatan Jual Daya: ${formatRupiah(powerIncome)}/detik</p>`;
            infoText += `<p>Biaya Perawatan: ${formatRupiah(stats.maintenance)}/detik</p>`;
        }
        if (stats.maintenance && building.type !== 'windTurbine' && building.type !== 'hospital') {
            infoText += `<p>Biaya Perawatan: ${formatRupiah(stats.maintenance)}/detik</p>`;
        }

        infoModalContent.innerHTML = infoText;
        infoModal.classList.add('modal-show');
    }
    
    function gameLoop() {
        // Player movement logic (only arrow keys)
        let moveX = 0, moveY = 0;
        if (keys['arrowup'] || touchControls.up) moveY -= player.speed;
        if (keys['arrowdown'] || touchControls.down) moveY += player.speed;
        if (keys['arrowleft'] || touchControls.left) moveX -= player.speed;
        if (keys['arrowright'] || touchControls.right) moveX += player.speed;

        let playerScreenX = player.x - mapOffset.x;
        let playerScreenY = player.y - mapOffset.y;
        
        // Adjust map offset to keep player visible
        const edgeBuffer = 200;

        if (playerScreenX + moveX < edgeBuffer) mapOffset.x -= player.speed;
        if (playerScreenX + moveX > canvas.width - edgeBuffer) mapOffset.x += player.speed;
        if (playerScreenY + moveY < edgeBuffer) mapOffset.y -= player.speed;
        if (playerScreenY + moveY > canvas.height - edgeBuffer) mapOffset.y += player.speed;
        
        // Update player position
        player.x += moveX;
        player.y += moveY;
        const worldSize = 5000;
        player.x = Math.max(0, Math.min(worldSize - player.width, player.x));
        player.y = Math.min(Math.max(0, player.y), worldSize - player.height);

        // Calculate total population
        let totalPopulation = 0;
        buildings.forEach(b => {
            if (b.type === 'house') {
                totalPopulation += b.population;
            }
        });
        population = totalPopulation;
        
        // Assign workers to buildings that need them
        let workersAssigned = 0;
        const businessBuildings = buildings.filter(b => buildingStats[b.type].workersRequired);
        
        // Reset workers assigned for all business buildings
        businessBuildings.forEach(b => b.workersAssigned = 0);

        let availableWorkers = population;
        for (const b of businessBuildings) {
            const workersNeeded = buildingStats[b.type].workersRequired;
            const workersToAssign = Math.min(workersNeeded, Math.max(0, availableWorkers));
            b.workersAssigned = workersToAssign;
            availableWorkers -= workersToAssign;
            workersAssigned += workersToAssign;
        }

        // Calculate total power output and usage
        totalPowerOutput = 0;
        totalPowerUsage = 0;
        buildings.filter(b => b.type === 'windTurbine').forEach(p => {
            totalPowerOutput += buildingStats.windTurbine.powerOutput;
        });

        // Set isPowered status for buildings and calculate total usage
        buildings.forEach(b => {
            const stats = buildingStats[b.type];
            if (stats.powerRequired) {
                const tileX = Math.floor(b.x / gridSize);
                const tileY = Math.floor(b.y / gridSize);
                if (isConnectedToRoad(tileX, tileY) && totalPowerOutput > totalPowerUsage) {
                    b.isPowered = true;
                    totalPowerUsage += stats.powerRequired;
                } else {
                    b.isPowered = false;
                }
            } else {
                b.isPowered = true;
            }
        });
        
        powerDisplay.textContent = `${totalPowerUsage} / ${totalPowerOutput}`;
        
        // Calculate income and expenses per second
        if (Date.now() - lastIncomeTime > incomeInterval) {
            let totalIncome = 0;
            let totalExpenditure = 0;
            
            buildings.forEach(b => {
                const stats = buildingStats[b.type];
                
                if (b.type === 'house') {
                    totalIncome += b.population * incomePerPersonPerSecond * (taxRate / 100);
                } else if (stats.workersRequired) {
                    if (b.isPowered && b.workersAssigned > 0) {
                        const taxGain = (baseIncomePerWorker[b.type] * b.workersAssigned) * (taxRate / 100);
                        totalIncome += taxGain;
                    }
                } else if (b.type === 'hospital') {
                    if (b.isPowered) {
                        const taxGain = (b.currentPatients || 0) * stats.treatmentCost;
                        totalIncome += taxGain * (taxRate / 100);
                    }
                    totalExpenditure += stats.maintenance;
                } else if (b.type === 'windTurbine') {
                    b.powerSold = b.isPowered ? totalPowerUsage : 0;
                    totalIncome += b.powerSold * pricePerUnitPower;
                    totalExpenditure += stats.maintenance;
                }
                
                if (stats.maintenance && b.type !== 'windTurbine' && b.type !== 'hospital') {
                    totalExpenditure += stats.maintenance;
                }
            });
            money += totalIncome;
            money -= totalExpenditure;
            lastIncomeTime = Date.now();
        }

        moneyDisplay.textContent = formatRupiah(money);
        populationDisplay.textContent = population;
        availableWorkersDisplay.textContent = Math.max(0, population - workersAssigned);

        ctx.fillStyle = '#f8fafc';
        ctx.fillRect(0, 0, canvas.width, canvas.height);

        // Draw the grid lines
        ctx.strokeStyle = '#94a3b8';
        ctx.lineWidth = 1;
        const screenGridSizeX = Math.ceil(canvas.width / gridSize) + 1;
        const screenGridSizeY = Math.ceil(canvas.height / gridSize) + 1;
        
        // Draw vertical lines
        for (let x = 0; x < screenGridSizeX; x++) {
            const drawX = (x * gridSize) - (mapOffset.x % gridSize);
            ctx.beginPath();
            ctx.moveTo(drawX, 0);
            ctx.lineTo(drawX, canvas.height);
            ctx.stroke();
        }
        
        // Draw horizontal lines
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

            if (building.type === 'house') {
                drawHouse(drawX, drawY, gridSize, gridSize, building.color);
            } else if (building.type === 'store') {
                drawStore(drawX, drawY, gridSize, gridSize, building.color);
            } else if (building.type === 'industrial') {
                drawIndustrial(drawX, drawY, gridSize, gridSize, building.color);
            } else if (building.type === 'park') {
                drawPark(drawX, drawY, gridSize, gridSize, building.flowers);
            } else if (building.type === 'hospital') {
                drawHospital(drawX, drawY, gridSize, gridSize, building.color);
            } else if (building.type === 'windTurbine') {
                drawWindTurbine(drawX, drawY, gridSize, gridSize);
            } else if (building.type === 'road') {
                drawRoad(building.x, building.y, gridSize, gridSize, building.color);
            } else {
                ctx.fillRect(drawX, drawY, gridSize, gridSize);
                ctx.strokeStyle = '#334155';
                ctx.strokeRect(drawX, drawY, gridSize, gridSize);
            }
        });
        
        playerScreenX = player.x - mapOffset.x;
        playerScreenY = player.y - mapOffset.y;
        
        ctx.fillStyle = player.color;
        ctx.fillRect(playerScreenX, playerScreenY, player.width, player.height);

        requestAnimationFrame(gameLoop);
    }

    /**
     * @param {number} x - X position on canvas.
     * @param {number} y - Y position on canvas.
     * @param {number} width - Width of the house.
     * @param {number} height - Height of the house.
     * @param {string} color - Color of the house.
     */
    function drawHouse(x, y, width, height, color) {
        ctx.fillStyle = color;
        ctx.beginPath();
        ctx.moveTo(x, y + height);
        ctx.lineTo(x, y + height * 0.4);
        ctx.lineTo(x + width * 0.5, y);
        ctx.lineTo(x + width, y + height * 0.4);
        ctx.lineTo(x + width, y + height);
        ctx.closePath();
        ctx.fill();
        ctx.strokeStyle = '#334155';
        ctx.stroke();

        ctx.fillStyle = '#6b7280';
        ctx.fillRect(x + width * 0.7, y + height * 0.1, width * 0.1, height * 0.2);
        
        ctx.fillStyle = '#4a5568';
        ctx.fillRect(x + width * 0.4, y + height * 0.6, width * 0.2, height * 0.4);
        
        ctx.fillStyle = '#94a3b8';
        ctx.fillRect(x + width * 0.15, y + height * 0.5, width * 0.2, height * 0.2);
        ctx.fillRect(x + width * 0.65, y + height * 0.5, width * 0.2, height * 0.2);
    }
    
    /**
     * @param {number} x - X position on canvas.
     * @param {number} y - Y position on canvas.
     * @param {number} width - Width of the store.
     * @param {number} height - Height of the store.
     * @param {string} color - Color of the store.
     */
    function drawStore(x, y, width, height, color) {
        // Draw the main body of the building
        ctx.fillStyle = color;
        ctx.fillRect(x, y + height * 0.2, width, height * 0.8);
        ctx.strokeStyle = '#334155';
        ctx.strokeRect(x, y + height * 0.2, width, height * 0.8);

        // Draw the roof
        ctx.fillStyle = '#334155';
        ctx.fillRect(x, y, width, height * 0.2);
        
        // Draw the chimney/roof decoration
        ctx.fillStyle = '#6b7280';
        ctx.fillRect(x + width * 0.8, y, width * 0.1, height * 0.2);

        // Add windows
        ctx.fillStyle = '#94a3b8';
        ctx.fillRect(x + width * 0.2, y + height * 0.4, width * 0.2, height * 0.2);
        ctx.fillRect(x + width * 0.6, y + height * 0.4, width * 0.2, height * 0.2);
        
        // Add a door
        ctx.fillStyle = '#4a5568';
        ctx.fillRect(x + width * 0.45, y + height * 0.6, width * 0.1, height * 0.4);
    }

    /**
     * Draws an industrial building with a single slanted roof and chimney,
     * precisely matching the user's reference image.
     * @param {number} x - X coordinate of the top-left corner of the building.
     * @param {number} y - Y coordinate of the top-left corner of the building.
     * @param {number} width - Total width of the building.
     * @param {number} height - Total height of the building.
     * @param {string} color - Main color of the building.
     */
    function drawIndustrial(x, y, width, height, color) {
        // Draw the main body of the building, which is a rectangle
        ctx.fillStyle = color;
        ctx.fillRect(x, y + height * 0.5, width, height * 0.5);

        // Draw the single-slanted roof
        ctx.fillStyle = '#4b5563'; // Roof color
        ctx.beginPath();
        ctx.moveTo(x, y + height * 0.5);
        ctx.lineTo(x + width, y + height * 0.2);
        ctx.lineTo(x + width, y + height * 0.5);
        ctx.lineTo(x, y + height * 0.5);
        ctx.closePath();
        ctx.fill();
        ctx.strokeStyle = '#374151'; // Roof outline
        ctx.stroke();

        // Draw the tall chimney on the left side of the roof
        ctx.fillStyle = '#9ca3af'; // Chimney color
        ctx.fillRect(x + width * 0.1, y + height * 0.15, width * 0.08, height * 0.35);

        // Add windows to the main body
        ctx.fillStyle = '#d1d5db';
        ctx.fillRect(x + width * 0.2, y + height * 0.65, width * 0.2, height * 0.15);
        ctx.fillRect(x + width * 0.6, y + height * 0.65, width * 0.2, height * 0.15);
    }
    
    /**
     * Draws a park with static flowers.
     * @param {number} x - X position on canvas.
     * @param {number} y - Y position on canvas.
     * @param {number} width - Width of the park.
     * @param {number} height - Height of the park.
     * @param {Array} flowers - The array of flower data to draw.
     */
    function drawPark(x, y, width, height, flowers) {
        // Draw the thin green ground strip
        const grassHeight = 2;
        ctx.fillStyle = '#4CAF50';
        ctx.fillRect(x, y + height - grassHeight, width, grassHeight);
        
        // Draw the flowers based on the pre-calculated data
        flowers.forEach(flower => {
            // Draw stem as a line
            ctx.strokeStyle = flower.stemColor;
            ctx.lineWidth = 1;
            ctx.beginPath();
            ctx.moveTo(x + flower.x, y + flower.y);
            ctx.lineTo(x + flower.x, y + flower.y - flower.height); 
            ctx.stroke();

            // Draw flower head
            ctx.fillStyle = flower.headColor;
            if (flower.type === 'circle') {
                ctx.beginPath();
                ctx.arc(x + flower.x, y + flower.y - flower.height, flower.headSize, 0, 2 * Math.PI);
                ctx.fill();
            } else if (flower.type === 'square') {
                ctx.fillRect(x + flower.x - flower.headSize/2, y + flower.y - flower.height - flower.headSize/2, flower.headSize, flower.headSize);
            }
        });
    }
    
    /**
     * Draws a hospital building based on the user's provided reference image,
     * with the color corrected and the small top corners removed.
     * @param {number} x - X position on canvas.
     * @param {number} y - Y position on canvas.
     * @param {number} width - Width of the building.
     * @param {number} height - Height of the building.
     * @param {string} color - Main color of the building.
     */
    function drawHospital(x, y, width, height, color) {
        // Set the main color for the building body
        ctx.fillStyle = color;

        // Draw the main rectangular body of the building
        ctx.fillRect(x + width * 0.1, y + height * 0.2, width * 0.8, height * 0.8);
        
        // Draw the roof platform
        ctx.fillRect(x + width * 0.1, y + height * 0.1, width * 0.8, height * 0.1);

        // Draw the white sign on top
        ctx.fillStyle = '#ffffff';
        const signWidth = width * 0.4;
        const signHeight = height * 0.2;
        const signX = x + (width - signWidth) / 2;
        const signY = y;
        ctx.fillRect(signX, signY, signWidth, signHeight);

        // Draw the red cross on the sign
        ctx.fillStyle = '#a71c1c';
        const crossSize = Math.min(signWidth, signHeight) * 0.7;
        const crossX = signX + (signWidth - crossSize) / 2;
        const crossY = signY + (signHeight - crossSize) / 2;
        // Vertical part of the cross
        ctx.fillRect(crossX + crossSize * 0.4, crossY, crossSize * 0.2, crossSize);
        // Horizontal part of the cross
        ctx.fillRect(crossX, crossY + crossSize * 0.4, crossSize, crossSize * 0.2);

        // Draw the door
        ctx.fillStyle = '#ffffff';
        const doorWidth = width * 0.2;
        const doorHeight = height * 0.2;
        const doorX = x + (width - doorWidth) / 2;
        const doorY = y + height * 0.8;
        ctx.fillRect(doorX, doorY, doorWidth, doorHeight);
        
        // Draw the windows
        const windowWidth = width * 0.2;
        const windowHeight = height * 0.15;
        ctx.fillRect(x + width * 0.15, y + height * 0.25, windowWidth, windowHeight);
        ctx.fillRect(x + width * 0.65, y + height * 0.25, windowWidth, windowHeight);
        ctx.fillRect(x + width * 0.15, y + height * 0.5, windowWidth, windowHeight);
        ctx.fillRect(x + width * 0.65, y + height * 0.5, windowWidth, windowHeight);
    }
    
    /**
     * Draws a wind turbine with a spinning blade animation.
     * @param {number} x - X coordinate of the top-left corner of the building.
     * @param {number} y - Y coordinate of the top-left corner of the building.
     * @param {number} width - Total width of the building.
     * @param {number} height - Total height of the building.
     */
    function drawWindTurbine(x, y, width, height) {
        // Base of the turbine
        const baseX = x + width * 0.4;
        const baseY = y + height * 0.8;
        ctx.fillStyle = '#a0a0a0';
        ctx.beginPath();
        ctx.moveTo(baseX, baseY);
        ctx.lineTo(baseX + width * 0.2, baseY);
        ctx.lineTo(baseX + width * 0.15, y + height * 0.6);
        ctx.lineTo(baseX + width * 0.05, y + height * 0.6);
        ctx.closePath();
        ctx.fill();

        // Tower
        const towerX = x + width * 0.45;
        const towerY = y + height * 0.6;
        const towerWidth = width * 0.1;
        const towerHeight = height * 0.4;
        ctx.fillStyle = '#94a3b8';
        ctx.fillRect(towerX, towerY, towerWidth, towerHeight);

        // Hub for the blades
        const hubX = towerX + towerWidth / 2;
        const hubY = towerY - height * 0.05;

        // --- Animated Blades ---
        ctx.save();
        ctx.translate(hubX, hubY);
        const rotationAngle = (Date.now() / 500) % (2 * Math.PI);
        ctx.rotate(rotationAngle);
        const bladeWidth = width * 0.07;
        const bladeLength = width * 0.4;
        ctx.fillStyle = '#6b7280';
        ctx.fillRect(-bladeLength, -bladeWidth / 2, bladeLength * 2, bladeWidth);
        ctx.fillRect(-bladeWidth / 2, -bladeLength, bladeWidth, bladeLength * 2);
        ctx.restore();

        // Redraw hub on top of blades
        ctx.fillStyle = '#4b5563';
        ctx.beginPath();
        ctx.arc(hubX, hubY, width * 0.05, 0, Math.PI * 2);
        ctx.fill();
    }
    
    /**
     * Draws the road with dynamic visual cues based on its neighbors.
     * @param {number} x - World X position of the road.
     * @param {number} y - World Y position of the road.
     * @param {number} width - Width of the road block.
     * @param {number} height - Height of the road block.
     * @param {string} color - Color of the road.
     */
    function drawRoad(x, y, width, height, color) {
        const drawX = x - mapOffset.x;
        const drawY = y - mapOffset.y;
        const tileX = Math.floor(x / gridSize);
        const tileY = Math.floor(y / gridSize);
        const stripeColor = '#f8fafc';

        // Draw the base road block
        ctx.fillStyle = color;
        ctx.fillRect(drawX, drawY, width, height);

        // Check for neighboring road blocks
        const hasTopNeighbor = findBuilding(tileX, tileY - 1)?.type === 'road';
        const hasBottomNeighbor = findBuilding(tileX, tileY + 1)?.type === 'road';
        const hasLeftNeighbor = findBuilding(tileX - 1, tileY)?.type === 'road';
        const hasRightNeighbor = findBuilding(tileX + 1, tileY)?.type === 'road';
        
        // Count neighbors to determine if it's an intersection
        const neighborCount = [hasTopNeighbor, hasBottomNeighbor, hasLeftNeighbor, hasRightNeighbor].filter(Boolean).length;

        // Don't draw lines on intersections (3 or 4 connections)
        if (neighborCount >= 3) {
            return;
        }

        // Helper to draw a dashed line
        function drawDashedLine(x1, y1, x2, y2) {
            ctx.beginPath();
            ctx.setLineDash([5, 5]); // Set dash pattern
            ctx.moveTo(x1, y1);
            ctx.lineTo(x2, y2);
            ctx.strokeStyle = stripeColor;
            ctx.lineWidth = 2;
            ctx.stroke();
            ctx.setLineDash([]); // Reset dash pattern
        }

        // --- Logic for drawing based on connections (excluding intersections) ---
        if (hasTopNeighbor && hasBottomNeighbor) {
            // Vertical road
            drawDashedLine(drawX + width / 2, drawY, drawX + width / 2, drawY + height);
        } else if (hasLeftNeighbor && hasRightNeighbor) {
            // Horizontal road
            drawDashedLine(drawX, drawY + height / 2, drawX + width, drawY + height / 2);
        } else {
            // Corner or dead-end
            const centerX = drawX + width / 2;
            const centerY = drawY + height / 2;
            const lineLength = width / 2;

            if (hasTopNeighbor && hasRightNeighbor) {
                // Top-right corner
                drawDashedLine(centerX, drawY, centerX, centerY);
                drawDashedLine(centerX, centerY, drawX + width, centerY);
            } else if (hasRightNeighbor && hasBottomNeighbor) {
                // Right-bottom corner
                drawDashedLine(centerX, centerY, drawX + width, centerY);
                drawDashedLine(centerX, centerY, centerX, drawY + height);
            } else if (hasBottomNeighbor && hasLeftNeighbor) {
                // Bottom-left corner
                drawDashedLine(centerX, centerY, centerX, drawY + height);
                drawDashedLine(drawX, centerY, centerX, centerY);
            } else if (hasLeftNeighbor && hasTopNeighbor) {
                // Left-top corner
                drawDashedLine(drawX, centerY, centerX, centerY);
                drawDashedLine(centerX, drawY, centerX, centerY);
            } else if (hasTopNeighbor) {
                // Dead end top
                drawDashedLine(centerX, centerY, centerX, drawY);
            } else if (hasBottomNeighbor) {
                // Dead end bottom
                drawDashedLine(centerX, centerY, centerX, drawY + height);
            } else if (hasLeftNeighbor) {
                // Dead end left
                drawDashedLine(drawX, centerY, centerX, centerY);
            } else if (hasRightNeighbor) {
                // Dead end right
                drawDashedLine(centerX, centerY, drawX + width, centerY);
            }
        }
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
            if (!isPopupMenuOpen) togglePopupMenu();
        } else {
            if (isPopupMenuOpen) togglePopupMenu();
        }
    }
    
    function updateButtonStyles() {
        moveModeButton.classList.remove('mode-active');
        destroyModeButton.classList.remove('mode-active');
        
        // Remove active class from all building buttons
        for (const btn in buildingButtons) {
            const buttonEl = buildingButtons[btn];
            if (buttonEl) {
                buttonEl.classList.remove('mode-active');
            }
        }
        
        // Re-apply active class based on current mode and type
        if (activeMode === 'move') {
            moveModeButton.classList.add('mode-active');
        } else if (activeMode === 'destroy') {
            destroyModeButton.classList.add('mode-active');
        } else if (activeMode === 'build' && buildingType) {
            // Note: We don't activate the main 'Bangun' button, just the specific building button
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
        player = { x: 0, y: 0, width: 28, height: 28, speed: 1.0, color: '#ef4444' };
        activeMode = 'move';
        buildingType = null;
        taxRate = 5;
        taxRateDisplay.textContent = taxRate;
        taxRateSlider.value = taxRate;
        totalPowerOutput = 0;
        totalPowerUsage = 0;
        updateButtonStyles();
        infoModal.classList.remove('modal-show');
    }

    function init() {
        // Event listener for keyboard
        window.addEventListener('keydown', (e) => {
            const key = e.key.toLowerCase();
            keys[key] = true;
            
            // Check for mode change shortcuts
            if (key === 'm') {
                setMode('move', null);
            } else if (key === 'x') {
                setMode('destroy', null);
            }
            
            // Check for build shortcuts
            if (key === 'h') setMode('build', 'house');
            else if (key === 'p') setMode('build', 'park');
            else if (key === 't') setMode('build', 'store');
            else if (key === 'i') setMode('build', 'industrial');
            else if (key === 'j') setMode('build', 'road');
            else if (key === 'o') setMode('build', 'hospital');
            else if (key === 'l') setMode('build', 'windTurbine');

            // Check for other shortcuts
            else if (key === 'g') {
                guideModal.classList.add('modal-show');
            } else if (key === 'r') {
                restartGame();
            }
        });

        window.addEventListener('keyup', (e) => {
            keys[e.key.toLowerCase()] = false;
        });

        // Event listener for touch controls on buttons
        const allMobileButtons = [
            landscapeControls.up, landscapeControls.down, landscapeControls.left, landscapeControls.right,
            portraitControls.up, portraitControls.down, portraitControls.left, portraitControls.right
        ];

        allMobileButtons.forEach(btn => {
            if (btn) {
                btn.addEventListener('pointerdown', (e) => {
                    e.preventDefault();
                    if (e.target.id.includes('up')) touchControls.up = true;
                    if (e.target.id.includes('down')) touchControls.down = true;
                    if (e.target.id.includes('left')) touchControls.left = true;
                    if (e.target.id.includes('right')) touchControls.right = true;
                    document.body.addEventListener('pointerup', resetTouchControls);
                });
            }
        });
        
        function resetTouchControls() {
            touchControls.up = false;
            touchControls.down = false;
            touchControls.left = false;
            touchControls.right = false;
            document.body.removeEventListener('pointerup', resetTouchControls);
        }

        canvas.addEventListener('click', (e) => {
            const rect = canvas.getBoundingClientRect();
            const scaleX = canvas.width / rect.width;
            const scaleY = canvas.height / rect.height;
            const mouseX = (e.clientX - rect.left) * scaleX;
            const mouseY = (e.clientY - rect.top) * scaleY;
            const tileX = Math.floor((mouseX + mapOffset.x) / gridSize);
            const tileY = Math.floor((mouseY + mapOffset.y) / gridSize);
            
            // New logic: click building for info when in move mode
            if (activeMode === 'move') {
                const clickedBuilding = findBuilding(tileX, tileY);
                if (clickedBuilding) {
                    showInfoModal(clickedBuilding);
                }
            } else if (activeMode === 'build' || activeMode === 'destroy') {
                
                // Build and destroy logic
                if (activeMode === 'build') {
                    const existingBuilding = findBuilding(tileX, tileY);
                    const stats = buildingStats[buildingType];
                    const cost = stats.cost;
                    if (!existingBuilding && money >= cost) {
                        const newBuilding = {
                            id: Date.now(), 
                            x: tileX * gridSize, 
                            y: tileY * gridSize, 
                            type: buildingType, 
                            color: stats.color,
                            population: stats.populationCapacity || 0,
                            currentPatients: buildingType === 'hospital' ? 0 : null,
                            needs: { happiness: 0 },
                            workersAssigned: 0,
                            isPowered: false
                        };
                        
                        // For parks, pre-calculate the static flower positions
                        if (buildingType === 'park') {
                            newBuilding.flowers = [];
                            const flowerTypes = [
                                { stemColor: '#8B4513', headColor: '#FF6347', height: 10, headSize: 4, type: 'circle' },
                                { stemColor: '#228B22', headColor: '#DA70D6', height: 14, headSize: 5, type: 'square' },
                                { stemColor: '#6B8E23', headColor: '#ADD8E6', height: 12, headSize: 6, type: 'circle' },
                                { stemColor: '#556B2F', headColor: '#FFA07A', height: 16, headSize: 3, type: 'square' }
                            ];
                            const numFlowers = Math.floor(gridSize / 10);
                            for (let i = 0; i < numFlowers; i++) {
                                const flower = flowerTypes[Math.floor(Math.random() * flowerTypes.length)];
                                newBuilding.flowers.push({
                                    x: Math.random() * (gridSize - 10) + 5,
                                    y: gridSize,
                                    stemColor: flower.stemColor,
                                    headColor: flower.headColor,
                                    height: flower.height,
                                    headSize: flower.headSize,
                                    type: flower.type
                                });
                            }
                        }
                        
                        buildings.push(newBuilding);
                        money -= cost;
                        calculateNeeds();
                    } else if (existingBuilding) {
                        showMessage('Lokasi ini sudah terisi!');
                    } else if (money < cost) {
                        showMessage('Uang tidak cukup!');
                    }
                } else if (activeMode === 'destroy') {
                    const buildingIndex = buildings.findIndex(b =>
                        Math.floor(b.x / gridSize) === tileX && Math.floor(b.y / gridSize) === tileY
                    );
                    if (buildingIndex !== -1) {
                        const destroyedBuilding = buildings.splice(buildingIndex, 1)[0];
                        const refund = buildingStats[destroyedBuilding.type].cost * 0.5;
                        money += refund;
                        showMessage(`Bangunan dihancurkan! Uang dikembalikan: ${formatRupiah(refund)}.`);
                        calculateNeeds();
                    }
                }
            }
        });

        // Event listener for tax rate slider
        taxRateSlider.addEventListener('input', (e) => {
            taxRate = parseInt(e.target.value);
            taxRateDisplay.textContent = taxRate;
            calculateNeeds();
        });

        // Event listeners for UI buttons
        buildMenuButton.addEventListener('click', togglePopupMenu);
        moveModeButton.addEventListener('click', () => {
            setMode('move', null);
        });
        destroyModeButton.addEventListener('click', () => {
            setMode('destroy', null);
        });
        restartButton.addEventListener('click', restartGame);

        // Show and hide guide modal
        guideButton.addEventListener('click', () => { guideModal.classList.add('modal-show'); });
        guideModalCloseButton.addEventListener('click', () => { guideModal.classList.remove('modal-show'); });
        
        // Hide info modal
        infoModalCloseButton.addEventListener('click', () => { infoModal.classList.remove('modal-show'); });
        
        window.addEventListener('click', (event) => {
            if (event.target === guideModal) {
                guideModal.classList.remove('modal-show');
            }
            if (event.target === infoModal) {
                infoModal.classList.remove('modal-show');
            }
            if (isPopupMenuOpen && !popupMenu.contains(event.target) && !buildMenuButton.contains(event.target)) {
                togglePopupMenu();
            }
        });
        
        for (const type in buildingButtons) {
            buildingButtons[type].addEventListener('click', () => {
                setMode('build', type);
            });
        }

        const resizeCanvas = () => {
            canvas.width = Math.min(800, window.innerWidth - 40);
            canvas.height = canvas.width * 0.75;
            gameLoop();
        };
        window.addEventListener('resize', resizeCanvas);
        resizeCanvas();

        restartGame();

        // Interval for population and patient updates
        setInterval(() => {
            buildings.filter(b => b.type === 'house').forEach(house => {
                let changeAmount = 0;
                if (house.needs.happiness >= 80) {
                    changeAmount = Math.floor(Math.random() * 2) + 1;
                } else if (house.needs.happiness < 50) {
                    changeAmount = -(Math.floor(Math.random() * 2) + 1);
                }
                
                let newPopulation = house.population + changeAmount;
                if (taxRate > 40 && newPopulation < 0) newPopulation = 0;
                else if (newPopulation < 1) newPopulation = 1;
                house.population = Math.min(buildingStats.house.populationCapacity, newPopulation);
            });
            let totalPopulation = 0;
            buildings.forEach(b => {
                if (b.type === 'house') totalPopulation += b.population;
            });
            population = totalPopulation;

            buildings.filter(b => b.type === 'hospital').forEach(hospital => {
                const stats = buildingStats.hospital;
                const randomPatients = Math.floor(Math.random() * (stats.patientCapacity + 1));
                hospital.currentPatients = Math.min(randomPatients, population);
            });
        }, 5000);
        setInterval(calculateNeeds, 2000);

        gameLoop();
    }

    // Use DOMContentLoaded instead of window.onload for better compatibility
    document.addEventListener('DOMContentLoaded', init);
</script>

</body>
</html>
