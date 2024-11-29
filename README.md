To solve this problem, we can use Terraform's **`local_file`** resource to create a Dockerfile and use the **`docker_image`** and **`docker_container`** resources from the Terraform Docker provider to build and run the container.

Here's a complete Terraform script for the task:

### 1. Create a `main.tf` file:

```hcl
# Define the Terraform provider
terraform {
  required_providers {
    docker = {
      source  = "kreuzwerker/docker"
      version = "~> 3.0"
    }
  }
}

provider "docker" {}

# Step 1: Copy the index.html file to the desired location
resource "local_file" "dockerfile" {
  filename = "${path.module}/Dockerfile"
  content = <<EOT
# Use the nginx:latest image as the base
FROM nginx:latest

# Copy index.html to the nginx html directory
COPY index.html /usr/share/nginx/html/
EOT
}

# Step 2: Copy the index.html file to the Docker build directory
resource "local_file" "index_file" {
  filename = "${path.module}/index.html"
  content  = file("${path.module}/../../wingst9-mock-challenge/index.html")
}

# Step 3: Build the Docker image
resource "docker_image" "webserver" {
  name         = "webserver"
  build {
    context    = "${path.module}"
    dockerfile = "${path.module}/Dockerfile"
  }
}

# Step 4: Run the Docker container
resource "docker_container" "webcontainer" {
  name  = "webcontainer"
  image = docker_image.webserver.latest
  ports {
    internal = 80
    external = 5800
  }
}
```

---

### 2. Steps to Execute:

1. **Install Terraform**: Ensure Terraform is installed on your system.
2. **Install Docker**: Ensure Docker is installed and running.
3. **Initialize Terraform**:
   Run the following command in the terminal to initialize the Terraform workspace:
   ```bash
   terraform init
   ```

4. **Apply Terraform Configuration**:
   Run the following command to apply the configuration and deploy the container:
   ```bash
   terraform apply
   ```

5. **Access the Deployed Nginx**:
   Open a browser and navigate to:
   ```
   http://localhost:5800/
   ```

---

### Explanation:

1. **`local_file` resource**: Creates the `Dockerfile` and copies `index.html` to the required location.
2. **`docker_image` resource**: Builds a Docker image named `webserver` using the generated `Dockerfile`.
3. **`docker_container` resource**: Runs a container named `webcontainer` using the built `webserver` image, exposing it on port `5800`.

This configuration ensures that all steps are automated using Terraform, aligning with the requirements.
