# Bend Monthly Events Skill

A Codex skill for researching Bend, Oregon events in parallel and producing a portable HTML report. It covers city event listings, independent venue calendars, breweries, and nearby towns.

Each source gets a fresh, temporary research agent for each run. The parent coordinator launches workers in parallel within the session's available capacity, collects their results, and closes/releases them where the host supports it. Completed workers are not reused for another source or another run. The coordinator verifies direct sources, merges duplicates, and records source coverage. If the host cannot support fresh workers or parallel execution, that limitation must be reported explicitly.

The agent files are included:

- [Coordinator lifecycle](skills/bend-weekend-event-list/agents/coordinator.md): spawning, queueing, result collection, and cleanup.
- [Source researcher role](skills/bend-weekend-event-list/agents/source-researcher.md): one-source instructions and JSON contract.
- [Native Codex agent definition](.codex/agents/bend-source-researcher.toml): the named `bend_source_researcher` role when running Codex in this checkout.
- [Source assignments](skills/bend-weekend-event-list/references/sources.json): the 18 sources, including Cascades Theatre.

`agents/openai.yaml` is skill UI metadata; it does not launch agents. The skill's coordinator instructions explicitly request spawning through the host's agent tools. The role definitions are reusable, while their research worker instances are temporary. No daemon or scheduled task is installed.

## Install

Python 3.10 or newer is required for the report renderer. Research requires an assistant with web access; parallel research also requires agent delegation tools. No API keys or Python packages are needed for HTML generation.

```bash
git clone https://github.com/srogerf/bend-monthly-events.git
mkdir -p ~/.codex/skills
cp -R bend-monthly-events/skills/bend-weekend-event-list ~/.codex/skills/
```

If you already have a skill with that name, compare or back up the existing directory before replacing it. Restart or start a new Codex session if needed to discover the installed skill.

When running Codex inside this repository, the native role is in `.codex/agents/`. Installing only the skill folder still includes both Markdown role files, so the coordinator can spawn generic subagents with the same instructions. To make the native role available in other projects, optionally copy `.codex/agents/bend-source-researcher.toml` into `~/.codex/agents/` after checking for an existing file with that name. The skill does not change global agent settings automatically.

Example request:

> Use $bend-weekend-event-list to find events in Bend this Friday through Sunday. Include nearby towns separately and save an HTML report.

Without a specified period, research defaults to the current month. Weekend and other explicit date ranges work too. Workers return one source result each. A local `run.json` records actual worker IDs and outcomes so the delegation can be reviewed; this execution manifest is not part of the event report or published to GitHub.

## HTML reports

```bash
python3 skills/bend-weekend-event-list/scripts/render_report.py \
  examples/events.sample.json /tmp/bend-events-sample.html
```

Open the resulting file in a browser. A pre-generated [sample report](examples/report.sample.html) is included; its events are fictional and clearly labeled. Reports default to a month calendar, with category checkboxes, period/custom date controls, and a list view. Advance-booking badges and a filter highlight verified requirements or recommendations, including booking links and deadlines where known. Reports also include a nearby section, direct links, prices and restrictions, verification notes, and source timestamps.

The filters use events already included in the report. Dates outside the researched range need a fresh research run; an empty filtered view is not evidence that there are no events. Styling and interaction are embedded for mobile and print; there are no external scripts, fonts, assets, or build tools. Without JavaScript, event listings remain readable.

When you choose to publish, upload the HTML file to any static host, or use it as `index.html` on GitHub Pages. Publishing is separate from generating a local report. This repository does not enable a hosted site automatically.

Source assignments are defined in [sources.json](skills/bend-weekend-event-list/references/sources.json), including Cascades Theatre / Cascades Theatrical Company. Theatre agents check individual performance dates rather than expanding a production's date range into daily shows. The shared [worker instructions](skills/bend-weekend-event-list/agents/source-researcher.md) and [report schema](skills/bend-weekend-event-list/references/report-schema.md) keep results consistent. Source calendar paths can change; research agents should follow current official links.

## Validation

```bash
python3 -m unittest discover -s tests -v
```

This checks report behavior and data handling, not live event availability. Research always refreshes dates, schedules, prices, and availability. Coverage is never claimed to be exhaustive.

Licensed under MIT. Independent community tooling; not affiliated with the listed venues or event publishers.
