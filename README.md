# Usage
- Open http://localhost:3000 in your browser to view the app.
- Input the `url` of `doc`, generate `embeddings` `pgvector` to online supabase
- Access http://localhost:3000/docs for asking questions

https://m.weibo.cn/status/4879926925199207?sourceType=weixin&from=10D3395010&wm=9006_2001&featurecode=newtitle&s_channel=4&s_trans=1922705517_4879926925199207&jumpfrom=weibocom



微博正文

宝玉xp 
3-16 14:38 来自 微博网页版 已编辑
上一篇 网页链接 介绍的Supabase技术文档接入AI搜索后，能支持自然语言，能写代码，甚至能debug，估计很多人好奇它是怎么实现的。

以前写过一篇介绍原理 网页链接 ，这里再针对Supabase的集成了ChatGPT的AI文档检索写一篇。

集成ChatGPT的AI文档检索要解决两个问题：1. 检索；2. 对话

比如说如图一所示，我问了一个问题：“如何使用Nextjs连接supalbase， 请用中文回复”

第一步要做的就是把所有满足这个输入条件的文章都找出来。但按照以前全文检索的思路，分词、再查找，几乎不可能，因为原始文档都是英文的，而我输入的是中文，根本无法按照关键字去检索。

这里用的是一种Embeddings的技术，OpenAI针对Embeddings提供了API，输入文本后，可以得到一串数字。

那么什么是Embeddings呢？

Embeddings是用来测量“关联性”的，可以用来：
- 搜索：看你搜索的问题和一组文本的相似度如何
- 建议：两种产品的相似度如何
- 分类：如何对文本进行分类

比如说下面有三个短语：
1. "狗咬耗子"
2. "犬类捕食啮齿动物"
3. "我家养了只狗"

那很明显1和2是类似的，即使它们之间没有任何相同的文字。那么怎么让计算机知道1和2很相似呢？

OpenAI 的 Embeddings 是如何工作的？

当然我们不需要关心背后的神经网络模型，Embeddings将文本压缩成分布式的连续值数据（vectors向量）。

如果我们把之前的短语对应的向量能画在坐标轴上，它看起来像图2那样。短句1和短句2会离得很近，而短句3和他们离得比较远。

OpenAI提供了Embeddings的API，可以事先将所有的文档转成文本向量数据，然后将结果存储到支持向量的数据库。如果你数据不大，存成csv文件，然后加载到内存，借助内存搜索也是一样的。具体可以参考Kindle GPT的实现：网页链接

当用户提问的时候，把用户的问题也借助Embeddings API也变成文本向量，然后使用向量搜索，就能找出来哪些结果是接近的。比如我提的问题，文档中的“Use Supabase with NextJS
”就很接近。

借助Embeddings，就能帮助用户检索到想要的结果了。（参考图3的流程图）

但这还不够，因为光检索到结果，只能给用户返回文档，而不能按照用户的要求返回中文，甚至生成代码。

这时候就要借助ChatGPT的和prompt了。

ChatGPT是一个AI聊天机器人，它有一个庞大的知识库，它能理解用户的指令，能写代码，但是它对你的文档却一无所知，所以用户在提问时，你需要把匹配到的文档，生成prompt，喂给ChatGPT，让ChatGPT将“用户的问题”、“搜索到的文档”结合自己的知识库，返回给用户最终的结果。

继续以图一中我的问题为例，给ChatGPT的prompt大概长这样：

----------------------
你是一个非常热情的 Supabase 代言人，热爱帮助他人！请根据以下来自 Supabase 文档的部分内容，仅使用这些信息以 Markdown 格式回答问题。如果你不确定且答案未在文档中明确给出，请说“抱歉，我不知道如何帮助您。”

上下文部分：
Create a NextJS app
Create a Next.js app using the create-next-app command.
Install the Supabase client library
cd my-app && npm install @supabase/supabase-js
// 此处略去若干字..............

问题："如何使用Nextjs连接supalbase， 请用中文回复"

以 Markdown 格式回答（如果有的话，包括相关代码片段）：
----------------------

有了这些信息，就足够ChatGPT帮助你按照Supabase上匹配的文档，给你回复甚至生成代码了。

参考文档：网页链接

如果你需要开源的ChatGPT文档检索回复的代码实现，可以参考 gpt3.5-turbo-pgvector 这个项目：
🔗 github.com/gannonh/gpt3.5-turbo-pgvector
长图




# Domain-specific ChatGTP (gpt-3.5-turbo) Starter App

⚠️ UPDATE: Now uses the new "ChatGPT API" (model gpt-3.5-turbo). More on the new API: <https://platform.openai.com/docs/guides/chat> 

Use this starter app to build your own ChatGPT style app trained on specific websites that you define. Live demo: <https://astro-labs.app/docs>

## Overview

ChatGPT is great for casual, general-purpose question-answers but falls short when domain-specific knowledge is needed. Further, it makes up answers to fill its knowledge gaps and never cites its sources, so it can't really be trusted. This starter app uses embeddings coupled with vector search to solve this, or more specifically, to show how OpenAI's GPT-3 API can be used to create a conversational interfaces to domain-specific knowledge.

Embeddings, as represented by vectors of floating-point numbers, measure the "relatedness" of text strings. These are super useful for ranking search results, clustering, classification, etc. Relatedness is measured by cosine similarity. If the cosine similarity between two vectors is close to 1, the vectors are highly similar and point in the same direction. In the case of text embeddings, a high cosine similarity between two embedding vectors indicates that the corresponding text strings are highly related.

This starter app uses embeddings to generate a vector representation of a document, and then uses vector search to find the most similar documents to the query. The results of the vector search are then used to construct a prompt for GPT-3, which is then used to generate a response. The response is then streamed to the user. Check out the Supabase blog posts on [pgvector and OpenAI embeddings](https://supabase.com/blog/openai-embeddings-postgres-vector) for more background.

Technologies used:
- Nextjs (React framework) + Vercel hosting
- Supabase (using their pgvector implementation as the vector database)
- OpenAI API (for generating embeddings and GPT-3 responses)
- TailwindCSS (for styling)

## Functional Overview

Creating and storing the embeddings:
- Web pages are scraped, stripped to plain text and split into 1000-character documents
- OpenAI's embedding API is used to generate embeddings for each document using the "text-embedding-ada-002" model
- The embeddings are then stored in a Supabase postgres table using pgvector; the table has three columns: the document text, the source URL, and the embedding vectors returned from the OpenAI API.

Responding to queries:
- A single embedding is generated from the user prompt
- That embedding is used to perform a similarity search against the vector database
- The results of the similarity search are used to construct a prompt for GPT-3
- The GTP-3 response is then streamed to the user.

## Getting Started

The following set-up guide assumes at least basic familiarity developing web apps with React and Nextjs. Experience with OpenAI APIs and Supabase is helpful but not required to get things working.

### Set-up Supabase

- Create a Supabase account and project at https://app.supabase.com/sign-in. NOTE: Supabase support for pgvector is relatively new (02/2023), so it's important to create a new project if your project was created before then.
-  First we'll enable the Vector extension. In Supabase, this can be done from the web portal through ```Database``` → ```Extensions```. You can also do this in SQL by running:
```
create extension vector;
```
- Next let's create a table to store our documents and their embeddings. Head over to the SQL Editor and run the following query:
```sql
create table documents (
  id bigserial primary key,
  content text,
  url text,
  embedding vector (1536)
);
```
- Finally, we'll create a function that will be used to perform similarity searches. Head over to the SQL Editor and run the following query:
```sql
create or replace function match_documents (
  query_embedding vector(1536),
  similarity_threshold float,
  match_count int
)
returns table (
  id bigint,
  content text,
  url text,
  similarity float
)
language plpgsql
as $$
begin
  return query
  select
    documents.id,
    documents.content,
    documents.url,
    1 - (documents.embedding <=> query_embedding) as similarity
  from documents
  where 1 - (documents.embedding <=> query_embedding) > similarity_threshold
  order by documents.embedding <=> query_embedding
  limit match_count;
end;
$$;
```

### Set-up local environment

- clone the repo: ```gh repo clone gannonh/gpt3.5-turbo-pgvector```
- unzip and open in your favorite editor (the following assumes VS Code on a Mac)
```bash
cd gpt3.5-turbo-pgvector
code .
```
- install dependencies
```bash
npm install
```
- create a .env.local file in the root directory to store environment variables:
```bash
cp .env.local.example .env.local
```
- open the .env.local file and add your Supabase project URL and API key. You can find these in the Supabase web portal under ```Project``` → ```API```. The API key should be stored in the ```SUPABASE_ANON_KEY``` variable and project URL should be stored under ```NEXT_PUBLIC_SUPABASE_URL```.
- Add your OPENAI PI key to .env.local. You can find this in the OpenAI web portal under ```API Keys```. The API key should be stored in the ```OPENAI_API_KEY``` variable.
- Start the app
```bash
npm run dev
```
- Open http://localhost:3000 in your browser to view the app.
