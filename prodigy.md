Training new categories with Prodigy 

https://prodi.gy/docs/cookbook<br>
https://support.prodi.gy/t/train-a-new-ner-entity-with-multi-word-tokens/227/2

## load the model 
`python -m spacy download es_core_news_md`

## create a dataset
`prodigy dataset gam "The GAM dataset" --author ajanco`

## create a PATTERNS.JSONL file with priming examples. Should take the form of:
```
{"label": "NOMBRE", "pattern": "Gustavo"} 
```

## Load the UI to generate training data 
`prodigy ner.teach gam es_core_news_md gam_text.txt --patterns nombre_PATTERNS.JSONL --label NOMBRE`

## Save with either the little disk icon in Prodigy or ctrl-C
```
Saved 890 annotations to database SQLite
Dataset: gam
Session ID: 2018-06-14_03-39-15`
```

## Train the model on the new category (this one is for 100 iterations)
`prodigy ner.batch-train gam es_core_news_md -n 100 --output /home/digitalscholarship/Projects/spaCy/models --label NOMBRE`

## Run inference for category NOMBRE
```python
import spacy
nlp = spacy.load('/home/digitalscholarship/Projects/spaCy/models')

with open('/home/digitalscholarship/Projects/spaCy/gam_text.txt', 'r') as f:
        nlp =spacy.load('/home/digitalscholarship/Projects/spaCy/models')
        doc = nlp(u'{}'.format(f.read()))
        for ent in doc.ents:
                if ent.label_ == 'NOMBRE':
                        print(ent.text)
```
