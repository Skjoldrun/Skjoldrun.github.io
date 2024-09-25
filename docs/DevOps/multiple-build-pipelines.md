---
layout: page
title: DevOps - Multiple build pipelines
parent: DevOps
---

# Multiple Build Pipelines 
DevOps can have multiple CI (continuous integration) pipelines for a project repository at once. You can create a pipe that gets triggered only for commits or pull requests into the master / main branch and another pipeline which gets triggered for a development branch only.


## YAML files

I have created multiple YAML files, each for a specific pipeline in my repo. The chosen names can show the targeted branch to trigger the pipe, e.g. `build-pipeline-main.yml` and `build-pipeline-development.yml`.

[![YAML Pipeline files](/assets/images/articles/DevOps/DevOps_yaml_pipeline_files.png)](/assets/images/articles/DevOps/DevOps_yaml_pipeline_files.png)

Each of these two files has a complete definition of a build pipe. Check the article [CI/CD Pipeline](/docs/DevOps/cicd-pipeline.md) for more details.


## DevOps create a pipe with an existing YAML file

After committing these files, you have to create or alter a pipeline to link to one of these YAML files as pipeline configuration. 

If you want to create a new pipeline, click the [New Pipeline] button, select the location of your repository and then choose the "Existing Azure Pipelines YAML file" option:

[![existing YAML file option](/assets/images/articles/DevOps/DevOps_existing_yaml_pipeline_file.png)](/assets/images/articles/DevOps/DevOps_existing_yaml_pipeline_file.png)

Then choose the branch and the path to the file:

[![existing YAML file path](/assets/images/articles/DevOps/DevOps_existing_yaml_pipeline_file_path.png)](/assets/images/articles/DevOps/DevOps_existing_yaml_pipeline_file_path.png)

You could edit the file content and then run and save it.


## DevOps alter existing pipe to use a YAML file or change it

If you have an existing pipeline and want to change it to a specific YAML file or change the target YAML file, then enter the pipe and click on the [...] button to either rename or move the pipeline for better organization, or enter the settings menu to alter the YAML target file:

[![Pipeline settings](/assets/images/articles/DevOps/DevOps_change_existing_pipeline.png)](/assets/images/articles/DevOps/DevOps_change_existing_pipeline.png)

