## Docker Network

A Docker network is a virtual communication layer that allows isolated Docker containers to securely talk to each other, to the host machine, and to external networks like the internet  

By default, Docker isolates containers completely. A Docker network acts as the virtual switchboard that selectively connects them

### The Bridge 
When you install Docker, it automatically creates a default virtual network card on your machine called a Bridge network (named `docker0`)
- The `IP Allocation:` Every time you launch a container, Docker automatically hooks it up to this virtual router and assigns it a unique, private internal IP address (usually starting with 172.17.x.x)
  >  pool : 172.17.x.x

- Inter-container Traffic: Because they are plugged into the same virtual router, containers can seamlessly talk to one another via these private IPs.
```
       +--------------------------------------------------------+

       |                     YOUR COMPUTER                      |
       |                                                        |
       |     +--------------------------------------------+     |
       |     |          VIRTUAL BRIDGE NETWORK            |     |
       |     |                (docker0)                   |     |
       |     +-----+--------------------------------+-----+     |
       |           |                                |           |
       |           | IP: 172.18.0.2                 | IP: 172.18.0.3
       |     +-----+------+                   +-----+------+    |
       |     |  CONTAINER |                   |  CONTAINER |    |
       |     |     A      |                   |     B      |    |
       |     | (Web App)  |=== Talks to ===>  | (Database) |    |
       |     +------------+                   +------------+    |
       +--------------------------------------------------------+

```
---
#### Example: Connecting a Web App to a Database
1. Create a custom network
   ```
     docker network create my-app-net
   ```
2. Start the database container on the network
   Launch a PostgreSQL database container and attach it directly to your newly created network using the --network flag.
   ```
   docker run -d --name db-container --network my-app-net postgres
   ```
3. Start the web application container on the same network
   Launch your web application container on that  same network.
   ```
    docker run -d --name web-container --network my-app-net -p 5000:5000 my-web-app
   ```
4. How they communicate
   Inside your web application's configuration code, you do not need to guess an IP address. You can simply use the container name as the database hostname:
   ```
     # in web-container pass the db string
     DATABASE_URL = "postgresql://db-container:5432/mydb"
   ```
   * Docker’s built-in DNS engine intercepts the request to db-container and automatically routes the traffic straight to the database container over the isolated my-app-net network.
   ```
---
#### Main Types of Docker Networks
You can specify different network drivers depending on your architectural needs:
- **Bridge (Default)**: Best for isolated containers running on the same single host machine.
- **Host**: Completely removes network isolation between the container and the Docker host. The container uses the host’s IP and ports directly (e.g., a container app running on port 80 runs directly on your computer's port 80)
- **Overlay**: Connects multiple Docker daemons across entirely different physical servers. This is the backbone used for clustered container setups like Docker Swarm.
- **None**: Completely shuts off all networking capabilities for the container. Ideal for high-security, isolated batch processing tasks.

##### "Host" Network Example (No Isolation)
In a host network, the container does not get its own IP address. It binds directly to your computer's network interface. If an application inside the container runs on port 80, it immediately occupies port 80 on your host machine.  
```
  docker run -d --name high-perf-web --network host nginx
```
- No Mapping Needed: You do not use the `-p` flag. The Nginx server instantly listens on your actual machine's port 80.

##### None" Network Example
The none network completely disables the network stack for the container. It has no external loopback interface, no IP address, and absolutely zero access to other containers or the internet.
```
  docker run -d --name secure-batch-job --network none alpine sleep 3600
```
- Best Used For: Running highly sensitive security tasks, calculating data encryption keys, parsing untrusted file uploads, or running heavy offline batch calculations where you must guarantee data cannot be leaked over a network

##### "Overlay" Network Example (Multi-Server Clustering)
An overlay network connects containers running across different physical computers or servers (called nodes) as if they were plugged into the exact same switch. This requires Docker to be running in Swarm Mode.

 **Example Setup:**
 1. Initialize the cluster on Server A:
 ```
   docker swarm init --advertise-addr <SERVER_A_IP>
 ```
 **(This gives you a token string to copy).**  
 2. Join the cluster from Server B:  
 
 Run the command provided by Server A on your second machine:
 ```
  docker swarm join --token <TOKEN> <SERVER_A_IP>:2377
 ```
 3. Create the Overlay Network (Run on Server A):
   ```
    docker network create --driver overlay --attachable cloud-fabric
   ```
 4. Launch Containers across the servers:
   a). On Server A: Run a database container attached to the overlay network.
   ```
    docker run -d --name remote-db --network cloud-fabric mongo
   ```
   b) On Server B: Run your app backend attached to the same overlay network.
   ```
    docker run -d --name cluster-app --network cloud-fabric my-node-app
   ```
   ** Overlay**:In networking, the word "overlay" means a virtual network that is built on top of an existing physical network
    Think of it like a digital map layer or a tunnel: the physical network handles the heavy lifting of moving data packets between machines, while the overlay network hides that physical complexity so your applications see a single, unified environment.
   
   
   ##### **Docker Swarm** is Docker’s native, built-in tool for container orchestration.It allows you to group multiple individual computers or cloud servers (called nodes) together into a single, highly available virtual machine cluster. Instead of managing individual containers on separate machines, you issue commands to the Swarm, and it automatically decides where to run your workloads.
   
