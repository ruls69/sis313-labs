# Laboratory Guide: Implementing a Reverse Proxy Load Balancer with NGINX

## Objective
The goal of this laboratory guide is to implement a reverse proxy load balancer using NGINX, ensuring that four team members can connect via a mobile hotspot for LAN connectivity between their laptops.

## Network Design
1. **Network Topology**  
   - A mobile hotspot will serve as the primary network for connecting all four laptops.  
   - Each laptop will act as a backend server (Web Server) connected to the NGINX reverse proxy.

2. **Connection Diagram**  
   - **Mobile Hotspot**  
      → Laptop A (Backend Server)  
      → Laptop B (Backend Server)  
      → Laptop C (Backend Server)  
      → Laptop D (NGINX Reverse Proxy)  

## VM Configuration
### Prerequisites
- Ensure that each laptop is running a virtualization tool capable of running VMs (e.g., VirtualBox, VMware).
- Each team member should set up their laptops to connect to the mobile hotspot.

### Step-by-Step Configuration
1. **Create a Virtual Machine**  
   - Create a VM on each laptop (A, B, and C) that will serve as the backend servers.  
   - Allocate resources (at least 2 GB RAM, 2 CPUs, and 20 GB disk space).  
   - Install a lightweight OS (e.g., Ubuntu Server).
   
2. **Install NGINX on Laptop D**  
   - Create a VM on Laptop D for the NGINX reverse proxy.  
   - Follow these commands to install NGINX:  
     ```bash  
     sudo apt update
     sudo apt install nginx
     ```  
   - Enable NGINX to start on boot:  
     ```bash  
     sudo systemctl enable nginx
     ```

3. **Configure Backend Servers**  
   - On each backend server (Laptops A-C), install a simple web application (e.g., a Python Flask app) to respond to requests.  
   - Ensure each application listens on a different port (e.g., 5000, 5001, 5002).

## Setup Instructions
1. **Network Configuration**  
   - Ensure each laptop has a static IP address assigned via the mobile hotspot settings.
   
2. **NGINX Configuration**  
   - On Laptop D, configure NGINX as follows:
     ```nginx  
     http {
         upstream backend {
             server <IP-of-Laptop-A>:5000;
             server <IP-of-Laptop-B>:5001;
             server <IP-of-Laptop-C>:5002;
         }
         server {
             listen 80;
             location / {
                 proxy_pass http://backend;
             }
         }
     }
     ```
   - Replace `<IP-of-Laptop-A>`, `<IP-of-Laptop-B>`, and `<IP-of-Laptop-C>` with the actual static IPs of the respective laptops.
   
3. **Testing the Load Balancer**  
   - Start the web applications on Laptops A, B, and C.  
   - Start the NGINX service on Laptop D:  
     ```bash  
     sudo systemctl start nginx
     ```  
   - Access the Load Balancer by navigating to `http://<IP-of-Laptop-D>` in a web browser.

## Evaluation Criteria
- **Functionality**  
   - Confirm that requests are correctly distributed across the backend servers.  
- **Performance**  
   - Evaluate response times during peak load conditions (e.g., simultaneously accessing the load balancer from multiple devices).
- **Documentation**  
   - Ensure all steps are documented, including any issues encountered and troubleshooting steps taken.

## Conclusion
By following this guide, the team should successfully implement a reverse proxy load balancer with NGINX, utilizing a mobile hotspot for LAN connectivity. This exercise will strengthen understanding of load balancing, reverse proxies, and network configurations.