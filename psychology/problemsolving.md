Breaking Problem Down

Tldr :
1. Do I understand the problem
2. Can I find a simple algorithm or heuristic for my problem
3. Is my problem in the catalogue of algorithm problems in the algorithm books
4. Are there special cases of my problem that I know how to solve
5. Which of the standard algorithm design paradigms are most relevant to my problem
6. Am I still stumped?


Understand, research, base case, patterns, scale, and repeat


1. Do I really understand the problem?
	(a) What exactly does the input consist of?
	(b) What exactly are the desired results or output?
	(c) Can I construct an input example small enough to solve by hand? What happens when I try to solve it?
	(d) How important is it to my application that I always find the optimal answer? Might I settle for something close to the best answer?
	(e) How large is a typical instance of my problem? Will I be working on 10 items? 1,000 items? 1,000,000 items? More?
	(f) How important is speed in my application? Must the problem be solved within one second? One minute? One hour? One day?
	(g) How much time and effort can I invest in implementation? Will I be limited to simple algorithms that can be coded up in a day, or do I have the freedom to experiment with several approaches and see which one is best?
	(h) Am I trying to solve a numerical problem? A graph problem? A geometric problem? A string problem? A set problem? Which for- mulation seems easiest?

2. Can I find a simple algorithm or heuristic for my problem?
	(a) Will brute force solve my problem correctly by searching through all subsets or arrangements and picking the best one?
		i. If so, why am I sure that this algorithm always gives the correct answer?
	 	ii. How do I measure the quality of a solution once I construct it?
		iii. Does this simple, slow solution run in polynomial or exponential time? Is my problem small enough that a. brute-force solution will suffice?
		iv. Am I certain that my problem is sufficiently well defined to ac-tually have a correct solution?
	(b) Can I solve my problem by repeatedly trying some simple rule, like picking the biggest item first? The smallest item first? A random item first?
		i. If so, on what types of inputs does this heuristic work well? Do these correspond to the data that might arise in my application?
		ii. On what inputs does this heuristic work badly? If no such ex-amples can be found, can I show that it always works well?
		iii. How fast does my heuristic come up with an answer? Does it have a simple implementation?

3. Is my problem in the catalog of algorithmic problems in the back of this book?
	(a) What is known about the problem? Is there an available implemen- tation that I can use?
	(b) Did I look in the right place for my problem? Did I browse through all the pictures? Did I look in the index under all possible keywords?
	(c) Are there relevant resources available on the World Wide Web? Did I do a Google Scholar search? Did I go to the page associated with this book: www.algorist.com?

4. Are there special cases of the problem that I know how to solve?
 	(a) Can I solve the problem efficiently when I ignore some of the input parameters?
	(b) Does the problem become easier to solve when some of the input parameters are set to trivial values, such as 0 or 1?
	(c) How can I simplify the problem to the point where I can solve it efficiently? Why can’t this special-case algorithm be generalized to a wider class of inputs?
	(d) Is my problem a special case of a more general problem in the catalog?

5. Which of the standard algorithm design paradigms are most relevant to my problem?
	(a) Is there a set of items that can be sorted by size or some key? Does this sorted order make it easier to find the answer?
	(b) Is there a way to split the problem into two smaller problems, perhaps by doing a binary search? How about partitioning the elements into big and small, or left and right? Does this suggest a divide-and- conquer algorithm?
	(c) Does the set of input objects have a natural left-to-right order among its components, like the characters in a string, elements of a permu- tation, or the leaves of a rooted tree? Could I use dynamic program- ming to exploit this order?
	(d) Are there certain operations being done repeatedly, such as searching, or finding the largest/smallest element? Can I use a data structure to speed up these queries? Perhaps a dictionary/hash table or a heap/priority queue?
	(e) Can I use random sampling to select which object to pick next? What about constructing many random configurations and picking the best one? Can I use a heuristic search technique like simulated annealing to zoom in on a good solution?
	(f) Can I formulate my problem as a linear program? How about an integer program?
	(g) Does my problem resemble satisfiability, the traveling salesman prob- lem, or some other NP-complete problem? Might it be NP-complete and thus not have an efficient algorithm? Is it in the problem list in the back of Garey and Johnson [GJ79]?

6. Am I still stumped?
	(a) Am I willing to spend money to hire an expert (like the author) to tell me what to do? If so, check out the professional consulting services mentioned in Section 22.4 (page 718).
	(b) Go back to the beginning and work through these questions again. Did any of my answers change during my latest trip through the list?