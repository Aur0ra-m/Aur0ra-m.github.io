# 黑盒扫描器开发-如何有效去除重复流量


# 背景
在写被动流量扫描工具时，常会碰到对相同流量进行重复扫描的情况。这无疑降低了扫描器的检测效率，同时由于还会保存重复的扫描结果， 影响最终的检出效果。为此需要设计出一个重复流量过滤器，在流量正式进入检测引擎前进行去重。

# 重复流量过滤器设计
**实现思路**：
   1. 先将请求对象进行标准化，处理为一个字符串数组
   2. 将标准化的结果用树结构进行保存   
   3. 搜索时采用DFS深度优先算法进行搜索，如果找到了就返回结果；如果没找到，就递归创建新的节点   
   4. 同时为了避免长时间运行导致树结构枝条庞大，所以附加了一个轻量级的GC, 
      用于对无效节点进行定期回收，有效提高树的稳定性。

### 请求标准化
将一个请求进行标准化，得到一个字符串数组：
1. 一般请求**GET http://www.baidu.com/index/test**
   -->
   ['http',
   'get','www.baidu.com','index','test']
   。整体按照schema、method、host、all path进行拆分
2. 特殊框架请求**GET http://www.baidu.com/?
   action=list** -->['http',
   'get','www.baidu.com','action{list}']
   。这里是对action进行特殊化解析为字符串（此种方式对其他框架的参数解析也是相类似的，但需要确定好整体的框架类型，避免无法被有效识别）。

### 节点树构造
首先设置一个root根节点，然后再根据请求对象进行标准化后得到的字符串数组，对树进行搜索和树节点创建即可。

**实现思路**：根据标准化后的数组，对节点树进行DFS搜索。如果找到了所有的节点，则返回重复标识；如果在某个节点开始就找不到了，那就先对树节点进行递归扩建，扩建完后返回非重复标识。
```go
func searchTree(parent *node, 
normalizedRequest []string, point int) bool {
   if point == len(normalizedRequest){
       return true
   }
   
   // 持续递归搜索
   for _, childNode := range parent.childNodes {
      if normalizedRequest[point] == childNode.value{
      return searchTree(childNode, 
      normalizedRequest, point+1)
      }
   }
   
   // 拓展新节点
   for i := point; i< len(normalizedRequest); i++{
      // 1.新建节点
      // 2.将新节点添加到parent子节点数组中
      // 3.循环创建新节点
   }
   
   return false
}
```

### GC定期回收无效节点
我采用的是通过访问次数来做生存时效判定。即在node中保存一个扫描次数，GC定期进行DFS扫描，每次扫描，则会对扫描次数进行+1操作。而每次搜索过程中，如果经过该节点，则会重置扫描次数为0。当某次GC
扫描，发现当前节点扫描次数超过指定次数（阈值）时，则会从节点树上清除当前节点，当前节点的所有子节点也会被同时删除。最终实现节点存活判定与无效节点的清除工作。

# 最终效果
![img.png](/images/imgs/url_duplication/img.png)
![img_1.png](/images/imgs/url_duplication/img_1.png)

可以看到当经过一个GC周期后，当扫描次数到5 的时候，节点就会被清除，而其他没有到达阈值的节点计数则会+1。经测试，最终的整体检索性能还是可以的，一般情况下是无感知的。

# 其他
1. 实现时，需要注意线程安全的问题
2. 对于其他特殊框架，需要做相关适配
