# Welcome to the CAMP!


The Core Analysis Modular Pipeline, the **CAMP**, is a software toolkit designed for dynamic and educational analyses of metagenomes, bacterial isolates, and, in general, all things microbial. The CAMP is broadly applicable and can be the main analytic workflow for many future projects. It is currently the primary analytic workflow for the Microbiome-in-a-Bottle project and the [MetaSUB Consortium](https://metasub.org>).

The core philosophy of the CAMP is anchored in **modularity**, which is meant to stand in stark contrast to the popular bioinformatic toolkits of "one-click pipelines." By defining every step in an analytic workflow as single, consistently documented and parameterized codebase, we aim to enable users to gain total control over and a deep understand of their bioinformatic analyses. 

<!--- Notifications about CAMP releases will be available through the [MetaSUB Twitter account](https://twitter.com/metasub?lang=en>). --->

Please post questions and issues related to CAMP tools on the GitHub repository of the specific module in question.

![Overview](https://github.com/Meta-CAMP/.github/blob/main/profile/Fig1.2.png)

An overview of the available metagenomics analysis modules in the Core Analysis Modular Pipeline (CAMP). All modules share the same internal architecture, but wrap a different set of algorithms (shown to the left of each box) customized to its particular analysis goals. Modules that are typically the beginning of analysis projects are coloured light blue, modules that are typically intermediate steps are coloured medium blue, and modules that are typically terminal analysis steps are coloured dark blue.

Note: The CAMP is under active development.

## Available Analysis Modules

### General-Purpose

#### [Short-read Preprocessing](https://github.com/Meta-CAMP/camp_short-read-quality-control)

Raw sequencing datasets are filtered for low-quality bases, low-complexity regions in reads, and extremely short reads using fastp (1). Reads can optionally be deduplicated. Filtered reads are trimmed of adapters using Trimmomatic (2). If host read removal is selected, trimmed and filtered reads are mapped using Bowtie2 and Samtools with the 'very-sensitive' flag to the host reference genome (here, the human reference genome assembly GRCh38), and mapped reads removed (3,4). As a last-pass, BayesHammer is used to correct sequencing errors (5). FastQC and MultiQC are used to generate overviews (ex. parameters such as per-base quality scores, sequence duplication levels) of processed dataset quality (6,7). 

#### [Short-read Assembly](https://github.com/Meta-CAMP/camp_short-read-assembly)

The processed sequencing reads can be assembled using MetaSPAdes (with optional flags for metaviral and/or plasmid assembly also available), MegaHIT, or both (8,9). Here, only MetaSPAdes was used. The assembly is subsequently summarized using MetaQUAST (10).

### MAG Inference and Quality-Checking

#### [MAG Binning](https://github.com/Meta-CAMP/camp_binning)

Processed sequencing reads are mapped back to the de novo assembled contigs using Bowtie2 and Samtools. This read coverage information, along with the contig sequences themselves, are used as input for the following binning algorithms: MetaBAT2, CONCOCT, SemiBin2, MaxBin2, VAMB, and MetaBinner (11 – 16). The sets of MAGs inferred by each algorithm are used as input for DAS Tool, an ensemble binning algorithm, to generate a set of consensus MAGs scored based on the presence/absence of single-copy genes (SCGs) (17). 

#### [MAG Quality-Checking](https://github.com/Meta-CAMP/camp_mag-qc)

The consensus refined MAGs are quality-checked using an array of parameters. CheckM2 calculates completeness, which is based on the number of lineage-specific marker gene sets present in a MAG, and contamination, which is the number of over-represented multiple copies of a marker gene in a MAG (18). gunc is also used to assess contamination (19). MAGs are classified using GTDB-Tk, which relies on approximately calculating average nucleotide identity (ANI) to a database of reference genomes (20). For MAGs with a species classification, their contig content is compared to the species' reference genome and genome-based completion, misassembly, and non-alignment statistics calculated using QUAST (21).
OTHER ANALYSIS GOALS

### Other Analysis Goals

#### [Short-read Taxonomic Classification](https://github.com/Meta-CAMP/camp_short-read-taxonomy)

The processed sequencing reads can be classified using MetaPhlan4, Kraken2/Bracken, and XTree (22 – 25). All three tools were used here. To estimate the relative abundance of a taxon, MetaPhlan4 calculates marker gene coverage, Bracken calculates the proportion of reads assigned to a taxon with k-mer uniqueness-based scaling, and XTree estimates directly from unique k-mer proportions. Since each of these output reports are of different formats, the raw reports from each algorithm are standardized in format for easier comparisons downstream.

#### [Viral Investigation](https://github.com/Meta-CAMP/camp_virus-phage-detect)

The processed sequencing reads are assembled with MetaSPAdes, and viral contigs are subsequently identified using the output assembly graph and ViralVerify (8,26). Contigs containing putative viral genetic material are also identified using VIBRANT, VirSorter, and VirFinder (27 – 29). The aggregated lists of contigs from the three inference algorithms is dereplicated using VirClust (30) and merged with the ViralVerify list, and the overall quality of the putative viruses is assessed using CheckV (31). 

#### [Gene Cataloguing](https://github.com/Meta-CAMP/camp_gene-catalog)

Open reading frames (ORFs) are identified in the de novo assembly using Bakta, and clustered using MMSeqs (32, 33). Genes are identified from these ORFs by alignment to the DIAMOND database to obtain the functional profile of the sample (34). 

#### [Nanopore Long-Read Quality Control](https://github.com/Meta-CAMP/camp_nanopore-quality-control)

Raw sequencing datasets are trimmed using PoreChop, and then low-quality bases are filtered out using NanoFilt (35, 36). Host reads are optionally removed using Minimap2 (37). FastQC and MultiQC are used to generate overviews (ex. parameters such as per-base quality scores, sequence duplication levels) of processed dataset quality (6,7). 

####  Decontamination

The decontamination module is still under construction and is not currently publicly available. A feature table of relative abundances (ex. operational taxonomic units (OTUs), taxa, metagenome-assembled genomes) is provided to Decontam and Recentrifuge, each of which estimates contamination from feature abundances either within or between samples respectively (38, 39).

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

CAMP: A modular metagenomics analysis system for integrated multi-step data exploration. Lauren Mak, Braden Tierney, Cynthia Ronkowski, Rodolfo Brizola Toscan, Berk Turhan, Michael Toomey, Juan Sebastian Andrade Martinez, Chenlian Fu, Alexander G Lucaci, Arthur Henrique Barrios Solano, João Carlos Setubal, James R Henriksen, Sam Zimmerman, Malika Kopbayeva, Anna Noyvert, Zana Iwan, Shraman Kar, Nikita Nakazawa, Dmitry Meleshko, Dmytro Horyslavets, Valeriia Kantsypa, Alina Frolova, Andre Kahles, David Danko, Eran Elhaik, Pawel Labaj, Serghei Mangul, The International MetaSUB Consortium, Christopher E. Mason, Iman Hajirasouliha. bioRxiv 2023.04.09.536171; doi: https://doi.org/10.1101/2023.04.09.536171
