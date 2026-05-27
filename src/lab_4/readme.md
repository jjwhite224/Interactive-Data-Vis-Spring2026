# Liverpool Opta CSVs for Observable

These files were cleaned from the uploaded Opta Analyst HTML exports.

## Files

- `liverpool_opta_passing_carrying.csv` — player carrying/progression totals and carry outcomes.
- `liverpool_opta_defending.csv` — player defensive actions and duel rates.
- `liverpool_opta_player_control_merged.csv` — merged player-level control dataset using the existing FBref player table plus Opta metrics. This is the easiest file to use in Observable.
- `README_data_dictionary.csv` — file-level data dictionary.

## Recommended Observable loading code

```js
const optaPassing = await FileAttachment("data/liverpool_opta_passing_carrying.csv").csv({ typed: true });
const optaDefending = await FileAttachment("data/liverpool_opta_defending.csv").csv({ typed: true });
const optaPlayerControl = await FileAttachment("data/liverpool_opta_player_control_merged.csv").csv({ typed: true });
```

## Notes

- Player names were harmonized for merging: `Hugo Ekitiké` → `Hugo Ekitike`, and `Andy Robertson` → `Andrew Robertson`.
- Percentage fields such as duel win rates are stored as numeric percentages, e.g. `56.1`, not `0.561`.
- Per-90 columns are based on `control_minutes / 90`, using FBref minutes where available.
