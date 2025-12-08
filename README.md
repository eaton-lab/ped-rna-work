



1. trinity assembly of cds and translation to proteins [Yue]
2. orthogroup hierarchical assembly of protein seqs [Yue]
3. alignment and trimming of cds sequences in orthogroups [`run_macse2_parallel`]
4. infer partitioned ML trees [`run_raxml_parallel`]
5. filter dataset to high quality alignments and trees [...]
6. infer sptree from unrooted gene trees [...]
7. infer set of reconciled gene trees given the species tree [...]
8. 

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
