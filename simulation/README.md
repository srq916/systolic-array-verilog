用蒙地卡罗法对PE共用cache的必要体积进行模拟。

这个模拟的逻辑是：尽管我不知道神经网络中每条行/列向量的0是怎么分布的，但对于大量的含0向量，其分布必然趋近于随机（大数定理）。
因此，我直接用大量次数的随机0/1来模拟处理一对稀疏向量的内积时，需要多大体积的cache（即有多大的cache可以确保不丢失任何数据）。

显然，对于固定稀疏比例、固定向量长度的单个PE，所需的cache体积是一个概率分布：比方说，大部分情况下4组reg就够了，极少部分情况下要5个，极端罕见情况下要6个，等等。

单个pe的模拟结果是这样的：
  - 向量长度50，0的比例10%，模拟次数一千次，最多需要5个额外的寄存器。
  - 向量长度50，0的比例50%，模拟次数一千次，最多需要13+个额外的寄存器。
很悲观，因为不做稀疏计算的话，一个pe内有4个寄存器，相当于翻了好几倍。而且，随着模拟次数增加（一万次，十万次），极端情况会越来越吓人，可能动不动要20+个额外寄存器。

而对于多个pe共用cache，就很乐观：(每次模拟会有轻微浮动)
  - 向量长度50，比例20%，8个pe共用cache，模拟1000次，平均每个pe需要2.625个额外寄存器。
  - 向量长度50，比例50%，8个pe共用cache，模拟100次，平均每个pe需要4个额外寄存器。
  - 向量长度50，比例50%，8个pe共用cache，模拟100000次，平均每个pe需要6个额外寄存器。
    
可以看出共用有两个好处：
1. 相同模拟次数下，平均单个pe的cache消耗变小了。
2. 随着模拟次数变多，cache消耗不会爆炸，会收敛在某个值上，这个值的tradeoff可以接受。


  
