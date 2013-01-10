# MITObim
=======

VERSION
=======
1.3

Copyright 2012 - Christoph Hahn

CONTACT
=======

Christoph Hahn <christoph.hahn@nhm.uio.no>

INTRODUCTION
============

This document contains instructions on how to use the MITObim pipeline described in the manuscript "Reconstructing mitochondrial genomes directly from genomic next generation sequencing reads - a baiting and iterative mapping approach" by Hahn et al., submitted to NAR methods online. The pipeline is at the moment intended to be used with illumina data, but can be readily modified for the use with other platforms data. The latest version of the wrapper script (including the proofreading function) has been uploaded 31.12.2012. The tutorial will be updated accordingly early 2013. 


PREREQUISITES
=============
- GNU utilities
- perl
- A running version of MIRA v3.4.1.1 (http://sourceforge.net/projects/mira-assembler/files/MIRA/stable/) is required. An excellent guide to MIRA v3.4 is available at http://mira-assembler.sourceforge.net/docs/DefinitiveGuideToMIRA.html.


WALKTHROUGH MITObim
===================

1. Inititial mapping assembly to (distant) reference using MIRA v3.4:

As part of the iterative baiting and mapping (MITObim) procedure one has to prepare a starting reference. This will be derived from the conserved regions of a (distantly) related mitochondrial genome. Your working directory must contain at least two things: (1) A mitochondrial genome of a (distantly) related species in fasta format (e.g. obtained from Genbank) named MYPROJECT_backbone_in.fasta. (2) Illumina data in fastq format and named MYPROJECT_in_solexa.fastq. A real data example is demonstrated in the testdata section.

For more detailed information on Solexa mapping assemblies with MIRA 3.4 see http://mira-assembler.sourceforge.net/docs/DefinitiveGuideToMIRA.html#sect_sxa_mapping_assemblies.

NOTE: This step might require substantial computational resources depending on the size of the illumina readpool (Memory requirements can be estimated using the miramem program shipping with MIRA). This increased memory consumption can be bypassed by limitating the readpool e.g. via an initial in-silico baiting step using mirabait (in-silico baiting program shipping with MIRA). While this strategy will impair the sensitivity of the approach, it works reasonably well when a not to distantly related reference is available. This method is included in the provided wrapper script and can be performed by using the --quick flag, together with providing a reference sequence in fasta format. A detailed example can be found in the testdata section (Tutorial II).

2. run MITObim.pl script:

usage: ./MITObim.pl <parameters>


parameters:

                -start <int>            iteration to start with, default=1
                -end <int>              iteration to end with, default=1
                -strain <string>        strainname as used in initial MIRA assembly
                -ref <string>           referencename as used in initial MIRA assembly
                -readpool <PATH>        path to readpool in fastq format
                -maf <PATH>             path to maf file from previous MIRA assembly


optional:

                --denovo                runs MIRA in denovo mode, default: mapping
                --pair                  finds pairs after baiting, default: no
                --quick <PATH>          starts process with initial baiting using provided fasta reference
                --noshow                do not show output of MIRA modules
                --help                  shows this information
                --proofread             applies proofreading
                --readlength <int>      read length of illumina library, default=150
                --insert <int>          insert size of illumina library, default=300

examples:

                ./MITObim.pl -start 1 -end 5 -strain StrainX -ref Gthymalli-mt -readpool /PATH/TO/readpool.fastq -maf /PATH/TO/assembly.maf
                ./MITObim.pl --quick /PATH/TO/reference.fasta -strain StrainY -ref Gthymalli-mt -readpool /PATH/TO/readpool.fastq
The script is performing three steps and iteratively repeating them: (i) Deriving reference sequence from privious mapping assembly, (ii) in silico baiting using the newly derived reference (iii) previously fished reads are mapped to the newly derived reference leading to an extension of the reference sequence. 

TESTDATA
========
The following tutorials are referring to MITObim v1.1. The latest version v1.3 has a new proofreading function implemented. Tutorials will be updated for version 1.3 in early 2013!!

The following tutorials will demonstrate how to recover the complete mitochondrial genome of T. thymallus using the mitochondrial genome of O. mykiss as a starting reference (assuming little Unix and no previous MIRA experience).

- download the MITObim wrapper script MITObim.pl and make it executable (chmod a+x MITObim.pl)
- download testdata.tgz and extract the contents (tar xvfz testdata.tgz)

The provided archive (testdata.tgz) contains two files:
testpool.fastq - subset of the dataset "thy" discussed in Hahn et al., containing the entire mitochondrial readpool of Thymallus thymallus along with a number of random reads.
O_mykiss_NC_001717.fasta - mitochondrial genome of Oncorhynchus mykiss in fasta format downloaded from Genbank.

Tutorial I:
-----------
1. Initial mapping assembly using MIRA v3.4:

Create a directory to work in and change into this directory:
-bash-4.1$ mkdir work
-bash-4.1$ cd work

The project name of the initial mapping assembly will be "initial-mapping-testpool-to-Omykiss-mt". Copy the readpool and the reference from the testdata directory to your work directory and rename them so that MIRA will recognize the files.
-bash-4.1$ cp /PATH/TO/testdata/testpool.fastq initial-mapping-testpool-to-Omykiss-mt_in.solexa.fastq 
-bash-4.1$ cp /PATH/TO/testdata/O_mykiss_NC_001717.fasta initial-mapping-testpool-to-Omykiss-mt_backbone_in.fasta

Run a mapping assembly (details on the options can be found at http://mira-assembler.sourceforge.net/docs/DefinitiveGuideToMIRA.html):
-bash-4.1$ mira --project=initial-mapping-testpool-to-Omykiss-mt --job=mapping,genome,accurate,solexa -MI:somrnl=0 -AS:nop=1 -SB:bft=fasta:bbq=30:bsn=Omykiss-mt-genome SOLEXA_SETTINGS -CO:msr=no -SB:dsn=testpool 

Look what happened:
-bash-4.1$ ls -hlrt
total 2.8M
-rw-r--r-- 1 chrishah users  17K Oct 21 22:50 initial-mapping-testpool-to-Omykiss-mt_backbone_in.fasta
-rw-r--r-- 1 chrishah users 2.7M Oct 21 22:51 initial-mapping-testpool-to-Omykiss-mt_in.solexa.fastq
drwxr-xr-x 6 chrishah users    4 Oct 21 22:51 initial-mapping-testpool-to-Omykiss-mt_assembly

The successfull MIRA instance created a directory "initial-mapping-testpool-to-Omykiss-mt_assembly", containing in turn four directories:
-bash-4.1$ cd initial-mapping-testpool-to-Omykiss-mt_assembly/
-bash-4.1$ ls -hlrt
total 2.0K
drwxr-xr-x 2 chrishah users  2 Oct 21 22:51 initial-mapping-testpool-to-Omykiss-mt_d_chkpt
drwxr-xr-x 2 chrishah users 15 Oct 21 22:51 initial-mapping-testpool-to-Omykiss-mt_d_tmp
drwxr-xr-x 2 chrishah users  9 Oct 21 22:51 initial-mapping-testpool-to-Omykiss-mt_d_results
drwxr-xr-x 2 chrishah users  8 Oct 21 22:51 initial-mapping-testpool-to-Omykiss-mt_d_info
 
All that is needed for the subsequent MITObim process is "initial-mapping-testpool-to-Omykiss-mt_out.maf" contained in the "initial-mapping-testpool-to-Omykiss-mt_d_results" directory.

2. Baiting and iterative mapping using the MITObim.pl script: 

Run the wrapper script:
-bash-4.1$ /PATH/TO/./MITObim.pl -start 1 -end 10 -strain testpool -ref O_mykiss_NC_001717 -readpool /PATH/TO/testpool.fastq -maf /PATH/TO/initial-mapping-testpool-to-Omykiss-mt_out.maf 

NOTE: 
- The strain name needs to be the same as used in the initial MIRA assembly.
- The full paths are required to the readpool and the maf file.

This will create 10 directories: 
-bash-4.1$ ls -hlrt
total 5.0K
drwxr-xr-x 3 chrishah users 4 Oct 21 23:45 iteration1
drwxr-xr-x 3 chrishah users 4 Oct 21 23:45 iteration2
drwxr-xr-x 3 chrishah users 4 Oct 21 23:45 iteration3
drwxr-xr-x 3 chrishah users 4 Oct 21 23:45 iteration4
drwxr-xr-x 3 chrishah users 4 Oct 21 23:45 iteration5
drwxr-xr-x 3 chrishah users 4 Oct 21 23:45 iteration6
drwxr-xr-x 3 chrishah users 4 Oct 21 23:45 iteration7
drwxr-xr-x 3 chrishah users 4 Oct 21 23:46 iteration8
drwxr-xr-x 3 chrishah users 4 Oct 21 23:46 iteration9
drwxr-xr-x 3 chrishah users 4 Oct 21 23:46 iteration10

Each directory is organized as the working directory above (initial mapping assembly), containing a MIRA assembly directory together with the referernce and readpool used in the respective iteration:
-bash-4.1$ cd iteration1
-bash-4.1$ ls -hlrt
total 2.3M
-rw-r--r-- 1 chrishah users  17K Oct 21 23:45 testpool-O_mykiss_NC_001717_backbone_in.fasta
-rw-r--r-- 1 chrishah users 383K Oct 21 23:45 hashstat.bin
-rw-r--r-- 1 chrishah users 1.9M Oct 21 23:45 testpool-O_mykiss_NC_001717_in.solexa.fastq
drwxr-xr-x 6 chrishah users    4 Oct 21 23:45 testpool-O_mykiss_NC_001717_assembly

If you examine the results you will find the complete mitochondrial genome of T.thymallus in any iteration directory subsequent to iteration 4. The number of identified mitochondrial reads remains stationary from iteration 4 on. Congratulations!

Tutorial II:
------------
The following tutorial bypasses the initial mapping assembly (which might require increased computational resources depending on the size of the initial readpool) by starting the process with an initial in silico baiting step. While this strategy will impair the sensitivity of the approach, it works reasonably well when a not too distantly related reference is available.

Run the MITObim.pl script with the --quick option, providing a reference in fasta format:
-bash-4.1$ /PATH/TO/./MITObim.pl -start 1 -end 20 -strain testpool -ref O_mykiss_NC_001717 -readpool ~/MITOBIM-perl/test/testdata/testpool.fastq --quick ~/MITOBIM-perl/test/testdata/O_mykiss_NC_001717.fasta

You will find that with this approach MITObim will find the mitochondrial genome and a stationary number of mitochondiral reads only after 19 iterations.
