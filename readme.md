# 1. Stateless Server
Stateless servers do not hold any state in memory. When you write HTTP servers, they typically do not maintain any in-memory variables. Instead, they rely on external storage, such as a database, to manage state.
### Advantages of Stateless Servers:
- **No Need for Stickiness:** Users can connect to any available server because there is no need to maintain a connection to a specific server. This makes load balancing straightforward.
- **Easy Autoscaling:** Stateless servers can easily scale up and down based on CPU usage. Traffic can be routed to any available server, making it simple to manage resources.
![image](https://github.com/user-attachments/assets/0cc2dd65-cac3-4c67-aba4-83041eb47145)
- Stateless Server Diagram
- In the diagram, users (u1 and u2) can connect to any instance of Backend1 or Backend2, which in turn interact with a Postgres database to manage state.
- **Why do we need backend, why can't we directly query the DB ?**
  Backend is needed for authentication and authorization, BD is responsible for providing you what belongs to you.


