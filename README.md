<!DOCTYPE html>
<html lang="de">
<head>
<meta charset="UTF-8">
<title>Florians Hausmeisterservice – Termin buchen</title>

<style>
body {
    font-family: Arial, sans-serif;
    background: #f4f6f4;
    margin: 0;
    color: #1f2d24;
}

header {
    background: #1f4d3a;
    color: white;
    padding: 20px;
    text-align: center;
}

section {
    max-width: 520px;
    margin: 40px auto;
    background: white;
    padding: 25px;
    border-radius: 8px;
}

h2 {
    color: #1f4d3a;
    margin-top: 25px;
}

select, input, button {
    width: 100%;
    padding: 12px;
    margin-top: 10px;
    font-size: 16px;
}

button {
    background: #ffa500;
    border: none;
    font-weight: bold;
    cursor: pointer;
}

button:disabled {
    background: #ccc;
}

.slot {
    padding: 10px;
    border: 1px solid #1f4d3a;
    margin-top: 8px;
    cursor: pointer;
    border-radius: 5px;
}

.slot.booked {
    background: #ddd;
    cursor: not-allowed;
}

.slot.selected {
    background: #ffa500;
}

.summary {
    margin-top: 20px;
    background: #f4f6f4;
    padding: 15px;
    border-radius: 5px;
}
</style>
</head>

<body>

<header>
<h1>Florians Hausmeisterservice</h1>
<p>Termin direkt buchen</p>
</header>

<section>

<h2>📅 Datum</h2>
<input type="date" id="date">

<h2>⏰ Uhrzeit</h2>
<div id="slots"></div>

<h2>🧰 Aufgabe</h2>
<select id="task">
    <option value="">Bitte auswählen</option>
    <option value="reinigung">Haus- & Treppenhausreinigung</option>
    <option value="garten">Gartenpflege / Heckenschnitt</option>
    <option value="winter">Winterdienst</option>
    <option value="reparatur">Kleinreparaturen</option>
    <option value="muell">Mülltonnenservice</option>
    <option value="kontrolle">Objektkontrolle</option>
</select>

<div class="summary" id="summary" style="display:none;"></div>

<button id="bookBtn" disabled>Termin verbindlich buchen</button>
<p id="msg"></p>

</section>

<script>
const times = ["08:00", "09:00", "10:00", "11:00", "13:00", "14:00", "15:00"];
let selectedTime = null;

const tasks = {
    reinigung: { name: "Haus- & Treppenhausreinigung", price: 29, unit: "Std.", duration: 1 },
    garten: { name: "Gartenpflege / Heckenschnitt", price: 39, unit: "Std.", duration: 2 },
    winter: { name: "Winterdienst", price: 45, unit: "Einsatz", duration: 1 },
    reparatur: { name: "Kleinreparaturen", price: 49, unit: "Std.", duration: 1 },
    muell: { name: "Mülltonnenservice", price: 15, unit: "Termin", duration: 1 },
    kontrolle: { name: "Objektkontrolle", price: 25, unit: "Termin", duration: 1 }
};

document.getElementById("date").addEventListener("change", loadSlots);
document.getElementById("task").addEventListener("change", updateSummary);

function loadSlots() {
    selectedTime = null;
    document.getElementById("bookBtn").disabled = true;
    document.getElementById("summary").style.display = "none";

    const date = document.getElementById("date").value;
    const slotsDiv = document.getElementById("slots");
    slotsDiv.innerHTML = "";

    if (!date) return;

    const bookings = JSON.parse(localStorage.getItem(date)) || [];

    times.forEach(time => {
        const div = document.createElement("div");
        div.className = "slot";
        div.innerText = time;

        if (bookings.find(b => b.time === time)) {
            div.classList.add("booked");
        } else {
            div.onclick = () => selectSlot(div, time);
        }

        slotsDiv.appendChild(div);
    });
}

function selectSlot(div, time) {
    document.querySelectorAll(".slot").forEach(s => s.classList.remove("selected"));
    div.classList.add("selected");
    selectedTime = time;
    updateSummary();
}

function updateSummary() {
    const taskKey = document.getElementById("task").value;
    const summary = document.getElementById("summary");

    if (!selectedTime || !taskKey) {
        summary.style.display = "none";
        document.getElementById("bookBtn").disabled = true;
        return;
    }

    const task = tasks[taskKey];
    const total = task.price * task.duration;

    summary.innerHTML = `
        <strong>Zusammenfassung</strong><br>
        🧰 ${task.name}<br>
        ⏱ Dauer: ${task.duration} ${task.unit}<br>
        💶 Preis: <strong>${total} €</strong>
    `;
    summary.style.display = "block";
    document.getElementById("bookBtn").disabled = false;
}

document.getElementById("bookBtn").onclick = () => {
    const date = document.getElementById("date").value;
    const taskKey = document.getElementById("task").value;
    const task = tasks[taskKey];

    const bookings = JSON.parse(localStorage.getItem(date)) || [];
    bookings.push({ time: selectedTime, task: task.name });
    localStorage.setItem(date, JSON.stringify(bookings));

    document.getElementById("msg").innerText =
        `✅ Termin gebucht:
📅 ${date}
⏰ ${selectedTime}
🧰 ${task.name}
💶 ${task.price * task.duration} €`;

    loadSlots();
};
</script>

</body>
</html>
