---
title: "Lab 2: Subway Staffing"
toc: true
---


```js
const incidents = await FileAttachment("./data/incidents.csv").csv({ typed: true });
const local_events = await FileAttachment("./data/local_events.csv").csv({ typed: true });
const upcoming_events = await FileAttachment("./data/upcoming_events.csv").csv({ typed: true });
const ridership = await FileAttachment("./data/ridership.csv").csv({ typed: true });
```

```js
const currentStaffing = {
  "Times Sq-42 St": 19,
  "Grand Central-42 St": 18,
  "34 St-Penn Station": 15,
  "14 St-Union Sq": 4,
  "Fulton St": 17,
  "42 St-Port Authority": 14,
  "Herald Sq-34 St": 15,
  "Canal St": 4,
  "59 St-Columbus Circle": 6,
  "125 St": 7,
  "96 St": 19,
  "86 St": 19,
  "72 St": 10,
  "66 St-Lincoln Center": 15,
  "50 St": 20,
  "28 St": 13,
  "23 St": 8,
  "Christopher St": 15,
  "Houston St": 18,
  "Spring St": 12,
  "Chambers St": 18,
  "Wall St": 9,
  "Bowling Green": 6,
  "West 4 St-Wash Sq": 4,
  "Astor Pl": 7
};
```
```js
const fareIncreaseDate = new Date("2025-07-15");

const ridershipWithEvent = ridership.map(d => {
  const eventMatch = local_events.find(
    e => +e.date === +d.date && e.nearby_station === d.station
  );
  return {
    ...d,
    event_name: eventMatch?.event_name ?? null,
    estimated_attendance: eventMatch?.estimated_attendance ?? 0,
    is_event_day: eventMatch ? "Event day" : "No event"
  };
});

const dailyRidership = Array.from(
  d3.rollup(
    ridership,
    v => d3.sum(v, d => d.entrances),
    d => +d.date
  ),
  ([date, entrances]) => ({ date: new Date(date), entrances })
).sort((a, b) => d3.ascending(a.date, b.date));

const eventRidershipSummary = Array.from(
  d3.rollup(
    ridershipWithEvent,
    v => d3.mean(v, d => d.entrances),
    d => d.is_event_day
  ),
  ([type, avg_entrances]) => ({ type, avg_entrances })
);

const responseByStation = Array.from(
  d3.rollup(
    incidents,
    v => d3.mean(v, d => d.response_time_minutes),
    d => d.station
  ),
  ([station, avg_response]) => ({ station, avg_response })
).sort((a, b) => d3.descending(a.avg_response, b.avg_response));

const staffingNeed = Array.from(
  d3.rollup(
    upcoming_events,
    v => d3.sum(v, d => d.expected_attendance),
    d => d.nearby_station
  ),
  ([station, total_attendance]) => ({
    station,
    total_attendance,
    current_staff: currentStaffing[station],
    attendance_per_staff: total_attendance / currentStaffing[station]
  })
).sort((a, b) => d3.descending(a.attendance_per_staff, b.attendance_per_staff));

const topThreeStations = staffingNeed.slice(0, 3);
```

Q1: How did local events impact ridership in summer 2025? What effect did the July 15th fare increase have?

Stations on event days consistently see higher ridership compared to normal days. The bar chart shows that average entrances on event days are significantly higher, confirming that local events drive subway traffic.

Looking at the time series, ridership trends remain relatively stable throughout the summer, but there is a noticeable shift around July 15 when the fare increase was introduced. After this date, overall ridership appears to slightly decline, suggesting that the price increase may have had a dampening effect on subway usage.

Overall, events increase ridership in the short term, while the fare increase may reduce it slightly over time.

```js
Plot.plot({
  title: "Daily Subway Entrances in Summer 2025",
  width: 800,
  height: 400,
  x: { label: "Date" },
  y: { label: "Total entrances" },
  marks: [
    Plot.line(dailyRidership, {
      x: "date",
      y: "entrances",
      stroke: "#2563eb", // blue
      strokeWidth: 2
    }),

    Plot.ruleX([fareIncreaseDate], {
      stroke: "#dc2626", // red
      strokeWidth: 2,
      strokeDasharray: "4,4"
    }),

    Plot.text(
      [{
        date: fareIncreaseDate,
        entrances: d3.max(dailyRidership, d => d.entrances),
        label: "Fare Increase"
      }],
      {
        x: "date",
        y: "entrances",
        text: "label",
        dy: -10,
        fill: "#dc2626"
      }
    )
  ]
})

```
```js
Plot.plot({
  title: "Average Entrances: Event vs Non-Event Days",
  width: 500,
  height: 400,
  marginLeft: 180,
marginRight: 40,
marginTop: 40,
marginBottom: 40,
  x: { label: "Average entrances" },
  y: { label: null },
  color: {
    domain: ["Event day", "No event"],
    range: ["#f97316", "#9ca3af"] // orange vs gray
  },
  marks: [
    Plot.barX(eventRidershipSummary, {
      x: "avg_entrances",
      y: "type",
      fill: "type",
      tip: true
    }),
    Plot.text(eventRidershipSummary, {
      x: "avg_entrances",
      y: "type",
      text: d => Math.round(d.avg_entrances).toLocaleString(),
      dx: 8
    })
  ]
})
```


Q2: How do the stations compare when it comes to response time? Which are the best, which are the worst?

Response times vary noticeably across stations. Stations like Fulton St, Houston St, and Times Sq-42 St have the fastest response times, suggesting more efficient operations or better staffing coverage.

On the other end, stations such as 59 St-Columbus Circle, West 4 St-Wash Sq, and Canal St have significantly slower response times. These stations may already be experiencing operational strain, which could become worse during high-traffic periods like the summer event season.

The difference between the best and worst stations highlights potential inequalities in staffing efficiency across the system.
```js
Plot.plot({
  title: "Average Incident Response Time by Station",
  width: 800,
  height: 600,
  marginLeft: 180,
  marginRight: 40,
  marginTop: 40,
  marginBottom: 40,

  x: { label: "Avg response time (minutes)" },
  y: { label: null },

  color: {
    domain: d3.extent(responseByStation, d => d.avg_response),
    scheme: "reds"
  },

  marks: [
    Plot.barX(responseByStation, {
      x: "avg_response",
      y: "station",
      fill: "avg_response",
      sort: { y: "x", reverse: true },
      tip: true
    }),
    Plot.ruleX(
      [d3.mean(responseByStation, d => d.avg_response)],
      {
        stroke: "black",
        strokeDasharray: "4,4"
      }
    )
  ]
})
```
Q3: Which three stations need the most staffing help for next summer based on the 2026 event calendar?

To determine staffing needs for summer 2026, I compared each station’s projected event attendance with its current staffing levels. I calculated an “attendance per staff member” metric to estimate how much demand each staff member would need to handle.

Stations with higher values are under more pressure, meaning each staff member would be responsible for a larger number of riders.

Based on this, the three stations that need the most staffing support are:

Canal St
34 St-Penn Station
23 St

These stations combine high projected event traffic with relatively low staffing levels, making them the most likely to become overwhelmed during peak period

```js
Plot.plot({
  title: "Event Demand per Staff Member (2026)",
  width: 800,
  height: 600,
  marginLeft: 180,
marginRight: 40,
marginTop: 40,
marginBottom: 40,
  color: {
    scheme: "purples"
  },
  marks: [
    Plot.barX(staffingNeed, {
      x: "attendance_per_staff",
      y: "station",
      fill: "attendance_per_staff",
      sort: { y: "x", reverse: true },
      tip: true
    }),

    Plot.text(topThreeStations, {
      x: "attendance_per_staff",
      y: "station",
      text: "station",
      dx: 8,
      fill: "black",
      fontWeight: "bold"
    })
  ]
  
})
```
Bonus: If I had to prioritize one station, I would choose Canal St. It has one of the lowest staffing levels but one of the highest projected event demands, meaning each staff member would be under the most pressure. Increasing staffing here would likely have the biggest impact on improving system performance during the summer.