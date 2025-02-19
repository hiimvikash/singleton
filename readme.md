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
## 1. First Principle (ugly way) : export simple object and keep on updating according to the moves.
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

### **`The Problem in the above approach is that a developer need to access the state variable directly and need to knows the structure of state variable completely to make any changes in the state. Can't we just abstract out and provide functions to the developer to perform any operation on state. ` **

## 2. Slightly better approach : CLASS AND OBJECT BASED 

**The GameManager class has the following methods:**
- addGame(game: Game): Adds a new game object to the games array.
- getGames(): Returns the array of games.
- addMove(gameId: string, move: string): Adds a move to the specified game identified by its gameId.
- logState(): Logs the current state of the games array to the console.
```js
// Gamemanager.ts
interface Game {
    id: string;
    whitePlayer: string;
    blackPlayer: string;
    moves: string[];
}

class GameManager {
    private games: Game[] = []; 
    // NO ONE CAN Directly access the variable like this : gameManagerInstance.games.push(...) from outside the class, THIS VARIABLE CAN BE ACCESED INSIDE CLASS ONLY so we will be exposing just the function to use.
  

    constructor(){
      this.games = [];
    }

    public addGame(game: Game) {
        this.games.push(game);
    }

    public getGames() {
        return this.games;
    }

    public addMove(gameId: string, move: string) {
        const game = this.games.find(game => game.id === gameId);
        if (game) {
            game.moves.push(move);
        }
    }

    public logState() {
        console.log(this.games);
    }
}

const gameManagerInstance = new GameManager();

export { gameManagerInstance };
```

#### `NOT TO DO :` creating separate instances of the GameManager class in every file that needs it. This approach can lead to potential inconsistencies and difficulties in managing the state, as multiple instances of the GameManager class will have their own separate state.

- Now you can use like this.

```js
// index.ts
import { gameManagerInstance } from "./GameManager";
import { startLogger } from "./logger";

startLogger();

setInterval(() => {
    gameManagerInstance.addGame({
        id: Math.random().toString(),
        "whitePlayer": "harkirat",
        "blackPlayer": "jaskirat",
        moves: []
    })
}, 5000)
```
**In this approach, a single instance of the GameManager class is created and exported from GameManager.ts. Other files (logger.ts and index.ts) import and use this shared instance, ensuring a consistent state across the application.**

## 3. Even better Approach : Singleton Pattern
The "Even Better" approach utilizes the Singleton pattern to completely prevent any developer from creating a new instance of the GameManager class. The Singleton pattern ensures that a **class has only one instance** and provides a global point of access to that instance.


- We can make the constructor private : but this will prevent in making even single instance of that class from outside.
- **Static methods and Static Variable :** 
  - This are related to class.
  - shared accross all the instance of class.
  - static methods can be called directly on the class.
**Static attributes -**
In JavaScript, the keyword static is used in classes to declare static methods or static properties. Static methods and properties belong to the class itself, rather than to any specific instance of the class. Here’s a breakdown of what this means
- **Example of a class with static attributes**
```js
class Example {
    static count = 0;

    constructor() {
        Example.count++;  // Increment the static property using the class name
    }
}

let ex1 = new Example();
let ex2 = new Example();
console.log(Example.count);  // Outputs: 2
```
## Final Singleton Code

```js
interface Game {
    id: string;
    whitePlayer: string;
    blackPlayer: string;
    moves: string[];
}

export class GameManager {
    private static instance: GameManager; // Create a static instance of the class
    private games: Game[] = [];

    private constructor() {
        // Private constructor ensures that a new instance cannot be created from outside
        this.games = [];
    }

    public static getInstance(): GameManager {
        if (!GameManager.instance) {
            GameManager.instance = new GameManager();
        }
        return GameManager.instance;
    }

    public addGame(game: Game) {
        this.games.push(game);
    }

    public getGames() {
        return this.games;
    }

    public addMove(gameId: string, move: string) {
        const game = this.games.find(game => game.id === gameId);
        if (game) {
            game.moves.push(move);
        }
    }

    public logState() {
        console.log(this.games);
    }
}
```
- logger.ts
```js
import { GameManager } from "./GameManager";

export function startLogger() {
    setInterval(() => {
        GameManager.getInstance().logState();
    }, 4000)
}
```

- index.ts
```js
import { GameManager } from "./GameManager";
import { startLogger } from "./logger";

startLogger();

setInterval(() => {
    GameManager.getInstance().addGame({
        id: Math.random().toString(),
        "whitePlayer": "harkirat",
        "blackPlayer": "jaskirat",
        moves: []
    })
}, 5000)
```



