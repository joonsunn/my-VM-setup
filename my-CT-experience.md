# Set up LXC to eventually run docker from it

## Install ubuntu CT

1. Download CT template.
2. Create CT

## Initial setup

1. Initial setup only creates root user. Best to create a non-root user straightaway after login:

   ```bash
   adduser newuser
   ```

   Replace `newuser` with whatever username you choose.

2. Add `newuser` to sudoers file: <https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-ubuntu-20-04>

   ```bash
   nano /etc/sudoers
   ```

   Scroll down until the line that says:

   ```bash
   # User privilege specification
   root    ALL=(ALL:ALL) ALL
   ```

   Add `newuser` to below `root`:

   ```bash
   newuser    ALL=(ALL:ALL) ALL
   ```

   Save and exit file.

3. Install `openssh-server` and start the service:

   ```bash
   sudo apt install openssh-server
   sudo systemctl start ssh
   ```

   Then ssh into the container to make life easier.

4. Do the usual update and upgrade.

## SSH stuff from client side

1. Generate ssh keys to be added to Github: <https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent?platform=linux>

   ```bash
   ssh-keygen -t ed25519 -C "your_email@example.com"
   exec ssh-agent bash
   ssh-add ~/.ssh/id_ed25519
   cat ~/.ssh/id_ed25519.pub
   ```

   Copy output and paste into Github ssh keys page.

   Test connection: <https://docs.github.com/en/authentication/connecting-to-github-with-ssh/testing-your-ssh-connection>

   ```bash
   ssh -T git@github.com
   ```

   Should see something like the following:

   ```bash
   Hi newuser! You've successfully authenticated, but GitHub does not provide shell access.
   ```

## Install VS Code

1. Download VS Code .deb file from VS Code website: <https://code.visualstudio.com/download>
2. Install via terminal: <https://code.visualstudio.com/docs/setup/linux>

   ```bash
   sudo apt install ./{code.deb}
   ```

3. Install `git`.
4. setup global configs: <https://git-scm.com/book/en/v2/Getting-Started-First-Time-Git-Setup>

   ```bash
   git config --global user.name "John Doe"
   git config --global user.email johndoe@example.com
   git config --global core.editor "code --wait"
   ```

5. If "Open folder in VS Code" option does not appear in context menu in a folder: <https://forums.linuxmint.com/viewtopic.php?t=286961>
   Create file `vscode.nemo_action` in `~/.local/share/nemo/actions` with following content:

   ```bash
    [Nemo Action]
    Name=Open in VS Code
    Comment=Open in VS Code
    Exec=code "%F"
    Icon-Name=com.visualstudio.code
    Selection=Any
    Extensions=dir;
   ```

## Installing Docker

UPDATE: follow instructions here better: <https://docs.docker.com/engine/install/ubuntu/>  
Info: <https://www.youtube.com/watch?v=Ax66SnZROKA>

1. Install docker:

   ```bash
   apt install docker.io docker-compose -y
   ```

2. Install Portainer

   ```bash
   docker volume create portainer_data
   docker run -d -p 8000:8000 -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest
   ```

3. Need to add user to `docker` group so as not need to `sudo` every command:

   ```bash
   sudo groupadd docker
   sudo usermod -aG docker $USER
   newgrp docker
   docker run hello-world
   ```

## Install NodeJS

1. Install `nvm`: <https://github.com/nvm-sh/nvm>

   ```bash
   curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
   ```

2. Make `bash` pickup `nvm`:

   ```bash
   source ~/.bashrc
   ```

3. Show list of NPM versions available:

   ```bash
   nvm list-remote
   ```

4. Install desired nodejs version:

   ```bash
   nvm install v20.11.0
   ```

   Alternatively, can install latest:

   ```bash
   nvm install node
   ```

5. Install `npm`:

   ```bash
   npm install -g npm@latest
   ```

## Proxying

Could not get Nginx Proxy Manager to work. Might revisit.

Set up Cloudflare tunnel. `Tunnel` seems to no longer appear under normal Cloudflare website dashboard. Need to "buy" a plan on Cloudflare Zero Trust: <https://www.cloudflare.com/products/tunnel/>

To set up Cloudflare Tunnel (follow all steps): <https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/get-started/create-local-tunnel/>

Install and run `cloudflared` as service: <https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/configure-tunnels/local-management/as-a-service/linux/>

Default commands does not work. Error: `Cannot determine default configuration path. No file [config.yml config.yaml] in [~/.cloudflared ~/.cloudflare-warp ~/cloudflare-warp /etc/cloudflared /usr/local/etc/cloudflared]`. Need to specify config file location for mine to work: <https://community.cloudflare.com/t/cloudflared-cannot-determine-default-configuration-path/334399/2>

```bash
sudo cloudflared --config /home/username/.cloudflared/config.yml service install
```

Basically, create tunnel, then register DNS records for tunnelling attached to that tunnel. Then run the tunnel.

Multiple DNS records can be registered to the same tunnel. Just add additional ingress rules in the same `config.yml` file once the additional DNS route `CNAME` has been created.

As a refresher, to create/add new `CNAME` DNS route to a tunnel:

```bash
cloudflared tunnel route dns <UUID or NAME> <hostname>
```

`UUID` refers to tunnel id. `hostname` is the new `CNAME` to add. This will be the full path e.g. `app1.DOMAINNAME.tld`.

To update tunneling config without downtime: <https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/deploy-tunnels/deploy-cloudflared-replicas/>

UPDATE: Easier to just use remotely-managed tunnel instead of CLI: <https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/get-started/create-remote-tunnel/>

Easier to make changes, especially adding routes to tunnel.

NOTE: Cloudflare recommends one tunnel per machine. Adding more identities to the same config file in CLI does not work anyway, hence why just use remotely-managed tunnels. Had problem adding routes to existing tunnel config (link #2 keeps going to link #1).

## Running Docker containers

In the root directory of the app (where the /src folder resides together with `Dockerfile` and `docker-compose.yml`):

1. Build (and rebuild after a `git pull`) image

   ```bash
   docker-compose build
   ```

2. Build container

   ```bash
   docker-compose up
   ```

3. Stop container

   ```bash
   docker-compose stop
   ```

4. Start container

   ```bash
   docker-compose start
   ```

5. Delete container

   ```bash
   docker-compose down
   ```

6. To run a `shell` session from within a container:

   ```bash
   docker exec -it container_name sh
   ```

   or

   ```bash
   docker compose exec -it container_name sh
   ```

Prefer to use `docker-compose` instead of plain `docker` because of potential to bundle apps together in the same service if needed.

Somehow the browser caches locally websites hosted on docker. Need to use incognito mode to check if website is refreshed properly.

## CI/CD workflow using Github Actions and self-hosted runner

1. At repo side: `"Actions"` -> `"Nodejs"`.
   At autogenerated workflow yaml file, modify the `runs-on` variable to be `self-hosted`
   Commit and push to main.
   Still at repo side: `"Settings"` -> `"Add Runner"` -> Follow instructions to install runner at server side.
2. At server side hosting the app: continuing from last step of first part above, when following the instructions to create a `"actions-runner"` folder, the folder can be named anything. For me I name it within an `"automated-repos"` folder, and name the actions runner folder using the project name.
   Then:

   ```bash
   ./run.sh
   ```

   Once app is running ok, chut it down and install the runner as a `systemctl` service:

   ```bash
   sudo ./svc.sh install
   sudo ./svc.sh start
   ```

Dockerfile for simple `create-react-app` app to run production build:

```Dockerfile
FROM node:20

WORKDIR /app

COPY package.json .

RUN npm install

COPY . .

RUN npm run build

EXPOSE 3000

CMD ["npx", "serve", "build"]
```

`docker-compose.yaml`:

```yaml
version: "3"
services:
  [REPLACE WITH APP NAME]:
    build: .
    pull_policy: build
    restart: always
    ports:
      - "3000:3000"
```

`.github/workflows/node.js.yml`:

```yaml
name: Node.js CI

on:
  push:
    branches: ["main"]
  pull_request:
    branches: ["main"]

jobs:
  build:
    runs-on: self-hosted

    strategy:
      matrix:
        node-version: [20.x]

    steps:
      - uses: actions/checkout@v3
      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
          cache: "npm"
      - run: docker-compose down
      - run: docker-compose build
      - run: docker-compose up --no-start
      - run: docker-compose start
```

Alternatively, can replace above docker commands with one: `docker-compose up -d --build`. This command will not delete the old container, but instead will build it then run it in detached mode. <https://stackoverflow.com/questions/42529211/how-to-rebuild-and-update-a-container-without-downtime-with-docker-compose>

Will need to relook into above workflow file naming, because it seems the workflow is mainly deploying Docker containers rather than pure Nodejs service.

## Set up self-hosted PaaS - Coolify

_Note: below steps for projects that are configured to run as Docker container_

<https://www.coolify.io/>

1. Install as per instructions.
2. Add project folliwing on-screen instructions.
3. For private Github repo, use Deploy Key. Select the key that is created in the following sub-steps:
   1. Create key pair in `Keys & Tokens` first. Name it appropriately.
   2. In Github, add this key-pair generated in Coolify to the `Deploy Keys` section in the repo settings. Select `push events`.
4. For `Build Pack`, select `Docker Compose`. Mayeb have to re-select `Docker Compose` again in the next step.
5. Once reach `Configuration` page, under `General`, check that Compose file can be detected by clicking `Reload Compose File`. May have to rename the pre-set `Docker Compose Location` a bit (default expects .y**a**ml extension. Rename location to point to .yml if necessary).
6. Under `Webhooks`, supply a string to `Github Webhook Secret`, click `Save`, then copy the URI on the left side (e.g. `http://[IP ADDR]:8000/webhooks/source/github/events/manual`) and paste in `Webhooks` section at the Github repo settings. The `Secret` should be the same as the one input into the Coolify configuration page. The `Payload URL` needs to be a URL that is accessible from the internet. This means the earlier Coolify Webhook URI needs to be modified to be a URL that is accessible from the internet, either via a tunneled URL (e.g. Cloudflare Tunnel) or need to ensure ports are properly forwarded in order to be able to listen to external connections.
   - Easiest to just set up Cloudflare Tunnel and configure a CNAME to point to the Coolify instance. A domain name to be managed from Cloudflare Dashboard is required.
   - To remember that the URI supplied to Github Wehbooks needs to match exactly the CLoudflare Tunneled URI, i.e. _https_ instead of _http_
7. Select `Just the push event`, then click `Add webhook`. Back at the `Configuration` page, click `Reload Compose File`.

## Set up Ollama and Llama Coder

<https://ollama.com/download>
<https://github.com/ollama-webui/ollama-webui>

Follow instructions for installation of Ollama normally in system, as well as webUI using docker.

If unable to connect (unable to access Ollama from local network), need to add following in Ollama.service:

```bash
systemctl edit ollama.service
```

Add:

```bash
[Service]
Environment="OLLAMA_HOST=0.0.0.0"
```

Then:

```bash
systemctl daemon-reload
systemctl restart ollama
```

In LlamaCoder in VS Code,

```bash
Endpoint: [IP address in HTTP] or [URL in HTTPS if routed through internet/cloudflareTunnel]
```

Inference: Model -> Recommend to just stick to stable-code. CodeLlama don't do very good job of generating code chunks, only one line at a time.

If want to use models other than the ones listed (e.g. llama2), choose custom, then under Inference:Custom:Model type in the model name as it appears in OllamaWebUI, e.g. `llama2:latest`.

If VS Code says model cannot be loaded and needs to be downloaded (even if already downloaded at webUI or other previous steps), just click `yes`. It will resolve by itself.

## Set up Continue.dev VS Code extension

For self-hosted models (using Ollama as provider), config as follows (example):

```json
"models": [
    {
      "title": "Code Llama",
      "provider": "ollama",
      "model": "codellama:7b-code-q4_K_S",
      "apiBase": "[IP or URL]"
    },
```

## Set up testing in Next.js project

<https://nextjs.org/docs/app/building-your-application/testing/jest>

<https://acubeddu87.medium.com/nextjs-14-app-router-and-unit-testing-f0ba74b5436b>

<https://stackoverflow.com/questions/62672119/test-suite-failed-to-run-cannot-find-module-or-its-corresponding-type-declaratio>
