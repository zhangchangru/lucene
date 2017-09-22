# lucene
关于全文检索lucene的简单运用
首先要导入lucence相关jar包
//建索引部分代码
IndexWriter writer = null;

		try {
			Directory directory = FSDirectory.open(new File("C:/lucenetest/index"));
			IndexWriterConfig iwConfig = new IndexWriterConfig(Version.LUCENE_36, new StandardAnalyzer(Version.LUCENE_36));
			writer = new IndexWriter(directory, iwConfig);
			iwConfig.setOpenMode(OpenMode.CREATE);
			Document document = null;
			File f = new File("C:/lucenetest/files");// 索引源文件位置
			long startTime = new Date().getTime();
			for (File file : f.listFiles()) {
				document = new Document();
				System.out.println("File " + file.getAbsolutePath() + "正在被索引....");
				document.add(new Field("path", file.getAbsolutePath(), Field.Store.YES, Field.Index.ANALYZED,
						Field.TermVector.WITH_POSITIONS_OFFSETS));
				System.out.println(file.getName());
				String content = FileReaderAll(file.getAbsolutePath(), "utf-8");
				document.add(new Field("body", content, Field.Store.YES, Field.Index.ANALYZED, Field.TermVector.WITH_POSITIONS_OFFSETS));
				writer.addDocument(document);
			}
			writer.close();
			long endTime = new Date().getTime();
			System.out.println("用时：" + (endTime - startTime));
		} catch (IOException e) {
			e.printStackTrace();
		} finally {
			try {
				writer.close();
			} catch (CorruptIndexException e) {
				e.printStackTrace();
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
    
    //查询部分代码
    public static void main(String[] args) {
		String indexFile = "C:/lucenetest/index";
		try {
			if (new File(indexFile).length() <= 1) {
				System.out.println("文件不存在");
				return;
			}
			IndexReader reader = IndexReader.open(FSDirectory.open(new File(indexFile)));
			IndexSearcher searcher = new IndexSearcher(reader);
			ScoreDoc[] hits = null;
			String queryString = "海上";
			// 指定多个默认搜索域
			// String[] fields = { "path", "body" };
			// MultiFieldQueryParser queryParser = new MultiFieldQueryParser(Version.LUCENE_36, fields,
			// new StandardAnalyzer(Version.LUCENE_36));
			Query query = null;
			Analyzer analyzer = new StandardAnalyzer(Version.LUCENE_36);
			QueryParser qp = new QueryParser(Version.LUCENE_36, "body", analyzer);
			query = qp.parse(queryString);
			SimpleHTMLFormatter formatter = new SimpleHTMLFormatter("<span class=\"highlight\">", "</span>");
			Scorer scorer = new QueryScorer(query);
			Highlighter hig = new Highlighter(formatter, scorer);
			try {
				System.out.println("111:" + hig.getBestFragment(analyzer, "body", query.toString()));
			} catch (InvalidTokenOffsetsException e) {
				e.printStackTrace();
			}
			if (searcher != null) {
				TopDocs results = searcher.search(query, 10); // 返回最多为10条记录
				hits = results.scoreDocs;

				if (hits.length > 0) {
					System.out.println("找到:" + hits.length + " 个结果!");
				}
				for (int i = 0; i < hits.length; i++) {
					System.out.println(hits[i]);
					System.out.println("权重：" + hits[i].score);
					System.out.println("555:" + searcher.doc(hits[i].doc).get("path"));
					System.out.println("666:" + searcher.doc(hits[i].doc).get("body"));
				}
				searcher.close();
			}
		} catch (CorruptIndexException e) {
			e.printStackTrace();
		} catch (IOException e) {
			e.printStackTrace();
		} catch (ParseException e) {
			e.printStackTrace();
		}

	}
    
