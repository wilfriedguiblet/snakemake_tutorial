# Overview
Learn some of the basics of Snakemake through the following tutorial.

1. Create a script to run Snakemake
2. Create variables to run a snakemake_config file
3. Create rules for three scenarios
4. Use script to invoke Snakemake

# Manifest Files
Manifest files have already been created in the `/snakemake_tutorial/manifest` directory. This includes

sample_manifest.csv
```
sample_id,file_name
sample1,sample1.fq
sample2,sample2.fq
```

# Activity
The task can be broken up into A. pre-processing, B. sample handling, C. rule creation, and D. Advanced Commands. All edits should be completed in the `/snakemake_tutorial/pipeline_todo/` directory.

## A. Pre-Processing 

- Create two different Snakemake commands, one for a dry run and one for a local run to the `run_snakemake.sh`
- Include the path to the workflow/Snakefile, the config/snakemake_config.yaml
- Include flags --printshellcmds, --verbose, --rerun-incomplete to both commands
- Include flag --cores 1 for the `run` command

## B. Sample Handling

- Create the parameters in the `config/snakemake_config.yaml`
```
'sampleManifest' which gives the path of the sampleManifest

'out_dir' which gives the path to the output dir (must exist)

'data_dir' which gives the path to the data dir found under "/snakemake_tutorial/pipeline_todo/data/"
```
- Create the sample dictionaries and project lists from the manifest in the `workflow/Snakefile`
```
`CreateSampleDicts` creates a dictionary matching sample_id to filename

`CreateProjLists` creates a project lists `sp_list` which contains all sample_ids and `file_list` which contains all filenames
```

## C. Create rules

- Create rule_all for each rule one at a time in the `workflow/Snakefile`.
- Create rule_all input for all input files, from the `file_list`
- Perform dry runs and complete runs between each rule creation.
- Rule A
    - input files should be `{sample_id}.fq`
    - output should be `{sample_id}_rulea.txt` and should be output to the `out_dir`
    - shell command should add a line "ruleA completed on a new line" to the original file
- Rule B
    - input files should be `get_input_files`
    - this definition will look up the name of the fq by taking in the `sample_id` as a wildcard, and using the `samp_dict`
    - output should be `{sample_id}_ruleb.txt` and should be output to the `out_dir`
    - shell command should add a line "ruleB completed on a new line" to the original file
- Rule C
    - input files should be all of Rule A's output files
    - output should be `merged_rulea.txt` and should be output to the `out_dir/final_output`
    - params should be def `get_rulec_cmd` which iterates through all samples and creates a command `cat {sample1}_rulea.txt {sample2}_rulea.txt >> {final_file}` 
    - shell command should touch the `{final_file}`, then run the `cmd` parameter
- Rule D
    - input files should be directly linked to Rule B's output files
    - output should be `{sample_id}_copied_ruleb.txt` and should be output to the `out_dir/final_output`
    - params should be def `get_ruled_cmd` which iterates through all samples and creates a command `cp /output/path/{sample_id}_ruleb.txt /output/path/final_output/{sample_id)_copies_ruleb.txt;` for each sample
    - shell command should run the `cmd` parameter

## D. Advanced commands

- Add features to the `run_snakemake.sh` file to include:
    - check if the output dir is created; if not create it
    - copy the snakemake_config.yaml file to the output dir; run this file for all snakemake runs
    - update the snakemake_config.yaml file with the output_dir given from the command line
    - update the snakemake_config.yaml file with the pipeline_dir based on the invocation of the pipeline

- Add features to the `workflow/Snakefile`:
    - flag rule A and rule B files so they are deleted after the pipeline completion

# Hints / Reminders
- Rule A and rule B are using the same input files, but only differ in how these files are being referenced. There are times when the sample_id of an input file will match, but other times (as when taking in a multiplexed ID when they will not be the same). Rule A handles cases where they match, rule B handles cases where they would not match.
- Use the expand feature for Rule C's input to gather all required input files
- Snakemake will automatically create directories that don't exist, such as the `out_dir/final_output` when they are listed as `output`
- Use the sp_list that was created to iterate through all of the samples to create the ruleC cmd
- Use the `rules.RuleName.output.OutputName` feature of Snakemake to link output and input files together

The output of the dry-run for all rules should look as follows:
```
job      count    min threads    max threads
-----  -------  -------------  -------------
A            2              1              1
B            2              1              1
C            1              1              1
D            2              1              1
all          1              1              1
total        8              1              1
```
