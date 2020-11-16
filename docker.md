# Docker

## Restart policy debugging:

https://stackoverflow.com/questions/29680274/how-to-check-if-the-restart-policy-works-of-docker/35086537#35086537

After running the container you can inspect its policy, restart coun and last started time:
docker inspect -f "{{ .HostConfig.RestartPolicy }}" <container_id>
docker inspect -f "{{ .RestartCount }}" <container_id>
docker inspect -f "{{ .State.StartedAt }}" <container_id>
Then you can look into container processes:
docker exec -it <container_id> ps -aux
The PID 1 process - is the main process, after its death the whole container would die.
Kill him using:
docker exec -it <container_id> kill -9  (maybe omit -9 option? it somehow obstructed the test on UI test iMAC (28))
And after this ensure that the container autorestarted:
docker inspect -f "{{ .RestartCount }}" <container_id>


## backup and restore folders in volumes

Volume Backup Tip:

in the same folder as dockerfile
`docker run --rm --volumes-from <running docker container id> -v $(pwd):/backup ubuntu tar cvf /backup/<backup tar file name> /<folder in container to backup>`

this command will start a new container that has access to the volumes of the indicated container

`-v $(pwd):/backup ubuntu` => this bit from above command will create a /backup folder in the new container that is mapped to $(pwd) in real filesystem

`tar cvf /backup/<backup tar file name> /<folder in container to backup>` => this bit from above will run in the container and will indireclty cause the tar file to appear in $(pwd)


`docker run --rm --volumes-from <running docker container id> -v $(pwd):/backup ubuntu bash -c "cd /<folder in container to restore> && tar xvf /backup/<backup tar file name> --strip 1"`

this command will start a new container that has access to the volumes of the indicated container

`-v $(pwd):/backup ubuntu` => this will create a /backup folder in the new container that is mapped to $(pwd) in real filesystem

`bash -c "cd /<folder in container to restore> && tar xvf /backup/<backup tar file name> --strip 1"` => this will extract the tarfile into some folder. Hopefully the same folder as above
  
