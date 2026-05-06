---
title: "Lab 3: Mayoral Mystery"
toc: false
---
# Mayoral Mystery: A Deep Dive into the Eleciton

This dashboard analyzes the campaign’s 2024 election performance across NYC community districts. I focus on three questions: where the candidate won or lost, how support varied by income level, and whether campaign outreach and issue alignment point toward a strategy for the next run.
<!-- Import Data -->
```js
const nyc = await FileAttachment("data/nyc.json").json();
const results = await FileAttachment("data/election_results.csv").csv({ typed: true });
const survey = await FileAttachment("data/survey_responses.csv").csv({ typed: true });
const events = await FileAttachment("data/campaign_events.csv").csv({ typed: true });
```


```js
// The nyc file is saved in data as a topoJSON instead of a geoJSON. Thats primarily for size reasons -- it saves us 3MB of data. For Plot to render it, we have to convert it back to its geoJSON feature collection. 
const nycdistricts = topojson.feature(nyc, nyc.objects.districts)
const electionClean = results.map(d => ({
  ...d,
  boro_cd: +d.boro_cd,
  total_votes: d.votes_candidate + d.votes_opponent,
  candidate_share_pct: 100 * d.votes_candidate / (d.votes_candidate + d.votes_opponent),
  margin_pct: 100 * (d.votes_candidate - d.votes_opponent) / (d.votes_candidate + d.votes_opponent),doors_per_1000: 1000 * d.gotv_doors_knocked / d.total_registered_voters,
  winner: d.votes_candidate > d.votes_opponent ? "Candidate" : "Opponent"
}));
const eventsClean = events.map(d => ({
  ...d,
  latitude: +d.latitude,
  longitude: +d.longitude,
  estimated_attendance: +d.estimated_attendance
}));
const resultByDistrict = new Map(
  electionClean.map(d => [d.boro_cd, d])
);

// Literal CSS colors per event type so dots don't share the plot's linear `color` scale (choropleth margins).
const eventTypeFill = {
  "Canvassing Kickoff": "#59a14f",
  "Community Meeting": "#4e79a7",
  Rally: "#e15759",
  Roundtable: "#edc949",
  "Town Hall": "#b07aa1",
  "Volunteer Training": "#ff9da7"
};
const surveyLong = survey.flatMap(d => [
  {
    issue: "Affordable housing",
    alignment: d.affordable_housing_alignment
  },
  {
    issue: "Public transit",
    alignment: d.public_transit_alignment
  },
  {
    issue: "Childcare support",
    alignment: d.childcare_support_alignment
  },
  {
    issue: "Small business taxes",
    alignment: d.small_business_tax_alignment
  },
  {
    issue: "Police reform",
    alignment: d.police_reform_alignment
  }
]);

```
## 1. District Wins and Campaign Events
```js
// Simple rendering of the NYC districts topoJSON
Plot.plot({
  title: "District winner by NYC community district, with campaign events",
  width: 850,
  height: 650,
  projection: {
    type: "mercator",
    domain: nycdistricts
    
  },
 color: {
  legend: true,
  label: "District winner",
  domain: ["Candidate", "Opponent"],
  range: ["#4e79a7", "#e15759"]
},
  r: {
    range: [2, 12],
    label: "Estimated event attendance"
  },
  marks: [
    Plot.geo(nycdistricts, {
     fill: d => resultByDistrict.get(+d.properties.BoroCD)?.winner,
      stroke: "white",
      strokeWidth: 0.75,
      title: d => {
        const result = resultByDistrict.get(+d.properties.BoroCD);

        return result
          ? `District ${result.boro_cd}
Candidate share: ${result.candidate_share_pct.toFixed(1)}%
Margin: ${result.margin_pct.toFixed(1)} pts`
          : `District ${d.properties.BoroCD}
No election data`;
      }
    }),

    Plot.dot(eventsClean, {
      x: "longitude",
      y: "latitude",
      r: "estimated_attendance",
      fill: (d) => eventTypeFill[d.event_type] ?? "#6c757d",
      stroke: "black",
      strokeWidth: 0.5,
      fillOpacity: 0.65,
      tip: true,
      title: d => `${d.event_type}
District ${d.boro_cd}
Attendance: ${d.estimated_attendance}`
    })
  ]
})

```

This map shows where the candidate won and lost across NYC community districts. Districts are colored by winner: blue districts were won by the candidate, while red districts were won by the opponent. The dots show campaign event locations, with larger dots representing higher estimated attendance. The map suggests support for the candidate was uneven across the city, with campaign activity concentrated in some areas more than others. 

## 2. Candidate Support by Income Category

```js
Plot.plot({
  title: "Average candidate vote share by income category",
  width: 760,
  height: 400,
  marginLeft: 80,
  x: {
    label: "Average candidate vote share (%)",
    grid: true,
    domain: [0, 70]
  },
  y: {
    label: null
  },
  color: {
    legend: false
  },
  marks: [
    Plot.ruleX([50], {
      strokeDasharray: "4,4"
    }),

    Plot.barX(
      electionClean,
      Plot.groupY(
        {x: "mean"},
        {
          y: "income_category",
          x: "candidate_share_pct",
          fill: "income_category",
          tip: true
        }
      )
    )
  ]
})
```

This chart groups districts by income category, and shows that the candidate performed best in low-income districts and weakest in high income districts. This indicates that income level was one of the strongest divides in the election.

## 3. GOTV Effort and Vote Share

```js
Plot.plot({
  title: "GOTV door-knocking and candidate vote share",
  subtitle: "Each dot is one district. Doors are normalized per 1,000 registered voters.",
  width: 760,
  height: 450,
  marginLeft: 60,
  x: {
    label: "Doors knocked per 1,000 registered voters",
    grid: true
  },
  y: {
    label: "Candidate vote share (%)",
    grid: true,
    domain: [20, 70]
  },
  color: {
    legend: true
  },
  marks: [
    Plot.ruleY([50], {
      strokeDasharray: "4,4"
    }),

    Plot.dot(electionClean, {
      x: "doors_per_1000",
      y: "candidate_share_pct",
      fill: "income_category",
      r: 5,
      tip: true
    })
  ]
})
```

This scatterplot compares GOTV effort with candidate performance. Each dot is one district. I normalized door-knocking by registered voters so that districts with different population sizes could be compared more fairly. Districts with more doors knocked per 1,000 voters tended to have higher candidate vote share, but this does not prove that door-knocking caused the higher support. The pattern may also reflect the campaign focusing on districts where the candidate was already more competitive, such as lower income districts. 

## 4. Survey Alignment by Campaign Issue

```js
Plot.plot({
  title: "Survey respondents aligned most with transit and housing, least with police reform",
  width: 760,
  height: 420,
  marginLeft: 150,
  x: {
    label: "alignment",
    grid: true,
    domain: [0, 5]
  },
  y: {
    label: null
  },
  color: {
    legend: false
  },
  marks: [
    Plot.ruleX([3], {
      strokeDasharray: "4,4"
    }),

    Plot.barX(
      surveyLong,
      Plot.groupY(
        {x: "mean"},
        {
          y: "issue",
          x: "alignment",
          fill: "issue",
          tip: true
        }
      )
    )
  ]
})
```

This chart averages the responses to the survey alignment score for each campaign issue.Respondents showed the strongest alignment with public transit and affordable housing, suggesting that the campaign’s affordability-related message was one of its stronger areas. Police reform had weaker alignment than the other issues, which suggests that the campaign may need to clarify or adjust that part of its message in a future run.

## Recommendation

The candidate’s strongest base was in low-income districts, where vote share was highest and GOTV activity appeared connected to stronger performance. For a future run, the campaign should keep invest more in these areas, but it also needs a stronger strategy for middle- and high-income districts where the opponent performed better. The survey results suggest that transit and housing should stay central to the message, while police reform may need clearer framing because it had the weakest average alignment.