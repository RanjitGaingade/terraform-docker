# Terraform Docker Nginx

This project demonstrates how to use **Terraform as an Infrastructure as Code (IaC) tool** to provision and manage a Docker container.

Terraform is used to:

- Configure the Docker provider
- Pull the Nginx Docker image
- Create and manage an Nginx container
- Configure Docker port mapping
- Inspect Terraform-managed infrastructure
- Destroy the infrastructure when it is no longer required

## Architecture

```text
                Terraform
                    |
                    v
            Docker Provider
                    |
                    v
             Docker Engine
                    |
                    v
          ┌──────────────────┐
          │  Nginx Container  │
          │ terraform-nginx   │
          │                   │
          │ Container Port 80 │
          └────────┬─────────┘
                   |
              Port Mapping
                9090:80
                   |
                   v
             localhost:9090
```

## Prerequisites

Make sure the following tools are installed:

- Terraform
- Docker
- Git

Verify the installations:

```bash
terraform version
docker --version
git --version
```

Make sure Docker is running before executing Terraform commands.

## Project Structure

```text
terraform-docker/
├── main.tf
├── .gitignore
└── README.md
```

## Terraform Configuration

The infrastructure is defined in `main.tf`.

```hcl
terraform {
  required_providers {
    docker = {
      source  = "kreuzwerker/docker"
      version = "~> 3.0"
    }
  }
}

provider "docker" {}

resource "docker_image" "nginx" {
  name         = "nginx:latest"
  keep_locally = true
}

resource "docker_container" "nginx" {
  name  = "terraform-nginx"
  image = docker_image.nginx.image_id

  ports {
    internal = 80
    external = 9090
  }
}
```

### Configuration Explanation

#### Terraform Block

```hcl
terraform {
  required_providers {
    docker = {
      source  = "kreuzwerker/docker"
      version = "~> 3.0"
    }
  }
}
```

This tells Terraform which provider is required.

- `docker` — provider name
- `kreuzwerker/docker` — Docker provider source
- `~> 3.0` — allows compatible 3.x versions

#### Docker Provider

```hcl
provider "docker" {}
```

This configures Terraform to communicate with the local Docker Engine.

#### Docker Image

```hcl
resource "docker_image" "nginx" {
  name         = "nginx:latest"
  keep_locally = true
}
```

Terraform downloads the `nginx:latest` image if it is not already available locally.

`keep_locally = true` tells Terraform to keep the image on the local Docker system when the Terraform resource is destroyed.

#### Docker Container

```hcl
resource "docker_container" "nginx" {
  name  = "terraform-nginx"
  image = docker_image.nginx.image_id
```

This creates a Docker container named:

```text
terraform-nginx
```

The container uses the image managed by:

```hcl
docker_image.nginx
```

#### Port Mapping

```hcl
ports {
  internal = 80
  external = 9090
}
```

This maps:

```text
Host Port       Container Port
9090      --->       80
```

Therefore, Nginx can be accessed at:

```text
http://localhost:9090
```

## Deployment Steps

### 1. Initialize Terraform

```bash
terraform init
```

This downloads the required Docker provider and initializes the Terraform working directory.

### 2. Format the Configuration

```bash
terraform fmt
```

This formats the Terraform configuration according to Terraform's standard formatting rules.

### 3. Validate the Configuration

```bash
terraform validate
```

This checks whether the Terraform configuration is syntactically valid and internally consistent.

### 4. Review the Execution Plan

```bash
terraform plan
```

Terraform compares the desired infrastructure with the current state and shows what it intends to create, change, or destroy.

Expected result:

```text
Plan: 2 to add, 0 to change, 0 to destroy.
```

The two resources are:

```text
docker_image.nginx
docker_container.nginx
```

### 5. Apply the Configuration

```bash
terraform apply
```

Review the plan and enter:

```text
yes
```

Terraform will create the Nginx image and container.

## Verify the Container

Check running Docker containers:

```bash
docker ps
```

You should see a container similar to:

```text
terraform-nginx
```

You can also check the container directly:

```bash
docker ps --filter name=terraform-nginx
```

## Test Nginx

Open the application in a browser:

```text
http://localhost:9090
```

Or test using curl:

```bash
curl http://localhost:9090
```

You should receive the default Nginx HTML response.

## Terraform State

List resources managed by Terraform:

```bash
terraform state list
```

Expected resources:

```text
docker_container.nginx
docker_image.nginx
```

Inspect the container:

```bash
terraform state show docker_container.nginx
```

Inspect the complete Terraform state:

```bash
terraform show
```

## Destroy the Infrastructure

When the container is no longer required:

```bash
terraform destroy
```

Review the destruction plan and enter:

```text
yes
```

Terraform will remove the resources it manages.

Because:

```hcl
keep_locally = true
```

is configured for the Docker image, the Nginx image is kept locally even after Terraform destroys the image resource.

## Terraform Workflow

The basic workflow used in this project is:

```text
Write Terraform Configuration
            |
            v
      terraform init
            |
            v
      terraform fmt
            |
            v
    terraform validate
            |
            v
      terraform plan
            |
            v
     terraform apply
            |
            v
       Verify Docker
            |
            v
    terraform destroy
```

## Useful Commands

| Command | Purpose |
|---|---|
| `terraform init` | Initialize Terraform and download providers |
| `terraform fmt` | Format Terraform configuration |
| `terraform validate` | Validate configuration |
| `terraform plan` | Preview infrastructure changes |
| `terraform apply` | Create/update infrastructure |
| `terraform state list` | List Terraform-managed resources |
| `terraform state show` | Inspect a resource in state |
| `terraform show` | Display Terraform state |
| `terraform destroy` | Destroy managed infrastructure |
| `docker ps` | List running containers |
| `curl http://localhost:9090` | Test Nginx |

## Git and GitHub

This project can be version-controlled using Git.

Initialize the repository:

```bash
git init
```

Add the project files:

```bash
git add .
```

Create the first commit:

```bash
git commit -m "Add Terraform Docker infrastructure"
```

Add the GitHub repository:

```bash
git remote add origin <your-github-repository-url>
```

Rename the branch:

```bash
git branch -M main
```

Push the project:

```bash
git push -u origin main
```

## .gitignore

Terraform generates local files that should generally not be committed to Git.

Recommended `.gitignore`:

```gitignore
.terraform/
*.tfstate
*.tfstate.*
crash.log
crash.*.log
```

The `.terraform.lock.hcl` file should normally **not** be ignored because it records provider dependency selections and helps maintain consistent provider versions.

## Important Notes

- Docker Desktop must be running before using the Docker provider.
- Port `9090` must be available on the host.
- The Docker provider communicates with the local Docker Engine.
- Terraform state should generally not be committed if it may contain sensitive information.
- Do not commit passwords, API keys, access keys, private keys, or other secrets.
- For production Terraform projects, consider using a remote and secured Terraform state backend.

## Learning Outcomes

By completing this project, the following concepts are demonstrated:

- Infrastructure as Code
- Terraform providers
- Terraform resources
- Docker image management with Terraform
- Docker container provisioning
- Port mapping
- Terraform state
- Terraform plan and apply workflow
- Infrastructure destruction using Terraform
- Git version control
- Basic GitHub project management

## Conclusion

This project demonstrates a simple Infrastructure as Code workflow using **Terraform and Docker**.

Instead of manually running:

```bash
docker run ...
```

the Docker infrastructure is defined declaratively in Terraform.

Terraform can then determine what infrastructure should exist and create, update, or destroy it based on the configuration.

