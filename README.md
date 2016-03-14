ProtVec 
--------------------------------------

Paper: [ProtVec: A Continuous Distributed Representation of
Biological Sequences](http://arxiv.org/pdf/1503.05140v1.pdf)

### 概略
通常生物情報は文字の配列で表現されるが、それをベクトルとして表現することによってより分析しやすく情報を収納することができるのではないかと提案されている。具体的な適用範囲としては、

1. family classification
2. protein visualization
3. structure prediction
4. disordered protein identification
5. protein-protein interaction prediction.

など。
classificationやpredictionはわかりやすい使い方だが、個人的にはprotein visualizationが最も効用が大きいのではないかと感じた。短い配列や、構造が既知の配列でない限り、現状簡単にタンパク質の全容を掴む方法が一般的に普及していないように感じるので、このような表現方法は一定の有用性があると考える。
この考えは一見奇妙に映るが自然言語ではある程度認知されており、word2vecなどは記憶に新しい。

### 実装

[前処理]
* uniprotのswis-protの全データを収集する
* 各配列を3つのn-gramのリストに変換する

``` 
'AGAMQSASM' => [['AGA', 'MQS', 'ASM'], ['GAM','QSA'], ['AMQ', 'SAS']]
```

* word2vecに読み込ませるために、変換した配列をテキストファイル形式に書き出す

[モデル構築]

word2vecのライブラリを用いれば基本大丈夫そう。
いろいろあるけど、gensimをここでは使う。ただしSkip-gramを論文では採用しているので注意。
gensimではsgパラメータを1に設定するとskip-gramになる。
> sg defines the training algorithm. By default (sg=0), CBOW is used. Otherwise (sg=1), skip-gram is employed.

前処理では上記のように単語をn-gramのリストへ変換するが、sequenceをqueryとしてモデルに投げるときは逆に再変換する必要がある。訓練済みモデルにn-gramを指定することで、対応するベクトルを得ることができる。論文ではそのベクトルの和をその配列に対応するベクトルとして扱っている。

```
seq = 'AGAMQSASM'
n_grams = split_to_grams(seq) 
gram_vecs = [to_gram_vec(n_gram) for n_gram in n_grams]
seq_vec = sum(gram_vecs)
```

論文での、ベクトルの次元数は100。
元の配列に対応するベクトルはn-gramベクトルの和なので次元数は変わらず、100次元。
negative samplingも行っているので忘れずに。


### Abstract of the paper

> We propose a new approach for representing biological sequences. This method, named protein-vectors or ProtVec for short, can be utilized in bioinformatics applications such as family classification, protein visualization, structure prediction, disordered protein identification, and protein-protein interaction prediction. Using the Skip-gram neural networks, protein sequences are represented with a single dense n-dimensional vector. This method was evaluated by classifying protein sequences obtained from Swiss-Prot belonging to 7,027 protein families where an average family classification accuracy of 94%±0.03% was obtained, outperforming existing family classification methods. In addition, our model was used to predict disordered proteins from structured proteins. Two databases of disordered sequences were used: the DisProt database as well as a database featuring the disordered regions of nucleoporins rich with phenylalanine-glycine repeats (FG-Nups). Using support vector machine classifiers, FG-Nup sequences were distinguished from structured Protein Data Bank (PDB) sequences with 99.81\% accuracy, and unstructured DisProt sequences from structured DisProt sequences with 100.0\% accuracy. These results indicate that by only providing sequence data for various proteins into this model, information about protein structure can be determined with high accuracy. This so-called embedding model needs to be trained only once and can then be used to ascertain a diverse set of information regarding the proteins of interest. In addition, this representation can be considered as pre-training for various applications of deep learning in bioinformatics.

### References
1. [Disordered Proteins](https://en.wikipedia.org/wiki/Intrinsically_disordered_proteins)
2. [DisProt](http://www.disprot.org/)
3. [gemsim word2vec](https://radimrehurek.com/gensim/models/word2vec.html)
4. [NIPS2013読み会: Distributed Representations of Words and Phrases and their Compositionality](http://www.slideshare.net/unnonouno/nips2013-distributed-representations-of-words-and-phrases-and-their-compositionality)
5. [Skip gram shirakawa_20141121
](http://www.slideshare.net/nttdata-msi/skip-gram-shirakawa20141121-41833306)
6. [論文紹介「Distributed Representations of Words and Phrases and their Compositionality」](http://qiita.com/nishio/items/3860fe198d65d173af6b)