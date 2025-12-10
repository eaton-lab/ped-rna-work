

# OUTLINE 

### 1. trinity assembly of cds and translation to proteins [Yue]
```trinity ...```

### 2. orthogroup inference from protein seqs in Orthofinder [Yue]
```orthofinder ...```

### 3. trim/filter/align cds sequences in each orthogroup [`twig macse`]
```./scripts/run_twig_macse_parallel -i HOGDIR/*/1.cds.fa -o RESULTS4/ -c 40```

### 4. infer partitioned ML trees from CDS [`raxml-ng`]
```./scripts/run_raxml_parallel 8 10 RESULTS4/*/*.final.nt.fa```

### 5. concatenate all trees into tree-set-full.nwk
```cat RESULTS4/*/*.final.nt.fa.raxml.support > SPTREE/tree-set-full.nwk

### 6. filter tree set to discard bad tips and small trees [`twig tree-filter`]
```twig tree-filter \
	-i SPTREE/tree-set-full.nwk \
	-d '|' -di 0 \
	-s 20 \
	-ei 5 -eo 10 --exclude-outliers \
	--imap SPTREE/IMAP \
	> SPTREE/tree-set-full-filtered.nwk \
	2> SPTREE/tree-set-full-filtered.log
# 8374 trees passed filtering
```

### 7. infer sptree from unrooted gene trees [`run_astral`]
```./scripts/run_astral \
	-t SPTREE/tree-set-full-filtered.nwk \
	-o SPTREE/sptree-full-filtered.nwk \
	-r SRR11748982 \
	-c 20 \
	2> SPTREE/sptree-full-filtered.log
```

### 8. root gene trees based on sptree outgroups to get rooted-multilabeled-trees [`twig tree-rooter`]
```twig tree-rooter \
	-i SPTREE/tree-set-full-filtered.nwk \
	-s SPTREE/sptree-full-filtered.nwk \
	-R SPTREE/OUTGROUPS.txt \ 
	> SPTREE/tree-set-full-filtered-rooted.nwk \
	2> SPTREE/tree-set-full-filtered-rooted.log
# 8296 trees rerooted
```

### 9. filter to keep only single-copy trees from the rooted trees [`twig tree-filter`]
```twig tree-filter \
	-i SPTREE/tree-set-full-filtered-rooted.nwk \
	-c 1 \
	> SPTREE/tree-set-full-filtered-rooted-singlecopy.nwk \
	2> SPTREE/tree-set-full-filtered-rooted-singlecopy.log
# 4100 trees
```

### 10. infer sptree for multilabeled filtered tree set in astral [...]
```./scripts/run_astral \
	-t SPTREE/tree-set-full-filtered-rooted.nwk \
	-o SPTREE/sptree-full-filtered-rooted.nwk \
	-r SRR11748982 \
	-c 20 \
	> SPTREE/sptree-full-filtered-rooted.log
```

### 11. infer sptree for multilabeled filtered tree set in astral [...]
```./scripts/run_astral \
	-t SPTREE/tree-set-full-filtered-rooted-singlecopy.nwk \
	-o SPTREE/sptree-full-filtered-rooted-singlecopy.nwk \
	-r SRR11748982 \
	-c 20 \
	> SPTREE/sptree-full-filtered-rooted-singlecopy.log
```

### 12. filter trees for network analysis 1 (require >=15 from IMAP)
```twig tree-filter \
	-i SPTREE/tree-set-full-filtered-rooted.nwk \
	-I SPTREE/SPECIES-SAMPLING-39.txt \
	-c 15 \
	--subsample 
# 
```



### 13. run csubst on rooted-multilabeled-tree set [...]
```
# prepare fastas to match to tip names


# prepare csubst trait file


# prepare 

```
    

12. run csubst on gtree, reconciled tree, and unreconciled tree
13. compute csubst score differences on trees

# filter: get subset rooted gtrees and keep collapsed root edge
$ scripts/run_tree_filter \
	# -k SPTREE/SPECIES-SAMPLING-31.txt \
	--imap SPTREE/SPECIES-SAMPLING-32.imap.txt \
	--minmap SPTREE/SPECIES-SAMPLING-32.minmap.txt \
	-m 15 \
	-t SPTREE/single-copy-gtrees.nwk \
	-s SPTREE/single-copy-sptree.nwk \
	-o SPTREE/single-copy-subset31.nwk \
	-O SPTREE/OUTGROUPS.txt \
	--collapse-outgroup outgroup

