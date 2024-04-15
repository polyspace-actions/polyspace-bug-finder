# Action for Running Polyspace Bug Finder Analysis

The [polyspace-bug-finder](#polyspace-bug-finder) action enables you to configure and run a Polyspace&reg; Bug Finder&trade; Server&trade; analysis and upload results to Polyspace Access&trade; (Uploads require Polyspace Access API key). The action supports only [self-hosted](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/about-self-hosted-runners) runners.


## Examples

### Run a Polyspace Bug Finder Server Analysis and Upload Results to Polyspace Access

You can use the `polyspace-bug-finder` action to analyze your files each time you push your changes to the `main` branch. The action takes the path of a compilation database file `compilation-database-file` as an input. This action assumes the paths to the Polyspace binaries are on your system `PATH` variable.

```yaml
name: Run Polyspace analysis on push
on: [push]
  branches:
    - 'main'
jobs:
  my-job:
    name: Run analysis
    runs-on: self-hosted

    steps:
    - name: Check out repository
      uses: actions/checkout@v3
    - name: Generate compilation database
      uses: cmake -DCMAKE_EXPORT_COMPILE_COMMANDS=ON .
    - name: Analyze
      uses: polyspace-actions/polyspace-bug-finder@24.1.0
      with:
        compilation-database-file: compile_commands.json
        checkers-file: checkers.xml
        host: ${{ env.ACCESS_HOST }}
        api-key: ${{ secrets.API_KEY }}
        project-name: ${{github.ref_name}}
        parent-project: ${{github.event.repository.name}}
```

### Analyze Only Files That Changed in Pull Request

Use the action to reduce the scope of the analysis to only consider files that have changes when you open a pull request. In this example, the Polyspace installation folder path (`/opt/Polyspace`) is provided with input `polyspace-installation-folder`.

```yaml
name: Run Polyspace Analysis
on:
  pull_request:
    types: [opened, reopened, edited, synchronize]
jobs:
  my-job:
    name: Run analysis
    runs-on: self-hosted

    steps:
    - name: Check out repository
      uses: actions/checkout@v3
    - name: Generate compilation database
      uses: cmake -DCMAKE_EXPORT_COMPILE_COMMANDS=ON .
    - name: Analyze pull request
      uses: polyspace-actions/polyspace-bug-finder@24.1.0
      with:
        polyspace-installation-folder: /opt/Polyspace
        compilation-database-file: compile_commands.json
        checkers-file: checkers.xml
        pull-request-reduced-analysis: true
        host: ${{ env.ACCESS_HOST }}
        api-key: ${{ secrets.API_KEY }}
        project-name: ${{github.head_ref}}_pr
        parent-project: ${{github.event.repository.name}}
```


## polyspace-bug-finder

When you define your workflow in the `.github/workflows` folder of your repository, specify this action as `polyspace-actions/polyspace-bug-finder@24.1.0`.

The action accepts these inputs. Unless otherwise specified, the inputs are optional.

#### **General Options**

Input                     | Description
------------------------- | ---------------
|`polyspace-installation-folder`| Polyspace installation folder path. If you add the path of the Polyspace executables to your `PATH` environment variable, you do not need to specify the installation folder path.<br>**Example:** `/usr/local/Polyspace/R2024a`|
| `working-directory`|Working directory folder path. Specify the path relative to `GITHUB_WORKSPACE`. If you do not specify a working directory, Polyspace uses `GITHUB_WORKSPACE`.|

#### **Configuration and Analysis Options**

Input                     | Description
------------------------- | ---------------
| `compilation-database-file`|Compilation database file path. You generate this file from your build system, for instance by using the command `cmake -DCMAKE_EXPORT_COMPILE_COMMANDS=ON`. The generated file contains compiler calls for all the translation units in your project. This input is not compatible with the `build-command` input. Polyspace uses the information from this file to extract your build system configuration. See also [Create Polyspace Options File from JSON Compilation Database](https://www.mathworks.com/help/bugfinder/ref/polyspaceconfigurecommand.html#mw_5d7170e9-e889-4504-b1f9-e7873663829a).<br>**Example:** `compile_commands.json`|
| `build-command`| Build command that you specify to perform a full build. Polyspace executes the build command and traces your build to extract your build system configuration. This input is not compatible with the `compilation-database-file` input.<br> **Example:** `make -B`|
| `checkers-file`| Checker activation file path. You can configure this XML file to specify which checkers to enable for the analysis. See also [Checkers activation file (-checkers-activation-file)](https://www.mathworks.com/help/bugfinder/ref/checkersactivationfilecheckersactivationfile.html). <br> **Example:** `checkers_activation.xml`  |
| `results-dir`| Path of folder where generated results are stored. The default results folder is `ps_results`.|
| `options-file`| Polyspace analysis options file path. If you use multiple options files, specify the path of one options file that references all the other options files.|
| `sarif-file`| Path of the generated SARIF file. The default name of the results file is `results.sarif`.|

#### **Polyspace Access Options**

Input                     | Description
------------------------- | ---------------
| `api-key`| The API Key used to log into Polyspace Access. See also [General User Manager Settings](https://www.mathworks.com/help/polyspace_access/install/configure-the-user-manager.html#mw_9d5be066-c809-463b-9fa8-b6a6a3ba2a8f)<br> **Example:** `0a17b7f1-4c91-4ae9-b74f`|
| `host`|Fully qualified host name of the machine that hosts Polyspace Access.<br>**Example:** `jsmith.example.machine.com`|
| `protocol`| HTTP protocol used by Polyspace Access to communicate with client machines. The default Polyspace Access configuration uses HTTPS. <br> **Example:** `https`|
| `port`| Port number used by Polyspace Access to communicate with client machines. The default Polyspace Access configuration uses port 9443. <br> **Example:** `1234`|
| `project-name`| Name of the uploaded results in Polyspace Access project explorer.<br> **Example:** `polyspace_example_bf`|
| `parent-project`|Name of the project folder under which the uploaded results are stored in Polyspace Access project explorer.

#### **Advanced Options**

Input                     | Description
------------------------- | ---------------
| `configure-extra-options`|Pass additional options to the [`polyspace-configure`](https://www.mathworks.com/help/bugfinder/ref/polyspaceconfigurecommand.html) executable. For instance, you can specify a custom compiler configuration file. <br> **Example:** `-compiler-config myCompiler.xml`|
| `bugfinder-extra-options`|Pass additional options to the [`polyspace-bug-finder-server`](https://www.mathworks.com/help/bugfinder/ref/polyspacebugfinderservercommand.html) executable. For instance, you can specify a configuration file that associates behaviors with code elements such as functions. <br> **Example:** `-code-behavior-specifications code-behavior-spec.xml` |
| `access-extra-options`| Pass additional options to the [`polyspace-access`](https://www.mathworks.com/help/bugfinder/ref/polyspaceaccess.html) executable. For instance, you can add a label to the uploaded results.<br> **Example**: `-label myLabel`|

#### **Push and Pull Request Options**

Input                     | Description
------------------------- | ---------------
|`push-reduced-analysis`|Set this input to `true` to analyze files only if they are in the list of changed files pushed to the repository and the file extensions match one of the extensions specified with `source-file-allow-list` input. By default, this input is set to `false`.|
| `pull-request-reduced-analysis`|Set this input to `true` to analyze files only if they are in the list of changed files of a pull request and the file extensions match one of the extensions specified with `source-file-allow-list`. By default, this input is set to `false`|
| `pull-request-number` | Pull request number that the analysis runs on if you set `pull-request-reduced-analysis` to `true` and run this action in a pull request context. If you do not specify a pull request number, the action uses the default value `${{ github.event.pull_request.number }}`.|
| `github-token`| Access token generated by Github for authentication in the workflow job. Polyspace uses the token for calculating the list of files that changed if either `push-reduced-analysis` or `pull-request-reduced-analysis` is set to `true`. The default value for this input is `${{ github.token }}`.|
| `source-file-allow-list`| Comma separated list of file extensions. Polyspace only analyzes files with the specified extensions. By default, the list includes these extensions for c and c++ sources and header files: `.cpp,.cxx,.hpp,.c,.h,.cc,.hh`.|

## See Also

- [Complete List of Polyspace Bug Finder Analysis Engine Options](https://www.mathworks.com/help/bugfinder/analysis-options.html)
- [Complete List of Polyspace Bug Finder Results](https://www.mathworks.com/help/bugfinder/check-reference.html)

## Contact Us

If you have any questions or suggestions, please contact MathWorks® at [continuous-integration@mathworks.com](mailto:continuous-integration@mathworks.com).
