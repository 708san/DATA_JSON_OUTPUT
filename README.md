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
    }
```
### createZeroShotNode
ゼロショットによってLLMにそれっぽい診断を出させる
プロンプトはstateで管理されていない
- ans
    - 疾患名
    - 順位
    - omim id
```
{
        "disease_name": "CORNELIA DE LANGE SYNDROME 1; CDLS1",
        "rank": 1,
        "OMIM_id": "OMIM:122470"
      }
```
### HPOwebSearchNode
- タイトル
- URL
- 要約

```
{
      "title": "Cornelia de Lange syndrome 1 (Concept Id: C4551851) - MedGen -",
      "url": "https://www.ncbi.nlm.nih.gov/medgen/1645760",
      "snippet": "Disease: Not specified in provided text (insufficient context to identify a single syndrome)\n\nGenetics: Not specified in provided text\n\nKey Phenotypes:\n- Synophrys\n- Highly arched and/or thick eyebrows\n- Long eyelashes\n- Short nasal bridge with anteverted nares\n- Small, widely spaced teeth\n- Microcephaly\n\nDifferentiating Features:\n- Combination of synophrys + long eyelashes + short nasal bridge with anteverted nares suggests a “Cornelia de Lange–like” facial gestalt, but the excerpt alone is not sufficient to distinguish among CdLS and related mimics (e.g., CdLS spectrum genes, KBG, Wiedemann-Steiner, Coffin-Siris).\n\nHallmark(s):\n- Not specified as pathognomonic in provided text\n\nKey Negative Finding(s):\n- Not stated in provided text\n\nUnique Constellation:\n- Synophrys + thick/arched eyebrows + long eyelashes + short nasal bridge with anteverted nares + small widely spaced teeth + microcephaly (a distinctive craniofacial pattern; full diagnosis requires additional systemic/neurodevelopmental features and genetics not provided)."
    }
```

### createDiagnosisNode
プロンプトはstateで管理されていない
- 診断結果
    - 参考情報
    - 回答
        - 疾患名
        - omim id
        - 説明
        - 順位
```
 {
        "disease_name": "CORNELIA DE LANGE SYNDROME 1; CDLS1",
        "OMIM_id": "OMIM:122470",
        "description": "Top multi-tool match: supported by PubCaseFinder (score 0.945), ZeroShot (rank 1), and GestaltMatcher (similarity 0.511); fits IUGR/postnatal growth retardation, microcephaly, synophrys/long eyelashes, hirsutism, thin upper lip, long philtrum, downturned mouth, limb/hand anomalies and limited elbow extension.",
        "rank": 1
      },
      {
        "disease_name": "CORNELIA DE LANGE SYNDROME 2; CDLS2",
        "OMIM_id": "OMIM:300590",
        "description": "Strong multi-tool match: supported by PubCaseFinder (score 0.930) and Phenotype Similarity Search (score 0.869); matches high palate, thin upper lip, microcephaly, low anterior hairline, synophrys/long eyelashes, hirsutism, global developmental delay and limited elbow movement/small hands.",
        "rank": 2
      }
```

### diseaseSearchNode
- memory
    - タイトル
    - URL
    - 要約
    - 関連する疾患
```
{
      "title": "AHDC1",
      "url": "https://en.wikipedia.org/wiki/AHDC1",
      "content": "[Source: Wikipedia] Disease: Xia-Gibbs Syndrome  \n\nGenetics: AHDC1; Typically de novo, autosomal dominant pattern  \n\nKey Phenotypes:\n\n- Global developmental delay  \n- Intellectual disability  \n- Generalized (global) hypotonia  \n- Obstructive sleep apnea  \n- Seizures  \n\nDifferentiating Features:\n\nHallmark(s): \n- Combination of global developmental delay/intellectual disability with generalized hypotonia and obstructive sleep apnea in the same patient  \n- Pathogenic/de novo variant in AHDC1  \n\nKey Negative Finding(s): \n- Not specified  \n\nUnique Constellation: \n- Global developmental delay/intellectual disability + marked hypotonia + obstructive sleep apnea + seizures, in the context of an AHDC1 mutation, strongly points to Xia-Gibbs Syndrome compared with other causes of developmental delay and hypotonia that lack prominent obstructive sleep apnea.",
      "disease_name": "XIA-GIBBS SYNDROME; XIGIS"
    },
    {
      "title": "CREB-binding protein",
      "url": "https://en.wikipedia.org/wiki/CREB-binding_protein",
      "content": "[Source: Wikipedia] Disease: Not specified (text describes the CREBBP/CBP protein, not a specific clinical syndrome)\n\nGenetics: Not specified\n\nKey Phenotypes:\n\n- Not specified in the provided text (only molecular and functional roles of CBP are described)\n\nDifferentiating Features:\n\nHallmark(s):  \n- None provided (no clinical syndrome or pathognomonic clinical signs described)\n\nKey Negative Finding(s):  \n- Not specified\n\nUnique Constellation:  \n- Not specified (no clinical symptom constellation is described, only involvement of CBP in various diseases at a molecular level)",
      "disease_name": "RUBINSTEIN-TAYBI SYNDROME 1; RSTS1"
    }
```
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



