---
layout: page
title: Box Breathing
permalink: /box/
---

Try this:



<script src="https://cdn.tailwindcss.com"></script>
<!-- <style>
        /* Custom styles for background and font */
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f0f4f8; /* Light blue-gray background */
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            margin: 0;
            flex-direction: column;
            padding: 20px;
        }
-->
<!--
      /* Animation for the breathing circle */
        .breathing-circle {
            animation-duration: 4s;
            animation-timing-function: linear;
            animation-iteration-count: infinite;
        }
-->
        .inhale-animation {
            animation-name: inhale;
        }
<!--
        .exhale-animation {
            animation-name: exhale;
        }
-->
<!--
        @keyframes inhale {
            0% { transform: scale(1); }
            100% { transform: scale(1.1); }
        }
-->
<!--
        @keyframes exhale {
            0% { transform: scale(1.1); }
            100% { transform: scale(1); }
        }
    </style> 
-->

<body class="bg-gray-100 flex items-center justify-center min-h-screen">
    <div class="container bg-white rounded-xl shadow-lg p-8 md:p-12 w-full max-w-md mx-auto text-center border border-gray-200">
        <h1 class="text-3xl md:text-4xl font-bold text-gray-800 mb-6">Box Breathing</h1>

        <!-- Breathing display area -->
        <div class="breathing-display mb-8 p-6 bg-blue-50 rounded-lg border border-blue-200">
            <div id="phase-icon" class="mb-4 flex justify-center items-center h-24">
                <!-- SVG icons will be dynamically inserted here -->
            </div>
            <p id="breathing-phase" class="text-2xl md:text-3xl font-semibold text-blue-700 mb-2">Select a duration</p>
            <p id="current-duration" class="text-xl md:text-2xl font-medium text-gray-600">00:00</p>
        </div>

        <!-- Session duration buttons -->
        <div class="flex flex-wrap justify-center gap-3 mb-8">
            <button class="duration-btn bg-blue-600 hover:bg-blue-700 text-white font-bold py-3 px-6 rounded-full shadow-md transition-colors duration-200" data-minutes="1">1 Min</button>
            <button class="duration-btn bg-blue-600 hover:bg-blue-700 text-white font-bold py-3 px-6 rounded-full shadow-md transition-colors duration-200" data-minutes="3">3 Min</button>
            <button class="duration-btn bg-blue-600 hover:bg-blue-700 text-white font-bold py-3 px-6 rounded-full shadow-md transition-colors duration-200" data-minutes="5">5 Min</button>
            <button class="duration-btn bg-blue-600 hover:bg-blue-700 text-white font-bold py-3 px-6 rounded-full shadow-md transition-colors duration-200" data-minutes="10">10 Min</button>
        </div>

        <!-- Stop Button -->
        <button id="stop-btn" class="bg-red-500 hover:bg-red-600 text-white font-bold py-3 px-8 rounded-full shadow-md transition-colors duration-200 opacity-50 cursor-not-allowed" disabled>Stop Exercise</button>
    </div>

    <script>
        // Get references to DOM elements
        const breathingPhaseElement = document.getElementById('breathing-phase');
        const currentDurationElement = document.getElementById('current-duration');
        const phaseIconElement = document.getElementById('phase-icon');
        const durationButtons = document.querySelectorAll('.duration-btn');
        const stopButton = document.getElementById('stop-btn');

        // Breathing cycle constants
        const BREATH_DURATION = 4; // seconds for each phase
        const PHASES = ['Inhale', 'Hold', 'Exhale', 'Hold'];
        const ICONS = {
            'Inhale': `<svg xmlns="http://www.w3.org/2000/svg" class="h-20 w-20 text-green-500 animate-pulse-slow breathing-circle inhale-animation" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
                          <path stroke-linecap="round" stroke-linejoin="round" d="M5 10l7-7m0 0l7 7m-7-7v18" />
                      </svg>`, // Up arrow for inhale
            'Hold': `<svg xmlns="http://www.w3.org/2000/svg" class="h-20 w-20 text-gray-500" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
                        <path stroke-linecap="round" stroke-linejoin="round" d="M8 12h.01M12 12h.01M16 12h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
                    </svg>`, // Ellipsis/dots for hold
            'Exhale': `<svg xmlns="http://www.w3.org/2000/svg" class="h-20 w-20 text-red-500 animate-pulse-slow breathing-circle exhale-animation" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
                          <path stroke-linecap="round" stroke-linejoin="round" d="M19 14l-7 7m0 0l-7-7m7 7V3" />
                      </svg>` // Down arrow for exhale
        };

        // State variables
        let currentPhaseIndex = 0;
        let breathingInterval;
        let sessionTimerInterval;
        let totalTimeInSeconds = 0;
        let timeLeft = 0;
        let isRunning = false;

        /**
         * Formats seconds into MM:SS string.
         * @param {number} seconds - The total seconds.
         * @returns {string} Formatted time string.
         */
        function formatTime(seconds) {
            const minutes = Math.floor(seconds / 60);
            const remainingSeconds = seconds % 60;
            return `${String(minutes).padStart(2, '0')}:${String(remainingSeconds).padStart(2, '0')}`;
        }

        /**
         * Updates the breathing phase display.
         */
        function updateBreathingPhase() {
            const currentPhase = PHASES[currentPhaseIndex];
            breathingPhaseElement.textContent = `${currentPhase} for ${BREATH_DURATION} seconds`;
            phaseIconElement.innerHTML = ICONS[currentPhase];

            // Add/remove animation classes based on phase
            if (currentPhase === 'Inhale') {
                phaseIconElement.querySelector('svg').classList.add('inhale-animation');
                phaseIconElement.querySelector('svg').classList.remove('exhale-animation');
            } else if (currentPhase === 'Exhale') {
                phaseIconElement.querySelector('svg').classList.add('exhale-animation');
                phaseIconElement.querySelector('svg').classList.remove('inhale-animation');
            } else {
                phaseIconElement.querySelector('svg').classList.remove('inhale-animation', 'exhale-animation');
            }

            currentPhaseIndex = (currentPhaseIndex + 1) % PHASES.length;
        }

        /**
         * Starts the breathing cycle.
         */
        function startBreathingCycle() {
            // Clear any existing interval to prevent duplicates
            if (breathingInterval) {
                clearInterval(breathingInterval);
            }
            currentPhaseIndex = 0; // Start from Inhale
            updateBreathingPhase(); // Initial update
            breathingInterval = setInterval(updateBreathingPhase, BREATH_DURATION * 1000);
        }

        /**
         * Stops the breathing cycle and session timer.
         */
        function stopExercise() {
            clearInterval(breathingInterval);
            clearInterval(sessionTimerInterval);
            breathingPhaseElement.textContent = 'Exercise Stopped';
            phaseIconElement.innerHTML = ''; // Clear icon
            currentDurationElement.textContent = formatTime(totalTimeInSeconds - timeLeft); // Show time elapsed
            isRunning = false;
            toggleButtons(true); // Enable duration buttons
            stopButton.disabled = true;
            stopButton.classList.add('opacity-50', 'cursor-not-allowed');
            stopButton.classList.remove('hover:bg-red-600');
        }

        /**
         * Starts the session timer.
         * @param {number} durationMinutes - The total duration in minutes.
         */
        function startSessionTimer(durationMinutes) {
            if (isRunning) return; // Prevent starting multiple sessions
            isRunning = true;

            totalTimeInSeconds = durationMinutes * 60;
            timeLeft = totalTimeInSeconds;
            currentDurationElement.textContent = formatTime(timeLeft);

            startBreathingCycle(); // Start the breathing cycle simultaneously

            // Disable duration buttons and enable stop button
            toggleButtons(false);
            stopButton.disabled = false;
            stopButton.classList.remove('opacity-50', 'cursor-not-allowed');
            stopButton.classList.add('hover:bg-red-600');

            sessionTimerInterval = setInterval(() => {
                timeLeft--;
                currentDurationElement.textContent = formatTime(timeLeft);

                if (timeLeft <= 0) {
                    stopExercise();
                    breathingPhaseElement.textContent = 'Session Completed!';
                    phaseIconElement.innerHTML = `<svg xmlns="http://www.w3.org/2000/svg" class="h-20 w-20 text-blue-500" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
                                                    <path stroke-linecap="round" stroke-linejoin="round" d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z" />
                                                </svg>`; // Checkmark icon
                }
            }, 1000); // Update every second
        }

        /**
         * Toggles the enabled state of duration buttons.
         * @param {boolean} enable - True to enable, false to disable.
         */
        function toggleButtons(enable) {
            durationButtons.forEach(button => {
                button.disabled = !enable;
                if (enable) {
                    button.classList.remove('opacity-50', 'cursor-not-allowed');
                    button.classList.add('hover:bg-blue-700');
                } else {
                    button.classList.add('opacity-50', 'cursor-not-allowed');
                    button.classList.remove('hover:bg-blue-700');
                }
            });
        }

        // Add event listeners to duration buttons
        durationButtons.forEach(button => {
            button.addEventListener('click', () => {
                const minutes = parseInt(button.dataset.minutes);
                startSessionTimer(minutes);
            });
        });

        // Add event listener to stop button
        stopButton.addEventListener('click', stopExercise);

        // Initial state when the page loads
        document.addEventListener('DOMContentLoaded', () => {
            // Set initial prompt
            breathingPhaseElement.textContent = 'Select a duration to start.';
            phaseIconElement.innerHTML = ''; // Clear any initial icon
            currentDurationElement.textContent = '00:00';
            toggleButtons(true); // Ensure buttons are enabled initially
        });
    </script>


   