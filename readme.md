# 1. Stateless Server
Stateless servers do not hold any state in memory. When you write HTTP servers, they typically do not maintain any in-memory variables. Instead, they rely on external storage, such as a database, to manage state.
### Advantages of Stateless Servers:
- **No Need for Stickiness:** Users can connect to any available server because there is no need to maintain a connection to a specific server. This makes load balancing straightforward.
- **Easy Autoscaling:** Stateless servers can easily scale up and down based on CPU usage. Traffic can be routed to any available server, making it simple to manage resources.
![image](https://github.com/user-attachments/assets/0cc2dd65-cac3-4c67-aba4-83041eb47145)
- Stateless Server Diagram
- In the diagram, users (u1 and u2) can connect to any instance of Backend1 or Backend2, which in turn interact with a Postgres database to manage state.
- **Why do we need backend, why can't we directly query the DB ?**
  Backend is needed for authentication and authorization basically backend make sure that user is able to access their own data only, not someone else data.

# 2. Stateful Server
Stateful servers hold state within the server's memory. This means that the server maintains in-memory variables that are used to manage the state of the application.

Example:-
- **In-Memory Cache:** Creating an in-memory cache to store frequently accessed data for improved performance. [Example Code](https://github.com/code100x/cms/blob/e905c71eacf9d99f68db802b24b7b3a924ae27f1/src/db/Cache.ts#L3)
  - This is not usefull if you have multiple instance of server because everytime user may connects to different server.
- **Real-Time Game State:** Storing the state of a game in memory for real-time multiplayer games. [Example Code](https://github.com/code100x/chess/blob/main/apps/ws/src/Game.ts#L41-L47)
  - Let understand the **problem if we use stateless backend in Realtime games :-**
    - You are playing chess and **you made a move `e4 e5`**, to validate this move you will **fetch the last board moves configuration** from server > **validate the made move** > then **store to server.**
    - This process will leads to lots of latency.

## Stickiness
In cases where the server holds state, there is a need for stickiness. Stickiness ensures that the user who is interested in a specific room or game state gets connected to the specific server that holds the relevant state.
- If two players are playing chess, they must connect to same server.
- If by any reason a player refreshes the game or something then stickiness ensures it should make him join the same server, as it will be having in memory state of *room1* storing the moves played till now(this will help in restoring the game).
## Let's Understand how to maintain STICKINESS : How application like GatherTown maintain stickiness with scalling?
- There is one **ROUTER** server : It has info about which instance of servers are healthy and which server contains which rooms.
  - example : let's say GatherTown scaled it server to 3 - [gt-be1, gt-be2, gt-be3] and this 3 server keep pinging in every 5min to routerServer that they're healthy.
- Now a user come and say I want to join room3, router will check if there already exist any room3, if yes then it will make the user join that server with room3 in it
- orelse it will create a new room in the server which has less cpu usuage and directs the user to that server.
![image](https://github.com/user-attachments/assets/caac5fb0-3971-44b8-aa25-cbf2ff257813)

# 3. How to do STATE MANAGEMENT IN BACKEND.
## First Principle (ugly way) : export simple object and keep on updating according to the moves.
![image](https://github.com/user-attachments/assets/7be9bfb7-7b44-4c51-b8b2-76df9972d28b)

![image](https://github.com/user-attachments/assets/7a093835-cba5-412b-8f1d-2b52763dbdfb)

When managing state in a JavaScript process, it's common to store state in a way that can be accessed and modified by multiple files. Here is an example approach:
- **`index.ts` - Pushes to games array:**
```js
import { games } from "./store";
import { startLogger } from "./logger";

startLogger();

setInterval(() => {
    games.push({
        "whitePlayer": "harkirat",
        "blackPlayer": "jaskirat",
        moves: []
    })
}, 5000)
```
- **`logger.ts` - Uses the games array to sync DB with in-memory state.**
```js
import { games } from "./store";

export function startLogger() {
    setInterval(() => {
        console.log(games);
    }, 4000)
}
```
- **`store.ts` - Exports the game array:**
```js
interface Game {
    whitePlayer: string;
    blackPlayer: string;
    moves: string[];
}

export const games: Game[] = [];
```




