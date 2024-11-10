First converted all old retina modisco objects into new modisco-lite objects.

`for x in `ls ~/oak/projects/retina_Howard/models/20220202_bpnet/fold0/modisco/`; do modisco convert -i ~/oak/projects/retina_Howard/models/20220202_bpnet/fold0/modisco/$x/modisco_results_allChroms_counts.hdf5 -o ./modiscolite_objs/${x}_modiscolite_results_allChroms_counts.hdf5 & done `

This makes it easy to run the scripts used for choroid.

Not strictly necessary, but just to reuse old code, remapped motifs to those curated in 20241103_motif_consolidation (separately for pos and neg).

for i in `ls ../20241103_motif_consolidation/pfms/` ; do python ./fetch_tomtom.py  -m ./modiscolite_objs/${i}_modiscolite_results_allChroms_counts.hdf5   -o tomtom/$i.txt -d ../20241103_motif_consolidation/tfs_final_w_rep.meme -n 1 -s 100 -tf 2 & done

for i in `ls ../20241103_motif_consolidation/pfms/` ; do python ./fetch_tomtom_negative.py  -m ./modiscolite_objs/${i}_modiscolite_results_allChroms_counts.hdf5   -o tomtom_negative/$i.txt -d ../20241103_motif_consolidation/tfs_final_w_rep_negative.meme -n 1 -s 100 -tf 2 & done
