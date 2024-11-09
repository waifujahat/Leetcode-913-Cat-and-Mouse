# Leetcode-913-Cat-and-Mouse
Solution for waifujahat | C# | 100% both using BFS

![Screenshot 2024-11-09 140928.png](https://assets.leetcode.com/users/images/e181f582-fa02-49be-864c-783423f0d37f_1731136960.6193707.png)

---


# Intuition
I can be visualized as a game played on an undirected graph where two players, a mouse and a cat, move along the graph, the game ends in one of three ways, here

1. Mouse wins when the mouse reaches node 0 (the hole)
2. Cat wins when the cat catches the mouse
3. Draw when the game reaches a repetitive state where both players are in the same position as before and the game continues indefinitely

So I determine, based on the graph and the starting positions of the mouse and cat, whether the mouse wins, the cat wins, or the result is a draw, assuming both players play optimally


---


# Approach

1. I can represent the game state as a triplet, (mouse_position, cat_position, turn), where turn can be either 0 (for the mouse’s turn) or 1 (for the cat’s turn), then my goal is to compute the result for the state (1, 2, 0), which is where the game starts with the mouse at node 1 and the cat at node 2

2. I can use a DP table `dp[mouse_position][cat_position][turn]` where `dp[mouse_position][cat_position][turn]` stores the result for a given state, like 1 means the mouse wins from this state, 2 means the cat wins from this state, 0 means the game is a draw from this state

3. I assume, if the mouse reaches the hole (node 0), it wins, if the cat catches the mouse (they are on the same node), the cat wins, then if a state has been encountered before, it means the game is a draw, initialize the dp table based on these conditions

4. The players will alternate turns so the mouse can move to any neighboring node except node 0 and the cat can move to any node except node 0, if it’s the mouse’s turn, the mouse tries to avoid the cat while heading toward node 0 (the hole) and if it’s the cat’s turn, the cat tries to catch the mouse by moving to the mouse's position or blocking its path

5. I use BFS approach with a queue to process all possible states, so for each state, I check if the game is over (either a win or a draw), and update the dp table accordingly, if neither player has won, the game continues to the next state based on the players possible moves


---


# Code
```csharp []
public class Solution {
    public int CatMouseGame(int[][] graph) {

    <Summary>
    //Mouse starts at node 1
    //Cat starts at node 2
    //Mouse wins if it reaches node 0 (the hole)
    //Cat wins if it catches the mouse (they meet on the same node)
    //The game continues until either the mouse wins, the cat wins, or it results in a draw

        //First, I create list of nodes where each node has a list of adjacent nodes (neighbors) and the way access to the neighbors of each node
        int numberOfNodes = graph.Length;

        var graphList = new List<int>[numberOfNodes];

        for (int i = 0; i < numberOfNodes; i++)
        {
            graphList[i] = new List<int>(graph[i]);
        }

        //Identifying Nodes with Edges to the Hole (Node 0)
        bool[] hasEdgeToHole = new bool[numberOfNodes];

        for (int i = 1; i < numberOfNodes; i++)
        {
            foreach (var neighbor in graphList[i])
            {
                if (neighbor == 0)
                {
                    hasEdgeToHole[i] = true;
                }
            }
        }

        //I create 3D table gameResults to store the results of the game for every possible state
        //Mouse's position, Cat's position and whose turn it is (0 for mouse, 1 for cat)
        //1 for Mouse wins from this state, 2 for Cat wins from this state, 0 for Draw and -1 if State not yet computed
        int[,,] gameResults = new int[numberOfNodes, numberOfNodes, 2];

        for (int i = 0; i < numberOfNodes; i++)
        {
            for (int j = 0; j < numberOfNodes; j++)
            {
                gameResults[i, j, 0] = gameResults[i, j, 1] = -1;
            }
        }

        int[,,] visitCounter = new int[numberOfNodes, numberOfNodes, 2];

        //I use a queue to process states using BFS, BFS will ensures the process states level by level, from base cases to the rest of the states
        Queue<(int mousePos, int catPos, int turn)> gameQueue = new Queue<(int, int, int)>();

        //Mouse wins if it reaches node 0
        for (int i = 1; i < numberOfNodes; i++)
        {
            gameResults[0, i, 1] = 1;
            gameResults[0, i, 0] = 1;
            gameQueue.Enqueue((0, i, 1)); 
            gameQueue.Enqueue((0, i, 0));
        }

        //Cat wins if it catches the mouse (both are at the same node)
        for (int i = 1; i < numberOfNodes; i++)
        {
            gameResults[i, i, 0] = 2;
            gameResults[i, i, 1] = 2;
            gameQueue.Enqueue((i, i, 0)); 
            gameQueue.Enqueue((i, i, 1)); 
        }

        //Processing the Game States
        //If it’s the mouse's turn, it moves to a neighboring node, avoiding the cat if possible
        //If it’s the cat's turn, it tries to catch the mouse by moving closer to the mouse
        while (gameQueue.Count > 0)
        {
            var (mousePos, catPos, turn) = gameQueue.Dequeue();
            int result = gameResults[mousePos, catPos, turn];

            // If result is already known, continue
            if (gameResults[1, 2, 0] != -1)
            {
                break;
            }

            // Mouse's turn
            if (turn == 0)
            {
                foreach (var neighbor in graphList[catPos])
                {
                    // Skip the hole node (mouse can't move there)
                    if (neighbor == 0) 
                    continue;

                    // Skip if result is already known
                    if (gameResults[mousePos, neighbor, 1] != -1) 
                    continue;

                    if (result == 2)
                    {
                        gameResults[mousePos, neighbor, 1] = result;
                        gameQueue.Enqueue((mousePos, neighbor, 1));
                    }
                    else
                    {
                        visitCounter[mousePos, neighbor, 1]++;

                        int maxDegree = hasEdgeToHole[neighbor] ? graphList[neighbor].Count - 1 : graphList[neighbor].Count;

                        if (visitCounter[mousePos, neighbor, 1] >= maxDegree)
                        {
                            gameResults[mousePos, neighbor, 1] = 1;
                            gameQueue.Enqueue((mousePos, neighbor, 1));
                        }
                    }
                }
            }
            else // Cat's turn
            {
                foreach (var neighbor in graphList[mousePos])
                {
                    // Skip if result is already known
                    if (gameResults[neighbor, catPos, 0] != -1) 
                    continue;

                    // If mouse wins, update result to 2 for cat's turn
                    if (result == 1)
                    {
                        gameResults[neighbor, catPos, 0] = result;
                        gameQueue.Enqueue((neighbor, catPos, 0));
                    }
                    else //I track how many times a country is visited during a cat's turn
                    {
                        visitCounter[neighbor, catPos, 0]++;

                        int maxDegree = graphList[neighbor].Count;

                        if (visitCounter[neighbor, catPos, 0] == maxDegree)
                        {
                            gameResults[neighbor, catPos, 0] = 2;
                            gameQueue.Enqueue((neighbor, catPos, 0));
                        }
                    }
                }
            }
        }
        if (gameResults[1, 2, 0] == -1)
        {
            return 0; //If no result is found, return draw
        }
        else
        {
            return gameResults[1, 2, 0];
        }
    }
}
