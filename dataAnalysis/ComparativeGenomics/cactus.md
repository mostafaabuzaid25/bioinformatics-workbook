---
title: Comparative Genomics
layout: single
header:
  overlay_color: "444444"
  overlay_image: /assets/images/dna.jpg
---


# Efficient multi-genome aligner with Progressive Cactus

Do you have hundreds of large vertebrate genomes? Are there no high quality reference species near any of them? Maybe you need Progressive Cactus! This prickly pipeline can align large vertebrate genomes and has been tested on a large dataset of 600 amniote genomes. 

**Publications**

* **Armstrong et al, 2020** - [Progressive Cactus is a multiple-genome aligner for the thousand-genome era](https://www.nature.com/articles/s41586-020-2871-y)
* **Paten et al, 2011** - [Cactus: Algorithms for genome multiple sequence alignment](https://pubmed.ncbi.nlm.nih.gov/21665927/)
* **PubMed** - [List of papers which cite the Cactus algorithm](https://pubmed.ncbi.nlm.nih.gov/?sort=pubdate&linkname=pubmed_pubmed_citedin&from_uid=21665927)

<hr />

## Draft: Reorganize the following text


If anyone is interested to run this on any of your genome, I've a run script that uses the container with persistent overlay script to make it more efficient.  The maize-to-maize alignment takes about 12 hrs on Nova huge node and it increases linearly with every addition of the genome.

**How to run progressive Cactus?**

This method uses persistant overlays, where you mount a writeable drive for the Container. If properly configured (setup overlay on the $TMPDIR), it will speed up the I/O and make it run faster. The downside is that the overlay can only use ext3 format which has the upper bound of 16Tb total size (roughly 25 maize genomes is the max you can run with this size).

**See more info here:**
https://sylabs.io/guides/3.5/user-guide/persistent_overlays.html

``` bash

#!/bin/sh
#SBATCH --nodes=1
#SBATCH --mem=0
#SBATCH --time=1-00:00:00
#SBATCH --partition=huge
#SBATCH -J cactus
#SBATCH -o cactus.o%j
#SBATCH -e cactus.e%j
#SBATCH --mail-user=username@email.com
#SBATCH --mail-type=begin
#SBATCH --mail-type=end
​
set -o nounset -o errexit -o xtrace
ml singularity
########################################
# parameters
########################################
readonly CACTUS_IMAGE=/work/triffid/arnstrm/NAM-MOA/non-masked/cactus_v1.0.0.sif
readonly JOBSTORE_IMAGE=jobStore.img
readonly SEQFILE=NAM-cactus.txt
readonly OUTPUTHAL=NAM-cactus.hal
# extra options to Cactus
#readonly CACTUS_OPTIONS='--root mr' # NOTE: specific to evolverMammals.txt; change/remove for other input seqFile
​
########################################
# ... don't modify below here ...
​
readonly CACTUS_SCRATCH=/mnt/bgfs/arnstrm/${SLURM_JOB_ID}
​
if [ ! -e "${JOBSTORE_IMAGE}" ]
then
  restart=''
  mkdir -m 777 ${CACTUS_SCRATCH}/upper
  truncate -s 10T "${JOBSTORE_IMAGE}"
  singularity exec --bind $CACTUS_SCRATCH ${CACTUS_IMAGE} mkfs.ext3 -d ${CACTUS_SCRATCH} "${JOBSTORE_IMAGE}"
else
  restart='--restart'
fi
​
#singularity exec --bind $CACTUS_SCRATCH ${CACTUS_IMAGE} e2fsck -f "${JOBSTORE_IMAGE}"
#singularity exec --bind $CACTUS_SCRATCH ${CACTUS_IMAGE} resize2fs "${JOBSTORE_IMAGE}" 2T
​
​
# Use empty /tmp directory in the container (to avoid, e.g., pip-installed packages in ~/.local)
mkdir -m 700 -p ${CACTUS_SCRATCH}/tmp
​
# the toil workDir must be on the same file system as the cactus jobStore
singularity exec --overlay ${JOBSTORE_IMAGE} ${CACTUS_IMAGE} mkdir -p /cactus/workDir
srun -n 1 singularity exec --bind $CACTUS_SCRATCH \
                           --cleanenv \
                           --no-home \
                           --overlay ${JOBSTORE_IMAGE} \
                           --bind ${CACTUS_SCRATCH}/tmp:/tmp \
                           ${CACTUS_IMAGE} \
  cactus ${CACTUS_OPTIONS-} ${restart-} --workDir=/cactus/workDir --binariesMode local /cactus/jobStore "${SEQFILE}" "${OUTPUTHAL}"
  
the input required is a text file, having location of the genomes and their short name (if you have a tree, it would speed this up).

```

the input file can be either a text file with short name and file location

```

B73 B73.chr-only.fa
B97 B97.chr-only.fa
CML103 CML103.chr-only.fa
CML228 CML228.chr-only.fa
CML247 CML247.chr-only.fa
CML277 CML277.chr-only.fa
CML322 CML322.chr-only.fa
CML333 CML333.chr-only.fa

```

or tree with the shortname, file location:

```

(M37W,(Tx303,((M162W,((Ky21,(Oh43,Oh7b):0.04664300893541594):0.004626244719863844,((HP301,(IL14H,P39):0.16165835119502328):0.04689661505165384,(B73,(B97,MS71):0.010608700907414835):0.01326176053007493):0.0048914649531996155):0.011260552502428117):0.0259188675537335,(parviglumis,((Tzi8,Mo18W):0.04350664577266931,((CML333,(CML322,((CML103,(CML247,CML277):0.004580436947144372):0.009996301777095812,((CML52,CML69):0.027913266423794738,(Ki11,(CML228,Ki3):0.05681998397810822):0.05052236453674854):0.0093997108824835):0.022981591644668158):0.011746200791024878):0.009075556769572455,(NC350,NC358):0.6553806225657925):0.014254493408218327):0.03136321107018052):0.016266960292499643):0.007967801111030329));
parviglumis Zea_mays_parviglumis-TIL11.chr-only.fa
B73 B73.chr-only.fa
B97 B97.chr-only.fa
CML103 CML103.chr-only.fa
CML228 CML228.chr-only.fa
CML247 CML247.chr-only.fa
CML277 CML277.chr-only.fa
CML322 CML322.chr-only.fa
CML333 CML333.chr-only.fa
CML52 CML52.chr-only.fa
CML69 CML69.chr-only.fa
HP301 HP301.chr-only.fa
IL14H IL14H.chr-only.fa
Ki11 Ki11.chr-only.fa
Ki3 Ki3.chr-only.fa
Ky21 Ky21.chr-only.fa
M162W M162W.chr-only.fa
M37W M37W.chr-only.fa
Mo18W Mo18W.chr-only.fa
MS71 MS71.chr-only.fa
NC350 NC350.chr-only.fa
NC358 NC358.chr-only.fa
Oh43 Oh43.chr-only.fa
Oh7b Oh7b.chr-only.fa
P39 P39.chr-only.fa
Tx303 Tx303.chr-only.fa
Tzi8 Tzi8.chr-only.fa

```

Having a tree will slightly speed up the process (it will select the options based on its divergence rather than estimating it from the data).

One more thing: with the output file (hal format), you can eventually add more genomes to the existing alignments without redoing everything.

However it depends on where your new genome falls within the tree
9:39
(if more ancestral, you will have to align it will every genome )

* **GitHub** - [ComparativeGenomicsToolkit/hal](https://github.com/ComparativeGenomicsToolkit/hal/blob/master/README.md)

many utility programs for hal format exists and they can also export in other formats as well.

... generating chain files (another page) should follow this.


---
[Table of contents](compGenomics_index.md)