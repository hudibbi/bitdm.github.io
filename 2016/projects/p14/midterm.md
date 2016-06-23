###域名服务器信息挖掘

组员：王恒怿2120151038  郑越2120151072  苏思悦2120151038

##1.本实验的概述：
主要对收集到的七天的DNS会话集进行预处理，实现了频繁域名路径解析和域名服务器流量预测。在域名路径解析过程中，我们使用java编程实现了Apriori算法。

在服务器流量预测过程中，我们实现了论文中[2]提出的算法，使用Apriori算法查找频繁段，并在此基础上，创新使用了后缀数组算法处理数据，很大的提升了算

法的速度。查找到的频繁段用于聚类域名服务器，方便服务器的部署优化，并可预测流量攻击。

##2.基于Apriori的频繁连续时间片段选择算法

2.1本算法的实现步骤如下，已实现部分有具体的实现细节描述：

（1）数据清理。

（2）找出每天频繁访问的域名。

（3）七天的频繁域名信息集成。

（4）统计各频繁域名的单位时间访问流量。

（5）挖掘各频繁域名的频繁项集。

（6）频繁序列筛选。

2.2已完成的步骤

（1）数据清理。对每一天的会话纪录进行统计，对日志分析可以发现，其时间戳应该是顺序的，但是在整点转换时发现，会话纪录会出现乱序现象——例如前一个

时刻为14:59:59，第二个时刻为15:00:01，而第三个时刻为14:59:58，第四条纪录为 15:00:02。这种整点转换处的时间戳混乱现象，会对单位时间的流量统计结果

产生较大的误差，为了降低统计误差，我们采取的措施为：如果某两条小时数相等的会话纪录中间夹杂了其他小时数的纪录，则认为这些夹杂的纪录为错误数据，

将其删除（如本例中，删除第三个时刻的会话纪录）。

（2）找出每天频繁访问的域名。对每天的会话纪录进行分析，统计每个域名在当天会话纪录中的访问次数，并设置阈值M，当该域名的访问次数大于M时，认为该域

名的访问是频繁的，纪录其域名和对应的访问次数。针对本数据源，我们的综合分析，将阈值M设置为250（因为绝大多数的频繁项访问次数在250-600之间），统计

结果详见文件夹“Session data result（每天访问量大于250的域名）”。

（3）七天的频繁域名信息集成。对连续七天的“频繁域名”进行求交集运算，得到连续七天中，一直被频繁访问的域名。对本数据源数据进行分析，共得到了23个

频繁访问的域名（结果见文件“Intersect_Url（7天内所有访问量高的域名）”）。

（4）统计各频繁域名的单位时间访问流量。对各个域名分别统计每天单位时间的访问流量。结合具体数据集情况和分析需求，我们将时间单位设置为单位小时，分

别统计每个小时的改域名的访问次数作为其流量，23个频繁域名，每个域名可以得到1个文件，文件内记录该域名七天内，每天24h单位小时的访问请求流量，结果

详见文件夹“result_perhour（各域名的每小时访问量）”。 • 对各域名标记流量变化序列。对23个频繁域名的单位小时流量进行分析，并对其访问量的变化情况

用u、s、d三个字符和当前时间戳的小时数予以标记。具体标记策略为：将某一小时的访问量与前一小时的访问量做比较，得到差值d。设置阈值D>0,如果d>D，则记

录字符串“T(h)”(时间戳的小时数部分)＋“u”(up)；如果d<-D，则记录字符串“T(h)”(时间戳的小时数部分)＋“d”(down)；如果｜d｜<=D；则记录字符

串“T(h)”(时间戳的小时数部分)＋“s”(steady)。特别地，对于每天的第一个小时，我们记录为s。

经过反复测试，我们发现，当阈值D>15时，各个时间段的流量状态基本都为s，这就失去了数据挖掘的意义；而如果阈值设置过小，很小的波动和很大的访问量变化

都会被予以同样的“u”或“d”标记，这也会同样导致挖掘的数据不能很好的反映整体的变化趋势，因此作为折中，我们将阈值D设置为10。通过此方法对23个频繁

域名的访问情况标记记录，得到23个文件，结果如文件夹“result D（各域名的s u d序列）”所示。

（5）挖掘各频繁域名的频繁项集。分别对各个域名七天的usd序列进行分析，采用Apriori算法和后缀数组名相结合的方式进行频繁序列的挖掘，得到各个域名的

usd频繁序列。与已有的方法不同，我们并不直接采用Apriori算法进行频繁项挖掘，相反，首先，对所有的频繁域名用后缀数组的方法，挖掘出7天共有的usd最长

序列项，得到最长的频繁序列长度L。然后对该域名的以Apriori算法，挖掘出长度小于L的所有频繁序列。两种方式相结合，以剪枝的策略进行搜索，避免Apriori

算法对于过长频繁序列的计算判断，可以大大提高算法效率。

##3.基于Apriori的频繁域名解析路径分析算法

3.1本算法的实现步骤、该算法还未实现：

（1）数据清理。使用2节中使用的数据清理规则对每天的会话记录中的错误数据进行清除。

（2）数据处理。在会话访问数据集中，统计记录第𝑖天记录𝑇𝑖中出现的所有IP地址(假设共有𝑁𝑖个IP地址)，对于每条访问记录，我们采用0/1标记的方式，对于出现的

IP地址标记为1，因此，我们将每天的会话记录转化为0-1矩阵

（3）频繁项集的获取。分别对7天的处理后的路径解析数据进行处理，采用Apriori算法对数据集进行处理得到域名服务器域名解析路径中的频繁项集，经过反复试

验测试，最终我们取参数𝑠𝑢𝑝=0.5% ,𝑐𝑜𝑛=1%来获取最终数据。



n