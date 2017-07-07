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
## Syntax - Structure

* WDL workflows fundamentally consists of one `workflow` block, and one or more `task` blocks (stages):
* These `task`s are executed using the `call` command

![](https://software.broadinstitute.org/wdl/img/WDL-workflow.png)
---
## Syntax - Workflow
A workflow can consist of a:
* Call
    ```
    workflow wf {
      call my_task
    }
    ```
+++
## Syntax - Workflow
A workflow can consist of a:
* Scatter (a parallelized loop):
    ```
    scatter(i in integers) {
      call task1{input: num=i}
      call task2{input: num=task1.output}
    }
    ```
+++
## Syntax - Workflow
A workflow can consist of a:
* Conditional
    ```
    # Call 'y', producing an Int output, in a conditional block:
    if (x_out) {
      call y
      Int y_out = y.out
    }
    ```
+++
## Syntax - Workflow
A workflow can consist of a:
* Output:
    This is optional. If this isn't defined, all outputs from all `call`s are used as outputs.
    ```
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
---
## Syntax - Tasks
![](https://software.broadinstitute.org/wdl/img/WDL-task-variables.png)
+++
## Syntax - Tasks
A task consists of:
* Inputs
    ```
    task test {
      String flags
    }
    ```
+++
A task consists of:
* Command
    ```
    task test {
      String flags
      command {
        ps ${flags}
      }
    }
    ```
+++
A task consists of:
* Output
    ```
    task test {
      command {
        python script.py
      }
      output {
        String a = "a"
        String ab = a + "b"
      }
    }
    ```
+++
A task consists of:
* Runtime
    ```
    task test {
      command {
        python script.py
      }
      runtime {
        docker: ["ubuntu:latest", "broadinstitute/scala-baseimage"]
      }
    }
    ```
---
## Strengths
+++
## Strengths - Community
* Actively developed, by a large organisation
* Official WDL scripts for Broad tools (GATK, Mutect etc)
* Large community already, meaning plenty of support
+++
## Strengths - Simplicity
* Very easy to write workflows, with only unix command-line experience
* Very easy to install (single Jar file)
* Forced simplicity in workflows
+++
## Strengths - Portability
* Uses Docker for easy dependency management
* Compatible with a large number of systems
* Work on CWL compatibility
---
## Weaknesses
+++
## Weaknesses - Debugging
* No interactive debugger
* Confusing logging:
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
* The execution engine is a server (?)
* People need to stop inventing DSLs!
* Miniscule standard library
* Still quite an immature language (features like Objects and imports are still in development)
---
## Comparison
### CWL

### Bpipe
