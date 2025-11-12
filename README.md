<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Dynamic AI Quiz Project</title>
    <!-- Tailwind CSS CDN - Used for modern, responsive, and clean design -->
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700;800&display=swap" rel="stylesheet">
    
    <style>
        /* Custom styles and font for a professional, app-like appearance */
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f0f4f8; /* Soft blue background */
        }
        .container-card {
            box-shadow: 0 20px 50px rgba(100, 108, 255, 0.2), 0 5px 15px rgba(0, 0, 0, 0.05);
            border: 1px solid #e0e7ff;
            background-color: white;
        }
        /* Primary (Indigo) Button Style */
        .primary-btn {
            background-color: #6366f1; /* Indigo 500 */
            color: white;
            padding: 14px 28px;
            border-radius: 12px;
            font-weight: 700;
            letter-spacing: 0.05em;
            transition: background-color 0.3s, transform 0.1s;
        }
        .primary-btn:hover:not(:disabled) {
            background-color: #4f46e5; /* Indigo 600 */
            transform: translateY(-2px);
        }
        .primary-btn:disabled {
            opacity: 0.6;
            cursor: not-allowed;
        }
        /* Option Button Styling */
        .option-button {
            transition: all 0.2s ease;
            box-shadow: 0 1px 3px rgba(0,0,0,0.05);
            border: 2px solid #e5e7eb;
        }
        .option-button:hover:not(:disabled) {
            border-color: #a5b4fc; /* Light Indigo hover */
            background-color: #eff6ff;
        }

        /* Animation for correct answer glow (Green flash) */
        .animate-pulse-once {
            animation: pulse-keyframe 0.5s ease-in-out;
        }
        @keyframes pulse-keyframe {
            0% { transform: scale(1); opacity: 1; }
            50% { transform: scale(1.01); opacity: 0.9; }
            100% { transform: scale(1); opacity: 1; }
        }
        /* Loading spinner */
        .loader {
            border: 4px solid rgba(255, 255, 255, 0.3);
            border-top: 4px solid #fff;
            border-radius: 50%;
            width: 20px;
            height: 20px;
            animation: spin 1s linear infinite;
        }
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
    </style>
</head>
<body class="p-4 sm:p-8 min-h-screen flex items-center justify-center">

    <!-- Main App Container -->
    <div class="w-full max-w-lg container-card p-6 sm:p-8 rounded-3xl">
        <h1 class="text-3xl sm:text-4xl font-extrabold text-indigo-700 mb-2 text-center">🧠 The AI Quiz Master</h1>
        <p class="text-center text-gray-500 mb-8 font-medium">Generate **unique** MCQs on any subject in real-time. Project Ready! 🚀</p>

        <!-- 1. Configuration/Input Section -->
        <div id="config-section" class="space-y-6">
            <div>
                <label for="topic-input" class="block text-base font-bold text-gray-700 mb-2">Enter Subject/Topic (Always unique!)</label>
                <input type="text" id="topic-input" placeholder="e.g., Data Structures, World History, or The Solar System"
                       class="w-full p-4 border border-indigo-300 rounded-xl focus:ring-indigo-500 focus:border-indigo-500 shadow-inner">
            </div>
            <div>
                <label for="difficulty-select" class="block text-base font-bold text-gray-700 mb-2">Select Difficulty Level</label>
                <select id="difficulty-select"
                        class="w-full p-4 border border-indigo-300 rounded-xl bg-white focus:ring-indigo-500 focus:border-indigo-500 shadow-inner">
                    <option value="School Level">School Level (Basic)</option>
                    <option value="College/Intermediate" selected>College/Intermediate (Medium)</option>
                    <option value="Expert/Advanced">Expert/Advanced (Advanced)</option>
                </select>
            </div>
            <button id="start-quiz-btn"
                    class="w-full primary-btn mt-6 flex items-center justify-center">
                <span id="button-text">Start The Challenge! (10 Questions)</span>
                <div id="loading-spinner" class="loader hidden ml-3"></div>
            </button>
            <div id="error-message" class="text-red-600 text-sm text-center font-medium mt-3 p-3 bg-red-100 rounded-lg hidden"></div>
        </div>

        <!-- 2. Quiz Display Section -->
        <div id="quiz-container" class="hidden mt-8">
            <!-- Progress and Score Bar -->
            <div class="mb-6 space-y-3">
                <div class="flex justify-between items-center text-sm font-semibold text-gray-700">
                    <span>Progress: <span id="current-q-display">0</span> / 10</span>
                    <span class="text-indigo-600">Score: <span id="score-display">0</span></span>
                </div>
                <div class="w-full bg-gray-200 rounded-full h-3">
                    <div id="progress-bar" class="bg-indigo-500 h-3 rounded-full transition-all duration-500" style="width: 0%"></div>
                </div>
            </div>

            <!-- Questions List -->
            <div id="questions-list" class="space-y-6">
                <!-- Questions will be dynamically added here by JS -->
            </div>
            
            <!-- Quiz Complete Message -->
            <div id="quiz-message" class="text-center text-xl font-extrabold mt-8 p-5 rounded-xl bg-green-100 text-green-700 hidden shadow-lg">
                Quiz Finished! Your Score is...
            </div>
            
            <button id="restart-quiz-btn" class="w-full primary-btn mt-6 bg-red-500 hover:bg-red-600">
                Try New Topic / Restart
            </button>
        </div>

        <!-- 3. Custom Modal (Used instead of built-in alert/confirm) -->
        <div id="custom-modal" class="fixed inset-0 bg-gray-900 bg-opacity-70 flex items-center justify-center p-4 z-50 hidden">
            <div class="bg-white rounded-xl p-6 max-w-sm w-full shadow-2xl transform transition-all">
                <h3 class="text-xl font-bold mb-4 text-gray-800" id="modal-title">Alert!</h3>
                <p class="text-gray-600 mb-6" id="modal-content">This is a message.</p>
                <div class="flex justify-end">
                    <button id="modal-close-btn" class="px-5 py-2 primary-btn bg-indigo-500 hover:bg-indigo-600">
                        OK
                    </button>
                </div>
            </div>
        </div>

    </div>

    <script type="module">
        // =================================================================
        // PROJECT CODE START: AI Dynamic Quiz Logic
        // =================================================================
        
        // Firebase Imports (Standard canvas requirement, although data storage is not used here)
        import { getFirestore } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
        import { getAuth } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        
        // UI element references
        const startQuizBtn = document.getElementById('start-quiz-btn');
        const restartQuizBtn = document.getElementById('restart-quiz-btn');
        const configSection = document.getElementById('config-section');
        const quizContainer = document.getElementById('quiz-container');
        const questionsList = document.getElementById('questions-list');
        const topicInput = document.getElementById('topic-input');
        const difficultySelect = document.getElementById('difficulty-select');
        const scoreDisplay = document.getElementById('score-display');
        const currentQDisplay = document.getElementById('current-q-display');
        const loadingSpinner = document.getElementById('loading-spinner');
        const buttonText = document.getElementById('button-text');
        const errorMessage = document.getElementById('error-message');
        const quizMessage = document.getElementById('quiz-message');
        const progressBar = document.getElementById('progress-bar');
        
        // Modal elements
        const customModal = document.getElementById('custom-modal');
        const modalTitle = document.getElementById('modal-title');
        const modalContent = document.getElementById('modal-content');
        const modalCloseBtn = document.getElementById('modal-close-btn');

        // Global State variables
        const TOTAL_QUESTIONS = 10;
        let currentQuizData = [];
        let score = 0;
        let answeredCount = 0;
        
        // Firebase Setup (Initialization must be included)
        const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : null;
        let app, db, auth;
        if (firebaseConfig) {
            try {
                app = initializeApp(firebaseConfig);
                db = getFirestore(app);
                auth = getAuth(app);
            } catch (error) {
                console.error("Firebase initialization failed:", error);
            }
        }
        
        // Custom Alert function (replaces window.alert for better UX)
        function showCustomAlert(title, message) {
            modalTitle.textContent = title;
            modalContent.textContent = message;
            customModal.classList.remove('hidden');
        }

        modalCloseBtn.onclick = () => {
            customModal.classList.add('hidden');
        };

        // --- Core AI Integration (Gemini API) ---
        
        /**
         * Fetches data from the Gemini API with exponential backoff for resilience.
         * @param {string} apiUrl - The API endpoint URL.
         * @param {object} payload - The request payload.
         * @param {number} retries - Maximum number of retries.
         */
        async function fetchWithBackoff(apiUrl, payload, retries = 3) {
            const apiKey = ""; 
            const finalApiUrl = `${apiUrl}?key=${apiKey}`;

            for (let i = 0; i < retries; i++) {
                try {
                    const response = await fetch(finalApiUrl, {
                        method: 'POST',
                        headers: { 'Content-Type': 'application/json' },
                        body: JSON.stringify(payload)
                    });

                    if (response.ok) {
                        return await response.json();
                    } else if (response.status === 429 && i < retries - 1) {
                        // Rate limit error, retry with backoff
                        const delay = Math.pow(2, i) * 1000 + Math.random() * 500;
                        await new Promise(resolve => setTimeout(resolve, delay));
                        continue;
                    } else {
                        throw new Error(`API request failed with status: ${response.status}`);
                    }
                } catch (error) {
                    if (i === retries - 1) throw error;
                    const delay = Math.pow(2, i) * 1000 + Math.random() * 500;
                    await new Promise(resolve => setTimeout(resolve, delay));
                }
            }
        }

        /**
         * Generates a quiz by calling the Gemini API and enforcing JSON structure.
         * @param {string} topic - The topic provided by the user.
         * @param {string} difficulty - The selected difficulty level.
         * @returns {Promise<Array>} A promise that resolves to an array of quiz questions.
         */
        async function generateQuiz(topic, difficulty) {
            // System instruction defines the model's role and output format.
            const systemPrompt = `You are a sophisticated quiz generator. Your task is to generate exactly ${TOTAL_QUESTIONS} unique, multiple-choice questions (MCQs) on the user-specified topic and difficulty level. Each question must have exactly four distinct options and clearly identify the correct answer text. The output MUST strictly adhere to the provided JSON schema. Ensure the questions are non-repetitive.`;
            const userQuery = `Generate a ${TOTAL_QUESTIONS}-question MCQ quiz on the topic: ${topic} with difficulty level: ${difficulty}.`;

            const payload = {
                contents: [{ parts: [{ text: userQuery }] }],
                tools: [{ "google_search": {} }], // Use Google Search for factual grounding
                systemInstruction: { parts: [{ text: systemPrompt }] },
                generationConfig: {
                    responseMimeType: "application/json",
                    responseSchema: { /* JSON Schema ensures structured output */
                        type: "ARRAY",
                        description: `A list of ${TOTAL_QUESTIONS} quiz questions.`,
                        items: {
                            type: "OBJECT",
                            properties: {
                                "question": { "type": "STRING", "description": "The quiz question text." },
                                "options": { "type": "ARRAY", "items": { "type": "STRING" }, "description": "Exactly four multiple-choice options." },
                                "correctAnswer": { "type": "STRING", "description": "The exact text of the correct option." }
                            },
                            required: ["question", "options", "correctAnswer"]
                        }
                    }
                }
            };
            
            const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent`;

            try {
                const result = await fetchWithBackoff(apiUrl, payload);
                const candidate = result.candidates?.[0];

                if (candidate && candidate.content?.parts?.[0]?.text) {
                    const jsonString = candidate.content.parts[0].text;
                    const quizData = JSON.parse(jsonString);

                    if (Array.isArray(quizData) && quizData.length === TOTAL_QUESTIONS) {
                        return quizData; 
                    }
                    throw new Error("Invalid number of questions or non-array response received.");
                }
                throw new Error("API response content is missing.");

            } catch (error) {
                console.error("Quiz Generation Error:", error);
                throw new Error(`Failed to generate questions. Please check your topic or try again.`);
            }
        }

        // --- Quiz UI Rendering and Interaction ---

        /**
         * Resets state and renders the new quiz questions to the UI.
         */
        function renderQuiz() {
            questionsList.innerHTML = ''; 
            score = 0;
            answeredCount = 0;
            scoreDisplay.textContent = '0';
            currentQDisplay.textContent = '0';
            progressBar.style.width = '0%';
            quizMessage.classList.add('hidden');

            currentQuizData.forEach((q, index) => {
                const questionElement = document.createElement('div');
                // Question card styling
                questionElement.className = 'p-5 bg-white rounded-xl shadow-md border border-gray-100'; 
                questionElement.innerHTML = `
                    <p class="text-lg font-extrabold text-indigo-700 mb-2">Question ${index + 1} of ${TOTAL_QUESTIONS}</p>
                    <p class="text-md font-semibold text-gray-800 mb-4">${q.question}</p>
                    <div class="space-y-3" id="options-${index}">
                        ${q.options.map((option, optIndex) => `
                            <button data-option-index="${optIndex}"
                                    data-question-index="${index}"
                                    class="option-button w-full text-left p-3 rounded-lg bg-gray-50 text-gray-700 font-medium">
                                ${option}
                            </button>
                        `).join('')}
                    </div>
                    <p class="mt-3 text-sm font-semibold hidden" id="feedback-${index}"></p>
                `;
                questionsList.appendChild(questionElement);
            });

            // Attach click listeners to all option buttons
            questionsList.querySelectorAll('.option-button').forEach(button => {
                button.addEventListener('click', handleOptionClick);
            });
        }

        /**
         * Handles the click event for an option button, checks answer, and updates UI.
         * @param {Event} event - The click event.
         */
        function handleOptionClick(event) {
            const clickedButton = event.currentTarget;
            if (clickedButton.disabled) return; 

            const qIndex = parseInt(clickedButton.getAttribute('data-question-index'));
            const selectedOption = clickedButton.textContent.trim();
            const questionData = currentQuizData[qIndex];
            const isCorrect = (selectedOption === questionData.correctAnswer.trim());
            
            const optionsContainer = document.getElementById(`options-${qIndex}`);
            const allOptions = optionsContainer.querySelectorAll('.option-button');

            // 1. Disable all options for the current question
            allOptions.forEach(btn => {
                btn.disabled = true;
                btn.classList.remove('hover:bg-indigo-50', 'hover:border-indigo-400');
            });
            
            // 2. Find and highlight the correct answer (Green background)
            allOptions.forEach(btn => {
                if (btn.textContent.trim() === questionData.correctAnswer.trim()) {
                    btn.classList.add('bg-green-200', 'border-green-500', 'text-green-800', 'animate-pulse-once');
                    btn.classList.remove('bg-gray-50');
                }
            });

            // 3. Highlight the user's selected answer (Red for incorrect, Green for correct)
            if (isCorrect) {
                score++;
                clickedButton.classList.remove('bg-green-200'); // Remove light green if it was the correct one
                clickedButton.classList.add('bg-green-500', 'text-white', 'border-green-700');
            } else {
                // Incorrect answer (Red)
                clickedButton.classList.remove('bg-gray-50');
                clickedButton.classList.add('bg-red-500', 'text-white', 'border-red-700');
                
                // Show feedback message
                const feedbackElement = document.getElementById(`feedback-${qIndex}`);
                feedbackElement.textContent = `❌ Incorrect! The correct answer is: ${questionData.correctAnswer.trim()}`;
                feedbackElement.classList.remove('hidden');
                feedbackElement.classList.add('text-red-600');
            }

            // 4. Update Score and Progress UI
            answeredCount++;
            scoreDisplay.textContent = score;
            currentQDisplay.textContent = answeredCount;
            
            const progress = (answeredCount / TOTAL_QUESTIONS) * 100;
            progressBar.style.width = `${progress}%`;
            
            // 5. Check for Quiz Completion
            if (answeredCount === TOTAL_QUESTIONS) {
                quizMessage.textContent = `🎉 Quiz Complete! You scored ${score} out of ${TOTAL_QUESTIONS}. Great job!`;
                quizMessage.classList.remove('hidden');
                quizMessage.classList.add('bg-indigo-100', 'text-indigo-700');
            }
        }


        /**
         * Handles the main button click to start quiz generation.
         */
        async function handleStartQuiz() {
            const topic = topicInput.value.trim();
            const difficulty = difficultySelect.value;
            
            errorMessage.classList.add('hidden');
            if (topic === '') {
                errorMessage.textContent = '❌ Please enter a clear topic to generate questions!';
                errorMessage.classList.remove('hidden');
                return;
            }
            
            // Set UI to Loading State
            startQuizBtn.disabled = true;
            buttonText.textContent = 'Generating Questions... AI at Work!';
            loadingSpinner.classList.remove('hidden');
            
            try {
                // Call AI to generate Quiz
                currentQuizData = await generateQuiz(topic, difficulty);
                
                // Switch UI to Quiz Display State
                configSection.classList.add('hidden');
                quizContainer.classList.remove('hidden');
                renderQuiz(); 
                
            } catch (error) {
                showCustomAlert("Generation Failed (API Error)", "Could not generate questions. Please ensure your topic is clear and try again.");
                console.error("Quiz Start Failed:", error);

            } finally {
                // Reset UI from Loading State
                startQuizBtn.disabled = false;
                buttonText.textContent = 'Start The Challenge! (10 Questions)';
                loadingSpinner.classList.add('hidden');
            }
        }

        // Event Listeners
        startQuizBtn.addEventListener('click', handleStartQuiz);
        restartQuizBtn.addEventListener('click', () => {
            // Switch back to configuration screen
            configSection.classList.remove('hidden');
            quizContainer.classList.add('hidden');
            questionsList.innerHTML = '';
            // Clear topic input for new subject
            topicInput.value = ''; 
        });

        // =================================================================
        // PROJECT CODE END
        // =================================================================
    </script>
</body>
</html>


