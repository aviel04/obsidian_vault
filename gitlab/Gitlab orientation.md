
## Gitlab Ombinbus

## gitlab architecture

- [[gitlab-shell]]
- pstgresql
- nginx
- grafana
- Prometheus
- gitlab-workhorse
- gitlab-rails
- Redis
- sidekiq
- puma
- praefect
- gitaly

---
## Create Gitlab-Server
- gitalb on LVM
	- add also to fstab
	- var/opt/gitalb var/log/gitalb
- where si the repo info (gitaly) on the machine
	- how do we chage the path 
- config git dataa for the server
- check if you can create a new repo in the gitlab ui 
- config a db for the gitlab
	- pgadmin 
	- certs 
	- use  external service (db) and connect to the gitlab service 

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