# データの説明
現状アクセス可能なデータ。
形式は渡し方によってある程度調整できると思います(Pydanticで定義しているものを修正すればできそう）
## 入力データの処理
logファイルにあるように**createHPODictNode**を経て、存在する表現型についてHPOのIDをキーとし、表現型の名前をバリューとするような辞書を作成する。
**createAbsentHPODictNode**を経て、この症例ではみられなかった表現型についてHPOのIDをキーとし、表現型の名前をバリューとするような辞書を作成する。

## ノードの出力
正規化されますが、同じデータで管理しているので単純に正規化を行う処理のノードを経るとデータが破壊的に変更されるはずです  
### GestaltMatcherNode
顔写真を用いて近い症例を持ってくるツール。(写真がある場合)近い順に症例を持ってくる
それぞれの症例には  
- 患者id
- 疾患名(原因遺伝子の場合もあり）
- 対応omim id(ない場合もあり)
- 類似スコア
```
{
      "subject_id": "3554",
      "syndrome_name": "CORNELIA DE LANGE SYNDROME 1; CDLS1",
      "omim_id": 122470,
      "image_id": "4891",
      "score": 0.5107692307692308
    }
```

### PCFNode  
pubCaseFinderの結果が返ってくる。
表現型の情報をもとに、近い疾患を持ってくる。
上から近い順に疾患が含まれている、それぞれの疾患は
- 英語のomimに対応した疾患名
- 説明
- 類似度
- omim id
```
{
      "omim_disease_name_en": "Cornelia de Lange syndrome 5",
      "description": "",
      "score": 0.9500001382287309,
      "omim_id": "OMIM:300882",
      "disease_name": "CORNELIA DE LANGE SYNDROME 5; CDLS5"
    }
```
### DiseaseSearchWithHPONode
表現型情報をもとに、類似度検索をかける
- 疾患情報
    - OMIM ID
    -  疾患名
    -  シノニム
    -  定義
    -  表現型のリスト
- 類似度
```
{
      "disease_info": {
        "OMIM_id": "300590",
        "disease_name": "CORNELIA DE LANGE SYNDROME 2",
        "synonym": "CDLS2, Cdls, X-linked, Cornelia DE Lange syndrome 2, Cornelia De Lange syndrome type 2, Cornelia De Lange syndrome, X-linked, Cornelia de Lange syndrome 2, Cornelia de Lange syndrome 2, X-linked dominant, Cornelia de Lange syndrome caused by mutation in SMC1A, SMC1A Cornelia de Lange syndrome, X-linked Cornelia De Lange syndrome",
        "definition": "An X-linked inherited form of Cornelia De Lange syndrome caused by mutations in the SMC1A gene mapped to chromosome Xp11.22. Patients have a milder form of the syndrome compared to patients with the NIPBL gene mutation.",
        "phenotype": [
          "High palate",
          "Thin upper lip vermilion",
          "Brachycephaly",
          "Microcephaly",
          "Low anterior hairline",
          "Smooth philtrum",
          "Narrow forehead",
          "Micrognathia",
          "Prominent nasal bridge",
          "Anteverted nares",
          "Short neck",
          "Downslanted palpebral fissures",
          "Ptosis",
          "Long eyelashes",
          "Thick eyebrow",
          "Synophrys",
          "Cutis marmorata",
          "Hirsutism",
          "Brachydactyly",
          "Intellectual disability",
          "Seizure",
          "Global developmental delay",
          "Intrauterine growth retardation",
          "Hypertrophic cardiomyopathy",
          "Short foot",
          "Gastroesophageal reflux",
          "Ventriculomegaly",
          "Poor speech",
          "Highly arched eyebrow",
          "Downturned corners of mouth",
          "Limited elbow movement",
          "Short stature",
          "Postnatal growth retardation",
          "Proximal placement of thumb",
          "Clinodactyly",
          "Cognitive impairment",
          "Small hand"
        ]
      },
      "similarity_score": 0.8693238496780396
    }```
### createZeroShotNode
ゼロショットによってLLMにそれっぽい診断を出させる
プロンプトはstateで管理されていない
- ans
    - 疾患名
    - 順位
    - omim id

### HPOwebSearchNode
- タイトル
- URL
- 要約

### createDiagnosisNode
プロンプトはstateで管理されていない
- 診断結果
    - 参考情報
    - 回答
        - 疾患名
        - omim id
        - 説明
        - 順位

### diseaseSearchNode
- memory
    - タイトル
    - URL
    - 要約
    - 関連する疾患

### reflectionNode
- ans
    - 疾患名
    - 正しいかどうか
    - 患者の要約
    - 根拠


### finalDiagnosisNode
- ans
    - 疾患名
    - omim id
    - 説明
    - 順位
- reference



