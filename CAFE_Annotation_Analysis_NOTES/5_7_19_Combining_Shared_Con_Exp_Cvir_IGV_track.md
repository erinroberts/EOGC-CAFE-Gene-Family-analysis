###5_7_19 Combining exp, con, shared annotations for C. virginica, into an IGV track

## GOALS:
1. Grab those XPs that were annotated from the C. virginica genome, grab their locations in the C. virginica genome
4. Keep the family identifiers that Fabio assigned into the headers of all the proteins

Making 5 separate tracks:
1. All C. virginica expanded, contracted and shared gene families
2. Just the C. virginica expanded
3. Just the C. virginica contracted
4. Just the C. virginica shared contracted
5. Just the C. virginica shared expanded

####################

### FIRST Are the gene families in the shared files the same the in the C.virginica files?

## QUESTION: within the regular_size_shared folder are regular_size_shared_exp and regular_size_shared_con. Are these duplicated in the regular_size_virginica_con and regular_size_virginica_exp

###  Script to do this
  -Gather the transcript IDs from each folder and save to local computer CAFE_Annotation_Analysis.
  $ array1=($(ls *.fa))
  $ for i in ${array1[@]}; do
   echo ${i}| sed 's/_.*//' >> regular_size_shared_exp.txt
  done

  $ scp erin_roberts@bluewaves:/data3/marine_diseases_lab/erin/CAFE_Annotation_Analysis/regular_size_shared/regular_size_shared_exp/regular_size_shared_exp.txt .

  #####  repeated this process for all the folders

  -Download into R. Code saved in Compare_CAFE_Gene_Families.R

  ## Conclusion: NO gene families are shared between the shared con and exp and the virginica con and exp, no overlapping

#####################

# 1. GOAL: Grab those XPs that were annotated from the C. virginica genome, grab their locations in the C. virginica genome

  ## 1. Move the original unmodified data to a folder. for all the shared, regular_size_virginica_exp and regular_size_virginica_con

    ### did the following commands for the regular_size_virginica_con , regular_size_virginica_con, regular_size_shared_con , regular_size_shared_exp
    $ mkdir original_shared_con_fasta
    $ mv *.fa original_shared_con_fasta

    ### also put the blast XML files from those originals into folder

    $ mkdir shared_exp_blast
    $ mv *.xml shared_exp_blast/

    ### Make Interproscan results for the gff3 and tsv files

    ### Make new directory ending in IGV_tracks in each folder

    $ mkdir virginica_exp_IGV_tracks

  ## 2. Copy the files and add the family ID as an identifier to the end of every sequence header line

    $ cd virginica_exp_IGV_tracks/
    $ cp ../original_virginica_exp_fasta/*.fa .


  ## 3. Subset the sequences that have been annotated to virginica and put those in separate folders in each directory

    ### Original Approach: grab the header and the sequences, realized this was unnecessary though

          $ for file in *.fa; do filename=${file%.fa}; awk '/>virginica/{n=NR+1} n>=NR' $file >> $filename.fa.virginica; done

      ### For those folders C. virginica sequences grep the protein ID in the GFF3 reference file and extract the position information on the chromosomes in the two relevant columns

        ###  extract XP and use it in grep search
            $ for file in *.fa.virginica; do
                filename=${grep "^>" $file | sed -e 's/>virginica_\(.*\)_/\1/' | cut -d'_' -f 1,2}
                grep "$filename" $P/ref_C_virginica-3.0_top_level.gff3' >> $filename.gff3.virginica
              done

              $ for file in *.fa.virginica; do
                  filename=${grep "^>" $file | sed -e 's/>virginica_\(.*\)_/\1/'}
                  grep "$filename" $P/ref_C_virginica-3.0_top_level.gff3' >> $filename.gff3.virginica
                done

                # in each folder run the following command to add the family name to the end of the protein name

                  $for file in *.fa; do filename=${file%.fa};   sed -i -e "/>/ s/\$/_$filename/" "$file"; done


## NEW APPROACH FOR CVIR ANNOTATED PROTEINS 
       ### just extract the header lines that have virginica in them from the original FA file wit the virginica, not needing the sequences really

       ### grep only lines with C. virginica

          $ for file in *.fa; do filename=${file%.fa}; grep 'virginica' $file | cut -d'_' -f 2,3 >> $filename.fa.virginica; done

      ###  Grep these protein sequences in GFF3 file

        $ for i in *.fa.virginica; do
            if ! [ -f "$i" ]; then
              continue
            fi
            while IFS= read -r line
              do
                grep "$line" /data3/marine_diseases_lab/shared/GCA_002022765.4_C_virginica-3.0_genomic.fna_index_reads/ref_C_virginica-3.0_top_level.gff3 | cut -f 1,4,5,9 >> $i.positions
              done < "$i"
          done

          ### Add the filename with the fam ID at the end of every line

            $ for f in *.fa.virginica.positions; do filename=${f%.fa.virginica.positions}; sed -i "s/$/\t$filename/" $f; done

            ### had mistake in command previously and need to now delete the last column for only the virginica_con and shared_exp

              $ for f in *.fa.virginica.positions; do awk 'NF{NF--};1' $f > $f.fixed ; done
              $ for f in *.fa.virginica.positions.fixed; do filename=${f%.fa.virginica.positions.fixed}; sed -i "s/$/\t$filename/" $f; done
          ### Concatenate all the files in a folder
              $ cat *.fa.virginica.positions.fixed > virginica_con_Cvir_XP_BED_info.txt
        ### Keep only unique lines from files
                $ sort virginica_con_Cvir_XP_BED_info.txt | uniq -c

                # Remove all of column 5 between ID= and protein_id=
                # this below seems to have a glitch because some of them have a transl attached to the end (fixed)

                $ sed -e 's/\(ID=\).*\(protein_id=\)/\1\2/' virginica_exp_Cvir_XP_BED_info_unique.txt > virginica_exp_Cvir_XP_BED_info_unique_shortened.txt
                $ sed -e 's/\(ID=\).*\(protein_id=\)/\1\2/' virginica_con_Cvir_XP_BED_info_unique.txt > virginica_con_Cvir_XP_BED_info_unique_shortened.txt
                $ sed -e 's/\(ID=\).*\(protein_id=\)/\1\2/' shared_con_Cvir_XP_BED_info_unique.txt > shared_con_Cvir_XP_BED_info_unique_shortened.txt
                $ sed -e 's/\(ID=\).*\(protein_id=\)/\1\2/' shared_exp_Cvir_XP_BED_info_unique.txt > shared_exp_Cvir_XP_BED_info_unique_shortened.txt

        ### Remove ID=protein_id= from each

                $ sed -e "s/ID=protein_id=//g" -i virginica_exp_Cvir_XP_BED_info_unique_shortened.txt
                $ sed -e "s/ID=protein_id=//g" -i virginica_con_Cvir_XP_BED_info_unique_shortened.txt
                $ sed -e "s/ID=protein_id=//g" -i shared_con_Cvir_XP_BED_info_unique_shortened.txt
                $ sed -e "s/ID=protein_id=//g" -i shared_exp_Cvir_XP_BED_info_unique_shortened.txt

        ### some of the files have a weird "transl thing attached", remove that with the following command, re

                $ awk '{split($5,a,/;/);$5=a[1]}1' virginica_exp_Cvir_XP_BED_info_unique_shortened.txt > virginica_exp_Cvir_XP_BED_info_unique_shortened2.txt
                $ awk '{split($5,a,/;/);$5=a[1]}1' virginica_con_Cvir_XP_BED_info_unique_shortened.txt > virginica_con_Cvir_XP_BED_info_unique_shortened2.txt
                $ awk '{split($5,a,/;/);$5=a[1]}1' shared_con_Cvir_XP_BED_info_unique_shortened.txt > shared_con_Cvir_XP_BED_info_unique_shortened2.txt
                $ awk '{split($5,a,/;/);$5=a[1]}1' shared_exp_Cvir_XP_BED_info_unique_shortened.txt > shared_exp_Cvir_XP_BED_info_unique_shortened2.txt

        ### Convert each to tab separated
                $ tr ' ' '\t' <virginica_exp_Cvir_XP_BED_info_unique_shortened2.txt >virginica_exp_Cvir_XP_BED_info_unique_shortened3.txt
                $ tr ' ' '\t' <virginica_con_Cvir_XP_BED_info_unique_shortened2.txt >virginica_con_Cvir_XP_BED_info_unique_shortened3.txt
                $ tr ' ' '\t' <shared_con_Cvir_XP_BED_info_unique_shortened2.txt >shared_con_Cvir_XP_BED_info_unique_shortened3.txt
                $ tr ' ' '\t' <shared_exp_Cvir_XP_BED_info_unique_shortened2.txt >shared_exp_Cvir_XP_BED_info_unique_shortened3.txt

        ### remove the first column

                $ cut -f2,3,4,5,6 virginica_exp_Cvir_XP_BED_info_unique_shortened3.txt > virginica_exp_Cvir_XP_BED_info_unique_shortened4.txt
                $ cut -f2,3,4,5,6 virginica_con_Cvir_XP_BED_info_unique_shortened3.txt > virginica_con_Cvir_XP_BED_info_unique_shortened4.txt
                $ cut -f2,3,4,5,6 shared_con_Cvir_XP_BED_info_unique_shortened3.txt > shared_con_Cvir_XP_BED_info_unique_shortened4.txt
                $ cut -f2,3,4,5,6 shared_exp_Cvir_XP_BED_info_unique_shortened3.txt > shared_exp_Cvir_XP_BED_info_unique_shortened4.txt

        ### concatenate columns 4 and 5 by changing the separator

                $ awk '{print $1"\t"$2"\t"$3"\t"$4"_"$5}' virginica_exp_Cvir_XP_BED_info_unique_shortened4.txt > virginica_exp_Cvir_XP_BED_info_unique_shortened5.txt
                $ awk '{print $1"\t"$2"\t"$3"\t"$4"_"$5}' virginica_con_Cvir_XP_BED_info_unique_shortened4.txt > virginica_con_Cvir_XP_BED_info_unique_shortened5.txt
                $ awk '{print $1"\t"$2"\t"$3"\t"$4"_"$5}' shared_con_Cvir_XP_BED_info_unique_shortened4.txt> shared_con_Cvir_XP_BED_info_unique_shortened5.txt
                $ awk '{print $1"\t"$2"\t"$3"\t"$4"_"$5}' shared_exp_Cvir_XP_BED_info_unique_shortened4.txt> shared_exp_Cvir_XP_BED_info_unique_shortened5.txt

        ### Add track line to top of each as header via nano
                track name ="regular_size_virginica_exp" description="Virginica expanded" color="#00FF00" itemRgb="On" #gffTags
                track name ="regular_size_virginica_con" description="Virginica contracted" color="#FF0000" itemRgb="On" #gffTags
                track name ="regular_size_shared_con" description="Shared contracted" color="#9932CC" itemRgb="On" #gffTags
                track name ="regular_size_shared_exp" description="Shared expanded color="#6495ED" itemRgb="On" #gffTags
                track name ="all_shared_virginica" description="all shared virginica" color="#FF7F50" itemRgb="On" #gffTags

        ### Download data to local /Users/erinroberts/Documents/PhD_Research/Oyster_resequencing/CAFE_Annotation_Analysis_TracksCopy and paste into Excel in appropriate fields

            211:CAFE_Annotation_Analysis_Tracks erinroberts$ pwd
            /Users/erinroberts/Documents/PhD_Research/Oyster_resequencing/CAFE_Annotation_Analysis_Tracks
            211:CAFE_Annotation_Analysis_Tracks erinroberts$ scp erin_roberts@bluewaves:/data3/marine_diseases_lab/erin/CAFE_Annotation_Analysis/regular_size_virginica/regular_size_virginica_exp/virginica_exp_IGV_tracks/virginica_exp_Cvir_XP_BED_info_unique_shortened5.txt .
                                                                                                                                                                                             100%   15MB  20.4MB/s   00:00
            211:CAFE_Annotation_Analysis_Tracks erinroberts$ scp erin_roberts@bluewaves:/data3/marine_diseases_lab/erin/CAFE_Annotation_Analysis/regular_size_virginica/regular_size_virginica_con/virginica_con_IGV_tracks/virginica_con_Cvir_XP_BED_info_unique_shortened5.txt .
                                                                                                                                                                                             100%   24KB   2.4MB/s   00:00
            211:CAFE_Annotation_Analysis_Tracks erinroberts$ scp erin_roberts@bluewaves:/data3/marine_diseases_lab/erin/CAFE_Annotation_Analysis/regular_size_shared/regular_size_shared_con/shared_con_IGV_tracks/shared_con_Cvir_XP_BED_info_unique_shortened5.txt .
                                                                                                                                                                                               100%   10MB  18.7MB/s   00:00
            211:CAFE_Annotation_Analysis_Tracks erinroberts$ scp erin_roberts@bluewaves:/data3/marine_diseases_lab/erin/CAFE_Annotation_Analysis/regular_size_shared/regular_size_shared_exp/shared_exp_IGV_tracks/shared_exp_Cvir_XP_BED_info_unique_shortened5.txt .


        ### To make the track with all the shared and c.virginica, Concatenate all and add the shared track header below

            $ for i in *shortened5.txt ; do cat $i >> combined_con_exp_Cvir_shortened5.txt ;done
            #remove lines beginning with track names so that you just have the new shared track name at the top
              $ sed -i '' '/^track/d' combined_con_exp_Cvir_shortened5.txt
              $ nano combined_con_exp_Cvir_shortened5.txt
                track name ="all_shared_virginica" description="all shared virginica" color="#FF7F50" itemRgb="On"


        ### Change the file ending to be .bed for all the files
            $ mv ALL_combined_con_exp_Cvir_shortened.txt ALL_combined_con_exp_Cvir_shortened.bed

        ### Add #gffTags to the tracks lines header so the Name column will show up in gff format

# THESE ARE YOUR BED FILES!!

#### OPENING THE FILES IN IGV SEPARATELY #####

1. Downloading the genome.fasta file GCF_002022765.2_C_virginica-3.0_genomic.fna

2. Open the BED files separately


#### SCRAP SCRIPT ######
	#####  wasn't used to make final pipeline:...removing XP? $ for file in *.fa.virginica; do filename=${file%.fa.virginica};   sed -i -e "/XP/ s/\$/_$filename/" "$file"; done


#### MISTAKE FOUND WITH SEPARATING TO COLUMN #####

  ##### saw when I loaded all the tracks to github it looked like some of the columns were separated by a space and not by a tab

  #### the files open in IGV perfectly however and seem to not be missing those sections 
