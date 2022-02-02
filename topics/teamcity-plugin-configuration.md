[//]: # (title: TeamCity plugin configuration)

The main Qodana functionality comes from the 'engine' shaped into the Docker image. If you want to go beyond the boundaries of the default settings, refer to the [Docker image guide](https://www.jetbrains.com/help/qodana/?qodana-jvm-docker-readme). Note that you don't need to write `docker run` on your own: the plugin will do it for you. You can just use all other options and provide them via the dedicated UI or DSL properties.

## Add Qodana analysis to your project builds on TeamCity

### Prerequisites

- You use TeamCity as a build server for your project. If not, learn how to do it in [TeamCity documentation](https://www.jetbrains.com/help/teamcity/teamcity-documentation.html).
- Your project language is included in the list of fully [supported technologies](https://www.jetbrains.com/help/qodana/supported-technologies.html).
- The [Qodana plugin](https://plugins.jetbrains.com/plugin/15498-qodana) is installed on your TeamCity server (you can do it yourself or contact the server's administrator to do this).


### (Optional) Add a configuration script
{id="add-script"}

Custom profile configuration for Qodana linters is stored in `qodana.yaml`. When using a CI system, you need to put this file to the working directory manually. Alternatively, you can write a script that will do it for you.

1. On TeamCity left navigation panel, select your project and go to **Edit configuration | Build Steps**.

2. Select **Add build step | Command Line**.

3. For **Step name**, specify an optional name, for example, "Add qodana.yaml".

4. For **Run**, select **Custom script**.

5. In the **Custom script** editor, write a script that adds a custom `qodana.yaml` to the working directory. 

   In the example below, the script appends the following inspection exclusions to the configuration file:

```shell
#!/bin/sh

FILE="./qodana.yaml"

/bin/cat <<EOM >$FILE
exclude:
  - name: Annotator
  - name: AnotherInspectionId
    paths:
      - relative/path
      - another/relative/path
  - name: CloneFinder
  - name: ProhibitedDependencyLicense
  
EOM  
```

### Add a Qodana runner

<!-- Documentation link for the Inspection profile needs to be improved: https://helpserver.labs.jb.gg/help/qodana/2021.3/qodana-yaml.html#Default+profiles -->
<!-- Report ID - where is this ID available? -->
<!-- Why is it needed to forward reports to TeamCity Tests? -->
<!-- Documentation about 'qodana run' parameters needs to be created -->
<!-- Additional arguments for 'docker run' - I need to supply the notation for this field. -->
<!-- Link to the entry point arguments needs to be improved, currently it doesn't work properly -->


2. Select **Add build step**.
3. Fill in the fields according to this table:

Assuming that you have already created your [project](https://www.jetbrains.com/help/teamcity/configure-and-run-your-first-build.html#Create+your+first+project) and configured your build, follow the steps below.    

1. Navigate to your build configuration.
   <img src="teamcity-plugin-1.png" alt="Navigating to the build configuration" width="706" border-effect="line"/>

2. On the build configuration page, select **Build Steps** from the **Edit configuration** list.
   <img src="teamcity-plugin-2.png" alt="Navigating to the step configuration" width="706" border-effect="line"/>

3. On the **Build steps** page, click the **Add build step** button.
   <img src="teamcity-plugin-3.png" alt="Creating a new build step" width="706" border-effect="line"/>

4. Using the **Runner type** list, select **Qodana** as a runner. On the **New Build Step** page, you can configure the `Qodana` runner
using basic options. Otherwise, click **Show advanced options** to expose all configuration options.
   <img src="teamcity-plugin-4.png" alt="Exposing all configuration options of the Qodana runner" width="706" border-effect="line"/>
   
5. Fill in the fields using this description.

   **Step name** uniquely identifies this step among other build steps.

   **Execute step** configures the build condition that will trigger this build step.
   
   **Working directory** sets the directory for the build process. For more information, see the [TeamCity](https://www.jetbrains.com/help/teamcity/2021.2/build-working-directory.html) documentation. You can leave this field empty if the `Checkout directory` parameter is specified on the **Version Control Settings** tab.

   **Report ID** identifies the report in case several inspection steps are configured for the build.

   **Fail threshold** configures the maximum number of problems accepted by Qodana without failing a build, specified as an integer value. For more information, see the [Quality gate](quality-gate.xml) section.

   The **Forward reports to TeamCity Tests** checkbox configures that all Qodana inspection report will be forwarded to TeamCity.

   The **Enable** checkbox enables the `Qodana` runner. 

   **Linter** configures the Qodana linter. For details, see the [Linters](linters.md) section.

   **Version** is by default set to `Latest`.

   **Inspection profile** defines the inspection profile. For more information, see the [Configure profile](qodana-yaml.md) section.
   The available values are:
      * `Recommended (default)` is the default profile containing a preselected set of IntelliJ inspections 
      * `Embedded profile` lets you select from any available profiles, see the [Default profiles](https://www.jetbrains.com/help/qodana/qodana-yaml.html#Default+profiles) section for details
      * `Path to the IntelliJ profile` lets you specify the path to a custom profile. Make sure that the `.idea` directory containing the profile file is added to your working directory.

   You can disable certain inspections later using the [`qodana.yaml`](https://www.jetbrains.com/help/qodana/qodana-yaml.html#exclude-paths) file or [Profile settings](https://www.jetbrains.com/help/qodana/ui-overview.html#Adjust+your+inspection+profile) in your HTML report.

   **Additional arguments for 'docker run'** configures the arguments accepted by a Docker image. For example, they can be the `-d` or `-changes` parameters.

   **Additional Qodana arguments** lets you extend the default Qodana functionality, see the [Configuration options](qodana-jvm-docker-techs.xml#Configuration+options) for details.

6. Click **Save**. Now you can run Qodana in the build.


## Add more runners to your build
You can add inspections by [Clone Finder](https://www.jetbrains.com/help/qodana/about-clone-finder.html) or [License Audit](https://www.jetbrains.com/help/qodana/about-license-audit.html) to your build steps.

When viewing analysis results for a specific build later, you can disable certain runners and inspections via [`qodana.yaml`](https://www.jetbrains.com/help/qodana/qodana-yaml.html) and update the [configuration script](#add-script).

### Prerequisites for adding more runners

- The Qodana plugin for teamCity is installed.
- The Clone Finder or License Audit plugins for teamCity are installed as necessary.

## Advanced configuration

[//]: # "delete? supplement? ...todo: Failure Conditions based on Qodana metrics"

Advanced configuration lets you report all found problems via the standard TeamCity tests mechanism. It means
you can assign investigations, mute, see history, and do everything else you can do with regular tests in TeamCity. Qodana reports tests in four different ways:

- per problem
- per inspection type
- per inspection type/per module (if this information is available)
- per inspection type/per file

## Logs

On TeamCity, open your project build page and go to the **Build Log** tab. Here you can find a standard shell output where errors are highlighted in yellow.

For more details, go to the **Artifacts** tab, where more detailed logs for TeamCity and Qodana are provided.

For more about TeamCity logs, see [TeamCity documentation](https://www.jetbrains.com/help/teamcity/teamcity-documentation.html).