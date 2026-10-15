# Coordinator: temporary parallel research

Run this workflow in the parent agent when the Bend events skill is invoked. The
source registry defines assignments; it does not define running background agents.
Each research run creates fresh worker instances, with one source per instance.

1. Resolve the date range and Pacific timezone. Create a unique run directory
   outside the repository, normally `/tmp/bend-events-<run-id>/`, with a
   `source-results/` directory. Read `references/sources.json`, the worker prompt
   in `agents/source-researcher.md`, and `references/report-schema.md`.
2. Inspect the host's available delegation tools and concurrent worker capacity.
   Reserve room for the coordinator. Keep remaining source assignments queued;
   do not assume the source count equals the number of simultaneous workers.
3. For each source, **spawn a fresh subagent** for this run and assignment. Use
   the named `bend_source_researcher` role when the host has loaded the bundled
   native configuration; otherwise use a generic subagent with the same worker
   prompt. An existing idle agent from earlier research is not a fresh worker.
4. Supply only the source registry entry, requested dates and geography, timezone,
   worker instructions, schema, and a unique result path. Prefer a fresh context
   over copying unrelated parent conversation. Name the task with the run ID and
   source ID. Do not give a worker another source after it finishes.
5. Run workers concurrently up to the available capacity. Wait for each result,
   inspect its JSON and coverage record, then close/release the worker with the
   host's lifecycle tools when available. Dispatch the next queued source as
   capacity becomes available. A host that retains completed threads may retain
   them in a finished state; do not represent that as a live background service
   or silently reuse the thread for another source.
6. Record the run locally in `run.json`: run ID, requested dates, delegation mode,
   and each source's actual agent/task ID, result path, and terminal outcome.
   Do not invent agent IDs. Keep this execution manifest out of the public event
   ledger and out of Git. Explain any host limitation that prevents fresh workers
   or parallel execution. Do not substitute sequential research unless the user
   changes the requested workflow; do not claim a sequential check ran in parallel.
7. A blocked website returns partial/unavailable coverage. A failed worker is a
   separate execution failure: report it or retry once with a new worker, retaining
   the failed attempt in the run manifest. Collect an outcome for every source.
8. Merge and verify the results in the parent, preserving separate performances,
   direct links, booking evidence, and uncertainty. Generate the local HTML and
   ledger. Ensure workers have finished or have been stopped before returning.
   Research and local output never authorize GitHub publication or site hosting.

The reusable role files remain installed; worker instances are temporary. This
workflow uses the host's agent tools. Markdown files and `openai.yaml` do not
themselves start processes or create scheduled jobs.
