
## Gitlab Omnibus

## gitlab architecture

- [x] [[Gitlab-Shell]] ✅ 2025-05-11
- [x] [[PostgreSQL]] ✅ 2025-05-11
- [x] [[Nginx]] ✅ 2025-05-11
- [ ] [[Grafana & Prometheus]]
- [x] [[Gitlab-workhorse]] ✅ 2025-05-11
- [ ] [[Gitlab-rails]]
- [ ] [[Redis]]
- [ ] [[Sidekiq]]
- [x] [[Puma]] ✅ 2025-05-11
- [ ] [[Praefect]]
- [ ] [[Gitaly]]

---
## Create Gitlab-Server
- gitlab on LVM
	- add also to fstab
	- var/opt/gitlab var/log/gitlab
- where is the repo info (gitaly) on the machine
	- how do we change the path 
- config git data for the server
- check if you can create a new repo in the gitlab UI 
- config a DB for the gitlab
	- pgadmin 
	- certs 
	- use  external service (DB) and connect to the gitlab service 

---
### what is sso and auth 
- how sso and auth are used in gitlab and why should we use them in gitalb
- openid oauth2
- config sso to gitlab 
- research CA on HTTPS
- how to update ca on gitlab
- why gitlab is failling to hold cert chains when there is multiple certs 
	- what is cert chain 
- what in etc/pki/ca-trust/source/anchors
- openssl of gitlab for the problem
- copy  the ca bundle to th eright address
- should we use a symbolic link for all the certs 
- research logrotate why gitlab uses it 
	- 20 rotating logs
	- active rotation when log is over a certein size
- 