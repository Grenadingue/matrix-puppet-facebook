# matrix-puppet-facebook [![#matrix-puppet-bridge:matrix.org](https://img.shields.io/matrix/matrix-puppet-bridge:matrix.org.svg?label=%23matrix-puppet-bridge%3Amatrix.org&logo=matrix&server_fqdn=matrix.org)](https://matrix.to/#/#matrix-puppet-bridge:matrix.org)

This is a [puppetted Matrix bridge](https://github.com/matrix-hacks/matrix-puppet-bridge) for Facebook.

## Features and limitations
* The bridge lets you use a personal matrix homeserver, the bridge itself, and your matrix client as a Facebook messenger (chat) client. What your matrix user says is said by your facebook user, and what other facebook users say is said by "ghost" users in Matrix that represent them.
* Only chat is bridged, not groups, pages or events.
To use the bridge, just wait until you receive a message from another facebook user, on facebook. The bridge, if it's properly configured, will create a room and a ghost user and that user acts as the other user, so you'll talk to them as if they were matrix users and they talk to your facebook user.
* You can't initiate direct-messages.
* History is not downloaded.
* "Facebook Protocol" room it is created to show errors, and avalible facebook users.
* Avatars for facebook users is download.

## prerequisites

- node.js: [nodejs.org: Installing Node.js via package manager](https://nodejs.org/en/download/package-manager/)
- git

## installation

```
git clone https://github.com/matrix-hacks/matrix-puppet-facebook
cd matrix-puppet-facebook
npm install
```

## configure

Create and update config.json to match your setup:

```
cp config.sample.json ./config.json
nano ./config.json
```

### login to facebook

```
node login.js
```
This prompts you for your Facebook username/password, logs in, and creates an appstate.json containing your login token. It will also prompt you about login approvals (i.e. 2FA) if you have them enabled on your Facebook account. Note this script may output some errors, but as long as appstate.json is written and works properly once you run the bridge, you can ignore them.

### register the app service

This will generate a `facebook-registration.yaml` file: 

```
node index.js -r -u "http://your-bridge-server:8090"
```

Note: The 'registration' setting in the config.json needs to set to the path of this file. By default, it already is.

Make sure that from the perspective of the homeserver, the url is correctly pointing to your bridge server. e.g. `url: 'http://your-bridge-server.example.org:8090'` and is reachable. If you use reverse proxy, you can use "http://localhost:8090".

Copy this `facebook-registration.yaml` file to your home server:

```
cp facebook-registration.yaml /etc/matrix-synapse/
```

Edit your homeserver.yaml file and update the `app_service_config_files` with the path to the `facebook-registration.yaml` file:

```
nano /etc/matrix-synapse/homeserver.yaml
```

It should look like this: 

```
app_service_config_files: ["/etc/matrix-synapse/facebook-registration.yaml"]
```

## run the bridge

If you plan to run this program as a service, you can continue at the next section.

Launch the bridge with ```./start.sh``` (see \* below for more details on this). This is a bash script, so it only works on linux / osx. If you're on windows, you'll need to take a look at the script and make an equivalent batch file. It should be very simple.

Restart your Homeserver.

## run as a service

If you use systemd you can run the bridge as a service, so it will start automatically on system boot.

First edit the paths in the unit file:

```
nano matrix-puppet-facebook.service
```

Make sure the matrix-synapse user has write permissions to this directory.

After editing enable the service: 

```
cp ./matrix-puppet-facebook.service /etc/systemd/system
systemctl daemon-reload
systemctl enable matrix-puppet-facebook.service
systemctl start matrix-puppet-facebook.service
```

Restart your homeserver:

```
systemctl restart matrix-synapse.service
```

## run in docker

You can also use the bridge with docker. Proceed with steps normally but instead of running script or service, compile the docker image:

```
docker build -t matrix-puppet-facebook .
```

Of course you need working docker on your system.

After that you can run the container:

```
docker run -d --restart=always -p 8090:8090 matrix-puppet-facebook
```

-d is --detach (runs in background), --restart is needed for restarting bridge container when it quits 
(it does that every few hours), -p binds port 8090 in container to 8090 on host

You can verify that containers works by running:

```
docker ps
```

## notes

\* Just to explain the reason for `start.sh`, facebook-chat-api contains a bug - https://github.com/Schmavery/facebook-chat-api/issues/555 that necessitates reconnecting to facebook periodically, otherwise message sending will start to fail after a couple of days. `start.sh` ensures that the process restarts properly any time it dies.

## Discussion, Help and Support

Join us in the [![Matrix Puppet Bridge](https://user-images.githubusercontent.com/13843293/52007839-4b2f6580-24c7-11e9-9a6c-14d8fc0d0737.png)](https://matrix.to/#/#matrix-puppet-bridge:matrix.org) room
