<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Jogo de Formar Palavras</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Nunito:wght@400;700;900&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Nunito', sans-serif;
        }
        .dragging {
            opacity: 0.5;
            transform: scale(1.1);
            box-shadow: 0 10px 20px rgba(0,0,0,0.2);
            cursor: grabbing;
        }
        .drag-over {
            transform: scale(1.05);
            background-color: #f0fdf4; /* green-50 */
        }
        #syllable-container.drag-over {
            background-color: #e5e7eb; /* gray-200 */
        }
        @keyframes tada {
            0% {transform: scale(1);}
            10%, 20% {transform: scale(0.9) rotate(-3deg);}
            30%, 50%, 70%, 90% {transform: scale(1.1) rotate(3deg);}
            40%, 60%, 80% {transform: scale(1.1) rotate(-3deg);}
            100% {transform: scale(1) rotate(0);}
        }
        .tada-animation {
            animation: tada 1s ease;
        }
        @keyframes shake {
            0%, 100% { transform: translateX(0); }
            10%, 30%, 50%, 70%, 90% { transform: translateX(-5px); }
            20%, 40%, 60%, 80% { transform: translateX(5px); }
        }
        .shake-animation {
            animation: shake 0.5s ease-in-out;
        }
    </style>
</head>
<body class="bg-blue-50 min-h-screen flex items-center justify-center p-4">

    <div class="bg-white rounded-2xl shadow-lg p-6 md:p-8 w-full max-w-2xl text-center">
        <h1 class="text-3xl md:text-4xl font-black text-blue-800 mb-2">Joguinho das Sílabas</h1>
        <p class="text-gray-500 mb-6">Arraste a sílaba colorida para o quadrado da mesma cor para formar a palavra!</p>

        <!-- Container do Emoji de Sucesso -->
        <div id="success-container" class="hidden mb-6 flex flex-col items-center">
            <span id="word-emoji" class="text-8xl md:text-9xl mb-4" style="display: inline-block;"></span>
            <h2 id="word-display" class="text-4xl font-bold text-green-600 tracking-widest uppercase"></h2>
        </div>

        <!-- Container para os quadrados de destino -->
        <div id="word-container" class="flex justify-center items-center gap-2 md:gap-4 mb-8 h-24">
            <!-- Gerado via JS -->
        </div>

        <!-- Container para as sílabas arrastáveis -->
        <div id="syllable-container" class="flex justify-center items-center flex-wrap gap-3 md:gap-4 p-4 bg-gray-100 rounded-lg min-h-[80px]">
            <!-- Gerado via JS -->
        </div>

        <button id="next-word-btn" class="mt-8 bg-blue-600 text-white font-bold py-3 px-8 rounded-full hover:bg-blue-700 transition-all transform hover:scale-105 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-opacity-50 hidden">
            Próxima Palavra
        </button>
    </div>

    <script>
        // --- CONFIGURAÇÃO DO JOGO ---
        const words = [
            { word: "BOLA", syllables: [{ text: "BO", color: "blue" }, { text: "LA", color: "red" }], emoji: "⚽️" },
            { word: "CASA", syllables: [{ text: "CA", color: "green" }, { text: "SA", color: "yellow" }], emoji: "🏠" },
            { word: "GATO", syllables: [{ text: "GA", color: "purple" }, { text: "TO", color: "orange" }], emoji: "🐈" },
            { word: "PATO", syllables: [{ text: "PA", color: "cyan" }, { text: "TO", color: "pink" }], emoji: "🦆" },
            { word: "RATO", syllables: [{ text: "RA", color: "blue" }, { text: "TO", color: "green" }], emoji: "🐀" },
            { word: "FACA", syllables: [{ text: "FA", color: "red" }, { text: "CA", color: "yellow" }], emoji: "🔪" },
            { word: "DADO", syllables: [{ text: "DA", color: "purple" }, { text: "DO", color: "orange" }], emoji: "🎲" },
            { word: "VACA", syllables: [{ text: "VA", color: "cyan" }, { text: "CA", color: "pink" }], emoji: "🐄" },
            { word: "LIVRO", syllables: [{ text: "LI", color: "blue" }, { text: "VRO", color: "red" }], emoji: "📖" },
            { word: "PEIXE", syllables: [{ text: "PEI", color: "green" }, { text: "XE", color: "yellow" }], emoji: "🐟" },
            { word: "SOFÁ", syllables: [{ text: "SO", color: "purple" }, { text: "FÁ", color: "orange" }], emoji: "🛋️" },
            { word: "MOTO", syllables: [{ text: "MO", color: "cyan" }, { text: "TO", color: "pink" }], emoji: "🏍️" },
            { word: "SAPATO", syllables: [{ text: "SA", color: "blue" }, { text: "PA", color: "red" }, { text: "TO", color: "green" }], emoji: "👞" },
            { word: "JANELA", syllables: [{ text: "JA", color: "yellow" }, { text: "NE", color: "purple" }, { text: "LA", color: "orange" }], emoji: "🖼️" },
            { word: "PETECA", syllables: [{ text: "PE", color: "cyan" }, { text: "TE", color: "pink" }, { text: "CA", color: "blue" }], emoji: "🏸" },
            { word: "MACACO", syllables: [{ text: "MA", color: "red" }, { text: "CA", color: "green" }, { text: "CO", color: "yellow" }], emoji: "🐒" },
            { word: "CAVALO", syllables: [{ text: "CA", color: "purple" }, { text: "VA", color: "orange" }, { text: "LO", color: "cyan" }], emoji: "🐎" },
            { word: "CORUJA", syllables: [{ text: "CO", color: "pink" }, { text: "RU", color: "blue" }, { text: "JA", color: "red" }], emoji: "🦉" },
            { word: "GAVETA", syllables: [{ text: "GA", color: "green" }, { text: "VE", color: "yellow" }, { text: "TA", color: "purple" }], emoji: "🗄️" },
            { word: "GIRAFA", syllables: [{ text: "GI", color: "orange" }, { text: "RA", color: "cyan" }, { text: "FA", color: "pink" }], emoji: "🦒" },
            { word: "TOMATE", syllables: [{ text: "TO", color: "red" }, { text: "MA", color: "blue" }, { text: "TE", color: "green" }], emoji: "🍅" },
            { word: "PANELA", syllables: [{ text: "PA", color: "yellow" }, { text: "NE", color: "purple" }, { text: "LA", color: "orange" }], emoji: "🍳" },
            { word: "SALADA", syllables: [{ text: "SA", color: "cyan" }, { text: "LA", color: "pink" }, { text: "DA", color: "blue" }], emoji: "🥗" },
            { word: "PIPOCA", syllables: [{ text: "PI", color: "red" }, { text: "PO", color: "green" }, { text: "CA", color: "yellow" }], emoji: "🍿" },
            { word: "NAVIO", syllables: [{ text: "NA", color: "purple" }, { text: "VI", color: "orange" }, { text: "O", color: "cyan" }], emoji: "🚢" },
            { word: "AVIÃO", syllables: [{ text: "A", color: "pink" }, { text: "VI", color: "blue" }, { text: "ÃO", color: "red" }], emoji: "✈️" },
            { word: "FOGUETE", syllables: [{ text: "FO", color: "green" }, { text: "GUE", color: "yellow" }, { text: "TE", color: "purple" }], emoji: "🚀" },
            { word: "TATU", syllables: [{ text: "TA", color: "orange" }, { text: "TU", color: "cyan" }], emoji: "🐾" },
            { word: "ZEBRA", syllables: [{ text: "ZE", color: "pink" }, { text: "BRA", color: "blue" }], emoji: "🦓" },
            { word: "XICARA", syllables: [{ text: "XI", color: "red" }, { text: "CA", color: "green" }, { text: "RA", color: "yellow" }], emoji: "☕" },
            { word: "BICICLETA", syllables: [{ text: "BI", color: "blue" }, { text: "CI", color: "red" }, { text: "CLE", color: "green" }, { text: "TA", color: "yellow" }], emoji: "🚲" },
            { word: "ABACAXI", syllables: [{ text: "A", color: "purple" }, { text: "BA", color: "orange" }, { text: "CA", color: "pink" }, { text: "XI", color: "cyan" }], emoji: "🍍" },
            { word: "MORANGO", syllables: [{ text: "MO", color: "blue" }, { text: "RAN", color: "red" }, { text: "GO", color: "green" }], emoji: "🍓" },
            { word: "BANANA", syllables: [{ text: "BA", color: "yellow" }, { text: "NA", color: "purple" }, { text: "NA", color: "orange" }], emoji: "🍌" },
            { word: "MELANCIA", syllables: [{ text: "ME", color: "pink" }, { text: "LAN", color: "cyan" }, { text: "CI", color: "blue" }, { text: "A", color: "red" }], emoji: "🍉" },
            { word: "COMPUTADOR", syllables: [{ text: "COM", color: "green" }, { text: "PU", color: "yellow" }, { text: "TA", color: "purple" }, { text: "DOR", color: "orange" }], emoji: "💻" },
            { word: "TELEFONE", syllables: [{ text: "TE", color: "pink" }, { text: "LE", color: "cyan" }, { text: "FO", color: "blue" }, { text: "NE", color: "red" }], emoji: "📱" },
            { word: "ESCOLA", syllables: [{ text: "ES", color: "green" }, { text: "CO", color: "yellow" }, { text: "LA", color: "purple" }], emoji: "🏫" },
            { word: "PROFESSOR", syllables: [{ text: "PRO", color: "orange" }, { text: "FES", color: "pink" }, { text: "SOR", color: "cyan" }], emoji: "👩‍🏫" },
            { word: "ALUNO", syllables: [{ text: "A", color: "blue" }, { text: "LU", color: "red" }, { text: "NO", color: "green" }], emoji: "🧑‍🎓" },
            { word: "CADERNO", syllables: [{ text: "CA", color: "yellow" }, { text: "DER", color: "purple" }, { text: "NO", color: "orange" }], emoji: "📓" },
            { word: "CANETA", syllables: [{ text: "CA", color: "pink" }, { text: "NE", color: "cyan" }, { text: "TA", color: "blue" }], emoji: "🖊️" },
            { word: "BORRACHA", syllables: [{ text: "BOR", color: "red" }, { text: "RA", color: "green" }, { text: "CHA", color: "yellow" }], emoji: "📝" },
            { word: "MOCHILA", syllables: [{ text: "MO", color: "purple" }, { text: "CHI", color: "orange" }, { text: "LA", color: "pink" }], emoji: "🎒" },
            { word: "FUTEBOL", syllables: [{ text: "FU", color: "cyan" }, { text: "TE", color: "blue" }, { text: "BOL", color: "red" }], emoji: "🥅" },
            { word: "BONECA", syllables: [{ text: "BO", color: "green" }, { text: "NE", color: "yellow" }, { text: "CA", color: "purple" }], emoji: "👧" },
            { word: "CARRINHO", syllables: [{ text: "CAR", color: "orange" }, { text: "RI", color: "pink" }, { text: "NHO", color: "cyan" }], emoji: "🚗" },
            { word: "FLORESTA", syllables: [{ text: "FLO", color: "blue" }, { text: "RES", color: "red" }, { text: "TA", color: "green" }], emoji: "🌳" },
            { word: "JARDIM", syllables: [{ text: "JAR", color: "yellow" }, { text: "DIM", color: "purple" }], emoji: "🌷" },
            { word: "PRAIA", syllables: [{ text: "PRAI", color: "orange" }, { text: "A", color: "pink" }], emoji: "🏖️" },
            { word: "SOL", syllables: [{ text: "SOL", color: "yellow" }], emoji: "☀️" },
            { word: "LUA", syllables: [{ text: "LU", color: "cyan" }, { text: "A", color: "blue" }], emoji: "🌙" },
            { word: "ESTRELA", syllables: [{ text: "ES", color: "red" }, { text: "TRE", color: "green" }, { text: "LA", color: "yellow" }], emoji: "⭐" },
            { word: "NUVEM", syllables: [{ text: "NU", color: "purple" }, { text: "VEM", color: "orange" }], emoji: "☁️" },
            { word: "CHUVA", syllables: [{ text: "CHU", color: "pink" }, { text: "VA", color: "cyan" }], emoji: "🌧️" },
            { word: "ARCOÍRIS", syllables: [{ text: "AR", color: "red" }, { text: "CO", color: "orange" }, { text: "Í", color: "yellow" }, { text: "RIS", color: "green" }], emoji: "🌈" },
            { word: "TIGRE", syllables: [{ text: "TI", color: "blue" }, { text: "GRE", color: "red" }], emoji: "🐅" },
            { word: "LEÃO", syllables: [{ text: "LE", color: "green" }, { text: "ÃO", color: "yellow" }], emoji: "🦁" },
            { word: "ELEFANTE", syllables: [{ text: "E", color: "purple" }, { text: "LE", color: "orange" }, { text: "FAN", color: "pink" }, { text: "TE", color: "cyan" }], emoji: "🐘" },
            { word: "MÃE", syllables: [{ text: "MÃE", color: "red" }], emoji: "👩" },
            { word: "PAI", syllables: [{ text: "PAI", color: "blue" }], emoji: "👨" },
            { word: "IRMÃO", syllables: [{ text: "IR", color: "green" }, { text: "MÃO", color: "yellow" }], emoji: "👦" },
            { word: "IRMÃ", syllables: [{ text: "IR", color: "purple" }, { text: "MÃ", color: "orange" }], emoji: "👧" },
            { word: "FAMÍLIA", syllables: [{ text: "FA", color: "pink" }, { text: "MÍ", color: "cyan" }, { text: "LI", color: "blue" }, { text: "A", color: "red" }], emoji: "👨‍👩‍👧‍👦" },
            { word: "AMIGO", syllables: [{ text: "A", color: "green" }, { text: "MI", color: "yellow" }, { text: "GO", color: "purple" }], emoji: "🧑‍🤝‍🧑" },
            { word: "FESTA", syllables: [{ text: "FES", color: "orange" }, { text: "TA", color: "pink" }], emoji: "🎉" },
            { word: "BOLO", syllables: [{ text: "BO", color: "cyan" }, { text: "LO", color: "blue" }], emoji: "🎂" },
            { word: "PRESENTE", syllables: [{ text: "PRE", color: "red" }, { text: "SEN", color: "green" }, { text: "TE", color: "yellow" }], emoji: "🎁" },
            { word: "BALÃO", syllables: [{ text: "BA", color: "purple" }, { text: "LÃO", color: "orange" }], emoji: "🎈" },
            { word: "MÚSICA", syllables: [{ text: "MÚ", color: "pink" }, { text: "SI", color: "cyan" }, { text: "CA", color: "blue" }], emoji: "🎵" },
            { word: "DANÇA", syllables: [{ text: "DAN", color: "red" }, { text: "ÇA", color: "green" }], emoji: "💃" },
            { word: "PIZZA", syllables: [{ text: "PIZ", color: "yellow" }, { text: "ZA", color: "purple" }], emoji: "🍕" },
            { word: "SORVETE", syllables: [{ text: "SOR", color: "orange" }, { text: "VE", color: "pink" }, { text: "TE", color: "cyan" }], emoji: "🍦" },
            { word: "CHOCOLATE", syllables: [{ text: "CHO", color: "blue" }, { text: "CO", color: "red" }, { text: "LA", color: "green" }, { text: "TE", color: "yellow" }], emoji: "🍫" },
            { word: "DINOSSAURO", syllables: [{ text: "DI", color: "purple" }, { text: "NOS", color: "orange" }, { text: "SAU", color: "pink" }, { text: "RO", color: "cyan" }], emoji: "🦖" },
            { word: "DRAGÃO", syllables: [{ text: "DRA", color: "blue" }, { text: "GÃO", color: "red" }], emoji: "🐉" },
            { word: "PRINCESA", syllables: [{ text: "PRIN", color: "green" }, { text: "CE", color: "yellow" }, { text: "SA", color: "purple" }], emoji: "👸" },
            { word: "PRÍNCIPE", syllables: [{ text: "PRÍN", color: "orange" }, { text: "CI", color: "pink" }, { text: "PE", color: "cyan" }], emoji: "🤴" },
            { word: "CASTELO", syllables: [{ text: "CAS", color: "blue" }, { text: "TE", color: "red" }, { text: "LO", color: "green" }], emoji: "🏰" },
            { word: "FADA", syllables: [{ text: "FA", color: "yellow" }, { text: "DA", color: "purple" }], emoji: "🧚" },
            { word: "BRUXA", syllables: [{ text: "BRU", color: "orange" }, { text: "XA", color: "pink" }], emoji: "🧙‍♀️" },
            { word: "PIRATA", syllables: [{ text: "PI", color: "cyan" }, { text: "RA", color: "blue" }, { text: "TA", color: "red" }], emoji: "🏴‍☠️" },
            { word: "TESOURO", syllables: [{ text: "TE", color: "green" }, { text: "SOU", color: "yellow" }, { text: "RO", color: "purple" }], emoji: "💎" },
            { word: "BARCO", syllables: [{ text: "BAR", color: "orange" }, { text: "CO", color: "pink" }], emoji: "⛵" },
            { word: "ILHA", syllables: [{ text: "I", color: "cyan" }, { text: "LHA", color: "blue" }], emoji: "🏝️" },
            { word: "FOGO", syllables: [{ text: "FO", color: "red" }, { text: "GO", color: "orange" }], emoji: "🔥" },
            { word: "ÁGUA", syllables: [{ text: "Á", color: "blue" }, { text: "GUA", color: "cyan" }], emoji: "💧" },
            { word: "TERRA", syllables: [{ text: "TER", color: "green" }, { text: "RA", color: "yellow" }], emoji: "🌎" },
            { word: "VERMELHO", syllables: [{ text: "VER", color: "red" }, { text: "ME", color: "pink" }, { text: "LHO", color: "orange" }], emoji: "🔴" },
            { word: "AMARELO", syllables: [{ text: "A", color: "yellow" }, { text: "MA", color: "orange" }, { text: "RE", color: "purple" }, { text: "LO", color: "pink" }], emoji: "🟡" },
            { word: "BRINQUEDO", syllables: [{ text: "BRIN", color: "cyan" }, { text: "QUE", color: "blue" }, { text: "DO", color: "red" }], emoji: "🧸" },
            { word: "MÉDICO", syllables: [{ text: "MÉ", color: "green" }, { text: "DI", color: "yellow" }, { text: "CO", color: "purple" }], emoji: "👨‍⚕️" },
            { word: "BOMBEIRO", syllables: [{ text: "BOM", color: "red" }, { text: "BEI", color: "orange" }, { text: "RO", color: "cyan" }], emoji: "👨‍🚒" },
            { word: "POLICIAL", syllables: [{ text: "PO", color: "blue" }, { text: "LI", color: "pink" }, { text: "CI", color: "green" }, { text: "AL", color: "yellow" }], emoji: "👮" },
            { word: "COZINHEIRO", syllables: [{ text: "CO", color: "purple" }, { text: "ZI", color: "orange" }, { text: "NHEI", color: "cyan" }, { text: "RO", color: "red" }], emoji: "👨‍🍳" },
            { word: "ABELHA", syllables: [{ text: "A", color: "yellow" }, { text: "BE", color: "pink" }, { text: "LHA", color: "blue" }], emoji: "🐝" },
            { word: "BORBOLETA", syllables: [{ text: "BOR", color: "green" }, { text: "BO", color: "purple" }, { text: "LE", color: "orange" }, { text: "TA", color: "cyan" }], emoji: "🦋" },
            { word: "CACHORRO", syllables: [{ text: "CA", color: "red" }, { text: "CHO", color: "yellow" }, { text: "RRO", color: "blue" }], emoji: "🐶" },
            { word: "PASSARINHO", syllables: [{ text: "PAS", color: "pink" }, { text: "SA", color: "green" }, { text: "RI", color: "purple" }, { text: "NHO", color: "orange" }], emoji: "🐦" },
            { word: "ESQUILO", syllables: [{ text: "ES", color: "cyan" }, { text: "QUI", color: "red" }, { text: "LO", color: "yellow" }], emoji: "🐿️" }
        ];

        const DISTRACTOR_SYLLABLES = ["PE", "TI", "CO", "FA", "MA", "LE", "RI", "VE", "NA", "LU", "BI", "SO", "JU", "BA", "DE", "PO", "GA", "TO", "CA", "DO", "FI", "GO", "JA", "LA", "MI", "NO", "PA", "QUE", "RA", "SA", "TE", "VA", "XI", "ZE"];

        const colorMap = {
            blue:   { bg: 'bg-blue-500',   border: 'border-blue-500' },
            red:    { bg: 'bg-red-500',    border: 'border-red-500' },
            green:  { bg: 'bg-green-500',  border: 'border-green-500' },
            yellow: { bg: 'bg-yellow-500', border: 'border-yellow-500' },
            purple: { bg: 'bg-purple-500', border: 'border-purple-500' },
            orange: { bg: 'bg-orange-500', border: 'border-orange-500' },
            pink:   { bg: 'bg-pink-500',   border: 'border-pink-500' },
            cyan:   { bg: 'bg-cyan-500',   border: 'border-cyan-500' },
        };

        // --- ELEMENTOS DO DOM ---
        const wordContainer = document.getElementById('word-container');
        const syllableContainer = document.getElementById('syllable-container');
        const successContainer = document.getElementById('success-container');
        const wordEmoji = document.getElementById('word-emoji');
        const wordDisplay = document.getElementById('word-display');
        const nextWordBtn = document.getElementById('next-word-btn');

        // --- ESTADO DO JOGO ---
        let shuffledWordIndices = [];
        let gameRound = 0;
        let draggedSyllable = null;

        function shuffleArray(array) {
            for (let i = array.length - 1; i > 0; i--) {
                const j = Math.floor(Math.random() * (i + 1));
                [array[i], array[j]] = [array[j], array[i]];
            }
            return array;
        }

        function startGame() {
            gameRound = 0;
            shuffledWordIndices = Array.from(Array(words.length).keys());
            shuffleArray(shuffledWordIndices);
            setupLevel();
        }

        function setupLevel() {
            const currentWordIndex = shuffledWordIndices[gameRound];
            const wordData = words[currentWordIndex];
            
            wordContainer.innerHTML = '';
            syllableContainer.innerHTML = '';
            successContainer.classList.add('hidden');
            nextWordBtn.classList.add('hidden');
            wordEmoji.textContent = '';

            wordData.syllables.forEach((syllable, i) => {
                const dropZone = document.createElement('div');
                dropZone.classList.add('w-16', 'h-16', 'sm:w-20', 'sm:h-20', 'md:w-24', 'md:h-24', 'border-4', 'rounded-lg', 'flex', 'items-center', 'justify-center', 'transition-all', 'duration-200', colorMap[syllable.color].border);
                dropZone.dataset.color = syllable.color;
                dropZone.dataset.index = i;
                wordContainer.appendChild(dropZone);
                addDropZoneEvents(dropZone);
            });
            
            const correctSyllables = [...wordData.syllables];
            const correctSyllableTexts = correctSyllables.map(s => s.text);
            const correctSyllableColors = correctSyllables.map(s => s.color);
            
            const availableDistractors = DISTRACTOR_SYLLABLES.filter(s => !correctSyllableTexts.includes(s));
            shuffleArray(availableDistractors);
            const chosenDistractorsText = availableDistractors.slice(0, 3);
            
            const allColors = Object.keys(colorMap);
            const availableDistractorColors = allColors.filter(c => !correctSyllableColors.includes(c));
            
            const distractorSyllableObjects = chosenDistractorsText.map(text => {
                shuffleArray(availableDistractorColors);
                return {
                    text: text,
                    color: availableDistractorColors[0] 
                };
            });
            
            const allSyllablesForLevel = [...correctSyllables, ...distractorSyllableObjects];
            const shuffledSyllables = shuffleArray(allSyllablesForLevel);

            shuffledSyllables.forEach(syllable => {
                const syllableEl = document.createElement('div');
                syllableEl.textContent = syllable.text;
                syllableEl.draggable = true;
                syllableEl.dataset.color = syllable.color;
                syllableEl.classList.add('w-16', 'h-16', 'sm:w-20', 'sm:h-20', 'md:w-24', 'md:h-24', 'text-white', 'text-3xl', 'sm:text-4xl', 'font-black', 'rounded-lg', 'flex', 'items-center', 'justify-center', 'cursor-grab', 'transition-all', 'duration-200', colorMap[syllable.color].bg);
                syllableContainer.appendChild(syllableEl);
                addSyllableEvents(syllableEl);
            });
        }

        function addSyllableEvents(syllableEl) {
            syllableEl.addEventListener('dragstart', e => {
                draggedSyllable = e.target;
                e.target.classList.add('dragging');
            });
            syllableEl.addEventListener('dragend', e => {
                e.target.classList.remove('dragging');
                draggedSyllable = null;
            });

            // --- Eventos de Toque para Celular ---
            let originalParent = null;
            let isTouching = false;

            syllableEl.addEventListener('touchstart', e => {
                if (e.target.parentNode.dataset.color) return; // Não iniciar arrasto de peça já encaixada

                isTouching = true;
                draggedSyllable = e.target;
                originalParent = draggedSyllable.parentNode;

                // Para um arrasto suave, usamos uma posição fixa e movemos com o dedo
                const rect = draggedSyllable.getBoundingClientRect();
                draggedSyllable.style.position = 'fixed';
                draggedSyllable.style.width = `${rect.width}px`;
                draggedSyllable.style.height = `${rect.height}px`;
                draggedSyllable.style.top = `${rect.top}px`;
                draggedSyllable.style.left = `${rect.left}px`;
                draggedSyllable.style.zIndex = '1000';
                
                document.body.appendChild(draggedSyllable); // Mover para o body para evitar problemas de recorte
                draggedSyllable.classList.add('dragging');

            }, { passive: false });

            document.addEventListener('touchmove', e => {
                if (!isTouching || !draggedSyllable) return;
                e.preventDefault(); 
                
                const touch = e.touches[0];
                // Centraliza o elemento no dedo
                draggedSyllable.style.left = `${touch.clientX - draggedSyllable.offsetWidth / 2}px`;
                draggedSyllable.style.top = `${touch.clientY - draggedSyllable.offsetHeight / 2}px`;

                // Destaca as zonas de soltar em potencial
                const dropTarget = document.elementFromPoint(touch.clientX, touch.clientY);
                document.querySelectorAll('[data-color], #syllable-container').forEach(dz => {
                    dz.classList.remove('drag-over');
                });
                
                if (dropTarget && (dropTarget.dataset.color || dropTarget.id === 'syllable-container' || dropTarget.closest('#syllable-container'))) {
                     const finalTarget = dropTarget.closest('[data-color], #syllable-container');
                     if(finalTarget) finalTarget.classList.add('drag-over');
                }

            }, { passive: false });

            document.addEventListener('touchend', e => {
                if (!isTouching || !draggedSyllable) return;
                isTouching = false;
                
                const touch = e.changedTouches[0];
                const dropTarget = document.elementFromPoint(touch.clientX, touch.clientY);
                
                // Limpa os estilos
                draggedSyllable.classList.remove('dragging');
                draggedSyllable.style.position = '';
                draggedSyllable.style.width = '';
                draggedSyllable.style.height = '';
                draggedSyllable.style.top = '';
                draggedSyllable.style.left = '';
                draggedSyllable.style.zIndex = '';
                document.querySelectorAll('.drag-over').forEach(el => el.classList.remove('drag-over'));
                
                const finalDropZone = dropTarget ? dropTarget.closest('[data-color]') : null;

                // Encontra e lida com o local onde a sílaba foi solta
                if (finalDropZone && !finalDropZone.hasChildNodes() && finalDropZone.dataset.color === draggedSyllable.dataset.color) {
                    finalDropZone.appendChild(draggedSyllable);
                    checkWinCondition();
                } else {
                    // Soltar inválido, retorna para o container de sílabas
                    syllableContainer.appendChild(draggedSyllable);
                    if (finalDropZone) { // Era uma zona de soltar, mas inválida
                        finalDropZone.classList.add('shake-animation');
                        setTimeout(() => finalDropZone.classList.remove('shake-animation'), 500);
                    }
                }
                draggedSyllable = null;
            });
        }

        function addDropZoneEvents(dropZone) {
            dropZone.addEventListener('dragover', e => {
                e.preventDefault();
                if (draggedSyllable && draggedSyllable.dataset.color === dropZone.dataset.color && !dropZone.hasChildNodes()) {
                     dropZone.classList.add('drag-over');
                }
            });
            dropZone.addEventListener('dragleave', e => dropZone.classList.remove('drag-over'));
            dropZone.addEventListener('drop', e => {
                e.preventDefault();
                dropZone.classList.remove('drag-over');
                if (draggedSyllable && draggedSyllable.dataset.color === dropZone.dataset.color && !dropZone.hasChildNodes()) {
                    dropZone.appendChild(draggedSyllable);
                    checkWinCondition();
                } else {
                    dropZone.classList.add('shake-animation');
                    setTimeout(() => dropZone.classList.remove('shake-animation'), 500);
                }
            });
        }
        
        function checkWinCondition() {
            const currentWordIndex = shuffledWordIndices[gameRound];
            const dropZones = wordContainer.children;
            const isComplete = Array.from(dropZones).every(zone => zone.hasChildNodes());

            if (isComplete) {
                let formedWord = '';
                Array.from(dropZones)
                     .sort((a, b) => a.dataset.index - b.dataset.index)
                     .forEach(zone => {
                        formedWord += zone.firstChild.textContent;
                     });
                if (formedWord === words[currentWordIndex].word) {
                    wordDisplay.textContent = words[currentWordIndex].word;
                    wordEmoji.textContent = words[currentWordIndex].emoji;
                    successContainer.classList.remove('hidden');
                    successContainer.classList.add('tada-animation');
                    nextWordBtn.classList.remove('hidden');
                }
            }
        }
        
        function loadNextWord() {
           gameRound++;
           if (gameRound >= words.length) {
               // Fim de jogo, reembaralha e começa de novo
               startGame();
               return;
           }
           successContainer.classList.remove('tada-animation');
           setupLevel();
        }

        // --- INICIALIZAÇÃO ---
        nextWordBtn.addEventListener('click', loadNextWord);
        
        syllableContainer.addEventListener('dragover', e => {
            e.preventDefault();
            syllableContainer.classList.add('drag-over');
        });
        syllableContainer.addEventListener('dragleave', e => syllableContainer.classList.remove('drag-over'));
        syllableContainer.addEventListener('drop', e => {
            e.preventDefault();
            syllableContainer.classList.remove('drag-over');
            if (draggedSyllable) {
                syllableContainer.appendChild(draggedSyllable);
                checkWinCondition();
            }
        });

        startGame();
    </script>
</body>
</html>




