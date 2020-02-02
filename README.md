# Hosting a Web Key Directory on GitHub Pages

This repository contains a simple [WKD](https://wiki.gnupg.org/WKD) for
openpgp key discovery created based on  instructions at
https://wiki.gnupg.org/WKDHosting

According to the instructions I found you should be able to host the WKD from
either the apex domain (direct URL scheme) or a special __openpgpkey__
subdomain. I couldn't make hosting from the apex domain working so I ended up
using the subdomain, which worked fine.

This means the CNAME you will need is __openpgpkey.yourdomain.com__.

First follow the [instructions](https://help.github.com/en/github/working-with-github-pages/configuring-a-custom-domain-for-your-github-pages-site) for creating the GitHub Pages site with a CNAME.

Now the URL __openpgpkey.yourdomain.com__ should be available and serving
content from your GitHub repository.

The next step is to create the directory structure and add the gpg key.

Go to your checked out repository and create a __.well-known__ folder at the
root.

```
cd openpgpkey  # assuming this is what the checked out repository is
mkdir -p .well-known/openpgpkey
cd .well-known
```

You need GPG >= 2.2.12 that has __gpg-wks-client__, otherwise see alternative
methods at https://wiki.gnupg.org/WKDHosting.

```
gpg --list-options show-only-fpr-mbox -k "@yourdomain.com" | $(gpgconf --list-dirs libexecdir)/gpg-wks-client -v --install-key

```

The command will create the folder structure under the __.well-known__ folder and add the key with the correct filename for the email address in the key. The command output looks like this:

```
gpg-wks-client: gpg: Total number processed: 1
gpg-wks-client: using key with user id 'Robert Mihaly <rob@mihalyr.com>'
gpg-wks-client: gpg: Total number processed: 1
gpg-wks-client: directory 'openpgpkey/mihalyr.com' created
gpg-wks-client: directory 'openpgpkey/mihalyr.com/hu' created
gpg-wks-client: policy file 'openpgpkey/mihalyr.com/policy' created
gpg-wks-client: key 96E4FD37F2D56178E2B7E3A2C89FE343D529E0CF published for 'rob@mihalyr.com'
```

The created folder structure will be similar to this:

```
.well-known
.well-known/openpgpkey
.well-known/openpgpkey/mihalyr.com
.well-known/openpgpkey/mihalyr.com/policy
.well-known/openpgpkey/mihalyr.com/hu
.well-known/openpgpkey/mihalyr.com/hu/xarhuw9jcphm6ir9akb945o6mpabjubu
```

I made also the following changes to the repository:

1. Removed all other files from the repo, only left the __.well-known__ folder and the __CNAME__ file that was added by GitHub when configured the page.
2. Added and empty index.html file, probably not needed by wanted to have at least a blank page when I was testing things.
3. Added __.no-jekyll__ file to tell GitHub to don't bother buidling this site with Jekyll.

This is currently what I have in my repo:

```
.  ..  CNAME  .git  index.html  .nojekyll  .well-known
```

That's it. Let's test it.


Newer GPG uses also WKD when using the `--locate-key` option e.g. the
following command should find the key now:

```
gpg --auto-key-locate clear,wkd,nodefault --verbose --locate-key you@yourdomain.com
```

Here is what the output looks like:

```
$ gpg --auto-key-locate clear,wkd,nodefault --verbose --locate-key rob@mihalyr.com
gpg: using pgp trust model
gpg: pub  ed25519/C89FE343D529E0CF 2019-11-10  Robert Mihaly <rob@mihalyr.com>
gpg: key C89FE343D529E0CF: "Robert Mihaly <rob@mihalyr.com>" not changed
gpg: Total number processed: 1
gpg:              unchanged: 1
gpg: auto-key-locate found fingerprint 96E4FD37F2D56178E2B7E3A2C89FE343D529E0CF
gpg: automatically retrieved 'rob@mihalyr.com' via WKD
pub   ed25519 2019-11-10 [C] [expires: 2021-11-09]
      96E4FD37F2D56178E2B7E3A2C89FE343D529E0CF
uid           [ unknown] Robert Mihaly <rob@mihalyr.com>
sub   ed25519 2019-11-10 [S] [expires: 2020-11-09]
sub   cv25519 2019-11-10 [E] [expires: 2020-11-09]
sub   ed25519 2019-11-10 [A] [expires: 2020-11-09]
```

It shows `automatically retrieved 'rob@mihalyr.com' via WKD` proving that the
hosting works.

Another method to test it is using `gpg-wks-client`:

```
$(gpgconf --list-dirs libexecdir)/gpg-wks-client -v --check you@yourdomain.com
```

The output should look like this:

```
$ $(gpgconf --list-dirs libexecdir)/gpg-wks-client -v --check rob@mihalyr.com
gpg-wks-client: public key for 'rob@mihalyr.com' found via WKD
gpg-wks-client: gpg: Total number processed: 1
gpg-wks-client: fingerprint: 96E4FD37F2D56178E2B7E3A2C89FE343D529E0CF
gpg-wks-client:     user-id: Robert Mihaly <rob@mihalyr.com>
gpg-wks-client:     created: Sun 10 Nov 2019 09:35:30 PM CET
gpg-wks-client:   addr-spec: rob@mihalyr.com
```

Now any email client using `gpg --locate-keys` should automatically find your
hosted key. You can find a list of email clients and email providers that are
known to be using WKD here https://wiki.gnupg.org/WKD
