# Ops tips

## reload updated dependencies ınto services without restarting server

sudo needrestart

sudo systemctl restart *

## do this if PID 1 needs to be restrated 

sudo systemctl daemon-reexec
