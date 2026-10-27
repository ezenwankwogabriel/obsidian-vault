This is a fully managed integration service that enables you to securely transfer data btw Software-as-a-Service (SaaS) applications and AWS.

For AWS SAA-C03, remember: AppFlow = move data between SaaS applications and AWS services.

Example
Suppose a company stores customer records in Salesforce and wants to analyse them in Amazon S3.

With AppFlow, you can configure a flow to:
1. Connect Salesforce as the source.
2. Select the customer fields to transfer.
3. Apply filters if needed.
4. Send the records to an S3 bucket.
5. Run the flow on a schedule or when supported events trigger it.
You don't need to build and maintain a custom integration service just to move the data.

## AppFlow vs. related AWS services

| Service                              | Main purpose                                                   | Exam clue                            |
| ------------------------------------ | -------------------------------------------------------------- | ------------------------------------ |
| Amazon AppFlow                       | Transfer data between SaaS apps and AWS                        | Salesforce → S3                      |
| AWS Glue                             | Discover, prepare, transform, and integrate data for analytics | ETL and data catalog                 |
| Amazon EventBridge                   | Route events to targets                                        | React to an event                    |
| Amazon DataSync                      | Move files and object data between storage systems             | NFS/SMB or on-premises storage → AWS |
| AWS Database Migration Service (DMS) | Migrate databases and replicate database changes               | Database → database                  |