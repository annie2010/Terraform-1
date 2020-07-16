# Exercise #3: Plans and Applies

* now we are actually going to create some infrastructure
* for this exercise, we are going to:

1. initialize our project directory (i.e., this exercise directory)
1. run a `terraform plan` to understand why planning makes sense, and should always be a part of your terraform flow
1. actually apply our infrastructure, in this case a single object within our s3 bucket
1. destroy what we created

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

### Initialization

* first, we need to run init since we're starting in a new exercise, or project directory:

```bash
terraform init
```

### Plan

* next we run a plan
  * a dry run that helps us understand what terraform intends to change when it does an apply

* recall from the previous exercise that we'll need to make sure our `student_alias` value gets passed in appropriately
  * pick whichever method of doing so, and then run your plan:

```bash
terraform plan
```

* your output should look something like this:

```
Refreshing Terraform state in-memory prior to plan...
The refreshed state will be used to calculate this plan, but will not be
persisted to local or remote state storage.


------------------------------------------------------------------------

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_s3_bucket_object.user_student_alias_object will be created
  + resource "aws_s3_bucket_object" "user_student_alias_object" {
      + acl                    = "private"
      + bucket                 = "devint-..."
      + content                = "This bucket is reserved for ..."
      + content_type           = (known after apply)
      + etag                   = (known after apply)
      + id                     = (known after apply)
      + key                    = "student.alias"
      + server_side_encryption = (known after apply)
      + storage_class          = (known after apply)
      + version_id             = (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.

------------------------------------------------------------------------

Note: You didn't specify an "-out" parameter to save this plan, so Terraform
can't guarantee that exactly these actions will be performed if
"terraform apply" is subsequently run.
```

* from the above output, we can see that terraform will create a single S3 object in our bucket
* important line to note is the one beginning with "Plan:"
  * `Plan: 1 to add, 0 to change, 0 to destroy.`

* terraform is designed to detect when there is configuration drift in resources that it created and then intelligently 
determine how to correct the difference
  * we'll cover this in more detail later

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

### Apply

* let's go ahead and have terraform create the S3 bucket object
  * maybe try a different method of passing in your `student_alias` variable when running apply:

```bash
terraform apply
```

* terraform will execute another plan, then ask you if you would like to apply the changes
 * type "yes" to approve, then let it do its magic...
 
* your output should look like the following:

```
An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_s3_bucket_object.user_student_alias_object will be created
  + resource "aws_s3_bucket_object" "user_student_alias_object" {
      + acl                    = "private"
      + bucket                 = "devint-..."
      + content                = "This bucket is reserved for ..."
      + content_type           = (known after apply)
      + etag                   = (known after apply)
      + id                     = (known after apply)
      + key                    = "student.alias"
      + server_side_encryption = (known after apply)
      + storage_class          = (known after apply)
      + version_id             = (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_s3_bucket_object.user_student_alias_object: Creating...
aws_s3_bucket_object.user_student_alias_object: Creation complete after 1s [id=student.alias]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

* now let's run a plan again:

```bash
terraform plan
```

You should notice a couple differences:

* terraform informs you that it is Refreshing the State
   * after the first apply, subsequent plans/applies
     * will check the infrastructure it created and
     * update the terraform state with any new information about the resource
     
* you'll notice that Terraform informed you that there are no changes to be made...why?

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

### Handling Changes

* let's try making a change to the s3 bucket object and allow terraform to correct it
  * let's change the content of our object
  * find `main.tf` and modify the s3 bucket stanza to reflect the following:

```hcl
# declare a resource stanza so we can create something.
resource "aws_s3_bucket_object" "user_student_alias_object" {
  bucket  = "devint-${var.student_alias}"
  key     = "student.alias"
  content = "This bucket is reserved for ${var.student_alias} ****ONLY****"
}
```

* run another apply:

```bash
terraform apply
```

* the important output for the plan portion of the apply that you should note:

```
Terraform will perform the following actions:

  # aws_s3_bucket_object.user_student_alias_object will be updated in-place
  ~ resource "aws_s3_bucket_object" "user_student_alias_object" {
        acl           = "private"
        bucket        = "devint-..."
      ~ content       = "This bucket is reserved for ..." -> "This bucket is reserved for ... ****ONLY****"
        content_type  = "binary/octet-stream"
        etag          = "94e32327b8007fa215f3a9edbda7f68c"
        id            = "student.alias"
        key           = "student.alias"
        storage_class = "STANDARD"
        tags          = {}
    }

Plan: 0 to add, 1 to change, 0 to destroy.
```

* a terraform plan informs you with a few symbols to tell you what will happen
  * `+` means that terraform plans to add this resource
  * `-` means that terraform plans to remove this resource
  * `-/+` means that terraform plans to destroy then recreate the resource
  * `+/-` is similar to the above, but in certain cases a new resource needs to be created before destroying the previous one, we'll cover how you instruct terraform to do this a bit later
  * `~` means that terraform plans to modify this resource in place (doesn't require destroy then re-create)
  * `<=` means that terraform will read the resource

* our above plan will modify our s3 object in place per our requested update to the file

* some resources or some changes require that a resource be recreated to facilitate that change, and those cases are usually expected
  * one example of this would be an AWS launch configuration
    * in AWS, launch configurations cannot be changed, only copied and modified once during the creation of the copy
    * terraform generally knows about these caveats and handles those changes gracefully
    * ...including updating dependent resources to link to the newly created resource
    * this greatly simplifies complex or frequent changes to any size infrastructure and reduces the possibility of human error

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

### Destroy

* when infrastructure is retired, terraform can destroy it gracefully, ensuring that all resources
are removed and in the order that their dependencies require
  * let's destroy our s3 bucket object:

```bash
terraform destroy
```

* you should see the following:

```
aws_s3_bucket_object.user_student_alias_object: Refreshing state... [id=student.alias]

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # aws_s3_bucket_object.user_student_alias_object will be destroyed
  - resource "aws_s3_bucket_object" "user_student_alias_object" {
      - acl           = "private" -> null
      - bucket        = "devint-di-..." -> null
      - content       = "This bucket is reserved for ... ****ONLY****" -> null
      - content_type  = "binary/octet-stream" -> null
      - etag          = "c7e49348083281f9dd997923fe6084b7" -> null
      - id            = "student.alias" -> null
      - key           = "student.alias" -> null
      - storage_class = "STANDARD" -> null
      - tags          = {} -> null
    }

Plan: 0 to add, 0 to change, 1 to destroy.

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

aws_s3_bucket_object.user_student_alias_object: Destroying... [id=student.alias]
aws_s3_bucket_object.user_student_alias_object: Destruction complete after 0s

Destroy complete! Resources: 1 destroyed.
```

* you'll notice that the destroy process is very similar to apply, just the other way around
  * ...and it also requires confirmation, which is a good thing
