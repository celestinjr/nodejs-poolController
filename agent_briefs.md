# Agent Briefs (3 Workstreams)

## Goal

Run 3 agents with minimal overlap:

- Agent A: General tab data-path issues (Personal Info + Time/Locality + Delays)
- Agent B: Alerts/Security mapping
- Agent C: Loading-stuck/reliability behavior

This split keeps shared protocol/parser work in one place and avoids conflicting edits across multiple agents.

## Shared Context for All Agents

- Discussion thread: <https://github.com/tagyoureit/nodejs-poolController/discussions/1090#discussioncomment-15925386>
- Newly extracted replay folders:
  - `data/1090/v3.008_replay.163`
  - `data/1090/v3.008_replay.164`
  - `data/1090/v3.008_replay.170`
  - `data/1090/v3.008_replay.171`
  - `data/1090/v3.008_replay.173`
- Historical comparison replays:
  - `data/1090/v3.008_replay.157_config1`
  - `data/1090/v3.008_replay.160_config2`

---

# Agent A Brief: General Tab (Personal Info + Time/Locality + Delays)

## Scope

- Personal Information mapping (inbound + outbound)
- Timezone/Clock settings and unintended coupling to body temps/modes
- Delays tab and new v3 fields (`Frz Cycle Time`, `Frz Override`, `Manual Operation Priority`)

## Why this is grouped

These items share the same config/state pathways and overlap in `Action 30` + `Action 168` handling. Keeping them together reduces parser conflicts.

## Exact old comment excerpts to use

> "Personal Information: The information in this section does not appear to link to the OCP (and maybe it isn't intended to). In particular, the City/State/Zip does not link between what is shown on the dashPanel and the WCP. Also, the Pool Alias, Owner and Zip fields on the dashPanel change when other changes are made, with the contents of the Zip field replacing the Pool Alias and then the Zip and Owner fields becoming blank. The Longitude field can't be edited in dashPanel and it does not track changes made with the WCP. The Latitude field can't be edited and is blank in dashPanel."

> "Timezone & Locality: Changing the settings from \"Internet / 12 hour\" to \"Manual / 24 Hour\" causes the Pool Set Point temp to change to 0 and the Spa Set Point Temp to change to 100 and the Spa Heat Mode to change from UltraTemp Only to Solar Only . (Replay 160)."

> "Delays: The settings in the Delays tab do not appear to link between dashPanel and the WCP. Also, dashPanel has a \"Manual Operation Priority\" checkbox that doesn't appear to match anything on the WCP. And, the WCP has \"Frz Cycle Time (min)\" and \"Frz Overide (min)\" fields that do not appear in dashPanel."

## Exact new comment excerpts to use

> "Personal Information All fields except Lat/Lon entered on dashPanel are retained in dashPanel. Lat is blank. Lat/Lon are display only in dashPanel. Zip code, City, State and Country entered in dashPanel are not passed to WCP (other fields don’t exist in WCP). Zip code entered in WCP appears as Pool Alias in dashPanel.City entered in WCP appears as Phone in dashPanelState, Country, Latitude and Longitude entered in WCP are not reflected in dashPanel"

> "There continues to be an incorrect linkage between the time settings and the Pool/Spa temperature settings."

> "Setting Clock Source to “Server” on dashPanel causes error (Replay 163): TypeError: Cannot read properties of undefined (reading 'val') at IntelliCenterSystemCommands.setTZ (/home/pi/nodejs-poolController/controller/boards/SystemBoard.ts:1152:98) ..."

> "Setting Clock Mode (12/24 hour) in dashPanel does transfer to WCP. However, if Clock Source is “Manual” in dashPanel and WCP, changing Clock Mode in dashPanel also causes WCP Mode to switch to “Internet” (Replay 164)."

> "8:04 Changed Frz Cycle Time from 15 to 20 ... 8:24 Frz Override changed from 30 to 90"

## Replay priority

1. `v3.008_replay.171` (personal info + delays + many config changes)
2. `v3.008_replay.170` (timezone/locality coupling)
3. `v3.008_replay.164` (manual/internet mode interaction)
4. `v3.008_replay.163` (server clock source exception)
5. Cross-check with `v3.008_replay.160_config2`

## Deliverables

- Byte-level mapping table (field -> action/type -> payload offsets -> direction)
- Confirmed root-cause list per symptom
- Patch list (file + method + behavior)
- Regression checklist for General tab

---

# Agent B Brief: Alerts and Security

## Scope

- Populate/validate Alerts tab mappings
- Populate/validate Security tab mappings
- Verify round-trip behavior for WCP/OCP <-> njsPC <-> dashPanel

## Exact old comment excerpts to use

> "Alerts and Security: These tabs are currently empty. I will have to take a further look at the OCP/WCP menus to see if there are corresponding entries."

## Exact new comment excerpts to use

> "Alerts and Notifications-Circuits 8:07 Alert Valve Rotation Delay from Off to On 8:08 Heater Cool Down Delay from Off to On"

> "Alerts and Notifications-Pumps NOTE: All of the Alerts in this section were changed from On to Off ... 8:09 Pentair VS/VF/VSF Rate and Power ... 8:11 Communication Lost"

> "Alerts and Notifications-Ultra Temp Heater NOTE: All of the Alerts in this section were changed from On to Off ... 8:17 Communication Lost"

> "Alerts and Notifications-IntelliChlor NOTE: All of the Alerts in this section were changed from On to Off ... 8:19 Communication Lost"

> "Start of REPLAY 173 ... 9:30 Enable Security – Administrator ... 9:32 Change Security Passcode from 7777 to 8888 ... 9:32 Enable Guest ... 9:33 Add “Vacation Mode” access to Guest ... 9:33 Add “All” access to Guest"

## Replay priority

1. `v3.008_replay.171` (alerts changes)
2. `v3.008_replay.173` (security changes)

## Deliverables

- Mapping matrix for each alert/security setting
- Missing fields report (present on WCP/OCP but absent in dashPanel/njsPC)
- Patch list and validation notes
- Any required API schema updates for dashPanel consumption

---

# Agent C Brief: Loading-Stuck and Reliability

## Scope

- Investigate root causes of loading hang/stalls under noisy comms
- Evaluate refresh-trigger behavior (ACK(168)/ACK(184)/piggyback/config queue churn)
- Propose mitigation strategy that does not suppress valid updates

## Exact old comment excerpts to use

> "A general issue which may be specific to my system is that periodically following a settings change initiated in dashPanel, the \"Loading\" message will appear and will hang before completion. That occurred at the end of Replay 157 with the updated hanging at 0%. Although nodejs-poolController continues to run, I have to restart it to clear the problem. My system is showing a lot of failed sends (55% failure rate) which might be related to this."

## Exact new comment excerpts to use

> "I restarted the access point that the Elfin uses and have had significantly fewer transmit errors since then."

> "[12/24/2025, 10:47:00 AM] warn: Registration rejected by OCP via Action 217 (status=4) - duplicate registration attempt?"

## Replay priority

1. `v3.008_replay.157_config1` (stuck at loading 0)
2. `v3.008_replay.160_config2` (comparison run)
3. Correlate with new runs:
   - `v3.008_replay.163`
   - `v3.008_replay.164`
   - `v3.008_replay.170`
   - `v3.008_replay.171`
   - `v3.008_replay.173`

## Deliverables

- Event-timeline of one failed and one successful config cycle
- Trigger/churn analysis (`Requesting IntelliCenter configuration`, queued items, completion rates)
- Proposed mitigation options with tradeoffs:
  - refresh debounce tuning
  - in-flight refresh guard
  - stale loading-state timeout/reset behavior
  - registration retry/backoff adjustments
- Recommended implementation order

---

# Coordination Rules (Important)

- Agent A owns changes in shared General/config parser paths.
- Agent B should not modify Agent A parser logic unless required and documented.
- Agent C should avoid behavior changes that alter field parsing/mapping.
- Final integration step should run all touched replay scenarios before merge.
