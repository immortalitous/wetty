# WeTTY = Web + TTY
> Terminal access in browser over HTTP/HTTPS

<!-- ALL-CONTRIBUTORS-BADGE:START - Do not remove or modify this section -->
![All Contributors](https://img.shields.io/badge/all_contributors-33-orange.svg?style=flat-square)
<!-- ALL-CONTRIBUTORS-BADGE:END -->

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://github.com/butlerx/wetty/blob/master/LICENSE)

Terminal over HTTP and HTTPS. WeTTy is an alternative to ajaxterm and anyterm (but much better than them), because WeTTy uses xterm.js which is a full fledged implementation of terminal emulation written entirely in JavaScript. WeTTy uses websockets rather then Ajax and hence better response time.

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Installation](#installation)
3. [Usage](#usage)
4. [Apache Configuration](#apache_configuration)
5. [HTTPS with SSL-certificate](#https_with_ssl-certificate)
6. [Autostart with systemd-file](#autostart_with_systemd-file)
7. [Customization](#customization)
    1. [Setup a custom favicon](#setup_a_custom_favicon)
    2. [Setup a custom theme](#setup_a_custom_theme)
8. [Flags](#flags)
    1. [Server Port](#server_port)
    2. [SSH Host](#ssh_host)
    3. [Default User](#default_user)
    4. [SSH Port](#ssh_port)
    5. [WeTTy URL](#wetty_url)
9. [File Downloading](#file_downloading)
10. [FAQ](#faq)
11. [Author](#author)
12. [Contributors](#contributors)
13. [License](#license)

<a name="prerequisites"/></a>
## Prerequisites
- python
- node >=6.9
- yarn

<a name="installation"/></a>
## Installation
```bash
$ git clone https://github.com/immortalitous/wetty.git
$ cd wetty
$ yarn
$ yarn build
```

<a name="usage"/></a>
## Usage
```sh
wetty [-h] [--port PORT] [--base BASE] [--sshhost SSH_HOST] [--sshport SSH_PORT] [--sshuser SSH_USER] [--host HOST] [--command COMMAND] [--forcessh] [--bypasshelmet] [--title TITLE] [--sslkey SSL_KEY_PATH] [--sslcert SSL_CERT_PATH]
```

Open your browser on `http://yourserver:3000/wetty` and you will prompted to login. Or go to `http://yourserver:3000/wetty/ssh/<username>` to specify the user before hand.

If you run it as root it will launch `/bin/login` (where you can specify the user name), else it will launch `ssh` and connect by default to `localhost`. The SSH connection can be forced using the `--forcessh` option.

If instead you wish to connect to a remote host you can specify the `--sshhost` option, the SSH port using the `--sshport` option and the SSH user using the `--sshuser` option.

<a name="apache_configuration"/></a>
## Apache Configuration
Create a new VirtualHost in your configuration with your favorite text editor:
```bash
$ nano /etc/apache2/sites-available/subdomain.domain.tld
```
And enter the following:
```sh
<VirtualHost *:80>
    ServerName subdomain.domain.tld
    ProxyPassMatch / http://127.0.0.1:PORT
</VirtualHost>
```

<a name="https_with_ssl-certificate"/></a>
## HTTPS with SSL-certificate
Modify the Apache configuration by adding a VirtualHost which handles the port `443`:
```sh
<VirtualHost *:80>
    ServerName subdomain.domain.tld
    Redirect / https://subdomain.domain.tld
</VirtualHost>

<VirtualHost *:443>
    ServerName subdomain.domain.tld
    ProxyPassMatch / http://127.0.0.1:PORT
    Include /path/to/config-file.conf
    SSLCertificateFile /path/to/fullchain.pem
    SSLCertificateKeyFile /path/to/privkey.pem
</VirtualHost>
```

<a name="autostart_with_systemd-file"/></a>
## Autostart with systemd-file
Copy the `wetty.service` file located in [./bin/](https://github.com/immortalitous/wetty/tree/master/bin) to the systemd folder:
```bash
$ cp /path/to/wetty/bin/wetty.service /etc/systemd/system/wetty.service
```

Standard configuration: `wetty -p 3000 --host 127.0.0.1 -b "" --forcessh --title ""`

<a name="customization"/></a>
## Customization

<a name="setup_a_custom_favicon"/></a>
### Setup a custom favicon
Copy your preferred `favicon.ico` with the following command:
```bash
$ cp /path/to/custom_favicon.ico /path/to/wetty/src/client/favicon.ico
```

<a name="setup_a_custom_theme"/></a>
### Setup a custom theme
Go into [./src/client/](https://github.com/immortalitous/wetty/tree/master/src/client):
```bash
$ cd /path/to/wetty/src/client/
```
And edit the `options.ts` file with your preferred text editor:
```bash
$ nano ./options.ts
```
Replace `FG_COLOR` and `BG_COLOR` with your custom colors:
```typescript
import { isUndefined } from 'lodash';

export default function loadOptions(): object {
  const defaultOptions = { fontSize: 14 , theme: { foreground: "FG_COLOR", background: "BG_COLOR" }};
  try {
    return isUndefined(localStorage.options)
      ? defaultOptions
      : JSON.parse(localStorage.options);
  } catch {
    return defaultOptions;
  }
}
```

<a name="flags"/></a>
## Flags
WeTTy can be run with the `--help` flag to get a full list of flags.

<a name="server_port"/></a>
### Server Port
WeTTy runs on port `3000` by default. You can change the default port by starting with the `--port` or `-p` flag.

<a name="ssh_host"/></a>
### SSH Host
If WeTTy is run as root while the host is set as the local machine it will use the `login` binary rather than SSH. If no host is specified it will use
`localhost` as the SSH host.

If instead you wish to connect to a remote host you can specify the host with the `--sshhost` flag and pass the IP or DNS address of the host you
want to connect to.

<a name="default_user"/></a>
### Default User
You can specify the default user used to SSH to a host using the `--sshuser`. This user can overwritten by going to
`http://yourserver:3000/wetty/ssh/<username>`. If this is left blank a user will be prompted to enter their username when they connect.

<a name="ssh_port"/></a>
### SSH Port
By default WeTTy will try to SSH to port `22`, if your host uses an alternative SSH port this can be specified with the flag `--sshport`.

<a name="wetty_url"/></a>
### WeTTy URL
If you'd prefer an HTTP base prefix other than `/wetty`, you can specify that with `-b` or `--base`.

**Do not set this to `/ssh/${something}`, as this will break username matching code.**

<a name="file_downloading"/></a>
## File Downloading
WeTTy supports file downloads by printing terminal escape sequences between a Base64 encoded file.

The terminal escape sequences used are `^[[5i` and `^[[4i` (VT100 for "enter auto print" and "exit auto print" respectively -
https://vt100.net/docs/tp83/appendixc.html).

An example of a helper script that prints the terminal escape characters and Base64's stdin:

```bash
$ cat wetty-download.sh
#!/bin/sh
echo '^[[5i'$(cat /dev/stdin | base64)'^[[4i'
```

You are then able to download files via WeTTy!

```bash
$ cat my-pdf-file.pdf | ./wetty-download.sh
```

WeTTy will then issue a popup like the following that links to a local file blob:<br />
`Download ready: file-20191015233654.pdf`

<a name="faq"/></a>
## FAQ
<a name="what_browser_are_supported?"/></a>
### What browsers are supported?
WeTTy supports all browsers that
[xterm.js supports](https://github.com/xtermjs/xterm.js#browser-support).

<a name="author"/></a>
## Author
**Cian Butler <butlerx@notthe.cloud>**
- Github: [@butlerx](https://github.com/butlerx)
- Twitter: [@cianbutlerx](https://twitter.com/cianbutlerx)

<a name="contributors"/></a>
## Contributors
<!-- ALL-CONTRIBUTORS-LIST:START - Do not remove or modify this section -->
<!-- prettier-ignore-start -->
<!-- markdownlint-disable -->
<table>
  <tr>
    <td align="center"><a href="http://cianbutler.ie"><img src="https://avatars1.githubusercontent.com/u/867930?v=4" width="100px;" alt="Cian Butler"/><br /><sub><b>Cian Butler</b></sub></a><br /><a href="https://github.com/butlerx/WeTTy/commits?author=butlerx" title="Code">💻</a> <a href="https://github.com/butlerx/WeTTy/commits?author=butlerx" title="Documentation">📖</a></td>
    <td align="center"><a href="http://about.me/krishnasrinivas"><img src="https://avatars0.githubusercontent.com/u/634494?v=4" width="100px;" alt="Krishna Srinivas"/><br /><sub><b>Krishna Srinivas</b></sub></a><br /><a href="https://github.com/butlerx/WeTTy/commits?author=krishnasrinivas" title="Code">💻</a></td>
    <td align="center"><a href="https://github.com/acalatrava"><img src="https://avatars1.githubusercontent.com/u/8502129?v=4" width="100px;" alt="acalatrava"/><br /><sub><b>acalatrava</b></sub></a><br /><a href="https://github.com/butlerx/WeTTy/commits?author=acalatrava" title="Code">💻</a></td>
    <td align="center"><a href="https://github.com/Strubbl"><img src="https://avatars3.githubusercontent.com/u/97055?v=4" width="100px;" alt="Strubbl"/><br /><sub><b>Strubbl</b></sub></a><br /><a href="https://github.com/butlerx/WeTTy/commits?author=Strubbl" title="Code">💻</a></td>
    <td align="center"><a href="https://github.com/2sheds"><img src="https://avatars3.githubusercontent.com/u/16163?v=4" width="100px;" alt="Oleg Kurapov"/><br /><sub><b>Oleg Kurapov</b></sub></a><br /><a href="https://github.com/butlerx/WeTTy/commits?author=2sheds" title="Code">💻</a></td>
    <td align="center"><a href="http://www.rabchev.com"><img src="https://avatars0.githubusercontent.com/u/1876061?v=4" width="100px;" alt="Boyan Rabchev"/><br /><sub><b>Boyan Rabchev</b></sub></a><br /><a href="https://github.com/butlerx/WeTTy/commits?author=rabchev" title="Code">💻</a></td>
    <td align="center"><a href="https://github.com/nosemeocurrenada"><img src="https://avatars1.githubusercontent.com/u/3845708?v=4" width="100px;" alt="Jimmy"/><br /><sub><b>Jimmy</b></sub></a><br /><a href="https://github.com/butlerx/WeTTy/commits?author=nosemeocurrenada" title="Code">💻</a></td>
  </tr>
  <tr>
    <td align="center"><a href="http://www.gerritforge.com"><img src="https://avatars3.githubusercontent.com/u/182893?v=4" width="100px;" alt="Luca Milanesio"/><br /><sub><b>Luca Milanesio</b></sub></a><br /><a href="https://github.com/butlerx/WeTTy/commits?author=lucamilanesio" title="Code">💻</a></td>
    <td align="center"><a href="http://anthonyjund.com"><img src="https://avatars3.githubusercontent.com/u/39376331?v=4" width="100px;" alt="Anthony Jund"/><br /><sub><b>Anthony Jund</b></sub></a><br /><a href="https://github.com/butlerx/WeTTy/commits?author=antonyjim" title="Code">💻</a></td>
    <td align="center"><a href="https://www.mirtouf.fr"><img src="https://avatars3.githubusercontent.com/u/5165058?v=4" width="100px;" alt="mirtouf"/><br /><sub><b>mirtouf</b></sub></a><br /><a href="https://github.com/butlerx/WeTTy/commits?author=mirtouf" title="Code">💻</a></td>
    <td align="center"><a href="https://cor-net.org"><img src="https://avatars1.githubusercontent.com/u/556693?v=4" width="100px;" alt="Bertrand Roussel"/><br /><sub><b>Bertrand Roussel</b></sub></a><br /><a href="https://github.com/butlerx/WeTTy/commits?author=CoRfr" title="Code">💻</a></td>
    <td align="center"><a href="https://www.benl.com.au/"><img src="https://avatars0.githubusercontent.com/u/6703966?v=4" width="100px;" alt="Ben Letchford"/><br /><sub><b>Ben Letchford</b></sub></a><br /><a href="https://github.com/butlerx/WeTTy/commits?author=benletchford" title="Code">💻</a></td>
    <td align="center"><a href="https://github.com/SouraDutta"><img src="https://avatars0.githubusercontent.com/u/33066261?v=4" width="100px;" alt="SouraDutta"/><br /><sub><b>SouraDutta</b></sub></a><br /><a href="https://github.com/butlerx/WeTTy/commits?author=SouraDutta" title="Code">💻</a></td>
    <td align="center"><a href="https://github.com/koushikmln"><img src="https://avatars3.githubusercontent.com/u/8670988?v=4" width="100px;" alt="Koushik M.L.N"/><br /><sub><b>Koushik M.L.N</b></sub></a><br /><a href="https://github.com/butlerx/WeTTy/commits?author=koushikmln" title="Code">💻</a></td>
  </tr>
  <tr>
    <td align="center"><a href="https://imu.li/"><img src="https://avatars3.githubusercontent.com/u/4085046?v=4" width="100px;" alt="Imuli"/><br /><sub><b>Imuli</b></sub></a><br /><a href="https://github.com/butlerx/WeTTy/commits?author=imuli" title="Code">💻</a></td>
    <td align="center"><a href="https://github.com/perpen"><img src="https://avatars2.githubusercontent.com/u/9963805?v=4" width="100px;" alt="perpen"/><br /><sub><b>perpen</b></sub></a><br /><a href="https://github.com/butlerx/WeTTy/commits?author=perpen" title="Code">💻</a></td>
    <td align="center"><a href="https://nathanleclaire.com"><img src="https://avatars3.githubusercontent.com/u/1476820?v=4" width="100px;" alt="Nathan LeClaire"/><br /><sub><b>Nathan LeClaire</b></sub></a><br /><a href="https://github.com/butlerx/WeTTy/commits?author=nathanleclaire" title="Code">💻</a></td>
    <td align="center"><a href="https://github.com/MiKr13"><img src="https://avatars2.githubusercontent.com/u/34394719?v=4" width="100px;" alt="Mihir Kumar"/><br /><sub><b>Mihir Kumar</b></sub></a><br /><a href="https://github.com/butlerx/WeTTy/commits?author=MiKr13" title="Code">💻</a></td>
    <td align="center"><a href="http://redhat.com"><img src="https://avatars0.githubusercontent.com/u/540893?v=4" width="100px;" alt="Chris Suszynski"/><br /><sub><b>Chris Suszynski</b></sub></a><br /><a href="https://github.com/butlerx/WeTTy/commits?author=cardil" title="Code">💻</a></td>
    <td align="center"><a href="http://9wd.de"><img src="https://avatars1.githubusercontent.com/u/1257835?v=4" width="100px;" alt="Felix Bartels"/><br /><sub><b>Felix Bartels</b></sub></a><br /><a href="https://github.com/butlerx/WeTTy/commits?author=fbartels" title="Code">💻</a></td>
    <td align="center"><a href="https://github.com/jarrettgilliam"><img src="https://avatars3.githubusercontent.com/u/5099690?v=4" width="100px;" alt="Jarrett Gilliam"/><br /><sub><b>Jarrett Gilliam</b></sub></a><br /><a href="https://github.com/butlerx/WeTTy/commits?author=jarrettgilliam" title="Code">💻</a></td>
  </tr>
  <tr>
    <td align="center"><a href="https://harrylee.me"><img src="https://avatars0.githubusercontent.com/u/7056279?v=4" width="100px;" alt="Harry Lee"/><br /><sub><b>Harry Lee</b></sub></a><br /><a href="https://github.com/butlerx/WeTTy/commits?author=harryleesan" title="Code">💻</a></td>
    <td align="center"><a href="http://andreask.cs.illinois.edu"><img src="https://avatars3.githubusercontent.com/u/352067?v=4" width="100px;" alt="Andreas Klöckner"/><br /><sub><b>Andreas Klöckner</b></sub></a><br /><a href="https://github.com/butlerx/WeTTy/commits?author=inducer" title="Code">💻</a></td>
    <td align="center"><a href="https://github.com/DenisKramer"><img src="https://avatars1.githubusercontent.com/u/23534092?v=4" width="100px;" alt="DenisKramer"/><br /><sub><b>DenisKramer</b></sub></a><br /><a href="https://github.com/butlerx/WeTTy/commits?author=DenisKramer" title="Code">💻</a></td>
    <td align="center"><a href="https://github.com/vamship"><img src="https://avatars0.githubusercontent.com/u/7143376?v=4" width="100px;" alt="Vamshi K Ponnapalli"/><br /><sub><b>Vamshi K Ponnapalli</b></sub></a><br /><a href="https://github.com/butlerx/WeTTy/commits?author=vamship" title="Code">💻</a></td>
    <td align="center"><a href="https://tridnguyen.com"><img src="https://avatars1.githubusercontent.com/u/1652595?v=4" width="100px;" alt="Tri Nguyen"/><br /><sub><b>Tri Nguyen</b></sub></a><br /><a href="https://github.com/butlerx/WeTTy/commits?author=tnguyen14" title="Documentation">📖</a></td>
    <td align="center"><a href="https://felix.pojtinger.com/"><img src="https://avatars1.githubusercontent.com/u/28832235?v=4" width="100px;" alt="Felix Pojtinger"/><br /><sub><b>Felix Pojtinger</b></sub></a><br /><a href="https://github.com/butlerx/WeTTy/commits?author=pojntfx" title="Documentation">📖</a></td>
    <td align="center"><a href="https://nealey.github.io/"><img src="https://avatars3.githubusercontent.com/u/423780?v=4" width="100px;" alt="Neale Pickett"/><br /><sub><b>Neale Pickett</b></sub></a><br /><a href="https://github.com/butlerx/WeTTy/commits?author=nealey" title="Code">💻</a></td>
  </tr>
  <tr>
    <td align="center"><a href="https://www.matthewpiercey.ml"><img src="https://avatars3.githubusercontent.com/u/22581026?v=4" width="100px;" alt="Matthew Piercey"/><br /><sub><b>Matthew Piercey</b></sub></a><br /><a href="https://github.com/butlerx/WeTTy/commits?author=mtpiercey" title="Documentation">📖</a></td>
    <td align="center"><a href="https://github.com/kholbekj"><img src="https://avatars3.githubusercontent.com/u/2786571?v=4" width="100px;" alt="Kasper Holbek Jensen"/><br /><sub><b>Kasper Holbek Jensen</b></sub></a><br /><a href="https://github.com/butlerx/WeTTy/commits?author=kholbekj" title="Documentation">📖</a></td>
    <td align="center"><a href="https://mastodon.technology/@farhan"><img src="https://avatars1.githubusercontent.com/u/10103765?v=4" width="100px;" alt="Farhan Khan"/><br /><sub><b>Farhan Khan</b></sub></a><br /><a href="https://github.com/butlerx/WeTTy/commits?author=khanzf" title="Code">💻</a></td>
    <td align="center"><a href="https://www.jurrevriesen.nl"><img src="https://avatars1.githubusercontent.com/u/7419259?v=4" width="100px;" alt="Jurre Vriesen"/><br /><sub><b>Jurre Vriesen</b></sub></a><br /><a href="https://github.com/butlerx/WeTTy/commits?author=jurruh" title="Code">💻</a></td>
    <td align="center"><a href="https://www.kartar.net/"><img src="https://avatars3.githubusercontent.com/u/4365?v=4" width="100px;" alt="James Turnbull"/><br /><sub><b>James Turnbull</b></sub></a><br /><a href="https://github.com/butlerx/WeTTy/commits?author=jamtur01" title="Code">💻</a></td>
  </tr>
</table>
<!-- markdownlint-enable -->
<!-- prettier-ignore-end -->
<!-- ALL-CONTRIBUTORS-LIST:END -->

This project follows the [all-contributors](https://github.com/all-contributors/all-contributors) specification.

<a name="license"/></a>
## License
Copyright © 2019 [Cian Butler <butlerx@notthe.cloud>](https://github.com/butlerx).<br />
This project is [MIT](https://github.com/immortalitous/wetty/blob/master/LICENSE) licensed.

---
