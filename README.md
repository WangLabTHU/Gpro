# GPrO: Generation, Prediction and Optimization - three steps to design artifitial promoters
## Install
`git clone https://github.com/WangLabTHU/Gpro.git`
## Quick Start
Using our data to quick start GPrO
### Demo 1: Design constitutive promoters of E. coli
In our previous research *'Synthetic promoter design in Escherichia coli based on a deep generative network'*, we designed artifitial promoters which had successfully expressed in E. coli. Using **Gpro** can easily reappear our result.

Train the generator using  natural E. coli constitutive promoters sequences:
```
import gpro
generator = gpro.Generators.WGAN()
generator.BuildModel('data/seq_ecoli.txt')
generator.Train()
```
Generate promoter sequences:
```
import numpy as np
N = 100000000 # the number of sequences to generate
seq = generator.Generator(np.random.normal(size=(N,generator.Z_DIM) ))
```
Train the Predictor using E. coli expression data:
```
predictor = gpro.Predictors.CNN()
predictor.BuildModel('data/exp_ecoli.txt')
predictor.Train()
```
Predict the expression of artifitial promoters:
```
exp = predictor.Predictor(seq)
```
Filter the Top 100 expression promoters to experiment：
```
I = np.argsort(exp)
I = I[::-1] # in descending order of expression
seqout = seq[I[:100]]
```

### Demo 2: Design promoter highly expression in yeast cell
To represent all function of **GPrO**, we provide the design process of highly-expressed promoter in yeast cell. In this section, we applied genetic algorithm to optimize promoter sequences.

In this demo, the AAE model was trained as a generator by promoters on the yeast natural genome. Then the prediction model was trained with the MPRA experimental data in yeast cells(Lubliner, et al., 2015), and the transcriptional strength of downstream genes was predicted by the promoter sequence. The genetic algorithm was used to search for the promoter sequence with the highest expression in the predictor. As a result,the expression level of the optimized artificial promoters was significantly higher than that of the yeast natural promoter sequence (Supplementary figure S2). It proved that Dpro can optimize the promoters with certain functional characteristics. 

Train Generator use natural yeast core promoter:
```
import gpro
gen = gpro.Generators.AAE()
gen.BuildModel('data/seq_yeast.txt')
gen.Train()
```
The input of Generator is a txt file, each line repressent a promoter sequence.

For example:
```
seq_yeast.txt:
TGGTGTGTGTTCCGAAACTGCTTCACCTGCCATTGTTTCGACGGCCACGGCTACTGTGAATGATGTTGTTACGGTCTATCCAACATGGAGGCCACAGACTACGAATGAAGAGTCTGTC
CACATTAGTTAATGCTTTTGATAAAATGTATATAAAGGCTGTCGTAATGTGCAGTAGTAAGGACCTGACTGTGTTTGTGGTTCTCTTCATTCTTGAACCTTGTCATTGGTAAAAGACC
AGAAAAAAAAATCGACATATAAAAGTGGGGCAGATACTTCGTGTGACAATGGCCAATTCAAGCCCTTTGGGCAGATGTTGCCCTTCTTCTTTCTTAAAAAGTCTTAGTACGATTGACC
TTTTTCCTTTATTCAACTTGGCGATGAGCTGAAAATTTTTTTGGTTAAGGACCCTTTAGAAGTATTGAATGTGGGAACAAAGACGACAAAAGGTAGTTTTTTCCTTGACTATACTGGT
AAAAACTTCATTAATAGTTACAACTTTTTCAATATCAAGTTGATTAAGAAAAAGAAAATTATTATGGGTTAGCTGAAAACCGTGTGATGCATGTCGTTTAAGGATTGTGTAAAAAAGT
TAATACTTACCCGATGTTAAATCTTATTTTTTCATGCAGTAAGTAATCATGCAACAAGAAAAACCCGTAATTAAGCGAACATAGAACAACTAGCATCCCCGATAAGACGGAATAGAAT
...
```
Promoters generated by generator are similar to input data

The following two graph would be saved at folder 'log/' after trained.

![Promoters generated by generator are similar to input data](https://github.com/WangLabTHU/Gpro/blob/master/yeast1.jpg)

Figure a present 6-mer frequences of natural promoter and generated promoter. Figure b plot JS distance of 6-mer distribution between generated sequences and train set (JS_train), generated sequences and valid set (JS_valid), train set and valid set (JS control, treated as performance of ideal generator).

Train the Predictor using yeast expression data:
```
pred = gpro.Predictors.CNN()
pred.BuildModel('data/exp_yeast.txt')
pred.Train()
```
The input of Predictive Module is a txt file, each line contain a promoter sequence with functional score, split by ' ' or '\t'.

For example:
```
exp_yeast.txt:
CAATTGCGTTTGATCTTATTTAATCTCTCTCTAATATTATTATTGAAAATAATATTTCTTTCCTATCAGCAATCGCTTACATATTCATAGTCATTTACTAGGGTATATACCAATAATA	10.753381
AATTTTCCTCGCAGTTTTTCGCTTGTTTAATGCGTATTTTCCCTGGATATATAAAATCCATCTGATTATTCGATAATCTCAAACAAACAACTCAAAACAAAAAAAACTAAATACAACA	12.363241
CCACTTCCTCAATTGCGTTTGATCTTATTTAATCTCTCTCTAATATATAGAAAAAAAAACCATCTGATTATTCGATAATCTCATCTCTAGAACAACACAGTTACTACAACAATCAATC	11.329836
TCTAATATATAAAATACGATTTATTTAGTTGTCTTTGTCAATCCTCATCTTTCTTTACTCATTATTTCATTTCGGTTTTGTCATCTCTAGAACAACACAGTTACTACAACAATCAATC	19.799712
ACCTTTCATCTGTATCCCGTATATTTAAGATGGCGTTTGCTTTCTCCGTTGATTTTTTTCCTTCTTAGTGATTTTTTTTCAAACAAACAACTCAAAACAAAAAAAACTAAATACAACA	13.796239
...
```
Predictor learn the motif features of yeast

Scatter of preditive value and true value would be saved at folder 'log/'.

![Predictor learn the motif features of yeast](https://github.com/WangLabTHU/Gpro/blob/master/yeast2.jpg)

Using Genetic algorithm to optimize promoter sequences:
```
opt = gpro.Optimizers.GeneticAthm(gen.Generator,pred.Predictor)
opt.run()
```
Promoters optimized by Optimizer have higher expression than natural promoters

Function score of generated sequences during optimizing, optimized sequences and following graph would be saved at folder 'EAresult/'

![Promoters optimized by Optimizer have higher expression than natural promoters](https://github.com/WangLabTHU/Gpro/blob/master/yeast3.jpg)
