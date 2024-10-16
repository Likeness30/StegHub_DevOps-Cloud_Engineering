# HTTP LOAD BALANCING METHOD
Load balancing is a method used to keep web applications running smoothly by spreading traffic across multiple servers. This improves resource use, reduces delays, and makes systems more reliable. NGINX and NGINX Plus make load balancing easy, acting as powerful HTTP load balancers in various setups.
To start load balancing with NGINX, you first create a group of servers, called an "upstream group," that share the incoming traffic. The traffic is directed based on rules. For example, if one server is too busy, another one with less traffic can handle the requests. This is where different load-balancing methods are used.

## Load Balancing Methods
* Round Robin (default): Sends requests evenly to all servers.
* Least Connections: Directs traffic to the server with the fewest active connections.
* IP Hash: Ensures requests from the same IP address go to the same server, ensuring consistency.
* Generic Hash: Routes requests based on custom data like the request URI.
* Least Time (NGINX Plus): Sends traffic to the server with the fastest response times.
* Random: Selects servers randomly, useful in complex environments.
You can also adjust these methods by giving more weight to certain servers (so they handle more traffic) or setting up backup servers to take over if the main servers fail.

## Additional Features
* Session Persistence: Keeps all requests from a user going to the same server, especially useful for apps that store user data.
* Server Slow-Start: Helps a recovering server gradually return to handling traffic without being overloaded.
* Connection Limits: Limits the number of connections to each server and queues extra requests if limits are reached.
NGINX Plus offers extra features like health checks, which automatically monitor servers and remove failing ones until they recover.

## In Summary
NGINX ensures your applications stay available, responsive, and scalable, no matter how large your setup gets.
