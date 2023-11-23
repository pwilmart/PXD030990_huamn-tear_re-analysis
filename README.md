# PXD030990_human-tear_re-analysis

## How well do single shot runs work for proteome profiling?

---

### Phil Wilmarth, PSR Core, OHSU
### November 22, 2023

Re-analysis of human tear data from [PXD030990](https://www.ebi.ac.uk/pride/archive/projects/PXD030990). The publication is [here](https://www.mdpi.com/1422-0067/23/4/2307):

> Jones, G., Lee, T.J., Glass, J., Rountree, G., Ulrich, L., Estes, A., Sezer, M., Zhi, W., Sharma, S. and Sharma, A., 2022. Comparison of different mass spectrometry workflows for the proteomic analysis of tear fluid. International journal of molecular sciences, 23(4), p.2307.

## Publication summary

Human tear fluid was collected from 11 heathy subjects using Schirmer's strips. The wetted strips were cut in half and two protein digestion strategies compared. Proteins were extracted from the strips and digested with trypsin after extraction in one method. In the other method, trypsin was added to the strips for in-strip digestion followed by peptide extraction. Each sample was run single shot on a Thermo Tribrid Fusion instrument in maximum peptide ID modes using a 120-minute LC separation. The 22 samples (11 subjects with two digestion methods) were run twice; one run with CID dissociation and a second run using HCD dissociation. The dataset was 44 RAW files of around 1.2 GB per file.

The data were processed with Proteome Discoverer v2.2. SEQUEST using a human Swiss-Prot FASTA file (about 20.3K sequences) with a precursor mass tolerance of 10 ppm, fragment ion tolerance of 0.6 Da. Variable oxidized Met and static alkylated Cys modifications were specified. Percolator was used for PSM validation. Q-value cutoffs were not listed. Protein inference was described as parsimonious. Use of decoy sequences for FDR control was not mentioned.

PD has default setting of a 6 amino acid minimum peptide length. Assuming some form of target/decoy was used, PD can set PSM cutoffs of medium (q less that 0.05) or high (q less than 0.01). It is not clear what was used. Protein reporting in PD uses protein ranking functions and allows single peptide per protein IDs by default. I think decoy proteins may be used in some form of protein FDR control. Protein inference details were missing.

The paper is nicely written and the interpretation of results seems sensible. The over 3,000 proteins identified claim is questionable and is more likely the consequence of poor protein ID error control in PD. This is more of a negative for the PD developers than the authors, although the burden of testing analysis tools lies with the users of said tools.   

## Re-analysis details

The RAW files were processed with the [PAW pipeline](https://github.com/pwilmart/PAW_pipeline) as follows:

- RAW files converted to MS2 format files with MSConvert
- Comet searches for peptide sequence assignments
  - human canonical UniProt reference FASTA file (about 20.5K sequences)
  - semi-tryptic cleavage option with up to 3 missed cleavages (bio fluids need semi-tryptic)
  - parent ion mass tolerance of 1.25 Da (wide tolerance searches work better)
  - fragment ion tolerance 1.0005 (ion trap analyzer used for MS2)
  - variable oxidation of Methionine
  - static alkylated Cys
- target/decoy method using accurate mass conditioned score histograms
  - linear discriminant function of Comet scores
  - minimum peptide length of 7 amino acids
  - Charge states 2+, 3+, and 4+
  - delta mass windows set at 0-Da, 1-Da, and any other delta mass
  - FDR analysis done separately on 2+, 3+, and 4+ peptides
  - FDR analysis done separately for semi-tryptic and fully tryptic peptides
  - FDR analysis done separately for unmodified and modified peptides
- basic and extended parsimony protein inference
  - basic parsimony logic groups identical peptide sets
  - basic parsimony logic removes peptide sets that are subsets
  - extended parsimony logic groups proteins with insufficient ID evidence
  - protein ID uses the two peptide rule
    - two distinct, full or semi tryptic, unmodified peptides per protein per samples

This is a basic, best practices pipeline. FASTA files are chosen carefully for completeness with minimal peptide redundancy, search setting are kept simple, wide tolerance searches are used so that accurate mass has classifier power (i.e., distinguishes correct from incorrect matches), proteins are grouped so that shared peptides do not adversely affect quantitation, and the two peptide rule controls random protein false discoveries. There are some blog posts that describe my pipeline in more detail: [what makes it different](https://pwilmart.github.io/blog/2021/06/06/PAW-pipeline-backstory) and some concepts for [bottom-up quantitative proteomics](https://pwilmart.github.io/blog/2019/09/21/shotgun-quantification).

## Dataset statistics

Here is the breakdown on the 44 RAW files - how many starting MS2 scans, how many scans had matches that made the 1% target/decoy FDR cutoff, and the scan-level PSM ID rates.

**Dataset statistics (comma is used for thousands separator and period is used for decimal point):**

Dissociation|Digestion Method|LC Name|MS2 Scans|Filtered Scans|ID Rate
---|---|---|---|---|---
CID|After extraction|T017_Post-extraction_digestion_CID|65,566|9,766|14.9%
CID|After extraction|T018_Post-extraction_digestion_CID|65,032|11,285|17.4%
CID|After extraction|T019_Post-extraction_digestion_CID|70,269|14,529|20.7%
CID|After extraction|T020_Post-extraction_digestion_CID|71,621|12,119|16.9%
CID|After extraction|T021_Post-extraction_digestion_CID|71,991|12,955|18.0%
CID|After extraction|T022_Post-extraction_digestion_CID|68,927|10,558|15.3%
CID|After extraction|T023_Post-extraction_digestion_CID|67,883|9,876|14.5%
CID|After extraction|T024_Post-extraction_digestion_CID|70,993|13,396|18.9%
CID|After extraction|T025_Post-extraction_digestion_CID|66,343|12,859|19.4%
CID|After extraction|T026_Post-extraction_digestion_CID|64,524|12,790|19.8%
CID|After extraction|T027_Post-extraction_digestion_CID|66,562|9,078|13.6%
CID|In strip|T017_in-strip_digestion_CID|70,096|12,766|18.2%
CID|In strip|T018_in-strip_digestion_CID|68,521|16,833|24.6%
CID|In strip|T019_in-strip_digestion_CID|70,724|14,543|20.6%
CID|In strip|T020_in-strip_digestion_CID|65,246|13,307|20.4%
CID|In strip|T021_in-strip_digestion_CID|72,039|13,571|18.8%
CID|In strip|T022_in-strip_digestion_CID|73,199|11,595|15.8%
CID|In strip|T023_in-strip_digestion_CID|70,586|12,621|17.9%
CID|In strip|T024_in-strip_digestion_CID|69,449|14,509|20.9%
CID|In strip|T025_in-strip_digestion_CID|67,950|13,044|19.2%
CID|In strip|T026_in-strip_digestion_CID|69,062|12,737|18.4%
CID|In strip|T027_in-strip_digestion_CID|68,453|13,928|20.3%
HCD|After extraction|T017_Post-extraction_digestion_HCD|62,031|9,359|15.1%
HCD|After extraction|T018_Post-extraction_digestion_HCD|62,159|10,749|17.3%
HCD|After extraction|T019_Post-extraction_digestion_HCD|72,196|12,579|17.4%
HCD|After extraction|T020_Post-extraction_digestion_HCD|68,588|11,003|16.0%
HCD|After extraction|T021_Post-extraction_digestion_HCD|68,699|10,978|16.0%
HCD|After extraction|T022_Post-extraction_digestion_HCD|64,249|9,363|14.6%
HCD|After extraction|T023_Post-extraction_digestion_HCD|62,922|9,881|15.7%
HCD|After extraction|T024_Post-extraction_digestion_HCD|68,707|12,013|17.5%
HCD|After extraction|T025_Post-extraction_digestion_HCD|65,603|10,147|15.5%
HCD|After extraction|T026_Post-extraction_digestion_HCD|69,264|11,923|17.2%
HCD|After extraction|T027_Post-extraction_digestion_HCD|62,595|8,844|14.1%
HCD|In strip|T017_in-strip_digestion_HCD|71,427|12,482|17.5%
HCD|In strip|T018_in-strip_digestion_HCD|69,743|15,447|22.1%
HCD|In strip|T019_in-strip_digestion_HCD|67,189|13,275|19.8%
HCD|In strip|T020_in-strip_digestion_HCD|65,861|12,615|19.2%
HCD|In strip|T021_in-strip_digestion_HCD|68,263|12,504|18.3%
HCD|In strip|T022_in-strip_digestion_HCD|67,482|9,603|14.2%
HCD|In strip|T023_in-strip_digestion_HCD|67,049|11,102|16.6%
HCD|In strip|T024_in-strip_digestion_HCD|68,384|13,118|19.2%
HCD|In strip|T025_in-strip_digestion_HCD|66,769|10,815|16.2%
HCD|In strip|T026_in-strip_digestion_HCD|72,934|11,593|15.9%
HCD|In strip|T027_in-strip_digestion_HCD|68,021|13,766|20.2%
Totals|||2,995,171|531,824|17.8%

---

We can summarize things by the four methods (digestion method and dissociation method) to get a better overview of the data.

**Summary by digestion/dissociation methods (averages)**

Dissociation|Digestion Method|MS2 Scans|Filtered Scans|ID Rate
---|---|---|---|---
CID|After extraction|68,156|11,746|17.2%
HCD|After extraction|66,092|10,622|16.1%
CID|In-strip|69,575|13,587|19.5%
HCD|In-strip|68,466|12,393|18.1%

There were nearly 3 million MS2 scans acquired from the 44 runs. Over half a million scans passed the 1% FDR cutoff. There are more PSMs identified from the in-strip digestion method. There are fewer identified PSMs from HCD compared to HCD despite acquiring a few more MS2 scans with the faster HCD dissociation.

> Note that the number of PSMs at 1% FDR implies (as many as) 5,318 incorrect peptide sequence assignments among the accepted scans (1% of 531,824). Given the filtered scan numbers above, that is from 88 to 168 incorrect PSMs per run (about 120 average). This means that we could have over 100 or so incorrect protein IDs per sample (assuming no protein FDR control) or as many as 5,000 or so experiment-wide. NOTE: the overall FDR was less than 1% because there were very small number of decoy matches for accurate mass (0-Da deltamass), fully tryptic, unmodified peptides (the peptide class with the most target matches). Instead of 5K decoy PSMs, the filtered data had closer to 3K decoy matches.

Some other notes about the analysis: 1+ peptides were not excluded from acquisition (charges 1 to 6 were present). The mass calibration was very good across all samples. The data were processed by dissociation method (22 samples per chunk) in case Comet score distributions differed for CID versus HCD data (they actually looked similar). Protein inference was done from the confident (1% FDR) PSMs from all 44 samples to produce a master list of putative protein IDs experiment-wide. That list was culled by requiring the two-peptide rule to be satisfied in at least one sample (per LC run here).

> Two distinct peptides across the whole experiment is not sufficient criteria for protein IDs. We know that the proteins are present in individual samples and we need to apply the two peptide rule to individual samples.

## How are PAW protein results summarized?

Some clarification on the PAW protein summary table is necessary. The two-peptide rule can cause some confusion as it can be applied at different levels in larger experiments. There is also the fear that there is a loss of protein IDs when using the 2-peptide rule. That may have been an important concern in the Q-STAR (a less sensitive, slow scanning early Q-TOF instrument from ABI) days when a "large proteome" was a couple hundred proteins. Very large datasets can be easily generated these days and the handful of very low abundance proteins that would be legitimate single peptide per protein IDs are still more trouble than they are worth (the nickname of "one-hit-wonders" is still on the nose). They will almost always be below the limits of quantitation in any experiment.

A feature of the PAW pipeline is to be as generous in discovery phase experiments as possible without shooting yourself in the foot with appreciable numbers of false identifications. The protein summary table is basically one row for each protein, protein group, or protein family inferred from the confidently identified PSMs in the entire experiment. The "experiment" is whatever set of biological samples you want a unified protein report for (you do not explicitly specify an experimental design at any point in my pipeline). Samples can consist of multiple LC runs (fractions) or have hidden sample dimensions (SILAC channels, TMT channels, etc.). The sets of columns in the protein summary tables are the samples. There are three sets of PSM/peptide count columns: total PSM counts, total unique PSM counts, and corrected total counts:

- **Total counts** are PSM counts for respective proteins where all peptides (shared and unique) are counted
- **Unique counts** are PSM counts of the unique peptides for the respective proteins
- **Corrected counts** are PSM counts where shared peptides have been fractionally split based on relative unique count evidence (for any proteins that share given peptides)

> **shared** and **unique** need to be carefully defined in any bottom-up proteomics data analysis. Here, a unique peptide has a one-to-one mapping to a reported protein, protein group (indistinguishable peptide sets), or protein family (nearly indistinguishable peptide sets). Shared peptides map to more than one protein, protein group, or protein family. The context for mapping is the final set of inferred proteins, not the original FASTA file (this distinction is **extremely** important for quantitative studies).

A protein row exists if that protein was identified by two distinct peptides in **at least** one sample across the experiment. The PSM counts for each sample (the cells in that row) can be greater than 2, equal to 2 (this has to be true in at least one sample), or less than 2. The counts can be zero or one for some of the columns. This strikes a sensible compromise between having sufficient evidence to report a protein and being able to track low abundance proteins across samples. Not all proteins that get reported in the protein summaries will be quantifiable, though. There are many low abundance proteins can be identified but not quantified in large scale experiments.

## Re-analysis results

Five protein summaries were done with different subsets of runs to get total number of proteins from all data and the numbers of proteins identified in each of the 4 methods (2 digestion methods and two dissociation types). The idea is to start with as much data as possible for protein inference and work back down to the sample-level protein identifications.

This is my solution to the complicated problem of summarizing experiments with larger number of samples. This can be approached in the other direction where proteins are inferred in each sample and some combination of those results are generated for the experiment-wide summary. This can be challenging because protein grouping and parsimony logic may produce different proteins for related sets of proteins in each sample. Logic to say two non-identical things are really the same between samples can be tricky to code.

**Total protein identifications in full experiment and by method.** Net proteins are after exclusion of decoy and contaminant proteins (keratins, trypsin, etc.). Decoy protein numbers are ballpark estimates of protein reporting error rates. There is no concept of protein FDR control in my pipeline (if results seem too noisy, PSMs need to be filtered more stringently).

Experiment subset|Number of samples|Net number of protein IDs|Number of decoy proteins
---|---|---|---
Whole experiment|44|478|9
Extracted proteins + CID|11|321|18
Extracted proteins + HCD|11|311|10
In-strip digest + CID|11|469|14
In-strip digest + HCD|11|441|14

<br>The paper claimed over 3,000 identified proteins and this re-analysis has only 478 identified proteins. Note that the best method (in-strip + CID) had 469 proteins, almost the same as the 478 experiment-wide. The paper also reported average numbers of proteins identified per sample (and standard deviations) for each of the 4 methods. Below we report the re-analysis average protein identification numbers and the numbers from the paper.
<br><br>

**Average number of protein IDs per sample by method.** Values are averages plus/minus standard deviations.

Method|Average Protein IDs (re-analysis)|Average Protein IDs (publication)
---|---|---
Extracted proteins + CID|163 +/- 41|489 +/- 90
Extracted proteins + HCD|157 +/- 34|496 +/- 85
In-strip digest + CID|235 +/- 65|666 +/- 161
In-strip digest + HCD|231 +/- 62|678 +/- 180

I know from analyses of many datasets that results from my pipeline and PD will be very similar if two peptides per protein is specified in PD. We can assume the extra identifications reported in the publication are from single peptide protein IDs. There are about 400 extra protein IDs per sample.

## Are extra protein IDs reliable?

How reliable are these extra protein IDs? Random noise matches are unlikely to be the same sample-to-sample. The paper reported 4 frequency of observation categories for each method: high (seen in 9, 10, or 11 of the samples), medium (seen in 6, 7, or 8 of the samples), low (seen in 3, 4, or 5 of the samples), and rare (seen in just 1 or 2 samples). We can compute similar quantities for the re-analysis results and compare to the publication.

**Observation frequencies from this re-analysis:**

Category|extracted CID|extracted HCD|in-strip CID|in-strip HCD
---|---|---|---|---
high|103|98|149|144
medium|41|37|65|61
low|79|79|107|106
rare|105|99|122|122

<br> **Observation frequencies from publication:**

Category|extracted CID|extracted HCD|in-strip CID|in-strip HCD
---|---|---|---|---
high|122|112|178|182
medium|92|90|153|147
low|248|258|366|373
rare|2237|2310|2596|2668

<br> There is a real explosion of protein IDs seen in just one or two of the 11 samples for the published results. Such a dramatic increase is a red flag. Identification numbers are much more stable across categories from the re-analysis.

If we exclude the "rare" category IDs and sum up the rest, we can get a ballpark estimate of the "more reliable" protein identifications.

**Sum of high, medium, and low categories by method:**

Source|extracted CID|extracted HCD|in-strip CID|in-strip HCD
---|---|---|---|---
Re-analysis|223|214|321|311
Publication|462|460|697|702

<br> The publication numbers are larger because there are some true single peptide per protein IDs. The 2-peptide rule used in the re-analysis is more strict.

## Estimate of how many single peptide per protein IDs are true

I can also do single peptide per protein IDs with my pipeline (results will be very noisy). With the target/decoy concatenated FASTA file, estimates of inferred decoy protein counts can be subtracted from inferred target protein matches to get net correct inferred protein ID estimates. I did this for a results summary of the 44-sample full experiment. Common contaminant proteins were excluded leaving 2,266 target protein matches. There were 1,558 decoy protein matches. Assuming the decoy protein count is an estimate of the incorrect target protein count, we have a net estimated correct target protein count of 708.

This 708 number is very close to the 700 numbers from the sum of the high, medium, and low observation frequency categories from the publication. Also, 708 is a little over 200 more protein IDs than the 478 proteins with the 2-peptide rule, suggesting about 230 single peptide per protein IDs are correct. This seems like a reasonable number of barely detectible proteins in human tear. I speculate that the single peptide per protein IDs from PD in the publication were mostly incorrect with the chosen parameters. The protein inference details and protein FDR control method were not detailed in the paper, so I cannot speculate further.

## Single shot experimental designs

Getting about 700 proteins from over 530K PSMs is nothing to write home about and (strongly) suggests single shot experimental designs have severe limitations. We have done some TMT-labeled human tear samples collected with Schirmer's strips in our core. We extracted proteins, did S-Trap digests, labeled with TMTpro, and ran 20-high pH reverse phase fractions on a Fusion Tribrid with a 2-hour low pH RP gradient for each fraction. An SPS-MS3 TMT method was used. More about the experiments can be found in [this publication](https://www.sciencedirect.com/science/article/abs/pii/S1542012423000198). Each 20-fraction experiment had about 370K MS2/MS3 scans acquired. About 66K scans were identified at a 1% FDR in each experiment. Those PSMs mapped to around 3K proteins with a 2-peptide per protein requirement for each experiment.

Fractionation gives almost 5 times as many protein IDs from PSM numbers that are 8 times smaller. You could do a fractionated 18-plex TMT experiment where over 3K proteins could be accurately quantified from tears in the same clock time as 18 single shot runs where a few hundred proteins could be quantified. The two experimental designs use the same number of hours of instrument time, but are not remotely equivalent in terms of quantitative proteome profiling.

If you are not a fan of DDA experiments, DIA won't change the situation. Wide window DDA won't do it either. Allowing chimeric precursors is going to help either. Nothing gets past the fact that you have limited dynamic range for peptides binding on the LC system and limited inter-scan dynamic range on the mass spec instrument. These are fundamental measurement science limitations. Think for a minute or two about acquiring 3 million MS2 scans, filtering to over 530K confident PSMs, and getting only 500-700 proteins from samples known to have well over 3,000 proteins (human tears).

There were real reasons why MudPIT methods were developed 25 years ago. Loading more digest and fractionating is not just about instrument scan rates and sampling. It is also about how you can get more of the peptides from the complex digest into the instrument. More total digest is used in fractionated experiments. Single shot experiments are limited by the laws of physics. Very short gradient single shot experiments seem like an incredibly bad idea to me with my scant 20 years experience in proteomics. I do not understand why smart folks think deep proteome profiling is possible with little separation of very complex peptide digests.

---

Phil Wilmarth
<br>November 22, 2023
