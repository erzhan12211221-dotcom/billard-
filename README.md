
# billard-
<!doctype html>
<html lang="ru">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Бильярдная — 3 стола</title>

<style>
* {
  box-sizing: border-box;
}

body {
  margin: 0;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
  background: #10151b;
  color: #f4f7fa;
}

header {
  padding: 18px 16px;
  background: #18212a;
  border-bottom: 1px solid #2b3742;
}

h1 {
  margin: 0;
  font-size: 24px;
}

.sub {
  color: #9eacb8;
  font-size: 14px;
  margin-top: 4px;
}

.wrap {
  max-width: 1000px;
  margin: auto;
  padding: 16px;
}

.grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 14px;
}

.card {
  background: #1a232c;
  border: 1px solid #2c3945;
  border-radius: 16px;
  padding: 16px;
}

.card.running {
  border-color: #43c477;
}

.card.paused {
  border-color: #e0a33b;
}

.name {
  font-size: 20px;
  font-weight: 700;
}

.status {
  font-size: 13px;
  color: #9eacb8;
  margin: 5px 0 15px;
}

.time {
  font-size: 42px;
  font-weight: 800;
  font-variant-numeric: tabular-nums;
}

.amount {
  font-size: 25px;
  font-weight: 700;
  margin: 5px 0 16px;
}

.buttons {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 8px;
}

button {
  border: 0;
  border-radius: 10px;
  padding: 12px 10px;
  font-size: 15px;
  font-weight: 700;
  cursor: pointer;
  background: #2b3946;
  color: white;
}

.start {
  background: #278b52;
}

.pause {
  background: #9a6a1e;
}

.stop {
  background: #a83b3b;
}

.reset {
  background: #394652;
}

.summary {
  margin-top: 16px;
  background: #18212a;
  border: 1px solid #2c3945;
  border-radius: 16px;
  padding: 16px;
  display: flex;
  justify-content: space-between;
  gap: 12px;
}

.big {
  font-size: 27px;
  font-weight: 800;
}

.muted {
  color: #9eacb8;
  font-size: 13px;
}

.settings {
  margin-top: 16px;
  background: #18212a;
  border: 1px solid #2c3945;
  border-radius: 16px;
  padding: 14px;
}

label {
  display: block;
  color: #b7c2cb;
  font-size: 13px;
  margin-bottom: 6px;
}

input {
  width: 120px;
  background: #10151b;
  color: white;
  border: 1px solid #394652;
  border-radius: 9px;
  padding: 10px;
  font-size: 16px;
}

.note {
  color: #8f9ca7;
  font-size: 12px;
  margin-top: 8px;
}

@media (max-width: 720px) {
  .grid {
    grid-template-columns: 1fr;
  }

  .time {
    font-size: 38px;
  }

  .summary {
    display: block;
  }
}
</style>
</head>

<body>

<header>
  <div class="wrap">
    <h1>🎱 Бильярдная</h1>
    <div class="sub">3 стола · учёт времени и оплаты</div>
  </div>
</header>

<main class="wrap">

  <div class="grid" id="tables"></div>

  <div class="summary">

    <div>
      <div class="muted">Заработано сейчас</div>
      <div class="big" id="totalMoney">0 сом</div>
    </div>

    <div>
      <div class="muted">Общее время</div>
      <div class="big" id="totalTime">00:00:00</div>
    </div>

  </div>

  <div class="settings">

    <label for="rate">
      Тариф, сом/час
    </label>

    <input
      id="rate"
      type="number"
      min="0"
      step="10"
      value="300"
    >

    <div class="note">
      Например: 300 сом/час = 5 сом/мин.
    </div>

  </div>

</main>

<script>

const KEY = "billiard3_v1";

let state = JSON.parse(
  localStorage.getItem(KEY) || "null"
);

if (!state) {

  state = {
    rate: 300,

    tables: [1, 2, 3].map(i => ({
      id: i,
      status: "stopped",
      startedAt: null,
      elapsed: 0
    }))
  };

}

const $ = id => document.getElementById(id);

$("rate").value = state.rate;

$("rate").addEventListener("change", () => {

  state.rate =
    Math.max(0, Number($("rate").value) || 0);

  save();
  render();

});

function save() {

  localStorage.setItem(
    KEY,
    JSON.stringify(state)
  );

}

function nowElapsed(table) {

  if (
    table.status === "running" &&
    table.startedAt
  ) {

    return table.elapsed +
      (Date.now() - table.startedAt) / 1000;

  }

  return table.elapsed;

}

function money(table) {

  return Math.floor(
    nowElapsed(table) / 60 *
    (state.rate / 60)
  );

}

function formatTime(seconds) {

  seconds = Math.max(
    0,
    Math.floor(seconds)
  );

  const hours =
    Math.floor(seconds / 3600);

  const minutes =
    Math.floor((seconds % 3600) / 60);

  const secs =
    seconds % 60;

  return [
    hours,
    minutes,
    secs
  ]
  .map(x => String(x).padStart(2, "0"))
  .join(":");

}

function start(index) {

  const table =
    state.tables[index];

  if (table.status !== "running") {

    table.status = "running";
    table.startedAt = Date.now();

    save();
    render();

  }

}

function pause(index) {

  const table =
    state.tables[index];

  if (table.status === "running") {

    table.elapsed =
      nowElapsed(table);

    table.startedAt = null;
    table.status = "paused";

    save();
    render();

  }

}

function stop(index) {

  const table =
    state.tables[index];

  if (table.status === "running") {

    table.elapsed =
      nowElapsed(table);

  }

  table.startedAt = null;
  table.status = "stopped";

  save();
  render();

}

function reset(index) {

  if (
    confirm(
      "Сбросить время и сумму для стола " +
      (index + 1) +
      "?"
    )
  ) {

    state.tables[index] = {

      id: index + 1,

      status: "stopped",

      startedAt: null,

      elapsed: 0

    };

    save();
    render();

  }

}

function render() {

  $("tables").innerHTML =
    state.tables.map((table, index) => {

      let statusText;

      if (table.status === "running") {

        statusText = "🟢 Игра идёт";

      } else if (table.status === "paused") {

        statusText = "🟡 Пауза";

      } else {

        statusText = "⚪ Свободен";

      }

      return `

      <section class="card ${table.status}">

        <div class="name">
          🎱 Стол ${index + 1}
        </div>

        <div class="status">
          ${statusText}
        </div>

        <div class="time">
          ${formatTime(nowElapsed(table))}
        </div>

        <div class="amount">
          ${money(table)} сом
        </div>

        <div class="buttons">

          <button
            class="start"
            onclick="start(${index})">
            ▶ Начать
          </button>

          <button
            class="pause"
            onclick="pause(${index})">
            ⏸ Пауза
          </button>

          <button
            class="stop"
            onclick="stop(${index})">
            ⏹ Закончить
          </button>

          <button
            class="reset"
            onclick="reset(${index})">
            ↺ Сбросить
          </button>

        </div>

      </section>

      `;

    }).join("");

  let totalSeconds =
    state.tables.reduce(
      (sum, table) =>
        sum + nowElapsed(table),
      0
    );

  let totalMoney =
    state.tables.reduce(
      (sum, table) =>
        sum + money(table),
      0
    );

  $("totalMoney").textContent =
    Math.floor(totalMoney) + " сом";

  $("totalTime").textContent =
    formatTime(totalSeconds);

}

setInterval(() => {

  render();

}, 1000);

render();

</script>

</body>
</html>