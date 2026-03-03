# Phase Implementation Plan: Dating Scenario

This document defines a step-by-step implementation plan for:

**Nightlife in Kødbyen, Copenhagen**

Follow phases in order. Only continue after each phase passes checks and you understand the behavior.

## Phase 1: One Minimal Working Agent Notebook

### New notebook files
- `notebooks/agent_bar_1.ipynb`

### Checks/tests to run
- `python scripts/verify_setup.py`
- `python scripts/validate_structure.py`
- `python -m pytest`
- Run all cells in `notebooks/agent_bar_1.ipynb`

### Understand before continuing
- State model behavior (`idle`, `talking`, `matched`, `removed`)
- Conversation start condition (within 1 meter)
- Match condition (same color and continuous conversation > 10 seconds)
- Pair locking rule (random at first contact, then locked)
- Why simulation runs fast (simulated time, not wall-clock time)

---

## Phase 2: Configuration Integration

### New notebook files
- None (update `notebooks/agent_bar_1.ipynb`)

### Checks/tests to run
- `python scripts/verify_setup.py`
- `python scripts/validate_structure.py`
- `python -m pytest`
- Re-run notebook and verify that changing config values changes behavior

### Understand before continuing
- Which values must come from `config.yaml`
- How to use `simulated_city.config.load_config()` correctly
- How to avoid hardcoded thresholds and constants in notebook logic

---

## Phase 3: MQTT Publishing

### New notebook files
- `notebooks/agent_match_logic.ipynb`

### Checks/tests to run
- `python scripts/verify_setup.py`
- `python scripts/validate_structure.py`
- `python -m pytest`
- Run notebooks and confirm messages are published with expected payload fields

### Understand before continuing
- Topic responsibilities (`city/agents/state`, `city/events/conversation`, `city/events/match`)
- Event freshness and duplicate handling for `city/events/match`
- Match authority rule (sensor confirms only when same color + >10 seconds)

---

## Phase 4: Second Agent with MQTT Subscription

### New notebook files
- `notebooks/agent_switch_mobility.ipynb`

### Checks/tests to run
- `python scripts/verify_setup.py`
- `python scripts/validate_structure.py`
- `python -m pytest`
- Run both notebooks together and verify switch behavior every 50 seconds

### Understand before continuing
- Subscription flow and state consistency across notebooks
- Switch policy (25% of all agents)
- Destination balancing rule (keep bar populations almost equal)
- Bar-balance tolerance (max difference of 5)

---

## Phase 5: Dashboard Visualization (Read-Only)

### New notebook files
- `notebooks/dashboard.ipynb`

### Checks/tests to run
- `python scripts/verify_setup.py`
- `python scripts/validate_structure.py`
- `python -m pytest`
- Run all active notebooks and verify dashboard updates in real time

### Understand before finishing
- Dashboard is read-only (no control commands)
- KPI window definition (`3 matches/min` = total matches / total runtime)
- How match/switch/state topics appear in the visualization

---

## Completion Criteria

You are ready for submission when:
- All phase checks pass
- Notebooks are split by responsibility (no monolithic notebook)
- MQTT communication is working across agents
- Dashboard is read-only and reflects live state/events
- PR includes:

```text
Docs updated: yes
Phases completed: Phase 1-5
Tests passing: yes
```
