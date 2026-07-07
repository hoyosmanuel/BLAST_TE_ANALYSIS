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
  Hipposideros_larvatus mHipLar1.2.pri.TOGA hLar
  Miniopterus_schreibersii mMinSch1.1.hap1.TOGA mSch
  Molossus_molossus mMolMol1.2.pri.TOGA mMol
  Myotis_daubentonii mMyoDau2.1.pri.TOGA mDau
  Myotis_myotis mMyoMyo1.6.pri.TOGA mMyo
  Myotis_mystacinus mMyoMys1.1.hap1.TOGA mMys
  Phyllostomus_discolor mPhyDis1.3.pri.TOGA pDis
  Pipistrellus_kuhlii mPipKuh1.2.pri.TOGA pKuh
  Pipistrellus_pygmaeus mPipKuh1.2.pri.TOGA pPyg
  Plecotus_auritus mPleAur1.1.pri.TOGA pAur
  Rhinolophus_ferrumequinum mRhiFer1.5.pri.TOGA rFer
  Rhinolophus_hipposideros mRhiHip1.1.hap1.TOGA rHip
  Rhinolophus_sinicus mRhiSin3.1.pri.TOGA rSin
  Rousettus_aegyptiacus mRouAeg1.4.pri.TOGA rAeg
  Vespertilio_murinus mVesMur1.1.pri.TOGA vMur
  EOF
  
  echo "DONE"
