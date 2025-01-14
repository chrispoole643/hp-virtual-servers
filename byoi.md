---

copyright:
  years: 2020, 2021
lastupdated: "2022-01-19"

subcollection: hp-virtual-servers

keywords: image, virtual server instance, instance, virtual server

---
{:external: target="_blank" .external}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:pre: .pre}
{:table: .aria-labeledby="caption"}
{:codeblock: .codeblock}
{:note: .note}
{:tip: .tip}
{:important: .important}

# Using your own image
{: #byoi}

Use your own Linux-based OCI image to create a new Hyper Protect Virtual Server. This feature is available by using the {{site.data.keyword.cloud}} CLI.
{: shortdesc}


The {{site.data.keyword.cloud}} CLI can be used only on the officially supported operating systems or architectures. It is recommended that you use the {{site.data.keyword.cloud}} Shell if your system is not supported.
{: note}

If you want to use {{site.data.keyword.cloud}} {{site.data.keyword.hpvs}} Secure Build Server to provision a server, refer to [Protecting your image build](/docs/hp-virtual-servers?topic=hp-virtual-servers-imagebuild).


Do not use personal information, for example, your name, as the instance name or as part of the instance name. The data that you provide when you provision an instance or interact with the hpvs cli is not considered to be personal data or credentials.
Learn more about IBM Cloud Hyper Protect Virtual Servers' Data usage and Certifications [here](https://www.ibm.com/software/reports/compatibility/clarity-reports/report/html/softwareReqsForProduct?deliverableId=165C6EF0FFDA11E8BABD512A6952EE1F).
{: note}

Before you provision a new server, check the [prerequisites](/docs/services/hp-virtual-servers?topic=hp-virtual-servers-byoi#byoi_pre), then complete the following steps:

1. [Create a custom image](/docs/services/hp-virtual-servers?topic=hp-virtual-servers-byoi#byoi_create).
2. [Creating a registration definition file by using the CLI](/docs/services/hp-virtual-servers?topic=hp-virtual-servers-byoi#byoi_regdef_cli).  
3. [Use your OCI image to provision a Hyper Protect Virtual Server](/docs/services/hp-virtual-servers?topic=hp-virtual-servers-byoi#byoi_provision).


## Prerequisites
{: #byoi_pre}

- Your custom Linux-based image must meet these requirements:
    - In the OCI Image format. {{site.data.keyword.hpvs}} supports only Linux-based OCI Images, which are built for the IBM LinuxONE and IBM Z platform (s390x architecture)
    - Available either in the [{{site.data.keyword.cloud}} Container Registry](https://cloud.ibm.com/kubernetes/catalog/registry) or [Docker Hub](https://hub.docker.com/) and signed by using [Docker Content Trust](https://docs.docker.com/engine/security/trust/content_trust/).
- You must install the {{site.data.keyword.cloud_notm}} CLI. Follow these [instructions](https://cloud.ibm.com/docs/cli?topic=cli-getting-started) to install and configure the {{site.data.keyword.cloud_notm}} CLI.

## Creating a custom image
{: #byoi_create}

1. Choose a base for your OCI Image. Either use one of the [publicly available images from Docker Hub](https://hub.docker.com/search?q=&type=image&image_filter=&architecture=s390x) or build one from scratch.
2. Use a build tool such as `docker build` to make your required modifications to the base. You can use Ubuntu to install these tools on a Hyper Protect Virtual Server if you don't have access to a system that runs on the IBM Z platform (s390x architecture), or you can't cross build your image. To install Docker on your Hyper Protect Virtual Server, run `apt update && apt install -y docker.io`. Be aware of these hints when you build your image:
    - The maximum allowed image size is 5 GB.
    - Set the entrypoint and command that is used to start the OCI image during the image build because these parameters can’t be configured afterward to provision your virtual server.
    - {{site.data.keyword.cloud}} Registry namespaces and Docker Hub organizations with a `-` in the name are not permitted.
    - The OCI image content is available on the virtual server boot disk that is mounted as `/`. If you need to use persistent storage, use the `/data` mount point.
    - If you want to use `systemd` in your image, make sure you set the start command to `/sbin/init` and configure the environment variable `RUNQ_SYSTEMD=1`.
    - The virtual server gets its own IP address, which means that all open ports are automatically mapped on the internal and public network. Make sure you restrict network access in your image to only the ports you require. `EXPOSE` rules from the image build are not applied.
    - The image must be tagged according to the following format to later “push” the image to the registry:
     `<registry url>/<namespace>/<name>:<tag>`
     where:
      - registry url: Within ICR: An example is `de.icr.io`, or for DockerHub: `docker.io`.
      - namespace: Is the [namespace created](https://cloud.ibm.com/docs/Registry?topic=Registry-getting-started#gs_registry_namespace_add) in the {{site.data.keyword.cloud_notm}} Container Registry, see also step 4 of this procedure.
      - name: Is the unique name for the repository (one repository can have multiple images that differ by the tag).
      - tag: Is the unique tag for the image.
3. After you create your image, check for vulnerable components. Most containers are built by using a collection of open source components that can suffer from known vulnerabilities. It is your responsibility to use scanning tools to identify if any of the components you include in the build are vulnerable. Scan the components before you distribute the image, for example, with [Vulnerability Advisor](https://cloud.ibm.com/docs/Registry?topic=va-va_index).
4. Push the image to either the {{site.data.keyword.cloud_notm}} Registry or Docker Hub with Docker Content Trust enabled. To do this using the {{site.data.keyword.cloud_notm}} Container Registry, follow these steps:
    - Follow the [instructions to create an API key](https://cloud.ibm.com/docs/account?topic=account-userapikey#create_user_key).
    - Follow the [instructions to create a namespace](https://cloud.ibm.com/docs/Registry?topic=Registry-getting-started#gs_registry_namespace_add) in {{site.data.keyword.cloud_notm}} Container Registry.
    - To log in to the registry on your console, run:
      ```sh
      echo "<API_Key>" | docker login -u "iamapikey" --password-stdin <registry_region>.icr.io
      ```
      {: pre}

    - Push the image by running:
      ```sh
      DOCKER_CONTENT_TRUST=1 DOCKER_CONTENT_TRUST_SERVER=https://notary.<registry_region>.icr.io docker push <image>
      ```
      {: pre}

5. Follow the [backup instructions](https://cloud.ibm.com/docs/Registry?topic=Registry-registry_trustedcontent#trustedcontent_backupkeys) to back up your keys after you push the image.


## Creating a registration definition file by using the CLI
{: #byoi_regdef_cli}

The registration definition file contains metadata about the OCI image you want to use for your Hyper Protect Virtual Server, such as the repository name and the credentials to pull the image. Because it can contain secret information, the file is  automatically encrypted and signed by the CLI before you can use it to create a new Virtual Server. Use the following commands to create an encrypted registration definition file.

Before you call the `hpvs registration-key-create` command, `gpg` must be installed on your system.

1. To create a `gpg` registration key, run the `hpvs registration-key-create` command. The resulting output files are the required inputs for the `hpvs registration-create` command.

   ```sh
   ibmcloud hpvs registration-key-create ID [--gpg-passphrase-path FILE-PATH] [-v VERBOSE]
   ```
   {: pre}

   Where:

   `ID`
   :   Is the user ID to set for the gpg registration key.

   `--gpg-passphrase-path FILE-PATH`
   :   Is the path for the file that contains the passphrase for the registration key. The passphrase must consist of at least 6 characters. To make sure that a new line is not appended, use `echo` with `-n` or cat with EOF. If the path is not specified, you are prompted for the passphrase.

   `-v, --verbose`
   :   Set to true for verbose output.

2. To create a registration file, run the `hpvs registration-create` command.

   ```sh
   ibmcloud hpvs registration-create [--repository-name REPO-NAME] [--cr-username USER-NAME --cr-pwd-path FILE-PATH | --no-auth] [--allowed-env-keys ENV-KEYS | --no-env] [--image-key-id IMAGE-KEY-ID] [--image-key-public-path PUBLIC-KEY] [--registration-key-private-path PRIVATE-KEY-PATH] [--registration-key-public-path PUBLIC-KEY-PATH] [--gpg-passphrase-path PASS-PHRASE] [--cap-add  CAPABILITIES] [--isv-secrets ISV-SECRETS | --no-isv-secrets]
   ```
   {: pre}

   If you enter the command without any parameters:

   ```sh
   ibmcloud hpvs registration-create
   ```
   {: pre}

   You are prompted to enter all the parameters.

   If the container registry does not require authentication, set the `-no-auth` parameter to prevent prompting. If no environment parameters are required, set the `-no-env` parameter, for example:

   ```sh
   ibmcloud hpvs registration-create --no-env --no-auth
   ```
   {: pre}

   Where:
   `--repository-name REPO-NAME`
   :   Is the fully qualified name for the repository.

   `--cr-username USER-NAME`
   :   Is the username for the login on the container repository. It can be any string of 4 - 30 characters.

   `--cr-pwd-path FILE-PATH`
   :   Is the path for the file that contains the container repository password.

   `--no-auth`
   :   Is the parameter that must be set if the image does not require authorization to download. In this case,  you don't need to provide `cr-username` and `cr-pwd-path` parameters. If you do, these parameters are ignored.

   `--allowed-env-keys ENV-KEYS`
   :   Specifies the allowed environment variable keys as a comma-separated list. The keys must be strings of 1 - 64 characters.

   `--no-env`
   :   This parameter can be set if the image does not require any allowed environment variables. In this case,  you don't need to provide the `allowed-env-keys` parameter. If you do, it is ignored.

   `--image-key-id IMAGE-KEY-ID`
   :   The ID of the root key that was used to sign the image. It must contain 64 characters. If the image-key-id is not specified, the command first tries to determine the ID automatically by requesting the container registry notary service. Optional.

   `--image-key-public-path PUBLIC-KEY`
   :   The path for the file that contains the public part of the key that is used to sign the image.  The public part of the key must be a minimum of 20 characters and base64 encoded. If the path is not specified, the command first tries to determine the public part of the key automatically by requesting the container registry notary service. Optional.

   `--registration-key-public-path PRIVATE-KEY-PATH`
   :   The path for the public key from the registration key pair.

   `--registration-key-private-path PUBLIC-KEY-PATH`
   :   The path for the private key from the registration key pair.

   `--gpg-passphrase-path PASS-PHRASE`
   :   The path for the `gpg` pass phrase used for the private part of the registration key. The passphrase must consist of at least 6 characters. To make sure that a new line is not appended, use `echo` with `-n` or `cat` with EOF.

   `--cap-add CAPABILITIES`
   :   The Linux capabilities to be enabled are specified as a comma separated list.

   `--isv-secrets ISV-SECRETS`
   :   The Linux secrets to be used in BYOI. The secrets are added in the `/isv_secrets/secrets.json` file, within the container.

   The ISV secrets is a key value pair that is separated by a colon, and you can specify a list secrets that are separated by a comma. Avoid adding spaces after the comma when you are specifying multiple secrets. If the size of the secret is large, it is recommended that you specify it by using the `--isv-secrets` command parameter. To update ISV secrets, you must create a new registration definition file with the updated ISV secrets and use the same registration definition file when running the `hpvs instance-update` command to update the instance.
   {: note}

   `--no-isv-secrets`
   :   If the image does not require any isv secrets, this parameter can be set. You need not provide the isv-secrets parameter, and if you do, it will be ignored.


## Using your OCI Image to provision a Hyper Protect Virtual Server
{: #byoi_provision}

You must use the [CLI](https://cloud.ibm.com/docs/hpvs-cli-plugin) `ibmcloud hpvs instance-create` command to use your own OCI Image to provision a Hyper Protect Virtual Server, as described [here](/docs/hp-virtual-servers?topic=hp-virtual-servers-provision#provision-cli).

### Example
{: #byoi_example}

```sh
ibmcloud hpvs instance-create myServerName lite-s dal13 --rd-path "C:\MyRegistrationDefinitions\registration.json.asc" -i latest
```

## Configuring your server
{: #byoi_config}


The following topic describes how you can configure your server. Use this description only for configurations that are not considered credentials or personal data.


In this description "environment variables" are not set as environment variables, instead they provided within `/.runqenv`.
{: note}

Instance-specific configurations for applications that run in OCI containers are generally provided as environment variables when a container is created.
Either the application consumes the variables, or a script, that runs before the actual application,   configures the application.
Before you can set environment variables, the registration definition must allow them.
You can provide environment variables by way of the  `e` or `--environment` option (cannot contain “,”). The environment variables are written to `/.runqenv`.

You can allow environment variables by name to `envs_whitelist` array within the registration definition.

The following variables cannot be set: “RUNQ_ROOTDISK”, “RUNQ_RUNQENV”, “RUNQ_SYSTEMD”, “IMAGE_TAG”, “REGION”, “PHASE”, “LPAR_NAME”, “CPC”, “RUNQ_CPU”, “RUNQ_MEM”, “POD”.

Variable names can have up to 64 characters and can consist of numbers, chars, and underscore. Variable values can have up to 4096 characters and can consist of base64 character set characters (uppercase letters, lowercase letters, and +, /).

**An example with one environment variable:**
```sh
ibmcloud hpvs instance-create myServerName lite-s dal13 --rd-path “C:\MyRegistrationDefinitions\registration.json.asc ” -i latest -e name=value
```
{: screen}

**An Example with multiple environment variables:**
```sh
ibmcloud hpvs instance-create myServerName lite-s dal13 --rd-path “C:\MyRegistrationDefinitions\registration.json.asc ” -i latest -e name1=value1 -e name2=value2
```
{: screen}

The initial user has the corresponding environment variable set within the container.
