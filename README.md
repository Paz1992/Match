<html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Juego de Arrastrar y Soltar Verbos</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&display=swap" rel="stylesheet">
    <!-- Chosen Palette: Calm Harmony -->
    <!-- Application Structure Plan: The application uses a single-view, dashboard-style layout. This structure was chosen for its directness and suitability for a focused game. The user is presented with all necessary components—instructions, score, image targets, and draggable verbs—without needing to navigate or scroll. This minimizes cognitive load and allows the user to concentrate entirely on the matching task. The interactive flow is intuitive: drag a verb, drop it on an image, receive immediate feedback, and see the score update. This design promotes a cycle of action and feedback, which is highly effective for learning and engagement. -->
    <!-- Visualization & Content Choices: The core of the application is a grid of interactive cards. Report Info: Associating verbs with images. Goal: Reinforce vocabulary through action. Viz/Presentation Method: A drag-and-drop interface with image cards (drop targets) and verb tags (draggable items). Interaction: Users physically move verbs to images. Justification: This kinesthetic interaction is more memorable and engaging than simple clicking. It turns a learning task into a hands-on game. Library/Method: Vanilla JS for logic, HTML Drag and Drop API for interactivity. The immediate visual feedback (color changes, messages) provides instant gratification and correction. -->
    <!-- CONFIRMATION: NO SVG graphics used. NO Mermaid JS used. -->
    <style>
        body { font-family: 'Inter', sans-serif; }
        .verb-tag.dragging { opacity: 0.5; transform: scale(1.1); }
        .drop-zone { transition: background-color 0.3s, border-color 0.3s; }
        .feedback-message { transition: opacity 0.5s, transform 0.5s; }
        .feedback-message.show { opacity: 1; transform: translateY(0); }
        .feedback-message.hide { opacity: 0; transform: translateY(-20px); }
    </style>
</head>
<body class="bg-gradient-to-br from-stone-100 to-sky-100 flex items-center justify-center min-h-screen p-4">

    <main id="game-container" class="bg-white/80 backdrop-blur-sm p-4 sm:p-6 lg:p-8 rounded-2xl shadow-xl max-w-5xl w-full mx-auto text-center">
        
        <header class="mb-6">
            <h1 class="text-3xl md:text-4xl font-extrabold text-slate-800 tracking-tight mb-2">Match the Verb</h1>
            <p class="text-base md:text-lg text-slate-600">Drag and drop the correct verb onto each image.</p>
        </header>

        <div class="mb-4 text-2xl font-bold text-slate-700">
            Score: <span id="score-display" class="text-sky-600">0</span>
        </div>

        <div id="message-area" class="h-12 flex items-center justify-center mb-4">
             <div id="message-content" class="feedback-message hide p-3 rounded-xl text-base font-semibold"></div>
        </div>

        <div id="image-grid" class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-6 mb-8">
        </div>

        <div class="bg-sky-50 p-5 rounded-2xl shadow-inner mb-8">
            <h2 class="text-2xl font-semibold text-sky-800 mb-4">Verbs</h2>
            <div id="verb-bank" class="flex flex-wrap justify-center items-center gap-4 min-h-[60px]">
            </div>
        </div>

        <footer id="action-buttons" class="flex justify-center gap-4">
            <button id="reset-button" class="bg-rose-600 text-white text-lg font-bold px-8 py-3 rounded-xl shadow-lg hover:bg-rose-700 transition-all duration-300 transform hover:scale-105">
                Reset Game
            </button>
        </footer>

    </main>

    <script>
        document.addEventListener('DOMContentLoaded', () => {
            const initialImageVerbPairs = [
                { id: 'img-listen', src: 'https://placehold.co/150x150/89c2d9/f8f9fa?text=Listen', alt: 'Persona escuchando música', correctVerb: 'Listen to' },
                { id: 'img-cook', src: 'https://placehold.co/150x150/f9c74f/272727?text=Cook', alt: 'Cocinero cocinando', correctVerb: 'Cook' },
                { id: 'img-sing', src: 'https://placehold.co/150x150/f8961e/f8f9fa?text=Sing', alt: 'Persona cantando', correctVerb: 'Sing' },
                { id: 'img-run', src: 'https://placehold.co/150x150/90be6d/272727?text=Run', alt: 'Persona corriendo', correctVerb: 'Run' },
                { id: 'img-eat', src: 'https://placehold.co/150x150/f3722c/f8f9fa?text=Eat', alt: 'Persona comiendo', correctVerb: 'Eat' },
                { id: 'img-talk', src: 'https://placehold.co/150x150/43aa8b/f8f9fa?text=Talk', alt: 'Personas hablando', correctVerb: 'Talk' },
            ];

            let gameState = {
                score: 0,
                imageAssignments: [],
                availableVerbs: [],
                draggedVerb: null
            };

            const scoreDisplay = document.getElementById('score-display');
            const imageGrid = document.getElementById('image-grid');
            const verbBank = document.getElementById('verb-bank');
            const resetButton = document.getElementById('reset-button');
            const actionButtonsContainer = document.getElementById('action-buttons');
            const messageContent = document.getElementById('message-content');
            let messageTimeout;

            function shuffleArray(array) {
                for (let i = array.length - 1; i > 0; i--) {
                    const j = Math.floor(Math.random() * (i + 1));
                    [array[i], array[j]] = [array[j], array[i]];
                }
                return array;
            }

            function showMessage(text, isCorrect) {
                clearTimeout(messageTimeout);
                messageContent.textContent = text;
                messageContent.classList.remove('bg-green-100', 'text-green-800', 'bg-red-100', 'text-red-800', 'hide');
                messageContent.classList.add('show');
                if (isCorrect) {
                    messageContent.classList.add('bg-green-100', 'text-green-800');
                } else {
                    messageContent.classList.add('bg-red-100', 'text-red-800');
                }
                messageTimeout = setTimeout(() => {
                    messageContent.classList.remove('show');
                    messageContent.classList.add('hide');
                }, 2500);
            }
            
            function updateScore(newScore) {
                gameState.score = Math.max(0, newScore);
                scoreDisplay.textContent = gameState.score;
            }

            function render() {
                imageGrid.innerHTML = '';
                verbBank.innerHTML = '';
                actionButtonsContainer.innerHTML = '';
                actionButtonsContainer.appendChild(resetButton);

                gameState.imageAssignments.forEach(pair => {
                    const isCorrect = pair.assignedVerb && pair.assignedVerb === pair.correctVerb;
                    const isAssigned = !!pair.assignedVerb;
                    
                    let borderColor = 'border-slate-400';
                    let bgColor = 'bg-slate-200';
                    if(isAssigned) {
                        borderColor = isCorrect ? 'border-green-500' : 'border-red-500';
                        bgColor = isCorrect ? 'bg-green-200' : 'bg-red-200';
                    }

                    const card = document.createElement('div');
                    card.className = 'bg-slate-50 p-4 rounded-xl shadow-md flex flex-col items-center justify-between min-h-[250px]';
                    card.dataset.imageId = pair.id;

                    card.innerHTML = `
                        <img src="${pair.src}" alt="${pair.alt}" class="w-36 h-36 rounded-lg object-cover mb-4 shadow-sm" onerror="this.onerror=null; this.src='https://placehold.co/150x150/CCCCCC/000000?text=Error';">
                        <div class="relative w-full h-14 flex items-center justify-center rounded-lg transition-all duration-300 drop-zone border-2 border-dashed ${bgColor} ${borderColor}">
                            <span class="text-xl font-semibold text-slate-700">${pair.assignedVerb || 'Drop Here'}</span>
                            ${isAssigned ? `<button data-verb="${pair.assignedVerb}" class="absolute top-1 right-1 text-slate-600 hover:text-slate-900 text-lg font-bold p-1 rounded-full bg-slate-100/50 hover:bg-slate-200/70 leading-none remove-verb-btn" aria-label="Remove assigned verb">&times;</button>` : ''}
                        </div>
                    `;
                    imageGrid.appendChild(card);
                });

                gameState.availableVerbs.forEach(verb => {
                    const verbEl = document.createElement('div');
                    verbEl.className = 'verb-tag bg-sky-600 text-white px-5 py-2 rounded-lg shadow-md cursor-grab hover:bg-sky-700 transition-colors duration-200';
                    verbEl.textContent = verb;
                    verbEl.draggable = true;
                    verbBank.appendChild(verbEl);
                });

                const assignedCount = gameState.imageAssignments.filter(p => p.assignedVerb).length;
                if (assignedCount === initialImageVerbPairs.length) {
                    const nextButton = document.createElement('button');
                    nextButton.id = 'next-button';
                    nextButton.className = "bg-green-600 text-white text-lg font-bold px-8 py-3 rounded-xl shadow-lg hover:bg-green-700 transition-all duration-300 transform hover:scale-105";
                    nextButton.textContent = "Next Round";
                    actionButtonsContainer.appendChild(nextButton);
                    nextButton.addEventListener('click', resetGame);
                }
                
                addEventListeners();
            }

            function handleDragStart(e) {
                if (e.target.classList.contains('verb-tag')) {
                    gameState.draggedVerb = e.target.textContent;
                    e.dataTransfer.setData('text/plain', gameState.draggedVerb);
                    e.target.classList.add('dragging');
                }
            }

            function handleDragEnd(e) {
                if (e.target.classList.contains('verb-tag')) {
                    e.target.classList.remove('dragging');
                }
            }
            
            function handleDragOver(e) {
                e.preventDefault();
            }

            function handleDrop(e) {
                e.preventDefault();
                const droppedVerb = e.dataTransfer.getData('text/plain');
                if (!droppedVerb) return;

                const cardElement = e.target.closest('[data-image-id]');
                if (!cardElement) return;
                
                const imageId = cardElement.dataset.imageId;
                const targetPair = gameState.imageAssignments.find(p => p.id === imageId);

                if (targetPair.assignedVerb === droppedVerb) return;

                if (targetPair.assignedVerb) {
                    if (targetPair.assignedVerb === targetPair.correctVerb) {
                        updateScore(gameState.score - 1);
                    }
                    gameState.availableVerbs.push(targetPair.assignedVerb);
                }

                targetPair.assignedVerb = droppedVerb;
                gameState.availableVerbs = gameState.availableVerbs.filter(v => v !== droppedVerb);

                if (targetPair.assignedVerb === targetPair.correctVerb) {
                    showMessage(`✔️ Correct! "${targetPair.correctVerb}" is the right verb.`, true);
                    updateScore(gameState.score + 1);
                } else {
                    showMessage(`❌ Incorrect! "${targetPair.assignedVerb}" is not the right verb.`, false);
                }

                render();
            }

            function handleRemoveVerb(e) {
                if (!e.target.classList.contains('remove-verb-btn')) return;

                const verbToRemove = e.target.dataset.verb;
                const imageId = e.target.closest('[data-image-id]').dataset.imageId;
                const targetPair = gameState.imageAssignments.find(p => p.id === imageId);
                
                if (targetPair.assignedVerb === targetPair.correctVerb) {
                    updateScore(gameState.score - 1);
                }

                targetPair.assignedVerb = null;
                gameState.availableVerbs.push(verbToRemove);
                
                render();
            }

            function addEventListeners() {
                document.querySelectorAll('.verb-tag').forEach(el => {
                   el.addEventListener('dragstart', handleDragStart);
                   el.addEventListener('dragend', handleDragEnd);
                });
                document.querySelectorAll('[data-image-id]').forEach(el => {
                   el.addEventListener('dragover', handleDragOver);
                   el.addEventListener('drop', handleDrop);
                });
                document.querySelectorAll('.remove-verb-btn').forEach(el => {
                   el.addEventListener('click', handleRemoveVerb);
                });
            }

            function resetGame() {
                const shuffledPairs = shuffleArray([...initialImageVerbPairs]);
                gameState.imageAssignments = shuffledPairs.map(pair => ({ ...pair, assignedVerb: null }));
                gameState.availableVerbs = shuffleArray(shuffledPairs.map(pair => pair.correctVerb));
                gameState.draggedVerb = null;
                updateScore(0);
                render();
            }

            resetButton.addEventListener('click', resetGame);
            resetGame();
        });
    </script>
</body>
</html>

 ELABORADO POR: MARIA PAZ PERALTA
