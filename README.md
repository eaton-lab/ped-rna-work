

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
	> SPTREE/tree-set-full.nwk
```

### 7. infer sptree from unrooted gene trees [`run_astral`]
```./scripts/run_astral
	-t SPTREE/tree-set-full.nwk \
	-o SPTREE/full-sptree.nwk \
	-r SRR11748982 \
	-c 20
```

### 8. root gene trees based on sptree outgroups to get rooted-multilabeled-trees [`twig tree-rooter`]
```twig tree-rooter \
	-i SPTREE/tree-set-full.nwk \
	-
	> SPTREE/tree-set-rooted-multi-copy.nwk ...

```

### 9. root gene trees based on sptree outgroups to get rooted-singlecopy-trees [`twig tree-rooter`]
```twig tree-rooter -i SPTREE/tree-set-full.nwk > SPTREE/tree-set-rooted-single-copy.nwk ...```

### 10. infer sptree in astral and networks in phylonet [...]
```...```

### 11. run csubst on rooted-multilabeled-tree set [...]
```...```

### 11. run generax to label edges as dups/losses [...]
twig macse \                                                                                                                                                             
    -i "${NT}" -o "${DIR}" -p "${HOG}" \                                                                                                                                 
    -mh "${HOMOLOGY11}" \                                                                                                                                                
    -mi "${HOMOLOGY12}" \                                                                                                                                                
    -mc "${MIN_COV}" \                                                                                                                                                   
    -tx "${TRIM1E}" \                                                                                                                                                    
    -ti "${TRIM1I}" \                                                                                                                                                    
    -xa \                                                                                                                                                                
    -l D | tee -a "${LOG}"
    

12. run csubst on gtree, reconciled tree, and unreconciled tree
13. compute csubst score differences on trees


```bash
# run macse pipeline on each CDS file
twig macse -...

# run tree inference on each CDS file
./scripts/run_raxml ...

# get all trees in one multi-fasta
cat RESULTS4/*/*.final.nt.fa.raxml.support > RESULTS4.trees.nwk

# trim/filter trees
twig ...
```


  -h, --help                                   show this help message and exit
  -d str, --delim str                          delimiter to split tip labels
  -i int [int ...], --idxs int [int ...]       index of delim'd items to keep
  -j str, --join str                           join character on delim'd items
  -k str [str ...], --keep-set str [str ...]   tip labels to keep
  -o path, --out path                          outfile path prefix else result is printed to stdout
  -t path [path ...], --trees path [path ...]  glob path to tree files
  -m int, --min-tips int                       min tips after pruning [4]
  -c int, --max-copies int                     filter trees with >c gene copies from a taxon [None]
  -O path, --outgroups path                    filepath containing tip labels as outgroups for -eo
  -eo float, --edge-outlier-outgroup float     exclude outgroup edges >eo stdev from the mean len
  -ei float, --edge-outlier-ingroup float      exclude ingroup edges >ei stdev from the mean len


```bash
# filter trees: single-copy 
$ scripts/run_tree_filter \
	-d "|" \
	-i 0 \
	-m 20 \
	-c 1 \
	-O SPTREE/OUTGROUPS.txt \
	-o SPTREE/single-copy-gtrees.nwk \
	-t RESULTS4/*/*.final.nt.fa.raxml.support
	
# infer species tree
$ scripts/run_astral \
	-c 20 \
	-r SRR11748982 \
	-t SPTREE/single-copy-gtrees.nwk \
	-o SPTREE/single-copy-gtrees-sptree.nwk

# root trees
./scripts/run_tree_rooting \
	-t SPTREE/single-copy-gtrees.nwk \
	-s SPTREE/single-copy-gtrees-sptree.nwk \
	-o SPTREE/single-copy-gtrees-rooted.nwk \
	-O SPTREE/OUTGROUPS-PLUS.txt

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
```
