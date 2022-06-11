### Public Key Retrieval is not allowed.
在我们使用MySQL8.0的，连接数据库会存在一定的问题
当提示。Public Key Retrieval is not allowed 错误的时候，我们可以在连接数据库的配置文件中加上

allowPublicKeyRetrieval=true

