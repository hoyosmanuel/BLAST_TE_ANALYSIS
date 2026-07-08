1. Extract the exons from the GTF 
----------------------------------

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
  nano gtf_analisis_2026_receta_vieja.sh

.. code-block:: bash

  #!/bin/bash
  #SBATCH --job-name=gtfRold
  #SBATCH --output=%x.%j.out
  #SBATCH --error=%x.%j.err
  #SBATCH --partition=nocona
  #SBATCH --nodes=1
  #SBATCH --ntasks=16
  #SBATCH --mem=128G
  #SBATCH --time=48:00:00
  
  set -euo pipefail
  
  export PATH=/lustre/work/mhoyosro/software/stringtie:${PATH}
  export PATH=/lustre/work/mhoyosro/software/gffread:${PATH}
  
  ori=/lustre/scratch/mhoyosro/project3/RESULTS_RNA_MAPPING
  out=/lustre/scratch/mhoyosro/project3/ANALISIS_2026
  
  mkdir -p "$out"
  
  enter_species () {
      species="$1"
      mkdir -p "$out/$species"
      cd "$out/$species" || exit 1
  }
  
  enter_species Eonycteris_spelaea
  cp "$ori/Eonycteris_spelaea/SRR6951022_MM.gtf" RNA_Eonycteris_spelaea.gtf
  
  enter_species Miniopterus_schreibersii
  cp "$ori/Miniopterus_schreibersii/ERR13757857_MM.gtf" RNA_Miniopterus_schreibersii.gtf
  
  enter_species Molossus_molossus
  cat > Molossus_molossus_list.txt << EOF
  $ori/Molossus_molossus/SRR18652230_MM.gtf
  $ori/Molossus_molossus/SRR18652231_MM.gtf
  EOF
  stringtie --merge -p 10 -o RNA_Molossus_molossus.gtf Molossus_molossus_list.txt
  
  enter_species Myotis_daubentonii
  cat > Myotis_daubentonii_list.txt << EOF
  $ori/Myotis_daubentonii/ERR13757850_MM.gtf
  $ori/Myotis_daubentonii/ERR13757851_MM.gtf
  $ori/Myotis_daubentonii/ERR13757852_MM.gtf
  $ori/Myotis_daubentonii/ERR13757853_MM.gtf
  $ori/Myotis_daubentonii/ERR13757854_MM.gtf
  EOF
  stringtie --merge -p 10 -o RNA_Myotis_daubentonii.gtf Myotis_daubentonii_list.txt
  
  enter_species Myotis_myotis
  cat > Myotis_myotis_list.txt << EOF
  $ori/Myotis_myotis/SRR18652217_MM.gtf
  $ori/Myotis_myotis/SRR18652218_MM.gtf
  $ori/Myotis_myotis/SRR18652219_MM.gtf
  EOF
  stringtie --merge -p 10 -o RNA_Myotis_myotis.gtf Myotis_myotis_list.txt
  
  enter_species Myotis_mystacinus
  cp "$ori/Myotis_mystacinus/ERR13757864_MM.gtf" RNA_Myotis_mystacinus.gtf
  
  enter_species Phyllostomus_discolor
  cat > Phyllostomus_discolor_list.txt << EOF
  $ori/Phyllostomus_discolor/SRR18652128_MM.gtf
  $ori/Phyllostomus_discolor/SRR18652129_MM.gtf
  $ori/Phyllostomus_discolor/SRR18652130_MM.gtf
  $ori/Phyllostomus_discolor/SRR18652131_MM.gtf
  $ori/Phyllostomus_discolor/SRR18652132_MM.gtf
  EOF
  stringtie --merge -p 10 -o RNA_Phyllostomus_discolor.gtf Phyllostomus_discolor_list.txt
  
  enter_species Pipistrellus_kuhlii
  cp "$ori/Pipistrellus_kuhlii/SRR18652220_MM.gtf" RNA_Pipistrellus_kuhlii.gtf
  
  enter_species Pipistrellus_pygmaeus
  cp "$ori/Pipistrellus_pygmaeus/ERR13757875_MM.gtf" RNA_Pipistrellus_pygmaeus.gtf
  
  enter_species Plecotus_auritus
  cat > Plecotus_auritus_list.txt << EOF
  $ori/Plecotus_auritus/ERR13757870_MM.gtf
  $ori/Plecotus_auritus/ERR13757871_MM.gtf
  $ori/Plecotus_auritus/ERR13757872_MM.gtf
  $ori/Plecotus_auritus/ERR13757873_MM.gtf
  $ori/Plecotus_auritus/ERR13757874_MM.gtf
  EOF
  stringtie --merge -p 10 -o RNA_Plecotus_auritus.gtf Plecotus_auritus_list.txt
  
  enter_species Rhinolophus_ferrumequinum
  cp "$ori/Rhinolophus_ferrumequinum/SRR18652180_MM.gtf" RNA_Rhinolophus_ferrumequinum.gtf
  
  enter_species Rhinolophus_hipposideros
  cat > Rhinolophus_hipposideros_list.txt << EOF
  $ori/Rhinolophus_hipposideros/ERR13757944_MM.gtf
  $ori/Rhinolophus_hipposideros/ERR13757945_MM.gtf
  $ori/Rhinolophus_hipposideros/ERR13757946_MM.gtf
  $ori/Rhinolophus_hipposideros/ERR13757947_MM.gtf
  $ori/Rhinolophus_hipposideros/ERR13757948_MM.gtf
  EOF
  stringtie --merge -p 10 -o RNA_Rhinolophus_hipposideros.gtf Rhinolophus_hipposideros_list.txt
  
  enter_species Rhinolophus_sinicus
  cp "$ori/Rhinolophus_sinicus/SRR17418361_MM.gtf" RNA_Rhinolophus_sinicus.gtf
  
  enter_species Rousettus_aegyptiacus
  cat > Rousettus_aegyptiacus_list.txt << EOF
  $ori/Rousettus_aegyptiacus/SRR18652228_MM.gtf
  $ori/Rousettus_aegyptiacus/SRR18652229_MM.gtf
  EOF
  stringtie --merge -p 10 -o RNA_Rousettus_aegyptiacus.gtf Rousettus_aegyptiacus_list.txt
  
  enter_species Vespertilio_murinus
  cp "$ori/Vespertilio_murinus/ERR13757846_MM.gtf" RNA_Vespertilio_murinus.gtf
  
  echo "DONE"

2. Extract the RNA_species_exones.bed
--------------------------------------

el código original era este

.. code-block:: bash

  export PATH=/lustre/work/mhoyosro/software/gffread:${PATH}
  . /home/mhoyosro/conda/etc/profile.d/conda.sh
  conda activate alineador
  
  cd /lustre/scratch/mhoyosro/project3/PLANCE/hipposideros
  awk '$3=="exon"' RNA_hipposideros.gtf > hipposideros_exones_only.gtf
  gff2bed < hipposideros_exones_only.gtf  > RNA_hipposideros_exones.bed
  
  cd /lustre/scratch/mhoyosro/project3/PLANCE/molossus
  awk '$3=="exon"' RNA_molossus.gtf > molossus_exones_only.gtf
  gff2bed < molossus_exones_only.gtf  > RNA_molossus_exones.bed
  
  cd /lustre/scratch/mhoyosro/project3/PLANCE/myotis
  awk '$3=="exon"' RNA_myotis.gtf > myotis_exones_only.gtf
  gff2bed < myotis_exones_only.gtf > RNA_myotis_exones.bed
  
  cd /lustre/scratch/mhoyosro/project3/PLANCE/phyllostomus
  awk '$3=="exon"' RNA_phyllostomus.gtf > phyllostomus_exones_only.gtf
  gff2bed < phyllostomus_exones_only.gtf  > RNA_phyllostomus_exones.bed
  
  cd /lustre/scratch/mhoyosro/project3/PLANCE/pipistrellus
  awk '$3=="exon"' RNA_pipistrellus.gtf > pipistrellus_exones_only.gtf
  gff2bed < pipistrellus_exones_only.gtf  > RNA_pipistrellus_exones.bed
  
  cd /lustre/scratch/mhoyosro/project3/PLANCE/rhinolophus
  awk '$3=="exon"' RNA_rhinolophus.gtf > rhinolophus_exones_only.gtf
  gff2bed < rhinolophus_exones_only.gtf  > RNA_rhinolophus_exones.bed
  
  cd /lustre/scratch/mhoyosro/project3/PLANCE/rousettus
  awk '$3=="exon"' RNA_rousettus.gtf > rousettus_exones_only.gtf
  gff2bed < rousettus_exones_only.gtf  > RNA_rousettus_exones.bed

Y el código actualizado es este:

.. code-block:: bash

  cd /lustre/scratch/mhoyosro/project3/SCRIPTS2
  nano RNA_gtf_to_exon_bed_all.sh

.. code-block:: bash

  #!/bin/bash
  #SBATCH --job-name=gtf2bed
  #SBATCH --output=%x.%j.out
  #SBATCH --error=%x.%j.err
  #SBATCH --partition=nocona
  #SBATCH --nodes=1
  #SBATCH --ntasks=1
  #SBATCH --mem=16G
  #SBATCH --time=12:00:00
  
  set -euo pipefail
  
  export PATH=/lustre/work/mhoyosro/software/gffread:${PATH}
  
  . /home/mhoyosro/conda/etc/profile.d/conda.sh
  conda activate alineador
  
  base=/lustre/scratch/mhoyosro/project3/ANALISIS_2026
  
  find "$base" -mindepth 2 -maxdepth 2 -name "RNA_*.gtf" | sort | while read gtf
  do
      dir=$(dirname "$gtf")
      species=$(basename "$dir")
      basefile=$(basename "$gtf" .gtf)
  
      exon_gtf="$dir/${species}_exones_only.gtf"
      bed="$dir/${basefile}_exones.bed"
  
      echo "======================================"
      echo "SPECIES: $species"
      echo "GTF: $gtf"
      echo "EXON_GTF: $exon_gtf"
      echo "BED: $bed"
  
      awk '$3=="exon"' "$gtf" > "$exon_gtf"
  
      if [[ ! -s "$exon_gtf" ]]; then
          echo "ERROR: exon GTF vacío: $exon_gtf"
          exit 1
      fi
  
      gff2bed < "$exon_gtf" > "$bed"
  
      if [[ ! -s "$bed" ]]; then
          echo "ERROR: BED vacío: $bed"
          exit 1
      fi
  
      echo "DONE"
  done
  
  echo "ALL DONE"

.. code-block:: bash

  chmod +x RNA_gtf_to_exon_bed_all.sh
  sbatch RNA_gtf_to_exon_bed_all.sh

3. Crear el FINAL_transcriptome.bed
--------------------------------------

Aquí vamos a combinar  dos fuentes de exones por especie: TOGA exons + RNA/StringTie exons → FINAL_transcriptome.bed

Pero para lo nuevo hay que hacerlo para 15 especies, usando:

``ANNOTATIONS/*_TOGA_expandedexons.bed``
``ANALISIS_2026/<species>/RNA_<species>_exones.bed``

.. code-block:: bash

  cd /lustre/scratch/mhoyosro/project3/SCRIPTS2
  nano make_FINAL_transcriptome_all.sh

.. code-block:: bash

  #!/bin/bash
  #SBATCH --job-name=finalBED
  #SBATCH --output=%x.%j.out
  #SBATCH --error=%x.%j.err
  #SBATCH --partition=nocona
  #SBATCH --nodes=1
  #SBATCH --ntasks=1
  #SBATCH --mem=16G
  #SBATCH --time=04:00:00
  
  set -euo pipefail
  
  ann=/lustre/scratch/mhoyosro/project3/ANNOTATIONS
  out=/lustre/scratch/mhoyosro/project3/ANALISIS_2026
  
  while read -r species toga prefix
  do
      dir="$out/$species"
  
      toga_bed="$ann/${toga}_expandedexons.bed"
      rna_bed="$dir/RNA_${species}_exones.bed"
  
      combined="$dir/${prefix}_combined_transcriptome.bed"
      final="$dir/${prefix}_FINAL_transcriptome.bed"
  
      echo "======================================"
      echo "$species"
      echo "TOGA: $toga_bed"
      echo "RNA:  $rna_bed"
  
      if [[ ! -s "$toga_bed" ]]; then
          echo "ERROR: falta TOGA $toga_bed"
          exit 1
      fi
  
      if [[ ! -s "$rna_bed" ]]; then
          echo "ERROR: falta RNA $rna_bed"
          exit 1
      fi
  
      cat "$toga_bed" "$rna_bed" > "$combined"
  
      sort -k1,1 -k4,4n "$combined" > "$final"
  
      echo "COMBINED:"
      wc -l "$combined"
  
      echo "FINAL:"
      wc -l "$final"
  
  done << EOF
  Eonycteris_spelaea mEonSpe1.2.hap1.TOGA eSpe
  Miniopterus_schreibersii mMinSch1.1.hap1.TOGA mSch
  Molossus_molossus mMolMol1.2.pri.TOGA mMol
  Myotis_daubentonii mMyoDau2.1.pri.TOGA mDau
  Myotis_myotis mMyoMyo1.6.pri.TOGA mMyo
  Myotis_mystacinus mMyoMys1.1.hap1.TOGA mMys
  Phyllostomus_discolor mPhyDis1.3.pri.TOGA pDis
  Pipistrellus_kuhlii mPipKuh1.2.pri.TOGA pKuh
  Plecotus_auritus mPleAur1.1.pri.TOGA pAur
  Rhinolophus_ferrumequinum mRhiFer1.5.pri.TOGA rFer
  Rhinolophus_hipposideros mRhiHip1.1.hap1.TOGA rHip
  Rhinolophus_sinicus mRhiSin3.1.pri.TOGA rSin
  Rousettus_aegyptiacus mRouAeg1.4.pri.TOGA rAeg
  Vespertilio_murinus mVesMur1.1.pri.TOGA vMur
  EOF
  
  echo "DONE"

.. code-block:: bash

  bash make_FINAL_transcriptome_all.sh




4. LIMPIAR
--------------------------------------

ahora hay que:

| → limpiar nombres raros
| → convertirlo a BED6 compatible
| → extraer secuencias FASTA del genoma
| → producir un transcriptoma FASTA por especie

.. code-block:: bash

  cd /lustre/scratch/mhoyosro/project3/SCRIPTS2
  nano make_BLAST2_transcriptome_fastas_2026.sh

.. code-block:: bash

  #!/bin/bash
  #SBATCH --job-name=blast2fa
  #SBATCH --output=%x.%j.out
  #SBATCH --error=%x.%j.err
  #SBATCH --partition=nocona
  #SBATCH --nodes=1
  #SBATCH --ntasks=1
  #SBATCH --mem=32G
  #SBATCH --time=12:00:00
  
  set -euo pipefail
  
  . /home/mhoyosro/conda/etc/profile.d/conda.sh
  conda activate alineador
  
  base=/lustre/scratch/mhoyosro/project3
  finaldir=$base/ANALISIS_2026
  blastdir=$base/BLAST2_2026
  refdir=$base/REF_GENOMES
  
  mkdir -p "$blastdir"
  
  while read -r species prefix ref
  do
      echo "======================================"
      echo "$species"
  
      in_bed="$finaldir/$species/${prefix}_FINAL_transcriptome.bed"
      outdir="$blastdir/$species"
      copied_bed="$outdir/${prefix}_FINAL_transcriptome.bed"
      processed_bed="$outdir/${prefix}_TRANSCRIPTOME_PROCESSED.bed"
      sorted_bed="$outdir/${prefix}_sorted.bed"
      fasta="$outdir/${prefix}_TRANSCRIPTOME.fasta"
      genome="$refdir/$ref"
  
      mkdir -p "$outdir"
  
      echo "INPUT BED: $in_bed"
      echo "GENOME:    $genome"
      echo "OUTDIR:    $outdir"
  
      if [[ ! -s "$in_bed" ]]; then
          echo "ERROR: falta FINAL bed: $in_bed"
          exit 1
      fi
  
      if [[ ! -s "$genome" ]]; then
          echo "ERROR: falta genoma: $genome"
          exit 1
      fi
  
      cp "$in_bed" "$copied_bed"
  
      # Pseudocódigo:
      # 1. Limpiar espacios, comillas y punto-y-coma en los nombres.
      # 2. Concatenar columnas 10 en adelante para construir un nombre trazable.
      # 3. Crear BED6:
      #    col1 = cromosoma
      #    col2 = start
      #    col3 = end
      #    col4 = nombre concatenado
      #    col5 = score
      #    col6 = strand
      sed 's/ /_/g; s/"//g; s/;/_/g' "$copied_bed" | \
      awk 'BEGIN{FS=OFS="\t"} {
          concatenated = $10
          for(i=11; i<=NF; i++) {
              if ($i != "") concatenated = concatenated "__" $i
          }
          print $1, $2, $3, concatenated, $5, $6
      }' > "$processed_bed"
  
      if [[ ! -s "$processed_bed" ]]; then
          echo "ERROR: BED procesado vacío: $processed_bed"
          exit 1
      fi
  
      sort -k1,1 -k2,2n "$processed_bed" > "$sorted_bed"
  
      bedtools getfasta \
          -fi "$genome" \
          -bed "$sorted_bed" \
          -fo "$fasta" \
          -name \
          -s
  
      rm -f "$sorted_bed"
  
      echo "PROCESSED BED:"
      wc -l "$processed_bed"
  
      echo "FASTA:"
      grep -c "^>" "$fasta"
  
  done << EOF
  Eonycteris_spelaea eSpe mEonSpe1.2.hap1.fa
  Miniopterus_schreibersii mSch mMinSch1.1.hap1.fa
  Molossus_molossus mMol mMolMol1.2.pri.fa
  Myotis_daubentonii mDau mMyoDau2.1.pri.fa
  Myotis_myotis mMyo mMyoMyo1.6.pri.fa
  Myotis_mystacinus mMys mMyoMys1.1.hap1.fa
  Phyllostomus_discolor pDis mPhyDis1.3.pri.fa
  Pipistrellus_kuhlii pKuh mPipKuh1.2.pri.fa
  Plecotus_auritus pAur mPleAur1.1.pri.fa
  Rhinolophus_ferrumequinum rFer mRhiFer1.5.pri.fa
  Rhinolophus_hipposideros rHip mRhiHip1.1.hap1.fa
  Rhinolophus_sinicus rSin mRhiSin3.1.pri.fa
  Rousettus_aegyptiacus rAeg mRouAeg1.4.pri.fa
  Vespertilio_murinus vMur mVesMur1.1.pri.fa
  EOF
  
  echo "DONE"

.. code-block:: bash

  chmod +x make_BLAST2_transcriptome_fastas_2026.sh
  bash make_BLAST2_transcriptome_fastas_2026.sh




5. Crear la base de datos RepeatPeps
--------------------------------------

.. code-block:: bash

  nano database.sh

.. code-block:: bash

  #!/bin/bash
  #SBATCH --job-name=mkblastdb
  #SBATCH --output=%x.%j.out
  #SBATCH --error=%x.%j.err
  #SBATCH --partition=nocona
  #SBATCH --nodes=1
  #SBATCH --ntasks=1
  #SBATCH --time=01:00:00
  
  set -euo pipefail
  
  . /home/mhoyosro/conda/etc/profile.d/conda.sh
  conda activate blast
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2_2026
  
  lib=/lustre/scratch/mhoyosro/project3/RepeatPeps.lib
  
  makeblastdb \
      -in "$lib" \
      -dbtype prot \
      -out RepeatPeps_db


.. code-block:: bash

  sbatch database.sh




6. Hacer el BLAST
--------------------------------------

.. code-block:: bash

  cd /lustre/scratch/mhoyosro/project3/SCRIPTS2
  nano blastx_transcriptomes_2026_array.sh

.. code-block:: bash

  #!/bin/bash
  #SBATCH --job-name=blastxTE
  #SBATCH --output=%x.%A_%a.out
  #SBATCH --error=%x.%A_%a.err
  #SBATCH --partition=nocona
  #SBATCH --array=0-13
  #SBATCH --nodes=1
  #SBATCH --ntasks=36
  #SBATCH --mem=128G
  #SBATCH --time=48:00:00
  
  set -euo pipefail
  
  . /home/mhoyosro/conda/etc/profile.d/conda.sh
  conda activate blast
  
  base=/lustre/scratch/mhoyosro/project3/BLAST2_2026
  db=/lustre/scratch/mhoyosro/project3/BLAST2_2026/RepeatPeps_db
  
  SPECIES=(
  "Eonycteris_spelaea"
  "Miniopterus_schreibersii"
  "Molossus_molossus"
  "Myotis_daubentonii"
  "Myotis_myotis"
  "Myotis_mystacinus"
  "Phyllostomus_discolor"
  "Pipistrellus_kuhlii"
  "Plecotus_auritus"
  "Rhinolophus_ferrumequinum"
  "Rhinolophus_hipposideros"
  "Rhinolophus_sinicus"
  "Rousettus_aegyptiacus"
  "Vespertilio_murinus"
  )
  
  PREFIX=(
  "eSpe"
  "mSch"
  "mMol"
  "mDau"
  "mMyo"
  "mMys"
  "pDis"
  "pKuh"
  "pAur"
  "rFer"
  "rHip"
  "rSin"
  "rAeg"
  "vMur"
  )
  
  species=${SPECIES[$SLURM_ARRAY_TASK_ID]}
  prefix=${PREFIX[$SLURM_ARRAY_TASK_ID]}
  
  dir="$base/$species"
  query="$dir/${prefix}_TRANSCRIPTOME.fasta"
  out="$dir/${prefix}_vs_RepeatPeps.raw.tsv"
  
  echo "======================================"
  echo "TASK:    $SLURM_ARRAY_TASK_ID"
  echo "SPECIES: $species"
  echo "PREFIX:  $prefix"
  echo "QUERY:   $query"
  echo "DB:      $db"
  echo "OUT:     $out"
  
  if [[ ! -s "$query" ]]; then
      echo "ERROR: falta query fasta: $query"
      exit 1
  fi
  
  if [[ ! -s "${db}.pin" ]]; then
      echo "ERROR: falta base BLAST: ${db}.pin"
      exit 1
  fi
  
  blastx \
    -query "$query" \
    -db "$db" \
    -evalue 1e-50 \
    -outfmt "6 qseqid sseqid pident length qstart qend sstart send evalue bitscore qlen slen qcovs qcovhsp" \
    -num_threads 36 \
    -max_target_seqs 10 \
    -out "$out"
  
  echo "DONE"
  wc -l "$out"


.. code-block:: bash

  chmod +x blastx_transcriptomes_2026_array.sh
  sbatch ./blastx_transcriptomes_2026_array.sh


7. Arreglar el output de BLASTX
--------------------------------------

Tenemos que arreglar el output crudo de BLASTX para convertirlo en una tabla legible, con coordenadas del exon, strand, familia TE, tipo TE y métricas BLAST.
escencialmente para cada archivo *_vs_RepeatPeps.raw.tsv, tenemos que

1. Tomar el qseqid generado por bedtools.
   Ejemplo:
   exon_name::scaffold:start-end(strand)

2. Cambiar "::" por "__"
   Propósito:
   evitar que el separador "::" interfiera cuando luego partimos por ":".

3. Cambiar ":" por tabulaciones.
   Propósito:
   separar:
   - nombre del exon
   - coordenadas genómicas

4. Separar el bloque start-end(strand).
   Ejemplo:
   1000-1200(+)
   pasa a:
   1000    1200    +

5. Cambiar "#" por tabulaciones.
   Propósito:
   separar información del subject BLAST:
   - TE_FAMILY
   - TE_TYPE

6. Agregar header.
   Resultado final:
   una tabla FINAL.tsv con 18 columnas.

.. code-block:: bash

  cd /lustre/scratch/mhoyosro/project3/SCRIPTS2
  nano polish_blastx_outputs_2026.sh

.. code-block:: bash

  #!/bin/bash
  #SBATCH --job-name=polishBLAST
  #SBATCH --output=%x.%j.out
  #SBATCH --error=%x.%j.err
  #SBATCH --partition=nocona
  #SBATCH --nodes=1
  #SBATCH --ntasks=1
  #SBATCH --mem=16G
  #SBATCH --time=04:00:00
  
  set -euo pipefail
  
  base=/lustre/scratch/mhoyosro/project3/BLAST2_2026
  
  header="qseqid_EXON_NAME	qseqid_start	qseqid_end	qseqid_sense	TE_FAMILY	TE_TYPE	pident	length	qstart	qend	sstart	send	evalue	bitscore	qlen	slen	qcovs	qcovhsp"
  
  while read -r species prefix
  do
      dir="$base/$species"
      raw="$dir/${prefix}_vs_RepeatPeps.raw.tsv"
  
      w1="$dir/${prefix}_vs_RepeatPeps.working1.tsv"
      w2="$dir/${prefix}_vs_RepeatPeps.working2.tsv"
      w3="$dir/${prefix}_vs_RepeatPeps.working3.tsv"
      w4="$dir/${prefix}_vs_RepeatPeps.working4.tsv"
      final="$dir/${prefix}_vs_RepeatPeps.FINAL.tsv"
  
      echo "======================================"
      echo "SPECIES: $species"
      echo "RAW:     $raw"
      echo "FINAL:   $final"
  
      if [[ ! -s "$raw" ]]; then
          echo "ERROR: raw BLAST file missing or empty: $raw"
          exit 1
      fi
  
      # Paso 1:
      # Cambiar "::" por "__".
      # Esto protege el separador doble antes de partir por ":".
      sed 's/::/__/g' "$raw" > "$w1"
  
      # Paso 2:
      # Cambiar ":" por tab.
      # Esto separa nombre/coordenadas dentro del qseqid.
      sed 's/:/\t/g' "$w1" > "$w2"
  
      # Paso 3:
      # Separar start-end(strand) en tres columnas:
      # start, end, strand.
      sed -E 's/^([^\t]*)\t([0-9]+)-([0-9]+)\(([+-\.])\)/\1\t\2\t\3\t\4/' "$w2" > "$w3"
  
      # Paso 4:
      # Separar "#" por tab.
      # Esto separa TE_FAMILY y TE_TYPE desde sseqid.
      sed 's/#/\t/g' "$w3" > "$w4"
  
      # Paso 5:
      # Agregar header.
      {
          echo -e "$header"
          cat "$w4"
      } > "$final"
  
      echo "RAW lines:"
      wc -l "$raw"
  
      echo "FINAL lines:"
      wc -l "$final"
  
      echo "FINAL columns:"
      awk 'NR==1{print NF; next} NR==2{print NF; exit}' "$final"
  
  done << EOF
  Eonycteris_spelaea eSpe
  Miniopterus_schreibersii mSch
  Molossus_molossus mMol
  Myotis_daubentonii mDau
  Myotis_myotis mMyo
  Myotis_mystacinus mMys
  Phyllostomus_discolor pDis
  Pipistrellus_kuhlii pKuh
  Plecotus_auritus pAur
  Rhinolophus_ferrumequinum rFer
  Rhinolophus_hipposideros rHip
  Rhinolophus_sinicus rSin
  Rousettus_aegyptiacus rAeg
  Vespertilio_murinus vMur
  EOF
  
  echo "DONE"

.. code-block:: bash

  chmod +x polish_blastx_outputs_2026.sh
  bash polish_blastx_outputs_2026.sh



8. Ordenar los archivos
--------------------------------------
Aquí se ordena el alineamiento BLASTX de un exon contra una proteína de TEs.
Pero como cada exon puede tener muchos hits.

Por ejemplo:

| Exon A  ---> LINE_RT
| Exon A  ---> Gypsy
| Exon A  ---> Mariner
| Exon A  ---> hAT

Todos aparecen mezclados.

Entonces hay que ordenar con 4 criterios

| 1. Agrupando todos los hits que pertenecen al mismo intervalo genómico.
| chr
| 1000-1100 -> todos juntos

| 2. Dentro de esa posición agrupar hits por exon.
| Exon1 y todos sus hits
| Exon2 y todos sus hits

| 3. Dentro de un mismo exon poner primero  hits con mayor identidad.
| Por ejemplo:
| 97%
| 94%
| 91%
| 89%

| 4. Si dos hits tienen la misma identidad, usar bitscore para desempatar.


.. code-block:: bash

  cd /lustre/scratch/mhoyosro/project3/SCRIPTS2
  nano orden.sh 

.. code-block:: bash

  #!/bin/bash
  #SBATCH --job-name=sortBLAST
  #SBATCH --output=%x.%j.out
  #SBATCH --error=%x.%j.err
  #SBATCH --partition=nocona
  #SBATCH --nodes=1
  #SBATCH --ntasks=1
  #SBATCH --cpus-per-task=4
  #SBATCH --mem=8G
  #SBATCH --time=02:00:00
  
  set -euo pipefail
  
  . /home/mhoyosro/conda/etc/profile.d/conda.sh
  conda activate biopython
  
  python << 'EOF'
  import pandas as pd
  from pathlib import Path
  
  base = Path("/lustre/scratch/mhoyosro/project3/BLAST2_2026")
  
  species = {
      "Eonycteris_spelaea": "eSpe",
      "Miniopterus_schreibersii": "mSch",
      "Molossus_molossus": "mMol",
      "Myotis_daubentonii": "mDau",
      "Myotis_myotis": "mMyo",
      "Myotis_mystacinus": "mMys",
      "Phyllostomus_discolor": "pDis",
      "Pipistrellus_kuhlii": "pKuh",
      "Plecotus_auritus": "pAur",
      "Rhinolophus_ferrumequinum": "rFer",
      "Rhinolophus_hipposideros": "rHip",
      "Rhinolophus_sinicus": "rSin",
      "Rousettus_aegyptiacus": "rAeg",
      "Vespertilio_murinus": "vMur"
  }
  
  for sp, prefix in species.items():
  
      infile = base / sp / f"{prefix}_vs_RepeatPeps.FINAL.tsv"
      outfile = base / sp / f"{prefix}_vs_RepeatPeps.FINAL_sorted.tsv"
  
      print("=" * 60)
      print(sp)
      print("Input :", infile)
      print("Output:", outfile)
  
      df = pd.read_csv(infile, sep="\t")
  
      df_sorted = df.sort_values(
          by=[
              "qseqid_start",
              "qseqid_end",
              "qseqid_EXON_NAME",
              "pident",
              "bitscore"
          ],
          ascending=[
              True,
              True,
              True,
              False,   # mayor identidad primero
              False    # mayor bitscore primero
          ]
      )
  
      df_sorted.to_csv(outfile, sep="\t", index=False)
  
      print("Filas:", len(df_sorted))
  
  print("\nDONE")
  EOF


.. code-block:: bash

  sbatch orden.sh 


Corregir el header

.. code-block:: bash

  cd /lustre/scratch/mhoyosro/project3/BLAST2_2026
  
  header=$'qseqid_EXON_NAME\tqseqid_start\tqseqid_end\tqseqid_sense\tTE_FAMILY\tTE_TYPE\tpident\tlength\tqstart\tqend\tsstart\tsend\tevalue\tbitscore\tqlen\tslen\tqcovs\tqcovhsp'
  
  find . -name "*_vs_RepeatPeps.FINAL.tsv" | while read f
  do
      echo "Corrigiendo header: $f"
      tmp="${f}.tmp"
      {
          printf "%s\n" "$header"
          tail -n +2 "$f"
      } > "$tmp"
      mv "$tmp" "$f"
  done




9. Quedarse con un solo hit de BLAST:
--------------------------------------
En este punto contamos con la siguiente información:

| qseqid_start ↑
| qseqid_end ↑
| qseqid_EXON_NAME ↑
| pident ↓
| bitscore ↓

entonces la función drop_duplicates(... keep="first") conserva, para cada exón, el hit con mejor pident y, si hay empate, mejor bitscore.
la lógica es:

Para cada archivo FINAL_ordenado.tsv: leer tabla, para cada grupo definido por: · qseqid_EXON_NAME ·qseqid_start ·qseqid_end: conservar solo la primera fila
como el archivo ya está ordenado: la primera fila es el mejor hit BLASTX de ese exón; guardar como: *_vs_RepeatPeps.FINAL_FILTRO_1.tsv

.. code-block:: bash

  cd /lustre/scratch/mhoyosro/project3/SCRIPTS2
  nano filtro1_drop_duplicates_2026.py

.. code-block:: bash

  import pandas as pd
  from pathlib import Path
  
  base = Path("/lustre/scratch/mhoyosro/project3/BLAST2_2026")
  
  files = {
      "Eonycteris_spelaea": "eSpe",
      "Miniopterus_schreibersii": "mSch",
      "Molossus_molossus": "mMol",
      "Myotis_daubentonii": "mDau",
      "Myotis_myotis": "mMyo",
      "Myotis_mystacinus": "mMys",
      "Phyllostomus_discolor": "pDis",
      "Pipistrellus_kuhlii": "pKuh",
      "Plecotus_auritus": "pAur",
      "Rhinolophus_ferrumequinum": "rFer",
      "Rhinolophus_hipposideros": "rHip",
      "Rhinolophus_sinicus": "rSin",
      "Rousettus_aegyptiacus": "rAeg",
      "Vespertilio_murinus": "vMur",
  }
  
  for species, prefix in files.items():
      infile = base / species / f"{prefix}_vs_RepeatPeps.FINAL_ordenado.tsv"
      outfile = base / species / f"{prefix}_vs_RepeatPeps.FINAL_FILTRO_1.tsv"
  
      df = pd.read_csv(infile, sep="\t")
  
      before = len(df)
  
      df_unique = df.drop_duplicates(
          subset=["qseqid_EXON_NAME", "qseqid_start", "qseqid_end"],
          keep="first"
      )
  
      after = len(df_unique)
  
      df_unique.to_csv(outfile, sep="\t", index=False)
  
      print(f"{species}: {before} -> {after} rows | saved: {outfile}")
  
  print("DONE")


.. code-block:: bash

  python3 filtro1_drop_duplicates_2026.py


10. Mejorar columnas 
----------------------------------

En este momento hay columnas muy largas, algo como asi:
``gene_id_MSTRG.49__transcript_id_MSTRG.49.1__exon_number_2____scaffold_m19_p_1``
o como asi:
``ENST00000302182.8__UBB__217_exon1__scaffold_m19_p_29``
Entonces ahora vamos a partir las columnas en 3 partes usando __:

.. code-block:: bash

  cd /lustre/scratch/mhoyosro/project3/SCRIPTS2
  nano filtro2_split_exon_name_2026.py

.. code-block:: bash

  import pandas as pd
  from pathlib import Path
  
  base = Path("/lustre/scratch/mhoyosro/project3/BLAST2_2026")
  
  files = {
      "Eonycteris_spelaea": "eSpe",
      "Miniopterus_schreibersii": "mSch",
      "Molossus_molossus": "mMol",
      "Myotis_daubentonii": "mDau",
      "Myotis_myotis": "mMyo",
      "Myotis_mystacinus": "mMys",
      "Phyllostomus_discolor": "pDis",
      "Pipistrellus_kuhlii": "pKuh",
      "Plecotus_auritus": "pAur",
      "Rhinolophus_ferrumequinum": "rFer",
      "Rhinolophus_hipposideros": "rHip",
      "Rhinolophus_sinicus": "rSin",
      "Rousettus_aegyptiacus": "rAeg",
      "Vespertilio_murinus": "vMur",
  }
  
  new_cols = ["qseqid_EXON_NAME_1", "qseqid_EXON_NAME_2", "qseqid_EXON_NAME_3"]
  
  for species, prefix in files.items():
      infile = base / species / f"{prefix}_vs_RepeatPeps.FINAL_FILTRO_1.tsv"
      outfile = base / species / f"{prefix}_vs_RepeatPeps.FINAL_FILTRO_2.tsv"
  
      df = pd.read_csv(infile, sep="\t")
  
      split_cols = df["qseqid_EXON_NAME"].astype(str).str.split("__", n=2, expand=True)
  
      while split_cols.shape[1] < 3:
          split_cols[split_cols.shape[1]] = ""
  
      split_cols = split_cols.iloc[:, :3]
      split_cols.columns = new_cols
  
      df = df.drop(columns=["qseqid_EXON_NAME"])
      df = pd.concat([split_cols, df], axis=1)
  
      df.to_csv(outfile, sep="\t", index=False, float_format="%.3f")
  
      print(f"{species}: {len(df)} rows -> {outfile}")
  
  print("DONE")

.. code-block:: bash

  python3 filtro2_split_exon_name_2026.py
