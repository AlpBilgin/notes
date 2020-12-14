# Network

## MAcos figure out the process that is blocking a port

macOS High Sierra and later, use this command:

sudo lsof -nP -iTCP:$PORT | grep LISTEN

On older versions, use one of the following forms:

sudo lsof -nP -iTCP:$PORT | grep LISTEN
sudo lsof -nP -i:$PORT | grep LISTEN
Substitute $PORT with the port number or a comma-separated list of port numbers.

The -n flag is for displaying IP addresses instead of host names. This makes the command execute much faster, because DNS lookups to get the host names can be slow (several seconds or a minute for many hosts).

The -P flag is for displaying raw port numbers instead of resolved names like http, ftp or more esoteric service names like dpserve, socalia.

To kill the PID:

sudo kill -9 <PID>
# kill -9 60401
