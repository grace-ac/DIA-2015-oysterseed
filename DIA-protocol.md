This protocol is adapted from [Roberts Lab DIA Analysis](https://github.com/RobertsLab/resources/blob/master/protocols/DIA-data-Analyses.md)

# DIA Analyses

Data-Independent Mass Spectrometry is a bottom-up proteomics method that separately gathers MS/MS spectra and MS survey spectra. The following workflow describes how to analyze DIA data.

### **Basic overview of the DIA pipeline**:

1) Convert raw files using MSConvert. Simultaneously demultiplex raw data files if data were collected with overlapping windows.

2) Use WALNUT to create a peptide spectral library from data 

3) Get peptide and protein abundance information from Skyline 

This wiki is based partly on the following [Evernote entry](https://www.evernote.com/shard/s347/sh/edcb06ab-d008-418f-b28f-52f6614f1c39/2984ab55f427fcfe). 

Note: this might be a little less intuitive than the [DDA pipeline](https://github.com/RobertsLab/resources/blob/master/protocols/DDA-data-Analyses.md). Additionally, the pipeline relies on Windows-based programs.

---

## Step 1: Convert RAW files

### Step 1a. Convert Raw files with MSConvert (No demultiplexing)
Output files from a mass spectrometer are in the .raw format, but PECAN requires demultiplexed .mzML files. [MSConvert](http://proteowizard.sourceforge.net/tools.shtml) is a GUI used to generate these files with the appropriate centroid peaks using 64-bit and zlib compression. However, demultiplexing has not been incorporated into the GUI so if you are demultiplexing you must use command line (see Step 1b).

Below are the settings required for MSConvert in the GUI:

![MSConvert-settings](https://raw.githubusercontent.com/sr320/LabDocs/master/img/MSConvert-settings.png)

Note that these setting [might vary](https://github.com/sr320/LabDocs/issues/486) based on version.


---

## Step 2. Prepare Peptide Database
This is done be identifying "reference proteome" and performing in silico digestion. 

### Step 2a. Merge reference proteome with Quality Standards.
Essentially concatenate the fasta files. Here is an example of a reference proteome- [C. gigas proteome](http://owl.fish.washington.edu/halfshell/bu-git-repos/nb-2017/C_gigas/data/Cg_Gigaton_proteins.fa). A tabular version of the QC peptide list for the January 2017 Lumos run can be found [here](http://owl.fish.washington.edu/generosa/Generosa_DNR/Pierce_PRTC.tabular).

### Step 2b. Run in silico tryptic digestion of reference proteome with quality standards.
This is done with [Protein Digestion Simulator](https://omics.pnl.gov/software/protein-digestion-simulator), a Windows program.

Settings for the in silico tryptic digest, as discussed on [Issue #483](https://github.com/sr320/LabDocs/issues/483). Each screenshot refers to a different tab in the program window. The screenshots are sequential (Tab #1-Tab #4).

Ensure the output, a digested reference proteome, is in a .tabular format.

![PDS tab 1](https://github.com/RobertsLab/Paper-DNR-Proteomics/blob/master/images/2017-02-19_final-Digest-Settings1.png?raw=true)

*Note: "Delimited Input File Options" depends on the format of your fasta file. This one only has protein ID and sequence.*

![PDS tab 2](https://github.com/RobertsLab/Paper-DNR-Proteomics/blob/master/images/2017-02-19_final-Digest-Settings2.png?raw=true)

![PDS tab 3](https://github.com/RobertsLab/Paper-DNR-Proteomics/blob/master/images/2017-02-19_final-Digest-Settings3.png?raw=true)

*Note: Use default settings*

![PDS tab 4](https://github.com/RobertsLab/Paper-DNR-Proteomics/blob/master/images/2017-02-19_final-Digest-Settings4.png?raw=true)

*Note: Use default settings*

To execute digest, on select "Parse and Digest" on Tab #2.

### Step 2c. Modify .tabular digested proteome
The digested proteome will provide information unnecessary for PECAN.

![full-digested-proteome](https://cloud.githubusercontent.com/assets/22335838/23740214/790b2008-0457-11e7-8c26-e3aea0881759.png)

The only columns needed for PECAN are the first two: "Protein_Name" and "Sequence". These columns can be cut out using `awk` or in [Galaxy](usegalaxy.org). The final .tabular digested proteome should look like this:

![modified-digested-proteome](https://cloud.githubusercontent.com/assets/22335838/23740215/790e76e0-0457-11e7-911f-aa65f6069b61.png)

Example
On Windows using Git Bash.
```

srlab@swan MINGW64 ~
$ cd Desktop/

srlab@swan MINGW64 ~/Desktop
$ cd grace/

srlab@swan MINGW64 ~/Desktop/grace
$ head Cg_Giga_cont_prtc_AA_digested_Mass400to6000.txt
Protein_Name    Sequence        Unique_ID       Monoisotopic_Mass       Predicte
d_NET   Tryptic_Name
CHOYP_043R.5.5|m.64252  SPSEDPDAPIENILQTNSVYKPK 1       2541.2598016    0.3655t2
.1
CHOYP_043R.5.5|m.64252  SPSEDPDAPIENILQTNSVYKPKK        2       2669.35475980.34
14      t2.2
CHOYP_043R.5.5|m.64252  SPSEDPDAPIENILQTNSVYKPKKEPTYDENVVVK     3       3942.973
762     0.3449  t2.3
CHOYP_043R.5.5|m.64252  SPSEDPDAPIENILQTNSVYKPKKEPTYDENVVVKIISQDTPTILR  45180.67
6764    0.5144  t2.4
CHOYP_043R.5.5|m.64252  KEPTYDENVVVK    5       1419.7245246    0.2186  t3.2
CHOYP_043R.5.5|m.64252  KEPTYDENVVVKIISQDTPTILR 6       2657.4275266    0.4593t3
.3
CHOYP_043R.5.5|m.64252  KEPTYDENVVVKIISQDTPTILRVSFTVNR  7       3460.85649280.56
58      t3.4
CHOYP_043R.5.5|m.64252  EPTYDENVVVK     8       1291.6295664    0.2301  t4.1
CHOYP_043R.5.5|m.64252  EPTYDENVVVKIISQDTPTILR  9       2529.3325684    0.4402t4
.2

srlab@swan MINGW64 ~/Desktop/grace
$ awk '{print $1,$2}' Cg_Giga_cont_prtc_AA_digested_Mass400to6000.txt | head
Protein_Name Sequence
CHOYP_043R.5.5|m.64252 SPSEDPDAPIENILQTNSVYKPK
CHOYP_043R.5.5|m.64252 SPSEDPDAPIENILQTNSVYKPKK
CHOYP_043R.5.5|m.64252 SPSEDPDAPIENILQTNSVYKPKKEPTYDENVVVK
CHOYP_043R.5.5|m.64252 SPSEDPDAPIENILQTNSVYKPKKEPTYDENVVVKIISQDTPTILR
CHOYP_043R.5.5|m.64252 KEPTYDENVVVK
CHOYP_043R.5.5|m.64252 KEPTYDENVVVKIISQDTPTILR
CHOYP_043R.5.5|m.64252 KEPTYDENVVVKIISQDTPTILRVSFTVNR
CHOYP_043R.5.5|m.64252 EPTYDENVVVK
CHOYP_043R.5.5|m.64252 EPTYDENVVVKIISQDTPTILR

srlab@swan MINGW64 ~/Desktop/grace
$ awk '{print $1,$2}' Cg_Giga_cont_prtc_AA_digested_Mass400to6000.txt \
> > Cg_Giga_cont_prtc_AA_M400-6000-2c.txt
```




---

## Step 3. Run WALNUT
WALNUT correlates your acquired peptide spectra to a database of known sequences and creates a library of proteins and peptides that you detected in your experiment. WALNUT requires several inputs, each of which must be prepared before running WALNUT in the command line.

### Step 3a. For "Background" and "Target", select "Edit" and choose a fasta file for your samples.

### Step 3b. Select "Add mzml" and add a sample file. Repeat this for each sample file (no need to wait until the previous ones have finished). You can also add multiple files at once using Ctrl+click or Shift+click.

### Step 3c. Choose "Save BLIB" (again, no need to wait until the files are finished processing)

### Step 3d. Wait for the search to finish (~30 min per file, very roughly. If it seems like it's taking multiple hours or a day per file then something is wrong)

![img](https://github.com/grace-ac/grace-ac.github.io/blob/master/notebook-images/Walnut01.PNG)


The .blib file you create will be used in the Skyline workflow (see below).

---

## Step 4: Add files to Skyline

The workflow below was adapted from [these tutorial slides](https://github.com/RobertsLab/project-pacific.oyster-larvae/blob/master/Skyline-example-files-ETS.sky/slides01.pdf).

### Step 4a: Collect necessary materials.

- Skyline (preferably Skyline daily, but you need to get permission to download it)
- .blib file from WALNUT (see Step 3)
- Undigested FASTA background proteome, including QC protein sequence
- Demultiplexed mzML files, or RAW files if demultiplexing was not necessary

### Step 4b: Import spectral library (.blib)

1. Open a new Skyline file
2. Under Settings >> Peptide Settings >> Library, click "Edit List". If your .blib is already on the list, select it. 
3. If your .blib is not already on the list, click "Add"
  - Name your library and select the .blib file
  - After clicking "OK", select the correct library from the list
4. Under "Pick peptides matching", select "Library"

<img width="363" alt="screen shot 2017-07-21 at 9 00 25 am" src="https://user-images.githubusercontent.com/22335838/28471776-181be092-6df3-11e7-9086-3a8eb037fb78.png">

### Step 4c: Add background proteome

Remember, this background is the fast version of the background proteome from `pecanpie`, undigested.

1. Settings >> Peptide Settings >> Digestion
2. Select "Add" under "Background proteome"
3. Name background
4. Click "Create" under "Proteome file" to choose where to save the background, creating/renaming it as a .protb
5. Click "Add file" under "FASTA files", and select your background proteome fasta file
6. Under Prediction tab, make sure "none" is selected for retention time predictor

<img width="475" alt="screen shot 2017-07-21 at 9 02 58 am" src="https://user-images.githubusercontent.com/22335838/28471874-6ac91fa8-6df3-11e7-845f-69c9b43c52e6.png">

### Step 4d: Populate the target analyze tree

1. Under File >> Import, select "FASTA"
2. Import the FASTA file used in Step 4c.

Skyline will keep the proteins, peptides, and transitions that match what it finds in the library provided in Step 4b.

<img width="246" alt="screen shot 2017-07-21 at 9 04 28 am" src="https://user-images.githubusercontent.com/22335838/28471929-a1037bb8-6df3-11e7-8c97-a1a47929d4d2.png">

### Step 4e: Adjust transition settings in Skyline

1. Settings >> Transition Settings >> Filter
2. Precursor charges: 1, 2, 3, 4, 5
3. Ion charges: 1 (this is the prevalent fragment, additional charges will increase interference)
4. Ion types: y, p
5. Product ions
  - From: ion 3
  - To: last ion -2

<img width="370" alt="screen shot 2017-07-21 at 9 06 28 am" src="https://user-images.githubusercontent.com/22335838/28472004-e6770df4-6df3-11e7-94b8-2c53c7984e92.png">

6. Settings >> Transition Settings >> Library
7. Ion match tolerance: 0.5 m/z
8. Unselect "if a library spectrum is available, pick its most intense ion"

<img width="421" alt="screen shot 2017-07-21 at 9 07 44 am" src="https://user-images.githubusercontent.com/22335838/28472096-219f3726-6df4-11e7-8d67-00cc4869714a.png">

9. Settings >> Transition Settings >> Full Scan
10. MS1 Filtering
  - Isotope peaks included: Count
  - Precursor mass analyzer: Pick the one that matches the instrument used for mass spectrometry
  - Resolving power: 60,000 at 200 m/z (Experiment-specific, consult methods)
11. MS/MS Filtering
  - Acquisition method: DIA
  - Product mass analyzer: Centroided
  - Mass accuracy: 15 ppm (Experiment-specific, consult methods)
  - Isolation scheme: Copy and paste list of isolation targets from Step 3b.
12. Retention Time Filtering
   - Use scans within 5 minute of MS/MS IDs

<img width="523" alt="screen shot 2017-07-21 at 9 12 55 am" src="https://user-images.githubusercontent.com/22335838/28472275-cdea9872-6df4-11e7-843c-bcf85b0f5d2b.png">

![img](https://github.com/grace-ac/grace-ac.github.io/blob/master/notebook-images/DIAStep4e-change-to-5mins.PNG)

### Step 4f: Import DIA data into Skyline

Under File > Import > Results, select "Add multi-injection replicates in files". Navigate to the directory with mzML files. Select parent directory with sample directories within it.


---

## Step 5: Within-Skyline Analyses

### Step 5a: Check that all QC peptides are chosen correctly

### Step 5b: Spot check peptides

1. Set up an Excel document for protein and peptide names as rows, and data file names as columns. An example can be found [here](https://github.com/RobertsLab/project-oyster-oa/blob/master/analyses/DNR_Skyline_20170524/error-checking/2017-06-10-error-checking.xlsx).
2. Randomly select about 100 peptides. Fill in the protein and peptide names in the Excel document.
3. For each peptide:
  - Identify the peak Skyline chose. The peak Skyline chose has a black arrow next to it.
  - If the correct peak was chosen, mark "1" in the Excel document.
  - If the wrong peak was chosen, or if Skyline identified noise as a peak, mark "0" in the Excel document. Then, select the correct peak in Skyline
    - Click on the correct peak
    - Right click, then select "Apply peak to all"
  - Shift peak boundaries so the entire peak is selected
    - Select the dashed lines around the peak, and drag to the correct location

Examples of correct and incorrect peaks can be found below.

Skyline picked the right peak (indicate with "1"):

![right-peak](https://cloud.githubusercontent.com/assets/22335838/26095588/fe990ed2-39d2-11e7-884e-47ad4eb78e2f.png)

This is the right peak because all of the transitions are eluting at the same retention time.

Skyline chose the wrong peak (indicate with "0"):

![wrong-peak](https://cloud.githubusercontent.com/assets/22335838/26095636/26fcdaca-39d3-11e7-8421-4978949643e6.png)

This is the wrong peak because only some of the transitions are eluting at that retention time. The correct peak would be the one on the right, in which all transitions are eluting at the same retention time.

Skyline identified noise as a peak (indicate with "0"):

![noise](https://cloud.githubusercontent.com/assets/22335838/26095664/3d39dcc0-39d3-11e7-9509-812cea8e71b2.png)

This is not a peak because there is only one transition eluting at that particular retention time, and all other transitions are not present (or are present at exremely low levels).

4. In the Excel document, calculate the following metrics:
  - Per-peptide success rate: Sum all "1" and "0" for each peptide (row), then divide by the number of samples in Skyline
  - Per-peptide error rate: 1 - success rate
  - Per-sample success rate: Sum all "1" and "0" for each sample (column), then divide by the number of peptides
  - Per-sample error rate: 1 - success rate
5. Send the spreadsheet to Emma Timmins-Schiffman for clearance before proceeding.

### Step 5c: Export Skyline data

Peak area integration values serve as a proxy for peptide and protein abundance, so it is important to export this information. Click File >> Export >> Report, select the export settings needed. Ensure the information is exported as a .csv file.

The RobertsLab Windows computer has several presets already established, including "Protein & Area, by replicate" and "SkylinetoMSstats". If analysis will not include MSstats, "Protein & Area, by replicate" may suffice.

![presets](https://user-images.githubusercontent.com/22335838/27842651-aa492a46-60c0-11e7-9f16-c7e0098fd022.png)

### Step 5d: Normalize peak areas

There are a few ways to do this:

1. From Emma: I do this by calculating the coefficients of variation for all my QC peptides across all samples. Since the same amount of QC was spiked into each sample, peak areas should be the same. I then select the QC peptides with CV<10 and I normalize all experimental peak areas by the average of those QC peak areas. You will then have a normalized dataset to analyze.
2. TIC: There is also an argument for normalizing by total ion current (TIC), which can be found on your raw files. TIC should correlate to amount of protein loaded into the mass spec and is probably an easier and more universal way of normalizing your data to differences in protein amount.
3. In MSstats, using the dataProcess function

