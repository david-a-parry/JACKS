## FORKED

This repository is a forked version from https://github.com/felicityallen/JACKS
with a rearranged folder structure for the sole purpose of allowing remote
installs via pip. For example:

    pip3 install git+git://github.com/david-a-parry/JACKS.git

Or in a conda environment using an environment.yml:

~~~
dependencies:
  - pip:
    - git+git+git://github.com/david-a-parry/JACKS.git
~~~

If installed via pip, the executables `JACKS.py` and `plot_jacks_heatmap.py` will be in `$HOME/.local/bin` (on linux systems). You may need to add this location to your `$PATH`, after which these script will be accessible:

~~~
export PATH=$PATH:$HOME/.local/bin
JACKS.py --help
~~~

All the code remains the work of the original authors.


## To run JACKS on full screen data

```bash
JACKS.py countfile replicatemapfile sgrnamappingfile --rep_hdr=replicate_hdr 
    --sample_hdr=sample_hdr --common_ctrl_sample=ctrl_sample --sgrna_hdr=sgrna_hdr --gene_hdr=gene_hdr --outprefix outprefix --ctrl_genes=negative_controls
```

where

* `countfile`  - A tab or comma (if comma, must end in '.csv') delimited file containing the raw counts of each guide in 
each cell line. 
The first column should contain the guide ids.
The remaining columns can contain the count data, with the first row containing the column headings, which should be 
the replicate identifiers, which should match those in the replicate map file.
Only those columns with an entry in the `replicatemapfile` file (see below) will be processed, other columns will be ignored.

e.g.
```
sgRNA	Control 1 Replicate A	Control 1 Replicate B	Control 2 Replicate A	Control 2 Replicate B	Sample 1 Replicate A	Sample 1 Replicate B	Sample 2 Replicate A	...
Guide 1	648	557	423	288	475	342	218...
Guide 2	609	229	990	386	616	218	578...
Guide 3	481	576	710	390	338	815	1733...
```

* `replicatemapfile` is a tab or comma (if comma, must end in `.csv`) delimited file containing the mappings from replicates
 to samples. The file can contain other columns, 
  - `--rep_hdr` specifies the column header of the column containing the replicate identifiers (matching the column headers in the count file). The default value is `Replicate`. In the example below this would be `Replicate Header`.
  - `--sample_hdr` specifies the column header of the column containing the sample mappings for each replicate (i.e. an identifier for the cell line or condition). The default value is `Sample`. In the example below this would be `Sample Header`.
  - `--common_ctrl_sample` For use of a common control sample across all measurements, this field specifies the sample identifier of the sample which is to be used as a control by JACKS for all samples. The default value is `CONTROL`
  - `--ctrl_sample_hdr` For specifying a different control per sample, this specifies the column header of the column in the replicatemapfile which contains the sample identifiers of the control for each sample (e.g. in the example below this is `Control Header`, or see `example/example_repmap_matched_ctrls.tab`). For control samples, the control and sample identifiers should be identical. The default value is None. 

e.g.
```
Replicate Header	Sample Header	Control Header
Control 1 Replicate A	Control 1	Control 1
Control 1 Replicate B	Control 1	Control 1
Control 2 Replicate A	Control 2	Control 2
Control 2 Replicate B	Control 2	Control 2
Sample 1 Replicate A	Sample 1	Control 1
Sample 1 Replicate B	Sample 1 	Control 1
Sample 2 Replicate A	Sample 2	Control 2
...
```

* `sgrnamappingfile` is a tab or comma (if comma, must end in `.csv`) delimited file containing the mappings from guides to genes.
The file can contain other columns, 
  - `--sgrna_hdr` specifies the column header of the column containing the guide identifiers. The default vale is `sgRNA`. In the example below this would be `sgRNA Header`.
  - `--gene_hdr` specifies the column header of the column containing the gene identifiers. The default vale is `Gene`. In the example below this would be `Gene Header`.
If the count file has a gene column, the count file can be reused here.

```
sgRNA Header	Gene Header
Guide 1	BRAF
Guide 2	BRAF
Guide 3	KRAS
...
```

* `--outprefix`: the output prefix of the JACKS output files. The following output files will be produced.
  -  `outprefix_gene_JACKS_results.txt` contains the gene essentiality scores E(w) for each cell line
  - `outprefix_gene_std_JACKS_results.txt` contains the standard deviations of the gene essentiality scores std(w) for each cell line
  - `outprefix_gene_pval_JACKS_results.txt` contains the p-values for the gene essentiality scores for each cell line (only output if negative control genes or guides are specified)
  - `outprefix_grna_JACKS_results.txt` contains the gRNA efficacy scores E(X) and E(X^2) for each guide
  -  `outprefix_JACKS_full_data.pickle` is a pickle file containing the full screen results

* `--ctrl_genes`: (Required if p-value output is wanted) Either, the name of a gene (as used in sgrnamappingfile) specifying a set of negative control guides (e.g. these could be intergenic, non-targeting etc) OR a text file containing a list (one per line) of genes to use as negative controls. Also used to infer variances in case of single replicate data. For positive selection use additional flag --positive, else negative selection will be applied.
  
* `--positive`: (optional) Use if positive selection is wanted, else will apply negative selection.
  
example:
```bash
JACKS.py example/example_count_data.tab example/example_repmap.tab example/example_count_data.tab --common_ctrl_sample=CTRL --gene_hdr=gene --outprefix=example_jacks/example_jacks --ctrl_genes=example/NEGv1.txt
```
```bash
JACKS.py example/example_count_data.tab example/example_repmap_matched_ctrls.tab example/example_count_data.tab --ctrl_sample_hdr=Control --gene_hdr=gene --outprefix=example_jacks_ctrls/example_jacks --ctrl_genes=example/NEGv1.txt
```

## OR to run JACKS on new screen with previously used library

```bash
JACKS.py countfile replicatemapfile sgrnamappingfile --rep-hdr=replicate_hdr 
    --sample_hdr=sample_hdr --common_ctrl_sample=ctrl_sample --sgrna-hdr=sgrna_hdr --gene-hdr=gene_hdr --reffile=grnaeffiacyfile --outprefix outprefix
```

all arguments as above except:
   
`--reffile` contains previously trained gRNA efficacy value. This is the file `outprefix_grna_JACKS_results.txt`
 returned above, or one of the pre-trained files available in https://github.com/felicityallen/JACKS/tree/master/reference_grna_efficacies
    
example:   

```
JACKS.py example/example_count_data.tab example/example_repmap.tab example/example_count_data.tab --common_ctrl_sample=CTRL --gene_hdr=gene --outprefix=example_jacks_ref/example_jacks --ctrl_genes=example/NEGv1.txt --reffile=example/example_grna_JACKS_results.txt```
```     
     
see `2018_paper_materials/README.txt` for further examples.

## Then, to plot heatmap outputs for a gene of interest

```bash
plot_heatmap.py picklefile gene (or "random" to randomly select one) outfile
```

where:

`picklefile`:  the pickle file output by run_JACKS.py

`gene`: the name of a gene to plot the output for (or "random" to select a random one)

`outfile`: the name of a .png output file to write the figure to

example:

```bash
plot_heatmap.py example_jacks/example_jacks_JACKS_results_full.pickle KRAS example_jacks/KRAS.png
```
 
