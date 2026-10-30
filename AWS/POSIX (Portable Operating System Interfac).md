A set of standards defining how operating systems such as Linux and Unix handle things like files, permissions, processes, and users.

### How this relates to EFS Access Points
Imagine two applications share an EFS file system:
- App A needs access to /app-a.
- App B needs access to /app-b.
You can configure separate EFS Access Points to provide different root directories and POSIX identities.

When an application accesses EFS through an access point, EFS can enforce the configured POSIX identity for file operations. The access point can also expose a specified directory as the application's root.
Exam takeaway
- POSIX permissions control who can read, write, or execute files.
- EFS Access Points can enforce a user/group identity and a restricted root directory.
- IAM permissions control AWS-level access to EFS APIs and access points; POSIX permissions govern file-system operations.