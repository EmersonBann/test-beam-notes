# Test beam re-calibration step 1 (data pclists) notes

24 November 2026

Used Claude to help write the commands

### Background

- Teresa's plan step 1 is "remake the data pclists"
- These are the cosmic muon files that the whole calibration starts from
- They have to be remade because they have the bug

Plan: find out how the pclist files were made

References:
- Robert Kralik's [TB calibration wiki](https://cdcvs.fnal.gov/redmine/projects/novaart/wiki/Test_Beam_Calibration_Instructions)
- The [job fcl](https://github.com/novaexperiment/novaprod/blob/aeda64936dc3840dc91e44eb396ca52227bfd4a2/novaproduction/fcl/testbeam/prod_tb_ddactivity1_pclist_job.fcl) linked in the wiki, `prod_tb_ddactivity1_pclist_job.fcl` from novaprod 
- [Test Beam data definitions wiki](https://cdcvs.fnal.gov/redmine/projects/novatestbeam/wiki/Data_Definitions)

### Questions / to-do

1. Will we use the same input definitions and files?
2. How will the re-make be run? Is novaprod access required to run the prod fcl? I may still have access from being a shifter
3. Need to learn how to put files into SAM, make sure metadata is being saved. Will ask prod convenors nearer the time of running things
4. Prestaging files, I know how to do this from prod shifting + recently it's changed so anyone can do it rather than just production team
5. Missing pclists (section 5 below): P2 19 subruns (18 no output at all, 1 pcliststop only), 3a 2 subruns. Should the remake cover them or just do the same as before? Or any ideas why they were left out?

### 1. What the fcl does

For each input file, `prod_tb_ddactivity1_pclist_job.fcl` runs two paths:

- `pchits`: removetbspills -> calhit -> slicer -> cosmictrack -> pclist
- `pcstophits`: removetbspills -> calhit -> slicer -> cosmictrack -> mefinder -> stop -> pcliststop

It writes two files per input file:
- `pclist_reco.root`
- `pcliststop_reco.root`

<div style="page-break-after: always;"></div>

### 2. Which pclist samples the current calibration used

From the data definitions wiki (pclist/pcstoplist section). One pclist + one pcliststop definition per sample:

| Sample | pclist definition |
|---|---|
| P2 | `prod_pclist_R23-04-05-testbeam-production.a_testbeam_ddactivity1_period2_v1` |
| 3a | `prod_pclist_R23-04-05-testbeam-production.a_testbeam_ddactivity1_epoch3a_v1` |
| 3b | `prod_pclist_R23-04-05-testbeam-production.a_testbeam_ddactivity1_epoch3b_v1` |
| 3c | `prod_pclist_R23-04-05-testbeam-production.a_testbeam_ddactivity1_epoch3c_v1` |
| 3d | `pclist_testbeam_detector_activity_epoch3d_R21-01-28-testbeam-production.a` |
| 3e | `pclist_testbeam_detector_activity_epoch3e_R21-01-28-testbeam-production.a` |
| P4 | `prod_pclist_R23-04-05-testbeam-production.a_testbeam_ddactivity1_period4_v1` |

pcliststop: same names with `pcliststop`

- Definitions wiki note: P2, 3a-3c and P4 were remade in April 2023 for a geometry fix. 3d and 3e were not affected, so they were kept from the older production.
- These are grouped for the attenuation fits as P2, P3 (3a-3c), P3 (3d-3e), P4 (ref calibration wiki).

### 3. How each definition is built

Looking at the definitions...

```
for D in <the 7 pclist definitions above>
do
  echo "== $D"
  samweb count-definition-files $D
  samweb describe-definition $D | grep Dimensions
done
```

| Sample | Files | Query |
|---|---|---|
| P2 | 16,713 | `defname: base_testbeam_detector_activity_period2 and online.detector testbeam and nova.release 'R23-04-05-testbeam-production.a' and data_tier pclist` |
| 3a | 10,558 | `defname: base_testbeam_detector_activity_epoch3a and online.detector testbeam and nova.release 'R23-04-05-testbeam-production.a' and data_tier pclist` |
| 3b | 638 | `defname: base_testbeam_detector_activity_epoch3b and online.detector testbeam and nova.release 'R23-04-05-testbeam-production.a' and data_tier pclist` |
| 3c | 337 | `defname: base_testbeam_detector_activity_epoch3c and online.detector testbeam and nova.release 'R23-04-05-testbeam-production.a' and data_tier pclist` |
| 3d | 1,410 | `defname: base_testbeam_detector_activity_epoch3d and data_tier pclist and nova.release 'R21-01-28-testbeam-production.a'` |
| 3e | 4,296 | `defname: base_testbeam_detector_activity_epoch3e and data_tier pclist and nova.release 'R21-01-28-testbeam-production.a'` |
| P4 | 21,910 | `defname: base_testbeam_detector_activity_period4 and online.detector testbeam and nova.release 'R23-04-05-testbeam-production.a' and data_tier pclist` |
| Total | 55,862 | |

Use: `samweb count-files "<query>"` or `samweb list-files "<query>"`

- Every definition is the whole sample for that period/epoch. Nothing is excluded at this stage.
- So epoch 4a and the masking-study runs (left out of the P4 calibration, ref calib wiki) must be dropped later, at the attenuation fit.

<div style="page-break-after: always;"></div>

### 4. What they were made from

`get-metadata` on one file per definition shows its parent file, release and who made it.

```
for D in <the 7 pclist definitions above>
do
  F=$(samweb list-definition-files $D | head -1)
  echo "== $D"
  samweb get-metadata $F | grep -E "File Name|Create Date|User:|NOVA.Release|sam_project_name|Parents"
done
```

| Sample | Made | Release | SAM project | Parent |
|---|---|---|---|---|
| P2 | 17 Apr 2023 | R23-04-05-testbeam-production.a | `novapro-testbeam_ddactivity1_p2_e3abc_p4_calib-20230417_0534` | ddactivity1 artdaq, R19-09-24 (.f) |
| 3a | 17 Apr 2023 | R23-04-05-testbeam-production.a | `novapro-testbeam_ddactivity1_p2_e3abc_p4_calib-20230417_0615` | ddactivity1 artdaq, R21-01-28 |
| 3b, 3c | 17 Apr 2023 | R23-04-05-testbeam-production.a | `novapro-testbeam_ddactivity1_p2_e3abc_p4_calib-20230417_0644` | ddactivity1 artdaq, R21-01-28 |
| 3d | 4 May 2021 | R21-01-28-testbeam-production.a | `alister1-alister1-calib-keepup-Fermigrid-R21-01-28-testbeam-production.a-testbeam-ddactivity1-3_days_ago-20210504_0530` | ddactivity1 artdaq, R21-01-28 |
| 3e | 18 May 2021 | R21-01-28-testbeam-production.a | `alister1-alister1-calib-keepup-Fermigrid-R21-01-28-testbeam-production.a-testbeam-ddactivity1-3_days_ago-20210518_0530` | ddactivity1 artdaq, R21-01-28 |
| P4 | 17 Apr 2023 | R23-04-05-testbeam-production.a | `novapro-testbeam_ddactivity1_p2_e3abc_p4_calib-20230417_0644` | ddactivity1 artdaq, R21-07-27 |

- User is `novapro` for all
- Inputs are the cosmic (DDActivity1) artdaq files, in three releases
- P2, 3a-3c and P4 were made by production in one batch (17 Apr 2023, release R23-04-05)
- Looks like 3d and 3e came from a calibration keep-up job by Adam Lister
- The fcl name isn't in the metadata, but the wiki says it's `prod_tb_ddactivity1_pclist_job.fcl`

<div style="page-break-after: always;"></div>

### 5. Input files

The inputs are the cosmic activity artdaq definitions listed on the data definitions wiki. 

| Sample | Input artdaq definition | artdaq files | pclist files | artdaq but no pclist |
|---|---|---|---|---|
| P2 | `artdaq_testbeam_detector_activity_period2_R19-09-24-testbeam-production.f` | 16,732 | 16,713 | 19 |
| 3a | `artdaq_testbeam_detector_activity_epoch3a_R21-01-28-testbeam-production.a` | 10,560 | 10,558 | 2 |
| 3b | `artdaq_testbeam_detector_activity_epoch3b_R21-01-28-testbeam-production.a` | 638 | 638 | 0 |
| 3c | `artdaq_testbeam_detector_activity_epoch3c_R21-01-28-testbeam-production.a` | 337 | 337 | 0 |
| 3d | `artdaq_testbeam_detector_activity_epoch3d_R21-01-28-testbeam-production.a` | 1,410 | 1,410 | 0 |
| 3e | `artdaq_testbeam_detector_activity_epoch3e_R21-01-28-testbeam-production.a` | 4,296 | 4,296 | 0 |
| P4 | `artdaq_testbeam_detector_activity_period4_R21-07-27-testbeam-production.a` | 21,910 | 21,910 | 0 |

#### 5a. Definition counts

```
samweb count-definition-files artdaq_testbeam_detector_activity_period2_R19-09-24-testbeam-production.f

for E in 3a 3b 3c 3d 3e; do samweb count-definition-files artdaq_testbeam_detector_activity_epoch${E}_R21-01-28-testbeam-production.a; done

samweb count-definition-files artdaq_testbeam_detector_activity_period4_R21-07-27-testbeam-production.a
```

-> 16732; 10560, 638, 337, 1410, 4296; 21910.

#### 5b. Which subruns have no pclist

- List the artdaq and pclist files, cut each name to run + subrun (`sed 's/_ddactivity1.*//'`), sort, save to `$LOGS/checks/`
- Compare the two lists with `comm` (columns: 1 = only artdaq, 2 = only pclist, 3 = both)
  - P2, P3: `comm -23` -> subruns with artdaq but no pclist
  - P4: `comm -3` -> subruns missing from either list. Printed nothing, so they're identical

P2:

```
samweb list-files "defname: base_testbeam_detector_activity_period2 and data_tier artdaq and nova.release 'R19-09-24-testbeam-production.f'" | sed 's/_ddactivity1.*//' | sort > $LOGS/checks/p2-artdaq-subruns.txt
samweb list-definition-files prod_pclist_R23-04-05-testbeam-production.a_testbeam_ddactivity1_period2_v1 | sed 's/_ddactivity1.*//' | sort > $LOGS/checks/p2-pclist-subruns.txt
comm -23 $LOGS/checks/p2-artdaq-subruns.txt $LOGS/checks/p2-pclist-subruns.txt
```

P3 epochs:

```
for E in 3a 3b 3c 3d 3e
do
  if [ $E = 3d ] || [ $E = 3e ]; then PREL=R21-01-28-testbeam-production.a; else PREL=R23-04-05-testbeam-production.a; fi
  samweb list-files "defname: base_testbeam_detector_activity_epoch$E and data_tier artdaq and nova.release 'R21-01-28-testbeam-production.a'" | sed 's/_ddactivity1.*//' | sort > $LOGS/checks/e$E-artdaq-subruns.txt
  samweb list-files "defname: base_testbeam_detector_activity_epoch$E and data_tier pclist and nova.release '$PREL'" | sed 's/_ddactivity1.*//' | sort > $LOGS/checks/e$E-pclist-subruns.txt
  echo "== epoch$E  artdaq: $(wc -l < $LOGS/checks/e$E-artdaq-subruns.txt)  pclist: $(wc -l < $LOGS/checks/e$E-pclist-subruns.txt)"
  echo "-- artdaq but no pclist:"
  comm -23 $LOGS/checks/e$E-artdaq-subruns.txt $LOGS/checks/e$E-pclist-subruns.txt
done
```

P4:

```
samweb list-definition-files artdaq_testbeam_detector_activity_period4_R21-07-27-testbeam-production.a | sed 's/_ddactivity1.*//' | sort > $LOGS/checks/p4-artdaq-subruns.txt
sed 's/_ddactivity1.*//' $LOGS/checks/pclist-p4-files.txt | sort > $LOGS/checks/p4-pclist-subruns.txt
comm -3 $LOGS/checks/p4-artdaq-subruns.txt $LOGS/checks/p4-pclist-subruns.txt
# returned nothing
```

Missing subruns:
- P2 (19):
```
r100416 s63              r100565 s04              r100583 s04
r100446 s00              r100577 s04              r100584 s04
r100488 s01              r100578 s04              r100588 s04
r100561 s04+s05          r100579 s04+s05          r100589 s04
r100563 s05              r100580 s04              r100591 s04
r100564 s04              r100581 s04
```
- 3a (2):
```
r101104 s14
r101153 s03
```

<div style="page-break-after: always;"></div>

#### 5c. The 19 P2 subruns in more detail

Event count of each artdaq file, and the last subrun of its run in the definition:

```
for S in $(comm -23 $LOGS/checks/p2-artdaq-subruns.txt $LOGS/checks/p2-pclist-subruns.txt)
do
  R=${S%_s*}
  LAST=$(grep $R $LOGS/checks/p2-artdaq-subruns.txt | tail -1)
  N=$(samweb get-metadata ${S}_ddactivity1_R19-09-24-testbeam-production.f_v1_data.artdaq.root | grep "Event Count")
  echo "$S   last subrun: $LAST   $N"
done
```

- The subruns are not empty: 774 to 16,497 events each (140,259 in total).
- 15 are the last subrun of their run; 4 are not (r100446 s00, r100488 s01, r100561 s04, r100579 s04).
- 16 of 19 are in runs 100561 to 100591.

Do they have a pcliststop?

```
samweb list-definition-files prod_pcliststop_R23-04-05-testbeam-production.a_testbeam_ddactivity1_period2_v1 | sed 's/_ddactivity1.*//' | sort > $LOGS/checks/p2-pcliststop-subruns.txt
comm -12 <(comm -23 $LOGS/checks/p2-artdaq-subruns.txt $LOGS/checks/p2-pclist-subruns.txt) $LOGS/checks/p2-pcliststop-subruns.txt
```

-> only r100488 s01. The other 18 have no output at all. r100488 s01 has a pcliststop but no pclist, which is odd.


