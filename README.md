# archean-server-updater

This script is an auto updater for the server for the game [Archean](https://store.steampowered.com/app/2941660/Archean/).

The script checks the Docker Hub API for an update to the container [xenon3d/archean-server-alpha](https://hub.docker.com/r/xenon3d/archean-server-alpha) once an hour. When the current running container's digest is different from the Docker Hub version, it pulls the new container, stops the running container, deletes the running container, then starts a new container under the same name.

## Running under systemd

Included is a systemd service file in systemd/archean-updater.service. You can use this as a template and place it either into /etc/systemd/system/ or into ~/.config/systemd/user depending on if you want to run the script as root or as a normal user.

### Running as root

First make sure that you check both the archean-updater file for the variables at the start of the file as well as check the archean-updater.service file to make sure that it has the correct path to the archean-updater file. Once that is completed you can run the following commands to add the service to systemd and enable it.

```bash
sudo git checkout https://github.com/bvierra/archean-updater.git /usr/local/archean-updater
```

Edit the /usr/local/archean-updater/archean-updater script to make sure all the variables at the top are correct. You can then run the script directly from the command line to make sure that it runs correctly.

```bash
sudo /usr/local/archean-updater/archean-updater
```

It should return something similar to:

```bash
INFO: Starting Archean-Server monitor...
INFO: Found required program: curl
INFO: Found required program: jq
INFO: Found required program: docker
INFO: Container archean_server exists
INFO: Container archean_server is running
INFO: There is no update - Digest: 9d82ece18954304389714dcd8eb04b7cee05e072e9c88c5d6659f8e35aa5160f
INFO: Sleeping for 3600 seconds
```

If so you can ctrl+c the program and continue on:

```bash
sudo cp /usr/local/archean-updater/systemd/archean-updater.service.system /etc/systemd/system/archean-updater.service
sudo systemctl daemon-reload
sudo systemctl enable archean-updater.service
```

This will create a log file that is located (by default) at /var/log/archean-updater.log

### Running as a user

Running this script as a normal user is fairly similar to running it as root.

First you need to download the script:

```bash
git checkout https://github.com/bvierra/archean-updater.git ~/archean-updater
```

If you are running docker as root and not as the user, you need to allow the user to run the docker command via sudo without needing a password. If you already allow the user to run sudo without a password for all commands then you can skip this. Otherwise run the following

```bash
echo "$USER ALL=NOPASSWD: /usr/bin/docker" | sudo tee -a /etc/sudoers.d/$USER > /dev/null
```

You also will need to allow the user to has services running even if they are not logged in. To do this run the following

```bash
sudo loginctl enable-linger $USER
```

For the variables in the archean-updater script you need to change the following:

```bash
Change this line:
#LOG_FILE=${HOME}/archean-updater/archean-updater.log
to:
LOG_FILE=${HOME}/archean-updater/archean-updater.log

Then change:
LOG_FILE=/var/log/archean-updater.log
to:
#LOG_FILE=/var/log/archean-updater.log
```

If you are running docker as root make sure to set DOCKER_ROOT=1

If you are running docker as the user, set DOCKER_ROOT=0

Please also read through all the variables and make sure they are correct for you setup. They are all documented in the script.

You can then run the script directly from the command line to make sure that it runs correctly.

```bash
~/archean-updater/archean-updater
```

It should return something similar to:

```bash
INFO: Starting Archean-Server monitor...
INFO: Found required program: curl
INFO: Found required program: jq
INFO: Found required program: docker
INFO: Container archean_server exists
INFO: Container archean_server is running
INFO: There is no update - Digest: 9d82ece18954304389714dcd8eb04b7cee05e072e9c88c5d6659f8e35aa5160f
INFO: Sleeping for 3600 seconds
```

If so you can ctrl+c the program and continue on:

You then need to copy the systemd service file to the correct directory and enable it

```bash
mkdir -p ~/.config/systemd/user
cp ~/archean-updater/systemd/archean-updater.service.user ~/.config/systemd/user/archean-updater.service
systemctl --user daemon-reload
systemctl --user enable archean-updater.service
```
