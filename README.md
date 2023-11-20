# installation
#Installing Miniconda

conda update conda
#Installing Wget
conda install wget



##shotgun distribution
wget https://data.qiime2.org/distro/shotgun/qiime2-shotgun-2023.9-py38-linux-conda.yml
conda env create -n qiime2-shotgun-2023.9 --file qiime2-shotgun-2023.9-py38-linux-conda.yml

##tiny distribution
wget https://data.qiime2.org/distro/tiny/qiime2-tiny-2023.9-py38-linux-conda.yml
conda env create -n qiime2-tiny-2023.9 --file qiime2-tiny-2023.9-py38-linux-conda.yml

##active conda environment
conda activate qiime2-<distro>-2023.9


####metadata
wget \
  -O "sample-metadata.tsv" \
  "https://data.qiime2.org/2023.9/tutorials/moving-pictures/sample_metadata.tsv"
  
##imorting data
mkdir emp-single-end-sequences


qiime tools import \
  --type EMPSingleEndSequences \
  --input-path emp-single-end-sequences \
  --output-path emp-single-end-sequences.qza


##Demultiplexing sequences
qiime demux emp-single \
  --i-seqs emp-single-end-sequences.qza \
  --m-barcodes-file sample-metadata.tsv \
  --m-barcodes-column barcode-sequence \
  --o-per-sample-sequences demux.qza \
  --o-error-correction-details demux-details.qza

###Sequence quality control and feature table construction
#option 1:DADA2
qiime dada2 denoise-single \
  --i-demultiplexed-seqs demux.qza \
  --p-trim-left 0 \
  --p-trunc-len 120 \
  --o-representative-sequences rep-seqs-dada2.qza \
  --o-table table-dada2.qza \
  --o-denoising-stats stats-dada2.qza

#option 2:Deblur
qiime quality-filter q-score \
 --i-demux demux.qza \
 --o-filtered-sequences demux-filtered.qza \
 --o-filter-stats demux-filter-stats.qza

##denoising deblur
qiime deblur denoise-16S \
  --i-demultiplexed-seqs demux-filtered.qza \
  --p-trim-length 120 \
  --o-representative-sequences rep-seqs-deblur.qza \
  --o-table table-deblur.qza \
  --p-sample-stats \
  --o-stats deblur-stats.qza


###FeatureTable and FeatureData summaries
qiime feature-table summarize \
  --i-table table.qza \
  --o-visualization table.qzv \
  --m-sample-metadata-file sample-metadata.tsv
qiime feature-table tabulate-seqs \
  --i-data rep-seqs.qza \
  --o-visualization rep-seqs.qzv

###Generate a tree for phylogenetic diversity analyses
qiime phylogeny align-to-tree-mafft-fasttree \
  --i-sequences rep-seqs.qza \
  --o-alignment aligned-rep-seqs.qza \
  --o-masked-alignment masked-aligned-rep-seqs.qza \
  --o-tree unrooted-tree.qza \
  --o-rooted-tree rooted-tree.qza

##Alpha and beta diversity analysis
qiime diversity core-metrics-phylogenetic \
  --i-phylogeny rooted-tree.qza \
  --i-table table.qza \
  --p-sampling-depth 1103 \
  --m-metadata-file sample-metadata.tsv \
  --output-dir core-metrics-results






