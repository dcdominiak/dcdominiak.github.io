---
title: "Lessons Learned During Udacity AIND, Part 1: Solving Puzzles, Game Playing Agents"
excerpt: "Part 1 of lessons learned during Udacity Artificial Intelligence Nanodegree covers the first part of Term 1."
layout: post
---

The Udacity Artificial Intelligence Nanodegree, or AIND for short, is a wide-ranging program that teaches you how to build AI systems. After buidling a Sudoku puzzle solver as a warm-up, it starts with a game playing agent for a simple (contrived) board game and ends with English-to-French neural network-based machine translation. In a series of posts, I will recount some of my key lessons learned and takeaways from the program.

Part 1: Solving Puzzles, Game Playing Agents (this post)
Part 2: Planning, Natural Language Recognition
Part 3: Deep Learning with Convolutional Neural Networks, Recurrent Neural Networks
Part 4: Interesting Deep Learning Applications
Part 5: NLP Capstone Project - Machine Translation

[Udacity Artificial Intelligence Nanodegree home page](https://www.udacity.com/course/artificial-intelligence-nanodegree--nd889)<br/>
[AIND Term 1 Syllabus](https://medium.com/udacity/ai-nanodegree-program-syllabus-term-1-in-depth-80c41297acaf)<br/>
[AIND Term 2 Syllabus](https://medium.com/udacity/ai-nanodegree-program-syllabus-term-2-deep-learning-in-depth-d935197b66ec)


## Sudoku Solver

I wrote a Sudoku puzzle solver that uses depth-first search with constraint propagation to _systematically_ inspect all the possible puzzle board configurations until the solution is found.

### How to frame a simple state space problem

The Sudoku puzzle solver doesn't use a state space that merely captures a puzzle board configuration, e.g. square A1 is 1, square A2 is unknown, square A3 is 7, etc. A more complex state space was created to better match the algorithm used to solve the puzzle.

For each square in the 9x9 grid, keep track of a list of possible values for that square. If a square's value is known for sure, the list contains only one digit, e.g. 9. If a square's value is commpletely unknown, the list contains every digit 1-9.

This state space definition allows us to efficiently search the state space for a solution to the puzzle. A solution will be a state in which each square has exactly one value, and that value is the correct value according to the rules of the puzzle. Note the valid puzzle board configurations for all puzzles are therefore only a subset of the state space. The complete state space includes all partially solved and completely solved puzzles.

### How to use state space search to solve a simple problem

The goal was to create an algoritm that searches through possible board configurations in an effecient way until it finds the solution, that is, the board configuration with the solved puzzle. The algorithm must be prepared to search through every possible configuration, but hopefully it'll find the answer long before all possibiliites are exhausted.

### Why optimized search is needed

Even a simple puzzle like Sudoku contains a staggering number of possible board (puzzle) configurations: 4.6 x 10^38! With a large state space, eliminating large portions of it is essential. This can be done by eliminating portions that aren't expected to yield a solution, or, in the case of Sudoku, are known not to contain the solution. An optimized search is absolutely necessary, and isn't too hard to define.

### Use depth-first as systematic search strategy

The first requirement for our search algorithm -- ensuring that each possible board configuration would eventually be searched, if necessary -- was met by choosing a depth-first search algorithm. Depth-first search is a way to evaluate every node in a tree exactly once, without missing any and without any duplication.

Depth-first search is typically thought of in terms of a tree data structure, but the Sudoku board doesn't resemble a tree. Depth-first search here means that we evaluate all possible values for a board square before moving on to the next square, e.g. D3 can be either '1' or '7', so first we'll evaluate '1', then '7', then (if necessary) move on to another square.

### Use constraint propagation to optimize state space search

How do you write an efficient search algorithm? Use constraint propagation. Constraint propagation is a technique used to reduce the search space by eliminating states in the state space that cannot be a valid solution according to the rules of the problem.

An example of constraint propagation for Sudoku is simply following one of the rules of the puzzle: the digits 1-9 must appear exactly once. For a given state, find a square with only one possible value, then remove that value from the possible values of each of the square's 8 neighbors. Recall that in Sudoku, each square has 3 sets of neighbors: the 3x3 grid, the 9-square row and the 9-square column. The constraint is propagated to each set of neighbors.

How well does constraint propagation work in this case? In [Peter Norvig's analysis](http://norvig.com/sudoku.html), the hardest Sudoku puzzles require searching only 64 board configurations (on average over the set of hard puzzles) out of the 10^38 or so. That's so much better than the brute force approach it's staggering.


## Adversarial Game Playing Agent

In this project, I built a game playing agent for a simple board game called Isolation. It explored "adversarial search," which is a short way of saying "an agent in a two-player game that determines the player's next move by searching through the set of possible moves and evaluating each one in turn, taking into account the likely opponent counter-moves."

### How to frame the problem of building a game agent

In order to build a good agent, you need to model a tree of hypothetical moves from the player you're controlling and the opponent. Each node in the tree is a game board configuration (describing where the game pieces are placed at that point in the game), and each edge represents moving one player piece according to the rules of the game. The tree isn't preconstructed, but built up on the fly, as needed to evaluate possible moves.

A typical evaluation goes something like this: OK, I have 4 possible moves. Let's try the first move. If I make that move, my opponent will make this move. Then I'll have 2 possible moves to counter my opponent's move. Let's try the first counter move. If I make that first move, I'll have lost the game. Make a note of that. Now let's try the second counter move...

Having a tree data structure allows the agent to easily back up to a certain move in the sequence of hypothetical moves and try a different move.

### Why search is fundamental to game agents

A game playing agent selects its next move by trying out all possible (legal) moves, predicting the opponent's countermove to each, then trying out all possible moves that counter those opponent moves, and so on. Why is this called search? What exactly is being searched? The technique searches the game state space, which for a board game such as Isolation is the set of board configurations.

Note that the game of Isolation has been carefully crafted to have a much smaller state space than that of chess. There are only two game pieces, one for each player. The game pieces move as chess knights, but each move changes the game board in a way that can't be reversed, dramatically limiting the number of possible moves and ensuring a clear winner each time.

### Minimax search

The Isolation game playing agent I developed uses iterative deepening minimax search with alpha-beta pruning as the algorithm for selecting the player's next move. This is a well-established technique for searching a game's state space for a _good_ move.

To understand why the search sophisticated algorithm works well, let's start with a naive implementation of a game playing agent: determine all possible legal moves and choose one at random. This algorithm has one important thing in common with any more sophisticated algorithm. They both choose one move from among the set of legal moves from the current game board. The difference is only in the "goodness" of the move. What's a "good" move? You don't need to be a computer scientist to answer this one. A good move is one that leads to a game board state in which the player's chances of winning are more favorable than with the other possible moves.

The minimax algorithm evaluates a sequence of moves by assuming that each player will make that move that's most advantageous for that player. It's relatively easy to develop intuition for the minimax algorithm. Say you're the game agent selecting moves for your player. You need to evaluate all possible moves for this turn and pick the "best" one. So you select a candidate move from among all the possible moves. You now need to predict the opponent's counter move. What move will the opponent make? Assume the opponext will make the move that most disadvantages your player. Now you need to evaluate a new set of candidate moves, moves made from a game board that reflects the initial candidate move and the assumed opponent counter move. This process continues until you reach a game board in which your player has either won or lost.

Each candidate move creates a branch in the tree, and a path through the tree represents a sequence of moves.

### Depth-limited minimax search

The classic minimax algorithm searches all the way to "terminal" states. In a terminal state, the game is either won or lost and the utility of that sequences of moves is clear. This is hardly ever implemented in practice, however, since the search spaces are so vast. Instead, after trying out a sequence of candidate moves of a certain length, the algorithm uses a heuristic to score that sequence. The score is an estimate of the utility of the game board configuration after all the moves in the sequence have been made. The way to think about utility is to answer the question, "How likely is the player to win the game with that board configuration as the starting point?" If the chances of winning are good, that board has high utility; otherwise its utility is low.

### Depth-limited minimax search with alpha-beta pruning and iterative deepening 

Alpha-beta pruning is an enhancement to the minimax search algorithm to "prune" branches from the search tree that will not yield good moves (at least as measured from the current game board state). By pruning, or ignoring, these branches, we reduce the size of the search tree for selecting the player's next move. This reduction in size allows the next-move selection to be made in less time, without the increasing the risk of mis-evaluating a move's utility. It's a better version of minimax search.

So what's the iterative deepening part? Iterative deepening is a technique used in a time-limited search strategy that quickly determines a good move, but keeps working to find a better move, until time runs out. You start out examining short sequences of moves, which clearly takes less time, then gradually increase the sequence length. Each time you increase the length of sequences being evaluated, you increase the fidelity of your analysis of the set of possbile next moves. The likelihood of the "best" next move truly being the best move increases.

