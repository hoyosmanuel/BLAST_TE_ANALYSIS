
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
---------------------------------------------------------
#primero necesito editar los puntos y comas y espacios raros:


sed 's/ /_/g; s/"//g; s/;/_/g' hLar_FINAL_transcriptome.bed > hLar_WORKING_1_transcriptome.bed


#Luego necesito concatener las columnas extra y poner un separador apropiado

awk 'BEGIN{FS=OFS="\t"} {
    # Concatenar columnas 10-16 con _ como separador
    concatenated = $10
    for(i=11; i<=16; i++) {
        if ($i != "") concatenated = concatenated "__" $i
    }
    # Imprimir columnas 1-9 + la concatenada
    print $1, $2, $3, $4, $5, $6, $7, $8, $9, concatenated
}' hLar_WORKING_1_transcriptome.bed > hLar_WORKING_2_transcriptome.bed


#Luego necesito poner la columa 10 donde corresponde y eliminar columnas inutules

awk 'BEGIN{FS=OFS="\t"} {
    # Mover columna 10 a posición 4 y eliminar columnas 7,8,9
    print $1, $2, $3, $10, $5, $6
}' hLar_WORKING_2_transcriptome.bed > hLar_WORKING_3_transcriptome.bed
