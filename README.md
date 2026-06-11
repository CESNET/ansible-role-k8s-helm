# Ansible role cesnet.k8s_helm
Ansible role for generating a temporary Helm chart for Kubernetes to deploy a set of objects representing an application 
as a single unit, that can be upgraded or deleted as a unit.

The principle is that this Ansible role generates a Helm chart in a temporary directory and deploys it using Helm.
The advantage of this approach when compared to creating independent Kubernetes objects is that Helm manages
all objects created as a Helm release as a unit that can be upgraded, rolled back or deleted in one step.

## Prerequisites

 * Ansible (recent version with Kubernetes and Helm collections)
 * [Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/) for Kubernetes tasks in Ansible
 * Kubectl's config in `~/.kube/config`
 * [Helm](https://helm.sh/docs/intro/install/#from-apt-debianubuntu)
 * [Helm Diff plugin](https://github.com/databus23/helm-diff) for idempotency of Helm tasks in Ansible

## Default settings

The role default settings are prepared for an application that is packaged in a container image created by Spring Boot,
but can be used also for other applications.

The default settings create the following objects:

 * secret.yaml - a Kubernetes Secret containing a set of files specified by variables `app_secret_binary_files`, `app_secret_text_files`, `app_secret_templated_files` and `app_secret_directory`
 * deployment.yaml - a Kubernetes Deployment with a single Pod with a single container with a mounted directory containing the files from the secret
 * service.yaml -  a Kubernetes Service that maps its TCP port 80 to port 8080 of the containers from the deployment
 * ingress.yaml - a Kubernetes Ingress that maintains a TLS certificate and forwards HTTP requests to the service 
 
 ## Variables

 * k8s_context - Kubernetes context matching definition in `~/.kube/config`
 * k8s_namespace - Kubernetes namespace matching definition in `~/.kube/config`
 * app_name - application name, used in many places as prefix
 * app_image_name - Docker image name without version
 * app_version - application version, used for container image version and in Helm chart application version
 * app_replicas_num - number of container replicas, default is 1
 * app_requests_cpu, app_requests_mem, app_limits_cpu, app_limits_mem - limits for container
 * app_public_dns_name - DNS name for the application, must be mapped to the Ingress IP
 * app_chart_version - version used for Chart.yaml file
 * app_helm_templates - list of Helm Chart templates to be created using Ansible templates, i.e. the Ansible template (Jinja2 syntax) deployment.yaml.j2 would generate the Helm template (Go syntax) deployment.yaml   
 * app_clean_up_helm_templates - list of Helm Chart templates to be deleted from the temporary local directory after role execution to protect sensitive data
 * app_secret_binary_files - list of files that are read as binary data, base64-encoded and added to the secret template in secret.yaml
 * app_secret_text_files - list of files similar to app_secret_binary_files
 * app_secret_templated_files - list of Ansible template names that are used to generate files added to the secret
 * app_secret_directory - name of directory that is recursively added to the secret data (paths to subdirectories are handled in the deployment.yaml by secret items mapping)
 * app_container_env - shell environment variables to be set for the container
 * app_secret_mount_dir - directory path where the secret data will be mounted as files, default is /workspace/config because this path is searched by Spring Boot apps for application properties files
 * app_container_probes - YAML to be added to spec.template.spec.containers[0] for setting probes, default is for Spring Boot Actuator

The difference between `app_secret_binary_files` and `app_secret_text_files` is that `app_secret_binary_files` are read using
the [ansible.builtin.unvault](https://github.com/ansible/ansible/blob/devel/lib/ansible/plugins/lookup/unvault.py) lookup
and `app_secret_text_files` are read using the [ansible.builtin.file](https://github.com/ansible/ansible/blob/devel/lib/ansible/plugins/lookup/file.py) lookup
with disabled stripping of white spaces. They behave identically as of Ansible 14, but may differ in the future.
