---
layout: single
title: Graphs in Python
date:   2017-03-16 13:15:02 +0100
categories: [dsp17, python]
tags: [graph, computer-science, python, data-structures, algorithms, programming]
excerpt: How to represent graph structure in Python?
---

There is no implementation of *graph* in **Python Standard Library**. The truth is that *graph* structure is rarely put
into standard libraries - I can come up with only one example of programming language which has this structure by
default: **Erlang** and its `digraph`. Anyway - today I want to focus on its implementation in **Python**, because
it's one of things in which I feel lack of pointers with comparision to **C/C++** languages.

Let's say we want to implement some graph algorithm (like *Dijkstra*) in **Python**, but we want to write as less code
as possible for *graph* structure implementation. So our requirements are:

* small amount of code (we want to be able to code it fast and forget even faster :wink:)
* ability to change our structure dynamically (we want to add some edges or vertices after *graph* initialization)
* *graph* is **undirected** (for each two vertices there can be at most one edge and edges don't have directions)

Generally there are many approaches in implementing *graphs*, but I want to focus on those two:

1. adjacency list
2. adjacency matrix

In this post I want to go through the first approach - the second I will describe next week.

## *Graph* as adjacency list in **Python**

*Graph* represented as an adjacency list is a structure in which for each vertex we have a list of adjacent vertices.
So for *graph* from this picture:

<img src="{{site.url}}/assets/images/graph.png" style="display: block; margin-left: auto; margin-right: auto;"/>

the adjacent lists for each vertex will look like this:


* for $$A$$: $$[B, C, D]$$
* for $$B$$: $$[A]$$
* for $$C$$: $$[A, D]$$
* for $$D$$: $$[A, C]$$

In **C++** we can achieve this kind of structure by creating a node structure for storing an array of pointers
to other nodes (better to use `std::vector` than regular array):
{% highlight c++ %}
    class node {
        std::vector<node*> neighbors;
    };
{% endhighlight %}
On a snippet above there is only a structure, but you can add edges just by pushing back initialized nodes to
`neighbours`, whole *graph* structure can be just a *list* (*vector*, *array*, etc) of nodes.

In **Python** it is not much harder. For algorithmic contests it should be enough to implement whole graph structure
with adding edges and vertices like this:

{% highlight python %}
    class Graph(dict):
        def __init__(self, vertex_list, edge_list):
            for v in vertex_list:
                self[v] = set()
            for e in edge_list:
                self.add_edge(e)

        def add_edge(self, edge):
            """
            Edge is a tuple of two vertices
            """
            self[edge[0]].add(edge[1])
            self[edge[1]].add(edge[0])

        def add_vertex(self, vertex):
            self[vertex] = set()

    # usage:
    graph = Graph(['A', 'B'], [('A', 'B')])
    graph.add_vertex('C')
    graph.add_edge([('A', 'C')]
    neighbours_of_A = graph['A']  # graph derives from dict

{% endhighlight %}

I've used here two basic data structures from **Python Standard Library**: `dict` and `set`. My `Graph` is
a dictionary in which I store vertices labels. Usage of `dict` allows quick access to elements.
In Python `dict` is implemented as a hash table, so the average time complexity of accessing an element is $$O(1)$$ -
the same as when using `std::vector` in **C++**.
For storing list of adjacent vertices I've used `set` data structure, because set gets rid of duplicates. Actually in
**Python** `set` is also implemented as hash table - it is a `dict` which stores dummy values. Using this structure
allows us to check if vertex is adjacent to other vertex in average time complexity $$O(1)$$ (the worst is $$O(|E|)$$
where $$|E|$$ is number of edges in *graph*).


### There is shorter implementation...

When you are a hardcore contestant in algorithmic competitions you can make implementation of a *graph* structure shorter
by using `defaultdict` (it's a bit modified `dict`, average time complexity of operations is the same):

{% highlight python %}
    from collections import defaultdict

    d = defaultdict(set)  # structure initialization

    # add an edge (remember to add to both vertices!)
    d[1].add(2)
    d[2].add(1)

    # d is defaultdict(<type 'set'>, {1: set([2]), 2: set([1])})
{% endhighlight %}

It seems that it's possible to have a structure implementation of graph in only one line...
This `deafultdict` uses a default value `set()` for not initialized keys - it's quite dangerous. *What will happen
when you add an edge for not existing vertex?* The previous implementation will throw `KeyError` exception in this case,
but this one will just create a new vertex along with this edge. Actually it can be a desired behaviour, anyway, you
should be aware of this - it's not a fault tolerant implementation.
Also if you forget to add an edge to both edge vertices funny things can happen.
Probably it's better to use it in this way for directed graphs and for undirected write a separate function which adds
an edge.

### Deleting vertex and edge

{% highlight python %}
    class Graph(dict):
        # ...

        def delete_edge(self, edge):
            self[edge[0]].remove(edge[1])
            self[edge[1]].remove(edge[0])

        def delete_vertex(self, vertex):
            temp_neighbour_set = self[vertex].copy()
            for neighbour_vertex in temp_neighbour_set:
                self.delete_edge((neighbour_vertex, vertex))
            del self[vertex]

{% endhighlight %}

Deleting an edge is quite easy - we just have to remove corresponding vertices from two sets. For deleting a vertex we
need to iterate over its adjacency set, remove an edge between this vertex and each neighbour, then finally remove it
from our dictionary. The average time complexity is $$O(deg(|V|))$$ (degree of vertex - number of its neighbours),
because checking if vertex is in set  of edges is in $$O(1)$$, deleting elements from set is $$O(1)$$ and deleting
from dictionary is also implemented  in $$O(1)$$, so only iterating through vertex neighbours matters. For
*sparse graphs* each vertex degree is small, so it will be almost $$O(1)$$.
Unfortunately the worst case complexity scenario can become $$O(|V|^2)$$, where $$|V|$$ is number of vertices.
It's because worst scenario complexity of deleting from set is $$O(|V|)$$) and our vertex can have at most $$|V| - 1$$
neighbours, when *graph* is dense.

### Pros and cons of using list adjacency approach in **Python**

List adjacency implementation of a *graph* is easy to understand, also it's quite readable (if you don't use any magic
approaches like `defaultdict(set)`).
It's good for *sparse graphs*, because it doesn't use any additional memory - only what's necessary to store vertices
and edges, so $$O(|V| + 2 * |E|)$$.
One of the biggest cons is that for *dense graphs* the dictionary operations can have a worst case scenario
about $$O(|V|)$$. The deletion of vertex can have quite bad time complexity (even $$O(|V|^2)$$...) - it can be done much
faster using other approaches. I guess in **C++** it can be done better even with this approach, because we can just set
a value of a node to `null` and after deletion of vertex, pointers of this vertex in other vertices adjacency lists will
point to this `null`. It's good to get rid of those `null pointers` in adjacency lists, but it can be postponed (for example
only done for each adjacency list when searching adjacent vertex). I think this amortized approach can't be easily
done in **Python**, because it lacks pointers.


You can find code from this post here: [gist](https://gist.github.com/vevurka/539d82eb0ba60c16aa8aa65610c627df).
Hopefully you enjoyed reading about this approach for implementing *graph* structure. Let me know your thoughts or
remarks in comments!