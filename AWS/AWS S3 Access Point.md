This is a named endpoint that provides a specific way to access objects in an S3 bucket, with its own access policy.

It solves a common problem: multiple apps or teams need access to same bucket, but each should have different permissions

How it works:
Picture a healthcare company stores all its data in one s3 bucket `healthcare-data`
Three teams need access to
- read patient dataset
- read and write processed files
- read compliance reports

Instead of managing every teams access through one complicated bucket policy, create separate access points

Each access point has its own policy, you can configure access-point network controls, such as restricting access to a **VPC**

## S3 Access Points vs. EFS Access Points

This is a useful exam distinction.

| Feature                  | S3 Access Point                                                    | EFS Access Point                                                                  |
| ------------------------ | ------------------------------------------------------------------ | --------------------------------------------------------------------------------- |
| Storage type             | Object storage                                                     | Shared file storage                                                               |
| Purpose                  | Simplifies and controls access to a bucket                         | Provides an application-specific entry point into a file system                   |
| Permissions and controls | Access-point policy and network restrictions                       | Can enforce a POSIX user/group identity and root directory                        |
| Typical use case         | Multiple applications access one bucket with different permissions | Multiple applications share EFS with different directory or identity restrictions |

- One bucket, many applications, separate access policies → S3 Access Points.
- Restrict S3 access to a VPC through an access point → Configure the access point's network origin as VPC.
- Give an application a restricted directory and POSIX identity in EFS → EFS Access Point.
- Need separate storage or isolation of data itself? An access point alone doesn't provide that; it only controls an access path.