title: Breadth First Search without Queue
date: 2014-2-17
tags: Python
---

{%img /images/generator.jpg%}
source: [かみやまねき](http://www.pixiv.net/member_illust.php?id=1315311)

### TL;DR: The most intuitive implementation of  BFS is using Queue

Just out of fun, I wonder whether BFS can be implemented without Queue.
While DFS can be easily done by recursion, naive BFS implementation just incurs loop mayhem.

The first solution jumped into my mind is to add a depth parameter into BFS function.
The search function only visits nodes whose depth equals to the parameter and skips nodes whose depth does not.
I am [not alone](http://stackoverflow.com/a/2549825/2198656)

`Generator` is [sweeping](http://koajs.com) [NodeJS](https://github.com/petkaantonov/bluebird/blob/master/API.md#generators) community (admittedly this is my exaggeration). I'm quite obsessed with generator's suspension power. Here's my try in Python.


    '''
        implicit BFS, without queue or depth
        incurs infinite circulation on loop graph
        implement for fun (Really need a queue)
    '''
    def bfs_gen(node):
        '''
        generator function on a node, yield an iterator of frontier nodes
        the first call yield an iterator contains the node itself
        each subsequent call yields an iterator that is composed of children's gen
        try line chains children's yields, and thus accumulates frontier nodes
        By recursion, the root node's bfs_gen yields all frontier nodes on the graph
        frontier is maintained on callstack, by the suspension feature of generators
        '''
        try:
            yield iter((node.value, ))
            # :( still needs a list to maintain generators
            children = [bfs_gen(n) for n in node.children]
        except AttributeError:
            raise StopIteration

        chain = 0xDEADBABE
        while chain != ():
            chain = ()
            for child in children:
                try:
                    chain = (e for it in (chain, next(child)) for e in it)
                except StopIteration:
                    continue
            yield chain

    def bfs(root, pred):
        '''
        make generator on root node
        call next method to yield frontier nodes
        apply predicate function on it, if any matches, return True
        else continues to yield more frontier
        '''
        gens = bfs_gen(root)
        try:
            while not any(pred(v) for v in next(gens)):
                continue
            return True
        except StopIteration:
            return False

    def main():
        '''
        3 4 5 6
         1   2
           0
        '''
        from collections import namedtuple

        node = namedtuple('Node', ['value', 'children'])
        six   = node(6, [None])
        five  = node(5, [None])
        four  = node(4, [None])
        three = node(3, [None])
        two   = node(2, [five, six])
        one   = node(1, [three, four])
        zero  = node(0, [one, two])

        def demo(value):
            '''dumb log'''
            print(value)
            return False

        # bfs(zero, demo)
        for i in bfs_gen(zero):
            demo(i.value)

    if __name__ == '__main__':
        main()


Wow, much ugly, very obscure. But a old post on certain [unsung site](http://code.activestate.com/recipes/231503-breadth-first-traversal-of-tree/?in=user-218935) gives me an interesting solution.

    def bfs(root):
        yield root
        for node in bfs(root):
            for child in node.children:
                yield label(child)

Tada, done! Let's peruse this piece of code.
We want a generator that yields a sequence of nodes, this can be done by induction.

1. Given the root of a tree, clearly the first node of the sequence is the root
2. Because the next level of frontier nodes in the sequence is the children of the current level frontier nodes, construct a new sequence that lags behind.
3. yield each child given its parent.

Doing this recursively makes a BFS generator.

The trick part is the second step, for a normal function, this will make infinite loops. But `generator` is expanded dynamically, execution jumps out of BFS, eschewing looping condition. Suspension of code also implicitly stores searching states, where calling stacks morph into a Queue. Of course, visited nodes also need to be maintained in a list(not checked here). Termination check is absent, as well.

Interestingly, adapting BFS into DFS only requires swapping two lines, just as substituting stack for queue in classical implementation.

    def dfs(root):
        yield root
        for node in node.children:
            for child in dfs(node):
                yield child

Whatever fun generators have brought, BFS/DFS should probably never done like this. Storing information in stack does not save space. Static language like C/C++ lacks such generator feature. And, notably, [function overhead](http://www.python.org/doc/essays/list2str.html) and [slow generator](http://www.ruby-doc.org/stdlib-1.8.7/libdoc/generator/rdoc/Generator.html) in most script language make developers repugnant to such winding trick.
