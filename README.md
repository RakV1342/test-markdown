# CSE 571 Group Project
## Team 8 - Topic 2: Liflong Planning

Team Members and Contribution
---
Rakshith Vishwanatha
* D\* Lite algorithm implementation for 5 node example cluster
* D\* Lite algorithm implementation for Packman Environment

Nicholas Downey
* A\* algorithm for baseline performance measurement
* Modification of Pacman environment for Local Observability
* Statistics and Data Collection

Nishanth BM
* Wrote LPA\* Algorithm for 5 node cluster
* Tested and validated D\* Lite algorithm on 5 node cluster
* Worked on report

Jayasurya SM
* Tested and validated D\* Lite algorithm on 5 node cluster
* Worked on report

This project works on comparing the performance between heuristic search like A\* and an incremental search algorithm like D\* Lite.  The algorithm has been implemented for the Pacman environment. Modifications have been made to the environemnt to allow for:

1. Local environment sensing by the robot and retention of information about walls/obstacles over time.
2. D\* Lite algorithm and retention of learned path and priority queue over time.
3. Data and Statistics collection for offline analysis.


## Usage

Go to the ```pacman_incremental_search``` folder and run the following commands:

### A\* Commands
```
pacman.py -l tinyMaze -z .5 -p SearchAgent -a fn=astar,heuristic=manhattanHeuristic
pacman.py -l mediumMaze -z .5 -p SearchAgent -a fn=astar,heuristic=manhattanHeuristic
pacman.py -l bigMaze -z .5 -p SearchAgent -a fn=astar,heuristic=manhattanHeuristic
```

### D\* Commands
```
pacman.py -l tinyMaze -z .5 -p SearchAgent -a fn=astar,heuristic=manhattanHeuristic
pacman.py -l mediumMaze -z .5 -p SearchAgent -a fn=astar,heuristic=manhattanHeuristic
pacman.py -l bigMaze -z .5 -p SearchAgent -a fn=astar,heuristic=manhattanHeuristic
```
