# AMAS
Alignment manipulation and summary statistics

If you are using this program, please cite [this publication](https://peerj.com/articles/1660/):

```
Borowiec, M.L. 2016. AMAS: a fast tool for alignment manipulation and computing of summary statistics. PeerJ 4:e1660.
```
## This fork: AMAS_JGLAHE
Fork of the main repo which:
- Adds a `metapartition` command -> collates discontinuous metapartitions within a superalignment and concatenates them into a new superalignment of contiguous metapartitons.
- Reduces restrictions on input partition file formatting -> accepts partition files for RAxML(-NG) and IQ-TREE2 (best_scheme, best_scheme.nex and best_model.nex).

## Installation

Use `AMAS.py` in the `amas` directory as a stand-alone program or clone it if you have [git installed](http://git-scm.com/book/en/v2/Getting-Started-Installing-Git) on your system.

If your system doesn't have a Python version 3.4 or newer (`AMAS` will work under Python 3.0 but you may noy be able to use it with multiple cores), you will need to [download and install it](http://www.python.org/downloads/). On Linux-like systems (including Ubuntu) you can install it from the command line using

It may be possible to use this version as a module, but only through manual configuration.

# Command line interface
`AMAS` can be run from the command line. Here is the general usage (you can view this in your command line with `python3 AMAS.py -h`):

```
usage: AMAS <command> [<args>]

The AMAS commands are:
  concat            Concatenate input alignments.
  convert           Convert to other file format.
  replicate         Create replicate data sets for phylogenetic jackknife.
  split             Split alignment according to a partitions file.
  summary           Write alignment summary.
  remove            Remove taxa from alignment.
  translate         Translate DNA alignment into protein alignment.
  trim              Remove columns from alignment.
  metapartitions    Runs `split` and concatenates the output.

Use AMAS <command> -h for help with arguments of the command of interest

positional arguments:
  command     Subcommand to run

optional arguments:
  -h, --help  show this help message and exit
```

To show help for individual commands, use `AMAS.py <command> -h` or `AMAS.py <command> --help`.

## Examples
For every `AMAS.py` run on the command line you need to specify action with `concat`, `convert`, `replicate`, `split`, or `summary` for the input to be processed.
Additionally, you need to provide three arguments required for all commands. The order in which the arguments are given does not matter:

1) input file name(s) with `-i` (or in long version: `--in-files`),

2) format with `-f` (`--in-format`),

3) and data type with `-d` (`--data-type`). 

The options available for the format are `fasta`, `phylip`, `nexus` (sequential), `phylip-int`, and `nexus-int` (interleaved). Data types are `aa` for protein alignments and `dna` for nucleotide alignments.

For example:
```
python3 AMAS.py concat -i gene1.nex gene2.nex -f nexus -d dna
```

If you have many files that you want to input in one run, you can use multiple cores of your computer to process them in parallel. The `summary` command supports `-c` or `--cores` with which you can specify the number of cores to be used:

```
python3 AMAS.py summary -f phylip -d dna -i *phy -c 12
```
In the above, we specified 12 cores. Note that this won't improve computing time if you're working with only one or very few files. The parallel processing is only used for the file parsing step and calculating alignment summaries.

In addition to overall alignment summaries, you can also print statistics calculated on a sequence (taxon) by sequence basis. Use `-s` or `--by-taxon` flag to turn it on. `AMAS` in this mode will print out one file with overall alignment summaries and a file with taxon summaries for each input alignment.

IMPORTANT! `AMAS` is fast and powerful, but be careful: it assumes you know what you are doing and will not prevent you overwriting a file. It will, however, print out a warning if this has happened. `AMAS` was also written to work with aligned data and some of the output generated from unaligned sequences won't make sense. Because of computing efficiency `AMAS` by default does not check if input sequences are aligned. You can turn this option on with `-e` or `--check-align`.

### Concatenating alignments
For example, if you want to concatenate all DNA phylip files in a directory and all of them have the `.phy` extension, you can run:
```
python3 AMAS.py concat -f phylip -d dna -i *phy
```
By default the output will be written to two files: `partitions.txt`, containing partitions from which your new alignment was constructed, and `concatenated.out` with the alignment itself in the fasta format. You can change the default names for these files with `-p` (`--concat-part`) and `-t` (`--concat-out`), respectively, followed by the desired name. The output format is specified by `-u` (`--out-format`) and can also be any of the following: `fasta`, `phylip`, `nexus` (sequential), `phylip-int`, or `nexus-int` (interleaved).

Below is a command specifying the concatenated file output format as nexus with `-u nexus`:
```
python3 AMAS.py concat -f fasta -d aa -i *fas -u nexus
```
Alignments to be concatenated need not have identical sets of taxa before processing: the concatenated alignment will be populated with missing data where a given locus is missing a taxon. However, if every file to be concatenated includes only unique names (for example species name plus sequence name: `D_melanogaster_NW_001845408.1` in one alignment, `D_melanogaster_NW_001848855.1` in other alignment etc.), you will first need to trim those names so that sequences from one taxon have equivalents in all files.   
In addition to the name, you can also specify the format of the partitions output file. By default, the format is the following:
```
AA = 1-605
AK = 606-1200
28S = 1201-1800
```
RAxML:
```
python3 AMAS.py concat -f phylip -d dna -i *phy --part-format raxml
```
```
DNA, AA = 1-605
DNA, AK = 606-1200
DNA, 28S = 1201-1800
```
Nexus:
```
python3 AMAS.py concat -f phylip -d dna -i *phy --part-format nexus
```
```
#NEXUS
Begin sets;
    charset AA = 1-605;
    charset AK = 606-1200;
    charset 28S = 1201-1800;
End;
```
Partitions can also be written by codon positions using the `-n` or `--codons` flag, either for alignments containing first and second or all three positions. In the above example, supplying `-n 123` would result in:
```
AA_pos1 = 1-605\3
AA_pos2 = 2-605\3
AA_pos3 = 3-605\3
AK_pos1 = 606-1200\3
AK_pos2 = 607-1200\3
AK_pos3 = 608-1200\3
28S_pos1 = 1201-1800\3
28S_pos2 = 1202-1800\3
28S_pos3 = 1203-1800\3
```
### Getting alignment statistics
This is an example of how you can summarize two protein fasta alignments by running:
```
python3 AMAS.py summary -f fasta -d aa -i my_aln.fasta my_aln2.fasta
```
By default `AMAS` will write a file with the summary of the alignment in `summary.txt`. You can change the name of this file with `-o` or `--summary-out`. You can also summarize a single or multiple sequence alignments at once. 

The statistics calculated include the number of taxa, alignment length, total number of matrix cells, overall number of undetermined characters, percent of missing data, AT and GC contents (for DNA alignments), number and proportion of variable sites, number and proportion of parsimony informative sites, and counts of all characters present in the relevant alphabet.

### Converting among formats
To convert all nucleotide fasta files with a `.fas` extension in a directory to nexus alignments, you could use:
```
python3 AMAS.py convert -d dna -f fasta -i *fas -u nexus
```
In the above, the required options are combined with `convert` command to convert the input files and `-u nexus` which indicates the output format.

`AMAS` will not overwrite over input here but will create new files instead, automatically appending appropriate extensions to the input file's name: `-out.fas`, `-out.phy`, `-out.int-phy`, `-out.nex`, or `-out.int-nex`.

### Splitting alignment by partitions (updated partition parsing AMAS_JGLAHE) 
If you have a partition file, you can split a concatenated alignment and write a file for each partition:
```
python3 AMAS.py split -f nexus -d dna -i concat.nex -l partitions.txt -u nexus
```
In the above one input file `concat.nex` was provided for splitting with `split` and partitions file `partitions.txt` with `-l` (same as `--split-by`). For splitting you should only use one input and one partition file at a time. This is an example partition file:
```
  AApos1&2  =  1-604\3, 2-605\3
  AApos3  =  3-606\3
  28SAutapoInDels=7583, 7584, 7587, 7593
```
If this was the `partitions.txt` file from the example command above, `AMAS` would write three output files called `concat_AApos1&2.nex`, `concat_AApos3.nex`, and `concat_28SautapoInDels.nex`. The partitions file will be parsed correctly as long as there ~~is~~ are none ~~text prior to the partition name (`CHARSET AApos1&2` or `DNA, AApos1&2` will not work) and commas separate ranges or individual sites in each partition~~ of the following characters in partition names: `<>!?$^;:\/,.`; members of [\s] are also not allowed in partition names.

Sometimes after splitting you will have alignments with taxa that have only gaps `-` or missing data `?`. If you want to these to not be included in the output , add `-j` or `--remove-empty` to the command line.

Partition files are parsed with `AMAS.FileParser.partition_parse()`, which in the JGLAHE fork of AMAS looks like this:
```
def partitions_parse(self):
    # parse partitions file using regex
    # original: `matches = re.finditer(r"^(\s+)?([^ =]+)[ =]+([\\0-9, -]+)", self.in_file_lines, re.MULTILINE)`
    # new version: more permissive -> handles PartionFinder/RAxML/(IQ-TREE 2)best_scheme.nex format partition files
    matches = re.finditer(
        r"""^[ \t]*                                 # start of line w/ zero-or-more (just) whitespaces/tabs
            (
             (?P<nexus>charset[ ]+)                 # case 1: (IQ-TREE 2)best_scheme.nex partition directive; partition name
             |
             (?P<raxml>[A-Za-z0-9_+\.]+,[ \t]+)     # case 2: RAxML/RAxML-NG model(+other pars); partition name
            )?
            (?P<partition_name>[A-Za-z0-9_@*&()\[\]{}'"%#|-]+) # partition name (handles various metcharacters, excludes [\s],<>?/:;~`\^$!)
            [ ]*=[ ]*                               # whitespace-padded (or unpadded) '=': (IQ-TREE 2)best_scheme.nex compatabiliy
            (?P<numbers>[\\0-9, -]+)                # position ranges w/ stride (multiple intervals; from original regex)
            (?P<nexus_term>[ ]*[;])?                # whitespace-prepended (or unprepended) ';' (nexus terminator)
        """,
        self.in_file_lines,
        re.MULTILINE | re.VERBOSE
    )

    # initiate list to store dictionaries with lists
    # of slice positions as values
    partitions = []
    add_to_partitions = partitions.append

    for match in matches:
        # initiate dictionary of partition name as key
        dict_of_dicts = {}
        # and list of dictionaries with slice positions
        list_of_dicts = []
        add_to_list_of_dicts = list_of_dicts.append
        # get parition name and numbers from parsed partition strings
        partition_name = match.group('partition_name')
        numbers = match.group('numbers')
        # remove any whitespace padding '-' (to be consistent with partition-writing format)
        numbers = re.sub(r"[ ]*-[ ]*", "-", numbers)
        # find all numbers that will be used to parse positions
        positions = re.findall(r"([^ ,]+)", numbers)

        for position in positions:
            # create dictionary for slicing input sequence
            # conditioning on whether positions are represented
            # by range, range with stride, or single number
            pos_dict = {}

            if "-" in position:
                m = re.search(r"([0-9]+)-([0-9]+)", position)
                pos_dict["start"] = int(m.group(1)) - 1
                pos_dict["stop"] = int(m.group(2))
            else:
                pos_dict["start"] = int(position) - 1
                pos_dict["stop"] = int(position)
    
            if "\\" in position:
                # Note: the value of `N` in `...\N` isn't read: the script simply assumes `N` is consistent with the number of
                # increments per interval when the alignment is parsed with a stride of 3 (designating each cpos).
                # E.g. For the partition file:
                #       ...`1-N\2`
                #       ...`2-N\2`
                #       ...`(N+1)-M\2`
                #       ...`(N+2)-M\2`
                # 3'cpos are ignored due to the absence of intervals `3-N...`, `(N+3)-M...`, not because the associated stride values are`\2`
                pos_dict["stride"] = 3
            elif "\\" not in position:
                pos_dict["stride"] = 1

            add_to_list_of_dicts(pos_dict)

        dict_of_dicts[partition_name] = list_of_dicts
        add_to_partitions(dict_of_dicts)

    return partitions
```
This version removes some of the partition formatting restrictions of the original AMAS repo version, with the updated partitions_parse() method facilitating the uses of native RAxML(-NG) and IQ-TREE2 partition files.

The following is a contrived example to demonstrate this, along with the updated `metapartions` command. Consider the following superalignment `concat.fas`:
```
>S01
CCCCCTGGCGCCGCCGCCGCCCCTCCTCCTCCTCCTCCCCCCCCC
>S02
AACAATGGAGCAGAAGAAGAAAATAATAATAATAATAAAAAAAAA
>S03
TTCTTTGGTGCTGATGTTGTTTTTTTTTTTTTTTATTTTTTTTTT
>S04
GGCGGTGGGGCGGAGGTGGGGGGTGGTGGTGTTGATGGGGGGGGG
>S05
CCCCCTGGCGCCGACGTCGGCCCTCCTCGTCTTCATCCCCCCCCC
>S06
AACAATGGCGCAGAAGTAGGAAATACTAGTATTAATAAAAAAAAA
>S07
TTCTTTGGCGCAGATGTTGGTTATTCTTGTTTTTATTTTTTTTTT
>S08
GGCGTTGGCGCAGATGTGGGGGATGCTGGTGTTGATGGGGGGGGG
>S09
CGCCTTGGCGCAGATGTGGGCCATCCTCGTCTTCATCCCCCCCCC
>S10
AGCATTGGCGCAGATGTGGGCAATACTAGTATTCATAAAAAAAAA
>S11
TGCTTTGGCGCAGATGTGGGCTATTCTTGTATTCATTTTATTTTT
>S12
GGCGTTGGCGCAGATGTGGGCGATGCTTGTATTCATGGGAGGTGG
>S13
CGCCTTGGCGCAGATGTGGGCCATGCTTGTATTCATCCCAGCTCC
>S14
AGCATTGGCGCAGATGTGGGCCATGCTTGTATTCATAAAAGATCA
>S15
TGCATTGGCGCAGATGTGGGCCATGCTTGTATTCATTTTAGATCT
>S16
TGCATGTTCTCATATTTGTGCCAGGCGTGGATGCAGGG?AGATCT
>S17
TGCATGTTCTCATATTTGTGCCAGGCGTGGATGCAGTTTAGATCT
>S18
AGCATGTTCTCATATTTGTGCCAGGCGTGGATGCAGAAAAGATCA
>S19
CGCCTGTTCTCATATTTGTGCCAGGCGTGGATGCAGCCCAGCTCC
>S20
GGCGTGTTCTCATATTTGTGCGAGGCGTGGATGCAGGGGAGGTGG
>S21
TGCTTGTTCTCATATTTGTGCTAGTCGTGGATGCAGTTTATTTTT
>S22
AGCATGTTCTCATATTTGTGCAAGACGAGGATGCAGAAAAAAAAA
>S23
CGCCTGTTCTCATATTTGTGCCAGCCGCGGCTGCAGCCCCCCCCC
>S24
GGCGTGTTCTCATATTTGTGGGAGGCGGGGGTGGAGGGGGGGGGG
>S25
TTCTTGTTCTCATATTTTTGTTAGTCGTGGTTGTAGTTTTTTTTT
>S26
AACAAGTTCTCATAATTATGAAAGACGAGGATGAAGAAAAAAAAA
>S27
CCCCCGTTCTCCTACTTCTGCCCGCCGCGGCTGCAGCCCCCCCCC
>S28
GGCGGGTTGTCGTAGTTGTGGGGGGGGGGGGTGGAGGGGGGGGGG
>S29
TTCTTGTTTTCTTATTTTTTTTTGTTGTTGTTGTAGTTTTTTTTT
>S30
AACAAGTTATCATAATAATAAAAGAAGAAGAAGAAGAAAAAAAAA
>S31
CCCCCGTTCTCCTCCTCCTCCCCGCCGCCGCCGCCGCCCCCCCCC
```
Its partition file `partitions.txt` contain various formatting challenges to test the parser, but it includes the basic directives found in native RAxML(-NG) and IQ-TREE2 partition files:
```
charset @@(partition_A_pos1) =      7 - 21\3       
  charset %s = 8  -21\3
    charset "partition_A_pos3" = 9-   21\3
Q.insect, (partition_B_pos)1 = 40-    45\3
Q.insect,   '[partition_B_pos2] = 41   -45\3
Q.insect, par&&tition_B_pos3   =,,  42-45\3 
    GTR+G4,    pa&rtition_C =  , , , ,,38 39            37,
9.20b, #partition&|_D_pos1 = 1-6\3 22 - 36\3
       partition_D_pos2= 2-6\3 23 -36\3
p1_3_a                 = 3-6\3             24 -      36\3
  charpartition mymodels =
    9.20b: partition_D_pos1;
end;
```
While most of the partitions here are not contiguous with respect to the `concat.fas`, this superalignment can be converted into one with contiguous metapartitons by running the command:
```
./AMAS.py metapartitions -i concat.fas -f fasta -d dna --no-san --no-mpan -l partitions.txt -t concat.out.fas`
```
This generates `concat.out.fas`:
```
>S01
GGGGGGCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCTTTTTT
>S02
GGGGGGCAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAACTTTTTT
>S03
GGGGGGCATTTTTTTTTTTTTTTTTTTTTTTTTTTTTACTTTTTT
>S04
GGGGGGCATGGGGGGGGGGGGGGGGGGGGGGGGGGGTACTTTTTT
>S05
GGGGGGCATGCCCCCCCCCCCCCCCCCCCCCCCCCGTACTTTTTT
>S06
GGGGGGCATGCAAAAAAAAAAAAAAAAAAAAAAACGTACTTTTTT
>S07
GGGGGGCATGCATTTTTTTTTTTTTTTTTTTTTACGTACTTTTTT
>S08
GGGGGGCATGCATGGGGGGGGGGGGGGGGGGGTACGTACTTTTTT
>S09
GGGGGGCATGCATGCCCCCCCCCCCCCCCCCGTACGTACTTTTTT
>S10
GGGGGGCATGCATGCAAAAAAAAAAAAAAACGTACGTACTTTTTT
>S11
GGGGGGCATGCATGCATTTTTTTTTTTTTACGTACGTACTTTTTT
>S12
GGGGGGCATGCATGCATGGGGGGGGGGGTACGTACGTACTTTTTT
>S13
GGGGGGCATGCATGCATGCCCCCCCCCGTACGTACGTACTTTTTT
>S14
GGGGGGCATGCATGCATGCAAAAAAACGTACGTACGTACTTTTTT
>S15
GGGGGGCATGCATGCATGCATTTTTACGTACGTACGTACTTTTTT
>S16
TTTTTTCATGCATGCATGCATG?GTACGTACGTACGTACGGGGGG
>S17
TTTTTTCATGCATGCATGCATTTTTACGTACGTACGTACGGGGGG
>S18
TTTTTTCATGCATGCATGCAAAAAAACGTACGTACGTACGGGGGG
>S19
TTTTTTCATGCATGCATGCCCCCCCCCGTACGTACGTACGGGGGG
>S20
TTTTTTCATGCATGCATGGGGGGGGGGGTACGTACGTACGGGGGG
>S21
TTTTTTCATGCATGCATTTTTTTTTTTTTACGTACGTACGGGGGG
>S22
TTTTTTCATGCATGCAAAAAAAAAAAAAAACGTACGTACGGGGGG
>S23
TTTTTTCATGCATGCCCCCCCCCCCCCCCCCGTACGTACGGGGGG
>S24
TTTTTTCATGCATGGGGGGGGGGGGGGGGGGGTACGTACGGGGGG
>S25
TTTTTTCATGCATTTTTTTTTTTTTTTTTTTTTACGTACGGGGGG
>S26
TTTTTTCATGCAAAAAAAAAAAAAAAAAAAAAAACGTACGGGGGG
>S27
TTTTTTCATGCCCCCCCCCCCCCCCCCCCCCCCCCGTACGGGGGG
>S28
TTTTTTCATGGGGGGGGGGGGGGGGGGGGGGGGGGGTACGGGGGG
>S29
TTTTTTCATTTTTTTTTTTTTTTTTTTTTTTTTTTTTACGGGGGG
>S30
TTTTTTCAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAACGGGGGG
>S31
TTTTTTCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCGGGGGG
```
The figure below more clearly demonstrates this transformation: `concat.fas` on the left is converted to `concat.out.fas` on the right using the `metapartitions` command above; visualized in [Aliview](https://github.com/AliView/AliView) v1.28


### Translating a DNA alignment into aligned protein sequences
You can translate a nucleotide alignment to amino acids with AMAS using one of the [NCBI translation tables](http://www.ncbi.nlm.nih.gov/Taxonomy/Utils/wprintgc.cgi). For example, to correctly translate an insect mitochondrial gene alignment that begins at a second codon position:
```
python3 AMAS.py translate -f nexus -d dna -i concat.nex --code 5 --reading-frame 2 --out-format phylip
```
`--code` and `--reading-frame` are the same as `-b` and `-k` and are both set to 1 (the standard genetic code and the first character of the alignment corresponds to the first codon position) by default. When translating, `AMAS` will contract gaps `-` and missing `?`, such that `---` becomes `-` in the translated alignment. A warning will be printed if stop codons are found and these are indicated as asterisks `*` in the output. See `AMAS.py translate -h` for more info.

### Creating replicate data sets
With `AMAS` you can create concatenated alignments from a proportion of randomly chosen alignments that can be used for, for example, a phylogenetic jackknife analysis. Say you have 1000 phylip files, each containing a single aligned locus, and you want to create 200 replicate phylip alignments, each built from 100 loci randomly chosen from all the input files. You can do this by specifying `replicate` command and following it with `-r` or `--rep-aln` followed by the number of replicates (in this case `200`) and number of alignments (`100`). Remember to supply the output format with `-u` if you want it to be other than `fasta`:
```
python3 AMAS.py replicate -r 200 100 -d dna -f phylip -i *phy -u phylip
```
### Removing taxa/sequences from alignment
It is possible to remove taxa from alignments:
```
python3 AMAS.py remove -x species1 species2 -d dna -f nexus -i *nex -u nexus-int -g no_species12_ 
```
The above will process all `nexus` files in the directory and remove taxa called `species1` and `species2`. The argument `-x` (the same as `--taxa-to-remove`) is followed by the names of sequences to be removed. Note that `AMAS` converts spaces into underscores and strips any quotes present in input sequence names before processing, so you may need to modify your names to remove accordingly. The argument `-g` (the same as `--out-prefix`) specifies a prefix to be added to output file names. The default prefix is 'reduced_'. You may want to realign your files after taxon removal.
  
### Checking if input is aligned
By specifying optional argument `-e` (`--check-align`), you can make `AMAS` check if your input files contain only aligned sequences. This option is disabled by default because it can substantially increase computation times in files with many taxa. Enabling this option also provides an additional check against misspecified input file format.

### TODO Metapartitions

# TODO AMAS as a Python module
Using `AMAS` inside your Python pipeline gives you much more flexibility in how the input and output are being processed. All the major functions of the command line interface can recreated using `AMAS` as a module. Following installation from [pip](https://pip.pypa.io/en/latest/installing.html) use:
```
pydoc amas.AMAS
```
To access detailed documentation for the classes and functions available.

You can import `AMAS` to your script with:
```
from amas import AMAS
```
The class used to manipulate alignments in `AMAS` is `MetaAlignment`. This class has to be instantiated with the same, named arguments as on the command line: `in_files`, `data_type`, `in_format`. You also need to supply the number of cores to be used with `cores`. MetaAlignment holds one or multiple alignments and its `in_files` option must be a list, even if only one file is being read.
```

meta_aln = AMAS.MetaAlignment(in_files=["gene1.phy"], data_type="dna",in_format="phylip", cores=1)
```
Creating MetaAlignment with multiple files is easy:
```
multi_meta_aln = AMAS.MetaAlignment(in_files=["gene1.phy", "gene1.phy"], data_type="dna", in_format="phylip", cores=2)
```
Now you can call the various methods on your alignments. `.get_summaries()` method will compute summaries for your alignments and produce headers for them as atuple with first element being the header and the second element a list of lists with the statistics:
```
summaries = meta_aln.get_summaries()
```
The header is different for nucleotide and amino acid data. You may choose to skip it and print only the second element of the tuple, that is a list of summary statistics:
```statistics = summaries[1]
```
`.get_parsed_alignments()` returns a list of dictionaries where each dictionary is an alignment and where taxa are the keys and sequences are the values. This allows you to, for example, print only taxa names in each alignment or do other manipulation of the sequence data:
```
# get parsed dictionaties
aln_dicts = multi_meta_aln.get_parsed_alignments()

# print only taxa names in the alignments:
for alignment in aln_dicts:
    for taxon_name in alignment.keys():
        print(taxon_name)
```
Similar to the above example, it is also easy to get translated amino acid alignment as a list of dictionaries (one per input alignment):
```
# get parsed dictionaties
aln_dicts = multi_meta_aln.get_translated(2, 1) # 2: vertebrate mitochondrial genetic code and 1: reading frame starting at first character
```
To split alignment use `.get_partitioned("your_partitions_file")` on a `MetaAlignment` with a single input file. `.get_partitioned()` returns a list of dictionaries of dictionaries, with `{ partition_name : { taxon : sequence } }` structure for each partition:
```
partitions = meta_aln.get_partitioned("partitions.txt")
```
`AMAS` uses `.get_partitions("your_partitions_file")` to parse the partition file:
```
parsed_parts = meta_aln.get_partitions("partitions.txt")
print(parsed_parts)
```

`.get_replicate(no_replicates, no_loci)` gives a list of parsed alignments (dictionaries), each a replicate constructed from the specified number of loci:
```
replicate_sets = multi_meta_aln.get_replicate(2, 2)
```
To concatenate multiple alignments first parse them with `.get_parsed_alignments()`, then pass to `.get_concatenated(your_parsed_alignments)`. This will return a tuple where the first element is the `{ taxon : sequence }` dictionary
of concatenated alignment and the second element is the partitions dict with `{ name : range }`.
```
parsed_alns = multi_meta_aln.get_parsed_alignments()
concat_tuple = multi_meta_aln.get_concatenated(parsed_alns)
concatenated_alignments = concat_tuple[0]
concatenated_partitions = concat_tuple[1]
```
Removing taxa from alignments is very easy:
```
spp_to_remove = ["taxon1", "taxon2", "taxon3"]
reduced_alns = multi_meta_aln.remove_taxa(spp_to_remove)
```
To print to file or convert among file formats use one of the `.print_format(parsed_alignment)` methods called with a parsed dictionary as an argument. These methods include `.print_fasta()`, `.print_nexus()`, `.print_nexus_int()`, `print_phylip()`, and `.print_phylip_int()`. They return an apporpriately formatted string.
```
for alignment in concatenated_alignments:
    nex_int_string = meta_aln.print_nexus_int(alignment)
    print(nex_int_string)
```