#!/bin/bash

if [ -z "$1" ]; then
  echo "Usage: $0 <command> [args...]"
  exit 1
fi

COMMAND=("$@")

# Function to kill a process and its child processes
kill_process_tree() {
    local parent_pid=$1
    local child_pids=$(pgrep -P $parent_pid)

    # Kill child processes recursively
    for child_pid in $child_pids; do
        kill_process_tree $child_pid
    done

    # Kill the parent process
    kill $parent_pid 2>/dev/null
}

# Function to handle termination signals
terminate() {
  echo "Terminating script..."
  kill_process_tree "$PID"
  exit 0
}

# Function to handle the special signal
restart_app() {
  echo "Restarting app..."
  kill_process_tree "$PID"
  start_app
}

# Function to start app
start_app() {
  setsid "${COMMAND[@]}" &
  PID=$!
}

# Trap termination signals
trap terminate SIGINT SIGTERM

# Trap special signal (USR1 in this case)
trap restart_app SIGUSR1

while true; do
  # Start the initial command
  start_app

  # Wait for the command to exit
  while kill -0 "$PID" 2>/dev/null; do
    wait "$PID"
  done

  # If the command exits, wait one second and restart it
  echo "Command crashed. Restarting in 1 second..."
  sleep 1
done
