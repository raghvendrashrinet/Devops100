### Create Apache2 on ubuntu ,listening on port 5000
```dockerfile
# Use the official stable Ubuntu base image
FROM ubuntu:22.04

# Avoid interactive prompts during package installation
ENV DEBIAN_FRONTEND=noninteractive

# Update packages, install apache2, and clean cache to save space
RUN apt-get update && apt-get install -y \
    apache2 \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Modify Apache ports configuration to listen on 5000 instead of 80
RUN sed -i 's/Listen 80/Listen 5000/g' /etc/apache2/ports.conf \
    && sed -i 's/<VirtualHost \*:80>/<VirtualHost \*:5000>/g' /etc/apache2/sites-available/000-default.conf

# Document that the container intends to listen on port 5000
EXPOSE 5000

# Start Apache in the foreground so the container stays running
CMD ["apache2ctl", "-D", "FOREGROUND"]
```

---
### Build Image and Run Container
1. Build
```
  docker build -t apache2_image .
```
2. Run Continer
```
  docker run -dt  -p 5002:5002 apache2_image
  # -p <host-port>:<container-port>
```
3. Access the web server
```
  curl localhost:5002
```

