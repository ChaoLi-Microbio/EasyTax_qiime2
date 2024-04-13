# Classifiers fitting for qiime2 (trained for full region and v4 region)
## 6 classifiers for qiime2 (Trained from GreenGene2, Sliva138.1 & RDP)

### First of all, install conda and qiime2 according to the official doc!
#Then, put the classifier into the folder with your data
	
	cd (your data directory/folder) 

#edit your metadata.txt file according to example file (barcodes, samples_id, and other categorical description column etc.)\
#activate conda environment(Not limited to qiime2-2022.11)

	conda activate qiime2-2022.11

#produce metadata(mainfest)\
#(change the format of "\t$PWD/"$1"_L001_R1_001.fastq\t$PWD/"$1"_L001_R2_001.fastq" below to match with your sample name)

	awk 'NR==1{print "sample-id\tforward-absolute-filepath\treverse-absolute-filepath"} \
	NR>1{print $1"\t$PWD/"$1"_L001_R1_001.fastq\t$PWD/"$1"_L001_R2_001.fastq"}' \
	metadata.txt > manifest

#demutiplexing (demux.qza)

	qiime tools import \
	--type 'SampleData[PairedEndSequencesWithQuality]' \
	--input-path manifest \
	--output-path demux.qza \
	--input-format PairedEndFastqManifestPhred33V2

#Denoise (table.qza, rep-seqs.qza), adjusting primer length lined up with your design

	time qiime dada2 denoise-paired \
	--i-demultiplexed-seqs demux.qza \
	--p-n-threads 14 \
	--p-trim-left-f 19 --p-trim-left-r 20 \
	--p-trunc-len-f 250 --p-trunc-len-r 250 \
	--o-table table.qza \
	--o-representative-sequences rep-seqs.qza \
	--o-denoising-stats denoising-stats.qza


#tax annotation by DATABASE (taxonomy.qza)

	qiime feature-classifier classify-sklearn \
	--i-classifier classifier_gg2_full.qza \
	--i-reads rep-seqs.qza \
	--o-classification taxonomy.qza

#Visualization\
#Produce and check the sequencing quality view.qiime2.org (Optional)

	qiime demux summarize \
	--i-data demux.qza \
	--o-visualization demux.qzv

#table.qzv

	qiime feature-table summarize \
	--i-table table.qza \
	--o-visualization table.qzv \
	--m-sample-metadata-file metadata.txt

#rep-seqs.qzv

	qiime feature-table tabulate-seqs \
	--i-data rep-seqs.qza \
	--o-visualization rep-seqs.qzv

#taxonomy.qzv

	qiime metadata tabulate \
	--m-input-file taxonomy.qza \
	--o-visualization taxonomy.qzv

#taxa-bar-plots.qzv

	qiime taxa barplot \
	--i-table table.qza \
	--i-taxonomy taxonomy.qza \
	--m-metadata-file metadata.txt \
	--o-visualization taxa-bar-plots.qzv

#denoising-stats.qzv

	qiime metadata tabulate \
	--m-input-file denoising-stats.qza \
	--o-visualization denoising-stats.qzv

#taxonomy.txt

	qiime tools export \
	--input-path taxonomy.qza \
	--output-path taxonomy/
  
#table, sequence, taxonomy txt_conversion\
#otutable.txt

	qiime tools export \
	--input-path table.qza \
	--output-path table/
	  cd table
	biom convert -i feature-table.biom -o feature-table.txt --to-tsv

#rep_sequence.fasta

	qiime tools export \
	--input-path rep-seqs.qza \
	--output-path seqs/

#Tree

	qiime phylogeny align-to-tree-mafft-fasttree \
	--i-sequences rep-seqs.qza \
	--output-dir phylogeny-align-to-tree-mafft-fasttree
