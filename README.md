*Forked from [utknoxville/openconnect-pulse-gui](https://github.com/utknoxville/openconnect-pulse-gui)*.

Status: **WIP**

# openconnect-pulse-gui

This script provides a browser login through a WebKitGTK2 window. The authentication cookie is printed to the console so that it can be used to call [OpenConnect](https://www.infradead.org/openconnect/) with parameter `--cookie=<AUTH-COOKIE>` afterwards. This allows OpenConnect to be compatible with web-based authentication mechanisms, such as SAML.

In contrast to the original project the browser can run as regular user account. Only `openconnect` has to called with `sudo` as user `root` to be able to add the tunneling network device. If `openconnect` is started with parameter `--setuid=${USER}` it will drop the root privileges to your user account after the connection has been established.


## Requirements

The script can be used with python2 or python3, however python3 is recommended.  The following packages are also required:

 - python-gi or python-gobject
 - webkit2gtk
 - openconnect

Instruction for specific distros can be found below.


### Debian/Ubuntu

    sudo apt install python3-gi gir1.2-webkit2-4.0 openconnect


### Fedora

    sudo dnf install python3-gobject webkit2gtk4.1 openconnect


### Arch

    sudo pacman -S python-gobject webkit2gtk openconnect


## Installation

**Version A**

This repo can be downloaded with `git clone https://github.com/hardcodes/openconnect-pulse-gui-cookie.git` or via the GitHub webpage.

- Copy the `openconnect_pulse_gui/openconnect_pulse_gui.py` script to a directory of your liking (it should be in your `$PATH` if you want to call it from your shell).

**Version B**

- Copy [`openconnect_pulse_gui.py`](https://raw.githubusercontent.com/hardcodes/openconnect-pulse-gui-cookie/refs/heads/master/openconnect_pulse_gui/openconnect_pulse_gui.py) to a directory of your liking (it should be in your `$PATH` if you want to call it from your shell).


## Usage

Simply call the script with the sign-in link / server URL as only required argument.

```bash
python3 openconnect_pulse_gui.py <URL>
```

Other arguments can be found by using

```bash
python3 openconnect-pulse-gui.py -h
```

Note that this script will **not** run openconnect, it will only print the command with the correct arguments to stdout.

If you want to capture the output in a variable before invoking `openconnect`, you can call it this way:

```bash
SSO_LOGIN_COOKIE=$(python3 openconnect_pulse_gui.py <URL>)
sudo -p "sudo (openconnect): " openconnect --background --protocol=nc --user=<VPN user account> --setuid=${USER} --cookie="${SSO_LOGIN_COOKIE}" <URL>
```


# Wrapper script

You can use the wrapper script `wrapper-script/vpn-wrapper-script` for convenience.

- Copy the file [`vpn-wrapper-script`](https://raw.githubusercontent.com/hardcodes/openconnect-pulse-gui-cookie/refs/heads/master/wrapper-script/vpn-wrapper-script) to a directory of your liking and make sure it's in the `$PATH`.
- Edit the file and
  - enter `VPN_USER` if you connect with another user name then your Linux account.
  - enter `VPN_URL` for your VPN connection,
  - set the full path for `PULSE_GUI_SCRIPT`.
- Make the file executable, e.g. `chmod 750 <your chosen directory>/vpn-wrapper-script`.


## Wrapper script - usage

Call

```bash
vpn-wrapper-script
```

The script will

- start the `openconnect_pulse_gui.py` with the URL you entered in the `VPN_URL` variable and capture the cookie in the variable `SSO_LOGIN_COOKIE`,
- invoke `openconnect` via `sudo` with the captured cookie for authentication, drop the `root` privileges afterwards and write the process id into the file `PID_FILE`.

When called again, the script will send `SIGINT` to the running `openconnect` process and disconnect.


# Login process

Anybody wishing to recreate this functionality either manually or using another library can with the following steps:

1. Send the user to the sign-in URL.  This will either give them the ability to log in directly or redirect them to an external authentication server.
1. Wait for a `Set-Cookie` header that contains the `DSID` cookie.  This is the authentication cookie used by Pulse Secure.
1. Pass the cookie to `openconnect` using `--protocol nc` and `-C 'DSID=<cookie-value>'`.  Note that some workflows may work with `--protocol pulse`, but at this time SAML-based logins do not.


