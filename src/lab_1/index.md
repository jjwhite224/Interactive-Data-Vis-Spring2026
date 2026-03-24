---
title: "Lab 1: Passing Pollinators"
toc: true
---

```js
import * as Plot from "npm:@observablehq/plot";
import * as d3 from "npm:d3";
```
```js
const pollinators = FileAttachment("./data/pollinator_activity_data.csv").csv({typed: true});
```
Q 1.1
```js
Plot.plot({title:"Average Body Mass by Pollinator Species",
    x: {label: "Pollinator Species"},
    y:{label: "Average Body Mass(g)"},
    marks: [
        Plot.barY(
            pollinators,
            Plot.groupX({y:"mean"},{x:"pollinator_species",y:"avg_body_mass_g", fill:"pollinator_species"})
        ),
        Plot.ruleY([0])
    ]
})
```
Q 1.2
```js
Plot.plot({title:"Average Wing Span by Pollinator Species",
    x: {label: "Pollinator Species"},
    y:{label: "Average Wing Span (mm)"},
    marks: [
        Plot.barY(
            pollinators,
            Plot.groupX({y:"mean"},{x:"pollinator_species",y:"avg_wing_span_mm", fill:"pollinator_species"})
        ),
        Plot.ruleY([0])
    ]
})
```


Q 2.1
```js
Plot.plot({title:"Ideal Weather Conditions for Pollination",
    x: {label: "Weather Condition"},
    y:{label: "Average Visit Count"},
    marks: [
        Plot.barY(
            pollinators,
            Plot.groupX({y:"mean"},{x:"weather_condition",y:"visit_count", fill:"weather_condition"})
        ),
        Plot.ruleY([0])
    ]
})
```
Q 2.2
```js
const filteredVisits = pollinators.filter(d => d.visit_count >= 10);

display(Plot.plot({title:"Pollinator visits vs Temp",
    x: {label: "Temperature (°C)"},
    y:{label: "Visit Count"},
    marks: [
        Plot.dot(
            filteredVisits,
            {x:"temperature",y:"visit_count", fill:"weather_condition",  title: d => `Flower: ${d.flower_species}
Nectar Production: ${d.nectar_production}
Weather: ${d.weather_condition}
Temperature: ${d.temperature}°C
Visits: ${d.visit_count}`,
             tip:true
            })  
    ]
})
)
```

Q 3
```js
Plot.plot({title:"Flowers with highest Nectar Production",
    x: {label: "Flower"},
    y:{label: "Nectar Prod."},
    marks: [
        Plot.barY(
            pollinators,
            Plot.groupX({y:"mean"},{x:"flower_species",y:"nectar_production", fill:"flower_species"})
        ),
        Plot.ruleY([0])
    ]
})
```