---
layout: post
title: "Cron replacement"
date: 2019-05-27
author: "Gregor J."
tags: shell
---

_Hilfe, ich habe kein `cron`! Wie kann ich einen Prozess bloss regelmäßig laufen lassen?_

Shell script to the rescue...
```bash
#!/bin/sh

# constantly repeat this command
REPEAT_COMMAND="date +%s"
# number of seconds to sleep after each command execution
REPEAT_SLEEP=2
# max. time the command has to run
# the program `timeout` kills the command after this time
MAX_RUNTIME="10s"
# my own PID
MY_PID=$$
# my own name
MY_NAME=$(basename "$0")

# main function
function main()
{
    if is_running;
    then
        (>&2 echo "Already running!")
        exit 1
    fi

    while true;
    do
        repeat_command
        sleep ${REPEAT_SLEEP}
    done
}

# determine whether this script is already running
function is_running()
{
    ps aux | grep "${MY_NAME}" | grep -v "grep" | grep -vq "${MY_PID}"
    return $?
}

# execute the command
function repeat_command()
{
    # Create a temporary file for commands output.
    local OUTPUT=$(mktemp)
    
    # Usage: timeout [-s SIG] SECS PROG ARGS
    # 
    # Runs PROG. Sends SIG to it if it is not gone in SECS seconds.
    # Default SIG: TERM.
    timeout ${MAX_RUNTIME} ${REPEAT_COMMAND} &> ${OUTPUT}
    
    # Display all captured output in case the command failed.
    local E=$?
    if [ ${E} -ne 0 ]; then
        echo "ERROR ${E} running command ${REPEAT_COMMAND}:"
        echo "---8><--------------------------------------------------------------------------"
        cat "${OUTPUT}"
        echo "--------------------------------------------------------------------------><8---"
    fi
    rm -f "${OUTPUT}"
    return ${E}
}

# start main
main

```

Beispiel mit busybox:

```bash
vi ./cron_replacement.sh
chmod 755 cron_replacement.sh
docker run -it --rm -v $(pwd):/gregor --workdir /gregor busybox:latest ./cron_replacement.sh
```

Beispiel, wie mit das Script auch nach dem logout des users noch weiterläuft:

```bash
nohup ./cron_replacement.sh &
```
