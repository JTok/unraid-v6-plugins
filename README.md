# DocGyver's Plugins for unRAID v6

## Details

These are based on OverByrn's collection of plugins for UnRAID v6<br>
I have made changes that allow both ssh and Denyhosts to work with v6.1.  I cannot say for certain that they are best practice but they do work on two servers that I maintain.<br>

The other items will likely be removed from the list below as afaik they are already being supported with Docker containers.

* **DenyHosts**: Analyzes the sshd server log messages to determine what hosts are attempting to hack into your system. It also determines what user accounts are being targeted. It keeps track of the frequency of attempts from each host. Additionally, upon discovering a repeated attack host, the /etc/hosts.deny file is updated to prevent future break-in attempts from that host.

* **SSH**: Provides encrypted communication sessions over a computer network using the SSH protocol.  SSH is typically used to log into a remote machine and execute commands, but it also supports tunneling, forwarding TCP ports and it can transfer files using the associated SSH file transfer (SFTP) or secure copy (SCP) protocols.


## Download Links
Use the following direct links to download each plugin:

DenyHosts - (https://raw.githubusercontent.com/docgyver/unraid-v6-plugins/master/denyhosts.plg)<br>
SSH - (https://raw.githubusercontent.com/docgyver/unraid-v6-plugins/master/ssh.plg)<br>

## Installation
<p>
1. Navigate to the Plugins tab on the Unraid interface<br>
2. Click the 'install plugin' sub tab<br>
3. Either paste the LINK or navigate to the .PLG file and press install<br>
4. Go to UnRAID WebGui -> Settings -> &lt;Plugin Name&gt; and configure your initial settings<br>
</p>

***
## Specific Information for SSH Plugin:

It is important to understand the way this plug-in decides what users are allowed SSH access and there are a number of settings which work together to provide a flexible and yet secure set of login options.  Please spend a few minutes reading the details below as it will help you configure your system correctly.

### Creating & Selecting Users
The plug-in allows multiple users to have SSH access.  Use the standard **Users** page in the UnRAID WebGUI to create and maintain your users.  Alternatively, you may create users from the command line - see **Advanced User Creation** section below.

When you enter the SSH plug-in, all users are listed and you are able to choose between none, one or many. If no users are selected, user *root* will be assumed - however read below to understand how to configure for *root* access:

### Configuration Options
* `Permit Root Login`

  This option determines if user *root* is enabled for SSH.  NB. Review the other options below as they affect whether root (or other users) can login, depending on the configuration of your users.  eg. password assigned/empty etc.

* `Password Authentication`

  If this option is set to "no", then it is not possible to login using SSH unless the user has been configured with a public / private key pair. (See ***Creating Key-Pairs*** section)

* `Permit Empty Passwords`

  If this option is set to "no" but the user you wish to login with has a blank password, then SSH login will not be possible.  The exception to this is rule is when a user has been configured with a public / private key pair.

* `Ciphers`

  This option allows you to select the encryption ciphers that are allowed for SSH connections. The default setting is for this plugin is *blank*. This means that the default ciphers for the system will be used. This option was added to allow workarounds for [CVE-2023-48795](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2023-48795) while UNRAID is still using an unpatched version of OpenSSH.
  This is entirely optional, and whether or not you want to change this setting depends on your threat model.
  My recommendation is to set this to `aes128-gcm@openssh.com,aes256-gcm@openssh.com` to be certain that you are not using any of the ciphers that are vulnerable to this exploit.
  However, you could also just set it to `-chacha20-poly1305@openssh.com` (be certain to include the '-' at the beginning of the string). You will need to make sure you are also not enabling any `aes(128|192|256)-cbc` ciphers while using the default MACs. See [this link](https://jfrog.com/blog/ssh-protocol-flaw-terrapin-attack-cve-2023-48795-all-you-need-to-know/) for more details.

>**Scenario 1:** you require SSH access for *root*.  *root* already has a password...
>- Enable option `PermitRootUser`
>- Enable option `Password Authentication`

>**Scenario 2:** you require SSH access for *root*.  *root* does **not** have a password...
>- Enable option `PermitRootUser`
>- Enable option `Password Authentication`
>- Enable option `Permit Empty Passwords`
>- or
>- Assign a private / public key-pair (See ***Creating Key-Pairs*** section)

>**Scenario 3:** you require SSH access for standard user.  User already has a password...
>- Select the user from the selection list
>- Enable option `Password Authentication`

### Advanced User Creation
When a user is created from within the UnRAID WebGUI, it is assigned a user id (UID) of 1000 or greater.  When you create a user from outside of UnRAID at command line level, you must ensure your user is created with a suitable UID.  The plug-in assumes if a user has a UID >= 1000, the user is *not* a system user as is therefore presented as an option in the plug-in.  You can check the UID settings on your UnRAID system by viewing /etc/login.defs:
>\#<br>
>\# Min/max values for automatic uid selection in useradd<br>
>\#<br>
>UID_MIN                  1000<br>
>UID_MAX                 60000<br>
>\# System accounts<br>
>SYS_UID_MIN               101<br>
>SYS_UID_MAX               999<br>

Create users from command line using the following;
<pre><code>useradd -m -g users -s /bin/bash someuser</pre></code>
The above command creates user "someuser", including home dir and valid shell.

### Creating Key-Pairs
You may have your users authenticate with a public / private key-pair.  This allows you to remove password authentication altogether if so desired.  Key-pairs provide an additional level of security, especially when assigned a passphrase.

The plug-in has been designed to check for the existance of a public key for each user set-up for SSH (inc. root).  Perform the following steps to configure a user for a key-pair.  We will use the example user `fred`:

<p>
1. In UnRAID **Users** page; create a new user (or do it from command line)<br>
2. Go to OpenSSH settings page and select that user for SSH.  Hit Apply and/or Start SSH<br>
3. Browse to the plug-in directory on your flash drive : /boot/config/plugins/ssh
- There should now be a sub-directory called `fred`
- Inside this directory is another called `.ssh`
- Open the **read_me.txt** file which you will find in this directory. This file contains full instructions on how to create a key-pair and configure for the plug-in.  It also contains steps on how to convert the OpenSSH private key into a Putty compatible format.  (all utilities needed are supplied as part of the plug-in installation). A master copy of *read_me.txt* is also stored in the root plugin directory (/boot/config/plugins/ssh).

***
## Support
Please provide feedback to any problems encountered or to request enhancements or missing features that you would like to see added.

Support requests should be made in the UnRAID forum. Please use the following thread for all support queries;
(http://lime-technology.com/forum/index.php?topic=47289.0)<br>


DocGyver..
