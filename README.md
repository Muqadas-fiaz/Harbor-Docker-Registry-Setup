# Harbor Docker Registry Setup on Debian

This guide walks you through the step-by-step installation and configuration of Harbor Docker Registry on a Debian system.

## Prerequisites

- A Debian-based system (e.g., Debian 10/11)
- Root or sudo access
- Basic understanding of Docker

## Step 1: Update System Packages

Before installing any software, it’s important to update the package list and upgrade the installed packages:

```bash
sudo apt-get update -y && sudo apt-get upgrade -y
sudo apt-get install -y curl vim wget
```
This ensures your system is up-to-date with the latest security patches and software.

## Step 2: Install Docker

Docker is the core component that Harbor relies on. Install Docker by following the commands below:

- Add Docker's official GPG key:

```bash
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
```
- Set up the Docker repository:

```bash
echo "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list
```
- Install Docker:

```bash
sudo apt-get update && sudo apt-get install -y docker-ce docker-ce-cli containerd.io
```
- Start and enable Docker:

```bash
sudo systemctl start docker
sudo systemctl enable docker
```
Docker should now be installed and running on your system.

## Step 3: Install Docker Compose

Download and install Docker Compose:

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/v2.20.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```
Verify the installation:

```bash
docker-compose --version
```
## Step 4: Download and Configure Harbor

- Download the latest version of Harbor from the official GitHub repository:

```bash
wget https://github.com/goharbor/harbor/releases/download/v2.7.0/harbor-online-installer-v2.7.0.tgz
```
- Extract the downloaded file:

```bash
tar xvf harbor-online-installer-v2.7.0.tgz
cd harbor
```
- Copy the default configuration file:

```bash
cp harbor.yml.tmpl harbor.yml
```
## Step 5: Create SSL Certificates

- Create a directory for the SSL certificate files:

```bash
sudo mkdir -p /etc/harbor/ssl
```
- Generate a private key:

```bash
sudo openssl genrsa -out /etc/harbor/ssl/private.key 2048
```
- Generate a self-signed certificate:

```bash
sudo openssl req -new -x509 -key /etc/harbor/ssl/private.key -out /etc/harbor/ssl/certificate.crt -days 365
```
- Verify that the files were created:

```bash
ls -l /etc/harbor/ssl
```

## Step 6: Configure Harbor

- Edit the harbor.yml file to match your environment:

```bash
sudo vim harbor.yml
```
Update the following lines:

```yaml
hostname: <your-ip-address>

https:
  port: 443
  certificate: /etc/harbor/ssl/certificate.crt
  private_key: /etc/harbor/ssl/private.key

harbor_admin_password: Harbor12345
```
   - hostname: Replace <your-ip-address> with your machine's IP address.
   - harbor_admin_password: Set a strong password.

## Step 7: Install Harbor

- Prepare the Harbor installation:

```bash
sudo ./prepare
```
- Install Harbor:

```bash
sudo ./install.sh
```

## Step 8: Configure Docker with Harbor’s SSL Certificate

Copy the Harbor certificate to Docker’s CA store:

```bash
sudo cp /etc/harbor/ssl/certificate.crt /usr/local/share/ca-certificates/harbor.crt
```
- Update the CA certificates:

```bash
sudo update-ca-certificates
```
-Restart Docker:

```bash
sudo systemctl restart docker
```

## Step 9: Restart Harbor

Restart Harbor services to apply the configurations:

```bash
sudo docker-compose down
sudo docker-compose up -d
```

## Step 10: Test the Configuration

Verify that you can log in to Harbor without SSL certificate errors:

```bash
docker login <your-ip-address>
```

Replace <your-ip-address> with the IP address of your Harbor server.








