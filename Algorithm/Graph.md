# Graph

[TOC]

## Basic Concepts

### Point (or Node, Vertex)

```java
class Node {
    int value;
    int in; // incoming degree
    int out; // outgoing degree
    ArrayList<Node> nexts;// Nodes point to
    ArrayList<Edge> edges;// edges from this Node

    Node(int value) {
        this.value = value;
        in = 0;
        out = 0;
        nexts = new ArrayList<>();
        edges = new ArrayList<>();
    }
}
```

### Edge

```java
class Edge {
    int weight;
    Node from;
    Node to;

    Edge(int weight, Node from, Node to) {
        this.weight = weight;
        this.from = from;
        this.to = to;
    }
}
```

### Graph

```java
class Graph {
    HashMap<Integer,Node> nodes;
    HashSet<Edge> edges;

    Graph() {
        nodes = new HashMap<>();
        edges = new HashSet<>();
    }
}
```

If the input is a matrix, each line represents [fromNode, toNode, edgeWeight]. The graph can be generated by:

```java
Graph createGraph(Integer[][] matrix) {
    Graph graph = new Graph();
    for (int i = 0; i < matrix.length; i++) {
        Integer from = matrix[i][0];
        Integer to = matrix[i][1];
        Integer weight = matrix[i][2];

        // generate nodes
        if (!graph.nodes.containsKey(from)) {
            graph.nodes.put(from, new Node(from));
        }
        if (!graph.nodes.containsKey(to)) {
            graph.nodes.put(to, new Node(to));
        }
        Node fromNode = graph.nodes.get(from);
        Node toNode = graph.nodes.get(to);

        // generate edges
        Edge newEdge = new Edge(weight, fromNode, toNode);
        fromNode.nexts.add(toNode);
        fromNode.out++;
        toNode.in++;
        fromNode.edges.add(newEdge);
        graph.edges.add(newEdge);
    }
    return graph;
}
```

## Storage Mode

### Adjacency List

An adjacency list is a collection of unordered lists used to represent a finite graph. Each list describes the set of neighbors of a [vertex](https://en.wikipedia.org/wiki/Vertex_(graph_theory)) in the graph

eg. ![](http://img.blog.csdn.net/20130429141605716)

### Adjacency Matrix

An adjacency matrix is a square matrix used to represent a finite graph. The elements of the matrix indicate whether pairs of vertices are adjacent or not in the graph

eg.

|                                          |                                          |
| ---------------------------------------- | ---------------------------------------- |
| ![](https://upload.wikimedia.org/wikipedia/commons/thumb/2/28/6n-graph2.svg/300px-6n-graph2.svg.png) | ![](https://wikimedia.org/api/rest_v1/media/math/render/svg/a773011024de5e3cbe8da03e97c79e1fe3101937) |

### Comparation

|          | Adjacency Matrix                         | Adjacency List                           |
| -------- | ---------------------------------------- | ---------------------------------------- |
| Merits   | Easy to judge if there is an edge between 2 nodes; Easy to add or remove edges | Space-saving, suitable for sparse graph  |
| Demerits | Unsuitable for sparse  graph             | Discommodious to compute incoming degree or outgoing degree of a vertex; Have to traverse twice to remove an edge |



## BreadthFirstSearch

```java
void bfs(Node node) {
    if (node == null) return;
    Queue<Node> queue = new LinkedList<>();// nodes to be visited
    HashSet<Node> set = new HashSet<>();// visited nodes
    queue.add(node);
    set.add(node);
    while (!queue.isEmpty()) {
        Node cur = queue.poll();
        // print cur

        for (Node next: cur.nexts) {
            if (!set.contains(next)) {
                set.add(next);
                queue.add(next);
            }
        }
    }
}
```

## DepthFirstSearch

```java
void dfs(Node node) {
    if (node == null) return;
    Stack<Node> stack = new Stack<>();// nodes to be visited
    HashSet<Node> set = new HashSet<>();// visited nodes
    stack.add(node);
    set.add(node);
    while (!stack.isEmpty()) {
        Node cur = stack.pop();
        // print cur

        for (Node next: cur.nexts) {
            if (!set.contains(next)) {
                stack.push(cur);// use this cur to find other paths from cur in further
                stack.push(next);
                set.add(next);
                // once found a path from cur, break to make sure depth-first
                break;
            }
        }
    }
}
```

## TopologySort

A sorted node list, there are only edges from the latter ones to the front ones

Only directed acyclic graph with at least one 0-in-degree node  have its TopologySort

For example, the adjacency list of a directed graph is:

```
A: [B, C]
B: [C]
C: [D]
D:
```

The TopologySort of this graph is [A, B, C, D]

TopologySort can be used to solve dependency problems in applications' installation

eg. Installing B needs to installing A before. In other words, A supports B and C. C needs B and D needs C. Therefore the right procedure is to install A, B, C, D orderly

```java
List<Node> sortedTopology(Graph graph) {
    HashMap<Node, Integer> inMap = new HashMap<>();
    Queue<Node> zeroInQueue = new LinkedList<>();
    for (Node node: graph.nodes.values()) {
        inMap.put(node, node.in);
        if (node.in == 0) zeroInQueue.add(node);
    }
    List<Node> result = new ArrayList<>();
    while (!zeroInQueue.isEmpty()) {
        // add 0-in nodes to result
        Node cur = zeroInQueue.poll();
        result.add(cur);
        for (Node next: cur.nexts) {
            // decrease all adjacent nodes's in-degree in inMap by 1
            inMap.put(next, inMap.get(next) - 1);
            if (inMap.get(next) == 0) zeroInQueue.add(next);
        }
    }
    return result;
}
```

More about [TopologySort](https://zh.wikipedia.org/zh-cn/%E6%8B%93%E6%92%B2%E6%8E%92%E5%BA%8F)

## Union Find

Some nodes are given: [1, 2, 3, 4, ...]

each of them is in one set: [1, 3], [2], [4, 5], [6, 13, 26], [7, 22]

For example, in a graph, the nodes can be seen as points, the sets can be seen as the connected components 

### Auxiliary values

```java
class Node {
  // definition
}
HashMap<Node, Node> rootMap;// (node, the root of node's set tree)
HashMap<Node, Integer> sizeMap;// (node, the size of node's set tree)
```

### Initialize sets

every set is like a tree, with one of its node as the root ndoe

```java
void init(List<Node> nodes) {
    rootMap.clear();
    sizeMap.clear();
    for (Node node: nodes) {
        // every node as a set which only contains itself
        rootMap.put(node, node);
        sizeMap.put(node, 1);
    }
}
```

### find()

return a's root node

```java
boolean isSameRoot(Node a, Node b) {
    return find(a) == find(b);
}

Node find(Node node) { // return the root of node's set tree
    Node root = rootMap.get(node);
    if (root != node) {
        // use recursion to find the origin root which node == node's root
        root = find(root);
    }
    rootMap.put(node, root);
    return root;
}
```

### union()

union a's set with b's set

```java
void union(Node a, Node b) {
    if (a == null || b == null) return;
    Node aRoot = find(a);
    Node bRoot = find(b);
    if (aRoot == bRoot) return;
    int aSetSize = sizeMap.get(aRoot);
    int bSetSize = sizeMap.get(bRoot);
    if (aSetSize <= bSetSize) {
        rootMap(aRoot, bRoot);
        sizeMap(bRoot, aSetSize + bSetSize);
    } else {
        rootMap(bRoot, aRoot);
        sizeMap(aRoot, aSetSize + bSetSize);
    }
}
```

> If the size of nodes is N, find() K times, union() F times, and (K + F) is $$O(K+F)$$, find() and union() can be done both in $$O(1)$$ time averagely
>
> In fact, the time complexity is $$O(\alpha(N))$$. $$\alpha(N)$$ is the *Inverse-Ackermann* function which increases extremely slow. When N is close to $$10^{80}$$, $$\alpha(N)$$ only return a number between 5 and 6. Therefore, the time complexiy is very close to $$O(1)$$ when N is very large. Be careful it won't be done in $$O(1)$$ time if N is not large enough.

### Simple Implementation

```java
class SimpleUnionFind {
    private final static int MAX_SIZE = 1001;
    int[] parent;
    int[] rank;

    public SimpleUnionFind() {
        parent = new int[MAX_SIZE];
        rank = new int[MAX_SIZE];
        for (int i = 0; i < parent.length; i++) {
            parent[i] = i;
        }
    }

    public int find(int a) {
        if (parent[a] != a) {
            parent[a] = find(parent[a]);
        }
        return parent[a];
    }

    public boolean isSameParent(int a, int b) {
        return find(a) == find(b);
    }

    public void union(int a, int b) {
        int ap = find(a);
        int bp = find(b);
        if (ap == bp) {
            return;
        }

        if (rank[ap] < rank[bp]) {
            parent[ap] = bp;
            rank[bp] += rank[ap];
        } else {
            parent[bp] = ap;
            rank[ap] += rank[bp];
        }
    }
}
```

## Minimum spanning tree

a subset of the edges of a connected, edge-weighted (un)directed graph that connects all the nodes together

Kruskal and Prim are both to generate a minimum spanning tree of an undirected graph, which has no loop.

### Kruskal

It is a greedy algorithm in graph theory as it finds a minimum spanning tree for a connected weighted graph adding increasing cost edges at each step. $$O(ElogE)$$

Use UnionFind to detect if a loop will be generated by adding one edge in the process

[Example](https://en.wikipedia.org/wiki/Kruskal%27s_algorithm#Example)

```java
Set<Edge> kruskalMST(Graph graph) {
    UnionFind unionFind = new UnionFind();
    unionFind.init(graph.nodes.values());
    // sort edges by weight
    PriorityQueue<Edge> queue = new PriorityQueue<>((a, b) -> a.weight - b.weight);
    for (Edge edge: graph.edges) queue.offer(edge);

    Set<Edge> result = new HashSet<>();
    while (!queue.isEmpty() || result.size < graph.nodes.size - 1) {
        // find the shortest edge in the queue
        Edge edge = queue.poll();
        if (!unionFind.isSameRoot(edge.from, edge.to)) {
            // if there is no path from edge.from to edge.to, a loop won't be generated by adding this edge 
            result.add(edge);
            unionFind.union(edge.from, edge.to);
        }
    }
    return result;
}
```

### Prim

It operates by building this tree one node at a time, from an arbitrary starting node at each step adding the cheapest possible edge from the tree to another node.$$O(E+VlogV)$$

```java
Set<Edge> primMST(Graph graph) {
    PriorityQueue<Edge> queue = new PriorityQueue<>((a, b) -> a.weight - b.weight);
    HashSet<Node> set = new HashSet<>();
    Set<Edge> result = new HashSet<>();
    for (Node node: graph.nodes.values()) {
        // from one node, start build a tree
        if (set.contains(node)) continue;
        set.add(node);
        // add edges from the node
        for (Edge edge: node.edges) {
            queue.offer(edge);
        }
        while (!queue.isEmpty()) {
            // in every loop, find the shortest edge to connect to the nodes not in the set 
            Edge edge = queue.poll();
            Node toNode = edge.to;
            if (!set.contains(toNode)) {
                set.add(toNode);
                result.add(edge);
                // add edges from the new added toNode
                for (Edge nextEdge: toNode.edges) {
                    queue.offer(nextEdge);
                }
            }
        }
    }
}
```

## Dijkstra

find the shortest paths between nodes in a connected, directed graph with non-negative edges

From a head node, compute the distances from the head to other nodes.

```java
HashMap<Node, Integer> dijkstra(Node head) {
    HashMap<Node Integer> distanceMap = new HashMap<>(); // record distances from head to other nodes
    // head to head: defalut 0
    distanceMap.put(head, 0);
    HashSet<Node> selectedNodes = new HashSet<>();

    // find the closest and unseleted node
    Node minNode = getMinDistanceAndUnselectedNode(distanceMap, selectedNodes);
    while (minNode != null) {
        // record distance from head to minNode
        int distance = distanceMap.get(minNode);
        for (Edge edge: minNode.edges) {
            // from minNode, update every adjacent nodes' distance
            Node toNode = edge.to;
            if (!distanceMap.containsKey(toNode)) distanceMap.put(toNode, distance + edge.weight);
            distanceMap.put(toNode, Math.min(distanceMap.get(edge.to), distance + edge.weight));
        }
        selectedNodes.add(minNode);
        // find the closest and unseleted node
        minNode = getMinDistanceAndUnselectedNode(distanceMap, selectedNodes);
    }
    return distanceMap;
}

Node getMinDistanceAndUnselectedNode(HashMap<Node, Integer> distanceMap, HashSet<Node> selectedNodes) {
    Node minNode = null;
    int minDistance = Integer.MAX_VALUE;
    for (Entry<Node, Integer> entry: distanceMap.entrySet()) {
        // find the node which its distance is the shortest and not in selectedNodes
        Node node = entry.getKey();
        int distance = entry.getValue();
        if (!selectedNodes.contains(node) && distance < minDistance) {
            minNode = node;
            minDistance = distance;
        }
    }
}
```

The distance update and findMin operation can be optimized by using a customized min heap

```java
class NodeRecord { // node with its distance from a head node to it
    Node node;
    int distance;
    NodeRecord(Node node, int distance) {
        this.node = node;
        this.distance = distance;
    }
}

class NodeHeap {
    Node[] nodes; // all nodes in the graph
    HashMap<Node, Integer> heapIndexMap; // (node, its index in the array)
    HashMap<Node, Integer> distanceMap; //  (node, its distance from a head node to it)
    int size; // heap size

    NodeHeap(int size) {
        nodes = new Node[size];
        heapIndexMap = new HashMap<>();
        distanceMap = new HashMap<>();
        this.size = 0;
    }

    boolean isEmpty() {
        return size == 0;
    }

    void addOrUpdateOrIgnore(Node node, int distance) {
        // add a node || update a node's distance and heapify || ignore
        if (inHeap(node)) { // update
            // if the node is in the heap, update its distance and heapify
            // because the update operation will only reduce the distance, so only swim() happens
            distanceMap.put(node, Math.min(distanceMap.get(node), distance));
            swim(node, heapIndexMap.get(node));
        }
        if (!isEntered(node)) { // add
            // add a node into the heap and heapify
            nodes[size] = node;
            heapIndexMap.put(node, size);
            distanceMap.put(node, distance);
            swim(node, size++);
        }
    }

    void swim(Node node, int index) {
        while (distanceMap.get(nodes[index]) < distanceMap.get(nodes[(index - 1) / 2])) {
            swap(index, (index - 1) / 2);
        }
    }

    void sink(int index, int size) {
        int left = index * 2 + 1;
        while (left < size) {
            // make sure nodes[left].distance < nodes[left + 1].distance
            if (left + 1 < size && distanceMap.get(nodes[left + 1]) < distanceMap.get(nodes[left])) {
                swap(left, left + 1);
            }
            // don't need to swap again
            if (distanceMap.get(index) <= distanceMap.get(left)) break;
            swap(left, index);
            index = left;
            left = index * 2 + 1;
        }
    }

    NodeRecord poll() {
        // poll a node with the shortest distance
        NodeRecord nodeRecord = new NodeRecord(nodes[0], distanceMap.get(nodes[0]));
        swap(0, size - 1);
        heapIndexMap.put(nodes[size - 1], -1);
        distanceMap.remove(nodes[size - 1]);
        nodes[size - 1] = null;
        sink(0, --size);
        return nodeRecord;
    }

    boolean isEntered(Node node) {
        return heapIndexMap.containsKey(node);
    }

    boolean inHeap(Node node) {
        return isEntered(node) && heapIndexMap.get(node) != -1;
    }

    void swap(int i, int j) {
        heapIndexMap.put(nodes[i], j);
        heapIndexMap.put(nodes[j], i);
        Node tmp = nodes[i];
        nodes[i] = nodes[j];
        nodes[j] = tmp;
    }
}

HashMap<Node, Integer> dijkstra(Node head, int size) {
    NodeHeap nodeHeap = new NodeHeap(size);
    nodeHeap.addOrUpdateOrIgnore(head, 0);
    HashMap<Node, Integer> result = new HashMap<>();
    while (!nodeHeap.isEmpty) {
        NodeRecord record = nodeHeap.poll();
        Node cur = record.node;
        int distance = record.dsitance;
        for (Edge edge: cur.edges) {
            nodeHeap.addOrUpdateOrIgnore(edge.to, edge.weight + distance);
        }
        result.put(cur, distance);
    }
    return result;
}
```

More about [Dijkstra](https://en.wikipedia.org/wiki/Dijkstra%27s_algorithm)

## Practice

### MinCostToTakeRoads(Facebook)

> There is a graph of roads and cities. Every city of the graph is A-type or B-type with a cost value. If we take a city, the neighbored roads will be taken. If the neighbored cities are the same type as the taken one, these cities will be taken too. Compute the minimum cost to take all roads.
>
> eg. If a city A(4) connects 3 cities B(1), the min cost is 3 (take all B's roads). If A's cost become 2, the result is 2 (take A).
>
> If A1 is connected with A2 and A1 is taken, the road connected to A1 and A2 will be taken including A1A2.

Greedy solution. The cost-performance of a city is (the cost of the city / the current connected untaken roads). Use this to build a min heap and compute the total cost

Refer to Knapsack in [RecursionDP.md](RecursionDP.md)

### [Longest Increasing Path in a Matrix](https://leetcode.com/problems/longest-increasing-path-in-a-matrix/description/)

> Given an integer matrix, find the length of the longest increasing path.
>
> From each cell, you can either move to four directions: left, right, up or down. You may NOT move diagonally or move outside of the boundary (i.e. wrap-around is not allowed).

DFS solution:

1. From every node, use DFS to find an increasing path
2. Use cache to memorize previous DFS results

```java
private static int[][] direction = { {-1, 0}, {1, 0}, {0, -1}, {0, 1} };
public int longestIncreasingPath(int[][] matrix) {
    if (matrix == null || matrix.length == 0 || matrix[0].length == 0) {
        return 0;
    }

    int res = 1;
    int[][] cache = new int[matrix.length][matrix[0].length];
    for (int i = 0; i < matrix.length; i++) {
        for (int j = 0; j < matrix[0].length; j++) {
            res = Math.max(res, dfs(matrix, i, j, cache));
        }
    }
    return res;
}

private int dfs(int[][] matrix, int i, int j, int[][] cache) {
    if (!isValid(matrix, i, j)) {
        return 0;
    }

    if (cache[i][j] != 0) {
        return cache[i][j];
    }

    cache[i][j] = 1;
    for (int[] d: direction) {
        int x = i + d[0];
        int y = j + d[1];
        if (isValid(matrix, x, y) && matrix[x][y] > matrix[i][j]) {
            int count = dfs(matrix, x, y, cache) + 1;
            cache[i][j] = Math.max(cache[i][j], count);
        }
    }
    return cache[i][j];
}

private boolean isValid(int[][] matrix, int x, int y) {
    return x >= 0 && x < matrix.length && y >= 0 && y < matrix[0].length;
}
```




