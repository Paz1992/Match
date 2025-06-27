<html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Match the Verb with the Picture</title>
    <!-- Cargar Tailwind CSS para el estilo -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        /* CSS personalizado para estilos base y elementos específicos del juego */
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f0f4f8; /* Fondo azul-gris claro */
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            margin: 0;
            padding: 20px;
            box-sizing: border-box;
        }
        .container {
            background-color: #ffffff;
            padding: 30px;
            border-radius: 16px; /* Esquinas redondeadas para el contenedor principal */
            box-shadow: 0 10px 25px rgba(0, 0, 0, 0.1); /* Sombra suave */
            text-align: center;
            max-width: 900px;
            width: 100%;
        }
        .title {
            font-size: 2.25rem; /* Tamaño de fuente grande para el título */
            font-weight: 700; /* Negrita */
            color: #2c3e50; /* Color de texto azul-gris oscuro */
            margin-bottom: 25px;
        }
        .score-display {
            font-size: 1.2rem;
            font-weight: 600;
            color: #34495e;
            margin-bottom: 15px;
        }
        .game-area {
            display: flex;
            flex-direction: column; /* Apilar verticalmente en pantallas pequeñas */
            gap: 20px;
        }
        /* Media query para pantallas más grandes (punto de ruptura md: y superior) */
        @media (min-width: 768px) {
            .game-area {
                flex-direction: row; /* Organizar horizontalmente en pantallas más grandes */
                justify-content: space-around;
            }
        }
        .images-section, .verbs-section {
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 15px;
            width: 100%; /* Ocupar todo el ancho en pantallas pequeñas */
        }
        /* Estilo para los contenedores de imagen */
        .image-container {
            background-color: #ecf0f1; /* Fondo gris claro */
            border: 2px dashed #bdc3c7; /* Borde punteado para simular un objetivo de arrastre */
            border-radius: 12px;
            padding: 10px;
            width: 100%; /* Ancho fluido */
            max-width: 160px; /* Ancho máximo para los contenedores de imagen */
            aspect-ratio: 1 / 1; /* Mantener relación de aspecto cuadrada */
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            position: relative;
            cursor: grab;
            overflow: hidden;
        }
        .image-container img {
            max-width: 100%;
            max-height: 100%;
            border-radius: 8px;
            object-fit: contain;
        }
        .image-container.has-verb {
            border: 2px solid #2ecc71; /* Borde verde para imágenes correctamente emparejadas */
        }
        .image-container.drag-over {
            background-color: #d0d7dd; /* Fondo ligeramente más oscuro al arrastrar sobre él */
            border-color: #3498db; /* Borde azul al arrastrar sobre él */
        }
        /* Estilo para la etiqueta del verbo arrastrado */
        .verb-label {
            position: absolute;
            bottom: 20px; /* Ajustado para dejar espacio para la retroalimentación individual */
            background-color: #34495e; /* Fondo azul-gris oscuro */
            color: white;
            padding: 4px 8px;
            border-radius: 6px;
            font-size: 0.85rem;
            pointer-events: none; /* Permite hacer clic a través de los elementos subyacentes */
        }
        /* Estilo para los mensajes de retroalimentación individuales debajo de cada imagen */
        .individual-feedback {
            position: absolute;
            bottom: 5px; /* Posicionado debajo de la etiqueta del verbo */
            font-size: 0.9rem; /* Fuente más pequeña para la retroalimentación individual */
            font-weight: 600;
            min-height: 1.2rem; /* Asegura espacio incluso cuando está vacío */
        }
        .individual-feedback.correct {
            color: #2ecc71; /* Verde para retroalimentación correcta */
        }
        .individual-feedback.incorrect {
            color: #e74c3c; /* Rojo para retroalimentación incorrecta */
        }
        /* Estilo para el indicador de carga de imagen */
        .loading-indicator {
            z-index: 10; /* Asegurarse de que esté por encima de la imagen */
            color: #2c3e50;
            font-size: 0.9rem;
            font-weight: 500;
            text-align: center;
        }

        /* Estilo para las tarjetas de verbo */
        .verb-card {
            background-color: #3498db; /* Fondo azul */
            color: white;
            padding: 12px 20px;
            border-radius: 10px;
            cursor: grab;
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
            transition: transform 0.2s ease-in-out;
            width: 100%; /* Ancho fluido */
            max-width: 130px; /* Ancho máximo para las tarjetas de verbo */
            text-align: center;
            font-weight: 600;
        }
        .verb-card:active {
            transform: scale(1.05); /* Ligero efecto de escala al activar */
        }
        .verb-card.dragging {
            opacity: 0.7; /* Reducir opacidad al arrastrar */
        }
        .verb-card.hidden {
            display: none; /* Ocultar tarjetas de verbo emparejadas */
        }
        /* Estilo para el grupo de botones */
        .button-group {
            display: flex;
            justify-content: center;
            gap: 15px;
            margin-top: 25px;
        }
        .button {
            background-color: #2c3e50; /* Botón azul-gris oscuro */
            color: white;
            padding: 12px 25px;
            border-radius: 10px;
            border: none;
            cursor: pointer;
            font-size: 1rem;
            font-weight: 600;
            transition: background-color 0.3s ease, transform 0.2s ease;
        }
        .button:hover {
            background-color: #34495e; /* Ligeramente más claro al pasar el ratón */
            transform: translateY(-2px); /* Ligero levantamiento al pasar el ratón */
        }
        .button:active {
            transform: translateY(0); /* Volver a la posición original al hacer clic */
        }
        /* Estilo del modal */
        .modal {
            display: none; /* Oculto por defecto */
            position: fixed; /* Permanecer en su lugar */
            z-index: 1000; /* Situarse en la parte superior de todo */
            left: 0;
            top: 0;
            width: 100%; /* Ancho completo */
            height: 100%; /* Alto completo */
            overflow: auto; /* Habilitar desplazamiento si es necesario */
            background-color: rgba(0,0,0,0.4); /* Negro con opacidad */
            justify-content: center;
            align-items: center;
        }
        .modal-content {
            background-color: #fefefe;
            margin: auto;
            padding: 20px;
            border: 1px solid #888;
            width: 90%; /* Ancho responsivo */
            max-width: 500px; /* Ancho máximo para pantallas más grandes */
            border-radius: 10px;
            text-align: center;
            box-shadow: 0 5px 15px rgba(0,0,0,0.3);
            position: relative;
        }
        .close-button {
            color: #aaa;
            position: absolute;
            top: 10px;
            right: 15px;
            font-size: 28px;
            font-weight: bold;
            cursor: pointer;
        }
        .close-button:hover,
        .close-button:focus {
            color: black;
            text-decoration: none;
            cursor: pointer;
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="title">Match the verb with the image</div>
        <div id="scoreDisplay" class="score-display">Puntuación: 0</div>
        <div class="game-area">
            <div class="images-section grid grid-cols-2 sm:grid-cols-3 gap-4 md:flex md:flex-row md:flex-wrap md:justify-center">
                <!-- Los contenedores de imagen se cargarán dinámicamente aquí -->
            </div>
            <div class="verbs-section grid grid-cols-2 sm:grid-cols-3 gap-4 md:flex md:flex-row md:flex-wrap md:justify-center">
                <!-- Las tarjetas de verbo se cargarán dinámicamente aquí -->
            </div>
        </div>
        <div class="button-group">
            <button id="restartButton" class="button">Reiniciar</button>
            <button id="nextButton" class="button hidden">Siguiente</button>
        </div>
    </div>

    <!-- El modal para mensajes -->
    <div id="messageModal" class="modal">
        <div class="modal-content">
            <span class="close-button">&times;</span>
            <p id="modalMessage" class="py-4 text-lg"></p>
        </div>
    </div>

    <script>
        // Array de datos del juego: ruta de la imagen y verbo correcto
        const gameData = [
            { id: 'img1', image: '', verb: 'run', prompt: 'a child running happily outdoors' },
            { id: 'img2', image: '', verb: 'read', prompt: 'a child reading a book, focused, indoors' }, // Añadido prompt para "reading"
            { id: 'img3', image: '', verb: 'eat', prompt: 'a child eating a healthy snack, happy' }, // Añadido prompt para "eating"
            { id: 'img4', image: '', verb: 'play', prompt: 'a child playing, with toys, happy' },
            { id: 'img5', image: '', verb: 'jump', prompt: 'a child jumping, energetic, in a park' },
            { id: 'img6', image: '', verb: 'dance', prompt: 'a child dancing, joyful, in a bright setting' }
        ];

        let draggedVerb = null; // Stores the verb DOM element being dragged
        let matchedPairs = 0;   // Counts the number of correctly matched pairs
        let score = 0;          // Tracks the player's score

        // Referencias a elementos del DOM
        const imagesSection = document.querySelector('.images-section');
        const verbsSection = document.querySelector('.verbs-section');
        const nextButton = document.getElementById('nextButton');
        const restartButton = document.getElementById('restartButton');
        const scoreDisplay = document.getElementById('scoreDisplay');
        const messageModal = document.getElementById('messageModal');
        const modalMessage = document.getElementById('modalMessage');
        const closeButton = document.querySelector('.close-button');

        /**
         * Mezcla un array in-situ usando el algoritmo Fisher-Yates (Knuth).
         * @param {Array} array El array a mezclar.
         * @returns {Array} El array mezclado.
         */
        function shuffleArray(array) {
            for (let i = array.length - 1; i > 0; i--) {
                const j = Math.floor(Math.random() * (i + 1));
                [array[i], array[j]] = [array[j], array[i]];
            }
            return array;
        }

        /**
         * Actualiza la puntuación mostrada en la pantalla.
         */
        function updateScoreDisplay() {
            scoreDisplay.textContent = `Puntuación: ${score}`;
        }

        /**
         * Genera una imagen utilizando la API de Imagen 3.0.
         * @param {string} prompt El prompt de texto para la generación de imagen.
         * @returns {Promise<string|null>} Una promesa que se resuelve con la URL de la imagen en base64 o null en caso de error.
         */
        async function generateImage(prompt) {
            const payload = { instances: { prompt: prompt }, parameters: { "sampleCount": 1 } };
            const apiKey = ""; // Canvas proporcionará esto automáticamente en tiempo de ejecución
            const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/imagen-3.0-generate-002:predict?key=${apiKey}`;

            try {
                const response = await fetch(apiUrl, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(payload)
                });
                const result = await response.json();

                if (result.predictions && result.predictions.length > 0 && result.predictions[0].bytesBase64Encoded) {
                    return `data:image/png;base64,${result.predictions[0].bytesBase64Encoded}`;
                } else {
                    console.error('La generación de imagen falló:', result);
                    showMessage('Error al generar la imagen. Usando una imagen de respaldo.', 'error');
                    return null;
                }
            } catch (error) {
                console.error('Error al obtener la imagen de la API:', error);
                showMessage('Error de red al generar la imagen. Usando una imagen de respaldo.', 'error');
                return null;
            }
        }

        /**
         * Inicializa el tablero del juego limpiando elementos previos,
         * mezclando los datos del juego y creando nuevos contenedores de imagen y tarjetas de verbo.
         */
        async function initializeGame() {
            imagesSection.innerHTML = ''; // Limpiar imágenes previas
            verbsSection.innerHTML = '';  // Limpiar verbos previos
            nextButton.classList.add('hidden'); // Ocultar el botón de siguiente
            matchedPairs = 0; // Restablecer el contador de pares emparejados
            score = 0;        // Restablecer la puntuación al inicio del juego
            updateScoreDisplay(); // Actualizar la visualización de la puntuación

            const shuffledGameData = shuffleArray([...gameData]); // Mezclar los datos del juego para las imágenes
            // Crear un array separado de verbos y mezclarlos independientemente
            const shuffledVerbs = shuffleArray(gameData.map(data => data.verb));

            // Crear y añadir contenedores de imagen a la sección de imágenes
            for (const item of shuffledGameData) { // Usar for...of para permitir await
                const imgContainer = document.createElement('div');
                imgContainer.className = 'image-container group';
                imgContainer.dataset.correctVerb = item.verb;

                const img = document.createElement('img');
                img.alt = `Imagen para ${item.verb}`;
                imgContainer.appendChild(img);

                // Crear el indicador de carga
                const loadingIndicator = document.createElement('div');
                loadingIndicator.className = 'loading-indicator absolute inset-0 flex items-center justify-center bg-gray-200 bg-opacity-75 rounded-lg';
                loadingIndicator.textContent = 'Cargando imagen...';
                loadingIndicator.style.display = 'none'; // Oculto por defecto, mostrar cuando sea necesario
                imgContainer.appendChild(loadingIndicator);

                // Crear la etiqueta del verbo (se adjuntará al arrastrar y soltar)
                const verbLabel = document.createElement('span');
                verbLabel.className = 'verb-label';

                // Crear el div para la retroalimentación individual debajo de cada imagen
                const individualFeedback = document.createElement('div');
                individualFeedback.className = 'individual-feedback';
                imgContainer.appendChild(individualFeedback);


                imagesSection.appendChild(imgContainer);

                // Generar imagen si el verbo tiene un prompt
                if (item.prompt) {
                    loadingIndicator.style.display = 'flex'; // Mostrar indicador de carga
                    img.src = 'https://placehold.co/120x120/CCCCCC/000000?text=Cargando...'; // Marcador de posición temporal
                    img.style.opacity = '0.5'; // Atenuar mientras carga

                    const imageUrl = await generateImage(item.prompt);
                    if (imageUrl) {
                        img.src = imageUrl;
                        img.onload = () => { // Asegurarse de que el indicador de carga se elimine solo después de que la imagen esté completamente cargada
                            loadingIndicator.style.display = 'none';
                            img.style.opacity = '1';
                        };
                        img.onerror = () => { // Fallback si la imagen generada falla al cargar
                            img.src = 'https://placehold.co/120x120/FF0000/FFFFFF?text=Error+Carga';
                            loadingIndicator.style.display = 'none';
                            img.style.opacity = '1';
                            showMessage(`No se pudo cargar la imagen generada para "${item.verb}". Usando una imagen de respaldo.`, 'error');
                        };
                    } else {
                        // Imagen de respaldo si la generación falló
                        img.src = 'https://placehold.co/120x120/FF0000/FFFFFF?text=Imagen+No+Generada';
                        loadingIndicator.style.display = 'none';
                        img.style.opacity = '1';
                    }
                } else {
                    img.src = item.image; // Usar la imagen predefinida para otros verbos
                }

                // Añadir escuchadores de eventos de arrastrar y soltar a los contenedores de imagen
                imgContainer.addEventListener('dragover', allowDrop);
                imgContainer.addEventListener('dragleave', handleDragLeave);
                imgContainer.addEventListener('drop', handleDrop);
            }

            // Crear y añadir tarjetas de verbo a la sección de verbos
            shuffledVerbs.forEach(verb => {
                const verbCard = document.createElement('div');
                verbCard.className = 'verb-card';
                verbCard.textContent = verb;
                verbCard.draggable = true; // Hacer la tarjeta arrastrable
                verbCard.dataset.verb = verb; // Almacenar el texto del verbo en un atributo de datos

                verbsSection.appendChild(verbCard);

                // Añadir escuchadores de eventos de arrastre a las tarjetas de verbo
                verbCard.addEventListener('dragstart', handleDragStart);
                verbCard.addEventListener('dragend', handleDragEnd);
            });
        }

        // --- Manejadores de arrastrar y soltar ---

        /**
         * Maneja el evento dragstart para las tarjetas de verbo.
         * Almacena el elemento del verbo arrastrado y establece los datos para la operación de arrastre.
         * @param {DragEvent} event El evento dragstart.
         */
        function handleDragStart(event) {
            draggedVerb = event.target; // Referencia al elemento DOM que se está arrastrando
            event.target.classList.add('dragging'); // Añadir retroalimentación visual para arrastrar
            // Establecer el texto del verbo como datos para la operación de arrastre
            event.dataTransfer.setData('text/plain', event.target.dataset.verb);
            event.dataTransfer.effectAllowed = 'move'; // Especificar el efecto de arrastre permitido
        }

        /**
         * Maneja el evento dragend para las tarjetas de verbo.
         * Elimina la retroalimentación visual de arrastre.
         * @param {DragEvent} event El evento dragend.
         */
        function handleDragEnd(event) {
            event.target.classList.remove('dragging');
            draggedVerb = null; // Limpiar la referencia del verbo arrastrado
        }

        /**
         * Maneja el evento dragover para los contenedores de imagen.
         * Previene el comportamiento predeterminado para permitir una caída y añade retroalimentación visual.
         * @param {DragEvent} event El evento dragover.
         */
        function allowDrop(event) {
            event.preventDefault(); // Necesario para permitir una caída
            // Añadir retroalimentación visual al objetivo de soltar
            event.target.closest('.image-container').classList.add('drag-over');
            event.dataTransfer.dropEffect = 'move'; // Indicar una operación de movimiento
        }

        /**
         * Maneja el evento dragleave para los contenedores de imagen.
         * Elimina la retroalimentación visual cuando el elemento arrastrado abandona el objetivo de soltar.
         * @param {DragEvent} event El evento dragleave.
         */
        function handleDragLeave(event) {
            event.target.closest('.image-container').classList.remove('drag-over');
        }

        /**
         * Maneja el evento drop en los contenedores de imagen.
         * Comprueba si el verbo soltado es correcto, actualiza la puntuación y proporciona retroalimentación.
         * @param {DragEvent} event El evento drop.
         */
        function handleDrop(event) {
            event.preventDefault(); // Prevenir el comportamiento predeterminado del navegador
            const targetContainer = event.target.closest('.image-container');
            targetContainer.classList.remove('drag-over'); // Eliminar el estilo de arrastrar sobre

            if (!draggedVerb) return; // Si no se está arrastrando ningún verbo, no hacer nada

            const droppedVerbText = draggedVerb.dataset.verb;
            const correctVerb = targetContainer.dataset.correctVerb;
            const individualFeedbackDiv = targetContainer.querySelector('.individual-feedback');

            // Limpiar retroalimentación previa para este contenedor
            individualFeedbackDiv.textContent = '';
            individualFeedbackDiv.classList.remove('correct', 'incorrect');

            // Quitar el verbo si ya existe
            const existingVerbLabel = targetContainer.querySelector('.verb-label');
            if (existingVerbLabel) {
                // Si el verbo arrastrado es el mismo que el ya asignado, no hacer nada y permitir arrastrarlo de nuevo si se suelta fuera
                if (existingVerbLabel.textContent === droppedVerbText) {
                    return;
                }
                // Si se está soltando un nuevo verbo en un contenedor con uno ya asignado,
                // mover el verbo existente de vuelta a la sección de verbos y luego procesar el nuevo.
                const previouslyAssignedVerb = existingVerbLabel.textContent;
                const originalVerbCard = verbsSection.querySelector(`[data-verb="${previouslyAssignedVerb}"]`);
                if (originalVerbCard) {
                    originalVerbCard.classList.remove('hidden');
                }
                existingVerbLabel.remove();
                targetContainer.classList.remove('has-verb');
                targetContainer.dataset.assignedVerb = ''; // Limpiar el verbo asignado
                // Si el intento anterior fue incorrecto, y se quita el verbo, no decrementar puntuación de nuevo.
                // En este caso, el score ya se decrementó en el drop anterior.
            }

            // Crear un elemento span para mostrar el verbo soltado debajo de la imagen
            const verbLabel = document.createElement('span');
            verbLabel.className = 'verb-label';
            verbLabel.textContent = droppedVerbText;
            targetContainer.appendChild(verbLabel);
            targetContainer.dataset.assignedVerb = droppedVerbText; // Almacenar el verbo asignado

            draggedVerb.classList.add('hidden'); // Ocultar la tarjeta de verbo de la lista

            // Comprobar si el verbo soltado es correcto y proporcionar retroalimentación individual
            if (droppedVerbText === correctVerb) {
                individualFeedbackDiv.textContent = '✔️ Correcto';
                individualFeedbackDiv.classList.add('correct');
                targetContainer.classList.add('has-verb'); // Añadir retroalimentación visual para la coincidencia correcta
                matchedPairs++; // Incrementar coincidencias correctas
                score++; // Incrementar puntuación por respuesta correcta
            } else {
                individualFeedbackDiv.textContent = '❌ Incorrecto';
                individualFeedbackDiv.classList.add('incorrect');
                score--; // Decrementar puntuación por respuesta incorrecta
            }
            updateScoreDisplay(); // Actualizar la visualización de la puntuación después de cada intento

            // Comprobar si todos los pares están emparejados
            if (matchedPairs === gameData.length) {
                showMessage("¡Felicidades! Has completado todas las combinaciones. Puedes reiniciar el juego.", "success");
                nextButton.classList.remove('hidden'); // Mostrar el botón de siguiente nivel
            }
        }

        // --- Funciones del modal de mensajes ---

        /**
         * Muestra un mensaje modal personalizado al usuario.
         * @param {string} message El texto del mensaje a mostrar.
         * @param {string} type El tipo de mensaje (ej. "info", "success", "error") para el estilo.
         */
        function showMessage(message, type = "info") {
            modalMessage.textContent = message;
            modalMessage.className = 'py-4 text-lg'; // Restablecer clases y aplicar estilo base
            if (type === "success") {
                modalMessage.classList.add('text-green-600'); // Texto verde de Tailwind
            } else if (type === "error") {
                modalMessage.classList.add('text-red-600'); // Texto rojo de Tailwind
            } else if (type === "info") {
                modalMessage.classList.add('text-blue-600'); // Texto azul de Tailwind
            }
            messageModal.style.display = 'flex'; // Mostrar el modal usando flex para centrar
        }

        /**
         * Oculta el mensaje modal personalizado.
         */
        function hideMessage() {
            modalMessage.style.display = 'none'; // Ocultar el modal
        }

        // Escuchadores de eventos para el botón de cerrar del modal y el clic fuera
        closeButton.addEventListener('click', hideMessage);
        window.addEventListener('click', (event) => {
            if (event.target == messageModal) {
                hideMessage();
            }
        });

        /**
         * Reinicia el juego completo a su estado inicial.
         */
        function resetGame() {
            initializeGame();
        }

        // --- Inicializar el juego cuando la ventana carga ---
        window.onload = async () => { // Hacer onload asíncrono para permitir await
            await initializeGame();
        };

        // Funcionalidad del botón Siguiente (actualmente solo reinicia el juego para una nueva ronda)
        nextButton.addEventListener('click', () => {
            initializeGame();
        });

        // Funcionalidad del botón Reiniciar
        restartButton.addEventListener('click', resetGame);
    </script>
</body>
</html>
 ELABORADO POR: MARIA PAZ PERALTA
