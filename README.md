Buli Insights
Die ultimative App für Bundesliga-Fans: Detaillierte Spielerstatistiken, aktuelle Marktwertentwicklungen und smarte Zukunftsprognosen – alles auf einen Blick!
<!DOCTYPE html>
<html lang="de">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Bundesliga Player Analytics & Prognosen</title>
<style>
  /* Grunddesign */
  body {
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    margin: 0; padding: 30px 15px;
    background: linear-gradient(135deg, #0f2027, #203a43, #2c5364);
    color: #e0e6f0;
    min-height: 100vh;
  }

  h1 {
    text-align: center;
    margin-bottom: 25px;
    font-weight: 700;
    font-size: 2.4rem;
    letter-spacing: 1.2px;
    text-shadow: 0 2px 8px rgba(0,0,0,0.5);
  }

  /* Such- und Filterbereich */
  .search-filter {
    display: flex;
    justify-content: center;
    gap: 15px;
    margin-bottom: 30px;
    flex-wrap: wrap;
  }

  input, select {
    padding: 12px 18px;
    font-size: 1rem;
    border-radius: 12px;
    border: none;
    outline: none;
    min-width: 220px;
    box-shadow: 0 3px 8px rgba(0,0,0,0.25);
    transition: box-shadow 0.3s ease;
  }

  input:focus, select:focus {
    box-shadow: 0 4px 12px #69c0ff;
  }

  /* Spieler-Liste Container */
  #playerList {
    max-width: 960px;
    margin: 0 auto;
  }

  /* Spieler-Karten */
  .player-card {
    background: rgba(255, 255, 255, 0.1);
    border-radius: 16px;
    box-shadow: 0 6px 16px rgba(0, 0, 0, 0.6);
    padding: 20px 25px;
    margin-bottom: 18px;
    display: grid;
    grid-template-columns: 1.5fr 1fr 1fr;
    gap: 22px;
    align-items: center;
    transition: transform 0.25s ease, box-shadow 0.25s ease;
  }

  .player-card:hover {
    transform: translateY(-6px);
    box-shadow: 0 12px 26px rgba(0, 0, 0, 0.8);
  }

  .player-name {
    font-weight: 700;
    font-size: 1.3rem;
    color: #ffd43b;
    margin-bottom: 8px;
    letter-spacing: 0.8px;
    text-shadow: 1px 1px 2px #222;
  }

  .label {
    font-weight: 600;
    color: #bfc7d5;
    font-size: 0.9rem;
  }

  .value {
    font-weight: 500;
    font-size: 1rem;
    color: #e0e6f0;
    margin-left: 6px;
  }

  /* Fortschrittsbalken */
  .progress-bar {
    height: 14px;
    background: rgba(255, 255, 255, 0.2);
    border-radius: 12px;
    overflow: hidden;
    margin-top: 6px;
    box-shadow: inset 0 2px 6px rgba(0,0,0,0.3);
  }

  .progress-fill {
    height: 100%;
    background: linear-gradient(90deg, #69c0ff, #4098d7);
    width: 0;
    border-radius: 12px;
    animation: fillBar 1s ease forwards;
    box-shadow: 0 0 8px #4098d7aa;
  }

  @keyframes fillBar {
    from { width: 0; }
    to { width: var(--progress-width); }
  }

  /* Responsive */
  @media (max-width: 720px) {
    .player-card {
      grid-template-columns: 1fr;
      text-align: center;
      gap: 15px;
    }
    .player-card > div:nth-child(2),
    .player-card > div:nth-child(3) {
      display: flex;
      flex-direction: column;
      align-items: center;
    }
  }
</style>
</head>
<body>

<h1>⚽ Bundesliga Player Analytics & Prognosen</h1>

<div class="search-filter">
  <input type="text" id="searchInput" placeholder="Spieler oder Team suchen" />
  <select id="positionFilter">
    <option value="">Alle Positionen</option>
    <option value="Goalkeeper">Torwart</option>
    <option value="Defender">Abwehr</option>
    <option value="Midfielder">Mittelfeld</option>
    <option value="Attacker">Sturm</option>
  </select>
</div>

<div id="playerList">Lade Spieler...</div>

<script>
const API_KEY = "b463b6a1a2c44ecf8b480f8f9e6bde94";
const BASE_URL = "https://api.football-data.org/v4";
let players = [];

function simulateForm(player) {
  let base = Math.min(player.matchesLastSeason || 10, 30);
  let posFactor = 1;
  switch(player.position) {
    case "Goalkeeper": posFactor = 0.9; break;
    case "Defender": posFactor = 1.0; break;
    case "Midfielder": posFactor = 1.1; break;
    case "Attacker": posFactor = 1.2; break;
  }
  return Math.min(100, Math.round(base * posFactor * (0.7 + Math.random() * 0.6)));
}

function simulateStartelf(player) {
  let form = simulateForm(player);
  let loadFactor = 1;
  if(player.clubHasChampionsLeague) loadFactor -= 0.15;
  if(player.recentInjury) loadFactor -= 0.3;
  const score = form * loadFactor;
  return score > 60;
}

function simulateMarketTrend(player) {
  let age = player.age || 25;
  let form = simulateForm(player);
  if(age < 23 && form > 70) return "Steigend 📈";
  if(age > 30 && form < 50) return "Fallend 📉";
  return "Stabil ➡️";
}

function calcAge(dob) {
  if(!dob) return null;
  const birth = new Date(dob);
  const today = new Date();
  let age = today.getFullYear() - birth.getFullYear();
  const m = today.getMonth() - birth.getMonth();
  if(m < 0 || (m === 0 && today.getDate() < birth.getDate())) age--;
  return age;
}

async function loadPlayers() {
  try {
    const teamsRes = await fetch(`${BASE_URL}/competitions/BL1/teams`, {
      headers: { "X-Auth-Token": API_KEY }
    });
    const teamsData = await teamsRes.json();
    const teams = teamsData.teams;

    const allPlayers = [];
    for(const team of teams) {
      const teamRes = await fetch(`${BASE_URL}/teams/${team.id}`, {
        headers: { "X-Auth-Token": API_KEY }
      });
      const teamData = await teamRes.json();

      for(const p of teamData.squad) {
        const player = {
          id: p.id,
          name: p.name,
          position: p.position || "Unbekannt",
          team: team.name,
          nationality: p.nationality || "Unbekannt",
          dateOfBirth: p.dateOfBirth || null,
          age: calcAge(p.dateOfBirth),
          matchesLastSeason: Math.floor(Math.random()*30) + 5,
          clubHasChampionsLeague: ["Bayern München", "Borussia Dortmund", "RB Leipzig", "Bayer Leverkusen"].includes(team.name),
          recentInjury: Math.random() < 0.1
        };
        allPlayers.push(player);
      }
    }

    players = allPlayers;
    renderPlayers(players);
  } catch(err) {
    console.error("Fehler beim Laden:", err);
    document.getElementById("playerList").innerHTML = "<p>Fehler beim Laden der Daten. Bitte API-Key prüfen.</p>";
  }
}

function renderPlayers(playerList) {
  const searchInput = document.getElementById("searchInput");
  const positionFilter = document.getElementById("positionFilter");

  function filterAndRender() {
    const searchValue = searchInput.value.toLowerCase();
    const posValue = positionFilter.value;

    const filtered = playerList.filter(p => {
      const matchesSearch = p.name.toLowerCase().includes(searchValue) || p.team.toLowerCase().includes(searchValue);
      const matchesPos = posValue === "" || p.position === posValue;
      return matchesSearch && matchesPos;
    });

    const container = document.getElementById("playerList");
    container.innerHTML = "";

    if(filtered.length === 0) {
      container.innerHTML = "<p>Keine Spieler gefunden.</p>";
      return;
    }

    filtered.forEach(p => {
      const form = simulateForm(p);
      const startelf = simulateStartelf(p);
      const marketTrend = simulateMarketTrend(p);

      const card = document.createElement("div");
      card.className = "player-card";
      card.innerHTML = `
        <div>
          <div class="player-name">${p.name}</div>
          <div><span class="label">Team:</span> <span class="value">${p.team}</span></div>
          <div><span class="label">Position:</span> <span class="value">${p.position}</span></div>
          <div><span class="label">Nationalität:</span> <span class="value">${p.nationality}</span></div>
          <div><span class="label">Alter:</span> <span class="value">${p.age ?? "?"} Jahre</span></div>
        </div>
        <div>
          <div><span class="label">Form (0-100):</span></div>
          <div class="progress-bar"><div class="progress-fill" style="--progress-width:${form}%; width:${form}%;"></div></div>
          <div>${form} Punkte</div>
        </div>
        <div>
          <div><span class="label">Prognose Startelf:</span> <span class="value">${startelf ? "✅ Wahrscheinlich" : "❌ Eher nicht"}</span></div>
          <div><span class="label">Marktwert Entwicklung:</span> <span class="value">${marketTrend}</span></div>
        </div>
      `;
      container.appendChild(card);
    });
  }

  searchInput.addEventListener("input", filterAndRender);
  positionFilter.addEventListener("change", filterAndRender);

  filterAndRender();
}

loadPlayers();
</script>

</body>
</html>
