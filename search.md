

# insert document：

* 每个文档进入后赋予数字递增id。类型为u32 此id为内部索引id
* 插入到rocksdb中 用户ID+文档pk 为key 文档json data 为value 
* 文档进入后通过用户定义的index schema解析，根据field索引类型调用不同的toknizer进行索引前分词工作
    索引类型包含：

        fulltext index， 
        keyworrd index 
        vector index

    example：

      title：“中华人民共和国”
      fulltext结果 ： token1：中华，token2：人民，token3：共和国
      age：30
      keyword结果：30

* 内置内存存储中包含一个 文档pK到id的索引，等同于keyword索引
* 标量索引生成倒排索引插入到内存kv中

        1. 倒排索引：
            key是token value是文档id列表 `roaring`数据结构。
            example: key:token = value:[docid1, docid2, docid3, docid4] 
        2. 逆文档索引
            key是文档ID value是 文档中所有的 (token，出现次数) 数组数据结构
            example: key:docid = value:[(token1, 1), (token2, 1), (token3, 1)]


* 总体流程 document -> rocksdb -> field parse -> insert into index (btree, hnsw)


* 向量索引存储
    1. 采用hnsw算法进行向量索引存储
    2. 使用id为文档唯一识别码


        
# delete document：
* 方案一，标记删除 后期统一处理？
* 方案二：
    * 根据主键 找到 文档id
    * 通过 id找到 逆文档索引 ，定位所有的倒排索引进行删除。
    * 删除rocksdb中 文档id 


# update document：

* delete+ insert


# get document：
* 通过pk 找到id，
* 通过id查询rocksdb 返回结果


# search document：
    TODO

