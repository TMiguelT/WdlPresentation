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

#This task calls GATK's tool, HaplotypeCaller in normal mode. This tool takes a pre-processed 
#bam file and discovers variant sites. These raw variant calls are then written to a vcf file.
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
* ![](https://software.broadinstitute.org/wdl/img/WDL-workflow.png)
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
A workflow can consist of a:
* Scatter (a parallelized loop):
    ```
    scatter(i in integers) {
      call task1{input: num=i}
      call task2{input: num=task1.output}
    }
    ```
+++
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
+++
A workflow can consist of a:
---
## Syntax - Tasks
![](https://software.broadinstitute.org/wdl/img/WDL-task-variables.png)
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
---
## Strengths

---
## Weaknesses
---
## Comparison
### CWL

### Bpipe
