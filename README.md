# PBS Jupyter Launcher

A lightweight Bash script for launching Jupyter on PBS-managed HPC clusters.

The launcher automatically submits a PBS job, waits for compute-node allocation, identifies the allocated node, and establishes an SSH local port-forwarding tunnel.

### Features

- Automatic PBS job submission
- Compute-node discovery via `qstat`
- PBS queue status monitoring
- Local port availability checking
- Foreground SSH port forwarding
- Press `Ctrl+C` to close the SSH tunnel without cancelling the PBS job
- Configurable working directory, PBS script, and Jupyter port

### Requirements

- Bash
- PBS (`qsub`, `qstat`)
- OpenSSH client
- SSH access from the login node to the allocated compute node
- A PBS job script that launches Jupyter on the allocated compute node

### Usage

Make the script executable:

```bash
chmod +x pjupyter
```

Run:

```bash
./pjupyter
```

Optional configuration:

```bash
export PJUPYTER_WORKDIR="$HOME/jupyterjob"
export PJUPYTER_PBS_SCRIPT="myjupyterjob.sh"
export PJUPYTER_PORT=33147
```

The Jupyter server should listen on the configured port on the first allocated compute node.

Once the SSH tunnel is established, access Jupyter through the forwarded port using an appropriate browser connection.

Press `Ctrl+C` to terminate the SSH tunnel. The PBS job will remain running.

### Notes

- SSH password authentication is supported.
- No SSH keys or credentials are stored by this script.
- A running PBS job does not necessarily mean Jupyter has finished starting.
- Cancelling a PBS job requires a separate `qdel` command.
- This script assumes Jupyter runs on the first allocated compute node and that the cluster permits SSH connections from login nodes to compute nodes.
