# WDL

![](https://software.broadinstitute.org/wdl/resources/img_shared/wdl-logo_white.png)

> a workflow language meant to be read and written by humans

* Developed the the Broad Institute
* First publicly released on [31 March 2016](https://software.broadinstitute.org/gatk/blog?id=7349)

---
## Syntax - Example
```scala
workflow helloHaplotypeCaller {
  call haplotypeCaller
}

task haplotypeCaller {
  File GATK
  File RefFasta
  File RefIndex
  File RefDict
  String sampleName
  File inputBAM
  File bamIndex
  command {
    java -jar ${GATK} \
      -T HaplotypeCaller \
      -R ${RefFasta} \
      -I ${inputBAM} \
      -o ${sampleName}.raw.indels.snps.vcf
  }
  output {
    File rawVCF = "${sampleName}.raw.indels.snps.vcf"
  }
}
```
---
## Syntax - Overall Structure

* WDL scripts fundamentally consists of one `workflow` block, and one or more `task` blocks (stages):
* These `task`s are executed using the `call` command

![](https://software.broadinstitute.org/wdl/img/WDL-workflow.png)
---
## Syntax - Workflow
A workflow can consist of a:
* Call
    ```scala
    workflow mutect2 {
      call indexBam {input: bam=sortBam.sortedBam}
      bamIndex = indexBam.index
    }
    ```
+++
## Syntax - Workflow
A workflow can consist of a:
* Scatter (a parallelized loop):
    ```scala
    workflow mutect2 {
      Array[Array[File]] samples = [tumourSamples, normalSamples]
      scatter(sample in samples) {
        call align {input: read1=sample[0], read2=sample[1], reference=genomeReference, referenceIndices=genomeIndices}
      }
    }
    ```
+++
## Syntax - Workflow
A workflow can consist of a:
* Output:
    This is optional. If this isn't defined, all outputs from all `call`s are used as outputs.
    ```scala
    workflow w {
      String w_input = "some input"
      
      call t
      call t as u
      
      output {
        String t_out = t.out
        String u_out = u.out
        String input_as_output = w_input
        String previous_output = u_out
      }
    }
    ```
+++
## Syntax - Workflow
A workflow can consist of a:
* Conditional
    ```scala
    # Call 'y', producing an Int output, in a conditional block:
    if (x_out) {
      call y
      Int y_out = y.out
    }
    ```
---
## Syntax - Tasks
![](https://software.broadinstitute.org/wdl/img/WDL-task-variables.png)
+++
## Syntax - Tasks
A task consists of:
* Inputs
    ```scala
    task indexBam {
      File bam
    }
    ```
+++
A task consists of:
* Command
    ```scala
    task indexBam {
      File bam
        
      command {
        samtools index -b ${bam} -
      }
    }
    ```
+++
A task consists of:
* Output
    ```scala
    task indexBam {
      File bam
        
      command {
        samtools index -b ${bam} -
      }

      output {
        File index = stdout()
      }
    }
    ```
+++
A task consists of:
* Runtime
    ```scala
    task indexBam {
      File bam
        
      command {
        samtools index -b ${bam} -
      }

      output {
        File index = stdout()
      }

      runtime {
        docker: "biocontainers/samtools"
      }
    }
    ```
---
## Cromwell
* https://github.com/broadinstitute/cromwell
* Cromwell is the only official WDL execution engine
* Single Jar file
* Supports multiple execution environments
* Does not support all WDL features (imports etc)
+++
## Cromwell
```
cromwell 29
Usage: java -jar /path/to/cromwell.jar [server|run] [options] <args>...

  --help                   Cromwell - Workflow Execution Engine
  --version                
Command: server
Starts a web server on port 8000.  See the web server documentation for more details about the API endpoints.
Command: run [options] workflow-source
Run the workflow and print out the outputs in JSON format.
  workflow-source          Workflow source file.
  -i, --inputs <value>     Workflow inputs file.
  -o, --options <value>    Workflow options file.
  -t, --type <value>       Workflow type.
  -v, --type-version <value>
                           Workflow type version.
  -l, --labels <value>     Workflow labels file.
  -p, --imports <value>    A directory or zipfile to search for workflow imports.
  -m, --metadata-output <value>
                           An optional directory path to output metadata.
```
---
## Strengths
+++
## Strengths - Community
* [Actively developed](https://github.com/broadinstitute/cromwell/graphs/contributors), by a large organisation
* [Official WDL scripts for Broad tools (GATK, Mutect etc)](https://github.com/broadinstitute/gatk-protected/tree/master/scripts/mutect2_wdl)
* [Large community already](http://gatkforums.broadinstitute.org/wdl/categories/ask-the-wdl-team), meaning plenty of support
+++
## Strengths - Simplicity
* Very easy to write workflows, with only unix command-line experience
* Very easy to install (single Jar file) and run (`java -jar Cromwell.jar run myWorkflow.wdl myWorkflow_inputs.json`
* Forced simplicity in workflows
+++
## Strengths - Portability
* Uses Docker for easy dependency management
* [Compatible with a large number of systems](https://github.com/broadinstitute/cromwell#backends)
* [Work on CWL compatibility](https://github.com/broadinstitute/wdl4s/pull/120) and [conversion](https://github.com/common-workflow-language/wdl2cwl)
---
## Weaknesses
+++
## Weaknesses - Debugging
* No interactive debugger
* [Confusing logging](https://raw.githubusercontent.com/TMiguelT/WdlPresentation/master/errorlog.txt)
```
[2017-07-07 03:30:44,58] [info] BackgroundConfigAsyncJobExecutionActor [64eaf010mutect2.sortBam:0:1]: executing: docker run \
  --rm -i \
   \
  --entrypoint /bin/bash \
  -v /mnt/WdlMutect2/cromwell-executions/mutect2/64eaf010-ee80-4172-858e-782ed9845fc7/call-sortBam/shard-0:/cromwell-executions/mutect2/64eaf010-ee80-4172-858e-782ed9845fc7/call-sortBam/shard-0 \
  biocontainers/samtools@sha256:2847c6d09a61c2c6a7554bf7f8fb0b44052f768517dc8eca435f21cd2e74256a /cromwell-executions/mutect2/64eaf010-ee80-4172-858e-782ed9845fc7/call-sortBam/shard-0/execution/script
[2017-07-07 03:30:44,60] [info] BackgroundConfigAsyncJobExecutionActor [64eaf010mutect2.sortBam:0:1]: job id: 20030
[2017-07-07 03:30:44,60] [info] BackgroundConfigAsyncJobExecutionActor [64eaf010mutect2.sortBam:0:1]: Status change from - to WaitingForReturnCodeFile
[2017-07-07 03:30:46,27] [info] BackgroundConfigAsyncJobExecutionActor [64eaf010mutect2.sortBam:0:1]: Status change from WaitingForReturnCodeFile to Done
[2017-07-07 03:30:46,34] [error] WorkflowManagerActor Workflow 64eaf010-ee80-4172-858e-782ed9845fc7 failed (during ExecutingWorkflowState): Job mutect2.sortBam:0:1 exited with return code 1 which has not 
been declared as a valid return code. See 'continueOnReturnCode' runtime attribute for more details.
Check the content of stderr for potential additional information: /mnt/WdlMutect2/cromwell-executions/mutect2/64eaf010-ee80-4172-858e-782ed9845fc7/call-sortBam/shard-0/execution/stderr
[2017-07-07 03:30:46,34] [info] WorkflowManagerActor WorkflowActor-64eaf010-ee80-4172-858e-782ed9845fc7 is in a terminal state: WorkflowFailedState
[2017-07-07 03:30:46,53] [info] Message [cromwell.engine.workflow.lifecycle.execution.WorkflowExecutionActor$CheckRunnable$] from Actor[akka://cromwell-system/user/SingleWorkflowRunnerActor/WorkflowManage
rActor/WorkflowActor-64eaf010-ee80-4172-858e-782ed9845fc7/WorkflowExecutionActor-64eaf010-ee80-4172-858e-782ed9845fc7#-704046265] to Actor[akka://cromwell-system/user/SingleWorkflowRunnerActor/WorkflowMan
agerActor/WorkflowActor-64eaf010-ee80-4172-858e-782ed9845fc7/WorkflowExecutionActor-64eaf010-ee80-4172-858e-782ed9845fc7#-704046265] was not delivered. [1] dead letters encountered. This logging can be tu
rned off or adjusted with configuration settings 'akka.log-dead-letters' and 'akka.log-dead-letters-during-shutdown'.
[2017-07-07 03:30:48,32] [info] Message [cromwell.backend.async.AsyncBackendJobExecutionActor$IssuePollRequest] from Actor[akka://cromwell-system/user/SingleWorkflowRunnerActor/WorkflowManagerActor/Workfl
owActor-64eaf010-ee80-4172-858e-782ed9845fc7/WorkflowExecutionActor-64eaf010-ee80-4172-858e-782ed9845fc7/64eaf010-ee80-4172-858e-782ed9845fc7-EngineJobExecutionActor-mutect2.align:1:1/64eaf010-ee80-4172-8
58e-782ed9845fc7-BackendJobExecutionActor-64eaf010:mutect2.align:1:1/BackgroundConfigAsyncJobExecutionActor#1713928586] to Actor[akka://cromwell-system/user/SingleWorkflowRunnerActor/WorkflowManagerActor/
WorkflowActor-64eaf010-ee80-4172-858e-782ed9845fc7/WorkflowExecutionActor-64eaf010-ee80-4172-858e-782ed9845fc7/64eaf010-ee80-4172-858e-782ed9845fc7-EngineJobExecutionActor-mutect2.align:1:1/64eaf010-ee80-
4172-858e-782ed9845fc7-BackendJobExecutionActor-64eaf010:mutect2.align:1:1/BackgroundConfigAsyncJobExecutionActor#1713928586] was not delivered. [2] dead letters encountered. This logging can be turned of
f or adjusted with configuration settings 'akka.log-dead-letters' and 'akka.log-dead-letters-during-shutdown'.
[2017-07-07 03:31:04,46] [info] SingleWorkflowRunnerActor workflow finished with status 'Failed'.
[2017-07-07 03:31:04,49] [error] Outgoing request stream error
akka.stream.AbruptTerminationException: Processor actor [Actor[akka://cromwell-system/user/StreamSupervisor-1/flow-13-0-unknown-operation#896307532]] terminated abruptly
[2017-07-07 03:31:04,49] [error] Outgoing request stream error
akka.stream.AbruptTerminationException: Processor actor [Actor[akka://cromwell-system/user/StreamSupervisor-1/flow-9-0-unknown-operation#-1309172815]] terminated abruptly
Workflow 64eaf010-ee80-4172-858e-782ed9845fc7 transitioned to state Failed
```
+++
## Weaknesses - Continued
* The execution engine is a server (?)
* People need to stop inventing DSLs!
* [Miniscule standard library](https://github.com/broadinstitute/wdl/blob/develop/SPEC.md#table-of-contents)
* Still quite an immature language (features like Objects and imports are [still in development](https://github.com/broadinstitute/wdl/blob/develop/SPEC.md#loops))
+++
## Weaknesses - Continued
* Their home page looks like this:
https://software.broadinstitute.org/wdl/
---
## Comparison
### CWL
* [Stages](http://www.commonwl.org/draft-3/UserGuide.html#First_example)
* [Workflows](http://www.commonwl.org/draft-3/UserGuide.html#First_workflow)

### Ruffus
* [Example](http://www.ruffus.org.uk/examples/bioinformatics/part2_code.html)
