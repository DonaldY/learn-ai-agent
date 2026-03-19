一个 word 文档、一个 pdf 文件、一个 youtube 视频、一个 url、一个 x 的推文等。
这种显然就不是直接创建 Document 对象了，而是要用各种 loader 来转换：


经过对应的 loader 处理后，变成 Document，之后再由嵌入模型向量化后存入知识库。

当然，有的文档可能会很大，比如一个 pdf 文件可能是一本书的大小。
这种很明显不能直接把转化后的 Document 向量化，需要先拆分文档。
也就是需要 Splitter：
大的文档经过 TextSplitter 分割后，变成一个个小文档，再给到嵌入模型做向量化。


● CharacterTextSplitter 功能 RecursiveCharacterTextSplitter 里都有
● TokenTextSplitter 严格按照 token，会破坏文档语义，不如 RecursiveCharacterTextSplitter 重写 lengthFunction
● 另外两个则是 RecursiveCharacterTextSplitter 的子功能。


token 是大模型输入的一个单位，可能一个单词是 1 到 2 个 token：
● apple 是 1 个 token
● pineapple 是 2 个 token
● 苹果是 1-2 个 token
