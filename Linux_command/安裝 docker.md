## Step 1: update packages & install prerequisites

```bash
sudo apt update
sudo apt install -y ca-certificates curl gnupg lsb-release
```

## Step 2: add Docker's official GPG key

```bash
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
  sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

## Step 3: Add the Docker repository

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

## Step 4: Install Docker Engine

```bash
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

## Step 5: Verify the installation

```bash
sudo docker run hello-world
```

## Step 6: (optinal but recommended) Run Docker without `sudo`

```bash
sudo usermod -aG docker $USER
newgrp docker   # applies the group change in current session
```