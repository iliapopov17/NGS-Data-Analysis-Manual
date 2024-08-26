# Preparing the alignment and building trees

- `04_04_Lab_journal.ipynb` - contains this whole pipeline done
- scripts - contains all R scripts used in this pipeline

> For this work, we will use the alignment of [SUP35 gene](https://www.yeastgenome.org/locus/S000002579) obtained by `prank` considering codons

**_Input_**

```bash
prank -codon -d=data/SUP35_10seqs.fa -o=data/SUP35_aln_prank.best.fas -F
```

<div style='justify-content: center'>
<img src="https://github.com/iliapopov17/NGS-Handbook/blob/main/04_Phylogenetics/04_04_Trees/imgs/tree_building_overview.png" align='center', width="50%">
</div>

_Tree building overview_

### 1) How to cut bad areas out of the alignment using `trimAl`?

**_Input_**

```bash
trimal -in data/SUP35_aln_prank.best.fas -out data/SUP35_aln_prank.trim.fas -automated1
```

### 2) How to fit an evolution model in `ModelTest` (`ModelTest-NG`)?

**_Input_**

```bash
modeltest-ng -i data/SUP35_aln_prank.trim.fas -o data/modeltest/SUP35_trim_modeltest
```

**_Output_**

```
                     Model         Score        Weight
   BIC             TIM3+G4    18180.5614        0.3950
   AIC           TIM3+I+G4    18041.1550        0.5377
  AICc           TIM3+I+G4    18041.1550        0.5377
```

In total we see that the `TIM3+G4` is recognised as the best model!

### 3) Building an ML-tree in RAxML-NG using the selected model
> Let's focus on the BIC for consistency.

First, let's check that our tree is being built at all

**_Input_**

```bash
raxml-ng --check --msa data/SUP35_aln_prank.trim.fas  --model TIM3+G4 --prefix data/raxml/SUP35_raxml_test
```

**_Output_**

```
Alignment can be successfully read by RAxML-NG.
```

Great! Let's go!

**_Input_**

```bash
raxml-ng --msa data/SUP35_aln_prank.trim.fas --model TIM3+G4 --prefix data/raxml/SUP35_raxml --threads 2 --seed 222  --outgroup SUP35_Kla_AB039749
```

### 4) Drawing the resulting tree (the best ML tree)

**_Input_**

```bash
Rscript scripts/draw_tree.R data/raxml/SUP35_raxml.raxml.bestTree imgs/SUP35_raxml.png
```

**_Output_**

<div style='justify-content: center'>
<img src="https://github.com/iliapopov17/NGS-Handbook/blob/main/04_Phylogenetics/04_04_Trees/imgs/SUP35_raxml.png" align='center', width="50%">
</div>

### 5) Model selection in ModelFinder (IQ-TREE)
> Which model of evolution was found to be the most appropriate for our alignment?

**_Input_**

```bash
iqtree2 -m MFP -s data/SUP35_aln_prank.trim.fas --prefix data/modelfinder/SUP35_MF2
head -42 data/modelfinder/SUP35_MF2.iqtree | tail -6
```

**_Output_**

```
Best-fit model according to BIC: TIM3+F+G4

List of models sorted by BIC scores: 

Model                  LogL         AIC      w-AIC        AICc     w-AICc         BIC      w-BIC
TIM3+F+G4         -8993.686   18035.372 +   0.0517   18035.972 +   0.0549   18170.092 +    0.737
```

We see that the model `TIM3+F+G4` is recognised as the best!

### 6) Do the models selected by ModelTest and ModelFinder differ, and how much?<br>
In general, we got the same thing. Only ModelFinder also threw in information about the empirical frequencies of the letters themselves in the alignment.

|    |ModelTest|ModelFinder|
|----|---------|-----------|
|Model|TIM3+G4|TIM3+F+G4|
|BIC|18180.5614|18170.092|

### 7) Build an ML tree in IQ-TREE using the selected model.

**_Input_**

```bash
iqtree2 -m TIM3+F+G4 -s data/SUP35_aln_prank.trim.fas --prefix data/iqtree/SUP35_iqtree
```

### 8) Drawing of the resulting tree (best ML tree)

**_Input_**

```bash
Rscript draw_tree.R data/iqtree/SUP35_iqtree.treefile imgs/SUP35_iqtree.png
```

**_Output_**

<div style='justify-content: center'>
<img src="https://github.com/iliapopov17/NGS-Handbook/blob/main/04_Phylogenetics/04_04_Trees/imgs/SUP35_iqtree.png" align='center', width="50%">
</div>

The trees obtained with RAxML and IQTREE are fundamentally similar.<br>
They have different views on how well the outer groups diverge, and they have different topologies.

### 9) Comparison of likelihood (log likelihood) of trees obtained with different models and before and after filtering
>What conclusion can be drawn from this?

**_Input_**

```bash
iqtree2 -s data/SUP35_aln_prank.best.fas -pre data/iqtree_unfilt/SUP35_iqtree_unfilt
grep "Log-likelihood" data/iqtree_unfilt/SUP35_iqtree_unfilt.iqtree
```

**_Output_**

```
Log-likelihood of the tree: -9696.9044 (s.e. 160.3706)
```

**_Input_**

```bash
grep "Log-likelihood" data/iqtree/SUP35_iqtree.iqtree
```

**_Output_**

```
Log-likelihood of the tree: -8993.1633 (s.e. 149.1347)
```

**_Input_**

```bash
iqtree2 -s data/SUP35_aln_prank.best.fas -m JC -pre data/iqtree_JC/SUP35_iqtree_JC
grep "Log-likelihood" data/iqtree_JC/SUP35_iqtree_JC.iqtree
```

**_Output_**

```
Log-likelihood of the tree: -10482.7253 (s.e. 176.1729)
```

|              |Unfilt|TIM3+F+G4|JC|
|--------------|------|---------|--|
|Log-likelihood|-9696.9044|-8993.1633|-10482.7253|

- Before filtering Log-likelihood = -9696.9044
- When using the `TIM3+F+G4` model = -8993.1633
- When using `JC` model = -10482.7253<br>

That said, the topology is the same everywhere! Anyway, if we only need to look at the topology, we can run iqtree with either model...

### 10) Generation of 100 replicas of a regular bootstrap

**_Input_**

```bash
time iqtree2 -s data/SUP35_aln_prank.trim.fas -m TIM3+F+G4 -redo -pre data/iqtree_bootstrap/SUP35_TIM3_b -b 100
```

### 11) Generation of 1000 ultrafast bootstrap replicas

**_Input_**

```bash
time iqtree2 -s data/SUP35_aln_prank.trim.fas -m TIM3+F+G4 -redo -pre data/iqtree_ultrafast_bootstrap/SUP35_TIM3_ufb -bb 1000
```

### 12) What is the difference between normal and ultrafast bootstrap runtimes and the values obtained?

Generating 100 replicas of regular bootstrap took 3:17.00, while generating 1000 replicas of ultrafast bootstrap took 3.258. That's a huge difference!

**_Input_**

```python
bootstrap_100 = 3 * 60 + 17
ultrafast_bootstrap_1000 = 3.258

print(f'Generation of ultrafast bootstrap is faster: {bootstrap_100 / ultrafast_bootstrap_1000} times')
```

**_Output_**

```
Generation of ultrafast bootstrap is faster: 60.46654389195826 times
```

### 13) Running the previous command, but with generation: 1000 ultrafast bootstrap + 1000 alrt + abayes

**_Input_**

```bash
iqtree2 -s data/SUP35_aln_prank.trim.fas -m TIM3+F+G4 -pre data/iqtree_ufb_alrt_abayes/SUP35_TIM3_B_alrt_abayes -bb 1000 -alrt 1000 -abayes
```

### 14) Drawing the resulting tree with three supports

**_Input_**

```bash
Rscript scripts/draw_tree_max.R data/iqtree_ufb_alrt_abayes/SUP35_TIM3_B_alrt_abayes.treefile imgs/SUP35_TIM3_B_alrt_abayes.png
```

**_Output_**

<div style='justify-content: center'>
<img src="https://github.com/iliapopov17/NGS-Handbook/blob/main/04_Phylogenetics/04_04_Trees/imgs/SUP35_TIM3_B_alrt_abayes.png" align='center', width="50%">
</div>

All values ​​- `alrt`, `abayes` and `ufb` are positively correlated - that is, if the indicators are high, then they are all high, as a rule. But this is not proportional.
In the tree above, the indicator values ​​are replaced with the symbols of the sun, yin-yang and asterisk for educational purposes. In real life they don’t use this, but giving bare numbers through a slash is also not comme il faut...
