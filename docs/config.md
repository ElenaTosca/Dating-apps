# Configuration (`simulated_city.config`)

This module loads workshop configuration from:

- `config.yaml` (committed defaults, safe to share)
- optional `.env` (gitignored, for secrets like broker credentials)

It returns a single `AppConfig` object that contains `MqttConfig`.

## Profile-Match Configuration Notes

For the profile-based matching scenario, keep these behavior decisions aligned with docs:

- Fixed global population: 50 agents across 4 bars.
- Central match authority: `agent_match_logic.ipynb` consumes candidate events and publishes final matches.
- Tie behavior: if multiple candidates are valid at once, select the first valid candidate.

### Recommended `dating_simulation` keys in `config.yaml`

Use these keys for the profile-based matching phases:

- `agent_count_total: 50`
- `bar_ids: ["bar_1", "bar_2", "bar_3", "bar_4"]`
- `height_min_cm: 150`
- `height_max_cm: 200`
- `hair_index_min: 0`
- `hair_index_max: 49`
- `height_tolerance_cm: 10`
- `hair_distance_exact: 10`
- `match_prob_same_hair: 0.3333333333`
- `match_prob_distance_10: 0.6666666667`
- `match_random_seed: 42`

These are read from `config.yaml` while the app still uses `simulated_city.config.load_config()` for the main config flow.


## Install

The base install already includes config support:

```bash
python -m pip install -e "."
```


## Data classes

### `MqttConfig`

Holds broker and topic settings.

Typical fields:

- `host`, `port`, `tls`
- `username`, `password` (usually loaded from environment variables)
- `client_id_prefix`, `keepalive_s`, `base_topic`


### `AppConfig`

Top-level config wrapper. Currently contains:

- `mqtt: MqttConfig`


## Functions

### `load_config(path="config.yaml") -> AppConfig`

Loads configuration, applying these rules:

1. Load `.env` from the current working directory if present.
2. Find `config.yaml`:
   - if `path` exists (or is absolute), use it
   - if `path` is a bare filename like `config.yaml`, search parent directories
     so notebooks in `notebooks/` still find the repo-root `config.yaml`
3. Read `mqtt.*` settings from YAML.
4. Optionally read credentials from environment variables named in YAML:
   - `mqtt.username_env`
   - `mqtt.password_env`

Example:

```python
from simulated_city.config import load_config

cfg = load_config()
print(cfg.mqtt.host, cfg.mqtt.port, cfg.mqtt.tls)
print("base topic:", cfg.mqtt.base_topic)
```


## Internal helpers (advanced)

These are used by `load_config()` and normally don’t need to be called directly:

- `_load_yaml_dict(path) -> dict`: reads a YAML mapping (or returns `{}`)
- `_resolve_default_config_path(path) -> Path`: notebook-friendly path resolution
