# marqo 搜索方式 (https://docs.marqo.ai/2.3.0/#get-index-stats)


### 向量检索，默认的的检索方式
````
import marqo

mq = marqo.Client(url="http://localhost:8882")


//创建索引  ， model embedding 方式？
mq.create_index("my-first-index", model="hf/e5-base-v2")

//增加文档，文档并没有指定schema
mq.index("my-first-index").add_documents(
    [
        {
            "Title": "The Travels of Marco Polo",
            "Description": "A 13th-century travelogue describing Polo's travels",
        },
        {
            "Title": "Extravehicular Mobility Unit (EMU)",
            "Description": "The EMU is a spacesuit that provides environmental protection, "
            "mobility, life support, and communications for astronauts",
            "_id": "article_591",
        },
    ],
    // 这里指定哪些字段进行索引，可以指定多个
    tensor_fields=["Description"],
)

// 用自然语言进行搜索
results = mq.index("my-first-index").search(
    q="What is the best outfit to wear on the moon?"
)

````

### Lexical search (支持 BM25 进行排序搜索)

````
result = mq.index("my-first-index").search("marco polo", search_method="LEXICAL")
````


### Multi modal and cross modal search
````
//设置 是图片url， 并且指定embedding 方式。
settings = {
    "treat_urls_and_pointers_as_images": True,  # allows us to find an image file and index it
    "model": "open_clip/ViT-B-32/laion2b_s34b_b79k",
}
response = mq.create_index("my-multimodal-index", **settings)



//插入文档下载图片
response = mq.index("my-multimodal-index").add_documents(
    [
        {
            "My_Image": "https://upload.wikimedia.org/wikipedia/commons/thumb/b/b3/Hipop%C3%B3tamo_%28Hippopotamus_amphibius%29%2C_parque_nacional_de_Chobe%2C_Botsuana%2C_2018-07-28%2C_DD_82.jpg/640px-Hipop%C3%B3tamo_%28Hippopotamus_amphibius%29%2C_parque_nacional_de_Chobe%2C_Botsuana%2C_2018-07-28%2C_DD_82.jpg",
            "Description": "The hippopotamus, also called the common hippopotamus or river hippopotamus, is a large semiaquatic mammal native to sub-Saharan Africa",
            "_id": "hippo-facts",
        }
    ],
    tensor_fields=["My_Image"],
)

// 根据文本搜索图片
results = mq.index("my-multimodal-index").search("animal")


//根据 图片搜索图片
results = mq.index("my-multimodal-index").search(
    "https://docs.marqo.ai/2.0.0/Examples/marqo.jpg"
)
````

### Searching using weights in queries

````
import marqo
import pprint

mq = marqo.Client(url="http://localhost:8882")

//创建索引默认的话就是向量搜索
mq.create_index("my-weighted-query-index")

//增加文章。只索引Description字段
mq.index("my-weighted-query-index").add_documents(
    [
        {
            "Title": "Smartphone",
            "Description": "A smartphone is a portable computer device that combines mobile telephone "
            "functions and computing functions into one unit.",
        },
        {
            "Title": "Telephone",
            "Description": "A telephone is a telecommunications device that permits two or more users to"
            "conduct a conversation when they are too far apart to be easily heard directly.",
        },
        {
            "Title": "Thylacine",
            "Description": "The thylacine, also commonly known as the Tasmanian tiger or Tasmanian wolf, "
            "is an extinct carnivorous marsupial."
            "The last known of its species died in 1936.",
        },
    ],
    tensor_fields=["Description"],
)

# 多条查询语句。每个查询语句 打不通的权重。下面是两条，一个1.1分一个1.0分
query = {
    # a weighting of 1.1 gives this query slightly more importance
    "I need to buy a communications device, what should I get?": 1.1,
    # This will lead to 'Smartphone' being the top result
    "The device should work like an intelligent computer.": 1.0,
}

results = mq.index("my-weighted-query-index").search(q=query)

print("Query 1:")
pprint.pprint(results)

# now we ask for a type of communications which predates the 21st century
query = {
    # a weighting of 1 gives this query a neutral importance
    "I need to buy a communications device, what should I get?": 1.0,
    # This will lead to 'Telephone' being the top result
    "The device should work like an intelligent computer.": -0.3,
}

results = mq.index("my-weighted-query-index").search(q=query)

print("\nQuery 2:")
pprint.pprint(results)
````


### 关于查询

````
results = mq.index("my-first-index").search(
    # 语义搜索
    q="New York", 
    # 标量搜索， 她的标量搜索和lucene差不多。
    filter_string="country:(United States) OR state:NY"
)


````


整体阅读下来


他的一个索引，相当于咱们的space，只有一种向量索引方式。没有schema。

他的输入是个json数据。通过field字段，指定哪些字段需要向量化，未指定字段默认就是标量。 

标量搜索 = lucene那种常用查询方式，

向量搜索完全采用自然语言的方式

