# Start Elastic Search

elastic search はじめてみるぞ! のリポジトリ

## Requirements

* docker
* docker-compose

## Setup

```bash
docker-compose build
docker-compose up -d
```

以上実行するとサーバーが3つ立ち上がります。

[NOTE]: jupyter で使っている image が若干重たいので少し時間がかかるかも知れません。 build しながらディレクトリ構成とか docker-compose.yml の構造, elasticsearch のドキュメントなど読んで時間が余ったら珈琲を飲みましょう。

## elasticsearch

今回の主題である elasticsearch は localhost:9300 でつながります。これを直接 REST API で叩いても当然つかうことが出来ます。
定義ファイルは `./elasticsearch` ディレクトリに入っています。

詳しくは elasticsearch 公式 docker document を参考にして下さい 

> https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html   
> Production Usage の時の注意点 https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html#_notes_for_production_use_and_defaults とかもあります。

## kibana

elasticsearch 用のクライアントアプリケーションです。 localhost:5601 で kibana container とつながっています。

kibana の container はデフォルトで `elasticsearch:9300` につなごうとするので elasticsearch も default の 9300 で起動するよう特に何もいじっていません。
設定ファイルを弄りたい場合には environment に記述する若しくは `kibana.yml` を bind して up すると良いです。詳しくは kibana の docker document を参考にして下さい。

> https://www.elastic.co/guide/en/kibana/current/docker.html

[http://localhost:5601](http://localhost:5601) にアクセスすると kibana server にアクセスできます。

## jupyter

python から elasticsearch 使いたいよね、ということで jupyter notebook server を用意しています。  
jupyter には [http://localhost:8888/tree/notebooks](http://localhost:8888/tree/notebooks) からアクセスできます。 password は `"dolphin"` です。

python 環境には elasticsearch が pip で入っていますので python client はデフォルトで使うことが出来ます。

## Sample notebook

いくつかサンプルのコードが入っていますので参考にして下さい。

### 特徴ベクトルでの類似度検索

`~/client/notebooks/sample_search_by_cosine-sim.ipynb`  に特徴ベクトルでの類似度検索を行うサンプルが入っています。  
128 次元のベクトルを 100000 件登録して、クエリのベクトルとコサイン類似度の意味で近い順にソートする、ということを行っています。

例えば以下のような例が応用でしょうか。

* 画像の特徴ベクトルでの類似度検索  
   (クエリ画像と、マスタ画像の類似度を arcface や center-loss などの metric learning で得たモデルの特徴量でソート)
* 文章を表す特徴量でのソート  
  (SWEM など文章を埋め込む(embedding)手法)

#### Result

検索の時間は以下のようになりました。 elasticsearch 爆足だぜ :D

| method                                   | %%timeit                                                              |
|------------------------------------------|-----------------------------------------------------------------------|
| scipy.spatial.distance & list内包記法    | 3.27 s ± 17.8 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)   |
| scipy.spatial.distance & joblib parallel | 5.2 s ± 108 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)     |
| elasticsearch                            | 43.7 ms ± 661 µs per loop (mean ± std. dev. of 7 runs, 10 loops each) |
