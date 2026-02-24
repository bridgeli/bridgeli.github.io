---
title: 全文检索工具-Lucene（solr）入门
author: Bridge Li
type: post
date: 2014-11-02T14:43:20+00:00

duoshuo_thread_id:
  - 1.1604454626757E+18
categories:
  - Java
tags:
  - maven lucene solr
---
最近闲着没事在写微信公众号，其中一个是聊天机器人，和网上的众多机器人原理一样，但是功能没那么强大（主要是只是库不够强大），但是怎么解决“如何根据用户的问题从回答库中找出最匹配的答案呢？”，大家最先想到的也许是数据库的 LIKE 就好了嘛，但是　LIKE 存在如下问题：  
1. 在问答库非常庞大的时候，LIKE 的效率会非常非常的慢；  
2. LIKE只适用于关键字匹配，并不适合自然语言匹配。举个例子：用户的问题“河南的省会是哪个城市？”，而数据库的的记录是“河南的省会是哪”，虽然无论是从字面上还是意义上都一样，都 LIKE 却无能为力；  
3. LIKE 无法计算相似度。也就是说 LIKE 返回多条记录时，无法确定那个是最佳答案所以此时全文检索引擎的优越性就体现出来了。

全文检索引擎的原理：扫描知识库的每一条记录并分词建立索引，索引记录了词在每一条记录中出现的位置和次数，当收到用户的问题时，也进行分词，然后从索引中找出包含这些词的所有记录，再分别计算相似度，然后可以找出相似度最高的一条记录返回给用户，下面老夫给出一个自己用Lucene（solr）写的例子，这个例子经验证是可以直接跑起来的，至于其众多API，大家可以自己去查官网文档，其实很容易理解。

```  
import java.io.File;
import java.io.IOException;
import java.util.List;

import javax.annotation.PostConstruct;
import javax.annotation.Resource;

import org.apache.lucene.document.Document;
import org.apache.lucene.document.Field;
import org.apache.lucene.document.IntField;
import org.apache.lucene.document.TextField;
import org.apache.lucene.index.IndexWriter;
import org.apache.lucene.index.IndexWriterConfig;
import org.apache.lucene.store.Directory;
import org.apache.lucene.store.FSDirectory;
import org.apache.lucene.util.Version;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Service;
import org.wltea.analyzer.lucene.IKAnalyzer;

import cn.bridgeli.livingsmallhelper.entity.Knowledge;
import cn.bridgeli.livingsmallhelper.mapper.KnowledgeMapper;
import cn.bridgeli.livingsmallhelper.service.ChatService;

public class SolrTest {
    private static final Logger LOG = LoggerFactory.getLogger(DataInit.class);

    private KnowledgeMapper knowledgeMapper;
    @Resource
    private ChatService chatService;

    public void createIndex() {

        List<Knowledge> knowledges = knowledgeMapper.query();

        Directory directory = null;
        IndexWriter indexWriter = null;
        File indexFile = new File(chatService.getIndexDir());
        if (!indexFile.exists()) {

            try {
                directory = FSDirectory.open(indexFile);
                IndexWriterConfig indexWriterConfig = new IndexWriterConfig(Version.LUCENE_46, new IKAnalyzer(true));
                indexWriter = new IndexWriter(directory, indexWriterConfig);
                for (Knowledge k : knowledges) {
                    Document document = new Document();
                    document.add(new TextField("question", k.getQuestion(), Field.Store.YES));
                    document.add(new IntField("id", k.getId(), Field.Store.YES));
                    document.add(new TextField("answer", k.getAnswer() == null ? "" : k.getAnswer(), Field.Store.YES));
                    document.add(new IntField("category", k.getCategory(), Field.Store.YES));

                    indexWriter.addDocument(document);
                }
                indexWriter.commit();
            } catch (IOException e) {
                LOG.error("IOException", e);
            } finally {
                try {
                    if (null != indexWriter) {
                        indexWriter.close();
                    }
                    if (null != directory) {
                        directory.close();
                    }
                } catch (IOException e) {
                    LOG.error("IOException", e);
                }
            }

        }
    }

    @SuppressWarnings("deprecation")
    private Knowledge searchIndex(String content) {
        Knowledge knowledge = null;
        Directory directory = null;
        IndexReader reader = null;
        try {
            directory = FSDirectory.open(new File(getIndexDir()));
            reader = IndexReader.open(directory);
            IndexSearcher searcher = new IndexSearcher(reader);
//这个question 就是以问题为索引去查找，和CreateIndex()中相对应  
            QueryParser queryParser = new QueryParser(Version.LUCENE_46, "question", new IKAnalyzer(true));
            Query query = queryParser.parse(QueryParser.escape(content));
//这个1 的含义就是找出最相似度最高的一条  
            TopDocs topDocs = searcher.search(query, 1);
            if (topDocs.totalHits > 0) {
                knowledge = new Knowledge();
                ScoreDoc[] scoreDocs = topDocs.scoreDocs;
                for (ScoreDoc scoreDoc : scoreDocs) {
                    Document doc = searcher.doc(scoreDoc.doc);
                    knowledge.setId(doc.getField("id").numericValue().intValue());
                    knowledge.setQuestion(doc.get("questory"));
                    knowledge.setAnswer(doc.get("answer"));
                    knowledge.setCategory(doc.getField("category").numericValue().intValue());
                }

            }
        } catch (IOException e) {
            LOG.error("IOException", e);
        } catch (ParseException e) {
            LOG.error("ParseException", e);
        } finally {
            try {
                reader.close();
                directory.close();
            } catch (IOException e) {
                LOG.error("IOException", e);
            }
        }
        return knowledge;
    }

    @Override
    public String getIndexDir() {
        String classpath = SolrTest.class.getResource("/").getPath();
        classpath = classpath.replaceAll("%20", " ");
        LOG.warn("==================" + classpath);
        return classpath + "index/";
    }

    @Test
    public void testSolr() throws IOException, ParseException {

        SolrTest solrTest = new SolrTest();
        File indexDir = new File(getIndexDir());
        if (!indexDir.exists()) {
            solrTest.createIndex(indexDir);
        }
        solrTest.searchIndex("你好啊");
    }

}
```

因为我是用maven写的，所以对应的pom文件，如下：  
```  
<dependency>  
  <groupId>org.apache.lucene</groupId>  
  <artifactId>lucene-core</artifactId>  
  <version>4.6.0</version>  
</dependency>

<dependency>  
  <groupId>org.apache.lucene</groupId>  
  <artifactId>lucene-queryparser</artifactId>  
  <version>4.6.0</version>  
</dependency>

<dependency>  
  <groupId>org.wltea</groupId>  
  <artifactId>IKAnalyzer</artifactId>  
  <version>2012FF_u1</version>  
</dependency>

```  
最后需要说明的是，maven的中央仓库竟然没有IKAnalyzer这个包，而这个包是中文分词的最为关键的一个包，不过所幸我发现osc的maven 仓库中有，所以大家可以去osc的maven库中去查找这个类库。