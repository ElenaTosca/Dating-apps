
### Scenario: Nightlife in Kødbyen, Copenhagen

The simulation models 50 people across 4 bars.

- People are split into 4 bar groups.
- Each person has profile attributes (including integer height and ordered hair-color index).
- Matching is decided centrally by the `agent_match_logic` notebook using profile-based rules.

People move around each bar and start conversations when they are within a 1 meter radius of each other.
If two people satisfy the profile match rules and are confirmed by `agent_match_logic`, they leave the bar together and are removed from the map after walking outside of the bar boundary.

### Simulation Components

#### 1) Trigger (Agents + Environment)
- Agents: people moving between and within bars
- Environment: nightlife setting with time-based events

#### 2) Observer (Sensors)
- Each bar senses proximity and conversation continuity, then publishes candidate events for centralized match evaluation.

#### 3) Control Center (Logic)
- Rule: final match authority is centralized in `agent_match_logic`, which applies profile matching rules.
- If multiple candidates are valid at the same moment, first valid candidate is selected.

#### 4) Response (Controller)
- Matched pairs leave the bar and disappear from the simulation map
- Every 50 seconds, 25% of all agents switch bars, and destinations are chosen to keep bars almost equal.

### Design Clarification (Before Coding)

This section formalizes the current scenario in technical language without implementation details.

#### 1) Four Components in Technical Language

- Trigger (agents and environment):
	- 50 person-agents move within four bar zones.
	- A periodic redistribution event moves a subset of eligible agents between bars.
- Observer (sensing layer):
	- Bar-level sensing tracks proximity interactions and continuous interaction duration.
	- Candidate pair interactions are published with profile fields (height and hair index).
- Control (decision logic):
	- The control layer (`agent_match_logic`) applies profile rules: height gate plus hair-condition probability.
	- If multiple candidates are valid at the same moment, first valid candidate is selected.
	- Eligibility rules determine whether agents can match, switch bars, or exit.
- Response (actuation):
	- Confirmed pairs transition to exit behavior and are removed after the exit duration.
	- At each redistribution interval, eligible agents are reassigned to new bars.

#### 2) MQTT Topic Map by Agent (Publish / Subscribe)

- Bar agent (one per bar):
	- Publish: `city/agents/state`, `city/events/conversation`
	- Subscribe: `city/control/switch`, `city/control/match_confirmed`
- Match logic agent:
	- Publish: `city/events/match`, `city/control/match_confirmed`
	- Subscribe: `city/events/conversation`, `city/agents/state`
- Mobility or switch agent:
	- Publish: `city/control/switch`, `city/events/switch`
	- Subscribe: `city/agents/state`
- Dashboard agent:
	- Publish: optional KPI topic only
	- Subscribe: `city/agents/state`, `city/events/match`, `city/events/switch`

#### 3) Configuration Parameters to Define

- MQTT broker and client settings:
	- Host, port, TLS enabled/disabled, credentials via environment variables, base topic, QoS, retain policy, reconnect policy
- Geography and map settings:
	- Bar polygons or centers, exit points, coordinate reference system, map center and zoom
- Population settings:
	- Total people, color distribution, initial per-bar allocation, max matches per person
- Behavior thresholds:
	- Interaction radius, match duration threshold, conversation timeout, exit duration, switch interval, switch fraction
- Time and reproducibility:
	- Tick interval, total simulation duration, random seed
- Data quality and validation:
	- Message freshness limit, duplicate handling policy, malformed payload policy
- Metrics:
	- Total matches, matches per minute, active population over time, matches per bar

#### 4) Ambiguities and Assumptions to Resolve

- No major unresolved ambiguities remain for core behavior.
- Optional refinements can be added later (payload schema details and KPI reporting format).

#### 5) Realistic Starting Values (v1)

- Population:
	- 50 total people across 4 bars
	- Each person has unique integer height and unique ordered hair-color index
- Time:
	- Tick interval: 1 second
	- Simulation duration: 900 seconds (15 minutes)
	- Match threshold: 10 continuous seconds
	- Conversation timeout: 20 seconds
	- Exit duration: 20 seconds
	- Switch interval: every 50 seconds
- Movement:
	- Speed range: 0.8 to 1.2 m/s
	- Interaction radius: 1 meter
- Switching:
	- 25% of all agents switch bars per interval
	- Destination selected to keep bar populations almost equal (within tolerance)
- Reliability:
	- Ignore state messages older than 5 seconds
	- For `city/events/match`: ignore duplicate `match_id` events and ignore events older than 5 seconds
	- Keep the first valid match event as final (no overwrite)
	- Use deterministic seed 42 for repeatable runs
- Success target:
	- At least 3 matches per minute

#### Resolved Clarification Decisions

- One person cannot match more than once in a single simulation run.
- All agents are allowed to switch bars.
- If multiple candidates are valid at the same moment, first valid candidate is selected.
- Initial bar populations should stay almost equal.
- The dashboard is read-only.
- A conversation starts when two people are within a 1 meter radius of each other.
- `agent_match_logic` confirms matches when both profile conditions are met (height and hair rule).
- "Outside the bar" is defined as crossing the bar boundary line.
- Success target: at least 3 matches per minute.
- Duplicate or late match events are ignored, and the first valid match event is final.

#### Finalized Design Decisions

Use this section as the finalized reference for implementation behavior.

- Conversation end condition: talking stops when people are more than 1 meter away from each other. Conversation lasts between 2 and 20 seconds.
- Pair locking timing: if multiple candidates are valid at the same moment, first valid candidate is selected and locked.
- Bar-balance tolerance: max difference between bars is 5 people.
- Switch destination policy: destination is selected to keep bar populations almost equal.
- Match authority policy: final match confirmation comes from `agent_match_logic` based on profile matching rules.
- Population scope policy: simulation size is fixed at 50 agents globally across 4 bars.
- Tie behavior policy: if multiple candidates are valid at the same moment, select the first valid candidate.
- KPI window definition: "3 matches per minute" is measured as total matches / total runtime
- Duplicate or late match events: ignore events older than 5 seconds, ignore duplicate `match_id`, and keep the first valid match as final.
- MQTT payload fields are finalized ✅:
	- `city/agents/state`: `event_id`, `person_id`, `bar_id`, `color`, `state`, `x`, `y`, `timestamp`
	- `city/events/conversation`: `event_id`, `conversation_id`, `person_a_id`, `person_b_id`, `bar_id`, `duration_seconds`, `distance_m`, `timestamp`
	- `city/events/match`: `match_id`, `person_a_id`, `person_b_id`, `color`, `bar_id`, `duration_seconds`, `confirmed_by`, `timestamp`
	- `city/events/switch`: `event_id`, `person_id`, `from_bar_id`, `to_bar_id`, `reason`, `timestamp`