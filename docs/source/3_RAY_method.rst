1. Extract the exons from the GTF 
--------------------

El código original mio empezaba asi:
Merge the gff files of the RNAseq

.. code-block:: bash

  #!/bin/bash
  #SBATCH --dependency=afterok:22343515
  #SBATCH --job-name=gtfR
  #SBATCH --output=%x.%j.out
  #SBATCH --error=%x.%j.err
  #SBATCH --partition=nocona
  #SBATCH --nodes=1
  #SBATCH --ntasks=16
  #SBATCH --mem=128G
  #SBATCH --time=48:00:00
  
  export PATH=/lustre/work/mhoyosro/software/stringtie:${PATH} 
  export PATH=/lustre/work/mhoyosro/software/gffread:${PATH}
  stringtie --merge -p 10 -o merged.gtf mergelist.txt
  ori=/lustre/scratch/mhoyosro/project3/RESULTS
  
  cd /lustre/scratch/mhoyosro/project3/PLANCE/hipposideros
  cp $ori/hipposideros/SRR17418361_MM.gtf RNA_hipposideros.gtf
  
  cd /lustre/scratch/mhoyosro/project3/PLANCE/molossus
  echo "$ori/molossus/SRR18652230_MM.gtf" > molossus_list.txt
  echo "$ori/molossus/SRR18652231_MM.gtf" >> molossus_list.txt
  stringtie --merge -p 10 -o RNA_molossus.gtf molossus_list.txt
  
  
  cd /lustre/scratch/mhoyosro/project3/PLANCE/myotis
  echo "$ori/myotis/SRR18652217_MM.gtf" > myotis_list.txt
  echo "$ori/myotis/SRR18652218_MM.gtf" >> myotis_list.txt
  echo "$ori/myotis/SRR18652219_MM.gtf" >> myotis_list.txt
  stringtie --merge -p 10 -o RNA_myotis.gtf myotis_list.txt
  
  
  cd /lustre/scratch/mhoyosro/project3/PLANCE/phyllostomus
  echo "$ori/phyllostomus/SRR18652128_MM.gtf" > phyllostomus_list.txt
  echo "$ori/phyllostomus/SRR18652129_MM.gtf" >> phyllostomus_list.txt
  echo "$ori/phyllostomus/SRR18652130_MM.gtf" >> phyllostomus_list.txt
  echo "$ori/phyllostomus/SRR18652131_MM.gtf" >> phyllostomus_list.txt
  echo "$ori/phyllostomus/SRR18652132_MM.gtf" >> phyllostomus_list.txt
  stringtie --merge -p 10 -o RNA_phyllostomus.gtf phyllostomus_list.txt
  
  cd /lustre/scratch/mhoyosro/project3/PLANCE/pipistrellus
  cp $ori/pipistrellus/SRR18652220_MM.gtf RNA_pipistrellus.gtf
  
  cd /lustre/scratch/mhoyosro/project3/PLANCE/rhinolophus
  cp $ori/rhinolophus/SRR18652180_MM.gtf RNA_rhinolophus.gtf
  
  cd /lustre/scratch/mhoyosro/project3/PLANCE/rousettus
  echo "$ori/rousettus/SRR18652228_MM.gtf" > rousettus_list.txt
  echo "$ori/rousettus/SRR18652229_MM.gtf" >> rousettus_list.txt
  stringtie --merge -p 10 -o RNA_rousettus.gtf rousettus_list.txt

En este punto tengo los insumos necesarios para reproducir estos resultados con los datos nuevos

.. code-block:: bash

  cd /lustre/scratch/mhoyosro/project3/SCRIPTS2
  nano merge_RNA_gtf_by_species.sh

.. code-block:: bash

  #!/bin/bash
  #SBATCH --job-name=gtfR
  #SBATCH --output=%x.%j.out
  #SBATCH --error=%x.%j.err
  #SBATCH --partition=nocona
  #SBATCH --nodes=1
  #SBATCH --ntasks=16
  #SBATCH --mem=128G
  #SBATCH --time=48:00:00
  
  set -euo pipefail
  
  export PATH=/lustre/work/mhoyosro/software/stringtie:${PATH}
  
  ori=/lustre/scratch/mhoyosro/project3/RESULTS_RNA_MAPPING
  out=/lustre/scratch/mhoyosro/project3/ANALISIS_2026
  
  mkdir -p "$out"
  
  for species_dir in "$ori"/*
  do
      species=$(basename "$species_dir")
      outdir="$out/$species"
      mkdir -p "$outdir"
  
      list="$outdir/${species}_list.txt"
      find "$species_dir" -maxdepth 1 -name "*_MM.gtf" | sort > "$list"
  
      n=$(wc -l < "$list")
  
      echo "======================================"
      echo "SPECIES: $species"
      echo "N GTF: $n"
      echo "LIST: $list"
  
      if [[ "$n" -eq 0 ]]; then
          echo "ERROR: no GTF found for $species"
          exit 1
      elif [[ "$n" -eq 1 ]]; then
          gtf=$(cat "$list")
          cp "$gtf" "$outdir/RNA_${species}.gtf"
          echo "Copied single GTF to $outdir/RNA_${species}.gtf"
      else
          stringtie --merge -p 10 -o "$outdir/RNA_${species}.gtf" "$list"
          echo "Merged GTFs to $outdir/RNA_${species}.gtf"
      fi
  done
  
  echo "DONE"

.. code-block:: bash

  chmod +x merge_RNA_gtf_by_species.sh
  sbatch merge_RNA_gtf_by_species.sh
