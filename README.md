# Backup Plan

This repository is adapted from Jeff Geerling's my-backup-plan repo. Jeff's data inventory and use case is similar to mine, so I've built upon his work for my own needs.

Having a solid and multi-tiered backup plan gives you peace of mind and the freedom to not feel tied to any particular devices or 'sacred and precious backup drive' that has all your data stored on it. An outline of recovery procedures provides guidance for testing backups and can serve as a playbook during an event. These events may be unplanned such as a device failure, or planned - such as my increasing disdain with subscription services like Dropbox.

- 3 Copies of all my data
- 2 Copies on different storage media
- 1 Offsite copy and immutable copy

Another way to express this could be: 

- Copy 1 is your local copy that you work on and interact with. 
- Copy 2 is your backup copy to easily restore. 
- Copy 3 is your offsite/immutable copy that can accessed in the event that copies 1 and 2 are not a viable option.

## Backup Strategy

With several devices and a mixture of cloud and on-premise services, I'd like a backup strategy that can accomodate a few different scenarios:

**Loss of device** (and local data) through loss, theft or damage
  example: ipad or laptop fails or goes misisng and must be replaced with minimal data loss
  example: ransomware encrypts local device

**Loss of service** access or availablility
  example: icloud or Google account is compromised and locked out. 
  example: service is unavailble through outage or limited connectivity
  
**Loss of infrastructure or site**
  example: Infrastructure failure scenario: internet access, wifi, firewall, NAS, local server services
  example: Natural disaster, fire, theft, extended power outage, etc.
  
The goal with any of these events is to minimize impact during an event. From my experience, I've presented risks in order from most likely to least likely. My goal is that the final version of this plan includes a playbook outlining recovery steps. 

### Data Inventory

Data sprawl is an ongoing challange for many of us. I use multiple laptops, mobile devices, usb drives to create digital content. I've come to recognize that my workflow isn't always condusive to an organized set with perfect backup policies. The first stage of my recovery plan is to consolodate, organize and deduplicate localized data. 


### Consolidation and Deduplication



#### Catagories of Data

- Local Files
- Photos Library
- Video Content
- Code
- Infrastructure Backups
- Music Library


#### Critical Device Inventory - each device I need a recovery procedure for

#### Device Map

| Data Source | Primary Backup | Secondary Backup | Immutable Backup |
|-------------|----------------|------------------|------------------|
| [Phone](https://github.com/nopnoplabs/backup-plan/edit/master/README.md#phone)       | iCloud         |                  |                  |
| [iPad](https://github.com/nopnoplabs/backup-plan/edit/master/README.md#ipad)        | iCloud         |                  |                  |
| Work Laptop | NAS            |                  |                  |
| Personal Lap| NAS            |                  |                  |
| Home Server | NAS            |                  |                  |
| NAS         |                |                  |                  |
| Network Dev.| NAS            |                  |                  |

### Service Inventory

#### Personal
- GSuite:Email, Google Drive, Calendar, Docs, Sheets
- Evernote
- iCloud: Photos, Music, Files

#### Professional
- Adobe Cloud
- GSuite
- Github

### Local Time Machine

Configured to backup to NAS

### Open Source Code Repositories


### Local network configuration

| Network Device   | Primary Backup | Secondary Backup | Immutable Backup |
|------------------|----------------|------------------|------------------|
| Firewall         | Script         |                  |                  |
| Core Switch      | Script         |                  |                  |
| Closet Switch    | Cloud Config   |                  |                  |
| Wifi Controller  | NAS            |                  |                  |

### Video Content and NAS Shares

<!-- The most important data I have on my NAS, and thus the target of much of my effort around ensuring I have a solid backup and recovery plan, is my video project files.

For every video I create (for YouTube and elsewhere), I create a folder, create a separate Final Cut Pro library (with all media stored inside), and generate all assets, scripts, etc. inside that same directory.

I typically work on the video locally on my Mac mini until it's complete, relying on hourly Time Machine backups to store all the footage over on my NAS.

But once complete, I copy a datestamped project folder over to the NAS, and then eventually delete that project folder off my Mac. At that point, there is only one copy in existence, and that makes me nervous.

To help alleviate my nerves, I maintain two backups:

  1. A weekly script, [`backup.sh`](backup.sh), runs every Sunday night and synchronizes _all_ my video project data, as well as a number of other important-but-large directories of my data, to an AWS S3 Glacier Deep Archive bucket.
  2. A daily sync job sychronizes all the shared volumes from my primary NAS to a secondary local NAS.

In this way I always have two local copies (on my two online NASes), plus one offsite offline copy (on AWS Glacier Deep Archive). I used to just have one onsite copy and one offsite, but I learned the lesson of having a backup copy onsite when I accidentally deleted a video project folder once... and then had to wait a day or so to download it from Glacier.

But I'm happy with Glacier, because for a backup that's 8TB and growing, I only pay about $4/month for the storage. And especially with redundant NASes, it's only meant to be a truly emergency-level backup, and waiting a bit to save on cost is not a problem.

### Things I'm still working on

  - End-to-End encryption: Some backups and some transports are not fully encrypted. I'd like to change that.
  - Complete Disaster Recovery plans: Some backups have only been hand-checked, and I don't currently have a complete (and _tested_) plan for the restore process. Trying to recover data the first time in an emergency is a recipe for accidentally making things worse.

## `main.yml` - Ansible playbook to configure my Backup Pi

There's an Ansible playbook that installs all the backup software I use, configures shared filesystem mounts, and configures a backup cron job, all on a single 'Backup' Raspberry Pi. To run the playbook:

  1. Make sure you have Ansible installed.
  2. Copy `example.inventory.ini` to `inventory.ini` and `example.config.yml` to `config.yml`, and modify them according to your needs.
  3. Run `ansible-galaxy install -r requirements.yml`
  4. Run the playbook: `ansible-playbook main.yml`

### Manual `rclone` setup

For security purposes, I don't keep the entire `rclone` config in this repository. I could via Ansible Vault, but I don't. Sue me.

So after running the playbook, for `rclone` to actually work, you'll need to do the following manually, one time:

Run `rclone config` following the [S3 setup instructions](https://rclone.org/s3/#amazon-s3).

  - Set the remote name to `personal`.
  - Set the type of storage to `s3`.
  - Set the S3 provider to `AWS`.
  - For access credentials, I created a limited `rclone` user in AWS Console and have an access key set up for that user.
  - Set the region to `us-east-1`.
  - Set the ACL to `private`.
  - Set the storage class to `DEEP_ARCHIVE`.

Do all of this as the `pi` user (or whatever user you're going to configure to run the backups).

### Manual `gickup` setup

I also avoid keeping the entire `gickup` config in this repository. Sosumi.

After running the playbook, you'll need to add a Gickup config file named `~/.gickup.yml`, with contents as seen in top of the `gickup.sh` file.

You will also need to make sure an SSH key has been added to your GitHub account so the backup server can access GitHub, and you should create a Personal Access Token and paste the token where indicated.

Do all of this as the `pi` user (or whatever user you're going to configure to run the backups).

## `backup.sh` - Rclone to S3 Glacier Deep Archive

You can manually run `backup.sh` the first time and watch it do its magic:

```
pi@backup:~ $ ./backup.sh 
```

The initial backup could take a while—mine took over two weeks :)

Caveats with Glacier Deep Archive:

  1. You can't easily move objects around inside the bucket. So don't just dump stuff into Deep Archive that's going to move around a lot, especially very large files that would need a re-upload or to be downloaded then moved.
  2. Retrieval takes time—at least 12 hours _just to restore an object to your Bucket so you can start downloading it_. There is no 'expedited' option with Deep Archive.

## `gickup.sh` - Synchronize GitHub repositories to NAS

You can manually run `gickup.sh` the first time and watch it do its magic:

```
pi@backup:~ $ ./gickup.sh 
```

The initial clone of all repositories takes an hour or two, but once complete, re-syncs should only take a few minutes, as it only fetches and pulls, and doesn't have to re-clone each repo.

Caveats with Gickup:

  1. You can't currently have Gickup prune repos that were deleted in the GitHub account.
  2. Cloning large repos (multiple GB) will cause the OOM killer to kill the process if running on a lower-memory Pi.

## Retriving content from S3 Glacier Deep Archive

**For individual file retrieval**, see: [Retrieving individual files from S3 Glacier Deep Archive using AWS CLI](https://www.jeffgeerling.com/blog/2021/retrieving-individual-files-s3-glacier-deep-archive-using-aws-cli)

-->

## Recovery Procedures

### Phone

Replace device, sign in to iCloud - preferably on a fast wifi connection. Default backup interval is daily.

### iPad

Replace device, sign in to iCloud - preferably on a fast wifi connection. Default backup interval is daily.


