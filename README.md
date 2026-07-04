# Reqtify Plugin for Jenkins

[Reqtify](https://www.3ds.com/) is a requirements traceability and reporting tool from Dassault Systèmes. This plugin integrates Reqtify with Jenkins so that a build can:

1. Generate a Reqtify report (**Reqtify: Generate Report** build step).
2. Call an arbitrary Reqtify function exposed by the opened project (**Reqtify: Call Function** build step).

Both build steps are also available as Pipeline steps for use in a `Jenkinsfile`.

## Prerequisites

* **Windows only.** The plugin locates `reqtify.exe` via the Reqtify COM registration (`HKCR\Reqtify.Application\CLSID`) and starts Reqtify in HTTP server mode (`-http`), so the Jenkins **controller** must be running Windows with Reqtify installed and licensed.
* **Reqtify version 2021x** or later is required.
* The Jenkins controller workspace for the job must already contain a Reqtify project (a `.rqtf` file) before either build step runs — the plugin asks Reqtify to open a project from that directory; it does not create one.
* The plugin manages a single background Reqtify process per language on the controller, so only one Reqtify project can be open at a time per language across builds.

## Configure Reqtify report generation build step

In the project configuration page, add a **Reqtify: Generate Report** build step.

![build_step](https://github.com/jenkinsci/reqtify-plugin/blob/master/images/generate_report.png)

This allows to fill following fields:

![reportModelsAndReportTemplates](https://github.com/jenkinsci/reqtify-plugin/blob/master/images/generate_report_build_step.PNG)

* **Report Name** - This is the report file name without any path or suffix. This report file will be created at the root of the Jenkins workspace.

* **Report Model** - This is the report model. The list contains both library and project report models. Choosing a model populates the report's own parameters below the field (scalar parameters render as text boxes, non-scalar parameters render as multi-select lists) — the same list is fetched live from Reqtify, so it can change from project to project.

* **Report Template** - This is the report template. The list contains both library and project report templates like HTML, DOCX, Excel, PDF etc.

* **Project Filter** - Optional. Restricts report generation to a named filter defined in the Reqtify project (for example to only include passed/failed tests). Leave it on "Select Project Filter" to generate the report without a filter.

## Configure calling function build step

In the project configuration page, add a **Reqtify: Call Function** build step.

![build_step](https://github.com/jenkinsci/reqtify-plugin/blob/master/images/call_function.PNG)

This allows to fill following fields:

![reportModelsAndReportTemplates](https://github.com/jenkinsci/reqtify-plugin/blob/master/images/call_function_build_step.png)

* **Function Name** - This is the function name to call during the build, selected from the list of functions exposed by the Reqtify project.

* **Function parameters** - Selecting a function populates its parameters below the field, fetched live from Reqtify. Scalar parameters (e.g. **aReq**) render as a text box; non-scalar parameters (e.g. **anIndex**) render as a multi-select list. The exact parameter names and count depend on the selected function.

## Pipeline (Jenkinsfile) usage

Both build steps are also available as Pipeline steps.

Generate a report with `reqtifyReport`:

```groovy
pipeline {
    agent { label 'windows' }
    stages {
        stage('Reqtify report') {
            steps {
                reqtifyReport(
                    nameReport: 'Report',
                    modelReport: 'MyReportModel',
                    templateReport: 'HTML',
                    reportArgumentList: ['ns_1', 'value']
                )
            }
        }
    }
}
```

Call a Reqtify function with `reqtifyFunction`:

```groovy
pipeline {
    agent { label 'windows' }
    stages {
        stage('Reqtify function') {
            steps {
                reqtifyFunction(
                    functionName: 'myFunction',
                    argumentList: ['ns_1', 'value']
                )
            }
        }
    }
}
```

`reportArgumentList` / `argumentList` entries are grouped by type, not positional: entries prefixed with `ns_` are collected into a non-scalar argument group and all other entries into a scalar argument group, and each group is sent to Reqtify separately. Relative order is preserved within each group, but not between scalar and non-scalar entries in the original list. Use **Pipeline Syntax** generator in Jenkins to build these snippets against your own project's models/functions, since the available names are specific to each Reqtify project.

Note: the Pipeline `reqtifyReport` step does not currently expose the **Project Filter** field available in the freestyle build step.

## Notes

* The plugin only works when a Reqtify project is present in the Jenkins workspace.
* **Reqtify version required: 2021x**
* A Windows Jenkins controller with a licensed Reqtify installation is required, see [Prerequisites](#prerequisites).
