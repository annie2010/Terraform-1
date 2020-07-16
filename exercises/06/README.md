# Exercise #6: Modules

* terraform is *ALL* about modules
* every terraform working directory is really just a module that could be reused by others
* this is one of the key capabilities of terraform

* in this exercise, we are going to modularize the code that we have been playing with during this whole workshop, but instead of
constantly redeclaring everything, we're going to reference the module that we've created and see if it works

* first, create a main.tf file in the main directory for the this exercise
   * inside the `main.tf` file you created, add the following:

```hcl
provider "aws" {
  version = "~> 2.0"
}

module "s3_bucket_01" {
  source        = "./modules/s3_bucket/"
  region        = "us-east-2"
  student_alias = var.student_alias
}

# We're not defining region in this module call, so it will use the default as defined in the module
# What happens when you remove the default from the module and don't pass here? Feel free to try it out.
module "s3_bucket_02" {
  source        = "./modules/s3_bucket/"
  student_alias = var.student_alias
}
```

* next, create a `variables.tf` file so we can capture `student_alias` to pass it through to our module:

```hcl
variable "student_alias" {
  description = "Your student alias"
}
```

* what we've done is create a `main.tf` config file that references a module stored in a
local directory, twice
  * this allows us to encapsulate any complexity contained by the module's codewhile still allowing us to pass variables into the module
 
* after doing this, you can then begin the init and apply process:

```bash
terraform init
terraform plan
terraform apply
```

* you'll notice that terraform manages each resource as if there is no module division
  * i.e., the resources are bucketed into one big change list
* ...but under the hood terraform's dependency graph will show some separation
* it's very difficult, for example, to create dependencies between two resources that are in different modules
* you can, however, use interpolation to create a variable dependency between two modules at the root level, ensuring one is created before the other

* specific applications where direct resource dependency is required really necessitates grouping those resources into a single module or project

### Finishing this exercise

* as usual...

```
terraform destroy
```
