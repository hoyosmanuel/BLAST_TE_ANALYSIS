
1) Set directories
-------------------

.. code-block:: bash

  mkdir /lustre/scratch/mhoyosro/project3/BLAST2
  cd /lustre/scratch/mhoyosro/project3/BLAST2
  mkdir hipposideros molossus myotis phyllostomus pipistrellus rhinolophus rousettus


2) Do a FASTA transcriptome for each species
---------------------------------------------

#Copy the bed transcriptome from the previous experiment:

.. code-block:: bash

  cd /lustre/scratch/mhoyosro/project3/PLANCE/hipposideros
  cp hLar_FINAL_transcriptome.bed   /lustre/scratch/mhoyosro/project3/BLAST2/hipposideros
  
  cd /lustre/scratch/mhoyosro/project3/PLANCE/molossus
  cp mMol_FINAL_transcriptome.bed   /lustre/scratch/mhoyosro/project3/BLAST2/molossus
  
  cd /lustre/scratch/mhoyosro/project3/PLANCE/myotis
  cp mMyo_FINAL_transcriptome.bed   /lustre/scratch/mhoyosro/project3/BLAST2/myotis
  
  cd /lustre/scratch/mhoyosro/project3/PLANCE/phyllostomus
  cp pDis_FINAL_transcriptome.bed   /lustre/scratch/mhoyosro/project3/BLAST2/phyllostomus
  
  cd /lustre/scratch/mhoyosro/project3/PLANCE/pipistrellus
  cp pKuh_FINAL_transcriptome.bed   /lustre/scratch/mhoyosro/project3/BLAST2/pipistrellus
  
  cd /lustre/scratch/mhoyosro/project3/PLANCE/rhinolophus
  cp rFer_FINAL_transcriptome.bed   /lustre/scratch/mhoyosro/project3/BLAST2/rhinolophus
  
  cd /lustre/scratch/mhoyosro/project3/PLANCE/rousettus
  cp rAeg_FINAL_transcriptome.bed   /lustre/scratch/mhoyosro/project3/BLAST2/rousettus


3) Editar
----------
| Tengo que hacer varias ediciones:

primero 
~~~~~~~
Necesito editar los puntos y comas y espacios raros:

.. code-block:: bash

  sed 's/ /_/g; s/"//g; s/;/_/g' hLar_FINAL_transcriptome.bed > hLar_WORKING_1_transcriptome.bed

Luego 
~~~~~~~
Necesito concatener las columnas extra y poner un separador apropiado

.. code-block:: bash

  awk 'BEGIN{FS=OFS="\t"} {
      # Concatenar columnas 10-16 con _ como separador
      concatenated = $10
      for(i=11; i<=16; i++) {
          if ($i != "") concatenated = concatenated "__" $i
      }
      # Imprimir columnas 1-9 + la concatenada
      print $1, $2, $3, $4, $5, $6, $7, $8, $9, concatenated
  }' hLar_WORKING_1_transcriptome.bed > hLar_WORKING_2_transcriptome.bed


Luego 
~~~~~~~
Necesito poner la columa 10 donde corresponde y eliminar columnas inutules

.. code-block:: bash

  awk 'BEGIN{FS=OFS="\t"} {
      # Mover columna 10 a posición 4 y eliminar columnas 7,8,9
      print $1, $2, $3, $10, $5, $6
  }' hLar_WORKING_2_transcriptome.bed > hLar_WORKING_3_transcriptome.bed


El comando para todo eso es este:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

  cd /lustre/scratch/mhoyosro/project3/BLAST2
  
  # Encontrar todos los archivos que coincidan con el patrón y procesarlos
  find . -name "*_FINAL_transcriptome.bed" -type f | while read input_file; do
      # Definir el archivo de salida
      output_file="${input_file%_FINAL_transcriptome.bed}_TRANSCRIPTOME_PROCESSED.bed"
      
      echo "Procesando: $input_file"
      
      # Aplicar los tres pasos en una sola tubería
      sed 's/ /_/g; s/"//g; s/;/_/g' "$input_file" | \
      awk 'BEGIN{FS=OFS="\t"} {
          concatenated = $10
          for(i=11; i<=16; i++) {
              if ($i != "") concatenated = concatenated "__" $i
          }
          print $1, $2, $3, $4, $5, $6, $7, $8, $9, concatenated
      }' | \
      awk 'BEGIN{FS=OFS="\t"} {
          print $1, $2, $3, $10, $5, $6
      }' > "$output_file"
      
      echo "Creado: $output_file"
  done



4) Pasar todos a FASTA:
-----------------------

.. code-block:: bash

  nano faster.sh

.. code-block:: bash

  #!/bin/bash
  #SBATCH --job-name=faster
  #SBATCH --output=%x.%j.out
  #SBATCH --error=%x.%j.err
  #SBATCH --partition=nocona
  #SBATCH --nodes=1
  #SBATCH --ntasks=36
  
  . /home/mhoyosro/conda/etc/profile.d/conda.sh
  conda activate alineador
  
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/hipposideros
  sort -k1,1 -k2,2n hLar_TRANSCRIPTOME_PROCESSED.bed > hLar_sorted.bed
  bedtools getfasta -fi /lustre/scratch/mhoyosro/project3/GENOMES/mHipLar1.2.pri.fa \
                    -bed hLar_sorted.bed \
                    -fo hLar_TRANSCRIPTOME.fasta \
                    -name -s
  rm hLar_sorted.bed
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/molossus
  sort -k1,1 -k2,2n mMol_TRANSCRIPTOME_PROCESSED.bed > mMol_sorted.bed
  bedtools getfasta -fi /lustre/scratch/mhoyosro/project3/GENOMES/mMolMol1.2.pri.fa \
                    -bed mMol_sorted.bed \
                    -fo mMol_TRANSCRIPTOME.fasta \
                    -name -s
  rm mMol_sorted.bed
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/myotis
  sort -k1,1 -k2,2n mMyo_TRANSCRIPTOME_PROCESSED.bed > mMyo_sorted.bed
  bedtools getfasta -fi /lustre/scratch/mhoyosro/project3/GENOMES/mMyoMyo1.6.pri.fa \
                    -bed mMyo_sorted.bed \
                    -fo mMyo_TRANSCRIPTOME.fasta \
                    -name -s
  rm mMyo_sorted.bed
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/phyllostomus
  sort -k1,1 -k2,2n pDis_TRANSCRIPTOME_PROCESSED.bed > pDis_sorted.bed
  bedtools getfasta -fi /lustre/scratch/mhoyosro/project3/GENOMES/mPhyDis1.3.pri.fa \
                    -bed pDis_sorted.bed \
                    -fo pDis_TRANSCRIPTOME.fasta \
                    -name -s
  rm pDis_sorted.bed
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/pipistrellus
  sort -k1,1 -k2,2n pKuh_TRANSCRIPTOME_PROCESSED.bed > pKuh_sorted.bed
  bedtools getfasta -fi /lustre/scratch/mhoyosro/project3/GENOMES/mPipKuh1.2.pri.fa \
                    -bed pKuh_sorted.bed \
                    -fo pKuh_TRANSCRIPTOME.fasta \
                    -name -s
  rm pKuh_sorted.bed
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/rhinolophus
  sort -k1,1 -k2,2n rFer_TRANSCRIPTOME_PROCESSED.bed > rFer_sorted.bed
  bedtools getfasta -fi /lustre/scratch/mhoyosro/project3/GENOMES/mRhiFer1.5.pri.fa \
                    -bed rFer_sorted.bed \
                    -fo rFer_TRANSCRIPTOME.fasta \
                    -name -s
  rm rFer_sorted.bed
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/rousettus
  sort -k1,1 -k2,2n rAeg_TRANSCRIPTOME_PROCESSED.bed > rAeg_sorted.bed
  bedtools getfasta -fi /lustre/scratch/mhoyosro/project3/GENOMES/mRouAeg1.4.pri.fa \
                    -bed rAeg_sorted.bed \
                    -fo rAeg_TRANSCRIPTOME.fasta \
                    -name -s
  rm rAeg_sorted.bed


5) Create RepeatPeps.lib DataBase
----------------------------------

.. code-block:: bash
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2
  lib=/lustre/scratch/mhoyosro/project3/RepeatPeps.lib
  
  . /home/mhoyosro/conda/etc/profile.d/conda.sh
  conda activate blast
  
  makeblastdb -in $lib -dbtype prot -out RepeatPeps_db


6) Do a BLASTX on each transcriptome
-------------------------------------

.. code-block:: bash

  nano blaster.sh

.. code-block:: bash

  #!/bin/bash
  #SBATCH --job-name=blaster
  #SBATCH --output=%x.%j.out
  #SBATCH --error=%x.%j.err
  #SBATCH --partition=nocona
  #SBATCH --nodes=1
  #SBATCH --ntasks=36
  
  . /home/mhoyosro/conda/etc/profile.d/conda.sh
  conda activate blast
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/hipposideros
  blastx -query hLar_TRANSCRIPTOME.fasta \
    -db /lustre/scratch/mhoyosro/project3/BLAST/RepeatPeps_db \
    -evalue 1e-50 \
    -outfmt "6 qseqid sseqid pident length qstart qend sstart send evalue bitscore qlen slen qcovs qcovhsp" \
    -num_threads 36 \
    -max_target_seqs 10 \
    -out hLar_vs_RepeatPeps.raw.tsv
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/molossus
  blastx -query mMol_TRANSCRIPTOME.fasta \
    -db /lustre/scratch/mhoyosro/project3/BLAST/RepeatPeps_db \
    -evalue 1e-50 \
    -outfmt "6 qseqid sseqid pident length qstart qend sstart send evalue bitscore qlen slen qcovs qcovhsp" \
    -num_threads 36 \
    -max_target_seqs 10 \
    -out mMol_vs_RepeatPeps.raw.tsv
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/myotis
  blastx -query mMyo_TRANSCRIPTOME.fasta \
    -db /lustre/scratch/mhoyosro/project3/BLAST/RepeatPeps_db \
    -evalue 1e-50 \
    -outfmt "6 qseqid sseqid pident length qstart qend sstart send evalue bitscore qlen slen qcovs qcovhsp" \
    -num_threads 36 \
    -max_target_seqs 5 \
    -out mMyo_vs_RepeatPeps.raw.tsv
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/phyllostomus
  blastx -query pDis_TRANSCRIPTOME.fasta \
    -db /lustre/scratch/mhoyosro/project3/BLAST/RepeatPeps_db \
    -evalue 1e-50 \
    -outfmt "6 qseqid sseqid pident length qstart qend sstart send evalue bitscore qlen slen qcovs qcovhsp" \
    -num_threads 36 \
    -max_target_seqs 10 \
    -out pDis_vs_RepeatPeps.raw.tsv
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/pipistrellus
  blastx -query pKuh_TRANSCRIPTOME.fasta \
    -db /lustre/scratch/mhoyosro/project3/BLAST/RepeatPeps_db \
    -evalue 1e-50 \
    -outfmt "6 qseqid sseqid pident length qstart qend sstart send evalue bitscore qlen slen qcovs qcovhsp" \
    -num_threads 36 \
    -max_target_seqs 10 \
    -out pKuh_vs_RepeatPeps.raw.tsv
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/rhinolophus
  blastx -query rFer_TRANSCRIPTOME.fasta \
    -db /lustre/scratch/mhoyosro/project3/BLAST/RepeatPeps_db \
    -evalue 1e-50 \
    -outfmt "6 qseqid sseqid pident length qstart qend sstart send evalue bitscore qlen slen qcovs qcovhsp" \
    -num_threads 36 \
    -max_target_seqs 10 \
    -out rFer_vs_RepeatPeps.raw.tsv
  
  cd /lustre/scratch/mhoyosro/project3/BLAST/rousettus
  blastx -query rAeg_TRANSCRIPTOME.fasta \
    -db /lustre/scratch/mhoyosro/project3/BLAST/RepeatPeps_db \
    -evalue 1e-50 \
    -outfmt "6 qseqid sseqid pident length qstart qend sstart send evalue bitscore qlen slen qcovs qcovhsp" \
    -num_threads 36 \
    -max_target_seqs 10 \
    -out rAeg_vs_RepeatPeps.raw.tsv

OJO!!! Hay una forma más rápida de hacer todo esto!! 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

  #!/bin/bash
  #SBATCH --job-name=Rcaster
  #SBATCH --output=%x.%A_%a.out
  #SBATCH --error=%x.%A_%a.err
  #SBATCH --partition=nocona
  #SBATCH --nodes=1
  #SBATCH --ntasks=36
  #SBATCH --array=0-6
  
  . /home/mhoyosro/conda/etc/profile.d/conda.sh
  conda activate blast
  
  # directorios
  DIRS=(
  "/lustre/scratch/mhoyosro/project3/BLAST2/hipposideros"
  "/lustre/scratch/mhoyosro/project3/BLAST2/molossus"
  "/lustre/scratch/mhoyosro/project3/BLAST2/myotis"
  "/lustre/scratch/mhoyosro/project3/BLAST2/phyllostomus"
  "/lustre/scratch/mhoyosro/project3/BLAST2/pipistrellus"
  "/lustre/scratch/mhoyosro/project3/BLAST2/rhinolophus"
  "/lustre/scratch/mhoyosro/project3/BLAST2/rousettus"
  )
  
  # queries
  QUERIES=(
  "hLar_TRANSCRIPTOME.fasta"
  "mMol_TRANSCRIPTOME.fasta"
  "mMyo_TRANSCRIPTOME.fasta"
  "pDis_TRANSCRIPTOME.fasta"
  "pKuh_TRANSCRIPTOME.fasta"
  "rFer_TRANSCRIPTOME.fasta"
  "rAeg_TRANSCRIPTOME.fasta"
  )
  
  # outputs
  OUTS=(
  "hLar_vs_RepeatPeps.raw.tsv"
  "mMol_vs_RepeatPeps.raw.tsv"
  "mMyo_vs_RepeatPeps.raw.tsv"
  "pDis_vs_RepeatPeps.raw.tsv"
  "pKuh_vs_RepeatPeps.raw.tsv"
  "rFer_vs_RepeatPeps.raw.tsv"
  "rAeg_vs_RepeatPeps.raw.tsv"
  )
  
  #Máximo 10 proteínas repetitivas por transcript.
  MAXTARGET=(10 10 10 10 10 10 10)
  
  # seleccion según array ID
  DIR=${DIRS[$SLURM_ARRAY_TASK_ID]}
  QRY=${QUERIES[$SLURM_ARRAY_TASK_ID]}
  OUT=${OUTS[$SLURM_ARRAY_TASK_ID]}
  MTS=${MAXTARGET[$SLURM_ARRAY_TASK_ID]}
  
  cd "$DIR"
  
  blastx \
    -query "$QRY" \
    -db /lustre/scratch/mhoyosro/project3/BLAST2/RepeatPeps_db \
    -evalue 1e-50 \
    -outfmt "6 qseqid sseqid pident length qstart qend sstart send evalue bitscore qlen slen qcovs qcovhsp" \
    -num_threads 36 \
    -max_target_seqs "$MTS" \
    -out "$OUT"
  

7) Arreglar el output
----------------------

7.1 Quitar :: y reemplazarlo por __
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

  cd /lustre/scratch/mhoyosro/project3/BLAST2/hipposideros
  sed 's/::/__/g' hLar_vs_RepeatPeps.raw.tsv > hLar_vs_RepeatPeps.working1.tsv
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/molossus
  sed 's/::/__/g' mMol_vs_RepeatPeps.raw.tsv > mMol_vs_RepeatPeps.working1.tsv
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/myotis
  sed 's/::/__/g' mMyo_vs_RepeatPeps.raw.tsv > mMyo_vs_RepeatPeps.working1.tsv
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/phyllostomus
  sed 's/::/__/g' pDis_vs_RepeatPeps.raw.tsv > pDis_vs_RepeatPeps.working1.tsv
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/pipistrellus
  sed 's/::/__/g' pKuh_vs_RepeatPeps.raw.tsv > pKuh_vs_RepeatPeps.working1.tsv
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/rhinolophus
  sed 's/::/__/g' rFer_vs_RepeatPeps.raw.tsv > rFer_vs_RepeatPeps.working1.tsv
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/rousettus
  sed 's/::/__/g' rAeg_vs_RepeatPeps.raw.tsv > rAeg_vs_RepeatPeps.working1.tsv


7.2 Quitar : y reemplazarlo por \t
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

  cd /lustre/scratch/mhoyosro/project3/BLAST2/hipposideros
  sed 's/:/\t/g' hLar_vs_RepeatPeps.working1.tsv > hLar_vs_RepeatPeps.working2.tsv
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/molossus
  sed 's/:/\t/g' mMol_vs_RepeatPeps.working1.tsv > mMol_vs_RepeatPeps.working2.tsv
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/myotis
  sed 's/:/\t/g' mMyo_vs_RepeatPeps.working1.tsv > mMyo_vs_RepeatPeps.working2.tsv
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/phyllostomus
  sed 's/:/\t/g' pDis_vs_RepeatPeps.working1.tsv > pDis_vs_RepeatPeps.working2.tsv
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/pipistrellus
  sed 's/:/\t/g' pKuh_vs_RepeatPeps.working1.tsv > pKuh_vs_RepeatPeps.working2.tsv
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/rhinolophus
  sed 's/:/\t/g' rFer_vs_RepeatPeps.working1.tsv > rFer_vs_RepeatPeps.working2.tsv
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/rousettus
  sed 's/:/\t/g' rAeg_vs_RepeatPeps.working1.tsv > rAeg_vs_RepeatPeps.working2.tsv


7.3 Modificar la segunda columna separando los valores
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

  cd /lustre/scratch/mhoyosro/project3/BLAST2/hipposideros
  sed -E 's/^([^\t]*)\t([0-9]+)-([0-9]+)\(([+-\.])\)/\1\t\2\t\3\t\4/'  hLar_vs_RepeatPeps.working2.tsv > hLar_vs_RepeatPeps.working3.tsv
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/molossus
  sed -E 's/^([^\t]*)\t([0-9]+)-([0-9]+)\(([+-\.])\)/\1\t\2\t\3\t\4/'  mMol_vs_RepeatPeps.working2.tsv > mMol_vs_RepeatPeps.working3.tsv
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/myotis
  sed -E 's/^([^\t]*)\t([0-9]+)-([0-9]+)\(([+-\.])\)/\1\t\2\t\3\t\4/'  mMyo_vs_RepeatPeps.working2.tsv > mMyo_vs_RepeatPeps.working3.tsv
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/phyllostomus
  sed -E 's/^([^\t]*)\t([0-9]+)-([0-9]+)\(([+-\.])\)/\1\t\2\t\3\t\4/'  pDis_vs_RepeatPeps.working2.tsv > pDis_vs_RepeatPeps.working3.tsv
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/pipistrellus
  sed -E 's/^([^\t]*)\t([0-9]+)-([0-9]+)\(([+-\.])\)/\1\t\2\t\3\t\4/'  pKuh_vs_RepeatPeps.working2.tsv > pKuh_vs_RepeatPeps.working3.tsv
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/rhinolophus
  sed -E 's/^([^\t]*)\t([0-9]+)-([0-9]+)\(([+-\.])\)/\1\t\2\t\3\t\4/'  rFer_vs_RepeatPeps.working2.tsv > rFer_vs_RepeatPeps.working3.tsv
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/rousettus
  sed -E 's/^([^\t]*)\t([0-9]+)-([0-9]+)\(([+-\.])\)/\1\t\2\t\3\t\4/'  rAeg_vs_RepeatPeps.working2.tsv > rAeg_vs_RepeatPeps.working3.tsv


7.4 Quitar # y reemplazarlo por  \t
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

  cd /lustre/scratch/mhoyosro/project3/BLAST2/hipposideros
  sed 's/#/\t/g' hLar_vs_RepeatPeps.working3.tsv > hLar_vs_RepeatPeps.working4.tsv
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/molossus
  sed 's/#/\t/g' mMol_vs_RepeatPeps.working3.tsv > mMol_vs_RepeatPeps.working4.tsv
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/myotis
  sed 's/#/\t/g' mMyo_vs_RepeatPeps.working3.tsv > mMyo_vs_RepeatPeps.working4.tsv
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/phyllostomus
  sed 's/#/\t/g' pDis_vs_RepeatPeps.working3.tsv > pDis_vs_RepeatPeps.working4.tsv
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/pipistrellus
  sed 's/#/\t/g' pKuh_vs_RepeatPeps.working3.tsv > pKuh_vs_RepeatPeps.working4.tsv
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/rhinolophus
  sed 's/#/\t/g' rFer_vs_RepeatPeps.working3.tsv > rFer_vs_RepeatPeps.working4.tsv
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/rousettus
  sed 's/#/\t/g' rAeg_vs_RepeatPeps.working3.tsv> rAeg_vs_RepeatPeps.working4.tsv


8) Ahora voy a ponerle headers a las tablas
-------------------------------------------

.. code-block:: bash

  cd /lustre/scratch/mhoyosro/project3/BLAST2/hipposideros
  echo -e "qseqid_EXON_NAME\tqseqid_start\tqseqid_end\tqseqid_sense\tTE_FAMILY\tTE_TYPE\tpident\tlength\tqstart\tqend\tsstart\tsend\tevalue\tbitscore\tqlen\tslen\tqcovs\tqcovhsp" \ | cat - hLar_vs_RepeatPeps.working4.tsv > hLar_vs_RepeatPeps.FINAL.tsv
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/molossus
  echo -e "qseqid_EXON_NAME\tqseqid_start\tqseqid_end\tqseqid_sense\tTE_FAMILY\tTE_TYPE\tpident\tlength\tqstart\tqend\tsstart\tsend\tevalue\tbitscore\tqlen\tslen\tqcovs\tqcovhsp" \ | cat - mMol_vs_RepeatPeps.working4.tsv > mMol_vs_RepeatPeps.FINAL.tsv
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/myotis
  echo -e "qseqid_EXON_NAME\tqseqid_start\tqseqid_end\tqseqid_sense\tTE_FAMILY\tTE_TYPE\tpident\tlength\tqstart\tqend\tsstart\tsend\tevalue\tbitscore\tqlen\tslen\tqcovs\tqcovhsp" \ | cat - mMyo_vs_RepeatPeps.working4.tsv > mMyo_vs_RepeatPeps.FINAL.tsv
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/phyllostomus
  echo -e "qseqid_EXON_NAME\tqseqid_start\tqseqid_end\tqseqid_sense\tTE_FAMILY\tTE_TYPE\tpident\tlength\tqstart\tqend\tsstart\tsend\tevalue\tbitscore\\tqlen\tslen\tqcovs\tqcovhsp" \ | cat - pDis_vs_RepeatPeps.working4.tsv > pDis_vs_RepeatPeps.FINAL.tsv
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/pipistrellus
  echo -e "qseqid_EXON_NAME\tqseqid_start\tqseqid_end\tqseqid_sense\tTE_FAMILY\tTE_TYPE\tpident\tlength\tqstart\tqend\tsstart\tsend\tevalue\tbitscore\tqlen\tslen\tqcovs\tqcovhsp" \ | cat - pKuh_vs_RepeatPeps.working4.tsv > pKuh_vs_RepeatPeps.FINAL.tsv
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/rhinolophus
  echo -e "qseqid_EXON_NAME\tqseqid_start\tqseqid_end\tqseqid_sense\tTE_FAMILY\tTE_TYPE\tpident\tlength\tqstart\tqend\tsstart\tsend\tevalue\tbitscore\tqlen\tslen\tqcovs\tqcovhsp" \ | cat - rFer_vs_RepeatPeps.working4.tsv > rFer_vs_RepeatPeps.FINAL.tsv
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/rousettus
  echo -e "qseqid_EXON_NAME\tqseqid_start\tqseqid_end\tqseqid_sense\tTE_FAMILY\tTE_TYPE\tpident\tlength\tqstart\tqend\tsstart\tsend\tevalue\tbitscore\tqlen\tslen\tqcovs\tqcovhsp" \ | cat - rAeg_vs_RepeatPeps.working4.tsv > rAeg_vs_RepeatPeps.FINAL.tsv

9) Ahora voy a ordenar por qseqid_start, qseqid_end y luego por qseqid_EXON_NAME
--------------------------------------------------------------------------------

.. code-block:: bash

  . $HOME/conda/etc/profile.d/conda.sh
  conda activate biopython
  ipython
  import pandas as pd
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/hipposideros
  df = pd.read_csv("hLar_vs_RepeatPeps.FINAL.tsv", sep="\t")
  # Ordenar por qseqid_start, qseqid_end, y luego qseqid_EXON_NAME2
  df_sorted = df.sort_values(by=["qseqid_start", "qseqid_end", "qseqid_EXON_NAME", "pident", "bitscore"],
  ascending=[True, True, True, False, True]  # pident de mayor a menor)
  # Guardar el archivo ordenado
  df_sorted.to_csv("hLar_vs_RepeatPeps.FINAL_ordenado.tsv", sep="\t", index=False)
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/molossus
  df = pd.read_csv("mMol_vs_RepeatPeps.FINAL.tsv", sep="\t")
  # Ordenar por qseqid_start, qseqid_end, y luego qseqid_EXON_NAME
  df_sorted = df.sort_values(by=["qseqid_start", "qseqid_end", "qseqid_EXON_NAME", "pident", "bitscore"],
  ascending=[True, True, True, False, True]  # pident de mayor a menor)
  # Guardar el archivo ordenado
  df_sorted.to_csv("mMol_vs_RepeatPeps.FINAL_ordenado.tsv", sep="\t", index=False)
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/myotis
  df = pd.read_csv("mMyo_vs_RepeatPeps.FINAL.tsv", sep="\t")
  # Ordenar por qseqid_start, qseqid_end, y luego qseqid_EXON_NAME
  df_sorted = df.sort_values(by=["qseqid_start", "qseqid_end", "qseqid_EXON_NAME", "pident", "bitscore"],
  ascending=[True, True, True, False, True]  # pident de mayor a menor)
  # Guardar el archivo ordenado
  df_sorted.to_csv("mMyo_vs_RepeatPeps.FINAL_ordenado.tsv", sep="\t", index=False)
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/phyllostomus
  df = pd.read_csv("pDis_vs_RepeatPeps.FINAL.tsv", sep="\t")
  # Ordenar por qseqid_start, qseqid_end, y luego qseqid_EXON_NAME
  df_sorted = df.sort_values(by=["qseqid_start", "qseqid_end", "qseqid_EXON_NAME", "pident", "bitscore"],
  ascending=[True, True, True, False, True]  # pident de mayor a menor)
  # Guardar el archivo ordenado
  df_sorted.to_csv("pDis_vs_RepeatPeps.FINAL_ordenado.tsv", sep="\t", index=False)
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/pipistrellus
  df = pd.read_csv("pKuh_vs_RepeatPeps.FINAL.tsv", sep="\t")
  # Ordenar por qseqid_start, qseqid_end, y luego qseqid_EXON_NAME
  df_sorted = df.sort_values(by=["qseqid_start", "qseqid_end", "qseqid_EXON_NAME", "pident", "bitscore"],
  ascending=[True, True, True, False, True]  # pident de mayor a menor)
  # Guardar el archivo ordenado
  df_sorted.to_csv("pKuh_vs_RepeatPeps.FINAL_ordenado.tsv", sep="\t", index=False)
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/rhinolophus
  df = pd.read_csv("rFer_vs_RepeatPeps.FINAL.tsv", sep="\t")
  # Ordenar por qseqid_start, qseqid_end, y luego qseqid_EXON_NAME
  df_sorted = df.sort_values(by=["qseqid_start", "qseqid_end", "qseqid_EXON_NAME", "pident", "bitscore"],
  ascending=[True, True, True, False, True]  # pident de mayor a menor)
  # Guardar el archivo ordenado
  df_sorted.to_csv("rFer_vs_RepeatPeps.FINAL_ordenado.tsv", sep="\t", index=False)
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/rousettus
  df = pd.read_csv("rAeg_vs_RepeatPeps.FINAL.tsv", sep="\t")
  # Ordenar por qseqid_start, qseqid_end, y luego qseqid_EXON_NAME
  df_sorted = df.sort_values(by=["qseqid_start", "qseqid_end", "qseqid_EXON_NAME", "pident", "bitscore"],
  ascending=[True, True, True, False, True]  # pident de mayor a menor)
  # Guardar el archivo ordenado
  df_sorted.to_csv("rAeg_vs_RepeatPeps.FINAL_ordenado.tsv", sep="\t", index=False)
  
  
Ojo!! Todo ese chorro se puede cambiar por este:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

  import pandas as pd
  from pathlib import Path
  
  folders_files = {
      "hipposideros": "hLar_vs_RepeatPeps.FINAL.tsv",
      "molossus": "mMol_vs_RepeatPeps.FINAL.tsv",
      "myotis": "mMyo_vs_RepeatPeps.FINAL.tsv",
      "phyllostomus": "pDis_vs_RepeatPeps.FINAL.tsv",
      "pipistrellus": "pKuh_vs_RepeatPeps.FINAL.tsv",
      "rhinolophus": "rFer_vs_RepeatPeps.FINAL.tsv",
      "rousettus": "rAeg_vs_RepeatPeps.FINAL.tsv"
  }
  
  # Ruta base
  base_path = Path("/lustre/scratch/mhoyosro/project3/BLAST2")
  
  # Iterar por cada carpeta y archivo
  for folder, infile in folders_files.items():
      folder_path = base_path / folder
      infile_path = folder_path / infile
      outfile_path = folder_path / infile.replace(".FINAL.tsv", ".FINAL_ordenado.tsv")
      
      # Leer archivo
      df = pd.read_csv(infile_path, sep="\t")
      
      # Ordenar
      df_sorted = df.sort_values(
          by=["qseqid_start", "qseqid_end", "qseqid_EXON_NAME", "pident", "bitscore"],
          ascending=[True, True, True, False, True]  # pident de mayor a menor
      )
      
      # Guardar archivo ordenado
      df_sorted.to_csv(outfile_path, sep="\t", index=False)
      
      print(f"Archivo ordenado guardado en: {outfile_path}")

10) “drop duplicates keeping first”.
-------------------------------------

.. code-block:: bash

  import pandas as pd
  cd /lustre/scratch/mhoyosro/project3/BLAST2/hipposideros
  # Leer archivo
  df = pd.read_csv("hLar_vs_RepeatPeps.FINAL_ordenado.tsv", sep="\t")
  # Eliminar duplicados basándose en estas columnas, manteniendo la primera fila de cada grupo
  df_unique = df.drop_duplicates(subset=["qseqid_EXON_NAME", "qseqid_start", "qseqid_end"], keep="first")
  # Guardar archivo resultante
  df_unique.to_csv(""hLar_vs_RepeatPeps.FINAL_FILTRO_1.tsv"", sep="\t", index=False)
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/molossus
  # Leer archivo
  df = pd.read_csv("mMol_vs_RepeatPeps.FINAL_ordenado.tsv", sep="\t")
  # Eliminar duplicados basándose en estas columnas, manteniendo la primera fila de cada grupo
  df_unique = df.drop_duplicates(subset=["qseqid_EXON_NAME", "qseqid_start", "qseqid_end"], keep="first")
  # Guardar archivo resultante
  df_unique.to_csv(""mMol_vs_RepeatPeps.FINAL_FILTRO_1.tsv"", sep="\t", index=False)
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/myotis
  # Leer archivo
  df = pd.read_csv("mMyo_vs_RepeatPeps.FINAL_ordenado.tsv", sep="\t")
  # Eliminar duplicados basándose en estas columnas, manteniendo la primera fila de cada grupo
  df_unique = df.drop_duplicates(subset=["qseqid_EXON_NAME", "qseqid_start", "qseqid_end"], keep="first")
  # Guardar archivo resultante
  df_unique.to_csv(""mMyo_vs_RepeatPeps.FINAL_FILTRO_1.tsv"", sep="\t", index=False)
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/phyllostomus
  # Leer archivo
  df = pd.read_csv("pDis_vs_RepeatPeps.FINAL_ordenado.tsv", sep="\t")
  # Eliminar duplicados basándose en estas columnas, manteniendo la primera fila de cada grupo
  df_unique = df.drop_duplicates(subset=["qseqid_EXON_NAME", "qseqid_start", "qseqid_end"], keep="first")
  # Guardar archivo resultante
  df_unique.to_csv(""pDis_vs_RepeatPeps.FINAL_FILTRO_1.tsv"", sep="\t", index=False)
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/pipistrellus
  # Leer archivo
  df = pd.read_csv("pKuh_vs_RepeatPeps.FINAL_ordenado.tsv", sep="\t")
  # Eliminar duplicados basándose en estas columnas, manteniendo la primera fila de cada grupo
  df_unique = df.drop_duplicates(subset=["qseqid_EXON_NAME", "qseqid_start", "qseqid_end"], keep="first")
  # Guardar archivo resultante
  df_unique.to_csv(""pKuh_vs_RepeatPeps.FINAL_FILTRO_1.tsv"", sep="\t", index=False)
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/rhinolophus
  # Leer archivo
  df = pd.read_csv("rFer_vs_RepeatPeps.FINAL_ordenado.tsv", sep="\t")
  # Eliminar duplicados basándose en estas columnas, manteniendo la primera fila de cada grupo
  df_unique = df.drop_duplicates(subset=["qseqid_EXON_NAME", "qseqid_start", "qseqid_end"], keep="first")
  # Guardar archivo resultante
  df_unique.to_csv(""rFer_vs_RepeatPeps.FINAL_FILTRO_1.tsv"", sep="\t", index=False)
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/rousettus
  # Leer archivo
  df = pd.read_csv("rAeg_vs_RepeatPeps.FINAL_ordenado.tsv", sep="\t")
  # Eliminar duplicados basándose en estas columnas, manteniendo la primera fila de cada grupo
  df_unique = df.drop_duplicates(subset=["qseqid_EXON_NAME", "qseqid_start", "qseqid_end"], keep="first")
  # Guardar archivo resultante
  df_unique.to_csv(""rAeg_vs_RepeatPeps.FINAL_FILTRO_1.tsv"", sep="\t", index=False)

  
Ojo!! Todo ese chorro se puede cambiar por este código
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

  import pandas as pd
  from pathlib import Path
  
  # Diccionario de carpetas y archivos ordenados
  folders_files = {
      "hipposideros": "hLar_vs_RepeatPeps.FINAL_ordenado.tsv",
      "molossus": "mMol_vs_RepeatPeps.FINAL_ordenado.tsv",
      "myotis": "mMyo_vs_RepeatPeps.FINAL_ordenado.tsv",
      "phyllostomus": "pDis_vs_RepeatPeps.FINAL_ordenado.tsv",
      "pipistrellus": "pKuh_vs_RepeatPeps.FINAL_ordenado.tsv",
      "rhinolophus": "rFer_vs_RepeatPeps.FINAL_ordenado.tsv",
      "rousettus": "rAeg_vs_RepeatPeps.FINAL_ordenado.tsv"
  }
  
  # Ruta base
  base_path = Path("/lustre/scratch/mhoyosro/project3/BLAST2")
  
  # Iterar por cada carpeta y archivo
  for folder, infile in folders_files.items():
      infile_path = base_path / folder / infile
      outfile_path = infile_path.with_name(infile.replace(".FINAL_ordenado.tsv", ".FINAL_FILTRO_1.tsv"))
      
      # Leer archivo
      df = pd.read_csv(infile_path, sep="\t")
      
      # Eliminar duplicados basándose en columnas específicas
      df_unique = df.drop_duplicates(subset=["qseqid_EXON_NAME", "qseqid_start", "qseqid_end"], keep="first")
      
      # Guardar archivo resultante
      df_unique.to_csv(outfile_path, sep="\t", index=False)
      
      print(f"Archivo filtrado guardado en: {outfile_path}")



11) Ahora necesito arreglar el qseqid_EXON_NAME de cada archivo
----------------------------------------------------------------

.. code-block:: bash

  import pandas as pd
  
  # Cargar archivo
  cd /lustre/scratch/mhoyosro/project3/BLAST2/hipposideros
  df = pd.read_csv("hLar_vs_RepeatPeps.FINAL_FILTRO_1.tsv", sep="\t")
  # Separar la columna 'qseqid_EXON_NAME' en 3 partes usando '__' como separador
  # Limita a 3 partes para que todo lo que quede vaya a la tercera columna
  df[['qseqid_EXON_NAME_1','qseqid_EXON_NAME_2','qseqid_EXON_NAME_3']] = df['qseqid_EXON_NAME'].str.split('__', n=2, expand=True)
  # Eliminar la columna original
  df = df.drop(columns=['qseqid_EXON_NAME'])
  # Reordenar columnas para poner las nuevas al inicio
  cols = ['qseqid_EXON_NAME_1','qseqid_EXON_NAME_2','qseqid_EXON_NAME_3'] + [c for c in df.columns if c not in ['qseqid_EXON_NAME_1','qseqid_EXON_NAME_2','qseqid_EXON_NAME_3']]
  df = df[cols]
  # Guardar resultado
  df.to_csv("hLar_vs_RepeatPeps.FINAL_FILTRO_2.tsv", sep="\t", index=False, float_format="%.3f")
  
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/molossus
  df = pd.read_csv("mMol_vs_RepeatPeps.FINAL_FILTRO_1.tsv", sep="\t")
  df[['qseqid_EXON_NAME_1','qseqid_EXON_NAME_2','qseqid_EXON_NAME_3']] = df['qseqid_EXON_NAME'].str.split('__', n=2, expand=True)
  df = df.drop(columns=['qseqid_EXON_NAME'])
  cols = ['qseqid_EXON_NAME_1','qseqid_EXON_NAME_2','qseqid_EXON_NAME_3'] + [c for c in df.columns if c not in ['qseqid_EXON_NAME_1','qseqid_EXON_NAME_2','qseqid_EXON_NAME_3']]
  df = df[cols]
  df.to_csv("mMol_vs_RepeatPeps.FINAL_FILTRO_2.tsv", sep="\t", index=False, float_format="%.3f")
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/myotis
  df = pd.read_csv("mMyo_vs_RepeatPeps.FINAL_FILTRO_1.tsv", sep="\t")
  df[['qseqid_EXON_NAME_1','qseqid_EXON_NAME_2','qseqid_EXON_NAME_3']] = df['qseqid_EXON_NAME'].str.split('__', n=2, expand=True)
  df = df.drop(columns=['qseqid_EXON_NAME'])
  cols = ['qseqid_EXON_NAME_1','qseqid_EXON_NAME_2','qseqid_EXON_NAME_3'] + [c for c in df.columns if c not in ['qseqid_EXON_NAME_1','qseqid_EXON_NAME_2','qseqid_EXON_NAME_3']]
  df = df[cols]
  df.to_csv("mMyo_vs_RepeatPeps.FINAL_FILTRO_2.tsv", sep="\t", index=False, float_format="%.3f")
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/phyllostomus
  df = pd.read_csv("pDis_vs_RepeatPeps.FINAL_FILTRO_1.tsv", sep="\t")
  df[['qseqid_EXON_NAME_1','qseqid_EXON_NAME_2','qseqid_EXON_NAME_3']] = df['qseqid_EXON_NAME'].str.split('__', n=2, expand=True)
  df = df.drop(columns=['qseqid_EXON_NAME'])
  cols = ['qseqid_EXON_NAME_1','qseqid_EXON_NAME_2','qseqid_EXON_NAME_3'] + [c for c in df.columns if c not in ['qseqid_EXON_NAME_1','qseqid_EXON_NAME_2','qseqid_EXON_NAME_3']]
  df = df[cols]
  df.to_csv("pDis_vs_RepeatPeps.FINAL_FILTRO_2.tsv", sep="\t", index=False, float_format="%.3f")
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/pipistrellus
  df = pd.read_csv("pKuh_vs_RepeatPeps.FINAL_FILTRO_1.tsv", sep="\t")
  df[['qseqid_EXON_NAME_1','qseqid_EXON_NAME_2','qseqid_EXON_NAME_3']] = df['qseqid_EXON_NAME'].str.split('__', n=2, expand=True)
  df = df.drop(columns=['qseqid_EXON_NAME'])
  cols = ['qseqid_EXON_NAME_1','qseqid_EXON_NAME_2','qseqid_EXON_NAME_3'] + [c for c in df.columns if c not in ['qseqid_EXON_NAME_1','qseqid_EXON_NAME_2','qseqid_EXON_NAME_3']]
  df = df[cols]
  df.to_csv("pKuh_vs_RepeatPeps.FINAL_FILTRO_2.tsv", sep="\t", index=False, float_format="%.3f")
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/rhinolophus
  df = pd.read_csv("rFer_vs_RepeatPeps.FINAL_FILTRO_1.tsv", sep="\t")
  df[['qseqid_EXON_NAME_1','qseqid_EXON_NAME_2','qseqid_EXON_NAME_3']] = df['qseqid_EXON_NAME'].str.split('__', n=2, expand=True)
  df = df.drop(columns=['qseqid_EXON_NAME'])
  cols = ['qseqid_EXON_NAME_1','qseqid_EXON_NAME_2','qseqid_EXON_NAME_3'] + [c for c in df.columns if c not in ['qseqid_EXON_NAME_1','qseqid_EXON_NAME_2','qseqid_EXON_NAME_3']]
  df = df[cols]
  df.to_csv("rFer_vs_RepeatPeps.FINAL_FILTRO_2.tsv", sep="\t", index=False, float_format="%.3f")
  
  cd /lustre/scratch/mhoyosro/project3/BLAST2/rousettus
  df = pd.read_csv("rAeg_vs_RepeatPeps.FINAL_FILTRO_1.tsv", sep="\t")
  df[['qseqid_EXON_NAME_1','qseqid_EXON_NAME_2','qseqid_EXON_NAME_3']] = df['qseqid_EXON_NAME'].str.split('__', n=2, expand=True)
  df = df.drop(columns=['qseqid_EXON_NAME'])
  cols = ['qseqid_EXON_NAME_1','qseqid_EXON_NAME_2','qseqid_EXON_NAME_3'] + [c for c in df.columns if c not in ['qseqid_EXON_NAME_1','qseqid_EXON_NAME_2','qseqid_EXON_NAME_3']]
  df = df[cols]
  df.to_csv("rAeg_vs_RepeatPeps.FINAL_FILTRO_2.tsv", sep="\t", index=False, float_format="%.3f")


OJO!! Todo ese chorro puede ser reemplazado por este código:
------------------------------------------------------------

.. code-block:: bash

  import pandas as pd
  from pathlib import Path
  
  # Lista de carpetas y archivos
  carpetas_archivos = {
      "hipposideros": "hLar_vs_RepeatPeps.FINAL_FILTRO_1.tsv",
      "molossus": "mMol_vs_RepeatPeps.FINAL_FILTRO_1.tsv",
      "myotis": "mMyo_vs_RepeatPeps.FINAL_FILTRO_1.tsv",
      "phyllostomus": "pDis_vs_RepeatPeps.FINAL_FILTRO_1.tsv",
      "pipistrellus": "pKuh_vs_RepeatPeps.FINAL_FILTRO_1.tsv",
      "rhinolophus": "rFer_vs_RepeatPeps.FINAL_FILTRO_1.tsv",
      "rousettus": "rAeg_vs_RepeatPeps.FINAL_FILTRO_1.tsv"
  }
  
  base_dir = Path("/lustre/scratch/mhoyosro/project3/BLAST2")
  
  for carpeta, archivo in carpetas_archivos.items():
      ruta = base_dir / carpeta / archivo
      df = pd.read_csv(ruta, sep="\t")
  
      # Separar qseqid_EXON_NAME en 3 columnas
      df[['qseqid_EXON_NAME_1','qseqid_EXON_NAME_2','qseqid_EXON_NAME_3']] = df['qseqid_EXON_NAME'].str.split('__', n=2, expand=True)
      df = df.drop(columns=['qseqid_EXON_NAME'])
  
      # Reordenar columnas
      cols = ['qseqid_EXON_NAME_1','qseqid_EXON_NAME_2','qseqid_EXON_NAME_3'] + \
             [c for c in df.columns if c not in ['qseqid_EXON_NAME_1','qseqid_EXON_NAME_2','qseqid_EXON_NAME_3']]
      df = df[cols]
  
      # Guardar nuevo archivo con sufijo _2
      nuevo_archivo = archivo.replace("FINAL_FILTRO_1", "FINAL_FILTRO_2")
      df.to_csv(base_dir / carpeta / nuevo_archivo, sep="\t", index=False, float_format="%.3f")
  
      print(f"Procesado: {carpeta}/{archivo}")


Finalmente voy a juntar todo en un excel que se llame FILTER2.xlsx y voy a arreglar los ultimos detalles manualmente
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

qcovs que es el  porcentaje de cobertura del query en BLAST. = porcentaje de la secuencia query completa que está cubierta por el alineamiento (o por el conjunto de HSPs para ese hit).
qlen = longitud total de la secuencia query (los trasncriptomas)

