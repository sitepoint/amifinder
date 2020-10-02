# amifinder

Locate the most recently created AMI in an AWS account matching a
given filter.


## Requirements

* Python 3.6 or later


## Setup

Set your AWS environment variables such as `AWS_ACCESS_KEY_ID` and
`AWS_SECRET_ACCESS_KEY`. The default region will be us-west-2 if
`AWS_DEFAULT_REGION` is not defined.

Alternatively, create an `~/.aws/config` file as described
[here](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/configuration.html#using-environment-variables).

The boto3 library for Python 3 needs to be installed using one of the
following options:

**Debian GNU/Linux**
```
$ apt install python3-boto3
```

**pip**
```
$ python3 -m pip install boto3
```

The `amifinder` script can optionally be placed somewhere in your
system PATH for convenient access.


## Usage

```
$ ./amifinder --help
usage: amifinder [-h] [--name NAME] [--architecture ARCHITECTURE]
                 [--hypervisor HYPERVISOR] [--ena-support ENA_SUPPORT]
                 [--state STATE] [--image-type IMAGE_TYPE]
                 [--root-device-type ROOT_DEVICE_TYPE]
                 [--block-device-mapping.volume-type BLOCK_DEVICE_MAPPING.VOLUME_TYPE]
                 ACCOUNT

Fetch the latest AMI of a given type.

positional arguments:
  ACCOUNT               The AWS account ID of the image owner.

optional arguments:
  -h, --help            show this help message and exit
  --name NAME
  --architecture ARCHITECTURE
  --hypervisor HYPERVISOR
  --ena-support ENA_SUPPORT
  --state STATE
  --image-type IMAGE_TYPE
  --root-device-type ROOT_DEVICE_TYPE
  --block-device-mapping.volume-type BLOCK_DEVICE_MAPPING.VOLUME_TYPE
$
```


## Examples

Fetch the latest AMI in the Debian 10 account:

```
$ ./amifinder 136693071363
Name: debian-10-arm64-20200928-407
Architecture: arm64
CreationDate: 2020-09-29T00:00:03.000Z
ImageId: ami-0d2c5c43cf2ea11a3
ImageLocation: 136693071363/debian-10-arm64-20200928-407
ImageType: machine
Public: True
OwnerId: 136693071363
State: available
  DeviceName: /dev/xvda
    DeleteOnTermination: True
    SnapshotId: snap-06198899f450add36
    VolumeSize: 8
    VolumeType: gp2
    Encrypted: False
Description: Debian 10 (20200928-407)
EnaSupport: True
Hypervisor: xen
RootDeviceName: /dev/xvda
RootDeviceType: ebs
SriovNetSupport: simple
VirtualizationType: hvm
$
```

Find the latest Ubuntu x86\_64 AMI:
```
$ ./amifinder --arch x86_64 099720109477
Name: ubuntu-minimal/images-testing/hvm-ssd/ubuntu-groovy-daily-amd64-minimal-20200930
Architecture: x86_64
CreationDate: 2020-09-30T20:50:34.000Z
ImageId: ami-0377770807669cb02
ImageLocation: 099720109477/ubuntu-minimal/images-testing/hvm-ssd/ubuntu-groovy-daily-amd64-minimal-20200930
ImageType: machine
Public: True
OwnerId: 099720109477
State: available
  DeviceName: /dev/sda1
    DeleteOnTermination: True
    SnapshotId: snap-0fbbb57426ca4c768
    VolumeSize: 8
    VolumeType: gp2
    Encrypted: False
  DeviceName: /dev/sdb
  VirtualName: ephemeral0
  DeviceName: /dev/sdc
  VirtualName: ephemeral1
Description: Canonical, Ubuntu Minimal, 20.10, UNSUPPORTED daily amd64 groovy image build on 2020-09-30
EnaSupport: True
Hypervisor: xen
RootDeviceName: /dev/sda1
RootDeviceType: ebs
SriovNetSupport: simple
VirtualizationType: hvm
$
```

The above is roughly equivalent to:
```
$ aws ec2 describe-images --output json --owners 099720109477 --filters Name=architecture,Values="x86_64" | jq 'last(.Images | sort_by(.CreationDate) | .[])'
{
  "Architecture": "x86_64",
  "CreationDate": "2020-09-30T20:50:34.000Z",
  "ImageId": "ami-0377770807669cb02",
  "ImageLocation": "099720109477/ubuntu-minimal/images-testing/hvm-ssd/ubuntu-groovy-daily-amd64-minimal-20200930",
  "ImageType": "machine",
  "Public": true,
  "OwnerId": "099720109477",
  "State": "available",
  "BlockDeviceMappings": [
    {
      "DeviceName": "/dev/sda1",
      "Ebs": {
        "DeleteOnTermination": true,
        "SnapshotId": "snap-0fbbb57426ca4c768",
        "VolumeSize": 8,
        "VolumeType": "gp2",
        "Encrypted": false
      }
    },
    {
      "DeviceName": "/dev/sdb",
      "VirtualName": "ephemeral0"
    },
    {
      "DeviceName": "/dev/sdc",
      "VirtualName": "ephemeral1"
    }
  ],
  "Description": "Canonical, Ubuntu Minimal, 20.10, UNSUPPORTED daily amd64 groovy image build on 2020-09-30",
  "EnaSupport": true,
  "Hypervisor": "xen",
  "Name": "ubuntu-minimal/images-testing/hvm-ssd/ubuntu-groovy-daily-amd64-minimal-20200930",
  "RootDeviceName": "/dev/sda1",
  "RootDeviceType": "ebs",
  "SriovNetSupport": "simple",
  "VirtualizationType": "hvm"
}
```

Find the latest Ubuntu AMI with "ubuntu-focal-20.04-amd64-server" in the name:
```
$ ./amifinder --name '*ubuntu-focal-20.04-amd64-server*' 099720109477
Name: ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-20200924
Architecture: x86_64
CreationDate: 2020-09-25T01:16:43.000Z
ImageId: ami-02c45ea799467b51b
ImageLocation: 099720109477/ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-20200924
ImageType: machine
Public: True
OwnerId: 099720109477
State: available
  DeviceName: /dev/sda1
    DeleteOnTermination: True
    SnapshotId: snap-00cebee15a6a0e9cb
    VolumeSize: 8
    VolumeType: gp2
    Encrypted: False
  DeviceName: /dev/sdb
  VirtualName: ephemeral0
  DeviceName: /dev/sdc
  VirtualName: ephemeral1
Description: Canonical, Ubuntu, 20.04 LTS, amd64 focal image build on 2020-09-24
EnaSupport: True
Hypervisor: xen
RootDeviceName: /dev/sda1
RootDeviceType: ebs
SriovNetSupport: simple
VirtualizationType: hvm
$
```
