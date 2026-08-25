# Kailo MCP Server

Connect your AI assistant to your own running data.

Kailo is a hosted [Model Context Protocol](https://modelcontextprotocol.io) server that gives Claude, Cursor, or any MCP client access to your own training history — activities, streams, race predictions — and then lets it build routes, author structured workouts, and write whole training plans straight back to your Garmin watch.

```
https://kailo.fit/mcp
```

| | |
| --- | --- |
| **Endpoint** | `https://kailo.fit/mcp` |
| **Transport** | Streamable HTTP |
| **Auth** | OAuth 2.1 — PKCE + Dynamic Client Registration |
| **Tools** | 61 public |
| **Account** | Free at [kailo.fit](https://kailo.fit) |

---

## Quick start

Kailo supports OAuth Dynamic Client Registration, so there is no API key to create and nothing to paste. Add the URL and your client walks you through sign-in.

### Claude (web, desktop, mobile)

Settings → **Connectors** → **Add custom connector** → paste `https://kailo.fit/mcp`.

### Claude Code

```bash
claude mcp add --transport http kailo https://kailo.fit/mcp
```

### Cursor / VS Code / any `mcp.json` client

```json
{
  "mcpServers": {
    "kailo": {
      "type": "http",
      "url": "https://kailo.fit/mcp"
    }
  }
}
```

### Anything else

Point your client at `https://kailo.fit/mcp`. Discovery documents are public:

| Document | URL |
| --- | --- |
| Server card | [`/.well-known/mcp.json`](https://kailo.fit/.well-known/mcp.json) |
| Protected Resource Metadata ([RFC 9728](https://datatracker.ietf.org/doc/html/rfc9728)) | [`/.well-known/oauth-protected-resource`](https://kailo.fit/.well-known/oauth-protected-resource) |
| Authorization Server Metadata ([RFC 8414](https://datatracker.ietf.org/doc/html/rfc8414)) | [`/.well-known/oauth-authorization-server`](https://kailo.fit/.well-known/oauth-authorization-server) |

---

## What you can ask

> *"How has my running been going this month?"*

> *"Compare my training volume this year vs last year."*

> *"Show me the heart rate curve from Saturday's long run — did I fade?"*

> *"Build me a 10 mile route from the Marina that passes a coffee shop, and send it to my Garmin."*

> *"Write me a 16 week marathon plan off my current fitness and put it in my account."*

---

## Tools

61 tools are available to every connected account. Tools marked **[Pro]** require a Kailo Pro subscription; everything else is free.

### Activity data & predictions (9)

| Tool | What it does |
| --- | --- |
| `context_get_activity_streams` | Get time-series data for a SINGLE activity (heart rate, pace, GPS, elevation, cadence, power). WARNING: Do NOT use for batch analysis across many... |
| `context_get_activity_summary` | Get detailed Strava activity summary including distance, pace, heart rate, elevation, and power data. |
| `context_get_authorized_data` | Check Strava connection status, plus Apple Health and Wahoo. See every fitness data source linked to Kailo, the channel each arrives on, and which one... |
| `context_get_marathon_prediction` | Get KAILO's marathon time predictions for a date range — computed by Kailo's own versioned model (`model_version` in the response) from the user's... |
| `context_get_marathon_training_benchmarks` | Call this before creating or evaluating a marathon training plan. Returns observational 16-week percentile curves derived from thousands of... |
| `context_get_period_summary` | Get Strava training stats for a time period. Returns total mileage, time, pace averages, and activity breakdown. |
| `context_get_schema` | Get available Strava activity types and field values. Discover what activity categories, stream types, and filters are available. |
| `context_get_subscription_status` | Check whether the current user has an active Pro subscription. Pro is required for all dataset_*, compute_*, and sheets_* tools. Use this to debug... |
| `context_list_activities` | List all Strava activities with pagination. Browse complete running, cycling, and workout history. |

### Route building (7)

| Tool | What it does |
| --- | --- |
| `route_add_waypoint` | Append a waypoint to the user's active route-builder draft. The new leg is routed from the previous waypoint via Mapbox using the draft's profile... |
| `route_finalize` | Publish the user's active route draft into an immutable Course. When push_to_garmin=true, the publish is followed by a chained course_push_to_garmin... |
| `route_find_places` | Find POIs of a given category near a point — e.g. coffee shops near the Marina. Returns up to `limit` candidates with name, address, lat/lng, and... |
| `route_fork_from_course` | Open a finalized course (a published route) for editing by forking it into a fresh route-builder draft owned by the current user. Returns a builder_url... |
| `route_get_status` | Return the user's active route-builder draft — profile, current waypoints, totals (distance + elevation gain estimate), and a compact polyline. Use... |
| `route_pop_waypoint` | Drop the last waypoint from the user's active route draft. Use this when an :add hit a bad path or the user changed their mind about the last leg — no... |
| `route_start` | Start a new route-builder draft. Returns a builder_url the user can open to watch the route render live as you append waypoints. **Always paste the... |

### Structured workouts (5)

| Tool | What it does |
| --- | --- |
| `workout_create` | Author a single, standalone structured workout (NOT part of a training plan) and save it to the user's Kailo workout library. Works for run, bike, pool... |
| `workout_get` | Read one of the user's standalone workouts back with its full structured spec (warmup / body / cooldown, with each step's goal and target) plus a... |
| `workout_list` | List the user's saved standalone workouts (newest first) with each workout's id, name, focus, and total distance. Use this to find a workout_id to read... |
| `workout_push_to_garmin` | Send one of the user's standalone Kailo workouts to their Garmin Connect account so it appears on their watch. Optionally schedule it on a date with... |
| `workout_remove_from_garmin` | Remove a previously-pushed standalone workout from the user's Garmin Connect account. No-op success if the workout was never pushed (or is already... |

### Training plans (5)

| Tool | What it does |
| --- | --- |
| `training_plan_activate` | Make a DRAFT training plan the user's active plan — what their plan page shows, what activities reconcile against, and what can be pushed to their... |
| `training_plan_add_week` | Append a week to a DRAFT plan you previously created (or replace an existing week with the same number). Provide plan_id, the 1-based week number, and... |
| `training_plan_create` | Persist a training plan that YOU have designed into the user's account as a draft. First reason out the full schedule yourself (weeks → days →... |
| `training_plan_get` | Read a training plan back with its full week-by-week schedule and the markdown context (plan narrative, per-week focus, per-day coaching notes) you... |
| `training_plan_update_workout` | Re-author a single day in a DRAFT plan you previously created. Identify the day by plan_id, week (1-based) and day (weekday, e.g. 'Thu'). Provide... |

### Courses (2)

| Tool | What it does |
| --- | --- |
| `course_push_to_garmin` | Send one of the user's GetFast courses to their Garmin Connect account. The watch picks it up on the next sync. Requires the user to have linked Garmin... |
| `course_remove_from_garmin` | Remove a previously-pushed course from the user's Garmin Connect account. No-op success if the course was never pushed. Does not affect the local... |

### Google Sheets (18)

| Tool | What it does |
| --- | --- |
| `sheets_add_tab` | [Pro] Add a new tab to a registered spreadsheet. Returns the Google `sheet_id` you'll need for rename/delete/duplicate. |
| `sheets_append_values` | [Pro] Append rows below the table anchored at `range`. Google walks downward to find the first empty row; the anchor doesn't have to be the literal... |
| `sheets_batch_format` | [Pro] Apply up to 50 styling ops in one Google call. Each item is `{range, format?, borders?, merge?}` (same fields as `format_cells`); 1 rate-limit... |
| `sheets_batch_read_values` | [Pro] Read up to 50 A1 ranges in a single Google call (one rate-limit token total). Truncation is cumulative across ranges; `next_cursor` resumes... |
| `sheets_batch_update_values` | [Pro] Write up to 50 ranges in one Google call. Each item is `{range, values}`; the call is atomic from Google's rate-limit POV (1 token total). |
| `sheets_clear_values` | [Pro] Clear all values from an A1 range. |
| `sheets_create_spreadsheet` | [Pro] Create a new Google Sheet (under the calling user's linked Google OAuth) and register it in one call. |
| `sheets_delete_tab` | [Pro] Delete a tab. Refused with `cannot_delete_last_tab` if it's the only tab; add a new tab first then retry. |
| `sheets_duplicate_tab` | [Pro] Duplicate `tab_id` to a new tab. Omit `new_title` to let Google name it `Copy of …`. |
| `sheets_format_cells` | [Pro] Style one range: background/text color, bold/italic, alignment, wrap, number format, borders, and/or merge. Set at least one of... |
| `sheets_freeze_dimensions` | [Pro] Freeze the first N rows and/or columns of a tab so they stay visible while scrolling. `tab_id` is the numeric `sheet_id`; pass 0 to unfreeze an axis. |
| `sheets_get_metadata` | [Pro] Fetch the live spreadsheet metadata (title, locale, tabs). Returns up to `max_tabs` tabs per call; use the opaque `next_cursor` to page through... |
| `sheets_list_spreadsheets` | [Pro] List the calling user's registered spreadsheets, cursor paginated. Returns `registration_id`, `google_spreadsheet_id`, cached `title`, and the... |
| `sheets_read_values` | [Pro] Read one A1 range. The response is sliced at a row boundary so it fits in `max_cells` total cells; use `next_cursor` to resume reading the rest.... |
| `sheets_register_spreadsheet` | [Pro] Register an existing Google Sheet under the calling user. The spreadsheet must already be accessible to the user's linked Google OAuth credential. |
| `sheets_rename_tab` | [Pro] Rename a tab. `tab_id` is the numeric `sheet_id` returned by `get_metadata` or any of the tab ops (stable across renames; the visible name is the... |
| `sheets_unregister_spreadsheet` | [Pro] Soft-delete the registration (the Google sheet itself is untouched). Pass `expected_version` for OCC — get it from the most recent... |
| `sheets_update_values` | [Pro] Overwrite a single range with a 2D array of values. |

### Python compute sessions (9)

| Tool | What it does |
| --- | --- |
| `compute_end_session` | [Pro] Terminate the compute session and cleanup resources. Variables and state will be lost. |
| `compute_execute` | [Pro] Execute Python code for numerical analysis; variables persist between executions. Report findings to the user as text in your response — print... |
| `compute_get_variable` | [Pro] Get the value of a variable from the session. Supports different output formats for DataFrames. |
| `compute_install_package` | [Pro] Install a pip package in the session. |
| `compute_list_files` | [Pro] List files written to the compute session working directory. Any file saved during code execution (plots, CSVs, exports, etc.) appears here. Also... |
| `compute_list_packages` | [Pro] List all installed packages in the session. |
| `compute_read_file` | [Pro] Read a text file (CSV, JSON, TXT) from the compute session working directory back into your context — e.g. to inspect a CSV you just wrote. Do... |
| `compute_session_status` | [Pro] Get current compute session status including available variables, installed packages, and session info. IMPORTANT: Wait at least 30 seconds... |
| `compute_start_session` | [Pro] Start an isolated Python kernel. Only pandas, numpy, and pyarrow are pre-installed — for anything else (scipy, sklearn, matplotlib, etc.) pass it... |

### Datasets (4)

| Tool | What it does |
| --- | --- |
| `dataset_delete` | [Pro] Delete a dataset you own. This cannot be undone. |
| `dataset_info` | [Pro] Get detailed information about a dataset including schema, row count, columns, and usage instructions. Returns a 'usage' field showing exactly... |
| `dataset_list` | [Pro] List accessible datasets (your own, shared with you, and public datasets). To load in compute session: download_dataset('name', 'name.parquet');... |
| `dataset_save_activities` | [Pro] Save Strava activities as Parquet datasets. Use include_streams=true for detailed analysis (hill climbing, pacing, HR zones). Creates two... |

### Settings & data (2)

| Tool | What it does |
| --- | --- |
| `data_recompute_race_predictions` | Clear and recompute the user's recent marathon prediction history (up to the last 60 days, clamped to their plan's history window) from their current... |
| `settings_set_race_prediction_source` | Change which connected provider's activity data feeds Kailo's race prediction model (the predictions themselves are always Kailo's). This is the... |

---

## Data sources

Kailo ingests from any combination of these — connect them at [kailo.fit/settings](https://kailo.fit/settings):

- **Strava** — activities and streams
- **Garmin Connect** — activities, plus sleep, HRV, resting heart rate, stress and Body Battery
- **Apple Health**
- **Wahoo**

Reads over MCP are served from Strava-sourced data by default; writes (workouts, courses) go to Garmin. See the note below for why.

### A note on Garmin data

Kailo's privacy policy does not permit forwarding Garmin data we received on your behalf via OAuth out through a public MCP connection. So on a standard Garmin connection, the Garmin-only tools (sleep, HRV, resting heart rate, daily wellness) are not exposed over MCP, and Garmin-sourced activities are not returned.

You can lift this for your own account by connecting Garmin directly, at which point the data flows from Garmin to your assistant rather than through us. See [kailo.fit/garmin-mcp-unlock](https://kailo.fit/garmin-mcp-unlock).

Everything still works normally inside the Kailo app — this boundary applies only to the public MCP surface.

---

## Authentication

OAuth 2.1, no manual credential handling:

1. Your client fetches `/.well-known/oauth-protected-resource` (or reads the `WWW-Authenticate` challenge on a `401`).
2. It registers itself via Dynamic Client Registration ([RFC 7591](https://datatracker.ietf.org/doc/html/rfc7591)) at `/oauth/register`.
3. You sign in and approve access in your browser.
4. The client receives a token scoped to `mcp:tools`.

Access is per-user and revocable at any time from your Kailo account settings.

---

## Troubleshooting

**"No activities found"** — Check that a data source is connected at [kailo.fit/settings](https://kailo.fit/settings). A fresh connection takes a few minutes to sync.

**Sleep / HRV tools are missing** — Expected on a standard Garmin connection; see [the Garmin note](#a-note-on-garmin-data).

**A tool returns `ENTITLEMENT_DENIED`** — That tool needs Pro. Ask your assistant to call `context_get_subscription_status` to confirm your tier.

**Authentication failed** — Disconnect and re-add the connector so it re-runs the OAuth flow.

---

## Links

- **Documentation** — [kailo.fit/docs/claude](https://kailo.fit/docs/claude)
- **Privacy policy** — [kailo.fit/privacy](https://kailo.fit/privacy)
- **Support** — [support@kailo.fit](mailto:support@kailo.fit)

---

## About this repository

Kailo is a hosted service — the server runs on our infrastructure and there is nothing to install or self-host. This repository is the public documentation and issue tracker for the MCP endpoint; the server implementation itself is not open source.

Found a bug or want a tool that doesn't exist yet? [Open an issue](https://github.com/getfast-ai/kailo-mcp/issues).
