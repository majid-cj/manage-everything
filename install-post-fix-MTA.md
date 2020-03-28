# How To Install and Configure Postfix on Ubuntu

1 — Install Postfix

    $ sudo apt update
    $ sudo DEBIAN_PRIORITY=low apt install postfix

**Use the following information to fill in your prompts correctly for your environment:**

    $ General type of mail configuration => Internet Site
    $ System mail name => your-domain.com
    $ Root and postmaster mail recipient => your linux user name but not root
    $ Other destinations to accept mail for => the default should work fine you
    $ Force synchronous updates on mail queue => No
    $ Local networks => The default should work for most scenarios
    $ Mailbox size limit => set it to 0 , it disables any size restriction
    $ Local address extension character => +
    $ Internet protocols to use => all

**now your postfix is installed and it will show your configurations**

# If you need to ever return to re-adjust these settings, you can do so by typing:

    $ sudo dpkg-reconfigure postfix

2 — Tweak the Postfix Configuration

    $ sudo postconf -e 'home_mailbox= Maildir/'
    $ sudo postconf -e 'virtual_alias_maps= hash:/etc/postfix/virtual'

3 — Map Mail Addresses to Linux Accounts

    $ sudo nano /etc/postfix/virtual

    <anything>@your-domain.com <linux_user_name_but_not_root>
    <anything>@your-domain.com <linux_user_name_but_not_root>

    eg:

    info@your-domain.com user_name

    # save and exit

**you can add more mails by appending them to this file**
**restart postfix service**

    $ sudo systemctl restart postfix


4 — Adjust the Firewall

    $ sudo ufw allow Postfix

5 — Setting up the Environment to Match the Mail Location

    $ echo 'export MAIL=~/Maildir' | sudo tee -a /etc/bash.bashrc | sudo tee -a /etc/profile.d/mail.sh

    $ source /etc/profile.d/mail.sh

6 — Install and Configure the Mail Client

    $ sudo apt install s-nail

    $ sudo nano /etc/s-nail.rc

**append the following lines at the bottom of this files**

    . . .
    set emptystart
    set folder=Maildir
    set record=+sent

7 — Initialize the Maildir and Test the Client

    $ echo 'init' | s-nail -s 'init' -Snorecord <linux_user_name_but_not_root>

**check to make sure the directory was created by looking for our ~/Maildir directory:**

    $ ls -R ~/Maildir

    the output will be something like


    /home/<linux_user_name_but_not_root>/Maildir/:
    cur  new  tmp

    /home/<linux_user_name_but_not_root>/Maildir/cur:

    /home/<linux_user_name_but_not_root>/Maildir/new:
    1463177269.Vfd01I40e4dM691221.mail.your-domain.com

    /home/<linux_user_name_but_not_root>/Maildir/tmp:

# if your reached here without any errors then your client is setup and your can manage client using

    $ s-nail

    to exit type

    ? q

    and hit ENTER

# check that everything is working fine

    $ cat "message body" | s-nail -s "message subject" -r info@your-domain.com yourmail@mail.com

    send mail to multiple account

    $ $ cat "message body" | s-nail -s "message subject" -r info@your-domain.com mail1@mail.com, mail2@mail.com

