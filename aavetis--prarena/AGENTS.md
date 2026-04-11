always use our venv called venv

main way to run the end to end workflow:
python3 collect_data.py # this gets new data into data.csv
python3 generate_chart.py # this generates the chart image, renders the index template, the readme template.
python3 -m http.server 8000 # this runs our web server so we can see the index page.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aavetis)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/aavetis)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
