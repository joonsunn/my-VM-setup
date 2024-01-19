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
    git config --global core.editor code
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

Set up Cloudflare tunnel: <https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/get-started/create-local-tunnel/>

Install and run `cloudflared` as service: <https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/configure-tunnels/local-management/as-a-service/linux/>

Default commands does not work. Need to specify config file location for mine to work:

```bash
sudo cloudflared --config /home/username/.cloudflared/config.yml service install
```
