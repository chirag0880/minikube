To deploy your static website on Minikube, you need to package the site into a container image (typically using Nginx), then create Kubernetes Deployment and Service resources to run and expose it.

1. Prepare Website and Container Image
First, put your static website files (HTML/CSS/JS, etc.) into a directory, then create a Dockerfile to package them into an Nginx image.
```bash
# Use official Nginx image as base
FROM nginx:alpine
# Copy your website files to Nginx default directory
COPY . /usr/share/nginx/html
# Expose port 80
EXPOSE 80
---
```bash
# For Linux
sudo apt-get update
sudo apt-get install git

# For macOS
brew install git
```
