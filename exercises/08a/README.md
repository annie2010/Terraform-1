# Exercise #8a: Importing an Existing Resource

* Create an S3 bucket manually, using the AWS Console

* Create a `main.tf` file with an `aws_s3_bucket` resource and "point" it to 
your manually-created bucket

```hcl
resource "aws_s3_bucket" ... {
   ...
}
```

* Import your the bucket so that `terraform` knows about it and can manage it
  * HINT: The `terraform` docs for `aws_s3_bucket` will tell you how to do this
  * Check to be sure `terraform` knows about the bucket

* Use `terraform` to destroy your imported bucket
