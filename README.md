1. Prepare Website and Container Image
First, put your static website files (HTML/CSS/JS, etc.) into a directory, then create a Dockerfile to package them into an Nginx image.

dockerfile
# Use official Nginx image as base
FROM nginx:alpine
# Copy your website files to Nginx default directory
COPY . /usr/share/nginx/html
# Expose port 80
EXPOSE 80
Build the image in the terminal (replace my-website with your desired name):

bash
docker build -t my-website:v1 .
Key step: Since Minikube uses its own internal container runtime, you need to load the locally built image into the Minikube environment, otherwise Kubernetes will fail to pull it.

bash
minikube image load my-website:v1
2. Create Kubernetes Configuration Files
Create a file named deployment.yaml that includes both Deployment and Service definitions.

yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-website-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-website
  template:
    metadata:
      labels:
        app: my-website
    spec:
      containers:
      - name: my-website
        image: my-website:v1
        imagePullPolicy: IfNotPresent # Prioritize using local image
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: my-website-service
spec:
  type: NodePort # Use NodePort to expose service
  selector:
    app: my-website
  ports:
  - port: 80
    targetPort: 80
    # nodePort: 30080 # Optional: manually specify port (30000-32767)
3. Deploy and Access
Apply the configuration and check status:

bash
kubectl apply -f deployment.yaml
kubectl get pods
After confirming the Pod status is Running, use Minikube's built-in command to access your website. This command will automatically open a browser or give you an accessible URL:

bash
minikube service my-website-service
If you only need the URL without automatically opening a browser, you can use:

bash
minikube service my-website-service --url
Common Troubleshooting
Image pull failure: Confirm you have executed minikube image load. If it still doesn't work, you can try setting imagePullPolicy: Never in the Deployment, forcing Kubernetes to only use local images.

Unable to access Service: Ensure you are using the minikube service command instead of directly accessing localhost. Ports on Minikube's internal network typically need to be exposed through this command.

minikube service my-website-service
┌───────────┬────────────────────┬─────────────┬───────────────────────────┐
│ NAMESPACE │        NAME        │ TARGET PORT │            URL            │
├───────────┼────────────────────┼─────────────┼───────────────────────────┤
│ default   │ my-website-service │ 80          │ http://192.168.49.2:31475 │
└───────────┴────────────────────┴─────────────┴───────────────────────────┘
🎉  Opening service default/my-website-service in default browser...
👉  http://192.168.49.2:31475
demo@demo:~/basic_site$ minikube service my-website-service --url
http://192.168.49.2:31475
demo@demo:~/basic_site$ i nned t  access in lan network
Found 15 web pages
The 192.168.49.2:31475 address you see is the internal IP of the Minikube virtual machine (or container), and other devices on your LAN cannot directly access this address.

To allow other devices on the LAN to access your website, the core approach is to forward a port on the host machine (the Linux machine running Minikube) to Minikube's internal service. Below are two feasible solutions.

Solution 1: Use iptables port forwarding (lightweight, no additional software required)
This is the most direct method. Assuming your host machine's LAN IP is 10.105.230.34 and you want other devices to access your website via port 8888.

Enable IP forwarding (if not already enabled):

bash
sudo sysctl -w net.ipv4.ip_forward=1
To make it permanent, edit /etc/sysctl.conf and set net.ipv4.ip_forward=1, then run sudo sysctl -p.

Add iptables forwarding rules:

bash
# Forward traffic destined for host port 8888 to Minikube's IP and NodePort
sudo iptables -t nat -A PREROUTING -d 10.105.230.34 -p tcp --dport 8888 -j DNAT --to-destination 192.168.49.2:31475
# Configure SNAT to ensure return traffic can return correctly
sudo iptables -t nat -A POSTROUTING -d 192.168.49.2 -p tcp --dport 31475 -j MASQUERADE
(Please replace 10.105.230.34 with your host machine's actual LAN IP)

Open the firewall port on the host machine:

bash
# If using ufw (Ubuntu/Debian)
sudo ufw allow 8888/tcp
# If using firewalld (CentOS/RHEL)
sudo firewall-cmd --add-port=8888/tcp --permanent && sudo firewall-cmd --reload
After configuration, other devices on the
