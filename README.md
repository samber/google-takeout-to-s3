# Google Takeout to AWS S3

🚨 Simple script to upload encrypted Google Takeout archives to S3.

Old backups are removed automatically 30 days after new backups.

## Setup

### Calendar reminder

Install one of the following reminder in your Calendar:

- monthly: go to webcal://bit.ly/monthly-reminder
- quarterly: go to webcall://bit.ly/quarterly-reminder

### Script

Add this to your .bashrc:

```
export GOOGLE_TAKEOUT_S3_BUCKET=my-google-takeout-backups

function backup_google_takeout() {
  if [ $# -eq 0 ]
  then
    echo "usage: backup_google_takeout google_takeout_archive.tar.gz" >&2
    return
  fi

  file=$1
  s3_bucket_name=${GOOGLE_TAKEOUT_S3_BUCKET}
  s3_file_name=google-takeout-archive.tar.gz

  echo '1- Encrypt'
  openssl aes-256-cbc -a -salt -in ${file} -out /tmp/${s3_file_name} || return
  echo 'ok\n'

  echo '2- Upload'
  aws s3 cp /tmp/${s3_file_name} s3://${s3_bucket_name}/${s3_file_name} --storage-class STANDARD_IA || return
  echo 'ok\n'

  read "response?3- Would you like to remove ${file} ? (y/n) "
  if [[ ${response} =~ ^[Yy]$ ]]
  then
    rm -f ${file}
    echo 'Removed'
  fi
  echo

  rm -f /tmp/${s3_file_name}

  echo "Recovery commands:"
  echo "$ aws s3 cp s3://${s3_bucket_name}/${s3_file_name} /tmp/${s3_file_name}"
  echo "$ openssl aes-256-cbc -a -d -in /tmp/${s3_file_name} -out /tmp/${s3_file_name}.new"
  echo "\nBye"
}
```

### S3 Bucket

1. Create an AWS S3 bucket
2. Enable versioning
3. Enable bucket encryption
4. Block all public access (!)
5. Go to "Management" tab
6. Create a lifecycle rule for data retention
  1. Enter a name
  2. Click `Next` twice
  3. Configure expiration for `Previous versions`
  4. Enter `30` days in the form
  5. Click `Next`
  6. Click `Save`

### Configure AWS cli

```
pip3 install awscli
aws configure
```

## Run

Few times a year, just go to [Google Takeout](https://takeout.google.com/settings/takeout), generate a archive and execute:

```
backup_google_takeout <google_takeout_archive.tar.gz>
```

Enter a password twice (for encryption).

## Costs

This script use `STANDARD Infrequent Access`, costing half the price of S3 `STANDARD` storage: $0.0131 in Paris.

`20 GB per backup x 2 versions x $0.0131 = $0.52`

Retrieving a 20 BG backup from S3 will cost you (from Paris):

`20 GB x ($0.09 + $0.01) = $2`

That's cheap !! 😍
  
## Todo

- Find a way to get archives automatically from a Google API.

## Contributing

Contributions are greatly welcomed ;)
