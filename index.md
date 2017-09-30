# How to setup this website

## What's your website's name?

Firstly you have to decide what's your website _name_ (domain) something like _www.yourwebsite.com_. It's your child and you should name it properly, so head to [www.namecheap.com]() and buy yourself a brand new domain.

I choose [www.jrodrigues.co]() and bought it setting auto-renew (TODO why? How domain expiration works?) and WhoisGuard (TODO why? What does it do? What happens if not set?) also with auto-renew.

Namecheap also has a freeDNS service (there are other free options as well like such and such (TODO which ones?)) that will be handy later on.

Alternatives:

 * gandi.net
 * some give-free-domain website? (TODO research)



## Give it a house!

We are going to serve the website using the [Google Cloud Platform](https://cloud.google.com/)

Setting up a basic VM instance (f1-micro) (TODO comment about always free plan, but try to use it first).

#### Create a VM
* name (name your website's home machine)
* zone (some near you?)
* machine f1-micro (a lightweight to begin with)
* boot disk (Debian GNU/Linux 9 (strech))
* allow http
* allow https

#### Change to static IP
* change the IP to a static one on Virtual Private Cloud (VPC) settings > External IP Addresses ([link](https://console.cloud.google.com/networking/addresses))
  * On "type" column change to "static" ([google how-to link](https://cloud.google.com/compute/docs/ip-addresses/reserve-static-external-ip-address#promote_ephemeral_ip))

#### Setup the VM
``` bash
# type the following commands (one line at a time)

# install apache2
sudo su
apt-get update
apt-get install apache2
# agree to install [y]

# edit index.html file
vim /var/www/html/index.html
```

TODO Make basic `vim` tutorial or change every `vim` to `code` and make a tutorial installing VSCode and it's cli `code`.

#### Glue the VM to the domain
* Create an A resource pointing to the static IP address


## List of references:

General basic info

 * Build your first website on the Google Cloud platform ([video on youtube](https://www.youtube.com/watch?v=KzJxwu2poIc))

DNS setup

 * Difference between A and CNAME ([DNSimple article](https://support.dnsimple.com/articles/differences-a-cname-records/))
 * Setting up a host (resource) record ([Namecheap article](https://www.namecheap.com/support/knowledgebase/article.aspx/434/10/how-do-i-set-up-host-records-for-a-domain))
 * Pseudo-precedence of CNAME over A records ([Quora answer](https://www.quora.com/Why-do-canonical-name-CNAME-records-take-precedence-over-address-A-records))
 * Resource Record (RR) types (e.g. A, CNAME, AAAA, ...) ([Namecheap article](https://www.namecheap.com/support/knowledgebase/article.aspx/579/2237/which-record-type-option-should-i-choose-for-the-information-im-about-to-enter))

\
\
\
\
\
\
\ 
# How to comunicate with the machine via SSH

#### Creating a ssh key pair
[reference](https://cloud.google.com/compute/docs/instances/connecting-to-instance?hl=pt-br)
``` bash
# Generate a rsa key pair
# [USERNAME] can be any value, but it's important in next steps
ssh-keygen -t rsa -f ~/.ssh/my-ssh-key -C [USERNAME]

# Make your private key only accessible by you and only readable
chmod 400 ~/.ssh/my-ssh-key

# Copy public key value
cat ~/.ssh/my-ssh-key.pub | pbcopy # (on a mac)
```
> `-t` stands for cryptography key (TODO is there another name for this?) `type`, we are using `rsa` (TODO link) but there are others like `rsa1`, `dsa`, `ecdsa`, `ed25519`, ...
> 
> `-f` stands for `file` and specifies the name and location of both files to be created (private and public key)
>
> `-C` stands for `Comment` and add a comment at the end of the file. Google identifies a word on the end of the file as a username associated with that key (TODO is it google or everyone?)

Paste public key value: on `Google Compute Engine > Metadata > SSH Keys`, click `edit` and `add item` ([link](https://console.cloud.google.com/compute/metadata/sshKeys))

On saving, that `[USERNAME]` will show to the left of the added key. This is the username associated to this key, and the instance will now accept incoming connection from this user with the respective private key.

So now you can connect typing
``` bash
ssh -i ~/.ssh/my-ssh-key [USERNAME]@<instance-ip-address>
```
> `-i` stands for `IdentityFile` that is, specifies a file that will securely identify the user `[USERNAME]`. In this case we are using rsa so the private key can identify the user when coupled with the public key, that we already deposit on the instance (via google web interface)

#### Making `ssh` easier

We can create a shorthand to the command above to make it easier to connect to our instance. For this we have to edit (or create) the file `~/.ssh/config`.

``` bash
# Open/create the file like so
vim ~/.ssh/config
```

Add the following lines
``` bash
Host <our-shorthand-name>
  HostName <instance-ip-address>
  User [USERNAME]
  IdentityFile ~/.ssh/my-ssh-key
```

Save and quit with `:wq`

Now you can access the instance via ssh with
``` bash
ssh <our-shorthand-name>
```

could be something like
``` bash
ssh gcloud
```

much nicer :)


# How to transform markdown to html (github like)
using `grip` ([link to stackoverflow answer](https://stackoverflow.com/questions/7694887/is-there-a-command-line-utility-for-rendering-github-flavored-markdown))
``` bash
pip install grip
```
TODO how to install pip

Using grip:
``` bash
grip --export <input.md> <output.html>
```

and add this line to the bottom o the `<style>` tag
``` css
    a[aria-hidden="true"] > span { display: none; }
```

# How to install git on debian
``` bash
sudo apt-get update
sudo apt-get install git-core
```

setup user name and email

``` bash
git config --global user.name "Your Name"
git config --global user.email "your@email.com"
```

# managing apache/nginx process on debian
This would be [different](https://www.cyberciti.biz/faq/star-stop-restart-apache2-webserver/) for `systemd` users
``` bash
# list all services available (TODO does it?)
ls /etc/init.d/

# check apache status
sudo /etc/init.d/apache2 status

# start apache
sudo /etc/init.d/apache2 start

# stop apache
sudo /etc/init.d/apache2 stop
```

# removing apache
``` bash
# removes apache2 folder only
sudo apt-get remove apache2

# remove apache2 remaining dependencies
sudo apt-get autoremove
```

# installing nginx
``` bash
sudo apt-get install nginx
```

# owner, group and permissions
Why is it important? How does it work?
``` bash
# TODO
chown owner_name:group_name file_name

# 1 - execute
# 2 - write
# 4 - read
# + = 0, 1, 2, 3, 4, 5, 6 or 7
#
# ___
# _    owner
#  _   group
#   _  others

chmod 644 file_name

# owner can read and write, group can read, others can read
```

