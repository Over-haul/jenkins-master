# Base Jenkins Master Container

## Building on this container

If you need more specific configuration than this, use the following Dockerfile as a template.

File `casc_configs/auth_config.yml` is specifically to **overwrite** the base container.

### Installing additional plugins

You can add plugin configurations by adding one or more to `jenkins-plugins.yaml` and running `/bin/jenkins-plugin-cli -f /path/to/jenkins-plugins.yaml`.

Plugin files need to be in the format specified by the base jenkins/jenkins container and described in the [Jenkins Docker README file](https://github.com/jenkinsci/docker/blob/master/README.md#plugin-version-format).

### Adding additional Configuration as Code

The Configuration as Code ([CasC](https://github.com/jenkinsci/configuration-as-code-plugin)) plugin is installed as part of the Jenkins Master container and will read all configuration files out of the `/usr/share/jenkins/ref/casc_configs/` directory. The CasC plugin docs have [examples](https://github.com/jenkinsci/configuration-as-code-plugin/tree/master/demos) for how to configure many popular plugins.

### Additional configuration with groovy

Groovy scripts can be added to `/usr/share/jenkins/ref/init.groovy.d/` to be run when Jenkins starts up. This is a good way to provide repeatable configuration if you need something outside of additional plugins or features coverered by the CasC plugin. Please note that scripts in this directory are run in alphabetical order so if you are relying on scripts from more than one layer of Docker images be sure to name your files appropraitely.

### `jenkins/jenkins:lts` container environment variables

|               Environment Variable | Default Value                                                                        | Description |
|                                --- | ---                                                                                  |         --- |
|                             `PATH` | `/opt/java/openjdk/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin` |           - |
|                             `LANG` | `en_US.UTF-8`                                                                        |           - |
|                         `LANGUAGE` | `en_US:en`                                                                           |           - |
|                           `LC_ALL` | `en_US.UTF-8`                                                                        |           - |
|                     `JAVA_VERSION` | `jdk8u282-b08`                                                                       |           - |
|                        `JAVA_HOME` | `/opt/java/openjdk`                                                                  |           - |
|                     `JENKINS_HOME` | `/var/jenkins_home`                                                                  |           - |
|         `JENKINS_SLAVE_AGENT_PORT` | `50000`                                                                              |           - |
|                              `REF` | `/usr/share/jenkins/ref`                                                             |           - |
|                  `JENKINS_VERSION` | `2.263.4`                                                                            |           - |
|                       `JENKINS_UC` | `https://updates.jenkins.io`                                                         |           - |
|          `JENKINS_UC_EXPERIMENTAL` | `https://updates.jenkins.io/experimental`                                            |           - |
| `JENKINS_INCREMENTALS_REPO_MIRROR` | `https://repo.jenkins-ci.org/incrementals`                                           |           - |
|          `COPY_REFERENCE_FILE_LOG` | `/var/jenkins_home/copy_reference_file.log`                                          |           - |

### `JCasC` environment variables

| Environment Variable | Default Value | Description |
|                  --- | ---           |         --- |
| `JENKINS_PUBLIC_URL` | -             |           - |
|          `SLACK_URL` | -             |           - |
|   `GITHUB_API_TOKEN` | -             |           - |


#### Add groovy scripts

```Dockerfile
# Add in team specific groovy configs
ADD groovy_scripts/10_some_script.groovy /usr/share/jenkins/ref/init.groovy.d/
```

## `docker-compose`

### `secrets.env`

Contains:

```text
GITHUB_API_TOKEN=<Your Token>
```

### `bitnami/openldap`

This image is used for local development, if you want a smaller container image you may remove `ldap` plugin from `plugins/01_base_auth_plugins.txt` and build again.

### Run locally

`docker-compose up` will give you a master that can run docker jenkins agents using label `docker-agent`

### Example: declare a shared library

```yaml
# Example below from https://github.com/jenkinsci/configuration-as-code-plugin/blob/master/demos/jenkins/jenkins.yaml#L63-L70A
# https://github.com/jenkinsci/configuration-as-code-plugin/blob/master/integrations/src/test/resources/io/jenkins/plugins/casc/GlobalLibrariesGitHubTest.yml
unclassified:
  globalLibraries:
    libraries:
      - name: "<name your users will reference in @Library()>"
        defaultVersion: "<default branch to checkout>"
        retriever:
          modernSCM:
            scm:
              github:
                repoOwner: "<your GitHub Org name>"
                repository: "<your jenkins library repository name>"
                credentialsId: "github-api-username" // assuming you are using the default credentials
```

### Example: create Multibranch Pipeline Job

```yaml
# Example below from https://github.com/jenkinsci/configuration-as-code-plugin/tree/master/demos/jobs
jobs:
  - script: >
      multibranchPipelineJob('<repository name>') {
        branchSources {
          github {
            // The id option in the Git and GitHub branch source contexts is now mandatory (JENKINS-43693).
            id('12312313') // IMPORTANT: use a constant and unique identifier
            scanCredentialsId('github-api-username') // assuming you are using the default credentials
            repoOwner('<your GitHub Org name>')
            repository('<repository name>')
          }
        }
      }
```

More at [jenkins-job-dsl](https://jenkinsci.github.io/job-dsl-plugin/)
