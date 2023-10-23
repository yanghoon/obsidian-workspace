# Migration

## Target and Check

* All-Users.git (accounts, groups)
* All-Projects.git (projects)
* database
* repositories (git)
* projects
* changes
* plugins
* config

## Backup and Recovery

## Version Upgrade

```bash
GERRIT_WAR=gerrit.war
java -jar $GERRIT_WAR init -d site_path
```
