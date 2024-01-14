# Sync
## with Git
1. Create Git Repository
2. Install `Opsidian Git` Plugin
3. Configure Pull/Push Interval
### on Desktop
1. ...
### on iOS
...
1. https://meganesulli.com/blog/sync-obsidian-vault-iphone-ipad/
2. https://gist.github.com/DannyQuah/f686c0e43b741468e12515cd79017489
	3. https://forum.obsidian.md/t/mobile-sync-with-git-on-ios-for-free-using-ish/20861

```bash
# Create Empty Vault

# apk update && apk add git
# mkdir obsidian
mount -t ios . obsidian
cd ~/obsidian
rm -rf .obsidian
git clone htttps://github.com/yanghoon/opsidian-workspace .

# on Each Devices
git pull
git push
```

4. https://velog.io/@joshuara7235/%EC%98%B5%EC%8B%9C%EB%94%94%EC%96%B8-%EC%82%AC%EC%9A%A9%ED%95%B4-%EB%B3%B4%EC%8B%A4%EB%9E%98%EC%9A%94
### Guides
* https://curtismchale.ca/2022/05/18/sync-your-obsidian-vault-for-free-with-github/
* https://publish.obsidian.md/git-doc/Installation#From+within+Obsidian
* https://github.com/denolehov/obsidian-git