$$

$$

# 索引结构

lucene创建索引是一个复杂的过程

## 词典索引

## 倒排表

## 正排表

## norm信息

# 索引缓存

TermsHashPerField类中存在三种缓存，均与指定的Field绑定，所以是Field间隔离的，定义如下

```
//存储term频率
final IntBlockPool intPool;
//存储term的postion偏移量
  final ByteBlockPool bytePool;
  //存储term内容字节编码
  final ByteBlockPool termBytePool;
```

![img](http://images2015.cnblogs.com/blog/438273/201703/438273-20170310201035436-1398379808.jpg)

# 索引新增

> http://blog.csdn.net/jj380382856/article/details/52411189
>
> http://mamicode.com/info-detail-113397.html
>
> http://blog.csdn.net/liweisnake/article/details/11364597
>
> https://www.kancloud.cn/digest/lulei-lucene/119383 good
>
> http://blog.csdn.net/conansonic/article/details/51886014 6.x源码分析 good
>
> http://www.cnblogs.com/miniqiang/p/4435834.html 4.7源码索引过程
>
> http://codepub.cn/2017/12/12/lucene-near-real-time-search/ lucene7.0索引可见
>
> http://www.cnblogs.com/daifei/p/3447267.htmlsolr索引提交控制参数
>
> minMergeDocs用于控制内存中持有的文档数量，也就是说，内存中文档被“刷”到磁盘前的数量
>
> http://www.blogjava.net/conans/articles/379550.html solr性能调优
>
> 

索引的操作在field粒度上，也就是以field来更新的，对应的实际索引操作类都是field级别的操作

## 新增流程

## 核心作用类

###  TermsHashPerField

  负责terms的索引写入内存工作，每个field都有对应的Terms，当field value被拆成terms，便经由此类完成索引过程，term字符串(CharTermAttribute)、term指针(AttributeSource.State)、term pos(PositionIncrementAttribute)等会被存储到内存中。

#### 新增流程

add()完成了新增逻辑

步骤如下

1. 根据term内容字节编码hash得到对应的termId(即内容相同的term，得到的termID也相同)

2. 判断termID是否已存在，不存在则新增posting，存在则修改posting

   如果hash得到的termID>0，则此termID不存在；若小于0，则termID=(-termID)-1得到实际已存在的termID

3. 继续处理下一个field

核心代码如下


   ```java
   void add() throws IOException {
       // We are first in the chain so we must "intern" the
       // term text into textStart address
       // Get the text & hash of this term.
       int termID = bytesHash.add(termAtt.getBytesRef());
         
       //System.out.println("add term=" + termBytesRef.utf8ToString() + " doc=" + docState.docID + " termID=" + termID);

       if (termID >= 0) {// New posting
         ......

         newTerm(termID);//生产此termID对应的postings，并加入此docID

       } else {
         termID = (-termID)-1;
         ......
         addTerm(termID);//将docID将入到此termID对应的postings尾部
       }
   	if (doNextCall) {
     //继续处理下一个field nextPerField.add(postingsArray.textStarts[termID]);
    }
     }
   ```


## 作用链



# 索引更新

# 索引删除