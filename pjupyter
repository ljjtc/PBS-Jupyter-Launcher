
#!/usr/bin/env bash

# ==========================================
# PBS Jupyter Launcher
#
# Automatically submit a PBS job, detect
# the allocated compute node, and establish
# a foreground SSH tunnel to Jupyter.
#
# Ctrl+C closes the SSH tunnel without
# cancelling the PBS job.
# ==========================================

# Configuration (override using environment variables)
WORKDIR="${PJUPYTER_WORKDIR:-$HOME/jupyterjob}"
PBS_SCRIPT="${PJUPYTER_PBS_SCRIPT:-myjupyterjob.sh}"
PORT="${PJUPYTER_PORT:-33147}"

CHECK_INTERVAL=5
QUEUE_TIMEOUT=7200

log() {
    echo "[$(date '+%H:%M:%S')] $*"
}

fail() {
    log "[ERROR] $*"
    exit 1
}

log "========================================"
log "PBS Jupyter Launcher"
log "========================================"

# ==========================================
# Step 1: Enter working directory
# ==========================================

log "[1/4] Entering working directory..."

cd "$WORKDIR" || fail "Cannot enter $WORKDIR"

[[ -f "$PBS_SCRIPT" ]] || fail "PBS script not found: $PBS_SCRIPT"

log "[OK] Working directory: $PWD"

# Check local port availability
if (echo > /dev/tcp/127.0.0.1/"$PORT") \
    >/dev/null 2>&1; then
    fail "Local port $PORT is already occupied."
fi

log "[OK] Local port $PORT is available."

# ==========================================
# Step 2: Submit PBS job
# ==========================================

log "[2/4] Submitting PBS job..."

JOBID=$(qsub "$PBS_SCRIPT") || fail "qsub failed."

[[ -n "$JOBID" ]] || fail "Empty PBS Job ID."

log "[OK] PBS Job ID: $JOBID"

# ==========================================
# Step 3: Wait for compute node
# ==========================================

log "[3/4] Waiting for PBS allocation..."

START_TIME=$SECONDS
LAST_LOG=0
NODE=""

while true; do

    JOB_INFO=$(qstat -f "$JOBID" 2>/dev/null) \
        || fail "PBS job not found or qstat failed: $JOBID"

    STATE=$(echo "$JOB_INFO" | awk \
        '$1=="job_state" {print $3; exit}')

    NODE=$(echo "$JOB_INFO" | awk \
        '$1=="exec_host" {
            split($3, hosts, "+")
            split(hosts[1], parts, "/")
            print parts[1]
            exit
        }')

    if [[ "$STATE" == "R" && -n "$NODE" ]]; then
        log "[OK] PBS job is running."
        log "[OK] Compute node: $NODE"
        break
    fi

    if [[ "$STATE" == "F" || "$STATE" == "E" || "$STATE" == "C" ]]; then
        fail "PBS job finished before Jupyter started."
    fi

    if (( SECONDS - START_TIME >= QUEUE_TIMEOUT )); then
        fail "PBS queue timeout. Job $JOBID may still be queued."
    fi

    if (( SECONDS - LAST_LOG >= 15 )); then
        log "[WAIT] PBS state: ${STATE:-unknown}"
        LAST_LOG=$SECONDS
    fi

    sleep "$CHECK_INTERVAL"
done

# ==========================================
# Step 4: Establish SSH tunnel
# ==========================================

log "[4/4] Establishing SSH tunnel..."

log "========================================"
log "PBS Job ID   : $JOBID"
log "Compute node : $NODE"
log "Local port   : $PORT"
log "Jupyter URL  : http://127.0.0.1:$PORT"
log "========================================"
log "Starting foreground SSH tunnel..."
log "Press Ctrl+C to close the tunnel."
log "The PBS job will NOT be cancelled."
log "========================================"

# Replace the shell with the foreground SSH process.
# Ctrl+C terminates SSH and closes the tunnel.

exec ssh -N \
    -o ExitOnForwardFailure=yes \
    -o ServerAliveInterval=60 \
    -o ServerAliveCountMax=3 \
    -L "127.0.0.1:${PORT}:localhost:${PORT}" \
    "$NODE"
