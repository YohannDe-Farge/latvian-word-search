<!DOCTYPE html>
<html lang="lv">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Latviešu Vārdu Meklēšana</title>
    <style>
        body {
            font-family: 'Georgia', serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            margin: 0;
            padding: 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        
        .container {
            background: rgba(255, 255, 255, 0.95);
            border-radius: 20px;
            padding: 30px;
            box-shadow: 0 20px 40px rgba(0,0,0,0.1);
            max-width: 800px;
            width: 100%;
        }
        
        h1 {
            color: #4a5568;
            text-align: center;
            margin-bottom: 30px;
            font-size: 2.5em;
            text-shadow: 2px 2px 4px rgba(0,0,0,0.1);
        }
        
        .game-area {
            display: grid;
            grid-template-columns: 1fr 300px;
            gap: 30px;
            align-items: start;
        }
        
        .grid-container {
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        
        .word-grid {
            display: grid;
            grid-template-columns: repeat(12, 35px);
            gap: 2px;
            background: #2d3748;
            padding: 10px;
            border-radius: 10px;
            box-shadow: inset 0 4px 8px rgba(0,0,0,0.3);
        }
        
        .cell {
            width: 35px;
            height: 35px;
            background: linear-gradient(145deg, #ffffff, #e2e8f0);
            border: 1px solid #cbd5e0;
            display: flex;
            align-items: center;
            justify-content: center;
            font-weight: bold;
            font-size: 16px;
            color: #2d3748;
            cursor: pointer;
            border-radius: 4px;
            transition: all 0.2s;
            user-select: none;
        }
        
        .cell:hover {
            background: linear-gradient(145deg, #bee3f8, #90cdf4);
            transform: translateY(-1px);
        }
        
        .cell.selected {
            background: linear-gradient(145deg, #68d391, #48bb78);
            color: white;
            transform: scale(1.1);
        }
        
        .cell.found {
            background: linear-gradient(145deg, #fbb6ce, #f687b3);
            color: white;
            animation: pulse 0.5s;
        }
        
        @keyframes pulse {
            0% { transform: scale(1); }
            50% { transform: scale(1.2); }
            100% { transform: scale(1); }
        }
        
        .word-list {
            background: linear-gradient(145deg, #f7fafc, #edf2f7);
            padding: 20px;
            border-radius: 15px;
            box-shadow: 0 8px 16px rgba(0,0,0,0.1);
        }
        
        .word-list h3 {
            color: #2d3748;
            margin-bottom: 15px;
            font-size: 1.3em;
            text-align: center;
        }
        
        .word-item {
            padding: 8px 12px;
            margin: 5px 0;
            background: white;
            border-radius: 8px;
            cursor: pointer;
            transition: all 0.3s;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }
        
        .word-item:hover {
            background: #e6fffa;
            transform: translateX(5px);
        }
        
        .word-item.found {
            background: linear-gradient(145deg, #c6f6d5, #9ae6b4);
            text-decoration: line-through;
            color: #2f855a;
        }
        
        .progress {
            text-align: center;
            margin-top: 20px;
            font-size: 1.2em;
            color: #2d3748;
        }
        
        .reset-btn {
            background: linear-gradient(145deg, #667eea, #764ba2);
            color: white;
            border: none;
            padding: 12px 24px;
            border-radius: 25px;
            font-size: 16px;
            cursor: pointer;
            margin-top: 20px;
            transition: all 0.3s;
            box-shadow: 0 4px 8px rgba(0,0,0,0.2);
        }
        
        .reset-btn:hover {
            transform: translateY(-2px);
            box-shadow: 0 6px 12px rgba(0,0,0,0.3);
        }
        
        @media (max-width: 768px) {
            .game-area {
                grid-template-columns: 1fr;
            }
            
            .word-grid {
                grid-template-columns: repeat(12, 28px);
            }
            
            .cell {
                width: 28px;
                height: 28px;
                font-size: 14px;
            }
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>🔤 Latviešu Vārdu Meklēšana</h1>
        
        <div class="game-area">
            <div class="grid-container">
                <div class="word-grid" id="wordGrid"></div>
                <button class="reset-btn" onclick="resetGame()">🔄 Sākt no jauna</button>
            </div>
            
            <div class="word-list">
                <h3>Meklējamie vārdi:</h3>
                <div id="wordList"></div>
                <div class="progress" id="progress">Atrasti: 0/10</div>
            </div>
        </div>
    </div>

    <script>
        const words = [
            'MĀJA', 'SAULE', 'ŪDENS', 'MEŽS', 'ZIEDS',
            'MĀTE', 'TĒVS', 'BĒRNS', 'PRIEKS', 'MĪLA'
        ];
        
        let grid = [];
        let foundWords = [];
        let selectedCells = [];
        let isSelecting = false;
        
        const gridSize = 12;
        
        function createGrid() {
            grid = Array(gridSize).fill().map(() => Array(gridSize).fill(''));
            foundWords = [];
            
            // Place words in grid
            words.forEach(word => {
                placeWord(word);
            });
            
            // Fill empty spaces with random letters
            const latvianLetters = 'AĀBCČDEĒFGHIĪJKĶLĻMNŅOPRSŠTUŪVZŽ';
            for (let i = 0; i < gridSize; i++) {
                for (let j = 0; j < gridSize; j++) {
                    if (grid[i][j] === '') {
                        grid[i][j] = latvianLetters[Math.floor(Math.random() * latvianLetters.length)];
                    }
                }
            }
        }
        
        function placeWord(word) {
            const directions = [
                [0, 1],   // horizontal
                [1, 0],   // vertical
                [1, 1],   // diagonal down-right
                [-1, 1],  // diagonal up-right
            ];
            
            let placed = false;
            let attempts = 0;
            
            while (!placed && attempts < 100) {
                const direction = directions[Math.floor(Math.random() * directions.length)];
                const startRow = Math.floor(Math.random() * gridSize);
                const startCol = Math.floor(Math.random() * gridSize);
                
                if (canPlaceWord(word, startRow, startCol, direction)) {
                    for (let i = 0; i < word.length; i++) {
                        const row = startRow + i * direction[0];
                        const col = startCol + i * direction[1];
                        grid[row][col] = word[i];
                    }
                    placed = true;
                }
                attempts++;
            }
        }
        
        function canPlaceWord(word, startRow, startCol, direction) {
            for (let i = 0; i < word.length; i++) {
                const row = startRow + i * direction[0];
                const col = startCol + i * direction[1];
                
                if (row < 0 || row >= gridSize || col < 0 || col >= gridSize) {
                    return false;
                }
                
                if (grid[row][col] !== '' && grid[row][col] !== word[i]) {
                    return false;
                }
            }
            return true;
        }
        
        function renderGrid() {
            const gridElement = document.getElementById('wordGrid');
            gridElement.innerHTML = '';
            
            for (let i = 0; i < gridSize; i++) {
                for (let j = 0; j < gridSize; j++) {
                    const cell = document.createElement('div');
                    cell.className = 'cell';
                    cell.textContent = grid[i][j];
                    cell.dataset.row = i;
                    cell.dataset.col = j;
                    
                    cell.addEventListener('mousedown', startSelection);
                    cell.addEventListener('mouseenter', continueSelection);
                    cell.addEventListener('mouseup', endSelection);
                    
                    gridElement.appendChild(cell);
                }
            }
            
            document.addEventListener('mouseup', endSelection);
        }
        
        function renderWordList() {
            const wordListElement = document.getElementById('wordList');
            wordListElement.innerHTML = '';
            
            words.forEach(word => {
                const wordElement = document.createElement('div');
                wordElement.className = 'word-item';
                wordElement.textContent = word;
                
                if (foundWords.includes(word)) {
                    wordElement.classList.add('found');
                }
                
                wordListElement.appendChild(wordElement);
            });
            
            updateProgress();
        }
        
        function updateProgress() {
            const progressElement = document.getElementById('progress');
            progressElement.textContent = `Atrasti: ${foundWords.length}/${words.length}`;
            
            if (foundWords.length === words.length) {
                setTimeout(() => {
                    alert('Apsveicu! Tu atradi visus vārdus! 🎉');
                }, 500);
            }
        }
        
        function startSelection(event) {
            event.preventDefault();
            isSelecting = true;
            selectedCells = [event.target];
            event.target.classList.add('selected');
        }
        
        function continueSelection(event) {
            if (isSelecting) {
                const lastCell = selectedCells[selectedCells.length - 1];
                if (lastCell && isAdjacent(lastCell, event.target)) {
                    if (!selectedCells.includes(event.target)) {
                        selectedCells.push(event.target);
                        event.target.classList.add('selected');
                    }
                }
            }
        }
        
        function endSelection() {
            if (isSelecting) {
                checkWord();
                clearSelection();
                isSelecting = false;
            }
        }
        
        function isAdjacent(cell1, cell2) {
            const row1 = parseInt(cell1.dataset.row);
            const col1 = parseInt(cell1.dataset.col);
            const row2 = parseInt(cell2.dataset.row);
            const col2 = parseInt(cell2.dataset.col);
            
            return Math.abs(row1 - row2) <= 1 && Math.abs(col1 - col2) <= 1;
        }
        
        function checkWord() {
            const selectedWord = selectedCells.map(cell => cell.textContent).join('');
            const reversedWord = selectedWord.split('').reverse().join('');
            
            if (words.includes(selectedWord) && !foundWords.includes(selectedWord)) {
                foundWords.push(selectedWord);
                selectedCells.forEach(cell => {
                    cell.classList.add('found');
                });
                renderWordList();
            } else if (words.includes(reversedWord) && !foundWords.includes(reversedWord)) {
                foundWords.push(reversedWord);
                selectedCells.forEach(cell => {
                    cell.classList.add('found');
                });
                renderWordList();
            }
        }
        
        function clearSelection() {
            selectedCells.forEach(cell => {
                if (!cell.classList.contains('found')) {
                    cell.classList.remove('selected');
                }
            });
            selectedCells = [];
        }
        
        function resetGame() {
            // Clear found markers
            document.querySelectorAll('.cell').forEach(cell => {
                cell.classList.remove('found', 'selected');
            });
            
            createGrid();
            renderGrid();
            renderWordList();
        }
        
        // Initialize game
        createGrid();
        renderGrid();
        renderWordList();
    </script>
</body>
</html>
