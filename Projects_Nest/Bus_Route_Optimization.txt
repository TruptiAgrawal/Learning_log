Problem statement :
------------------  
I am working on a bus route optimization problem where we will be considering three main aspects
1. nodes = bus stops
2. edges = Cost ( Time taken between each stop)
3. Number of students boarding per stop 

The goal is to find an optimal set of routes that minimizes a specific objective that is to minimize the time taken. 

The available dataset includes the following information:
*   A comprehensive list of all bus stops.
*   The number of students assigned to each bus stop.
*   The average travel time between each pair of adjacent stops.

Constraints:
*   The bus stop locations are constant and do not change.
*   The routes and their associated costs (travel times) are the variables that can be modified and optimized WE ARE NOT CONSIDERING THE DISTANCES. 

understanding of this problem : 
----------------------------- 
-> I am essentially solving a graph optimization problem, specifically a route optimization / vehicle routing problem (VRP) variant with time minimization as the main objective.

-> This is a combinatorial optimization problem similar to:
1. Traveling Salesman Problem (TSP) - 1 bus 
2. Capacitated Vehicle Routing Problem (CVRP) - N buses  
Both are NP-hard, meaning exact solutions may be computationally expensive for large networks, so heuristics or metaheuristics (e.g., genetic algorithms, simulated annealing, tabu search) are often used.

Approach followed : 
-----------------
Phase 1: Data Collection
Phase 2: Data Structure Creation
Phase 3: Algorithm Implementation
Phase 4: Optimized Routes 

Algorithms Considered : 
---------------------
1. Genetic Algorithm
2. Dijkstra's Algorithm 
3. Floyd warshall's Algorithm 
4. Clarke and wright's Algorithm 
5. Simulated annealing 

My part so far : 
--------------- 
- Explored Genetic Algorithm 
- Explored how is that relavant to the project
- how can it be implemented and the role of each function 
- Visual debugging screen to potray the same 

References ? 
---------- 
! https://en.wikipedia.org/wiki/Travelling_salesman_problem 
! https://www.geeksforgeeks.org/dsa/types-of-complexity-classes-p-np-conp-np-hard-and-np-complete/ 
! https://en.wikipedia.org/wiki/Traveling_purchaser_problem 
! https://en.wikipedia.org/wiki/Vehicle_routing_problem
! https://en.wikipedia.org/wiki/Combinatorial_optimization 
! https://en.wikipedia.org/wiki/Integer_programming 
! https://en.wikipedia.org/wiki/Linear_programming 
! https://en.wikipedia.org/wiki/Genetic_algorithm 
! https://en.wikipedia.org/wiki/Evolutionary_algorithm 
! https://en.wikipedia.org/wiki/Metaheuristic 
! https://en.wikipedia.org/wiki/Dijkstra%27s_algorithm 