---
layout: single
title: Matrix graph representation in Python
date:   2017-03-24 12:15:02 +0100
categories: [dsp17, python, cs]
excerpt: How to represent graph structure as matrix in Python?
---

Last week I wrote how to represent *graph* structure as
[**adjacency list**]({{ site.baseurl }}{% link _posts/2017-03-16-graphs_in_python.md %}).
This week time has come to describe how we can represent *graphs* in a a *matrix* representation.

Our requirements (the same as previous week):

* small amount of code (we want to code it fast on algorithmic contest)
* ability to change our structure dynamically (we want to add some edges or vertices after *graph* initialization)
* *graph* is **undirected** (for each two vertices there can be at most one edge and edges don't have directions)

## *Graph* as matrix in **Python**

*Graph* represented as a *matrix* is a structure which is usually represented by a $$2$$-dimensional *array* (table)
indexed with *vertices*. Value in cell described by *row-vertex* and *column-vertex* corresponds to an *edge*.
So for *graph* from this picture:

<img src="{{site.url}}/assets/images/graph.png" style="display: block; margin-left: auto; margin-right: auto;"/>

we can represent it by an array like this:
<table>
    <thead>
        <tr>
            <th># cell</th>
            <th>A</th>
            <th>B</th>
            <th>C</th>
            <th>D</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><b>A</b></td>
            <td>0</td>
            <td>1</td>
            <td>1</td>
            <td>1</td>
        </tr>
        <tr>
            <td><b>B</b></td>
            <td>1</td>
            <td>0</td>
            <td>0</td>
            <td>0</td>
        </tr>
        <tr>
             <td><b>C</b></td>
             <td>1</td>
             <td>0</td>
             <td>0</td>
             <td>1</td>
        </tr>
        <tr>
             <td><b>D</b></td>
             <td>1</td>
             <td>0</td>
             <td>1</td>
             <td>0</td>
        </tr>
    </tbody>
</table>

For example $$cell[A][B] = 1$$, because there is an edge between $$A$$ and $$B$$, $$cell[B][D] = 0$$, because
there is no edge between $$B$$ and $$D$$.

In **C++** we can easily represent such structure using $$2$$-dimensional arrays (`std::vector` is better than regular
array):

{% highlight C++ %}
    std::vector<std::vector<int>> graph;
{% endhighlight %}

Of course then we have to initialize every cell in our array:

{% highlight C++ %}
    // graph initializing

    for (int i = 0; i < number_of_vertices; i++) {
        graph.push_back(vector<int>()); // adding an empty rows
    }
    for (int j = 0; j < number_of_vertices; j++) {
        for (int i = 0; i < graph.size(); i++) {
            graph[i].push_back(i * j); // add columns to rows
        }
    }

    // adding an edge example (we need to add both ends):
    graph[2][3] = 1;
    graph[3][2] = 1;
{% endhighlight %}

In **Python** a *list* is an equivalent of an array. The most obvious implementation of a structure could look like this:
{% highlight python %}
    class ListGraph(object):
        def __init__(self, number_of_vertices):
            self.matrix = [[0] * number_of_vertices for _ in range(number_of_vertices)]

        def add_edge(self, v1, v2):
            self.matrix[v1][v2] = 1
            self.matrix[v2][v1] = 1

{% endhighlight %}

It seems in **Python** we can initialize this structure in much shorter way (actually in one line - look at `__init__`).
For this implementation we need $$O(|V| ^ 2)$$ memory, where $$|V|$$ is number of vertices. Of course as *graph* is
**undirected** we can skip half of this array, because we duplicate edge information, but I'm not going to cover
it in this post, because for **Python** I've a better solution. So, think about this question:

*Is there a way to omit information about not existing edges?*

And now think about `dict`.

### Using `dict` for matrix graph implementation

Ok, so we want to store our **graph-matrix** in a `dict`. As it's a **key-value** store, we want to know what information
to store in *keys* and what *value* should they have. As we still want to be able to write something like
this: `graph[i][j]` to access an edge it becomes clear that **values** are *edges* and as **keys** we should have
pairs of *vertices*. So let's use **`tuple` of vertices** as keys!

{% highlight python %}
    graph = {}  # yep, that's it - we don't need to initialize whole structure

    # add edge
    graph[(2, 3)] = 1  # or anything what is not None
    graph[(3, 2)] = 1

    graph[(3, 2)]  # it's 1
    graph[(0, 1)]  # it's None, so there is no edge between 0, 1

{% endhighlight %}

That's it! When there is an *edge* between two vertices we will have a `not None` value otherwise `None`. For
**sparse graphs** we can save a lot of memory here, because we will actually use $$|E| * 2$$ where $$|E|$$ is number
of edges. Of course, we are unable to tell easily how many *vertices* our *graph* has (we have to check all keys). If
we need this kind of information we can create a more sophisticated implementation:

{% highlight python %}
    class DictGraph(dict):

        def __init__(self, vertices):
            self.vertices = vertices  # vertices is a list or set

        def add_edge(self, v1, v2):
            if v1 in self.vertices and v2 in self.vertices:
                self[(v1, v2)] = 1
                self[(v2, v1)] = 1
{% endhighlight %}

Class `DictGraph` derives from `dict`, so we still can use it as dictionary, but we have also a `list` of *vertices*.
Now it's easy to check if *vertex* is in our *graph* - it's enough that we check if it's in `vertices` *list*.

## Comparision of `dict` and `list` graph implementations

### Adding a vertex

Let's see how to add a vertex in both `dict` and `list` implementations mentioned above.

#### Adding a vertex in `list` implementation

{% highlight python %}
    class ListGraph(object):
        # ...

        def add_vertex(self):
            self.number_of_vertices += 1
            for vertices in self.matrix:
                vertices.append(0)
            self.matrix.append([0] * self.number_of_vertices)
{% endhighlight %}

To add a vertex $$v$$ we need to add to our *matrix* **new column** and **new row**. So we have to iterate over all
*vertices lists* and append $$v$$ there (you can think about it as adding a **new column**). Next
we have to create an array of vertices for $$v$$ (adding a **new row**). Time complexity is $$O(|V|)$$.
We also need about $$2 * |V|$$ space.

#### Adding a vertex in `dict` implementation
{% highlight python %}
    class DictGraph(dict):
    # ...

        def add_vertex(self, v):
            self.vertices.append(v)
{% endhighlight %}

Here we have to append a vertex $$v$$ to vertices list (if we use one, it's not necessary). As it doesn't have
any edges with other vertices it doesn't require any action. Necessary time: $$O(1)$$ (worst case
is $$O(|V|)$$, space is $$O(1)$$.

### Removing an edge

In both implementation it's easy:

#### Removing an edge in `list` implementation
{% highlight python %}
    class ListGraph(object):
        # ...

        def remove_edge(self, v1, v2):
            self.matrix[v1][v2] = 0
            self.matrix[v2][v1] = 0
{% endhighlight %}

Just put $$0$$ in both ends of edge. Time $$O(1)$$. We don't save any space.

#### Removing an edge in `dict` implementation
{% highlight python %}
    class DictGraph(dict):
        # ...
        def remove_edge(self, v1, v2):
            del self[(v1, v2)]
            del self[(v2, v1)]
{% endhighlight %}

Here we can actually delete both ends of edge from dictionary. Time $$O(1)$$,
saving a little of space.

### Removing a vertex

Removing a vertex is more complicated, especially in `list` implementation.

#### Removing a vertex in `list` implementation
{% highlight python %}
    class ListGraph(object):
        # ...
        def remove_vertex(self, v):
            for vertices in self.matrix:
                vertices.remove(vertices[v])
            self.matrix.remove(self.matrix[v])
            self.number_of_vertices -= 1
{% endhighlight %}

To remove *vertex* $$v$$ we have to remove its **column** and **row** in a table. So we have iterate over all
vertices lists, remove vertex $$v$$ from them (removing a **column**), finally remove whole list corresponding
to this vertex (removing **row**). Time complexity is $$O(|V|)$$.

#### Removing a vertex in `dict` implementation
{% highlight python %}
    class DictGraph(dict):
    # ...
        def remove_vertex(self, v):
            keys = list(self.keys())
            for v1, v2 in keys:
                if v1 == v or v2 == v and self.get((v1, v2)):
                    del self[(v1, v2)]
            self.vertices.remove(v)
{% endhighlight %}


To remove *vertex* $$v$$ from dictionary *graph* we need to remove all its *edges*, so we have to
to check all **keys** (`tuple` of two vertices) and if any of them has $$v$$ remove it - it is removing an edge.
After that we need to remove $$v$$ from *vertices list* if we use it.
Time complexity is $$O(|E|)$$, because we iterate over all edges, not vertices.
Actually for **dense graphs** it is worse than in `list` implementation.

## Summing up

It seems that except removing a *vertex* in **dense graphs**, `dict` implementation of *graph-matrix* is much better
than `list` implementation. It saves much more *space*, because it doesn't need to save information
about *non-existing* edges. Also for me it's much *more readable*, because it allows usage of any kind of
names for vertices - you can name them using numbers, letters, whatever you want (as long as you don't add the same
vertex twice, better to check it before adding?). Dictionaries in **Python** are really powerful - they are implemented
as **hash tables** that's way it has really good time complexities for dictionary operations.



You can find code of both implementations and some tests as examples of usage here:
[**gist**](https://gist.github.com/vevurka/c285fad3c80455d8fec0b6fe2d5387b2).
Hopefully you enjoyed reading about this approach for implementing *graph* structure. Let me know your thoughts or
remarks in comments!
