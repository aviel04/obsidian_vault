## Data Not Included in [[Backup]]
- [Mattermost data](https://docs.gitlab.com/integration/mattermost/#back-up-gitlab-mattermost)
- Redis (and thus Sidekiq jobs)
- [Object storage](https://docs.gitlab.com/administration/backup_restore/backup_gitlab/#object-storage) on Linux package (Omnibus) / Docker / Self-compiled installations
- [Global server hooks](https://docs.gitlab.com/administration/server_hooks/#create-global-server-hooks-for-all-repositories)
- [File hooks](https://docs.gitlab.com/administration/file_hooks/)
- GitLab configuration files (`/etc/gitlab`)
- TLS- and SSH-related keys and certificates
- Other system files

# Strategies
Backup Strategies Depends on Various Factors of your gitlab setup (gitlab deployment configuration, data volume, storage location).
those determine which methods to use, where to store, and how to structure the backup schedule. 

**larger gitlab instance can use these strategies:**
- incremental [[Backup]]
- [[Backup]] of specific repositories
- [[Backup]] across multiple storage locations

---
## Simple [[Backup]] Procedure
(1k  ref architecture up to 1000 users) with less then 100GB we can use this simple [[Backup]]:
1. Run the [backup command](https://docs.gitlab.com/administration/backup_restore/backup_gitlab/#backup-command).
2. Back up [object storage](https://docs.gitlab.com/administration/backup_restore/backup_gitlab/#object-storage), if applicable.
3. Manually back up [configuration files](https://docs.gitlab.com/administration/backup_restore/backup_gitlab/#storing-configuration-files).
```shell
sudo gitlab-backup create # will take longer as gitlab data grows
```


---

## Scaling Backup
when the data grows we use options in the backup command ( back up git repo concurrently and incremental repository ) to reduce the time it takes.

but sometimes even the backup with options become impractical so we need architecture changes.

## Data that needs to be backed up
- [[Postgresql]] databases
- git repositories
- blobs
- container registry
- configuration files
- and more...


## Skipping 
we can do a partial [[Backup]] using the option `Skip` to skip certain aspects/components in the gitlab [[Backup]]

---
## gitlab backup tool

```ruby
# To create a backup
sudo gitlab-backup create
# This corresponds to gitlab-rake gitlab:backup:create

# To restore a previously-captured backup
sudo gitlab-backup restore BACKUP=<backup_id>
# This corresponds to gitlab-rake gitlab:backup:restore BACKUP=<backup_id>
```

### Continue Reading
https://docs.gitlab.com/development/backup_and_restore/backup_gitlab/
https://docs.gitlab.com/administration/backup_restore/backup_gitlab/#skipTarget
