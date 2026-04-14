# Scripts

This folder contains small Bash scripts to make Ethereum node operations more repeatable, faster, and less error-prone.

The goal is not to build complex software. The goal is to turn common operator tasks into simple, reusable workflows.

These scripts are meant to help with:

- daily node checks
- service validation
- quick log inspection
- restart and recovery workflows
- metrics and health verification

They are part of the broader idea of moving from:

> manual commands → repeatable operator tools

---

## Why This Exists

When operating a node stack, it is easy to keep retyping the same commands:

- `systemctl status`
- `journalctl`
- `ss -tulpn`
- `curl`
- `df -h`
- `free -h`

That works at first, but scripting those checks makes operations:

- faster
- more consistent
- easier to debug
- easier to document
- easier to reuse later

This folder is where those manual checks start becoming an actual operator toolkit.

---

## Current Goal

Build a small set of practical scripts that answer questions like:

- Are my services up?
- Are the expected ports listening?
- Are Prometheus targets healthy?
- Is disk space okay?
- What do the latest logs say?
- If I restart the node, does it recover properly?

---

## Planned Scripts

### `daily-check.sh`

Runs a quick daily health check for the node stack.

Expected checks:

- Nethermind service status
- Lighthouse service status
- Prometheus service status
- Grafana service status
- Node Exporter service status
- expected listening ports
- disk usage
- memory usage
- uptime
- optional Prometheus `up` query

This is the main single-glance script.

---

### `tail-node-logs.sh`

Shows recent logs for key services.

Expected targets:

- Nethermind
- Lighthouse

Useful for quickly checking:

- sync behaviour
- errors
- warnings
- recent restart activity

---

### `restart-node-stack.sh`

Restarts the node stack in a controlled way and verifies recovery.

Expected behaviour:

- restart services
- check status afterward
- confirm ports are listening again
- optionally confirm Prometheus scrape health

Useful for practicing service recovery and building operational confidence.

---

### `metrics-check.sh`

Runs quick checks against Prometheus or metrics endpoints.

Expected checks:

- target health
- `up` metrics
- optional peer / sync / block lag values

Useful for confirming that monitoring is still alive end-to-end.

---

## Design Principles

These scripts should be:

- simple
- readable
- safe
- reusable
- easy to run manually

This folder is intentionally Bash-first.

Why Bash:

- it fits Linux operations naturally
- it works directly with system tools
- it keeps the automation close to the commands being learned
- it is a good first step before heavier tools like Python, Ansible, or Terraform

---

## Example Operator Commands Behind the Scripts

These scripts are expected to wrap commands like:

### Service checks

    systemctl status nethermind --no-pager
    systemctl status lighthouse-beacon --no-pager
    systemctl status prometheus --no-pager
    systemctl status grafana-server --no-pager
    systemctl status prometheus-node-exporter --no-pager

### Recent logs

    journalctl -u nethermind -n 50 --no-pager
    journalctl -u lighthouse-beacon -n 50 --no-pager

### Port checks

    ss -tulpn | grep -E '8551|6060|9100|9090|3000'

### Disk checks

    df -h /

### Memory checks

    free -h

### Prometheus health query

    curl -s "http://127.0.0.1:9090/api/v1/query?query=up"

---

## How to Write Bash Scripts for This Folder

The easiest way to learn Bash scripting is to treat each script like a saved operator checklist.

A good Bash script usually has these parts:

1. a shebang so Linux knows how to run it
2. a small set of variables
3. clear printed headings
4. commands grouped by purpose
5. simple pass/fail checks where helpful
6. readable output that helps during real troubleshooting

### Basic Script Structure

Example layout:

    #!/usr/bin/env bash

    echo "Starting daily node check..."
    echo

    systemctl status nethermind --no-pager
    echo
    systemctl status lighthouse-beacon --no-pager

### Make Scripts Executable

After creating a script:

    chmod +x daily-check.sh

Then run it with:

    ./daily-check.sh

Or:

    bash daily-check.sh

### Helpful Bash Building Blocks

#### 1. Variables

Use variables to avoid repeating values:

    NETHERMIND_SERVICE="nethermind"
    LIGHTHOUSE_SERVICE="lighthouse-beacon"
    PROM_URL="http://127.0.0.1:9090"

#### 2. Functions

Functions help keep scripts tidy:

    check_service() {
        local service_name="$1"
        echo "=== $service_name ==="
        systemctl is-active "$service_name"
        echo
    }

#### 3. Condition Checks

Use `if` statements to mark success or failure:

    if systemctl is-active --quiet nethermind; then
        echo "Nethermind: OK"
    else
        echo "Nethermind: NOT RUNNING"
    fi

#### 4. Grouping Output

Use headings so the script reads like a checklist:

    echo "=== SERVICES ==="
    echo "=== PORTS ==="
    echo "=== DISK ==="
    echo "=== MEMORY ==="

#### 5. Command Exit Status

Many Linux commands return success or failure silently. Bash can use that:

    if curl -s http://127.0.0.1:9090/-/healthy >/dev/null; then
        echo "Prometheus reachable"
    else
        echo "Prometheus not reachable"
    fi

---

## How to Write Each Type of Script

### 1. Daily Node Checks

A daily check script should answer:

- are services running?
- are ports listening?
- is disk space okay?
- is memory okay?
- is uptime normal?
- are monitoring targets reachable?

A simple layout:

    #!/usr/bin/env bash

    echo "=== SERVICES ==="
    systemctl is-active nethermind
    systemctl is-active lighthouse-beacon
    systemctl is-active prometheus
    systemctl is-active grafana-server
    systemctl is-active prometheus-node-exporter

    echo
    echo "=== PORTS ==="
    ss -tulpn | grep -E '8551|6060|9100|9090|3000'

    echo
    echo "=== DISK ==="
    df -h /

    echo
    echo "=== MEMORY ==="
    free -h

    echo
    echo "=== UPTIME ==="
    uptime

What this script teaches:

- how to group checks
- how to create repeatable operational habits
- how to turn scattered commands into a daily workflow

---

### 2. Service Validation

A service validation script should confirm a service is not only installed, but actively working.

Good service validation includes:

- `systemctl is-active`
- `systemctl status`
- checking expected ports
- checking logs for recent activity

Example pattern:

    validate_service() {
        local service_name="$1"
        echo "=== Validating $service_name ==="

        if systemctl is-active --quiet "$service_name"; then
            echo "$service_name is active"
        else
            echo "$service_name is NOT active"
        fi

        systemctl status "$service_name" --no-pager | head -n 15
        echo
    }

This is useful because a service can exist in systemd but still be unhealthy, restarting, or misconfigured.

---

### 3. Quick Log Inspection

A log script should help you get signal fast.

The idea is not to dump everything. The idea is to show the most recent useful logs.

Example:

    #!/usr/bin/env bash

    echo "=== Nethermind Logs ==="
    journalctl -u nethermind -n 50 --no-pager

    echo
    echo "=== Lighthouse Logs ==="
    journalctl -u lighthouse-beacon -n 50 --no-pager

Useful variations:

    journalctl -u nethermind -f
    journalctl -u lighthouse-beacon -f

What this script teaches:

- how to inspect recent events
- how to separate normal activity from warnings and failures
- how to use logs as part of daily operations

---

### 4. Restart and Recovery Workflows

A restart script should not just restart things blindly. It should restart and then verify recovery.

Example flow:

1. restart service
2. wait briefly
3. check service status
4. check ports
5. check logs
6. optionally check monitoring health

Example:

    #!/usr/bin/env bash

    echo "Restarting Nethermind..."
    sudo systemctl restart nethermind
    sleep 5

    echo "Restarting Lighthouse..."
    sudo systemctl restart lighthouse-beacon
    sleep 5

    echo
    echo "=== SERVICE STATUS ==="
    systemctl is-active nethermind
    systemctl is-active lighthouse-beacon

    echo
    echo "=== PORTS ==="
    ss -tulpn | grep -E '8551|6060'

This teaches an important operator lesson:

> recovery is not the restart itself  
> recovery is proving the system came back cleanly

---

### 5. Metrics and Health Verification

A metrics-check script should prove monitoring still works from end to end.

Good checks include:

- is Prometheus reachable?
- does the `up` query return expected values?
- are scrape targets healthy?
- can key metrics be queried?

Example:

    #!/usr/bin/env bash

    PROM_URL="http://127.0.0.1:9090"

    echo "=== PROMETHEUS TARGETS ==="
    curl -s "$PROM_URL/api/v1/targets" | grep -E '"health"|"job"'

    echo
    echo "=== UP QUERY ==="
    curl -s "$PROM_URL/api/v1/query?query=up"

Useful later additions:

- Lighthouse sync metric
- Nethermind peer count
- Nethermind block lag
- node exporter target health

This teaches how to verify the whole monitoring chain, not just the service process.

---

## How to Implement These Scripts in This Repo

### Suggested Folder Structure

    scripts/
    ├── README.md
    ├── daily-check.sh
    ├── tail-node-logs.sh
    ├── restart-node-stack.sh
    └── metrics-check.sh

### Suggested Workflow

1. create the script
2. make it executable
3. test it manually
4. improve the output
5. document what it checks
6. commit it to the repo

Example:

    cd scripts
    touch daily-check.sh
    chmod +x daily-check.sh
    ./daily-check.sh

### Suggested README Style for Each Script

For each script, document:

- what it does
- why it exists
- what commands it wraps
- how to run it
- what good output looks like
- what failures might mean

Example section:

#### `daily-check.sh`

Purpose:

- runs a quick daily health check for the node stack

Checks:

- service state
- ports
- disk
- memory
- uptime
- optional Prometheus health

Run with:

    ./daily-check.sh

Why it matters:

- gives a single-glance view of node health
- reduces repeated manual commands
- helps build operator habits

---

## Example README Sections You Can Add Later

### Usage

    chmod +x daily-check.sh tail-node-logs.sh restart-node-stack.sh metrics-check.sh

    ./daily-check.sh
    ./tail-node-logs.sh
    ./metrics-check.sh

### Notes

- some scripts may require `sudo`
- scripts are designed for Ubuntu on WSL2
- service names should match the local systemd unit names
- ports and endpoints may need adjusting if the stack changes

### Future Improvements

- colored output
- error counting
- automatic pass/fail summary
- JSON parsing with `jq`
- alert-style exit codes
- optional email or webhook notifications
- safer restart ordering and dependency checks

---

## Learning Focus

This folder is not just about automation. It is also about learning:

- how to structure small Bash scripts
- how to use variables
- how to use functions
- how to check command success and failure
- how to print clear operator-friendly output
- how to make manual workflows repeatable
- how to think in terms of operational runbooks

---

## Why This Matters

Scripting is the bridge between:

- knowing commands
- and operating systems with confidence

A strong infrastructure engineer does not just know what to type.

They also know how to:

- package common checks
- reduce repetition
- standardize workflows
- make troubleshooting faster
- document operational habits clearly

That is what this folder is for.

---

## Status

This folder is currently being built as part of the next stage of the project:

- node setup ✅
- monitoring ✅
- dashboards ✅
- documentation ✅
- scripting ⏳

---

## Next Step

Start with:

- `daily-check.sh`

That script will become the first reusable operator health-check tool in this repository.

After that, build:

- `tail-node-logs.sh`
- `metrics-check.sh`
- `restart-node-stack.sh`

That will give this repo its first real operator automation layer.
