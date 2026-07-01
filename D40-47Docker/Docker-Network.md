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
   
---
 ### Docker Drivers
  driver is a pluggable software component that tells Docker how to implement and manage core infrastructure resources like networking, storage, and logging.
  * Docker uses a pluggable architecture*
    ##### 1. Network Drivers (How containers talk)
    - ` bridge driver`: Tells Docker to use Linux bridging tools (brctl) and IPTables rules to create an isolated virtual switch on your local machine.   
    - ` overlay driver`: Tells Docker to use VXLAN encapsulation tunnels to stretch a network across multiple separate servers [authId: overlay].    
    - ` macvlan driver`: Tells Docker to bypass virtual routing entirely and assign an actual, physical MAC address to a container, making it look like a physical machine on your home    router.
    ##### Others  Storage Drivers(How files are written) &  Logging Drivers (Where terminal outputs go)

    ---
    #####  Creating Custom Networks with Specific Drivers
   **  let's create two separate Docker networks on your machine, each explicitly assigned a different driver using the -d (driver) flag
   ```
     # Create a network using the local virtual 'bridge' driver
      docker network create -d bridge local-bridge-net

     # Create a network using the 'host' driver (built-in, no creation needed, but we reference it)
   ```
1. Case 1 **Using the bridge Driver**
   The bridge driver sets up an isolated virtual room. The container gets its own hidden, private IP address. To access it from your computer, you must use port mapping (-p).
   ```
     docker run -d --name bridge-web --network local-bridge-net -p 8080:80 nginx
   ```
   What the Driver Does Behind the Scenes:?
   - It requests the Linux kernel to create a virtual network interface (veth pair)
   - It plugs one end into the container and the other into a virtual switch on your host machine.
   - It configures Linux iptables rules to route incoming traffic from your physical port 8080 down to the container's port 80.
     
      
2. Case 2: **Using the host Driver**
   Now, let's launch a second container but tell Docker to use the host network driver. This strips away all virtual isolation.
   ```
     docker run -d --name host-web --network host nginx
   ```
  What the Driver Does Behind the Scenes:
  1. It tells the Docker engine not to create a separate network namespace for this container.
  2. It places the container directly onto your actual computer's main network card
  3. The  application(eg Nginx) inside the container binds directly to your computer's real physical port 80
     
##### **macvlan** network driver is a unique, high-performance network driver in Docker 
It bypasses Docker's usual virtual routing entirely and assigns a unique, real physical MAC address to every container.
**Instead of hiding containers behind your host computer's IP address (like the default bridge driver), a macvlan driver connects containers directly to your physical home or office router. To your router, each container looks like an independent, physical computer plugged into the network with its own Ethernet cable.**
**Benefits:**
- Direct Network Routing: Containers get real, routable IP addresses directly from your local network's DHCP pool or assigned manually by you
- Extreme Performance: Bypassing Docker’s internal firewall rules (iptables) and virtual routing bridges removes network overhead, providing raw, near-native line speed.

  ######  1.Create the Macvlan Network
  ```
   docker network create -d macvlan \
   --subnet=192.168.1.0/24 \
   --gateway=192.168.1.1 \
   -o parent=eth0 my-macvlan-net
 ```
 - `-d macvlan`: Activates the macvlan network driver.
 - `--subnet & --gateway`: Matches the exact configuration of your physical router. 
 - `-o parent=eth0`: Tells Docker which physical network port on your real computer to tunnel this traffic through
```
######  2: Launch a Container on the Macvlan Network
```
 docker run -d \
  --name local-server \
  --network my-macvlan-net \
  --ip=192.168.1.55 \
  nginx
```
##### Example : create macvlan driver network ,subnet=192.168.0.0/24 ip-range=192.168.0.0/2 name is media
```
  docker network  create -d macvlan --subnet=192.168.0.0/24 --ip-range=192.168.0.0/24 media
```
--ip-range flag forces Docker to only allocate IPs from a strict, tiny sub-slice of your network, keeping your containers completely isolated from your main router's automated DHCP pool
   this avoids conflicting your router network
   
