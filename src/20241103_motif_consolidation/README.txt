Based on  /mnt/lab_data3/surag/kundajelab/choroid/src/analysis/20231114_motif_consolidation/README.txt

for x in `ls /oak/stanford/groups/akundaje/surag/projects/retina_Howard/models/20220202_bpnet/fold0/modisco/`; do mkdir pfms/$x ;  python ~/kundajelab/surag-scripts/modisco/modisco_to_pfm.py -m /oak/stanford/groups/akundaje/surag/projects/retina_Howard/models/20220202_bpnet/fold0/modisco/$x/modisco_results_allChroms_counts.hdf5 -f 2 -o pfms/$x/ ; done 

(for x in `ls pfms/*/*` ; do printf ">$x\n" ; cat $x ; done) |   sed "s/\//_/g" | sed 's/pfms_//' > modisco.counts.pfm

gimme cluster modisco.counts.pfm ./gimme_cluster_counts -t 0.999 

mkdir gimme_cluster_counts/meme
python scripts/pfms_to_meme.py -i gimme_cluster_counts/clustered_motifs.pfm -o gimme_cluster_counts/meme/

Ran TomTom v5.3.0 to match against Vierstra motif hits.
`mkdir gimme_cluster_counts/tomtom`
`for x in $(ls gimme_cluster_counts/meme/*) ; do (y=$(basename -s ".meme" $x) ; tomtom -no-ssc -oc . -verbosity 1 -text -min-overlap 5 -mi 1 -dist pearson -evalue -thresh 10.0 $x /srv/www/kundaje/surag/resources/motif_archetypes/pfm_meme_format/motifs.meme.txt  > gimme_cluster_counts/tomtom/$y.vierstra.tomtom.tsv) & done`

For each cluster extracted TomTom annotations.
`for x in  $(ls gimme_cluster_counts/tomtom/*) ; do cat $x | head -2 | tail -1 ; done | cut -f1,2,5|  sort -k3g | cut -f1-3 > tfs_initial.tsv`

Filtered the above file, made the following changes and stored to tfs_final.tsv:
- `Mullerglia_pattern_26`: one-off, weak-ish
- `Cone_pattern_19`: one-off, not major
- `Rodbipolar_pattern_19`: one-off
- `OFFconebipolar_pattern_22`: one-off
- `Average_244`: not major
- `Average_260`: not major
- `Average_254`: seems weak
- `Average_259`: not very common

To get representative motif for each cluster (use the version from the cell type with most instances):
`python scripts/get_representative.py --tfs tfs_final.tsv --pfm pfms/ --cluster-key gimme_cluster_counts/cluster_key.txt  -o tfs_final_w_rep.tsv`
`rm tfs_final.tsv`

Removed 3rd column (p-values) from `tfs_final_w_rep.tsv`.

Added names manually for each cluster. Unsure about `Average_273`.

Stored as MEME file: `python scripts/pfms_to_meme_final_list.py -i tfs_final_w_rep.tsv -o tfs_final_w_rep.meme`


-----------
For repressors, did similar.

Used scripts/modisco_negative_to_pfm.py 

for x in `ls /oak/stanford/groups/akundaje/surag/projects/retina_Howard/models/20220202_bpnet/fold0/modisco/`; do mkdir pfms_negative/$x ;python scripts/modisco_negative_to_pfm.py -m /oak/stanford/groups/akundaje/surag/projects/retina_Howard/models/20220202_bpnet/fold0/modisco/$x/modisco_results_allChroms_counts.hdf5 -f 2 -o pfms_negative/$x/ ; done

(for x in `ls pfms_negative/*/*` ; do printf ">$x\n" ; cat $x ; done) |   sed "s/\//_/g" | sed 's/pfms_//' > modisco.negative.counts.pfm

gimme cluster modisco.negative.counts.pfm  ./gimme_cluster_counts_negative -t 0.999

mkdir gimme_cluster_counts_negative/meme
python scripts/pfms_to_meme.py -i gimme_cluster_counts_negative/clustered_motifs.pfm -o gimme_cluster_counts_negative/meme/

mkdir gimme_cluster_counts_negative/tomtom
for x in $(ls gimme_cluster_counts_negative/meme/*) ; do (y=$(basename -s ".meme" $x) ; tomtom -no-ssc -oc . -verbosity 1 -text -min-overlap 5 -mi 1 -dist pearson -evalue -thresh 10.0 $x /srv/www/kundaje/surag/resources/motif_archetypes/pfm_meme_format/motifs.meme.txt  > gimme_cluster_counts_negative/tomtom/$y.vierstra.tomtom.tsv) & done

For each cluster extracted TomTom annotations.
`for x in  $(ls gimme_cluster_counts_negative/tomtom/*) ; do cat $x | head -2 | tail -1 ; done | cut -f1,2,5|  sort -k3g | cut -f1-3 > tfs_initial_negative.tsv`

Filtered the above file, made the following changes and stored to tfs_final_negative.tsv. Overall a little cautious, especially when there are motifs that are activators and aren't seen more than 1 time:
- `negative_Retinalganglioncell_pattern_3`: one appearance of YY1 (pos motif), not as many counts 
- `negative_GABAamacrine_pattern_5`: 2bp, low IC

python scripts/get_representative.py --tfs tfs_final_negative.tsv --pfm pfms_negative/ --cluster-key gimme_cluster_counts_negative/cluster_key.txt  -o tfs_final_w_rep_negative.tsv

rm tfs_final_negative.tsv

Removed 3rd column (p-values) 

Added names manually for each cluster. 

Sotred as MEME file: `python scripts/pfms_negative_to_meme_final_list.py  -i tfs_final_w_rep_negative.tsv -o tfs_final_w_rep_negative.meme`
