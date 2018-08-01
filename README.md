# text-classification
A text classification project using the dataset from YunYi Cup

### Problem
The data set contains 22w (x,y) records.

x is a comment text on some tourist attractions, y is the corresponding comment score range from 1 to 5

This is a supervised learning problem. At test time, given a comment, we want to predict it's score.

Since the score is an ordinal variable, it turns out that regression is more suitable than classification here.
**So the metric is mse**

Note: I use 60% of the data as training set and 40% as validation set, no test set.

### Baseline Model
Stacking: 
1. first layer:  Logistic regression and Naive Bayes
2. second layer: xgboost

validation mse: 0.405770

### NN with static word2vec embedding
Some common parameters:
```
max_features = 20000
maxlen = 150
embed_size = 128
```
The word2vec embedding is trained from the 22w comment texts with gensim word2vec model.

#### Bidrectional GRU with spatial dropout
```
def get_model():
    inp = kl.Input(shape=(maxlen,))
    embed = kl.Embedding(max_features,embed_size,weights=[embedding_matrix],trainable=False)(inp)
    embed = kl.SpatialDropout1D(0.2)(embed)
    gru = kl.Bidirectional(kl.CuDNNGRU(128,return_sequences=True))(embed)
    maxpool = kl.GlobalMaxPool1D()(gru)
    avgpool = kl.GlobalAvgPool1D()(gru)
    out = kl.concatenate([maxpool,avgpool])
    out = kl.Dense(1)(out)
    model = ks.models.Model(inp,out)
    model.compile(loss="mse", optimizer='adam', metrics=[])
    return model
```
![gru_with_spatial_dropout](http://ok669z6cd.bkt.clouddn.com/gru_spatialdrop_static.png)

The best validation mse is 0.406670, slightly worse than the baseline model.
