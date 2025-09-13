# Private Docker Registry with SSL and Authentication

This guide walks you through setting up a private Docker registry with SSL certificates and HTTP basic authentication.

## Prerequisites

- Ubuntu/Debian-based system
- Root or sudo access
- Basic understanding of Docker and networking

## Server Setup

### 1. Install Docker

Install Docker using the official installation script:

```bash
curl -fsSL https://get.docker.com | sh
```

### 2. Install Apache Utils (for htpasswd)

```bash
sudo apt install -y apache2-utils
```

### 3. Create Directory Structure

```bash
mkdir registry-server
cd registry-server
mkdir registry
cd registry
mkdir -p auth certs data
```

### 4. Create User Authentication

Create a user account for registry access (replace `uzair` with your desired username):

```bash
htpasswd -Bc ./auth/htpasswd uzair
```

You'll be prompted to enter a password for the user.

### 5. Generate SSL Certificates

Navigate to the certs directory and create the certificate configuration:

```bash
cd certs
nano san.cnf
```

Paste the following configuration (replace `REGISTRY_NAME` with your desired registry hostname):

```ini
[req]
default_bits       = 4096
distinguished_name = req_distinguished_name
x509_extensions    = v3_req
prompt             = no
[req_distinguished_name]
CN = REGISTRY_NAME
[v3_req]
subjectAltName = @alt_names
[alt_names]
DNS.1 = REGISTRY_NAME
```

Generate the SSL certificate and private key:

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:4096 \
  -keyout domain.key -out domain.crt -config san.cnf -extensions v3_req
```

### 6. Create Docker Compose Configuration

Navigate back to the `registry-server` directory and create the Docker Compose file:

```bash
cd ../../
nano docker-compose.yml
```

Add the following configuration:

```yaml
version: "3.8"
services:
  registry:
    image: registry:2.8.2
    restart: always
    ports:
      - "5000:5000"
    environment:
      REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY: /var/lib/registry
      REGISTRY_AUTH: htpasswd
      REGISTRY_AUTH_HTPASSWD_PATH: /auth/htpasswd
      REGISTRY_HTTP_TLS_CERTIFICATE: /certs/domain.crt
      REGISTRY_HTTP_TLS_KEY: /certs/domain.key
    volumes:
      - ./registry/data:/var/lib/registry
      - ./registry/auth:/auth
      - ./registry/certs:/certs
```

### 7. Start the Registry

```bash
docker compose up -d
```

## Local Testing (Same Machine)

To test the registry locally on the same machine:

1. Add the registry hostname to your hosts file:

```bash
sudo nano /etc/hosts
```

Add the following line (replace `REGISTRY_NAME` with your chosen name):

```
127.0.0.1 REGISTRY_NAME
```

2. Test the registry:

```bash
curl --cacert /path/to/registry-server/registry/certs/domain.crt https://REGISTRY_NAME:5000/v2/
```

## Client Machine Setup

### Method 1: Per-Registry Certificate (Recommended)

1. Add the registry hostname to the client's hosts file:

```bash
sudo nano /etc/hosts
```

Add the following line (replace with actual registry server IP and hostname):

```
REGISTRY_IP REGISTRY_NAME
```

2. Create Docker certificate directory:

```bash
sudo mkdir -p /etc/docker/certs.d/REGISTRY_NAME:5000
```

3. Copy the certificate from the registry server:

```bash
# Copy domain.crt from registry server to client
sudo cp /path/to/registry/certs/domain.crt /etc/docker/certs.d/REGISTRY_NAME:5000/ca.crt
```

4. Restart Docker:

```bash
sudo systemctl restart docker
```

### Method 2: Global Certificate Trust

For system-wide certificate trust:

1. Copy certificate to system certificate store:

```bash
sudo cp /path/to/registry/certs/domain.crt /usr/local/share/ca-certificates/REGISTRY_NAME.crt
```

2. Update system certificates:

```bash
sudo update-ca-certificates
```

3. Restart Docker:

```bash
sudo systemctl restart docker
```

## Using the Registry

### Login to Registry

```bash
docker login REGISTRY_NAME:5000
```

Enter the username and password created in step 4.

### Push an Image

```bash
# Tag an existing image
docker tag your-image:latest REGISTRY_NAME:5000/your-image:latest

# Push to registry
docker push REGISTRY_NAME:5000/your-image:latest
```

### Pull an Image

```bash
docker pull REGISTRY_NAME:5000/your-image:latest
```

## Troubleshooting

### Common Issues

1. **Certificate errors**: Ensure the certificate's Common Name (CN) matches the hostname used to access the registry.

2. **Authentication failures**: Verify the htpasswd file is correctly mounted and the username/password are correct.

3. **Connection refused**: Check if the registry service is running and ports are accessible.

### Verify Registry Status

```bash
# Check if container is running
docker ps

# Check registry logs
docker compose logs registry

# Test registry API
curl -u username:password https://REGISTRY_NAME:5000/v2/_catalog
```

## Security Considerations

- Use strong passwords for registry authentication
- Regularly update the registry Docker image
- Consider using a reverse proxy (nginx/Apache) for additional security features
- Monitor registry access logs
- Backup registry data regularly

## File Structure

```
registry-server/
├── docker-compose.yml
└── registry/
    ├── auth/
    │   └── htpasswd
    ├── certs/
    │   ├── domain.crt
    │   ├── domain.key
    │   └── san.cnf
    └── data/
        └── (registry storage)
```

## Notes

- Replace `REGISTRY_NAME` with your actual registry hostname throughout the configuration
- Replace `REGISTRY_IP` with the actual IP address of your registry server
- The certificate generated is valid for 365 days
- Default registry port is 5000, but can be changed in the docker-compose.yml file
