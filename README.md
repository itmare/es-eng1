---

ES1
===

---

0.Indexing data
---------------

#### blogs index

##### logstash를 통해, blog 데이터 indexing

-	logstash 설치

```shell
wget https://artifacts.elastic.co/downloads/logstash/logstash-6.4.0.rpm
rpm -vi logstash-6.4.0.rpm

cd /usr/share/logstash
bin/logstash --version
```

-	데이터 업로드

```shell
scp -i ~/.ssh/itmare3_rsa /Users/jinwookchung/Desktop/git-note/es-engineer/datasets/blogs.csv itmare3@es01:/home/itmare3/
scp -i ~/.ssh/itmare3_rsa /Users/jinwookchung/Desktop/git-note/es-engineer/datasets/blogs.conf itmare3@es01:/home/itmare3/

cd /usr/share/logstash
mkdir datasets
cd datasets
mv /home/itmare3/blogs
mv /home/itmare3/blogs.csv /usr/share/logstash/datasets/
mv /home/itmare3/blogs.conf /usr/share/logstash/datasets/
```

-	indexing

```shell
cd /usr/share/logstash
vi blogs.conf #==> csv파일 위치로 경로 수정
bin/logstash -f datasets/blogs.conf
```

```shell
vi ~/.bash_profile

export LS_HOME=/home/ec2-user/logstash
PATH=$PATH:$LS_HOME/bin
```

<br>

###### logs_server

-	filebeat를 통해, log 데이터 indexing

-	데이터 업로드

```shell
scp -i ~/.ssh/itmare3_rsa /Users/jinwookchung/Desktop/git-note/es-engineer/datasets/elastic_blog_curated_access_logs.tar.gz itmare3@es01:/home/itmare3/
scp -i ~/.ssh/itmare3_rsa /Users/jinwookchung/Desktop/git-note/es-engineer/datasets/filebeat.yml itmare3@es01:/home/itmare3/

cd /etc/filebeat
mkdir datasets
mv /home/itmare3/elastic_blog_curated_access_logs.tar.gz /etc/filebeat/datasets
tar -zxvf /etc/filebeat/datasets/elastic_blog_curated_access_logs.tar.gz
mv /home/itmare3/filebeat.yml /etc/filebeat/
```

-	indexing

```shell

cd /home/elastic/
vi filebeat.yml #==> log파일 경로 수정
filebeat -e -c filebeat.yml
```

<br><br>

1.Elastic Stack Overview
------------------------

2.Getting Started with Elasticsearch
------------------------------------

```shell
GET /

# response
{
  "name": "Y-fW5AH",
  "cluster_name": "elasticsearch",
  "cluster_uuid": "14TRgGKJSA2zI0wsP98DAg",
  "version": {
    "number": "6.3.1",
    "build_flavor": "default",
    "build_type": "tar",
    "build_hash": "eb782d0",
    "build_date": "2018-06-29T21:59:26.107521Z",
    "build_snapshot": false,
    "lucene_version": "7.3.1",
    "minimum_wire_compatibility_version": "5.6.0",
    "minimum_index_compatibility_version": "5.0.0"
  },
  "tagline": "You Know, for Search"
}
```

###### es 인스턴스 버전은?

<details><summary> 정답 </summary> - 6.3.1</details>

###### 노드의 이름은?

<details><summary> 정답 </summary> - Y-fW5AH</details>

###### 클러스터의 이름은?

<details><summary> 정답 </summary> - elasticsearch</details>

###### 어떻게 노드와 클러스터의 이름이 할당 되었나?

<details><summary> 정답 </summary> - 노드는 랜덤생성, 클러스터 이름의 default는 elasticsearch</details>

<br>

```shell
GET /_cat/indices?v

# response
health status index        uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   logs_server3 RIQi8NtxROCeTtNIgpjF2A   5   1     584814            0    203.9mb        203.9mb
yellow open   blogs        Kprg3NgVQ_uMLKFtgvu5YA   5   1       1594            0      6.2mb          6.2mb
yellow open   logs_server2 dbVJ3RxTTQagltf3iKp-FA   5   1     584599            0    202.7mb        202.7mb
yellow open   logs_server1 lSCHs2CURZ-VnkjyxoyEyw   5   1     582063            0    198.8mb        198.8mb
green  open   .kibana      n59ZjM5KTDiVl7bBXzfROg   1   0          5            1     30.1kb         30.1kb
```

###### `?v`를 안 붙이고 실행하면?

<details><summary> 정답 </summary> - 컬럼의 header가 사라진다.</details>

###### 인덱스의 수와 그것들의 이름은?

<details><summary> 정답 </summary>

-	5개, index 컬럼의 이름  

</details>

###### blogs인덱스에 몇개의 도큐먼트가 있나?

<details><summary> 정답 </summary> - 1594</details>

<br>

| id | title                  | category    | date          | author_first_name | author_last_name | author_company |
|----|------------------------|-------------|---------------|-------------------|------------------|----------------|
| 1  | Better query execution | Engineering | July 15, 2015 | Adrien            | Grand            | Elastic        |
| 2  | The Story of sense     |             | May 28, 2015  | Boaz              | Leskes           |                |

###### 첫번째 row를 json document로 표현?

<details><summary> 정답 </summary>

```shell
{
  "id": "1",
  "title": "Better query execution",
  "category": "Engineering",
  "date":"July 15, 2015",
  "author":{
    "first_name": "Adrian",
    "last_name": "Grand",
    "company": "Elastic"
  }
}
```

</details>

###### 두번째 row를 json document로 표현

<details><summary> 정답 </summary>

```shell
# empty field는 생략하거나, null(생략과 같음), "" (empty string)
{
  "id": "2",
  "title": "The Story of Sense",
  "date":"May 28, 2015",
  "author":{
    "first_name": "Boaz",
    "last_name": "Leskes"
  }
}
```

</details>

<br>

###### 상위 json데이터를 indexing하기, id field를 다른 field와 마찬가지로 document에 인덱싱하고, URL에도 추가하기, index이름을 my_blogs로 하고 \_doc과 id 추가하기.

<details><summary> 정답 </summary>

```shell
PUT my_blogs/_doc/1
{
  "id": "1",
  "title": "Better query execution",
  "category": "Engineering",
  "date":"July 15, 2015",
  "author":{
    "first_name": "Adrian",
    "last_name": "Grand",
    "company": "Elastic"
  }
}
PUT my_blogs/_doc/2
{
  "id": "2",
  "title": "The Story of Sense",
  "date":"May 28, 2015",
  "author":{
    "first_name": "Boaz",
    "last_name": "Leskes"
  }
}
```

</details>

<br>

###### post를 사용해서 다음을 인덱싱해보기

```shell
{
  "id": "57",
  "title": "Phrase Queries: a world without Stopwords",
  "date":"March 7, 2016",
  "category": "Engineering",
  "author":{
    "first_name": "Gabriel",
    "last_name": "Moskovicz"
  }
}
```

<details><summary> 정답 </summary>

```shell
POST my_blogs/_doc/
{
  "id": "57",
  "title": "Phrase Queries: a world without Stopwords",
  "date":"March 7, 2016",
  "category": "Engineering",
  "author":{
    "first_name": "Gabriel",
    "last_name": "Moskovicz"
  }
}
```

</details>

<br>

###### GET command를 사용해서 id가 1인 document 가져오기, index는 my_blogs, type은 \_doc을 사용)

<details><summary> 정답 </summary>

```shell
GET my_blogs/_doc/1
```

</details>

<br>

###### my_blogs 인덱스에서 id 2 document를 삭제하기

```shell
# 제대로 삭제 됐다면, GET으로 id 2를 호출할때, 다음과 같은 response
{
  "_index": "my_blogs",
  "_type": "_doc",
  "_id": "2",
  "found": false
}
```

<details><summary> 정답 </summary>

```shell
DELETE my_blogs/_doc/2
```

</details>

<br>

###### my_blogs 인덱스를 삭제하기

<details><summary> 정답 </summary>

```shell
DELETE my_blogs
```

</details>

<br><br><br><br>

3.Querying Data
---------------

###### blogs 인덱스에 있는 모든 document들을 query하기

```shell
# response
{
  "hits":{
    "total":1594
  }
}
```

<details><summary> 정답 </summary>

```shell
GET blogs/_search
{
  "query":{
    "match_all":{}
  }
}
```

</details>

<br>

###### size 파라미터를 추가해서 100개의 response를 출력하기

<details><summary> 정답 </summary>

```shell
GET blogs/_search
{
  "size": 100,
  "query":{
    "match_all":{}
  }
}
```

</details>

<br>

###### May, 2017년 5월에 publish된 blog post를 query하기, (response: 44 hits)

<details><summary> 정답 </summary>

```shell
GET blogs/_search
{
  "query":{
    "range":{
      "publish_date":{
        "gte":"2017-05-01",
        "lte":"2017-05-31"
      }
    }
  }
}
```

</details>

<br>

###### match query를 사용해서 **title** field에서 "elastic" 단어가 있는 blog를 찾기 (response: 259 hits)

<details><summary> 정답 </summary>

```shell
GET blogs/_search
{
  "query":{
    "match":{
      "title":"elastic"
    }
  }
}
```

</details>

<br>

###### **match** query를 사용해서 **title** field의 "elastic stack"을 query했을때, hit의 수가 증가(262)하는 이유는 뭘까?

<details><summary> 정답 </summary>

-	"elastic stack"을 match로 query하면, elastic 또는 stack을 포함하는 blog를 리턴한다. 즉, title에 stack이란 단어만 가지고 있는 blog도 리턴한다.

</details>

<br>

###### **operator**를 사용해서 **title** field의 "elastic stack"을 쿼리할떄 결과값이 elastic과 stack을 모두 포함하게 쿼리하기 (response: 70 hits)

<details><summary> 정답 </summary>

```shell
GET blogs/_search
{
  "query":{
    "match":{
      "title":{
        "query":"elastic stack",
        "operator":"and"
      }
    }
  }
}
```

</details>

<br>

###### **content** field에 "search"를 포함한 blog 쿼리하기 (432 hits)

<details><summary> 정답 </summary>

```shell
GET blogs/_search
{
  "query":{
    "match":{
      "content":"search"
    }
  }
}
```

</details>

<br>

###### **content** field에 "search" 또는 "analytics"를 포함한 blog 쿼리하기 (476 hits)

<details><summary> 정답 </summary>

```shell
GET blogs/_search
{
  "query":{
    "match":{
      "content":"search analytics"
    }
  }
}
```

</details>

<br>

###### **content** field에 "search" 와 "analytics" 모두 포함한 blog 쿼리하기 (110 hits)

<details><summary> 정답 </summary>

```shell
GET blogs/_search
{
  "query":{
    "match":{
      "content":{
        "query":"search analytics",
        "operator":"and"
      }
    }
  }
}
```

</details>

<br>

###### **match_phrase** search를 사용해서 "search analytics" phrase를 **content** field에서 상위 3개만 리턴하게 쿼리하기 (6 hits)

<details><summary> 정답 </summary>

```shell
GET blogs/_search
{
  "size":3,
  "query":{
    "match_phrase":{
      "content":"search analytics"
    }
  }
}
```

</details>

<br>

###### **match_phrase** search를 사용해서 "search"와 "analytics" 사이에 하나의 단어가 추가되는 것을 허락하는 phrase 쿼리하기 (41 hits)

<details><summary> 정답 </summary>

```shell
GET blogs/_search
{
  "query":{
    "match_phrase":{
      "content":{
        "query":"search analytics",
        "slop": 1
      }
    }
  }
}
```

</details>

<br>

###### **content** field에서 performance 또는 optimizations 또는 improvements를 가진 blog를 쿼리하기 (374 hits)

<details><summary> 정답 </summary>

```shell
GET blogs/_search
{
  "query":{
    "match":{
      "content":"performance optimizations improvements"
      }
    }
  }
}
```

</details>

<br>

###### **content** field에서 performance, optimizations 또는 improvements 중 적어도 2개의 단어를 포함하는 blog 쿼리하기 (82 hits)

<details><summary> 정답 </summary>

```shell
GET blogs/_search
{
  "query":{
    "match":{
      "content":{
        "query":"performance optimizations improvements",
        "minimum_should_match": 2
      }
    }
  }
}
```

</details>

<br>

###### 위의 결과값을 튜닝, **must_not** 옵션을 사용해서, **title** 에 release, releases 또는 released을 포함하는 blog를 제외하고 쿼리하기 (47 hits)

<details><summary> 정답 </summary>

```shell
GET blogs/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "content": {
              "query":"performance optimizations improvements",
              "minimum_should_match": 2
            }
          }
        }
      ],
      "must_not": [
        {
          "match": {
            "title": "release releases released"
          }
        }
      ]
    }
  }
}
```

</details>

<br>

-	must: 문서가 모든 필터에 매치될때 (AND)
-	should: 문서가 하나의 필터라도 매치될때 (OR)
-	must_not: 문서가 필터에 매치되지 않을때 (NOT)

4.Text Analysis and Mappings
----------------------------

###### 아래 document를 가지고 **tmp_blogs** 라는 이름으로 새로운 index 만들기

```shell
{
  "title": "Elastic Cloud and Meltdown",
  "publish_date": "2018-01-07T23:00:00.000Z",
  "author": "Elastic Engineering",
  "category": "Engineering",
  "content": " Elastic is aware of the Meltdown and Spectre vulnerabilities...",
  "url": "/blog/elastic-cloud-and-meltdown",
  "locales": ["de-de","fr-fr"]
}
```

<details><summary> 정답 </summary>

```shell
PUT tmp_blogs/_doc/1
{
  "title": "Elastic Cloud and Meltdown",
  "publish_date": "2018-01-07T23:00:00.000Z",
  "author": "Elastic Engineering",
  "category": "Engineering",
  "content": " Elastic is aware of the Meltdown and Spectre vulnerabilities...",
  "url": "/blog/elastic-cloud-and-meltdown",
  "locales": ["de-de","fr-fr"]
}
```

</details>

<br>

###### 자동생성된 **tmp_blogs** 의 mapping 보기

<details><summary> 정답 </summary>

```shell
GET tmp_blogs/_mapping

# response
{
  "tmp_blogs": {
    "mappings": {
      "_doc": {
        "properties": {
          "author": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "category": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "content": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "locales": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "publish_date": {
            "type": "date"
          },
          "title": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "url": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          }
        }
      }
    }
  }
}
```

</details>

<br>

##### Field Datatype

-	**keyword datatype**: 정확한 값으로 검색할때 사용,
	-	sorting 가능, aggregation 가능, analyze 불가능
	-	ex) email address, hostname, status code, zip code, tag (filtering, ex: `status`가 `published`인 모든 blog포스트를 찾을때)
-	**text datatypes**: full-text로 인덱싱되는 필드타입, `analyzer`를 통해 단어로 검색가능하도록 인덱싱 됨
	-	sorting 불가능, aggregation 불가능, analyze 가능
	-	ex) email본문, 상품설명

<br>

###### 상위 mapping에서 볼때, data type관련 무엇을 변경해야 할까?

<details><summary> 정답 </summary>

-	title: 일반 string이므로 **text**, 하지만 검색과 정렬이 필요할지모르니 `fields` parameter를 사용해 **keyword** 추가 설정
-	publish_date: **date** 으로 설정
-	author: 일반 string이므로 **text**, 하지만 검색과 정렬이 필요할지모르니 `fields` parameter를 사용해 **keyword** 추가 설정
-	category: 검색 필요, 일반적으로 카테고리는 드랍다운메뉴 또는 리스트, 고로 **keyword** 로 설정
-	content: 정렬(sorting)과 집계(aggregation)가 필요없음, 고로 **text** 로 설정
-	url: 일반 string이므로 **text**, 하지만 검색과 정렬이 필요할지모르니 `fields` parameter를 사용해 **keyword** 추가 설정
-	locales:

</details>

###### **tmp_blogs** 를 삭제하고, 위의 근거를 토대로 **tmp_blogs** index를 다시 mapping하기

<details><summary> 정답 </summary>

```shell
PUT tmp_blogs
{
  "mappings": {
    "_doc":{
      "properties":{
        "title":{
          "type":"text",
          "fields":{
            "keyword":{
              "type":"keyword",
              "ignore_above":256
            }
          }
        },
        "publish_date":{
          "type":"date"
        },
        "author":{
          "type":"text",
          "fields":{
            "keyword":{
              "type":"keyword",
              "ignore_above":256
            }
          }
        },
        "category":{
          "type":"keyword"
        },
        "content":{
          "type":"text"
        },
        "url":{
          "type":"text",
          "fields":{
            "keyword":{
              "type":"keyword",
              "ignore_above":256
            }
          }
        },
        "locales":{
          "type":"keyword"
        }
      }
    }
  }
}
```

</details>

###### 아래 샘플 document를 indexing하고, **category** field에서 **engineering** 과 **Engineering** 을 검색해 보기, 어떤것이 리턴되고 리턴되지 않는지 확인해보고, 그 이유는?

```shell
PUT tmp_blogs/_doc/1
{
  "title": "Elastic Cloud and Meltdown",
  "publish_date": "2018-01-07T23:00:00.000Z",
  "author": "Elastic Engineering",
  "category": "Engineering",
  "content": " Elastic is aware of the Meltdown and Spectre vulnerabilities...",
  "url": "/blog/elastic-cloud-and-meltdown",
  "locales": ["de-de","fr-fr"]
}
```

<details><summary> 정답 </summary>

```shell
GET tmp_blogs/_search
{
  "query":{
    "match":{
      "category": "engineering"   # 또는 "Engineering"
    }
  }
}
```

-	**keyword** field에는 text analysis이 없다. "engineering"은 **category** field의 값과 매치되지 않는다.

</details>

###### 다음 문장과 함께 **_analyze** API 사용해 보기

```shell
"Introducing beta releases: Elasticsearch and Kibana Docker images!"
```

<details><summary> 정답 </summary>

```shell
GET _analyze
{
  "text": "Introducing beta releases: Elasticsearch and Kibana Docker images!"
}
```

</details>

###### default analyzer는 무엇이었나?, **whitespace** analyzer와 무슨 차이가 있나?

<details><summary> 정답 </summary> - **standard** analyzer

```shell
GET _analzye
{
  "analyzer": "whitespace",
  "text": "Introducing beta releases: Elasticsearch and Kibana Docker images!"
}
```

-	**whitespace** analyzer는 대문자를 소문자로 변경하지 않고, punctuation을 제거하지 않는다.

</details>

##### built-in analyzers

-	Standard Analyzer: default analzyer, punctuation 제거, 대문자를 소문자로, stop word 제거
-	Whitespace Analyzer: whitespace로만 term 분리
-	Simple Analyzer: 모든 알파벳은 소문자로, punctuation 제거
-	Stop Analyzer: simple analzyer에 stop word 제
-	Keyword Analyzer: 어떤 text이던간에 single term으로써 정확히 같은 text를 리턴한다.

###### 다음을 analyze했을때, **lowercaser** 와 **snowball** filter와 함께 **standard** tokenizer를 사용하기, **english** analyzer와의 차이점은?

```shell
"text": "This release includes mainly bug fixes."
```

<details><summary> 정답 </summary>

-	**english** analyzer는 **snowball** stemmer를 사용하지 않고, **english_stemmer** 를 사용한다.
	-	english stemmer: mainly => mainli
	-	snowball stemmer: mainly => main

</details>

<br>

###### **_analyzer** 를 사용해서, 다음을 만족하는 analyzer 만들기

-	**standard** tokenizer
-	**lowercase** token filter
-	**asciifolding** token filter

```shell
"text": "Elasticsearch é um motor de buscas distribuído."
```

<details><summary> 정답 </summary>

```shell
GET _analyze
{
  "tokenizer": "standard",
  "filter": ["lowercase", "asciifolding"],
  "text": "Elasticsearch é um motor de buscas distribuído."
}
```

-	**é/í** 가 **e/i** 로 변경

</details>

<br>

###### **standard** 와 **english** analyzer, 즉 built-in analyzer는 완벽하지 않다. 다음 문젱을 분석해 보자

```shell
"text": "C++ can help it and your IT systems."
```

<details><summary> 정답 </summary>

```shell
GET _analyze
{
  "analyzer": "english",
  "text": "C++ can help it and your IT systems."
}

# response
{
  "tokens": [
    {
      "token": "c",
      "start_offset": 0,
      "end_offset": 1,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "can",
      "start_offset": 4,
      "end_offset": 7,
      "type": "<ALPHANUM>",
      "position": 1
    },
    {
      "token": "help",
      "start_offset": 8,
      "end_offset": 12,
      "type": "<ALPHANUM>",
      "position": 2
    },
    {
      "token": "your",
      "start_offset": 20,
      "end_offset": 24,
      "type": "<ALPHANUM>",
      "position": 5
    },
    {
      "token": "system",
      "start_offset": 28,
      "end_offset": 35,
      "type": "<ALPHANUM>",
      "position": 7
    }
  ]
}
```

-	"C++"의 기호가 사라지고, "IT"에 변화가 생긴다.

</details>

<br>

###### 다음의 조건에 성립하는, **my_analyzer** 란 이름의 analyzer를 포함하는 **analysis_test** index를 만들어 보자

-	"c++"(소/대문자 모두)를 "cpp"로 변환
-	"IT"를 (not "it") "\_IT\_"로 변환
-	모든 text를 소문자로 변환
-	default stop word 제거
-	다음 term 제거: can, we, our, you, your, all

<details><summary> 정답 </summary>

```shell
# before
GET _analyze
{
  "analyzer": "english",
  "text": "C++ can help it and your IT systems."
}

# create index
PUT analysis_test
{
  "settings":{
    "analysis":{
      "char_filter":{
        "cpp_it":{
          "type": "mapping",
          "mappings": ["c++ => cpp", "C++ => cpp", "IT => _IT_"]
        }
      },
      "filter":{
        "my_stop":{
          "type":"stop",
          "stopwords":["can","we","our","you","your","all"]
        }
      },
      "analyzer":{
        "my_analyzer":{
          "tokenizer":"standard",
          "char_filter":["cpp_it"],
          "filter":["lowercase","stop","my_stop"]
        }
      }
    }
  }
}

# after
GET analysis_test/_analyze
{
  "analyzer":"my_analyzer",
  "text": "C++ can help it and your IT systems."
}
```

</details>

<br>

###### 기존의 **blogs** index의 **content** field에서 **c++** 과 **IT** 를 쿼리해 보자

```shell
GET blogs/_search
{
  "query": {
    "match": {
      "content": "c++"
    }
  }
}

GET blogs/_search
{
  "query": {
    "match": {
      "title": "IT"
    }
  }
}
```

###### 위의 부족함을 보완하기 위해, 아래의 조건을 가지 적절한 analyzer를 만들어보자

-	위에서 구성한 **my_analyzer** 를 활용해서, **blogs_analyzed** index 생성하기
-	**my_analyzer** 이름의 analyzer를 multi-field(**content** 와 **title** field)에 추가하기 (multi-field type: text)

<details><summary> 정답 </summary>

```shell
PUT blogs_analyzed
{
  "settings": {
    "analysis": {
      # filter 추가 1
      "char_filter": {
        "cpp_it": {
          "type": "mapping",
          "mappings": ["c++ => cpp", "C++ => cpp", "IT => _IT_"]
        }
      },
      # filter 추가 2
      "filter": {
        "my_stop": {
          "type": "stop",
          "stopwords": ["can", "we", "our", "you", "your", "all"]
        }
      },
      # 상위 두개의 filter를 analyzer의 filter에 추가
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "standard",
          "char_filter": ["cpp_it"],
          "filter": ["lowercase", "stop", "my_stop"]
        }
      }
    }
  },
  "mappings": {
    "_doc":{
      "properties":{
        "title":{
          "type":"text",
          "fields":{
            "keyword":{
              "type":"keyword",
              "ignore_above":256
            },
            # title field에 analyzer 추가
            "my_analyzer":{
              "type":"text",
              "analyzer": "my_analyzer"
            }
          }
        },
        "publish_date":{
          "type":"date"
        },
        "author":{
          "type":"text",
          "fields":{
            "keyword":{
              "type":"keyword",
              "ignore_above":256
            }
          }
        },
        "category":{
          "type":"keyword"
        },
        "content":{
          "type":"text",
          "fields":{
            # content field에 analyzer 추가
            "my_analyzer":{
              "type":"text",
              "analyzer": "my_analyzer"
            }
          }
        },
        "url":{
          "type":"text",
          "fields":{
            "keyword":{
              "type":"keyword",
              "ignore_above":256
            }
          }
        },
        "locales":{
          "type":"keyword"
        }
      }
    }
  }
}
```

-	[참고: Mapping Char Filter](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-mapping-charfilter.html)

</details>

<br>

###### **blogs** index에서 **blogs_analyzed** index로 reindex하기

```shell
POST _reindex
{
  "source": {"index": "blogs"},
  "dest":   {"index": "blogs_analyzed"}
}
```

###### **.my_analzyer** field를 사용해서 검색하고 결과 확인하기 (1 hit)

<details><summary> 정답 </summary>

```shell
GET blogs_analyzed/_search
{
  "_source": "title",
  "query":{
    "match":{
      "title.my_analyzer": "IT"
    }
  }
}

GET blogs_analyzed/_search
{
  "_source": "content",
  "query": {
    "match": {
      "content.my_analyzer": "c++"
    }
  }
}
```

</details>

5.The Distributed Model
-----------------------

###### 클러스터의 상태를 체크해보기

-	클러스터의 이름은?
-	클러스터의 노드 수는?
-	현재 마스터 노드는?
-	클러스터에 있는 index 수는?
-	logs_server2 index의 샤드 수와 할당된 샤드의 위치(node)는?

<details><summary> 정답 </summary>

```shell
GET _cluster/state

GET _cat/nodes?v
GET _cat/indices?v
GET _cat/shards?v
GET _cat/shards/logs_server2?v
```

</details>

<br>

###### 4개의 primary shard와 2개의 replica를 가진 **test** index를 생성하자

<details><summary> 정답 </summary>

```shell
PUT test
{
  "settings": {
    "number_of_shards": 4,
    "number_of_replicas": 2
  }
}
```

</details>we

######

6.Troubleshooting Elasticsearch
-------------------------------

######

<details><summary> 정답 </summary>

</details>

7.Improving Search Results
--------------------------

######

<details><summary> 정답 </summary>

</details>

8.Aggregating Data
------------------

######

<details><summary> 정답 </summary>

</details>

9.Securing Elasticsearch
------------------------

######

<details><summary> 정답 </summary>

</details>

10.Best Practices
-----------------

######

<details><summary> 정답 </summary>

</details>

\.

.

.

.

.

ES2
===

1.Field Modeling
----------------

2.Fixing Data
-------------

3.Advanced Search & Aggregations
--------------------------------

4.Cluster Management
--------------------

5.Capacity Planning
-------------------

6.Elasticsearch Internals
-------------------------

7.Document Modeling
-------------------

8.Monitoring and Alerting
-------------------------

9.From Dev to Production
------------------------

.

.

.

.

.

.

.

.

.

.

.

.

.

.

.

---

---

Exam Objectives
---------------

To be fully prepared for the Elastic Certified Engineer exam, candidates should be able to complete all of the following exam objectives with only the assistance of the [Elastic documentation](https://www.elastic.co/guide/index.html):

<br>

#### Installation and Configuration

-	Deploy and start an Elasticsearch cluster that satisfies a given set of requirements
-	Configure the nodes of a cluster to satisfy a given set of requirements
-	Secure a cluster using Elasticsearch Security ([doc](https://www.elastic.co/guide/en/elasticsearch/plugins/current/security.html)\)
-	Define role-based access control using Elasticsearch Security ([doc](https://www.elastic.co/guide/en/kibana/current/development-security-rbac.html)\)

#### Indexing Data

-	Define an index that satisfies a given set of requirements

-	Perform index, create, read, update, and delete operations on the documents of an index

-	Define and use index aliases

-	Define and use an index template for a given pattern that satisfies a given set of requirements

-	Define and use a dynamic template that satisfies a given set of requirements

-	Use the Reindex API and Update By Query API to reindex and/or update documents

-	Define and use an ingest pipeline that satisfies a given set of requirements, including the use of Painless to modify documents

#### Queries

-	Write and execute a search query for terms and/or phrases in one or more fields of an index

-	Write and execute a search query that is a Boolean combination of multiple queries and filters

-	Highlight the search terms in the response of a query

-	Sort the results of a query by a given set of requirements

-	Implement pagination of the results of a search query

-	Use the scroll API to retrieve large numbers of results

-	Apply fuzzy matching to a query

-	Define and use a search template

-	Write and execute a query that searches across multiple clusters

#### Aggregations

-	Write and execute metric and bucket aggregations

-	Write and execute aggregations that contain sub-aggregations

-	Write and execute pipeline aggregations

	#### Mappings and Text Analysis

-	Define a mapping that satisfies a given set of requirements

-	Define and use a custom analyzer that satisfies a given set of requirements

-	Define and use multi-fields with different data types and/or analyzers

-	Configure an index so that it properly maintains the relationships of nested arrays of objects

-	Configure an index that implements a parent/child relationship

#### Cluster Administration

-	Allocate the shards of an index to specific nodes based on a given set of requirements

-	Configure shard allocation awareness and forced awareness for an index

-	Diagnose shard issues and repair a cluster’s health

-	Backup and restore a cluster and/or specific indices

-	Configure a cluster for use with a hot/warm architecture

-	Configure a cluster for cross cluster search
