# MetaG Testing

Metagenome Workflow (MetaG) testing was performed on a San Diego Super Computing (SDSC) server nmdc-edge.org as was the Metatranscriptomic workflow testing.
MetaG requires a working instance of [Docker](https://www.docker.com/) as well as [cromwell](https://github.com/broadinstitute/cromwell) to run the workflow. 
All third party depedencies are pulled down in the containers included in the end to end worflow.


## MetaG Inputs:

The inputs json files for MetaG testing were extended from the current [input](https://github.com/microbiomedata/metaG/blob/main/inputs.json) to ensure 
parameter inputs per each step in the workflow.

- `input_files`: Full path to the fastq file. The files can be interleaved or paired end.
- `input_fq1`: Path to R1 paired end fastq. Leave as [] if using interleaved.
- `input_fq2`: Path to R2 paired end fastq. Leave as [] if using interleaved.
- `input_interleaved`: Set true if input_files is/are interleaved, or if using paired end set False. 
- `DoReadsQC`: Set to true to turn on [ReadsQC](https://github.com/microbiomedata/ReadsQC) in workflow, else set False. 
- `readsQC_outdir`: Full path to output directory for ReadsQC step of MetaG. 
- `DoReadbasedAnalysis`: Set to true to turn on [ReadbasedAnalysis](https://github.com/microbiomedata/ReadbasedAnalysis) in workflow, else set False.
- `readbasedAnalysis_outdir`: Fall path to output directory for ReadbasedAnalysis output of workflow. 
- `readbasedAnalysis_prefix`: File prefix for Readbased Analysis step
- `readbasedAnalysis_paired"`: Set true if input is paired end, else set false. 
- `readbasedAnalysis_enabled_tools`: Set Tools options to true or false in Key:Value style to determine what tools are applied: {"gottcha2":true/false,"kraken2":true/fale,"centrifuge":true/false}
- `DoMetaAssembly`: Set true to run [Assembly](https://github.com/microbiomedata/metaAssembly) step of MetaG, else set false. 
- `metaAssembly_rename_contig_prefix`: Set contig naming schema for assembly step
- `metaAssembly_outdir`: Full path to assembly output directory for this step in workflow. 
- `DoAnnotation`: Set true to run [Annotation](https://github.com/microbiomedata/mg_annotation) step of workflow, else set false. 
- `metaAnnotation_outdir`: Full path for annotation output directory.
- `metaAnnotation_imgap_project_id`: Prefix for file naming schema for annotation output. 
- `DoMetaMAGs`: Set true to run Metagenome Assembled Genomes [metaMAGs](https://github.com/microbiomedata/metaMAGs) step, else set false.
- `metaMAGs_outdir`: Full path for metaMAGS outdir.
- `metaMAGs_proj_name"`: Prefix for file names. 
- `metaMAGs_map_file`: User can provide mapping file in txt format, else set to null.
- `metaMAGs_domain_file`: User can provide domain file, else set to null. 

Example of input:

```json
    {
    "main_workflow.input_files":["$edge_home/test/test_metag/Ecoli_10x-int.fastq.gz"],
    "main_workflow.input_fq1":[],
    "main_workflow.input_fq2":[],
    "main_workflow.input_interleaved":true,
    "main_workflow.DoReadsQC":true,
    "main_workflow.readsQC_outdir":"$edge_home/test_metag/ReadsQC",
    "main_workflow.DoReadbasedAnalysis":true,
    "main_workflow.readbasedAnalysis_outdir":"$edge_home/test_metag/ReadbasedAnalysis",
    "main_workflow.readbasedAnalysis_prefix":"test",
    "main_workflow.readbasedAnalysis_paired":true,
    "main_workflow.readbasedAnalysis_enabled_tools":{"gottcha2":true,"kraken2":true,"centrifuge":true},
    "main_workflow.DoMetaAssembly":true,
    "main_workflow.metaAssembly_rename_contig_prefix":"test",
    "main_workflow.metaAssembly_outdir":"$edge_home/test_metag/MetagenomeAssembly",
    "main_workflow.DoAnnotation":true,
    "main_workflow.metaAnnotation_outdir":"$edge_home/test_metag/MetagenomeAnnotation",
    "main_workflow.metaAnnotation_imgap_project_id":"test",
    "main_workflow.DoMetaMAGs":true,
    "main_workflow.metaMAGs_outdir":"$edge_home/test_metag/MetagenomeMAGs",
    "main_workflow.metaMAGs_proj_name":"test",
    "main_workflow.metaMAGs_map_file":null,
    "main_workflow.metaMAGs_domain_file":null
    }
```

There are 3 requiremnents needed ro run MetaG command:

- `workflowSource`: Full path to input.json file for MetaT
- `workflowInputs`: Full path to .wdl. This is the main workflow contianing tasks to run MetaT.
- `worfkowDependencies`: Zip file containing all WDL dependencies. Needs to be provided to allow main workflow imports for metaT.wdl

The **imports.zip** file was created by running ``` zip -r imports.zip wdls/``` for MetaG depedencies. 


## Running MetaG

With inputs prepared, MetaG is run by calling ``` curl http://localhost:8000/api/workflows/v1 -F "workflowSource=@$edge_home/test/test_metag/metaG_pipeline.wdl" -F "workflowInputs=@$edge_home/test/test_metag/pipeline_inputs.json" -F "workflowDependencies=@$edge_home/test/test_metag/imports.zip" ```

The curl command sends a request to the Cromwell API workflow submission endpoint with the required files, workflowSource, workflowInputs and 
workflowDependencies included as ``` -F, --form options ```.

Once the command is submitted, the workflow can be followed by either query the contents of systemd (super user permissions needed):
``` sudo journalctl -f -n100 -u cromwell.service ```

Or by tracking the progress of the workflow in the logfile generated by cromwell which can be found by its unique ID:

``` /expanse/projects/nmdc/cromwell/cromwell-workflow-logs/workflow.3a6d3d62-2801-4c4a-ba36-e89781d8ab36.log ```

The unique ID, in format: ``` 3a6d3d62-2801-4c4a-ba36-e89781d8ab36 ``` is generated as soon as the job is submitted and is viewable right away on the terminal 
screen. 

## Outputs

All outputs follow the structure of included workflows ReadsQC, ReadbasedAnalysis, MetagenomeAssembly, MetagenomeAnnotation, and MetagenomeMAGs. Output can be 
found where _outdir was specified in inputs for each workflow step.

Expected outputs can be found each workflow step here:
[ReadsQC](https://github.com/microbiomedata/ReadsQC)
[ReadbasedAnalysis](https://github.com/microbiomedata/ReadbasedAnalysis) 
[MetagenomeAssembly](https://github.com/microbiomedata/metaAssembly)
[MetagenomeAnnotation](https://github.com/microbiomedata/mg_annotation)
[MetagenomeMAGs](https://github.com/microbiomedata/metaMAGs)

## Navigating Files for MetaG Testing on nmdc-edge

- `Test Directory`: $edge_home/test/
- `MetaG Test Directory`: $edge_home/test/test_metag
- `MetaG Input File`: $edge_home/test_metag/pipeline_inputs.json
- `MetaG Main WDL`: $edge_home/test/test_metag/metaG_pipeline.wdl
- `MetaG Imports File`: $edge_home/test/test_metag/imports.zip
- `MetaG WDL's`: $edge_home/test/test_metag/wdls

For the job to run successfully, all data mounts must be located within ```$edge_home/ ``` for the cromwell configuration see the inputs 
and allow permissions. 
