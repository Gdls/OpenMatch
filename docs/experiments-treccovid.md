# Cord19

## Inference
Get checkpoints.
```
wget http://boston.lti.cs.cmu.edu/appendices/CORD19-Zhuyun-Dai/covidscibert_msmarco9600steps.zip -P ./checkpoints
unzip ./checkpoints/covidscibert_msmarco9600steps.zip -d ./checkpoints
```

Get data from Google Drive.
* [round1](https://drive.google.com/open?id=17CEoLecus232pCDwCECaJD4vNfh4OQao)
* [round2](https://drive.google.com/open?id=1O6e8gXFnykkhN2icMCuWlMZkKUv6B3fV)

Preprocess.
```
python ./data/preprocess.py \
  -input_trec anserini.covid-r2.fusion2.txt \
  -input_queries questions_cord19-rnd2.txt \
  -input_docs cord19_0501_titabs.jsonl \
  -output ./data/test_trec-covid-round2.jsonl
```

Inference.
```
CUDA_VISIBLE_DEVICES=0 \
python inference.py \
        -task classification \
        -model bert \
        -max_input 1280000 \
        -test ./data/test_trec-covid-round2.jsonl \
        -vocab allenai/scibert_scivocab_uncased \
        -pretrain allenai/scibert_scivocab_uncased \
        -checkpoint ./checkpoints/checkpoint-9600/pytorch_model.bin \
        -res ./results/scibert.trec \
        -max_query_len 32 \
        -max_doc_len 256 \
        -batch_size 32
```

## train
Get training data.
```
wget http://boston.lti.cs.cmu.edu/appendices/CORD19-Zhuyun-Dai/seanmed.train.320K.tsv -P ./data
```

Preprocess the *tsv* file to jsonl: *{"query":, "doc":, "label":}* and produce *dev* file.
```
python ./data/preprocess.py \
  -input_trec run.covid-r1.fusion.query-udel.bm25.txt \
  -input_qrels qrels-cord19-round1.txt \
  -input_queries questions_cord19-rnd1.txt \
  -input_docs cord19_0501_titabs.jsonl \
  -output ./data/dev_trec-covid-round1.jsonl
```

Train.
```
CUDA_VISIBLE_DEVICES=0 \
python train.py \
        -task classification \
        -model bert \
        -train ./data/seanmed.train.320K.jsonl \
        -max_input 1280000 \
        -save ./checkpoints/scibert.bin \
        -dev ./data/dev_trec-covid-round1.jsonl \
        -qrels qrels-cord19-round1.txt \
        -vocab allenai/scibert_scivocab_uncased \
        -pretrain allenai/scibert_scivocab_uncased \
        -res ./results/scibert.trec \
        -metric ndcg_cut_10 \
        -max_query_len 32 \
        -max_doc_len 256 \
        -epoch 5 \
        -batch_size 16 \
        -lr 2e-5 \
        -eval_every 1000
```

For ReInfoSelect training, add
```
-reinfoselect
```
