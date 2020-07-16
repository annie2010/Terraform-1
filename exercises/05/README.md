# Exercise #5: Interacting with Providers

* providers are plugins that terraform uses to understand various external APIs and cloud providers
* thus far, we've used the AWS provider
  * in this exercise, we're going to modify the AWS provider we've been using to create our bucket in a different region

### Add the second provider

* add this variable stanza to the `variables.tf` file:

```hcl
variable "region_alt" {
  default = "us-west-2"
}
```

* then, add this provider block with the new region to `main.tf` just under the existing provider block
  * note the `alias` argument–this is necessary when you have duplicate providers:

```hcl
provider "aws" {
  version = "~> 2.0"
  region = "${var.region_alt}"
  alias = "alternate"
}
```

* we will also need to specify the alternate provider when creating your bucket:

```hcl
  provider = aws.alternate
```

* now let's provision and bring up another s3 bucket in this other region:

```bash
terraform init
terraform apply
terraform show
```
* the above should show that you have a bucket now named `devint-[your student alias]-alt` that was created in the
us-west-2 region

*NOTE:* that at the beginning of our course we set the `AWS_DEFAULT_REGION` environment variable in your Cloud9 environment.
Along with this variable and the access key and secret key, terraform is able to use these environment variables for the AWS
provider as defaults unless you override them in the HCL provider stanza.

* we'll be looking more at using providers in other exercises as we move along

### Finishing this exercise

* run the following to finish:

```
terraform destroy
```
