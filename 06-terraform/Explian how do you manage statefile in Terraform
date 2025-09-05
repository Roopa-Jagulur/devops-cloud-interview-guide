Terraform Statefile Management:
Statefile is brain of Terraform, contians information about:
- resources 
- resource dependencies 
- last (which is current state of running resources on the cloud) state of your infrastructure

Terraform always refers to statefile whenever we try to modify existing resources and update state of the statefile as modified, 
accordingly.

- Need terraform statefile in centralised location for anyone who wants to update existing resources on cloud 
  which are already created using terraform code.
- Statefile stored in centralised location i.e remote backend (S3, blob, gcs-these are also capable of locking the statefile) support by terraform because of statefile's sensitive information. 
- When someone want to modify statefile then the statefile is locked on thier name and released lock after statefile usage. To avoid concurrent usage of statefile / to avoid confusing teraform which request to process.

in our organisation: (for interview)
