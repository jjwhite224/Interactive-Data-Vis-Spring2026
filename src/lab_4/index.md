---
title: "After the Flattening"
toc: false
---

# [After the Flattening](https://jwhite44.commons.gc.cuny.edu/2025/12/21/after-salah-and-trent-why-liverpools-system-flattened/)

This project continues my earlier Liverpool analysis by asking what happened after the team lost its clearest creative structure. My previous argument focused on how Liverpool became flatter once the Salah–Trent axis faded. This version moves the question forward by asking did Arne Slot’s Liverpool rebuild a stable system, or did the team become a collection of useful parts without a reliable core?

Across the season, Liverpool often looked like a team trying to solve instability through control. They kept possession. They produced goals. They had enough individual quality to win matches. But the problem was repeatability. The same patterns did not consistently appear week to week, especially when the midfield changed, availability dropped, or the team had to control matches away from Anfield.

The central question is: **did Slot’s Liverpool find a new core after the Salah–Trent era, or did the search for control expose how unstable the system had become?**

```js
const matches = await FileAttachment("data/liverpool_matches_completed.csv").csv({ typed: true });
const matchMetrics = await FileAttachment("data/liverpool_match_metrics_long.csv").csv({ typed: true });
const players = await FileAttachment("data/liverpool_players.csv").csv({ typed: true });
const playersLong = await FileAttachment("data/liverpool_players_long.csv").csv({ typed: true });
const attackProfiles = await FileAttachment("data/liverpool_player_attack_profiles_long.csv").csv({ typed: true });
const venueSummary = await FileAttachment("data/liverpool_team_venue_summary.csv").csv({ typed: true });
const optaPassing = await FileAttachment("data/liverpool_opta_passing_carrying.csv").csv({ typed: true });
const optaDefending = await FileAttachment("data/liverpool_opta_defending.csv").csv({ typed: true });
const xgSummary = await FileAttachment("data/liverpool_player_xg_summary.csv").csv({ typed: true });
const playerXgLong = await FileAttachment("data/liverpool_player_xg_match_long.csv").csv({ typed: true });
const teamXg = await FileAttachment("data/liverpool_team_xg_by_match.csv").csv({ typed: true });
function normalizeName(name) {
  return String(name)
    .normalize("NFD")
    .replace(/[\u0300-\u036f]/g, "")
    .toLowerCase()
    .replace(/[^a-z0-9]+/g, "_")
    .replace(/^_|_$/g, "");
}
function safeNumber(value) {
  const number = +value;
  return Number.isFinite(number) ? number : 0;
}
const xgByPlayer = new Map(
  xgSummary.map(d => [d.player_join_key, d])
);
```
```html
<style>
  :root {
    color-scheme: light;
    color: #222222;
    background: #f5f6f7;
  }

  body {
    font-family: Inter, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
    line-height: 1.6;
    color: #212121;
    background: #f5f6f7;
    margin: 0;
    padding: 0 1rem 2rem;
  }

  h1,
  h2,
  h3 {
    color: #111111;
    letter-spacing: -0.02em;
    margin-top: 2.2rem;
    margin-bottom: 0.75rem;
    line-height: 1.2;
  }

  h1 {
    max-width: 900px;
    font-size: clamp(2.25rem, 2.1vw, 3rem);
  }

  h2 {
    font-size: clamp(1.45rem, 1.25vw, 1.8rem);
  }

  p,
  .section-text,
  .note {
    max-width: 760px;
    margin: 0 0 1.5rem;
    font-size: 1rem;
    color: #333333;
  }

  .hero {
    max-width: 900px;
    margin-bottom: 2rem;
  }

  .section-text {
    line-height: 1.75;
  }

  .card {
    background: #ffffff;
    border: 1px solid #e8eaed;
    border-radius: 18px;
    padding: 1.1rem 1.2rem;
    box-shadow: 0 18px 42px rgba(15, 23, 42, 0.08);
    min-height: 110px;
    display: flex;
    flex-direction: column;
    justify-content: center;
  }

  .grid.grid-cols-4 {
    gap: 1rem;
    display: grid;
    grid-template-columns: repeat(4, minmax(0, 1fr));
    margin-bottom: 1.5rem;
  }

  .kpi-number {
    font-size: 2rem;
    font-weight: 800;
    line-height: 1;
    color: #111111;
  }

  .kpi-label {
    font-size: 0.9rem;
    color: #616161;
    margin-top: 0.35rem;
  }

  .note {
    color: #5f6368;
    font-size: 0.95rem;
    margin-top: 0.35rem;
  }

  figure,
  .plot {
    background: #ffffff;
    border-radius: 20px;
    border: 1px solid rgba(224, 224, 224, 0.85);
    box-shadow: 0 18px 36px rgba(15, 23, 42, 0.06);
    margin: 2rem 0;
    padding: 1rem;
  }

  figure > svg,
  .plot > svg {
    border-radius: 16px;
  }

  .plot .plot-title,
  .plot .plot-subtitle,
  figure .plot-title,
  figure .plot-subtitle {
    color: #111111;
    margin: 0 0 0.4rem;
    line-height: 1.3;
  }

  .plot .plot-title {
    margin-bottom: 0.25rem;
  }

  .plot .plot-subtitle {
    margin-top: 0;
    margin-bottom: 0.75rem;
    color: #4f5257;
    font-size: 0.98rem;
  }

  .plot .plot-legend,
  figure .plot-legend {
    gap: 0.75rem;
  }

  table,
  th,
  td {
    border-color: #e1e1e1;
  }

  a {
    color: #1a73e8;
    text-decoration: none;
  }

  a:hover {
    text-decoration: underline;
  }
</style>
```
```js
const totalPoints = d3.sum(matches, d => d.points);
const goalsFor = d3.sum(matches, d => d.goals_for);
const goalsAgainst = d3.sum(matches, d => d.goals_against);
const goalDifference = d3.sum(matches, d => d.goal_diff);

const homePoints = venueSummary.find(d => d.venue === "Home")?.points;
const awayPoints = venueSummary.find(d => d.venue === "Away")?.points;
const optaPassingClean = optaPassing.map(d => ({
  ...d,
  player: d.player,
  apps: +d.apps,
  mins: +d.mins,
  all_carries_total: +d.all_carries_total,
  all_carries_distance_m: +d.all_carries_distance_m,
  all_carries_avg_m: +d.all_carries_avg_m,
  progressive_total: +d.progressive_total,
  progressive_distance_m: +d.progressive_distance_m,
  progressive_avg_m: +d.progressive_avg_m,
  ended_with_shot: +d.ended_with_shot,
  ended_with_goal: +d.ended_with_goal,
  ended_with_chance: +d.ended_with_chance,
  ended_with_assist: +d.ended_with_assist
}));

const optaDefendingClean = optaDefending.map(d => ({
  ...d,
  player: d.player,
  apps: +d.apps,
  mins: +d.mins,
  tackles: +d.tackles,
  ints: +d.ints,
  pos_won: +d.pos_won,
  blocks: +d.blocks,
  clearances: +d.clearances,
  ground_duels_total: +d.ground_duels_total,
  ground_duels_won: +d.ground_duels_won,
  ground_duels_pct: +String(d.ground_duels_pct).replace("%", ""),
  aerial_duels_total: +d.aerial_duels_total,
  aerial_duels_won: +d.aerial_duels_won,
  aerial_duels_pct: +String(d.aerial_duels_pct).replace("%", "")
}));
const optaPlayerControl = players.map(player => {
  const passing = optaPassingClean.find(d => d.player === player.player);
  const defending = optaDefendingClean.find(d => d.player === player.player);

  const mins = +player.minutes || +passing?.mins || +defending?.mins || 0;
  const nineties = mins / 90;

  return {
    ...player,

    opta_mins: mins,

    all_carries_total: passing?.all_carries_total ?? 0,
    progressive_total: passing?.progressive_total ?? 0,
    progressive_distance_m: passing?.progressive_distance_m ?? 0,
    ended_with_chance: passing?.ended_with_chance ?? 0,
    ended_with_shot: passing?.ended_with_shot ?? 0,

    tackles: defending?.tackles ?? 0,
    ints: defending?.ints ?? 0,
    pos_won: defending?.pos_won ?? 0,
    blocks: defending?.blocks ?? 0,
    clearances: defending?.clearances ?? 0,
    ground_duels_pct: defending?.ground_duels_pct ?? 0,
    aerial_duels_pct: defending?.aerial_duels_pct ?? 0,

    progressive_carries_per90: nineties ? (passing?.progressive_total ?? 0) / nineties : 0,
    chance_carries_per90: nineties ? (passing?.ended_with_chance ?? 0) / nineties : 0,
    possession_won_per90: nineties ? (defending?.pos_won ?? 0) / nineties : 0,
    tackles_ints_per90: nineties ? ((defending?.tackles ?? 0) + (defending?.ints ?? 0)) / nineties : 0,
    blocks_clearances_per90: nineties ? ((defending?.blocks ?? 0) + (defending?.clearances ?? 0)) / nineties : 0
  };
});
const playersWithXg = optaPlayerControl.map(d => {
  const xg = xgByPlayer.get(normalizeName(d.player));
  const mins = +d.minutes || +d.opta_mins || 0;
  const nineties = mins / 90;

  const goals = safeNumber(d.goals);
  const goalsPer90 = safeNumber(d.goals_per90) || (nineties ? goals / nineties : 0);
  const xgPer90 = safeNumber(xg?.premier_league_xg_per90);

  return {
    ...d,
    xg_total: safeNumber(xg?.premier_league_xg_total),
    xg_per90: xgPer90,
    goals_per90_clean: goalsPer90,
    goals_minus_xg_per90: goalsPer90 - xgPer90,
    xg_minutes: safeNumber(xg?.premier_league_minutes),
    xg_position: xg?.primary_position_from_xg_page
  };
});
```
```html
<div class="grid grid-cols-4">
  <div class="card">
    <div class="kpi-number">${totalPoints}</div>
    <div class="kpi-label">Points</div>
  </div>

  <div class="card">
    <div class="kpi-number">${goalsFor}</div>
    <div class="kpi-label">Goals For</div>
  </div>

  <div class="card">
    <div class="kpi-number">${goalsAgainst}</div>
    <div class="kpi-label">Goals Against</div>
  </div>

  <div class="card">
    <div class="kpi-number">${goalDifference > 0 ? "+" + goalDifference : goalDifference}</div>
    <div class="kpi-label">Goal Difference</div>
  </div>
</div>


```
## 1. Liverpool started fast, then lost rhythm

<div class="section-text">
Liverpool’s season did not collapse in one clean moment. It moved in pulses, early control, sudden losses, uneven recovery, and stretches where the team looked functional without feeling dominant. That rhythm matters because inconsistency is the central story of the season. The same team could look controlled one week and exposed the next, suggesting that the issue was less about talent and more about the lack of a repeatable structure.
</div>

```js
display(Plot.plot({
  title: "Liverpool’s 2025–26 Premier League season as a result strip",
  width: width,
  height: 220,
  marginLeft: 28,
  x: { label: "Match number" },
  y: { label: null },
  color: {
    domain: ["W", "D", "L"],
    range: ["#1a7f37", "#999999", "#c1121f"],
    legend: true
  },
  marks: [
    Plot.cell(matches, {
      x: "match_number",
      fill: "result",
      tip: true,
      channels: {
        Match: d => d.match_number,
        Opponent: "opponent",
        Venue: "venue",
        Result: "result_label",
        Score: d => `${d.goals_for}-${d.goals_against}`,
        Possession: d => `${d.possession}%`
      }
    })
  ]
}));
```
## 2. The title defense lost shape in stages

<div class="section-text">
The cumulative points line shows a team that never fully fell apart, but also never settled into a title-level rhythm. The early weeks suggested continuity, but once the first run of dropped points arrived, the problem looked less like one bad result and more like a system struggling to stabilize. Liverpool were strong enough to recover from setbacks, but not consistent enough to turn those recoveries into momentum.
</div>

```js


const turningPoints = [
  {
    match_number: 5,
    cumulative_points: matches.find(d => d.match_number === 1)?.cumulative_points,
    note: "Perfect start"
  },
  {
    match_number: 9,
    cumulative_points: matches.find(d => d.match_number === 9)?.cumulative_points,
    note: "First major wobble"
  },
  {
    match_number: 27,
    cumulative_points: matches.find(d => d.match_number === 27)?.cumulative_points,
    note: "Late-season drift"
  }
];


display(Plot.plot({
  title: "Cumulative points by match",
  width,
  height: 380,
  y: { label: "Cumulative points", grid: true },
  x: { label: "Match number" },
  color: {
    domain: ["W", "D", "L"],
    range: ["#1a7f37", "#999999", "#c1121f"],
    legend: true
  },
  marks: [
    Plot.line(matches, {
      x: d => +d.match_number,
      y: d => +d.cumulative_points,
      stroke: "black",
      strokeWidth: 2
    }),

    Plot.dot(matches, {
      x: d => +d.match_number,
      y: d => +d.cumulative_points,
      fill: "result",
      r: 6,
      opacity: 0.92,
      tip: true,
      channels: {
        Match: d => d.match_number,
        Opponent: "opponent",
        Venue: "venue",
        Score: d => `${d.goals_for}-${d.goals_against}`,
        Formation: "formation_clean"
      }
    }),

    Plot.text(turningPoints, {
      x: "match_number",
      y: "cumulative_points",
      text: "note",
      dy: -24,
      fontSize: 12,
      lineWidth: 10
    }),

    Plot.ruleY([0])
  ]
}));


```

## 3. Possession was not the same as control

<div class="section-text">
Slot’s Liverpool could still have the ball, but possession did not always translate into dominance. A stable team uses possession to reduce chaos by creating chances, protecting the midfield, and limiting transitional danger. Liverpool often had one part of that equation, but not all three. This chart asks whether control was actually secure, or whether possession sometimes covered up a team that was still vulnerable underneath.
</div>

```js
const avgPossession = d3.mean(matches, d => +d.possession);

// Count how many matches share the same possession + goal difference
const matchesWithOverlap = matches.map(d => {
  const key = `${d.possession}-${d.goal_diff}`;

  const overlappingMatches = matches.filter(row =>
    +row.possession === +d.possession &&
    +row.goal_diff === +d.goal_diff
  );

  const overlapIndex = overlappingMatches.findIndex(row =>
    row.match_number === d.match_number
  );

  const overlapCount = overlappingMatches.length;

  return {
    ...d,
    overlap_key: key,
    overlap_index: overlapIndex,
    overlap_count: overlapCount,
    possession_jittered: +d.possession + (overlapIndex - (overlapCount - 1) / 2) * 0.9,
    goal_diff_jittered: +d.goal_diff + (overlapIndex - (overlapCount - 1) / 2) * 0.16
  };
});

display(Plot.plot({
  title: "Possession vs goal difference",
  subtitle: "Slight jitter separates matches with identical possession and goal-difference values.",
  width,
  height: 420,
  x: { label: "Possession %", grid: true },
  y: { label: "Goal difference", grid: true },
  color: {
    domain: ["W", "D", "L"],
    range: ["#1a7f37", "#999999", "#c1121f"],
    legend: true
  },
  marks: [
    Plot.ruleX([avgPossession], {
      stroke: "#999",
      strokeDasharray: "4 4"
    }),

    Plot.ruleY([0], {
      stroke: "#999",
      strokeDasharray: "4 4"
    }),

    Plot.dot(matchesWithOverlap, {
      x: "possession_jittered",
      y: "goal_diff_jittered",
      fill: "result",
      stroke: "white",
      strokeWidth: 1.5,
      r: 7,
      opacity: 0.92,
      tip: true,
      channels: {
        Match: d => d.match_number,
        Opponent: "opponent",
        Venue: "venue",
        Result: "result_label",
        Score: d => `${d.goals_for}-${d.goals_against}`,
        Possession: d => `${d.possession}%`,
        "Goal difference": "goal_diff",
        "Overlapping matches": "overlap_count",
        Formation: "formation_clean"
      }
    }),

    Plot.text([
      { possession: avgPossession + 3, goal_diff: 2.5, label: "Control became dominance" },
      { possession: avgPossession + 3, goal_diff: -2, label: "Possession without security" }
    ], {
      x: "possession",
      y: "goal_diff",
      text: "label",
      fontSize: 12,
      fill: "#555"
    })
  ]
}));
```
```js
/*
const heatmapMetrics = [
  "goals_for",
  "goals_against",
  "goal_diff",
  "points",
  "possession"
];

const matchMetricsFilteredRaw = matchMetrics.filter(d =>
  heatmapMetrics.includes(d.metric)
);

const matchMetricsFiltered = matchMetricsFilteredRaw.map(d => {
  const metricRows = matchMetricsFilteredRaw.filter(row => row.metric === d.metric);
  const minValue = d3.min(metricRows, row => row.value);
  const maxValue = d3.max(metricRows, row => row.value);

  return {
    ...d,
    normalized_value: maxValue !== minValue
      ? (d.value - minValue) / (maxValue - minValue)
      : 0.5,
    label_value: d.value
  };
});

display(Plot.plot({
  title: "Match-by-match metric matrix",
  width,
  height: 440,
  marginLeft: 120,
  x: {
    label: "Match number"
  },
  y: {
    label: null
  },
  color: {
    scheme: "YlOrRd",
    legend: true
  },
  marks: [
    Plot.cell(matchMetricsFiltered, {
      x: "match_number",
      y: "metric_label",
      fill: "normalized_value",
      stroke: "white",
      tip: true,
      channels: {
        Opponent: "opponent",
        Venue: "venue",
        Result: "result_label",
        "Value": "label_value"
      }
    })
  ]
}));
*/
```
## 4. Anfield protected the system

<div class="section-text">
The home and away split shows that Liverpool’s control was not equally stable everywhere. At Anfield, the team could still lean on rhythm, territory, and pressure. Away from home, the margins became thinner. That matters because a truly settled system usually travels. If Liverpool’s control depended too much on context, then the issue was not just finishing, injuries, or individual mistakes. It was a lack of tactical consistency.
</div>

```js
const homeAwayDumbbell = [
  "points",
  "goals_for",
  "goals_against",
  "goal_diff",
  "wins",
  "draws",
  "losses"
].map(metric => ({
  metric,
  Home: venueSummary.find(d => d.venue === "Home")[metric],
  Away: venueSummary.find(d => d.venue === "Away")[metric]
}));

display(Plot.plot({
  title: "Home vs away split",
  width,
  height: 360,
  marginLeft: 110,
  x: { grid: true },
  y: { label: null },
  color: {
    domain: ["Home", "Away"],
    range: ["#c1121f", "#333333"],
    legend: true
  },
  marks: [
    Plot.ruleX([0]),

    Plot.link(homeAwayDumbbell, {
      x1: "Home",
      x2: "Away",
      y1: "metric",
      y2: "metric",
      stroke: "#ccc",
      strokeWidth: 3
    }),

    Plot.dot(homeAwayDumbbell, {
      x: "Home",
      y: "metric",
      fill: () => "Home",
      r: 6,
      tip: true,
      channels: {
        Metric: "metric",
        Location: () => "Home",
        Value: "Home"
      }
    }),

    Plot.dot(homeAwayDumbbell, {
      x: "Away",
      y: "metric",
      fill: () => "Away",
      r: 6,
      tip: true,
      channels: {
        Metric: "metric",
        Location: () => "Away",
        Value: "Away"
      }
    })
  ]
}));
```
## 5. The attack spread out, but never became a hierarchy

```js
const attackingXgPlayers = playersWithXg
  .filter(d =>
    String(d.is_outfield).toLowerCase() === "true" &&
    +d.minutes > 500 &&
    d.xg_minutes > 500
  )
  .map(d => ({
    ...d,
    finishing_status:
      d.goals_minus_xg_per90 < -0.05 ? "Underperformed xG" :
      d.goals_minus_xg_per90 > 0.05 ? "Outperformed xG" :
      "Near xG"
  }));

display(Plot.plot({
  title: "Chance volume vs finishing: Liverpool lacked a reliable attacking hierarchy",
  subtitle: "xG per 90 shows chance volume. Goals minus xG per 90 shows finishing over/underperformance.",
  width,
  height: 480,
  x: {
    label: "Premier League xG per 90",
    grid: true
  },
  y: {
    label: "Goals per 90 minus xG per 90",
    grid: true
  },
  r: {
    range: [4, 18]
  },
  color: {
    domain: ["Underperformed xG", "Near xG", "Outperformed xG"],
    range: ["#c1121f", "#999999", "#1a7f37"],
    legend: true
  },
  marks: [
    Plot.ruleY([0], {
      stroke: "#777",
      strokeDasharray: "4 4"
    }),

    Plot.dot(attackingXgPlayers, {
      x: "xg_per90",
      y: "goals_minus_xg_per90",
      r: "minutes",
      fill: "finishing_status",
      stroke: "white",
      strokeWidth: 1.5,
      opacity: 0.85,
      tip: true,
      channels: {
        Player: "player",
        Position: "primary_position",
        "Minutes played": "minutes",
        "xG total": d => Math.round(d.xg_total * 100) / 100,
        "xG per 90": d => Math.round(d.xg_per90 * 100) / 100,
        "Goals per 90": d => Math.round(d.goals_per90_clean * 100) / 100,
        "Goals - xG per 90": d => Math.round(d.goals_minus_xg_per90 * 100) / 100
      }
    }),


    Plot.text(
      attackingXgPlayers.filter(d =>
        ["Mohamed Salah", "Hugo Ekitiké", "Cody Gakpo", "Florian Wirtz", "Alexander Isak", "Federico Chiesa"].includes(d.player)
      ),
      {
        x: "xg_per90",
        y: "goals_minus_xg_per90",
        text: "player",
        dx: 8,
        fontSize: 11
      }
    )
  ]
}));
```
<div class="section-text">
The xG data sharpens the lack-of-core argument. Liverpool had chance creation, but not enough repeatable attacking structure. Some players carried meaningful xG without consistently converting it into goals, while others were useful in smaller bursts but did not play enough minutes to become dependable reference points. The result was not a team without attacking talent. It was a team without a clear attacking order.
</div>

```js
/*
const selectedPlayerMetric = view(Inputs.select
(["progressive_carries_per90", "chance_carries_per90", "goals_assists_per90", "shots_on_target_per90","ended_with_chance","ended_with_shot"], {
  value: "goals_assists_per90",
  label: "Select player metric"
}));




display(Plot.plot({
  title: `Minutes vs ${selectedPlayerMetric.replaceAll("_", " ")}`,
  width,
  height: 440,
  x: { label: "Minutes", grid: true },
  y: { label: selectedPlayerMetric.replaceAll("_", " "), grid: true },
  r: { range: [3, 18] },
  color: { legend: true },
  marks: [
    Plot.dot(optaPlayerControl.filter(d => d.is_outfield === "True" && +d.opta_mins > 500), {
      x: d => +d.opta_mins,
      y: d => +d[selectedPlayerMetric],
      r: d => +d[selectedPlayerMetric],
      fill: "primary_position",
      opacity: 0.75,
      tip: true,
      channels: {
        Player: "player",
        Position: "primary_position",
        Minutes: "opta_mins",
        "Progressive carries": "progressive_total",
        "Chance carries": "ended_with_chance",
        "Goals + Assists /90": "goals_assists_per90",
        "Shots /90": "shots_per90"
      }
    })
  ]
}));
*/
```

## 6. The missing core: who progressed control and who protected it?

<div class="section-text">
The Opta data makes the control problem clearer. Control is not just possession. It is progression, recovery, and protection. A team needs players who can advance the ball, win it back, and stop attacks from becoming dangerous. Liverpool had players who could do each of those things, but the responsibilities were scattered. The players who moved Liverpool forward were not always the same players who protected transitions or stabilized the midfield. That separation made control harder to repeat from match to match.
</div>

```js
display(Plot.plot({
  title: "Progression vs protection: Liverpool’s control responsibilities were split",
  subtitle: "Progressive carries per 90 compared with possession wins per 90.",
  width,
  height: 460,
  x: {
    label: "Progressive carries per 90",
    grid: true
  },
  y: {
    label: "Possession won per 90",
    grid: true
  },
  r: {
    range: [4, 18]
  },
  color: {
    legend: true
  },
  marks: [
    Plot.ruleX([d3.mean(optaPlayerControl, d => d.progressive_carries_per90)], {
      stroke: "#999",
      strokeDasharray: "4 4"
    }),

    Plot.ruleY([d3.mean(optaPlayerControl, d => d.possession_won_per90)], {
      stroke: "#999",
      strokeDasharray: "4 4"
    }),

    Plot.dot(
      optaPlayerControl.filter(d => midfieldPositions.includes(d.primary_position) && +d.minutes > 500 && d.is_outfield === "True"),
      {
        x: "progressive_carries_per90",
        y: "possession_won_per90",
        r: "minutes",
        fill: "primary_position",
        opacity: 0.8,
        stroke: "white",
        tip: true,
        channels: {
          Player: "player",
          Position: "primary_position",
          Minutes: "minutes",
          "Progressive carries / 90": d => Math.round(d.progressive_carries_per90 * 100) / 100,
          "Possession won / 90": d => Math.round(d.possession_won_per90 * 100) / 100,
          "Tackles + Int / 90": d => Math.round(d.tackles_ints_per90 * 100) / 100,
          "Chances from carries": "ended_with_chance"
        }
      }
    ),

    Plot.text(
      optaPlayerControl.filter(d =>
        ["Ryan Gravenberch", "Dominik Szoboszlai", "Alexis Mac Allister", "Florian Wirtz", "Mohamed Salah"].includes(d.player)
      ),
      {
        x: "progressive_carries_per90",
        y: "possession_won_per90",
        text: "player",
        dx: 8,
        fontSize: 11
      }
    )
  ]
}));
```
<div class="section-text">
By this point, the question is not just who scored or assisted. Liverpool’s problem was bigger than output. The team searched for consistency through control, but control requires a core of players who are available, trusted, positionally stable, and able to connect possession with defensive security. This chart compares each core player to the median player in his position group. Red does not mean the player was bad overall. It means that, within this squad, he fell below his positional baseline in that category.
</div>

```js
const midfieldPositions = ["MF"];



const corePlayerPool = optaPlayerControl
  .filter(d => String(d.is_outfield).toLowerCase() === "true" && +d.minutes > 500)
  .map(d => ({
    ...d,
    minutes: safeNumber(d.minutes),
    minutes_pct: safeNumber(d.minutes_pct),
    goals_assists_per90: safeNumber(d.goals_assists_per90),
    shots_on_target_per90: safeNumber(d.shots_on_target_per90),
    progressive_carries_per90: safeNumber(d.progressive_carries_per90),
    chance_carries_per90: safeNumber(d.chance_carries_per90),
    possession_won_per90: safeNumber(d.possession_won_per90),
    tackles_ints_per90: safeNumber(d.tackles_ints_per90),
    blocks_clearances_per90: safeNumber(d.blocks_clearances_per90),
    team_goal_diff_per90_on_pitch: safeNumber(d.team_goal_diff_per90_on_pitch)
  }));

const underwhelmingMetrics = [
  {
    key: "availability",
    label: "Availability",
    value: d => d.minutes_pct
  },
  {
    key: "output",
    label: "G+A /90",
    value: d => d.goals_assists_per90
  },
  {
    key: "shot_threat",
    label: "SOT /90",
    value: d => d.shots_on_target_per90
  },
  {
    key: "progression",
    label: "Prog Carries /90",
    value: d => d.progressive_carries_per90
  },
  {
    key: "chance_carrying",
    label: "Chance Carries /90",
    value: d => d.chance_carries_per90
  },
  {
    key: "recovery",
    label: "Poss Won /90",
    value: d => d.possession_won_per90
  },
  {
    key: "defensive_activity",
    label: "Tkl+Int /90",
    value: d => d.tackles_ints_per90
  },
  {
    key: "team_impact",
    label: "Team GD /90",
    value: d => d.team_goal_diff_per90_on_pitch
  }
];

function positionMedian(metric, position) {
  const positionRows = corePlayerPool.filter(d => d.primary_position === position);
  return d3.median(positionRows, d => metric.value(d)) 
    ?? d3.median(corePlayerPool, d => metric.value(d)) 
    ?? 0;
}

function positionDeviation(metric, position) {
  const positionRows = corePlayerPool.filter(d => d.primary_position === position);
  return d3.deviation(positionRows, d => metric.value(d)) || 1;
}

const playerUnderperformanceLong = corePlayerPool.flatMap(player =>
  underwhelmingMetrics.map(metric => {
    const raw = metric.value(player);
    const median = positionMedian(metric, player.primary_position);
    const deviation = positionDeviation(metric, player.primary_position);
    const z = (raw - median) / deviation;

    return {
      player: player.player,
      position: player.primary_position,
      metric: metric.key,
      metric_label: metric.label,
      raw_value: raw,
      position_median: median,
      z_score: z,
      status:
        z < -0.35 ? "Below positional median" :
        z > 0.35 ? "Above positional median" :
        "Near positional median"
    };
  })
);

const playerUnderperformanceSummary = d3.rollups(
  playerUnderperformanceLong,
  rows => ({
    below_count: rows.filter(d => d.z_score < -0.35).length,
    avg_z: d3.mean(rows, d => d.z_score)
  }),
  d => d.player
).map(([player, values]) => ({
  player,
  below_count: values.below_count,
  avg_z: values.avg_z
}));

const playerOrder = playerUnderperformanceSummary
  .sort((a, b) => d3.descending(a.below_count, b.below_count) || d3.ascending(a.avg_z, b.avg_z))
  .map(d => d.player);

  display(Plot.plot({
  title: "Where Liverpool’s core players fell below their positional baseline",
  subtitle: "Each cell compares a player to the median player in his position group. Red marks underwhelming areas.",
  width,
  height: 560,
  marginLeft: 150,
  marginBottom: 90,
  x: {
    label: null,
    tickRotate: -35
  },
  y: {
    label: null,
    domain: playerOrder
  },
  color: {
    domain: ["Below positional median", "Near positional median", "Above positional median"],
    range: ["#c1121f", "#d9d9d9", "#1a7f37"],
    legend: true
  },
  marks: [
    Plot.cell(playerUnderperformanceLong, {
      x: "metric_label",
      y: "player",
      fill: "status",
      stroke: "white",
      tip: true,
      channels: {
        Player: "player",
        Position: "position",
        Metric: "metric_label",
        "Player value": d => Math.round(d.raw_value * 100) / 100,
        "Position median": d => Math.round(d.position_median * 100) / 100,
        "Z-score": d => Math.round(d.z_score * 100) / 100,
        Status: "status"
      }
    })
  ]
}));
```

<div class="section-text">
The player-level data makes the lack of core clearer. Liverpool did not simply need more names contributing. They needed the same players to repeatedly connect availability, progression, recovery, and team impact. Instead, the squad looks fragmented. Some players offered attacking output without enough control value. Others helped defensively without consistently progressing the team. Several players were useful in one category but underwhelming in another. Liverpool had pieces, but not enough complete profiles forming a reliable spine.
</div>



## The Slot question

<div class="section-text">
So, is this on Arne Slot? The data does not support a simple answer. Liverpool were not a team with no ideas. They still tried to control matches through possession, still generated attacking output, and still had enough individual quality to win games. But the data also does not fully let Slot off the hook. The problem was that control rarely became consistency.

Liverpool did not settle into a rhythm, did not consistently protect matches away from home, and did not turn redistributed attacking responsibility into a clear hierarchy. Slot’s challenge was not only to replace players like Trent Alexander-Arnold and Luis Díaz. It was to replace the structure they gave the team. This dashboard suggests that he found partial solutions, but not a repeatable one.
</div>


## Data note

This dashboard uses cleaned FBref match-log and player-stat data alongside Opta Analyst player-level passing/carrying and defending data. The FBref match table supports result, venue, possession, formation, and scoring analysis. The Opta tables add deeper player-level context around progression, possession wins, duels, blocks, clearances, and carries that ended in attacking actions. Because these sources use different units, several player charts convert totals to per-90 rates or normalize values within each metric.

## Conclusion

Liverpool’s 2025–26 season was not simply a failure of talent. It was a failed search for consistency through control. The team could still produce goals, hold possession, and find useful contributions across the squad, but those qualities did not consistently appear in the same matches or through the same player profiles.

Compared with the clearer Salah–Trent structure, this version of Liverpool looked flatter, more distributed, more flexible, but less stable. The issue was not only that Liverpool lost creative reference points. It was that the new system never fully replaced the core they created. The midfield did not consistently anchor matches, availability disrupted rhythm, and attacking responsibility spread across players without becoming a reliable hierarchy.

My answer to the Slot question is this: **Slot was not the only reason Liverpool became inconsistent, but his system did not solve the inconsistency either.** Injuries, availability, squad transition, and the loss of the old creative axis all matter. Still, a manager’s job is to turn those pieces into a stable core. In 2025–26, Liverpool often looked like a team searching for that core rather than playing from one.

