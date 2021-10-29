# ColabFold
-----------------
**MMseqs2 API status**
```diff
  16Aug2021: WARNING - MMseqs2 API is undergoing upgrade, you may see error messages.
  17Aug2021: If you see any errors, please report them.
  17Aug2021: We are still debugging the MSA generation procedure...
  20Aug2021: WARNING - MMseqs2 API is undergoing upgrade, you may see error messages.
             To avoid Google Colab from crashing, for large MSA we did -diff 1000 to get 
             1K most diverse sequences. This caused some large MSA to degrade in quality,
             as sequences close to query were being merged to single representive.
             We are working on updating the server (today) to fix this, by making sure
             that both diverse and sequences close to query are included in the final MSA.
             We'll post update here when update is complete.
+ 21Aug2021  The MSA issues should now be resolved! Please report any errors you see.
+            In short, to reduce MSA size we filter (qsc > 0.8, id > 0.95) and take 3K
+            most diverse sequences at different qid (sequence identity to query) intervals 
+            and merge them. More specifically 3K sequences at qid at (0→0.2),(0.2→0.4),
+            (0.4→0.6),(0.6→0.8) and (0.8→1). If you submitted your sequence between
+            16Aug2021 and 20Aug2021, we recommend submitting again for best results!
  21Aug2021  The use_templates option in AlphaFold2_mmseqs2 is not properly working. We are
             working on fixing this. If you are not using templates, this does not affect the
             the results. Other notebooks that do not use_templates are unaffected.
+ 21Aug2021  The templates issue is resolved!
```
-----------------

<p align="center"><img src="https://github.com/sokrypton/ColabFold/raw/main/.github/ColabFold_Marv_Logo.png" height="250"/></p>

### Making Protein folding accessible to all via Google Colab!

| Notebooks | monomers | complexes | mmseqs2 | jackhmmer | templates   |
| :-------- | -------  | --------- | ------- | --------- | ----------- |
| [AlphaFold2_mmseqs2](https://colab.research.google.com/github/sokrypton/ColabFold/blob/main/AlphaFold2.ipynb) | Yes | No | Yes | No | Yes | 
| [AlphaFold2_advanced](https://colab.research.google.com/github/sokrypton/ColabFold/blob/main/beta/AlphaFold2_advanced.ipynb) | Yes | Yes | Yes | Yes | No |
| [AlphaFold2_batch](https://colab.research.google.com/github/sokrypton/ColabFold/blob/main/batch/AlphaFold2_batch.ipynb) | Yes | No | Yes | No | Yes | 
| [RoseTTAFold](https://colab.research.google.com/github/sokrypton/ColabFold/blob/main/RoseTTAFold.ipynb) | Yes | No | Yes | No | No | 
| [AlphaFold2](https://colab.research.google.com/github/deepmind/alphafold/blob/main/notebooks/AlphaFold.ipynb) (from Deepmind) | Yes | No | No | Yes | No | 
||
| **OLD retired notebooks** | | | | | |
| [AlphaFold2_complexes](https://colab.research.google.com/github/sokrypton/ColabFold/blob/main/AlphaFold2_complexes.ipynb) | No | Yes | No | No | No | 
| [AlphaFold2_jackhmmer](https://colab.research.google.com/github/sokrypton/ColabFold/blob/main/beta/AlphaFold_wJackhmmer.ipynb) | Yes | No | Yes | Yes | No |
| [AlphaFold2_noTemplates_noMD](https://colab.research.google.com/github/sokrypton/ColabFold/blob/main/verbose/alphafold_noTemplates_noMD.ipynb) |
| [AlphaFold2_noTemplates_yesMD](https://colab.research.google.com/github/sokrypton/ColabFold/blob/main/verbose/alphafold_noTemplates_yesMD.ipynb) |


### FAQ
- Can I use the models for **Molecular Replacement**?
  - Yes, but be **CAREFUL**, the bfactor column is populated with pLDDT confidence values (higher = better). Phenix.phaser expects a "real" bfactor, where (lower = better). See [post](https://twitter.com/cheshireminima/status/1423929241675120643) from Claudia Millán.
- What is the maximum length?
  - Limits depends on free GPU provided by Google-Colab `fingers-crossed`
  - For GPU: `Tesla T4` or `Tesla P100` with ~16G the max length is ~1400
  - For GPU: `Tesla K80` with ~12G the max length is ~1000
  - To check what GPU you got, open a new code cell and type `!nvidia-smi`
- Is it okay to use the MMseqs2 MSA server (`cf.run_mmseqs2`) on a local computer?
  - You can access the server from a local computer if you queries are serial from a single IP. Please do not use multiple computers to query the server.
- Where can I download the databases used by ColabFold?
  - The databases are available [here](https://colabfold.mmseqs.com/)

### Running locally

Please checkout the [jax documentation](https://github.com/google/jax#pip-installation-gpu-cuda) for how to make jax work on your GPU or TPU

```shell
pip install "colabfold[alphafold] @ git+https://github.com/konstin/ColabFold"
pip install --upgrade "jax[cuda]" -f https://storage.googleapis.com/jax-releases/jax_releases.html  # Note: wheels only available on linux.
```

```shell
colabfold_batch <directory_with_fasta_files> <result_dir> 
```

### Generating MSAs

First create a directory for the databases on a disk with sufficient storage (940GB (!)). Depending on where you are, this will take a couple of hours: 

```shell
./setup_databases.sh /path/to/db_folder
```

Download and unpack mmseqs (Note: The required features aren't in a release yet, so currently, you need to compile the latest version from source yourself). If mmseqs is not in your `PATH`, replace `mmseqs` below with the path to your mmseqs:

```shell
# This needs a lot of CPU
colabfold_search.sh mmseqs input_sequences.fasta /path/to/db_folder search_results uniref30_2103_db "" colabfold_envdb_202108_db 1 0 1
# This just does a bit of IO
python /home/konsti/ColabFold/colabfold/merge_and_split_msas.py search_results msas
# This needs a GPU
colabfold_batch msas predictions
```

This will create intermediate folders `search_results` and `msas` that you can eventually delete, and a `predictions` folder with all pdb files. 

### Tutorials & Presentations
- ColabFold Tutorial presented at the Boston Protein Design and Modeling Club. [[video]](https://www.youtube.com/watch?v=Rfw7thgGTwI) [[slides]](https://docs.google.com/presentation/d/1mnffk23ev2QMDzGZ5w1skXEadTe54l8-Uei6ACce8eI). 

### Projects based on ColabFold

- [Run ColabFold (AlphaFold2_advanced) on your local computer](https://github.com/YoshitakaMo/localcolabfold) by Yoshitaka Moriwaki
- [Colab for protein structure predictions for Discoba species](https://github.com/zephyris/discoba_alphafold) by Richard John Wheeler
- [Cloud-based molecular simulations for everyone](https://github.com/pablo-arantes/Making-it-rain) by Pablo R. Arantes, Marcelo D. Polêto, Conrado Pedebos and Rodrigo Ligabue-Braun
- [getmoonbear is a webserver to predict protein structures](https://www.getmoonbear.com/AlphaFold2) by Stephanie Zhang and Neil Deshmukh
- [AlphaFold2 IDR complex prediction](https://github.com/normandavey/AlphaFold2-IDR-complex-prediction) by Balint Meszaros

### Acknowledgments
- We would like to thank the [RoseTTAFold](https://github.com/RosettaCommons/RoseTTAFold) and [AlphaFold](https://github.com/deepmind/alphafold) team for doing an excellent job open sourcing the software. 
- Also credit to [David Koes](https://github.com/dkoes) for his awesome [py3Dmol](https://3dmol.csb.pitt.edu/) plugin, without whom these notebooks would be quite boring!
- A colab by Sergey Ovchinnikov (@sokrypton), Milot Mirdita (@milot_mirdita) and Martin Steinegger (@thesteinegger).

### How do I reference this work?

- Mirdita M, Ovchinnikov S and Steinegger M. ColabFold - Making protein folding accessible to all. <br />
  bioRxiv (2021) doi: [10.1101/2021.08.15.456425](https://www.biorxiv.org/content/10.1101/2021.08.15.456425v1)
- If you’re using **AlphaFold**, please also cite: <br />
  Jumper et al. "Highly accurate protein structure prediction with AlphaFold." <br />
  Nature (2021) doi: [10.1038/s41586-021-03819-2](https://doi.org/10.1038/s41586-021-03819-2)
- If you are using **RoseTTAFold**, please also cite: <br />
  Minkyung et al. "Accurate prediction of protein structures and interactions using a three-track neural network." <br />
  Science (2021) doi: [10.1126/science.abj8754](https://doi.org/10.1126/science.abj8754)

[![DOI](https://zenodo.org/badge/doi/10.5281/zenodo.5123296.svg)](https://doi.org/10.5281/zenodo.5123296)
