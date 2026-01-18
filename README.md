# meu-habit-tracker
App pessoal para acompanhar treinos, consumo de água e constância usando calendário visual.
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <title>Habit Trainer</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />

  <style>
    :root {
      --bg: #f8fafc;
      --card: #ffffff;
      --primary: #6366f1;
      --text: #1f2937;
    }

    body {
      margin: 0;
      font-family: system-ui, -apple-system, BlinkMacSystemFont;
      background: var(--bg);
      color: var(--text);
    }

    .app {
      display: flex;
      gap: 20px;
      padding: 20px;
      max-width: 1200px;
      margin: auto;
    }

    /* ---------- CALENDÁRIO ---------- */
    .calendar {
      background: var(--card);
      border-radius: 16px;
      padding: 20px;
      width: 100%;
    }

    .calendar-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 10px;
    }

    .calendar-header button {
      border: none;
      background: var(--primary);
      color: #fff;
      border-radius: 8px;
      padding: 6px 10px;
      cursor: pointer;
    }

    .calendar-grid {
      display: grid;
      grid-template-columns: repeat(7, 1fr);
      gap: 8px;
    }

    .day {
      background: #e5e7eb;
      padding: 12px 0;
      border-radius: 10px;
      text-align: center;
      cursor: pointer;
      font-weight: 500;
    }

    .day.workout { background: #86efac; }
    .day.water   { background: #93c5fd; }
    .day.both    { background: #d8b4fe; }

    /* ---------- PAINEL DO DIA ---------- */
    .day-panel {
      background: var(--card);
      border-radius: 16px;
      padding: 20px;
      width: 100%;
    }

    .day-panel h3 {
      margin-top: 0;
    }

    label {
      display: block;
      margin-top: 12px;
      font-size: 14px;
    }

    select, input, textarea {
      width: 100%;
      padding: 8px;
      border-radius: 8px;
      border: 1px solid #d1d5db;
      margin-top: 4px;
    }

    button.save {
      margin-top: 16px;
      width: 100%;
      padding: 10px;
      border-radius: 10px;
      border: none;
      background: var(--primary);
      color: #fff;
      font-size: 16px;
      cursor: pointer;
    }

    /* ---------- RESPONSIVO ---------- */
    @media (max-width: 768px) {
      .app {
        flex-direction: column;
      }
    }
  </style>
</head>

<body>
  <div class="app">

    <!-- CALENDÁRIO -->
    <div class="calendar">
      <div class="calendar-header">
        <button id="prevMonth">◀</button>
        <h2 id="monthYear"></h2>
        <button id="nextMonth">▶</button>
      </div>
      <div class="calendar-grid" id="calendarGrid"></div>
    </div>

    <!-- PAINEL DO DIA -->
    <div class="day-panel" id="dayPanel">
      <h3 id="selectedDate">Selecione um dia</h3>

      <label>Treino</label>
      <select id="workoutType">
        <option value="">Não treinei</option>
        <option value="pernas">Pernas</option>
        <option value="gluteos">Glúteos</option>
        <option value="superior">Superior</option>
        <option value="funcional">Funcional</option>
        <option value="alongamento">Alongamento</option>
      </select>

      <label>Água (ml)</label>
      <input type="number" id="waterInput" placeholder="Ex: 2000" />

      <label>Observação</label>
      <textarea id="noteInput" rows="3"></textarea>

      <button class="save" id="saveDay">Salvar</button>
    </div>

  </div>

  <script>
    const calendarGrid = document.getElementById("calendarGrid");
    const monthYear = document.getElementById("monthYear");
    const WATER_GOAL = 2000;

    let currentDate = new Date();
    let selectedDateKey = null;

    function getData() {
      return JSON.parse(localStorage.getItem("habitData")) || {};
    }

    function saveData(data) {
      localStorage.setItem("habitData", JSON.stringify(data));
    }

    function renderCalendar() {
      calendarGrid.innerHTML = "";
      const year = currentDate.getFullYear();
      const month = currentDate.getMonth();

      monthYear.innerText = currentDate.toLocaleDateString("pt-BR", {
        month: "long",
        year: "numeric"
      });

      const firstDay = new Date(year, month, 1).getDay();
      const daysInMonth = new Date(year, month + 1, 0).getDate();
      const data = getData();

      for (let i = 0; i < firstDay; i++) {
        calendarGrid.appendChild(document.createElement("div"));
      }

      for (let day = 1; day <= daysInMonth; day++) {
        const dayDiv = document.createElement("div");
        dayDiv.classList.add("day");
        dayDiv.innerText = day;

        const dateKey = `${year}-${month + 1}-${day}`;

        if (data[dateKey]) {
          const { treino, agua } = data[dateKey];
          if (treino && agua >= WATER_GOAL) dayDiv.classList.add("both");
          else if (treino) dayDiv.classList.add("workout");
          else if (agua >= WATER_GOAL) dayDiv.classList.add("water");
        }

        dayDiv.onclick = () => openDay(dateKey);
        calendarGrid.appendChild(dayDiv);
      }
    }

    function openDay(dateKey) {
      selectedDateKey = dateKey;
      const data = getData()[dateKey] || {};

      document.getElementById("selectedDate").innerText = dateKey;
      document.getElementById("workoutType").value = data.treino || "";
      document.getElementById("waterInput").value = data.agua || "";
      document.getElementById("noteInput").value = data.obs || "";
    }

    document.getElementById("saveDay").onclick = () => {
      if (!selectedDateKey) return;

      const data = getData();
      data[selectedDateKey] = {
        treino: document.getElementById("workoutType").value,
        agua: Number(document.getElementById("waterInput").value),
        obs: document.getElementById("noteInput").value
      };

      saveData(data);
      renderCalendar();
    };

    document.getElementById("prevMonth").onclick = () => {
      currentDate.setMonth(currentDate.getMonth() - 1);
      renderCalendar();
    };

    document.getElementById("nextMonth").onclick = () => {
      currentDate.setMonth(currentDate.getMonth() + 1);
      renderCalendar();
    };

    renderCalendar();
  </script>
</body>
</html>
