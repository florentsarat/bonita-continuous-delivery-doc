# Release notes

## Breaking changes
The following changes introduce incompatibility with BCD 3.3:

* BCD controller docker image is now based on Debian Buster instead of Alpine Linux.

If you use a custom controller image you may need to update it depending on your customization (ie: use apt package manager and not apk anymore)

## Limitations and known issues

* The same BCD stack cannot be managed with multiple BCD controller instances due to the use of Terraform "local" backend.
* Due to [Ansible issue #35255](https://github.com/ansible/ansible/issues/35255) the warning message "could not match supplied host pattern" is displayed when there is no load balancer required for a noncluster deployment:
  ```
  [WARNING]: Could not match supplied host pattern, ignoring: load_balancer
  ```

## What's new in 4.0.0 (2020-09-01)

### Features
* feat: upgrade to python3
* feat: upgrade terraform (aws and azure)
* feat: upgrade ansible
* feat: upgrade to java 11
* feat: upgrade bonita version to 7.11.0
* feat: use Debian Buster(slim) as base distribution instead of Alpine Linux

### Bug fixes
* fix: fix google captcha auth
* fix: fix controller version in version command
* fix: fix python interpreter path for ansible
* fix: don't force a specific version of nodejs to build custom themes
* fix: allow user to mount $HOME/.m2 to use maven settings and cache
