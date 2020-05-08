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
- [x] Bumping z-push
- [x] Adding support for https://github.com/imapsync/imapsync/
- [x] Improving the webmail experience (roundcube bump, roundcube plugins, other clients)
- [x] Code cleanup
- [x] Multidomain autoconfig support
- [x] Replace ntp with crony
- [ ] Delegate some configuration to specialized roles (i.e. ssh, php, etc.)
- [ ] Security audit & various hardening
- [ ] Create/modify users without ansible, keep these somewhere (so that they don't get overwritten)

## Requirements

This role currently requires simmilar DNS records:

```
example.com 	MX 	mail.example.com
example.com 	A 	<your-ip-addr>
autoconfig.example.com 	A 	<your-ip-addr>
mail.example.com 	A 	<your-ip-addr>
```

A full DNS configuration guide (including SPF, DKIM, PTR, and DMARC) will be generated for each virtual domain in /opt/ubermail

## Example playbook

```
###
### Ubermail configuration
### 

mail_rspamd_password: <your-very-long-random-string-here>
domain: "ubermail.com"
main_user_name: "admin" # setting root here will fail the google authenticator common task because /home/$user is expected, not /root
common_timezone: 'Europe/Prague'

###
### Ubermail domains
###


mail_virtual_domains:
  - { pk_id: 1,  name: "{{ domain }}", rfc2142_destination: "{{ admin_email }}" }
  - { pk_id: 2,  name: "example.com", rfc2142_destination: "{{ admin_email }}"  }
  - { pk_id: 3,  name: "example.org", rfc2142_destination: "{{ admin_email }}"  }

# if {{ rfc2142_destination }} is set, common mailboxes, specifically:
# root@domain, abuse@domain, hostmaster@domain, postmaster@domain, webmaster@domain
# will become aliases of {{ rfc2142_destination }}

mail_virtual_users:
  # ubermail.com
  - { domain_pk_id: 1, domain: "{{ domain }}", account: "{{ main_user_name }}", password: "admins-secret-password" }
  # example.com personal
  - { domain_pk_id: 2, domain: example.com,    account: user1, password: "password1" }
  - { domain_pk_id: 2, domain: example.com,    account: user2, password: "password2" }
  # example.org personal
  - { domain_pk_id: 3, domain: example.org, account: looser1, password: "pwdlo1" }
  - { domain_pk_id: 3, domain: example.org, account: looser2, password: "pwdlo2" }

mail_virtual_aliases: # pk = source
  - { domain_pk_id: 2, source: "hello@example.com", destination: "user1@example.com, looser2@example.org" }
  - { domain_pk_id: 2, source: "bye@example.com",   destination: "user2@example.com" }

mail_virtual_sieves:
  - domain: "example.com"
    account: user1
    rule:
      - name: move-beer
        cond: 'anyof (header :contains "subject" "beer")'
        acts: |
           fileinto "INBOX.beer";
```

## Expectable result

- A complete mailstack with postfix/dovecot/rspamd/solr/roundcubemail/certbot
- Rspamd providing graylisting, antivirus, spamfiltering, dkim
- Monitoring and security provided by monit, collectd, and lots of other utilities
- Webmail on https://mail.example.com/
- Rspamd administration on https://mail.example.com/rspamd/
- Autoconfig on https://autoconfig.example.com/

## Testing, ensuring mail delivery

- Verify DKIM/DMARC/SPF/PTR and more at https://www.mail-tester.com/ to probe for DNS settings errors. You should get a 
  perfect 10/10
- Check https://whatismyipaddress.com/blacklist-check to see, if your server's IP address isn't listed on common blacklists.
- Check if Google delivers your e-mails to gmail mailboxes. Even a perfectly managed mailserver with no spam behavior
  can get penalized. Try reporting to https://support.google.com/mail/contact/msgdelivery and/or registering txt record 
  for your domains with at https://postmaster.google.com. All hail our new overlord :-/
- If e-mails still don't deliver, try to remove all asigned IPv6 addresses and/or completely disable the IPv6 stack on your
  server/VPS. Google (and some others providers) can penalize heavily, if IPv6 is detected on a mailserver.
- Need more extensive delivery testing? Signup for a free account https://glockapps.com/ to run tests against all major
  freemail providers.

## Contributions welcomed

Wanna send a PR? Wanna join as a dev/release eng.? Wanna help otherwise? Great! I'll be really happy to welcome you on board.

## Notes

- Test your (inbound mail) spamfilter with https://www.emailsecuritycheck.net/
- Test your (outbound mail) spammyness with https://www.mail-tester.com/
- Spam lists etc. to check for:
  - https://ipremoval.sms.symantec.com/ipr/lookup (Symantec)
  - https://postmaster.verizonmedia.com/ (AOL)
  - https://sendersupport.olc.protection.outlook.com/snds/addnetwork.aspx (Outlook)


## Similar projects

Ubermail is not for you? Grab a copy of

- http://github.com/sovereign/sovereign
- https://github.com/Mailu/Mailu (https://mailu.io)
- https://github.com/mattsta/mailweb (https://matt.sh/email2018)
- https://github.com/mailcow/mailcow-dockerized (https://mailcow.email/)
- https://www.iredmail.org/

