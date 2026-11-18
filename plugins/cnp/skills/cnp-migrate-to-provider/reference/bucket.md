# To an S3 bucket

The platform's bucket speaks the S3 API. Read the connection from
environment variables only: `S3_ENDPOINT`, `S3_BUCKET`, `S3_REGION`,
`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`. Use path-style addressing
(the endpoint is not a virtual-host S3), for example `s3={"addressing_style":
"path"}` in boto3 or `forcePathStyle: true` in the AWS SDK for JavaScript.

## From MinIO

MinIO already speaks S3: usually only the endpoint, bucket and keys change.
Remove the MinIO service and its volume from `docker-compose` once the code
reads the variables above; keep a local MinIO only for tests.

## From local files

- Replace each write to disk with an upload (`put_object`), each read with
  a download or a streamed `get_object`, each delete with `delete_object`.
- Keys: keep the relative path the app used (`avatars/<user>.png`), never an
  absolute path of the container.
- Files served to browsers: return a short-lived presigned URL rather than
  proxying the bytes through the app.
- Remove the volume that held the files from the deployment once nothing
  writes to it.

## Check before the pull request

- Tests run against a local MinIO with the same variables.
- No key, secret or endpoint value in the code or the repository.
- Large files are streamed, not read whole into memory.
