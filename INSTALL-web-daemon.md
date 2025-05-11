Installing this modified version as a web daemon
=================================================

**NOTE**: This deployment will only give a Signal web frontend for one account. This isn't a generic Signal web frontend !

Getting the source code
-----------------------

The code is currently only in the 'new-presage' branch of this fork.
Some bits (like the presage upgrade) might be upstreamed, but the rest is probably way too specific and experimental to be merged.

TL;DR: `git clone https://github.com/Tofee/axolotl.git -b new-presage`

Build for aarch64
-----------------

There is a helper for building for this target:
`make build-aarch64`

You'll end up with a binary in target/aarch64-unknown-linux-gnu/release/axolotl .

Deployment
----------

1. Binaries and web frontend

Copy the binary and the web frontend to a dedicated aarch64 server:

```sh
scp target/aarch64-unknown-linux-gnu/release/axolotl myserver:/usr/local/axolotl/
scp -r axolotl-web/dist myserver:/usr/local/axolotl/axolotl-web
```

Now the rest of the steps are done on *myserver*

Eventually, dedicate a user to axolotl:
```sh
sudo useradd -M axolotl
sudo usermod -L axolotl
sudo chown axolotl -R /usr/local/axolotl
```

2. Configuration folder

When started, axolotl will want to use a directory to store its configuration. In the proposed systemd service, XDG_CONFIG_HOME is set to `/home/%i/.config` so that axolotl will use that as a base.
So, for the wanted *user*, create the config directory and give axolotl write access:
```sh
install -m 0660 -g axolotl $USER/.config/axolotl.nanuc
```

3. systemd service

The provided systemd service can be used to start axolotl (replace *user* with the username you want to use to store axolotl's config):
```sh
sudo systemctl start axolotl@user
```

4. nginx reverse proxy

Axolotl will listen on localhost:8091 for the HTTP web pages, and on 8090 port for the Websocket.
An nginx reverse proxy configuration can look like this:

```nginx
location / {
		# Forward incoming requests to local axolotl daemon
		proxy_pass http://localhost:9081;

		# Allow upgrades to websockets
		proxy_set_header Host $host;

		# WebSocket support
		proxy_http_version 1.1;
		proxy_set_header Upgrade $http_upgrade;
		proxy_set_header Connection "upgrade";
}

location /ws {
		# Forward incoming requests to local axolotl daemon
		proxy_pass http://localhost:9080/ws;

		# Allow upgrades to websockets
		proxy_set_header Host $host;

		# WebSocket support
		proxy_http_version 1.1;
		proxy_set_header Upgrade $http_upgrade;
		proxy_set_header Connection "upgrade";
}
```

5. Security concerns

**Be careful with this experiment**: your web frontend, exposed to the world, should be properly secured (authentication, SSL, etc).
Anyone with access to the proxify'ed web frontend will be able to use your Signal account!
