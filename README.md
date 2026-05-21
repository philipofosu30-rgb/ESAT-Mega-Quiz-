<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>ESAT Mega Practice App</title>
<style>
    body {
        background-color: #121212;
        color: #e0e0e0;
        font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
        display: flex;
        justify-content: center;
        align-items: center;
        min-height: 100vh;
        margin: 0;
        padding: 15px;
        box-sizing: border-box;
    }
    .quiz-container {
        width: 100%;
        max-width: 450px;
        background-color: #1e1e1e;
        padding: 24px;
        border-radius: 16px;
        box-shadow: 0 4px 20px rgba(0,0,0,0.5);
    }
    .header {
        font-size: 1.2rem;
        font-weight: bold;
        margin-bottom: 8px;
        color: #ffffff;
        display: flex;
        justify-content: space-between;
        align-items: center;
    }
    .subject-tag {
        font-size: 0.75rem;
        text-transform: uppercase;
        background-color: #3b82f6;
        color: white;
        padding: 4px 10px;
        border-radius: 20px;
        letter-spacing: 0.5px;
    }
    .stats-bar {
        display: flex;
        align-items: center;
        justify-content: space-between;
        margin-bottom: 25px;
        margin-top: 15px;
    }
    .progress-container {
        flex-grow: 1;
        background-color: #333;
        height: 6px;
        border-radius: 4px;
        margin-right: 12px;
        overflow: hidden;
    }
    .progress-bar {
        height: 100%;
        width: 0%;
        background-color: #3b82f6;
        transition: width 0.2s ease;
    }
    .counter {
        font-size: 0.85rem;
        color: #aaa;
        white-space: nowrap;
    }
    .score-badge {
        padding: 4px 8px;
        border-radius: 6px;
        font-size: 0.8rem;
        font-weight: bold;
        margin-left: 6px;
    }
    .score-wrong { background-color: rgba(239, 68, 68, 0.2); color: #ef4444; }
    .score-right { background-color: rgba(34, 197, 94, 0.2); color: #22c55e; }
    .question-text {
        font-size: 1.1rem;
        line-height: 1.45;
        margin-bottom: 25px;
        color: #ffffff;
        min-height: 60px;
    }
    .options-list {
        display: flex;
        flex-direction: column;
        gap: 12px;
        margin-bottom: 30px;
    }
    .option-btn {
        background-color: #2a2a2a;
        color: #e0e0e0;
        border: 1px solid #3a3a3a;
        padding: 15px;
        border-radius: 12px;
        text-align: left;
        font-size: 0.95rem;
        cursor: pointer;
        transition: background-color 0.15s, border-color 0.15s;
    }
    .option-btn:hover { background-color: #333333; }
    .option-btn.selected {
        border-color: #3b82f6;
        background-color: rgba(59, 130, 246, 0.1);
    }
    .option-btn.correct {
        border-color: #22c55e;
        background-color: rgba(34, 197, 94, 0.15);
        color: #22c55e;
    }
    .option-btn.incorrect {
        border-color: #ef4444;
        background-color: rgba(239, 68, 68, 0.15);
        color: #ef4444;
    }
    .nav-buttons {
        display: flex;
        flex-direction: column;
    }
    .primary-btn {
        background-color: #0b57d0;
        color: white;
        border: none;
        padding: 15px;
        border-radius: 100px;
        font-size: 1rem;
        font-weight: 500;
        cursor: pointer;
        text-align: center;
    }
    .primary-btn:hover { background-color: #0b4fc0; }
</style>
</head>
<body>

<div class="quiz-container">
    <div class="header">
        <span>ESAT Engine</span>
        <span class="subject-tag" id="subject-label">Maths</span>
    </div>
    <div class="stats-bar">
        <div class="progress-container">
            <div class="progress-bar" id="progress"></div>
        </div>
        <div class="counter" id="q-count">1 / 500</div>
        <span class="score-badge score-wrong" id="wrong-score">✕ 0</span>
        <span class="score-badge score-right" id="right-score">✓ 0</span>
    </div>

    <div id="quiz-content">
        <div class="question-text" id="question">Loading dynamic question module...</div>
        <div class="options-list" id="options"></div>
        <div class="nav-buttons">
            <button class="primary-btn" id="next-btn" onclick="checkOrNext()">Check Answer</button>
        </div>
    </div>
</div>

<script>
let currentIdx = 0;
const totalQuestions = 500;
let rightAns = 0;
let wrongAns = 0;
let selectedOpt = null;
let answered = false;
let currentQuestionObj = {};

const subjects = ["Maths", "Science", "General Knowledge", "English", "Physics"];

const staticPool = [
    { s: "General Knowledge", q: "Which country is located in West Africa and known for its rich gold coast history?", a: ["Nigeria", "Ghana", "Kenya", "South Africa"], c: 1 },
    { s: "English", q: "Identify the antonym of the word 'Ample'.", a: ["Scarce", "Abundant", "Spacious", "Minimalist"], c: 0 },
    { s: "Science", q: "What is the process by which liquid water changes directly into a vapor below its boiling point?", a: ["Condensation", "Evaporation", "Sublimation", "Melting"], c: 1 },
    { s: "Physics", q: "What is the unit of electrical resistance?", a: ["Volt", "Ampere", "Ohm", "Watt"], c: 2 },
    { s: "Maths", q: "Factorize completely: x² - 9.", a: ["(x-3)(x-3)", "(x+3)(x-3)", "(x+9)(x-1)", "x(x-9)"], c: 1 },
    { s: "English", q: "Choose the correct spelling:", a: ["Accomodation", "Accommodation", "Acomodation", "Accomodasion"], c: 1 }
];

function generateDynamicQuestion(index) {
    if (index < staticPool.length) {
        return staticPool[index];
    }

    const sub = subjects[index % subjects.length];
    let difficultyMultiplier = Math.floor(index / 10) + 2; 
    
    if (sub === "Maths") {
        let val1 = Math.floor(Math.random() * 12) + difficultyMultiplier;
        let val2 = Math.floor(Math.random() * 10) + 3;
        let choice = Math.random() > 0.5;
        
        if (choice) {
            let ans = val1 * val2;
            return {
                s: "Maths",
                q: `Solve for x if: ${val1}x = ${ans}`,
                a: [`x = ${val2}`, `x = ${val2 + 2}`, `x = ${val2 - 1}`, `x = ${Math.floor(ans/2)}`],
                c: 0
            };
        } else {
            let constant = Math.floor(Math.random
