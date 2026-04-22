# ツール概要
<img width="1000" height="782" alt="image" src="https://github.com/user-attachments/assets/7007f1d8-b3fb-4178-a74d-2755d357180b" />


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
"ans": [
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
      }],
"reference": "1. PubCaseFinder report (phenotype-based ranking with scores).\n2. Zero-Shot Diagnosis report (generative AI ranked list).\n3. GestaltMatcher report (facial analysis similarity scores).\n4. Phenotype Similarity Search report (vector search with similarity scores and OMIM IDs)."
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

```
{
        "disease_name": "CORNELIA DE LANGE SYNDROME 5; CDLS5",
        "Correctness": true,
        "PatientSummary": "Male with prenatal and postnatal growth failure (IUGR and postnatal growth retardation) and microcephaly, with global developmental delay and delayed CNS myelination. He has a CdLS-like craniofacial gestalt (synophrys, long eyelashes, low anterior hairline, thin upper lip vermilion, long philtrum, downturned mouth corners, low-set ears, high palate) and hirsutism. Limb/hand findings include limited elbow extension, small hands, 5th-finger clinodactyly, and toe syndactyly.",
        "DiagnosisAnalysis": "Evidence FOR CDLS5 (HDAC8-related Cornelia de Lange syndrome): The patient’s overall pattern is highly congruent with the core CdLS spectrum—prenatal-onset growth restriction/failure to thrive, neurodevelopmental delay, and a recognizable “CdLS facial gestalt.” The literature summary provided for CdLS5 emphasizes exactly this triad (facial gestalt + growth failure + neurodevelopmental delay) as central to the diagnosis, with skeletal anomalies also being part of the spectrum [1]. The patient’s specific facial features (synophrys, long eyelashes, low anterior hairline, long philtrum, thin upper lip, downturned mouth corners) collectively support a CdLS gestalt rather than representing only non-specific dysmorphism, and his limb findings (limited elbow extension, small hand, clinodactyly, toe syndactyly) align with the CdLS “skeletal anomalies” category [1].\n\nEvidence AGAINST / requiring caution: The cited CdLS5 case report highlights prominent generalized dystonia/spastic quadriparesis and also notes normal brain MRI in that individual [1]. This patient instead has delayed CNS myelination and no dystonia/spastic quadriparesis reported; however, dystonia is presented in the paper as a “novel phenotype” rather than a required hallmark for CdLS5, so its absence does not strongly argue against HDAC8-related CdLS [1]. Conversely, delayed myelination is not discussed in the provided excerpt and could reflect broader neurodevelopmental heterogeneity or a separate/co-occurring process; it is a point to monitor but not a decisive contradiction based on the limited literature provided. Additionally, several commonly discussed CdLS features (e.g., major limb reduction defects, genital anomalies, diaphragmatic hernia, seizures) are absent here, but these are not mandatory for CdLS and are variably present across the spectrum; the patient already demonstrates multiple higher-specificity gestalt features plus growth restriction and limb involvement, which together outweigh the missing items.\n\nSynthesis: Given the strong convergence of (1) characteristic CdLS facial gestalt, (2) prenatal/postnatal growth failure, (3) neurodevelopmental delay, and (4) compatible limb/hand anomalies—all explicitly within the CdLS5 phenotype framework described—CDLS5 remains a clinically credible diagnosis and should not be deprioritized. The main caution is that the provided literature excerpt centers on dystonia as a distinguishing feature for that specific report; absence of dystonia does not negate CdLS5, but confirmatory molecular testing for HDAC8 (and broader CdLS genes) would be essential, and the delayed myelination warrants consideration of additional/alternative etiologies if genetic results are negative or if the phenotype evolves atypically.",
        "references": [
          "[PubMed] (https://pubmed.ncbi.nlm.nih.gov/38910710/): “Key Phenotypes: - Distinctive CdLS facial gestalt (dysmorphic facies) - Neurodevelopmental delay - Failure to thrive (often with prenatal growth restriction) - Skeletal anomalies (CdLS spectrum) - Generalized dystonia with marked axial and appendicular involvement (reported phenotype) - Spastic quadriparesis.”"
        ]
      },
      {
        "disease_name": "Craniofrontonasal syndrome (CFNS) / craniofrontonasal dysplasia (EFNB1-related)",
        "Correctness": false,
        "PatientSummary": "Male with intrauterine and postnatal growth retardation, microcephaly, global developmental delay with delayed CNS myelination. Dysmorphic/ectodermal features include synophrys, hirsutism, long eyelashes, low anterior hairline, long philtrum, thin upper lip, downturned mouth corners, high-arched palate, and low-set ears. Limb findings include toe syndactyly, 5th-finger clinodactyly, small hands, and limited elbow extension.",
        "DiagnosisAnalysis": "Evidence FOR CFNS: The patient has several features that can occur in CFNS, including high-arched palate, low anterior hairline, low-set ears, and limb anomalies such as 5th-finger clinodactyly and cutaneous syndactyly (toe syndactyly fits the general pattern of syndactyly described in CFNS). Neurodevelopmental delay and growth impairment are also reported in CFNS and overlap with this case.\n\nEvidence AGAINST / caution: The core defining CFNS craniofacial constellation is not documented here: there is no mention of coronal craniosynostosis, hypertelorism, bifid nasal tip, or facial asymmetry—features emphasized as hallmark/differentiating for CFNS. Likewise, key ectodermal hallmarks (dry/frizzy/curled hair and longitudinal nail ridging/splitting) are not reported; instead, the case lists hirsutism/synophrys/long eyelashes, which are not highlighted as characteristic CFNS signatures in the provided sources. Sex also materially weakens the diagnosis: EFNB1-related CFNS is X-linked with paradoxically greater severity in heterozygous females, while males are often mildly affected or even clinically unaffected; this patient is a male with significant neurodevelopmental issues and multiple anomalies, which is possible but would be less typical and would ideally be accompanied by the classic craniofrontonasal malformations if CFNS were the unifying diagnosis. Finally, the presence of microcephaly and delayed CNS myelination are not presented as typical CFNS hallmarks in the supplied literature and may suggest an alternative neurodevelopmental syndrome.\n\nSynthesis: Although there is partial overlap (syndactyly/clinodactyly, palate, hairline/ears, DD/growth impairment), the absence of CFNS-defining craniofrontonasal malformations (craniosynostosis, hypertelorism, bifid nasal tip, facial asymmetry) plus absence of characteristic hair/nail dysplasia makes the match weak. Given that the supporting findings are largely nonspecific or broadly shared across many syndromic conditions, CFNS should be deprioritized unless additional targeted clinical review (head shape/history of craniosynostosis, interorbital distance, nasal tip morphology, facial asymmetry, hair texture, nail ridging/splitting) reveals the hallmark CFNS pattern or EFNB1 testing supports it.",
        "references": [
          "[Wikipedia] (https://en.wikipedia.org/wiki/Craniofrontonasal_dysplasia): \"Coronal craniosynostosis (unilateral or bilateral)\"; \"Orbital hypertelorism\"; \"Bifid nasal tip (often with broad nasal base)\"; \"Facial asymmetry\".",
          "[Wikipedia] (https://en.wikipedia.org/wiki/Craniofrontonasal_dysplasia): \"Dry, frizzy, curled hair\"; \"Longitudinal nail ridging and/or splitting\".",
          "[Wikipedia] (https://en.wikipedia.org/wiki/Craniofrontonasal_dysplasia): \"Less frequent: low anterior hairline, low-set ears, dental crowding, maxillary hypoplasia, high-arched palate\"; \"Limb anomalies: clinodactyly (often 5th finger), cutaneous syndactyly\".",
          "[PubMed] (https://pubmed.ncbi.nlm.nih.gov/41096135/): \"X-linked disorder with paradoxical severity: heterozygous females more severely affected than hemizygous males (often mild or asymptomatic)\"; \"Developmental delay / neurodevelopmental delay\"; \"Growth impairment\"."
        ]
      }
```


### finalDiagnosisNode
- ans
    - 疾患名
    - omim id
    - 説明
    - 順位
- reference

```
{
    "ans": [
      {
        "disease_name": "CORNELIA DE LANGE SYNDROME 1; CDLS1",
        "OMIM_id": "OMIM:122470",
        "description": "The patient has a classic Cornelia de Lange (CdLS) gestalt: intrauterine and postnatal growth retardation, microcephaly, global developmental delay, synophrys, long eyelashes, hirsutism, long philtrum, thin upper lip vermilion, downturned mouth corners, high palate, and limb/digital findings (small hands, 5th-finger clinodactyly, toe syndactyly, limited elbow extension), all of which are described as characteristic for CdLS [1,2]. Pathophysiologically, CdLS is a cohesinopathy (often NIPBL in CDLS1) leading to dysregulated gene expression during development, explaining the combination of craniofacial patterning, growth failure, neurodevelopmental impairment, and limb anomalies [1,2]. Multi-tool prioritization also strongly supports CdLS1 as the top match (PubCaseFinder/ZeroShot/GestaltMatcher) [3].",
        "rank": 1
      }],
 "reference": "[1] Source type: Literature/clinical overview (Wikipedia). Summary: Describes Cornelia de Lange syndrome with characteristic facial gestalt (e.g., long/thick eyebrows/synophrys, long philtrum, thin upper lip, downturned mouth corners), prenatal/postnatal growth restriction, microcephaly, hirsutism, and limb anomalies including small hands/feet, clinodactyly, and 2–3 toe syndactyly. Relevance: Directly matches the patient’s growth restriction, microcephaly, CdLS-like face, hirsutism, and limb/digital findings. URL: https://en.wikipedia.org/wiki/Cornelia_de_Lange_syndrome\n[2] Source type: Literature (PubMed). Summary: Review/summary emphasizing CdLS key phenotypes including distinctive craniofacial dysmorphism, skeletal anomalies, neurodevelopmental impairment, and cutaneous findings such as hirsutism and synophrys. Relevance: Supports the specificity of the patient’s synophrys/hirsutism plus facial and limb phenotype as CdLS-spectrum features. URL: https://pubmed.ncbi.nlm.nih.gov/41499064/\n[3] Source type: Diagnosis assistant tool outputs (PubCaseFinder, ZeroShot, GestaltMatcher, Phenotype Similarity Search). Summary: Multi-tool ranking strongly prioritizes CdLS1/2/3/5 among top matches and includes WDSTS and CFNS among additional candidates (with similarity scores/ranks). Relevance: Independent phenotype- and facial-gestalt-based prioritization supports the ranked differential used here. URL: Not provided (tool reports referenced in prompt)\n[4] Source type: Literature (PubMed case report). Summary: Reports HDAC8-related Cornelia de Lange syndrome (CDLS5) highlighting CdLS facial gestalt, neurodevelopmental delay, failure to thrive with prenatal growth restriction, and skeletal anomalies; also notes dystonia/spastic quadriparesis in the reported individual. Relevance: Provides cited CDLS5 phenotype anchors that match this patient’s CdLS gestalt, growth failure, developmental delay, and skeletal findings. URL: https://pubmed.ncbi.nlm.nih.gov/38910710/\n[5] Source type: Literature/clinical overview (Wikipedia via provided link). Summary: Describes Wiedemann–Steiner syndrome with developmental delay, pre/postnatal growth deficiency/short stature, characteristic facial features (including long philtrum, low-set ears, high-arched palate), and hypertrichosis cubiti/hypertrichosis. Relevance: Matches the patient’s developmental delay, growth restriction, and hypertrichosis/hirsutism with overlapping facial findings. URL: https://www.google.com/search?q=https://en.wikipedia.org/wiki/Wiedemann%E2%80%93Steiner_syndrome\n[6] Source type: Literature (PubMed). Summary: ‘Revisiting Wiedemann-Steiner Syndrome: Novel Variants and Broadened Clinical Spectrum’ summarizing expanded WDSTS phenotypes including developmental delay and additional systemic findings such as skeletal and genitourinary involvement. Relevance: Supports that musculoskeletal anomalies can be part of WDSTS, consistent with the patient’s limb findings. URL: https://www.google.com/search?q=https://pubmed.ncbi.nlm.nih.gov/41320952/\n[7] Source type: Literature (PubMed). Summary: WDSTS cohort/summary noting key phenotypes including developmental delay, hypertrichosis, failure to thrive/short stature, facial dysmorphism, and musculoskeletal anomalies. Relevance: Directly overlaps with the patient’s growth failure, hirsutism, facial gestalt, and limb anomalies. URL: https://www.google.com/search?q=https://pubmed.ncbi.nlm.nih.gov/41053724/\n[8] Source type: Literature/clinical overview (Wikipedia). Summary: Filippi syndrome description including syndactyly type I (commonly toes), microcephaly, intellectual disability, growth retardation, craniofacial abnormalities, and other digital anomalies such as 5th-finger clinodactyly. Relevance: Matches the patient’s microcephaly, toe syndactyly, clinodactyly, developmental delay, and growth restriction. URL: https://en.wikipedia.org/wiki/Filippi_syndrome\n[9] Source type: Literature (PubMed). Summary: Filippi syndrome paper emphasizing hallmark triad of facial dysmorphism + syndactyly + microcephaly. Relevance: Directly supports Filippi syndrome as a strong candidate given the patient’s matching triad elements and neurodevelopmental impairment. URL: https://pubmed.ncbi.nlm.nih.gov/41370039/\n[10] Source type: Literature/clinical overview (Wikipedia). Summary: Craniofrontonasal dysplasia description listing features including coronal craniosynostosis/hypertelorism/bifid nasal tip as classic signs and also mentions less frequent findings such as low anterior hairline, low-set ears, high-arched palate, and limb anomalies like 5th-finger clinodactyly and syndactyly. Relevance: Supports partial phenotypic overlap (palate/hairline/ears and digital anomalies) for lower-ranked consideration. URL: https://en.wikipedia.org/wiki/Craniofrontonasal_dysplasia\n[11] Source type: Literature (PubMed). Summary: Review noting CFNS is X-linked with paradoxical severity (heterozygous females often more affected than males), and includes discussion of developmental delay and growth impairment as possible features. Relevance: Provides inheritance/sex-effect context for considering CFNS in a male and supports that DD/growth issues can occur. URL: https://pubmed.ncbi.nlm.nih.gov/41096135/"
  }
```
### formatted reflection
1. CHROMOSOME 1p36 DELETION SYNDROME, DISTAL

Status: Correctness: true

Patient Summary

Male patient with global developmental delay, seizures, hypotonia with later emerging spasticity/hyperreflexia, and feeding difficulty (poor suck), plus dyspnea and diaphragmatic/abdominal wall issues (hiatus hernia, diastasis recti). Neuroimaging shows periventricular leukomalacia, lateral ventricle dilatation, and an arachnoid cyst. He has skeletal and musculoskeletal anomalies (thoracic scoliosis, hamstring contractures, metatarsus adductus, pectus carinatum) and craniofacial dysmorphism including hypertelorism, bulbous nose, and prominent nasal bridge; cutis marmorata and abnormal visual evoked potentials are also present.

Diagnosis Analysis

Evidence FOR the diagnosis:

The core triad of global developmental delay/intellectual disability, hypotonia, and seizures is highly characteristic of 1p36 deletion syndrome. The literature describes moderate to severe intellectual disability with delayed motor development and hypotonia [3, 25, 26], and seizures/epilepsy as a frequent feature [3, 25, 26]. This patient has global developmental delay, documented hypotonia, and seizures, directly aligning with the syndrome’s central neurodevelopmental phenotype.

Structural brain abnormalities are a major hallmark of 1p36 deletion syndrome. Structural CNS anomalies, particularly ventriculomegaly, are repeatedly emphasized: ventriculomegaly is listed among typical brain findings [3] and as a prenatal hallmark [24]. This patient has lateral ventricle dilatation and periventricular leukomalacia, plus an arachnoid cyst. While periventricular leukomalacia is not a classic feature, the presence of ventriculomegaly/ventricular dilatation matches the reported pattern of structural brain anomalies [3, 24]. The arachnoid cyst is an additional brain anomaly, which is directionally consistent with a syndromic brain-malformation phenotype even if not specifically listed in the brief summaries.

Feeding and gastrointestinal issues are common in 1p36 deletion syndrome; feeding difficulties such as dysphagia and esophageal reflux are noted [3]. This patient’s poor suck and hiatus hernia, along with diastasis recti, reflect early feeding/upper GI dysfunction and congenital abdominal wall anomalies that fit a broad syndromic context. Dyspnea could relate to neuromuscular weakness or structural issues, which is consistent with a multisystem developmental syndrome, though not specific.

Craniofacial dysmorphism is a well-described component. The syndrome is associated with craniofacial anomalies including nasal bone abnormalities [24] and characteristic facial features (midface hypoplasia, straight eyebrows, deep-set eyes) [25, 26]. This patient has hypertelorism, bulbous nose, and a prominent nasal bridge, clearly indicating facial dysmorphism. While not a textbook description, the presence of abnormal nasal structure and hypertelorism supports a syndromic craniofacial pattern, which is compatible with the variability seen in 1p36 deletion syndrome [24–26]. Abnormal visual evoked potentials also align directionally with the vision abnormalities frequently reported (hypermetropia, myopia, strabismus) [3].

Musculoskeletal and thoracic anomalies are common in chromosomal syndromes and do not conflict with 1p36. Scoliosis and other skeletal anomalies are frequently seen in children with significant hypotonia and later spasticity; while not specifically highlighted in the brief literature excerpts, thoracic scoliosis, pectus carinatum, contractures, and metatarsus adductus are plausible downstream consequences of neurodevelopmental impairment and altered tone. Cutis marmorata is nonspecific but may be seen in neurologically impaired or hypotonic infants.

Overall, the presence of: (1) global developmental delay; (2) hypotonia; (3) seizures; (4) structural brain anomalies including ventriculomegaly; (5) feeding difficulties; and (6) craniofacial dysmorphism and visual pathway abnormality collectively reproduces the “unique constellation” described for 1p36 deletion syndrome [3, 24–26].

Evidence AGAINST or requiring caution:

Several hallmark or common features of 1p36 deletion syndrome are not documented in this case summary. Severe speech impairment is noted as a hallmark [3], but there is no specific information on speech; this is essentially missing rather than negative. Behavioral problems (self-injurious behavior, stereotypies) [3, 25] and congenital heart defects (ventricular septal defect and other cardiac anomalies) [24–26] are not mentioned. However, absence of documentation is not equivalent to definite absence, and the provided phenotype list is clearly incomplete regarding behavioral and cardiac evaluation.

Some features here are not classic in 1p36 deletion summaries: periventricular leukomalacia suggests a hypoxic-ischemic or premature-infant pattern; hamstring contractures and spasticity imply an evolving upper motor neuron phenotype, potentially cerebral palsy. These could reflect perinatal brain injury rather than primary chromosomal syndrome. However, ventriculomegaly is a core 1p36 feature [3, 24], and a child with 1p36 deletion could be more vulnerable to perinatal complications, so these findings do not actively contradict the diagnosis; they more likely represent an overlapping or compounding etiology.

No features are clearly discordant with 1p36 deletion syndrome based on the provided literature. There is no mention of hyperphagia, lissencephaly, or polydactyly, whose presence might point towards alternative chromosomal conditions [24–26]. The combination of early hypotonia followed by spasticity, scoliosis, and orthopedic deformities is compatible with a child who had significant early neurologic insult, which fits with structural brain anomalies and seizures, both part of 1p36.

Synthesis and weighting:

The decisive elements are the strong overlap with the core neurological and structural features of 1p36 deletion syndrome: global developmental delay/intellectual disability, hypotonia, seizures, and structural brain anomalies including ventriculomegaly [3, 24–26]. Feeding difficulty and GI disturbance (poor suck, hiatus hernia) are directionally consistent with reported dysphagia/reflux [3]. Craniofacial anomalies involving the nasal region and eye spacing are compatible with a variable facial phenotype [24–26].

The missing data on speech, behavior, and cardiac status limits full confirmation, but the absence of documentation does not significantly argue against the diagnosis. Potential perinatal brain injury features (periventricular leukomalacia) complicate the picture but do not provide a strong alternative unifying syndromic explanation for the craniofacial dysmorphism, seizures, ventriculomegaly, and global developmental delay. There are no strong contradictory findings.

Given the high concordance with the syndrome’s defining constellation and the lack of major negative evidence, 1p36 deletion syndrome remains a clinically plausible and reasonably well-supported diagnosis that should not be deprioritized. Genetic testing (chromosomal microarray or targeted analysis for 1p36 deletion) would be required for confirmation, but based on phenotype alone, the diagnosis withstands rigorous scrutiny.

2. HYPOTONIA, INFANTILE, WITH PSYCHOMOTOR RETARDATION AND CHARACTERISTIC FACIES 3; IHPRF3

Status: Correctness: true

Patient Summary

Male patient with severe congenital/infantile hypotonia, global developmental delay, seizures, abnormal brain imaging (periventricular leukomalacia, lateral ventricle dilatation, arachnoid cyst), and significant feeding problems including poor suck and hiatus hernia with associated dyspnea. He has pyramidal signs (spasticity, hyperreflexia), contractures (hamstring contractures), thoracic scoliosis, and foot deformity (metatarsus adductus), along with chest wall anomaly (pectus carinatum) and cutis marmorata. Facial features include hypertelorism, epicanthus, a bulbous nose with prominent nasal bridge, fitting a dysmorphic craniofacial profile.

Diagnosis Analysis

Evidence FOR IHPRF3 / TBCK‑related neurodevelopmental disorder:

Core neurodevelopmental picture: The literature defines IHPRF3 / TBCK‑related neurodevelopmental disorder as a severe infantile hypotonia syndrome with psychomotor retardation / global developmental delay and intellectual disability as hallmarks [32][33]. The patient has documented hypotonia and global developmental delay, which are central to the diagnosis. This aligns well with the described hallmark of “severe infantile hypotonia combined with psychomotor retardation and intellectual disability” [32] and “profound hypotonia combined with global developmental delay” [33].

Feeding and respiratory difficulties: TBCK‑related disorder is characterized by feeding difficulties and dysphagia, often requiring gastrostomy, and progressive respiratory issues [31]. The patient’s poor suck, hiatus hernia (which may exacerbate reflux/feeding problems), and dyspnea support significant bulbar and respiratory involvement, consistent with the feeding difficulties and respiratory insufficiency reported in TBCK‑related NDD [31].

Seizures and neurologic complications: Seizures are part of the broader TBCK‑related neurodevelopmental phenotype [31], even though in at least one IHPRF3 case they were absent [33]. This patient does have seizures, which therefore remain compatible with the spectrum covered in the broader TBCK‑related NDD description [31].

Motor system signs, contractures, and scoliosis: TBCK‑related NDD includes distal weakness progressing proximally with motor neuronopathy and development of contractures and neuromuscular scoliosis [31]. This patient has spasticity, hyperreflexia, hamstring contractures, thoracic scoliosis, and metatarsus adductus. While the classic description emphasizes distal wasting and motor neuronopathy, the presence of contractures and scoliosis is directly concordant with reported TBCK features [31], and upper motor neuron signs (spasticity, hyperreflexia) can be compatible with central motor pathway involvement in a severe neurodevelopmental disorder.

Brain imaging abnormalities: TBCK‑related NDD has progressive cortical atrophy and white matter lesions, and IHPRF3 case reports describe slightly widened ventricles and subarachnoid space [31][33]. The patient shows periventricular leukomalacia and lateral ventricle dilatation, which are white matter and ventricular abnormalities in the same general domain of CNS structural abnormalities as described in TBCK‑related disease [31][33], even if not identical. The presence of an arachnoid cyst is non‑specific but does not conflict with this pattern.

Craniofacial dysmorphism: IHPRF3 is associated with craniofacial dysmorphism, including mildly coarse facial features, hypertelorism, tented upper lip with exaggerated Cupid’s bow, macroglossia, and arched eyebrows [33]. The patient has hypertelorism, epicanthus, a bulbous nose, and a prominent nasal bridge, which indicate a dysmorphic facial phenotype. Although the exact facial gestalt differs from the specific features listed (no mention of tented upper lip, Cupid’s bow, macroglossia, or arched eyebrows), the presence of hypertelorism and distinctive nasal morphology is directionally consistent with a syndromic craniofacial pattern, and the literature emphasizes that the unique constellation is based on severe hypotonia plus facial dysmorphism, not on an invariant facial pattern [32][33].

Musculoskeletal and systemic features: TBCK‑related NDD reports contractures and neuromuscular scoliosis [31]; this patient has contractures, thoracic scoliosis, and a chest wall deformity (pectus carinatum). These findings broadly fit a severe neurodevelopmental / neuromuscular phenotype with skeletal secondary changes, consistent with TBCK‑related disease [31]. Cutis marmorata and diastasis recti are not classic for IHPRF3 but are nonspecific and compatible with chronic hypotonia and systemic illness.

Evidence AGAINST or requiring caution:

Missing hallmark progression details: A hallmark of the broader TBCK‑related NDD is progressive distal muscle wasting with motor neuronopathy, developmental regression, and neurologic decompensation during illness [31]. These features are not reported in the patient description (no explicit mention of distal wasting, regression, or decompensation). Their absence in the summary may simply reflect incomplete phenotyping rather than true absence, but they are important distinguishing traits, especially for older children or adolescents.

Facial gestalt differences: The IHPRF3 case report emphasizes a characteristic combination of facial traits—tented upper lip with exaggerated Cupid’s bow, macroglossia, arched eyebrows, and mildly coarse features [33]. The current patient’s facial description (hypertelorism, epicanthus, bulbous nose, prominent nasal bridge) overlaps only partially. The lack of classic IHPRF3 facial details weakens the specificity, but variable expressivity and limited facial description here reduce the weight of this discrepancy.

Limited information on vision and systemic complications: TBCK‑related NDD often includes optic atrophy and recurrent nephrolithiasis/UTIs, dyslipidemia, and left ventricular hypertrophy in some individuals [31]. The patient has abnormal visual evoked potentials, which suggests some visual pathway involvement and is not in conflict, but there is no mention of optic atrophy or systemic metabolic/cardiac/renal issues. However, these are not mandatory for diagnosis and may be age‑dependent or under‑evaluated.

Periventricular leukomalacia (PVL) vs classic TBCK MRI: PVL is often associated with perinatal injury; TBCK literature highlights progressive cortical atrophy and white matter lesions rather than classic PVL [31]. The presence of PVL might suggest a perinatal hypoxic‑ischemic component, but it does not exclude a genetic neurodevelopmental disorder; the lateral ventricle dilatation and white matter involvement are still compatible with a TBCK‑related process [31][33]. This nuance introduces some diagnostic uncertainty but is not strongly contradictory.

Synthesis and weighting:

The strongest alignment between this patient and IHPRF3/TBCK‑related NDD lies in the combination of: (1) severe infantile hypotonia and global developmental delay [32][33], (2) significant feeding problems and respiratory difficulties [31], (3) seizures within the broader TBCK‑related spectrum [31], (4) contractures and scoliosis [31], and (5) structural brain anomalies with ventricular enlargement/white matter involvement [31][33]. The presence of hypertelorism and other facial anomalies supports the concept of a syndromic dysmorphic disorder [33], even if the exact facial pattern differs. Missing details about progressive distal muscle wasting, regression, and the more specific facial gestalt reduce diagnostic confidence but do not fundamentally oppose the diagnosis, especially given known phenotypic variability and that the succinct case description may be incomplete.

Conversely, there are no strong contradictory features that clearly point away from TBCK‑related disease; most atypical or unmentioned findings (cutis marmorata, diastasis recti, pectus carinatum, PVL) are either nonspecific or potentially secondary to severe hypotonia and chronic illness. On balance, the overall constellation is much more specific than a generic cerebral palsy or hypoxic‑ischemic picture and is highly compatible with IHPRF3/TBCK‑related neurodevelopmental disorder. Therefore, the proposed diagnosis should remain prioritized as a plausible and clinically coherent explanation, while recognizing that molecular confirmation (TBCK biallelic variants) is essential.


