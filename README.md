# FS3 Client Quickstart Guide
[![Slack](https://slack.min.io/slack?type=svg)](https://slack.min.io) [![Go Report Card](https://goreportcard.com/badge/minio/mc)](https://goreportcard.com/report/minio/mc) [![Docker Pulls](https://img.shields.io/docker/pulls/minio/mc.svg?maxAge=604800)](https://hub.docker.com/r/minio/mc/) [![license](https://img.shields.io/badge/license-AGPL%20V3-blue)](https://github.com/minio/mc/blob/master/LICENSE)

FS3 Client (mc) provides a modern alternative to UNIX commands like ls, cat, cp, mirror, diff, find etc. It supports filesystems and Amazon S3 compatible cloud storage service (AWS Signature v2 and v4).

```
alias       set, remove and list aliases in configuration file
ls          list buckets and objects
mb          make a bucket
rb          remove a bucket
cp          copy objects
mirror      synchronize object(s) to a remote site
cat         display object contents
head        display first 'n' lines of an object
pipe        stream STDIN to an object
share       generate URL for temporary access to an object
find        search for objects
sql         run sql queries on objects
stat        show object metadata
mv          move objects
tree        list buckets and objects in a tree format
du          summarize disk usage recursively
retention   set retention for object(s)
legalhold   set legal hold for object(s)
diff        list differences in object name, size, and date between two buckets
rm          remove objects
encrypt    manage bucket encryption config
event       manage object notifications
watch       listen for object notification events
undo        undo PUT/DELETE operations
policy      manage anonymous access to buckets and objects
tag         manage tags for bucket(s) and object(s)
ilm         manage bucket lifecycle
version     manage bucket versioning
replicate   configure server side bucket replication
admin       manage MinIO servers
update      update mc to latest release
list        list swan info
car         generate car file for filecoin offline deal
send        send filecoin deal
```

## Install FS3-mc
### Option 1. Docker Container
#### Stable
```
docker pull minio/mc
docker run minio/mc ls play
```

#### Edge
```
docker pull minio/mc:edge
docker run minio/mc:edge ls play
```

**Note:** Above examples run `mc` against MinIO [_play_ environment](#test-your-setup) by default. To run `mc` against other S3 compatible servers, start the container this way:

```
docker run -it --entrypoint=/bin/sh minio/mc
```

then use the [`mc config` command](#add-a-cloud-storage-service).

#### GitLab CI
When using the Docker container in GitLab CI, you must [set the entrypoint to an empty string](https://docs.gitlab.com/ee/ci/docker/using_docker_images.html#overriding-the-entrypoint-of-an-image).

```
deploy:
  image:
    name: minio/mc
    entrypoint: ['']
  stage: deploy
  before_script:
    - mc alias set minio $MINIO_HOST $MINIO_ACCESS_KEY $MINIO_SECRET_KEY
  script:
    - mc cp <source> <destination>
```

### Option 2. GNU/Linux
#### Binary Download
| Platform | Architecture | URL |
| ---------- | -------- |------|
|GNU/Linux|64-bit Intel|https://dl.min.io/client/mc/release/linux-amd64/mc |
||64-bit PPC|https://dl.min.io/client/mc/release/linux-ppc64le/mc |

```
wget https://dl.min.io/client/mc/release/linux-amd64/mc
chmod +x mc
./mc --help
```


### Option 3. Install from Source
Source installation is only intended for developers and advanced users. If you do not have a working Golang environment, please follow [How to install Golang](https://golang.org/doc/install). Minimum version required is [go1.13](https://golang.org/dl/#stable)

```sh
GO111MODULE=on go get github.com/filswan/fs3
```

## Add a Cloud Storage Service
If you are planning to use `mc` only on POSIX compatible filesystems, you may skip this step and proceed to [everyday use](#everyday-use).

To add one or more Amazon S3 compatible hosts, please follow the instructions below. `mc` stores all its configuration information in ``~/.mc/config.json`` file.

```
mc alias set <ALIAS> <YOUR-S3-ENDPOINT> <YOUR-ACCESS-KEY> <YOUR-SECRET-KEY> --api <API-SIGNATURE> --path <BUCKET-LOOKUP-TYPE>
```

<*ALIAS*> is simply a short name to your cloud storage service. S3 end-point, access and secret keys are supplied by your cloud storage provider. API signature is an optional argument. By default, it is set to "S3v4".

Path is an optional argument. It is used to indicate whether dns or path style url requests are supported by the server. It accepts "on", "off" as valid values to enable/disable path style requests.. By default, it is set to "auto" and SDK automatically determines the type of url lookup to use.

### Example - MinIO Cloud Storage
MinIO server displays URL, access and secret keys.

```
mc alias set minio http://192.168.1.51 BKIKJAA5BMMU2RHO6IBB V7f1CwQqAcwo80UEIJEjc5gVQUSSx5ohQ9GSrr12
```

### Example - Amazon S3 Cloud Storage
Get your AccessKeyID and SecretAccessKey by following [AWS Credentials Guide](http://docs.aws.amazon.com/general/latest/gr/aws-security-credentials.html).

```
mc alias set s3 https://s3.amazonaws.com BKIKJAA5BMMU2RHO6IBB V7f1CwQqAcwo80UEIJEjc5gVQUSSx5ohQ9GSrr12
```

**Note**: As an IAM user on Amazon S3 you need to make sure the user has full access to the buckets or set the following restricted policy for your IAM user

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowBucketStat",
            "Effect": "Allow",
            "Action": [
                "s3:HeadBucket"
            ],
            "Resource": "*"
        },
        {
            "Sid": "AllowThisBucketOnly",
            "Effect": "Allow",
            "Action": "s3:*",
            "Resource": [
                "arn:aws:s3:::<your-restricted-bucket>/*",
                "arn:aws:s3:::<your-restricted-bucket>"
            ]
        }
    ]
}
```

### Example - Google Cloud Storage
Get your AccessKeyID and SecretAccessKey by following [Google Credentials Guide](https://cloud.google.com/storage/docs/migrating?hl=en#keys)

```
mc alias set gcs  https://storage.googleapis.com BKIKJAA5BMMU2RHO6IBB V8f1CwQqAcwo80UEIJEjc5gVQUSSx5ohQ9GSrr12
```

## Run FS3-mc
### Step 1. Test Your Setup
`mc` is pre-configured with https://play.min.io, aliased as "play". It is a hosted MinIO server for testing and development purpose.  To test Amazon S3, simply replace "play" with "s3" or the alias you used at the time of setup.

**Example:**

List all buckets from https://play.min.io

```
mc ls play
[2016-03-22 19:47:48 PDT]     0B my-bucketname/
[2016-03-22 22:01:07 PDT]     0B mytestbucket/
[2016-03-22 20:04:39 PDT]     0B mybucketname/
[2016-01-28 17:23:11 PST]     0B newbucket/
[2016-03-20 09:08:36 PDT]     0B s3git-test/
```

### Stap 2. Make a bucket:
`mb` command is used to create a new bucket.

**Example:**
```
mc mb play/mybucket
Bucket created successfully `play/mybucket`.
```

### Step 3. Copy Objects:
`cp` command is used to copy data from one or more sources to a target.

**Example:**
```
mc cp myobject.txt play/mybucket
myobject.txt:    14 B / 14 B  ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓  100.00 % 41 B/s 0
```

### Step 4. Send a deal
you may send an offline deal to a miner
#### Prepare your environment
 - A running lotus node at local
 - A filecoin wallet with sufficient balance to send deal, set as environment variable $FIL_WALLET
 - MinIO credentials set as environment variables $ENDPOINT, $ACCESS_KEY, $SECRET_KEY

#### Step 4.1. Generate CAR file
`car generate` command is used to generate a car file
```
--car-dir: folder for splitted pieces which are in form of .car
--slice-size: size of each pieces. Be careful, a lot of cpu, memory and time would be consumed if slice size is very large.
--parallel: number goroutines run when building ipld nodes
--graph-name: graph-name will be used as a prefix of each pieces
--calc-commp: calculation of pieceCID, default value is false.
--parent-path: usually just be the same as /path/to/dataset, it's just a method to figure out relative path when building IPLD graph
```

**Example:**
  
```
mc car generate --car-dir=path/to/car-dir --slice-size=17179869184 --parallel=2 --graph-name=gs-test --calc-commp=true --parent-path=/path/to/dataset
```

#### Step 4.2. Upload CAR file
`cp` command is used to upload the CAR file to FS3, then you can share it to your storage provider.

**Example:**
  
```mc cp /path/to/car_file play/mybucket```

#### Step 4.3. Send deal
`send` command is used to send an offline deal to a designated storage provicer, a fully synchronized lotus node at local is required.

```
--piece-cid
--piece-size
--data-cid
--from: specify filecoin wallet to use, default: $FIL_WALLET
--price: specify the deal price for each GiB of file, default: 0
--start: specify days for storage provider to process the file, default: 7
--duration: specify length in day to store the file, default: 365
--upload: specify whether upload the generated csv to FS3 or not, default: false
          In order to connect to your FS3 instance, you need to set environment variables of ACCESS_KEY, SECRET_KEY and ENDPOINT
--minio-bucket: specify the bucket name used in FS3, if '--upload is set to true', default: swan
```
**Example:**
```
mc send --piece-cid baga6ea4seaqdmps47pxpgpclxo4xgqtkoxylwasf4mm524wldmguqgu45rce2pq --piece-size 2130706432 --data-cid QmcRx2dFaScfu61Vp13gZPk87zT4BL5PdG5n7Pnr93oPRc
```

<a name="everyday-use"></a>
## Everyday Use

### Shell aliases
You may add shell aliases to override your common Unix tools.

```
alias ls='mc ls'
alias cp='mc cp'
alias cat='mc cat'
alias mkdir='mc mb'
alias pipe='mc pipe'
alias find='mc find'
```

### Shell autocompletion
In case you are using bash, zsh or fish. Shell completion is embedded by default in `mc`, to install auto-completion use `mc --autocompletion`. Restart the shell, mc will auto-complete commands as shown below.

```
mc <TAB>
admin    config   diff     find     ls       mirror   policy   session  sql      update   watch
cat      cp       event    head     mb       pipe     rm       share    stat     version
```

## Explore Further
- [MinIO Client Complete Guide](https://docs.min.io/docs/minio-client-complete-guide)
- [MinIO Quickstart Guide](https://docs.min.io/docs/minio-quickstart-guide)
- [The MinIO documentation website](https://docs.min.io)

## Contribute to MinIO Project
Please follow MinIO [Contributor's Guide](https://github.com/minio/mc/blob/master/CONTRIBUTING.md)

## License
Use of `mc` is governed by the GNU AGPLv3 license that can be found in the [LICENSE](https://github.com/minio/mc/blob/master/LICENSE) file.
