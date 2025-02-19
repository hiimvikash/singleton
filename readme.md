# 1. Stateless Server
Stateless servers do not hold any state in memory. When you write HTTP servers, they typically do not maintain any in-memory variables. Instead, they rely on external storage, such as a database, to manage state.
### Advantages of Stateless Servers:
- **No Need for Stickiness:** Users can connect to any available server because there is no need to maintain a connection to a specific server. This makes load balancing straightforward.
- **Easy Autoscaling:** Stateless servers can easily scale up and down based on CPU usage. Traffic can be routed to any available server, making it simple to manage resources.