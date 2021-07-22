# Lucene笔记

## Lucene架构













## segments策略

### segments合并

Lucene的索引文件，会包含很多个segments文件，每个segment中包含多个documents文件，一个segment中会有完整的正向索引和反向索引。
在搜索时，Lucene会遍历这些segments，以segments为基本单位独立搜索每个segments文件，而后再把搜索结果合并。

建立索引文件的过程，实际就是把documents文件一个个加入索引中，Lucene的做法是最开始为每个新加入的document独立生成一个segment，放在内存中。而后，当内存中segments数量到达一个阙值时，合并这些segments，新生成一个segment加入文件系统的segments列表中。

而当文件系统的segments数量过多时，势必影响搜索效率，因此需要不断合并这些segments文件。

### 合并策略

第一种策略实现代码位于IndexWriter#optimize()函数，其实就是把所有segments文件合并成一个文件。合并的算法是递归合并列表最后的mergeFactor个segments文件直到合并成一个文件。这儿mergeFactor是Lucene的一个参数。

第二种策略实现代码位于IndexWriter#maybeMergeSegments()函数中

















###