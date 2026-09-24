/* ==========================================
   TELSA BET - VIRTUAL MONEY DEMO
   No real-money gambling or payments.
========================================== */

const STARTING_BALANCE = 100000;

let balance = Number(localStorage.getItem("telsa_balance"));

if (!Number.isFinite(balance)) {
  balance = STARTING_BALANCE;
}

let selections = JSON.parse(
  localStorage.getItem("telsa_selections") || "[]"
);

let betHistory = JSON.parse(
  localStorage.getItem("telsa_bets") || "[]"
);


/* ==========================================
   DEMO MATCHES
========================================== */

const matches = [
  {
    id: 1,
    league: "Premier League",
    time: "18:00",
    home: "Arsenal",
    away: "Chelsea",
    odds: {
      home: 1.72,
      draw: 3.80,
      away: 4.60
    }
  },

  {
    id: 2,
    league: "LaLiga",
    time: "19:30",
    home: "Barcelona",
    away: "Sevilla",
    odds: {
      home: 1.45,
      draw: 4.70,
      away: 6.20
    }
  },

  {
    id: 3,
    league: "Premier League",
    time: "20:00",
    home: "Liverpool",
    away: "Manchester City",
    odds: {
      home: 2.20,
      draw: 3.60,
      away: 2.85
    }
  },

  {
    id: 4,
    league: "Champions League",
    time: "20:45",
    home: "Real Madrid",
    away: "Bayern Munich",
    odds: {
      home: 1.90,
      draw: 3.70,
      away: 3.40
    }
  },

  {
    id: 5,
    league: "Serie A",
    time: "21:00",
    home: "Inter Milan",
    away: "Juventus",
    odds: {
      home: 1.95,
      draw: 3.30,
      away: 3.75
    }
  }
];


/* ==========================================
   DOM ELEMENTS
========================================== */

const balanceElement = document.getElementById("balance");
const matchesElement = document.getElementById("matches");
const selectionsElement = document.getElementById("selections");

const totalOddsElement = document.getElementById("totalOdds");
const potentialWinElement = document.getElementById("potentialWin");

const stakeElement = document.getElementById("stake");
const slipCountElement = document.getElementById("slipCount");

const betSlip = document.getElementById("betSlip");
const floatingSlip = document.getElementById("floatingSlip");

const closeSlip = document.getElementById("closeSlip");
const clearSlip = document.getElementById("clearSlip");
const placeBet = document.getElementById("placeBet");

const toast = document.getElementById("toast");

const sportsSection = document.getElementById("sports-section");
const betsSection = document.getElementById("bets-section");

const historyElement = document.getElementById("bet-history");


/* ==========================================
   SAVE DATA
========================================== */

function saveData() {
  localStorage.setItem("telsa_balance", balance.toString());

  localStorage.setItem(
    "telsa_selections",
    JSON.stringify(selections)
  );

  localStorage.setItem(
    "telsa_bets",
    JSON.stringify(betHistory)
  );
}


/* ==========================================
   FORMAT MONEY
========================================== */

function money(amount) {
  return "₦" + Number(amount).toLocaleString("en-NG", {
    minimumFractionDigits: 2,
    maximumFractionDigits: 2
  });
}


/* ==========================================
   UPDATE BALANCE
========================================== */

function updateBalance() {
  balanceElement.textContent = money(balance);
}


/* ==========================================
   RENDER MATCHES
========================================== */

function renderMatches() {

  matchesElement.innerHTML = "";

  matches.forEach(match => {

    const card = document.createElement("div");

    card.className = "match-card";

    card.innerHTML = `
      <div class="match-top">
        <span>${match.league}</span>
        <span>${match.time}</span>
      </div>

      <div class="teams">
        <div class="team">${match.home}</div>
        <div class="team">${match.away}</div>
      </div>

      <div class="markets">

        <button
          class="odd"
          data-match="${match.id}"
          data-market="Home"
          data-odd="${match.odds.home}">
          <span>1 · ${match.home}</span>
          <strong>${match.odds.home.toFixed(2)}</strong>
        </button>

        <button
          class="odd"
          data-match="${match.id}"
          data-market="Draw"
          data-odd="${match.odds.draw}">
          <span>X · Draw</span>
          <strong>${match.odds.draw.toFixed(2)}</strong>
        </button>

        <button
          class="odd"
          data-match="${match.id}"
          data-market="Away"
          data-odd="${match.odds.away}">
          <span>2 · ${match.away}</span>
          <strong>${match.odds.away.toFixed(2)}</strong>
        </button>

      </div>
    `;

    matchesElement.appendChild(card);
  });

  updateSelectedButtons();
}


/* ==========================================
   ADD / REMOVE SELECTION
========================================== */

function selectOdd(matchId, market, odd) {

  const match = matches.find(item => item.id === matchId);

  if (!match) return;

  /*
    One selection per match.
    If the user clicks another market from
    the same match, it replaces the old one.
  */

  selections = selections.filter(
    selection => selection.matchId !== matchId
  );

  selections.push({
    matchId,
    home: match.home,
    away: match.away,
    market,
    odd: Number(odd)
  });

  saveData();

  renderSlip();

  openSlip();

  showToast(`${market} selected`);
}


function removeSelection(matchId) {

  selections = selections.filter(
    selection => selection.matchId !== matchId
  );

  saveData();

  renderSlip();

  updateSelectedButtons();
}


/* ==========================================
   RENDER BET SLIP
========================================== */

function renderSlip() {

  slipCountElement.textContent = selections.length;

  if (selections.length === 0) {

    selectionsElement.innerHTML = `
      <div class="empty-slip">
        <div>🎟️</div>
        <p>Your bet slip is empty</p>
        <small>Click an odd to add a selection.</small>
      </div>
    `;

    totalOddsElement.textContent = "0.00";
    potentialWinElement.textContent = "₦0.00";

    return;
  }

  selectionsElement.innerHTML = "";

  selections.forEach(selection => {

    const item = document.createElement("div");

    item.className = "selection";

    item.innerHTML = `
      <button
        class="remove-selection"
        data-remove="${selection.matchId}">
        ×
      </button>

      <div class="selection-title">
        ${selection.home} vs ${selection.away}
      </div>

      <div class="selection-market">
        ${selection.market}
      </div>

      <div class="selection-odd">
        Odds: ${selection.odd.toFixed(2)}
      </div>
    `;

    selectionsElement.appendChild(item);
  });

  const totalOdds = calculateTotalOdds();

  totalOddsElement.textContent = totalOdds.toFixed(2);

  updatePotentialWin();
}


/* ==========================================
   TOTAL ODDS
========================================== */

function calculateTotalOdds() {

  if (selections.length === 0) {
    return 0;
  }

  return selections.reduce(
    (total, selection) => total * selection.odd,
    1
  );
}


/* ==========================================
   POTENTIAL WIN
========================================== */

function updatePotentialWin() {

  const stake = Number(stakeElement.value) || 0;

  const totalOdds = calculateTotalOdds();

  const potential = stake * totalOdds;

  potentialWinElement.textContent = money(potential);
}


/* ==========================================
   UPDATE SELECTED ODDS
========================================== */

function updateSelectedButtons() {

  document.querySelectorAll(".odd").forEach(button => {

    const matchId = Number(button.dataset.match);

    const market = button.dataset.market;

    const selected = selections.some(
      selection =>
        selection.matchId === matchId &&
        selection.market === market
    );

    button.classList.toggle("selected", selected);
  });
}


/* ==========================================
   PLACE BET
========================================== */

function placeDemoBet() {

  if (selections.length === 0) {

    showToast("Select at least one odd first");

    return;
  }

  const stake = Number(stakeElement.value);

  if (!Number.isFinite(stake) || stake <= 0) {

    showToast("Enter a valid stake");

    stakeElement.focus();

    return;
  }

  if (stake < 100) {

    showToast("Minimum demo stake is ₦100");

    return;
  }

  if (stake > balance) {

    showToast("Insufficient demo balance");

    return;
  }

  const totalOdds = calculateTotalOdds();

  const potentialWin = stake * totalOdds;

  const bet = {

    id:
      "TB" +
      Date.now().toString().slice(-8),

    date:
      new Date().toLocaleString("en-NG"),

    selections:
      JSON.parse(JSON.stringify(selections)),

    stake,

    totalOdds,

    potentialWin,

    status: "Pending"
  };

  balance -= stake;

  betHistory.unshift(bet);

  selections = [];

  stakeElement.value = "";

  saveData();

  updateBalance();

  renderSlip();

  updateSelectedButtons();

  showToast("Demo bet placed successfully");

  setTimeout(() => {

    closeBetSlip();

    showSection("bets");

  }, 700);
}


/* ==========================================
   BET HISTORY
========================================== */

function renderHistory() {

  if (betHistory.length === 0) {

    historyElement.innerHTML = `
      <div class="no-bets">
        <div style="font-size:40px;margin-bottom:10px;">🎟️</div>
        <strong>No bets yet</strong>
        <p style="margin-top:7px;">
          Your demo bets will appear here.
        </p>
      </div>
    `;

    return;
  }

  historyElement.innerHTML = "";

  betHistory.forEach(bet => {

    const card = document.createElement("div");

    card.className = "history-card";

    let selectionsHTML = "";

    bet.selections.forEach(selection => {

      selectionsHTML += `
        <div class="history-selection">
          <strong>
            ${selection.home} vs ${selection.away}
          </strong>
          <br>
          ${selection.market}
          @ ${selection.odd.toFixed(2)}
        </div>
      `;
    });

    card.innerHTML = `
      <div class="history-top">

        <div>
          <strong>Demo Bet</strong>
          <div class="history-id">
            ${bet.id} · ${bet.date}
          </div>
        </div>

        <span class="status">
          ${bet.status}
        </span>

      </div>

      ${selectionsHTML}

      <div class="history-bottom">

        <span>
          Stake: <strong>${money(bet.stake)}</strong>
        </span>

        <span>
          Potential: <strong>${money(bet.potentialWin)}</strong>
        </span>

      </div>

      <div class="history-bottom">
        <span>Total Odds</span>
        <strong>${bet.totalOdds.toFixed(2)}</strong>
      </div>
    `;

    historyElement.appendChild(card);
  });
}


/* ==========================================
   OPEN / CLOSE BET SLIP
========================================== */

function openSlip() {
  betSlip.classList.add("open");
}

function closeBetSlip() {
  betSlip.classList.remove("open");
}


/* ==========================================
   CLEAR SLIP
========================================== */

function clearSelections() {

  selections = [];

  saveData();

  renderSlip();

  updateSelectedButtons();

  showToast("Bet slip cleared");
}


/* ==========================================
   NAVIGATION
========================================== */

function showSection(section) {

  document.querySelectorAll(".nav-btn").forEach(button => {

    button.classList.toggle(
      "active",
      button.dataset.section === section
    );
  });

  if (section === "sports") {

    sportsSection.classList.add("active");
    betsSection.classList.remove("active");

  } else {

    sportsSection.classList.remove("active");
    betsSection.classList.add("active");

    renderHistory();
  }
}


/* ==========================================
   TOAST MESSAGE
========================================== */

let toastTimer;

function showToast(message) {

  toast.textContent = message;

  toast.classList.add("show");

  clearTimeout(toastTimer);

  toastTimer = setTimeout(() => {

    toast.classList.remove("show");

  }, 2500);
}


/* ==========================================
   EVENT LISTENERS
========================================== */

/* Odds */

matchesElement.addEventListener("click", event => {

  const button = event.target.closest(".odd");

  if (!button) return;

  const matchId = Number(button.dataset.match);

  const market = button.dataset.market;

  const odd = Number(button.dataset.odd);

  selectOdd(matchId, market, odd);
});


/* Remove selection */

selectionsElement.addEventListener("click", event => {

  const button =
    event.target.closest("[data-remove]");

  if (!button) return;

  removeSelection(
    Number(button.dataset.remove)
  );
});


/* Stake */

stakeElement.addEventListener(
  "input",
  updatePotentialWin
);


/* Place bet */

placeBet.addEventListener(
  "click",
  placeDemoBet
);


/* Clear */

clearSlip.addEventListener(
  "click",
  clearSelections
);


/* Close */

closeSlip.addEventListener(
  "click",
  closeBetSlip
);


/* Floating slip */

floatingSlip.addEventListener(
  "click",
  openSlip
);


/* Navigation */

document.querySelectorAll(".nav-btn").forEach(button => {

  button.addEventListener("click", () => {

    showSection(button.dataset.section);

  });

});


/* ==========================================
   INITIAL LOAD
========================================== */

updateBalance();

renderMatches();

renderSlip();

renderHistory();
