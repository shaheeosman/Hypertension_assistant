<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AHFA Simulator</title>
    <!-- Load Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Load Lucide Icons -->
    <script src="https://unpkg.com/lucide@latest"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap');
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f7f9fc;
        }
        .card {
            box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05);
            transition: all 0.3s ease;
        }
        .status-pill {
            padding: 4px 12px;
            border-radius: 9999px;
            font-weight: 600;
            font-size: 0.875rem;
        }
        .dot-pulse {
            display: flex;
            align-items: center;
        }
        .dot-pulse span {
            display: inline-block;
            width: 8px;
            height: 8px;
            margin: 0 2px;
            background: #4f46e5;
            border-radius: 50%;
            animation: pulse 1.5s infinite ease-in-out;
        }
        .dot-pulse span:nth-child(2) { animation-delay: 0.2s; }
        .dot-pulse span:nth-child(3) { animation-delay: 0.4s; }

        @keyframes pulse {
            0%, 100% { opacity: 1; transform: scale(1); }
            50% { opacity: 0.5; transform: scale(0.8); }
        }
    </style>
</head>
<body class="p-4 sm:p-8 flex justify-center items-start min-h-screen">

    <div class="w-full max-w-4xl">
        <header class="text-center mb-8">
            <h1 class="text-3xl font-bold text-gray-800 flex items-center justify-center space-x-3">
                <i data-lucide="activity" class="w-7 h-7 text-indigo-600"></i>
                <span>AHFA Decision Support Simulator</span>
            </h1>
            <p class="text-gray-500 mt-1">Simulating the Core AI Engine's follow-up and classification logic.</p>
        </header>

        <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">

            <!-- Input Card -->
            <div class="lg:col-span-1 card bg-white p-6 rounded-xl border border-gray-200">
                <h2 class="text-xl font-semibold text-gray-700 mb-4">Input Home BP Reading</h2>
                <form id="bpForm">
                    <div class="mb-4">
                        <label for="systolic" class="block text-sm font-medium text-gray-600 mb-1">Systolic BP (SBP, top number)</label>
                        <input type="number" id="systolic" placeholder="e.g., 135" min="50" max="300" required class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-indigo-500 focus:border-indigo-500 transition duration-150">
                    </div>
                    <div class="mb-6">
                        <label for="diastolic" class="block text-sm font-medium text-gray-600 mb-1">Diastolic BP (DBP, bottom number)</label>
                        <input type="number" id="diastolic" placeholder="e.g., 85" min="30" max="200" required class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-indigo-500 focus:border-indigo-500 transition duration-150">
                    </div>
                    <button type="submit" class="w-full bg-indigo-600 text-white py-2 rounded-lg font-semibold hover:bg-indigo-700 transition duration-200">
                        Run AHFA Analysis
                    </button>
                </form>
            </div>

            <!-- Output Cards -->
            <div class="lg:col-span-2 space-y-6">

                <!-- Classification Output -->
                <div class="card bg-white p-6 rounded-xl border border-gray-200">
                    <h2 class="text-xl font-semibold text-gray-700 mb-4 flex items-center space-x-2">
                        <i data-lucide="pulse" class="w-5 h-5 text-red-500"></i>
                        <span>Risk Prediction Model (3A) Output</span>
                    </h2>
                    <div id="bpOutput" class="space-y-3">
                        <div class="flex items-center justify-between border-b pb-2">
                            <span class="text-gray-500">Classification:</span>
                            <div id="classificationResult" class="status-pill bg-gray-100 text-gray-700">Awaiting Input...</div>
                        </div>
                        <div class="flex items-center justify-between border-b pb-2">
                            <span class="text-gray-500">Urgency Flag:</span>
                            <div id="urgencyFlag" class="status-pill bg-gray-100 text-gray-700">N/A</div>
                        </div>
                        <div class="flex items-center justify-between">
                            <span class="text-gray-500">Clinical Follow-up Action:</span>
                            <div id="clinicalAction" class="text-right text-sm text-gray-700 font-medium">Please enter a BP reading.</div>
                        </div>
                    </div>
                </div>

                <!-- LLM Agent Output -->
                <div class="card bg-white p-6 rounded-xl border border-gray-200">
                    <h2 class="text-xl font-semibold text-gray-700 mb-4 flex items-center space-x-2">
                        <i data-lucide="message-square" class="w-5 h-5 text-indigo-500"></i>
                        <span>LLM Agent Follow-up Message (4.2)</span>
                    </h2>
                    <div id="llmLoading" class="hidden dot-pulse p-4 justify-center">
                        <span></span><span></span><span></span>
                        <span class="ml-2 text-indigo-600 font-medium">Generating personalized message...</span>
                    </div>
                    <div id="llmOutput" class="p-4 bg-indigo-50 rounded-lg border border-indigo-200 text-gray-700 italic min-h-[5rem]">
                        The personalized follow-up message will appear here.
                    </div>
                    <div id="llmError" class="text-red-600 mt-2 text-sm hidden"></div>
                </div>

            </div>

        </div>
    </div>

    <!-- Firebase SDK Imports (Required Boilerplate) -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // --- GLOBAL FIREBASE/AUTH SETUP (MANDATORY) ---
        const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : {};
        let db, auth;

        if (Object.keys(firebaseConfig).length > 0) {
            const app = initializeApp(firebaseConfig);
            db = getFirestore(app);
            auth = getAuth(app);
            
            async function authenticate() {
                try {
                    if (typeof __initial_auth_token !== 'undefined') {
                        await signInWithCustomToken(auth, __initial_auth_token);
                    } else {
                        await signInAnonymously(auth);
                    }
                    console.log("Firebase initialized and user authenticated.");
                } catch (error) {
                    console.error("Firebase authentication failed:", error);
                }
            }
            authenticate();
        } else {
            console.log("Firebase config not available. Running simulation without database connection.");
        }
        // --- END FIREBASE/AUTH SETUP ---
    </script>

    <script>
        // Lucide Icons initialization
        document.addEventListener('DOMContentLoaded', () => {
            lucide.createIcons();
        });

        const bpForm = document.getElementById('bpForm');
        const classificationResult = document.getElementById('classificationResult');
        const urgencyFlag = document.getElementById('urgencyFlag');
        const clinicalAction = document.getElementById('clinicalAction');
        const llmLoading = document.getElementById('llmLoading');
        const llmOutput = document.getElementById('llmOutput');
        const llmError = document.getElementById('llmError');

        const AHFA_MOCK_DATA = {
            adherenceScore: 0.92, // Simulated high adherence
            comorbidities: "Type 2 Diabetes, Mild Sleep Apnea"
        };
        
        const AHFA_MODEL_CONFIG = {
            apiKey: "", // Left empty for runtime injection
            apiUrl: "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent",
        };

        /**
         * Simulates the Anomaly and Risk Prediction Model (3A) and
         * the Clinical Follow-up Protocol (4.1).
         * @param {number} sbp Systolic Blood Pressure
         * @param {number} dbp Diastolic Blood Pressure
         * @returns {{classification: string, color: string, urgency: string, action: string}}
         */
        function classifyBloodPressure(sbp, dbp) {
            let classification, color, urgency, action;

            if (sbp >= 180 || dbp >= 120) {
                classification = "Severe Crisis";
                color = "bg-red-600 text-white";
                urgency = "Severe";
                action = "EMERGENCY PROTOCOL: Immediately prompt patient to call 911 or emergency services. Flag for same-day clinical review if asymptomatic.";
            } else if (sbp >= 140 || dbp >= 90) {
                classification = "Stage 2 Hypertension";
                color = "bg-red-400 text-white";
                urgency = "High";
                action = "PRIORITY ALERT: Send immediate notification to re-measure and contact care team. Flag for immediate clinical review (within 48 hours).";
            } else if (sbp >= 130 || dbp >= 80) {
                classification = "Stage 1 Hypertension";
                color = "bg-orange-400 text-white";
                urgency = "Medium";
                action = "Increase medication adherence reminders. Prompt self-assessment of side effects. Notify clinician of sustained trend.";
            } else if (sbp >= 120 && dbp < 80) {
                classification = "Elevated";
                color = "bg-yellow-400 text-gray-800";
                urgency = "Low";
                action = "Send focused educational content (DASH diet, exercise). Increase HBPM frequency reminders. Schedule non-urgent consultation (3-6 months).";
            } else { // SBP < 120 and DBP < 80
                classification = "Normal";
                color = "bg-green-500 text-white";
                urgency = "None";
                action = "Send positive reinforcement message. Check in on lifestyle habits monthly. Long-term follow-up (6-12 months).";
            }

            return { classification, color, urgency, action };
        }

        /**
         * Simulates the LLM Agent (4.2) using the Gemini API.
         * @param {string} bpClassification The BP classification string.
         * @param {number} sbp The Systolic BP reading.
         * @param {number} dbp The Diastolic BP reading.
         */
        async function generateFollowUpMessage(bpClassification, sbp, dbp) {
            llmLoading.classList.remove('hidden');
            llmOutput.textContent = '';
            llmError.classList.add('hidden');

            const systemPrompt = `You are a warm, empathetic AI Health Coach (the AHFA) supporting a patient with hypertension. Your goal is to provide encouraging, personalized, and clinically relevant feedback based on the patient's data.

            The patient's current average home BP reading is ${sbp}/${dbp} mmHg, classified as "${bpClassification}".
            Their simulated medication adherence score is ${AHFA_MOCK_DATA.adherenceScore.toFixed(2)} (High), and they have comorbidities of: ${AHFA_MOCK_DATA.comorbidities}.
            
            Generate a short, single-paragraph follow-up message (under 60 words).
            - For **Normal** or **Elevated**, focus on positive reinforcement and one simple lifestyle suggestion (e.g., DASH diet).
            - For **Stage 1**, focus on supportive consistency with medication/lifestyle.
            - For **Stage 2** or **Severe Crisis**, provide a serious but calm message emphasizing the need for immediate clinician contact. Do NOT advise medication changes.`;

            const userQuery = `Generate the follow-up message for the patient based on their reading of ${sbp}/${dbp} mmHg and the classification: ${bpClassification}.`;

            const payload = {
                contents: [{ parts: [{ text: userQuery }] }],
                systemInstruction: {
                    parts: [{ text: systemPrompt }]
                },
                // We omit the tools property as grounding is not necessary for this simulated response based on internal data
            };

            const maxRetries = 3;
            for (let attempt = 0; attempt < maxRetries; attempt++) {
                try {
                    const response = await fetch(`${AHFA_MODEL_CONFIG.apiUrl}?key=${AHFA_MODEL_CONFIG.apiKey}`, {
                        method: 'POST',
                        headers: { 'Content-Type': 'application/json' },
                        body: JSON.stringify(payload)
                    });

                    if (!response.ok) {
                        throw new Error(`API returned status ${response.status}`);
                    }

                    const result = await response.json();
                    const text = result.candidates?.[0]?.content?.parts?.[0]?.text;

                    if (text) {
                        llmOutput.textContent = text;
                        llmLoading.classList.add('hidden');
                        return; // Success, exit loop
                    } else {
                        throw new Error("Received empty or malformed response from the model.");
                    }

                } catch (error) {
                    console.error(`Attempt ${attempt + 1} failed:`, error);
                    if (attempt < maxRetries - 1) {
                        const delay = Math.pow(2, attempt) * 1000; // Exponential backoff
                        await new Promise(resolve => setTimeout(resolve, delay));
                    } else {
                        llmLoading.classList.add('hidden');
                        llmError.textContent = "Error: Could not generate personalized message. Please check the console.";
                        llmError.classList.remove('hidden');
                    }
                }
            }
        }

        // Event Listener for Form Submission
        bpForm.addEventListener('submit', (e) => {
            e.preventDefault();
            const systolic = parseInt(document.getElementById('systolic').value);
            const diastolic = parseInt(document.getElementById('diastolic').value);

            if (isNaN(systolic) || isNaN(diastolic)) {
                alert("Please enter valid numeric values for both BP readings.");
                return;
            }

            // 1. Run Classification (Simulated ML Model 3A & Protocol 4.1)
            const result = classifyBloodPressure(systolic, diastolic);

            // Update UI with Classification Results
            classificationResult.textContent = `${systolic}/${diastolic} mmHg - ${result.classification}`;
            classificationResult.className = `status-pill ${result.color}`;
            urgencyFlag.textContent = result.urgency;
            urgencyFlag.className = `status-pill ${result.color}`;
            clinicalAction.textContent = result.action;
            
            // 2. Generate Personalized Message (Simulated LLM Agent 4.2)
            generateFollowUpMessage(result.classification, systolic, diastolic);
        });

    </script>
</body>
</html>
