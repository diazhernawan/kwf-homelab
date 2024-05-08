1. Create directories that store your stacks and stores Dockge's stack

        sudo mkdir -p /opt/stacks /opt/dockge
        cd /opt/dockge

2. Download the compose.yaml

        curl https://raw.githubusercontent.com/louislam/dockge/master/compose.yaml --output compose.yaml

3. Start the server

        sudo docker compose up -d

4. If you are using docker-compose V1 or Podman

        sudo docker-compose up -d
