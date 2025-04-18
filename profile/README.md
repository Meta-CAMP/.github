# Welcome to CAMP ⛺️


The Core Analysis Modular Pipeline, the **CAMP**, is a software toolkit designed for dynamic and educational analyses of metagenomes, bacterial isolates, and, in general, all things microbial. The CAMP is broadly applicable and can be the main analytic workflow for many future projects. It is currently the primary analytic workflow for the Microbiome-in-a-Bottle project and the [MetaSUB Consortium](https://metasub.org>).

The core philosophy of the CAMP is anchored in **modularity**, which is meant to stand in stark contrast to the popular bioinformatic toolkits of "one-click pipelines." By defining every step in an analytic workflow as single, consistently documented and parameterized codebase, we aim to enable users to gain total control over and a deep understand of their bioinformatic analyses. 

<!--- Notifications about CAMP releases will be available through the [MetaSUB Twitter account](https://twitter.com/metasub?lang=en>). --->

Please post questions and issues related to CAMP tools on the GitHub repository of the specific module in question.

![Overview](https://github.com/Meta-CAMP/.github/blob/main/profile/Fig1.3.png)

An overview of the available metagenomics analysis modules in the Core Analysis Modular Pipeline (CAMP). All modules share the same internal architecture, but wrap a different set of algorithms (shown to the left of each box) customized to its particular analysis goals. Modules that are typically the beginning of analysis projects are coloured light blue, modules that are typically intermediate steps are coloured medium blue, and modules that are typically terminal analysis steps are coloured dark blue. The CAMP is under active development!

## Available Analysis Modules

### General-Purpose

#### [Short-read Preprocessing](https://github.com/Meta-CAMP/camp_short-read-quality-control)

Raw sequencing datasets are filtered for low-quality bases, lowcomplexity regions in reads, extremely short reads, and adapter content using fastp [Chen et al., 2018]. Reads can optionally be deduplicated. If host read removal is selected, trimmed and filtered reads are mapped using Bowtie2 and Samtools with the ’verysensitive’ flag to the host reference genome (here, the human reference genome assembly GRCh38), and mapped reads removed [Langmead and Salzberg, 2012, Li et al., 2009]. As a last-pass, BayesHammer or Tadpole is used to correct sequencing errors [Nikolenko et al., 2013, Bushnell, 2014]. FastQC and MultiQC are used to generate overviews (e.g. parameters such as per-base quality scores, sequence duplication levels) of processed dataset quality [Andrews, 2010, Ewels et al., 2016].  

#### [Nanore Long-read Preprocessing](https://github.com/Meta-CAMP/camp_nanopore-quality-control)

Raw sequencing datasets are trimmed using PoreChop, and then low-quality bases are filtered out using chopper [Wick, 2018, De Coster and Rademakers 2023]. Host reads are optionally removed using Minimap2 [Li, 2018]. FastQC and MultiQC are used to generate overviews (e.g. parameters such as per-base quality scores, sequence duplication levels) of processed dataset quality [Andrews, 2010, Ewels et al., 2016].

### Assembly

#### [Short-read Assembly](https://github.com/Meta-CAMP/camp_short-read-assembly)

The processed sequencing reads can be assembled using MetaSPAdes (with optional flags for metaviral and/or plasmid assembly also available), MegaHIT, or both [Nurk et al., 2017, Li et al., 2015]. For the purposes of this study, only MetaSPAdes was used. The assembly is subsequently summarized using QUAST [Gurevich et al., 2013].  

#### [Hybrid Assembly](https://github.com/Meta-CAMP/camp_hybrid-assembly)

For short-read-first hybrid assembly, processed short sequencing reads are assembled with hybridmetaSPAdes, with the draft assembly polished by processed long reads [Antipov et al. 2016]. For long-readfirst hybrid assembly, processed long reads are assembled with MetaFlye [Kolmogorov et al. 2020], before being first being corrected with the long reads using Medaka and then with short reads using PolyPolish [medaka, Wick and Holt 2022]. The assemblies are subsequently summarized and compared using QUAST [Gurevich et al., 2013].  

### MAG Inference and Quality-Checking

#### [MAG Binning](https://github.com/Meta-CAMP/camp_binning)

Processed sequencing reads are mapped back to the _de novo_ assembled contigs using Bowtie2 and Samtools. This read coverage information, along with the contig sequences themselves, are used as input for the following binning methods: MetaBAT2, CONCOCT, SemiBin, MaxBin2, VAMB, and MetaBinner [Kang et al., 2019, Alneberg et al., 2014, Pan et al., 2023, Nissen et al., 2021, Wu et al., 2015, Wang et al., 2023]. The sets of MAGs inferred by each algorithm are used as input for DAS Tool, an ensemble binning methods, to generate a set of consensus MAGs scored based on the presence/absence of single-copy genes (SCGs) [Sieber et al., 2017]. Some of the contig pre-processing scripts were adapted from the MAG Snakemake workflow [Saheb Kashaf et al., 2021].

#### [MAG Quality-Checking](https://github.com/Meta-CAMP/camp_mag-qc)

The consensus refined MAGs are quality-checked using an array of parameters. CheckM2 calculates completeness and contamination based on the annotated gene content of a MAG [Chklovski et al., 2023]. CheckM1 calculates per-MAG short-read coverage and strain heterogeneity based on the proportion of presence of multiple copies of single-copy marker genes that pass a high amino acid identity threshold [Parks et al., 2015]. gunc is also used to assess contamination [Orakov et al., 2021]. MAGs are classified using GTDB-Tk, which relies on approximately calculating average nucleotide identity (ANI) to a database of reference genomes [Chaumeil et al., 2019]. For MAGs with a species classification, their contig content is compared to the species’ reference genome and genome-based completion, misassembly, and non-alignment statistics calculated using dnadiff and QUAST [Marçais et al., 2018, Gurevich et al., 2013]. Each MAG’s gene content, with an emphasis on tRNA and rRNA genes in accordance with MIMAG genome reporting standards [Bowers et al., 2017], is summarized using prokka [Seemann et al. 2014]. 

The overall score per MAG was calculated using CheckM2- calculated completeness and contamination, and the contiguity metric N50: completeness − 5 × contamination + 0.5 × log(N50). This equation was adapted from dRep’s overall score equation, which is based on completeness, contamination, contiguity, strain heterogeneity, and genome size, and [Olm et al., 2017]. dRep used this score to select a representative genome from a cluster of genomes with similar sequences. Various versions of this equation exist, with most variants setting the coefficients for strain heterogeneity and genome size to 0 [Williams et al., 2024, Gurbich et al., 2023, Parks et al., 2017].  

### Other Analysis Goals

#### [Short-read Taxonomic Classification](https://github.com/Meta-CAMP/camp_short-read-taxonomy)

The processed sequencing reads can be classified using MetaPhlan4, Kraken2/Bracken, and/or XTree [Wood et al., 2019, Lu et al., 2017, Blanco-Miguez et al., 2022, GabeAl, 2022]. XTree, formerly referred to as UTree, is additionally included as an experimental shortread classification option, but this study focuses on comparing the taxa discovered by the two published field-standard tools- Kraken2 and MetaPhlan4. To estimate the relative abundance of a taxon, MetaPhlan4 calculates marker gene coverage and Bracken calculates the proportion of reads assigned to a taxon with k-mer uniquenessbased scaling [Blanco-Miguez et al., 2022]. Since each of these output reports are of different formats, the raw reports from each algorithm are standardized in format for easier comparisons downstream.  

#### [Viral Investigation](https://github.com/Meta-CAMP/camp_virus-phage-detect)

The processed sequencing reads are assembled with MetaSPAdes, and viral contigs are subsequently identified using the output assembly graph and ViralVerify [Nurk et al., 2017, Antipov et al., 2020]. Contigs containing putative viral genetic material are also identified using VIBRANT, VirSorter2, DeepVirFinder, and geNomad [Kieft et al., 2020, Roux et al., 2015, Ren et al. 2021, Camargo et al. 2023]. The aggregated lists of contigs from the three inference methods is dereplicated using VirClust [Moraru, 2023] and merged with the ViralVerify list, and the overall quality of the putative viruses is assessed using CheckV [Nayfach et al., 2020].  

#### [Gene Cataloguing](https://github.com/Meta-CAMP/camp_gene-catalog)

Open reading frames (ORFs) are identified in the de novo assembly using Bakta, and clustered using MMSeqs [Schwengers et al., 2021, Hauser et al., 2016]. Genes are identified from these ORFs by alignment to the DIAMOND database to obtain the functional profile of the sample [Buchfink et al., 2014]. 

#### Decontamination

The decontamination module is still under construction and is currently not publicly available. A feature table of relative abundances (e.g. operational taxonomic units (OTUs), taxa, metagenome-assembled genomes) is provided to Decontam and Recentrifuge, each of which estimates contamination from feature abundances either within or between samples respectively [Davis et al., 2018, Martí, 2019].  

#### [Long-Read Profiling](https://github.com/Meta-CAMP/camp_motus-profiler)

The profiling module wraps mOTUs [Ruscheweyh et al., 2021], which estimates the relative abundance of taxa from a short- or long-read sequencing dataset, and calls single-nucleotide variants from marker genes. 

## Making New Analysis Modules

For full instructions on how to set up custom CAMP-style modules, please see the [module template](https://github.com/Meta-CAMP/CAMP_Module_Template).

## The CAMP Core Principles

The CAMP is meant as an alterative to one-click approaches, built on three core principles.

1) One module, one job, one output

Going from short reads to, say, binned Metagenome-Assembled-Genomes, requires many intermediate steps and file types. This means that, in a single pipeline if one software dependency breaks, if a given user has an incompatible system with just one underlying tool, if one bug pops up in the code, the whole thing can fall apart.

Each module executes a single analytic task and provides the user with fully flexible parameters. Most run multiple different pieces of software, encouraging comparisons in how, say, different taxonomic profilers can yield different results.

Additionally, every module takes a standardized set of inputs and outputs, allowing them to be easily strung together.

2) Designed for algorithmic understanding

One of the first steps in using a module is manually setting the parameters.yaml file. While an extra bit of effort, this encourages the user to think about what they’re running, instead of just pushing go. We’ve tried to walk the line between ease of use and encouraging understanding of the underlying process.

<!--- As part of this, we are going to be implementing extremely substantive documentation for each module. Every README will, upon release of the full CAMP, have a “Theory” section that describes how the algorithms implemented at a certain section work. Ideally, the CAMP should equate to a semester long course in metagenomic analysis, with enough rich detail to take someone with minimal command-line experience all the way to a competent analyst. --->

3) Flexible development

Who are we to presume what your needs are, bioinformatically speaking. By separating tasks into modules, we’ve aimed to generate a toolkit that is maximally flexible. Further, if you need to build something else, constructing a new module based on existing pieces of software takes only a couple of hours for an experienced developer. This is in large-part due to our automated module-structure generation that every repository uses. Once you understand how one module is put together, you understand them all.

## Citing the CAMP

If you use the CAMP, please cite it as below, as well as the software it wraps! For a list of the software, please see the Methods section in the manuscript.

CAMP: A modular metagenomics analysis system for integrated multi-step data exploration. Lauren Mak, Braden Tierney, Wei Wei, Cynthia Ronkowski, Rodolfo Brizola Toscan, Berk Turhan, Michael Toomey, Juan Sebastian Andrade Martinez, Chenlian Fu, Alexander G Lucaci, Arthur Henrique Barrios Solano, João Carlos Setubal, James R Henriksen, Sam Zimmerman, Malika Kopbayeva, Anna Noyvert, Zana Iwan, Shraman Kar, Nikita Nakazawa, Dmitry Meleshko, Dmytro Horyslavets, Valeriia Kantsypa, Alina Frolova, Andre Kahles, David Danko, Eran Elhaik, Pawel Labaj, Serghei Mangul, The International MetaSUB Consortium, Christopher E. Mason, Iman Hajirasouliha. bioRxiv 2023.04.09.536171; doi: https://doi.org/10.1101/2023.04.09.536171
