IMPORTANT ❗ ❗ ❗ Please remember to destroy all the resources after each work session. You can recreate infrastructure by creating new PR and merging it to master.
  
![img.png](doc/figures/destroy.png)

1. Authors:

   Z2

   https://github.com/mik00laj/tbd-workshop-1
   
2. Follow all steps in README.md.

3. Select your project and set budget alerts on 5%, 25%, 50%, 80% of 50$ (in cloud console -> billing -> budget & alerts -> create buget; unclick discounts and promotions&others while creating budget).
  ![img.png](doc/figures/discounts.png)
![Budget](https://github.com/user-attachments/assets/3b26cf21-3a85-4848-be58-88b6342b2d55)
![Budget2](https://github.com/user-attachments/assets/ef72cae5-ba52-473a-a1c5-1dea2bc87d67)
5. From avaialble Github Actions select and run destroy on main branch.
   ![Destroy](https://github.com/user-attachments/assets/90356f63-11b0-44c7-a10a-30c9f50d5234)

7. Create new git branch and:
    1. Modify tasks-phase1.md file.
    
    2. Create PR from this branch to **YOUR** master and merge it to make new release. 
    
![PULL REQUEST](https://github.com/user-attachments/assets/267f0371-5705-48c6-8925-d342f90f80e8)



8. Analyze terraform code. Play with terraform plan, terraform graph to investigate different modules.

    ***describe one selected module and put the output of terraform graph for this module here***
   
9. Reach YARN UI
   
   ***place the command you used for setting up the tunnel, the port and the screenshot of YARN UI here***
   
10. Draw an architecture diagram (e.g. in draw.io) that includes:
    1. VPC topology with service assignment to subnets
    2. Description of the components of service accounts
    3. List of buckets for disposal
    4. Description of network communication (ports, why it is necessary to specify the host for the driver) of Apache Spark running from Vertex AI Workbech
  
    ***place your diagram here***

11. Create a new PR and add costs by entering the expected consumption into Infracost
For all the resources of type: `google_artifact_registry`, `google_storage_bucket`, `google_service_networking_connection`
create a sample usage profiles and add it to the Infracost task in CI/CD pipeline. Usage file [example](https://github.com/infracost/infracost/blob/master/infracost-usage-example.yml) 

   ***place the expected consumption you entered here***

   ***place the screenshot from infracost output here***

11. Create a BigQuery dataset and an external table using SQL
    
    ***place the code and output here***
   
    ***why does ORC not require a table schema?***

  
12. Start an interactive session from Vertex AI workbench:

    ***place the screenshot of notebook here***
   
13. Find and correct the error in spark-job.py

    ***describe the cause and how to find the error***

14. Additional tasks using Terraform:

    1. Add support for arbitrary machine types and worker nodes for a Dataproc cluster and JupyterLab instance

    Arbitraty machine types for a Dataproc cluster are already supported via the `machine_type` variable, so it was skipped.

    ['modules/dataproc/variables.tf'](modules/dataproc/variables.tf)

    ```
    variable "worker_count" {
      type        = number
      default     = 2
      description = "Number of worker nodes"
    }
    ```

    ['modules/dataproc/main.tf'](modules/dataproc/main.tf)

    ```
    cluster_config {
      ...
      worker_config {
        num_instances = var.worker_count
        ...
      }
    }
    ```

    ['modules/vertex-ai-workbench/variables.tf'](modules/vertex-ai-workbench/variables.tf)

    ```
    variable "machine_type" {
      type        = string
      default     = "e2-standard-2"
      description = "Machine type to use for the notebook instance"
    }
    ```

    ['modules/vertex-ai-workbench/main.tf'](modules/vertex-ai-workbench/main.tf)

    ```
    resource "google_notebooks_instance" "tbd_notebook" {
      ...
      machine_type = var.machine_type
      ...
    }
    ```
    
    2. Add support for preemptible/spot instances in a Dataproc cluster

    ['modules/dataproc/variables.tf'](modules/dataproc/variables.tf)

    ```
    variable "preeemptible_worker_count" {
      type        = number
      default     = 0
      description = "Number of preemptible worker nodes"
    }
    ```

    ['modules/dataproc/main.tf'](modules/dataproc/main.tf)

    ```
    preemptible_worker_config {
      num_instances = var.preeemptible_worker_count

      disk_config {
        boot_disk_type    = "pd-standard"
        boot_disk_size_gb = 100
      }

      preemptibility = "SPOT"
    }
    ```
    
    3. Perform additional hardening of Jupyterlab environment, i.e. disable sudo access and enable secure boot

    ['modules/vertex-ai-workbench/main.tf'](modules/vertex-ai-workbench/main.tf)

    ```
    metadata = {
      vmDnsSetting : "GlobalDefault"
      notebook-disable-root: true
    }
    ```

    ```
    shielded_instance_config {
      enable_secure_boot          = true
      enable_vtpm                 = true
      enable_integrity_monitoring = true
    }
    ```

    4. (Optional) Get access to Apache Spark WebUI

    ['modules/dataproc/main.tf'](modules/dataproc/main.tf)

    ```
    cluster_config {
      endpoint_config {
        enable_http_port_access = true
      }
      ...
    }
    ```
