# Übermail
An Ansible role to build and maintain your own mailstack. Hardforked and refactored from Sovereign.

## Intro
Sovereign (https://github.com/sovereign/sovereign) is an awesome set of Ansible playbooks that you can use to build and maintain your own personal cloud (email, calendar, contacts, file sync, IRC bouncer, VPN, and more) based entirely on open source software. 
While using Sovereign, i kept running into small but annoying issues, that mosty stemmed from misconfiguration, outdated packages and a huge maintainance burden / attack surface due to providing a complete cloud. Eventually I ended up using (a patched and redacted version of) Sovereign, that focused on providing only the mailstack, but better (for me at least). 

Übermail is a hardfork of my patched and redacted version of Sovereign. With Übermail my aim is to 

- further refactor the code while making up my mind on "how" along the way
- stick to providing a complete mailstack only
- cherrypicking ideas from Sovereign-propper and other cool projects such as Mailcow or IRedMail.

## Roadmap

- [x] Ported from Debian 8 to Ubuntu 18.04
- [x] Postgres bump to 10
- [x] Dropping postgrey in favour of the graylisting rspamd filter
- [x] Bumping Solr/Tomcat
- [ ] Bumping z-push
- [x] Adding support for https://github.com/imapsync/imapsync/
- [x] Improving the webmail experience (roundcube bump, roundcube plugins, other clients)
- [x] Code cleanup
- [ ] Multidomain autoconfig support
- [x] Replace ntp with crony
- [ ] ...

## Requirements

This role currently requires simmilar DNS records:

```
example.com 	MX 	mail.example.com
example.com 	A 	<your-ip-addr>
autoconfig.example.com 	A 	<your-ip-addr>
mail.example.com 	A 	<your-ip-addr>
```

## Expectable result

- A complete mailstack with postfix/dovecot/rspamd/solr/roundcubemail/certbot
- Rspamd providing graylisting, antivirus, spamfiltering, dkim
- Monitoring and security provided by monit, collectd, and lots of other utilities
- Webmail on https://mail.example.com/
- Rspamd administration on https://mail.example.com/rspamd/

## Contributions welcomed

Wanna send a PR? Wanna join as a dev/release eng.? Wanna help otherwise? Great! I'll be really happy to welcome you on board.

## Notes

- Test your (inbound mail) spamfilter with https://www.emailsecuritycheck.net/
- Test your (outbound mail) spammyness with https://www.mail-tester.com/