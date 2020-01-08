关于几个平衡搜索树的深入探讨
=========================

## Splay树

> Splay树基本操作的分摊时间复杂度为`O(logn)`，如何证明这个结论？

对于任意一棵伸展树`S`，引入势能函数$\Phi(S) = \Sigma_{v \in S} rank(v)$，其中$rank(v) = log(size(v))$，即以`v`为树根的子树的规模的对数。通过简单的分析可以看到，`S`越平衡，则它的势能越小，`S`越失衡，则它的势能越大。特殊地，当`S`是一条单链的时候，$\Phi(S) = O(nlogn)$；而`S`是一棵满二叉树的时候，$\Phi(S) = O(n)$。

将第`i`次操作的分摊时间复杂度$A_i$，写作实际的时间复杂度与势能变化量之和，即

$$
A_i = T_i + \Delta\Phi
$$

然后就伸展树当中存在的三种元操作，即`zig`，`zigzig`以及`zigzag`（其余的操作无非是这三个操作的对称情况），分别证明满足

$$
A_i \le 3\cdot [rank_i(v) - rank_{i - 1}(v)]
$$

这样，对于伸展树中一次基本操作（如`search`），其作用无非是将节点`v`逐层伸展到了根节点，因此分摊时间复杂度为$A \le 1 + 3\cdot[rank(r) - rank(v)] = O(logn)$。

## AVL树

> 高度为h的AVL树，至少拥有`fib(h + 3) - 1`个节点，此时对应了一棵`fib-AVL`数。

可以归纳地证明该结论。

> 对于任意一棵高度为h的AVL树，其叶节点的最小深度为$\lceil \frac{h}{2} \rceil$。

可以归纳地证明该结论。

> 对于一棵高度为h的AVL树进行删除，至多只会有一个节点失衡，但是至多需要进行`O(h) = O(logn)`次调整。具体说来，至多需要$\lfloor \frac{h}{2} \rfloor$次调整操作。

可以构造一棵高度为`h`的`fib-AVL`树，删除它的最左侧节点。当`h`为偶数时，该节点是叶节点，作为深度最低的一个节点，它的深度恰好是`AVL`树的深度下界，即$\lceil \frac{h}{2} \rceil = \frac{h}{2}$，对它进行删除后，该节点的所有祖先节点都需要一次调整，故一共需要$\frac{h}{2}$次调整操作。

当`h`为奇数时，最左侧节点`m`并非叶节点，它拥有一个右孩子，该右孩子是深度最低的叶节点。因此`m`的深度为$\lceil \frac{h}{2} \rceil - 1 = \lfloor \frac{h}{2} \rfloor$。将`m`删除，其实质是删除它的右孩子，这将导致`m`的所有祖先节点都进行一次调整操作，故一共需要$\lfloor \frac{h}{2} \rfloor$次调整。

综上，删除操作后最多进行的调整次数为$\lfloor \frac{h}{2} \rfloor$。

需要注意的是这里说的是调整操作，而不是旋转操作。在`fib-AVL`树中，每次这样的调整都可以通过单旋来完成。但是，可以通过对`fib-AVL`树的结构做简单的改变，使得这样的调整需要双旋来完成。这样，总体的旋转次数为$2 \cdot \lfloor \frac{h}{2} \rfloor$次。

> 证明：按照递增次序，将$2^{h + 1} - 1$个关键码依次插入到一棵初始为空的`AVL`树中，必然得到高度为`h`的一棵满二叉树。

下面归纳地证明该结论。

+ 当`h = 0`时，结论显然成立。
+ 假设对于任意高度不大于`h`的`AVL`树，该结论都是成立的。
+ 下面讨论将$2^{h + 2} - 1$个节点以递增次序插入到一棵初始为空的`AVL`树的过程。

	- 首先插入前$[0, 2^{h + 1} - 1)$个关键码。根据归纳假设，此时将形成一棵高度为`h`的`AVL`树。
	- 然后插入$[2^{h + 1} - 1， 3\cdot 2^{h} - 1)$的关键码。这些关键码将全部被插入到右子树中，而对左子树没有任何影响。根据归纳假设，此时右子树形成了一棵高度为`h`的`AVL`树，因此全树的高度为`h + 1`。
	- 插入第$3\cdot 2^{h} - 1$个关键码。此时在根节点不满足`AVL`树的平衡条件，因此需要做一次单旋调整。调整以后，左子树成为一棵高度为`h`的满二叉树，右子树的高度也为`h`，但是在最底层只有一个关键码，即刚刚插入的关键码。
	- 继续插入$[3\cdot 2^{h} - 1, 2^{h + 2} - 1)$个关键码，同样，这些关键码将被插入到右子树，而与左子树无关。根据归纳假设，插入完成后，右子树成为了一棵高度为`h`的满二叉树，此时全树也就成为了一棵高度为`h + 1`的满二叉树。
证毕。

实际上，可以类似地证明，将$2^{h + 1} - 1$个关键码以递减次序，依次插入到一棵初始为空的`AVL`树中，也可以得到一棵高度为`h`的满二叉树。

## B树

关于B树，首先需要明确高度的定义，乃是加上了外部节点后的树的高度。其次是说这里有两种统计口径，即节点数量和关键码数量。节点数量是指超级节点的数量，而关键码则是指一个数据的关键码，两者的关系是：一个超级节点中含有多个关键码。

> 对于含有`N`个关键码的B树，其高度`h`的上下限。

$$
log_m^{N + 1} \le h \le log_{\lceil{\frac{m}{2}}\rceil}^{\lfloor\frac{N + 1}{2}\rfloor} + 1
$$

这个结论的推导要会。需要注意的是，推导的过程，是通过高度为`h`的B树，所含有最多节点数量和最少节点数量，来得到`h`应该满足的关系的。为了将推导过程中使用的节点数量，转化为关键码数量，利用了B树外部节点的性质——外部节点的数量等于关键码的数量加一。实际上，这个性质的本质是二叉树二度节点数量与零度节点数量的关系。

> 关于B树插入和删除过程中的分裂和合并次数

考虑一次分裂操作，有两种情况，即在根节点分裂以及在非根节点分裂。如果是在非根节点分裂，一次分裂后B树的高度`h`不变，但是节点数量加一；如果是在根节点分裂，分裂后B树的高度`h`加一，节点数量加二。无论如何，一次分裂以后，`n - h`的值都会增加一，故可以之作为B树分裂次数的计数器。

同样地可以证明，一次B树的合并操作后，`n - h`的值将会减少一。

因此，如果对一棵初始状态`h = 1`的B树连续插入`N`个关键码，最终得到一棵高度为`h`，节点数量为`n`的B树，则累计的的分裂次数为`n - h`，平均每次插入的分裂次数为$\frac{n - h}{N} \le \frac{n - h}{n(\lceil \frac{m}{2} \rceil - 1)} = \frac{1}{\lceil \frac{m}{2} \rceil - 1}$。因此，平均$\lceil \frac{m}{2} \rceil - 1$次插入操作才会导致一次分裂。对于删除与合并操作，也有同样的结论。

尽管如此，单次的插入和删除操作，至多仍需要`O(logn)`次分裂或合并。在最坏情况下，对于任意的`m`为奇数，都可以构造一棵B树，交替地对它进行插入和删除操作，每次操作都需要`O(logn)`次分裂或者合并调整。

> 为什么B树的插入不采用旋转策略，而是统一采用分裂策略？

+ 分裂策略总是可行，程序逻辑简单。
+ 根据上面的分析，分裂操作通常不会频繁发生。
+ 空间利用率也不至于显著降低，因为至少有50%。
+ 树高也不至于明显增加，根据上面分析，树高主要取决于关键码的总数，而与节点数量几乎没有关系。