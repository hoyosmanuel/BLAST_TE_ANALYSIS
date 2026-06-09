
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



