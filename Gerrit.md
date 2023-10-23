# Backup and Recovery

* All-Users.git (accounts, groups)
* All-Projects.git (projects)
* database
* repositories (git)
* projects
* changes
* plugins
* config

# Version Upgrade

```bash
NEW_VERSION=
java -jar gerrit-$NEW_VERSION.war init -d site_path
```
