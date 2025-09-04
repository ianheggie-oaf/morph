[![CI](https://github.com/openaustralia/morph/actions/workflows/ruby.yml/badge.svg)](https://github.com/openaustralia/morph/actions/workflows/ruby.yml)
[![Maintainability](https://qlty.sh/gh/openaustralia/projects/morph/maintainability.png)](https://qlty.sh/gh/openaustralia/projects/morph)

# morph.io: A scraping platform

* A [Heroku](https://www.heroku.com/) lookalike system for [Scrapers](https://en.wikipedia.org/wiki/Web_scraping)
* All code and collaboration through [GitHub](https://github.com/)
* Write your scrapers in Ruby, Python, PHP, Perl or JavaScript (NodeJS, PhantomJS)
* Simple API to grab data
* Schedule scrapers or run manually
* Process isolation via [Docker](https://www.docker.com/)
* Email alerts for broken scrapers

## Hardware requirements

A system capable of running a supported version of Ubuntu LTS, macOS/X or MS Windows is required. 
The Virtual machines / docker containers take around 4 GB, so 8 GB of memory is probably the minimum,
preferably 16 Gb if you use an IDE with all the bells and whistles.

Application Code Development Environment
----------------------------------------

### Requirements

Docker compose is used to provide a consistent development environment.

Install either a supported version of [Docker Engine](https://docs.docker.com/engine/install/)
directly or as part of [Docker Desktop](https://docs.docker.com/desktop/)
develop the Ruby on Rails application code base.

* Docker compose is used to run up containers for
    * elasticsearch
    * mitmproxy,
    * MySQL,
    * Redis,
    * Ruby, SQLite 3 on main development container,
 
  (See below for more details about installing Docker)

### Installing Docker

#### On Linux

Install [Docker engine](https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/) for Ubuntu.

Your user account should be able to manipulate Docker (just add your user to the `docker` group).

#### On Mac OS X

Install [Docker Desktop for Mac](https://docs.docker.com/docker-for-mac/install/).

### Starting docker containers

Morph needs various services to run. We've made things easier for development by using docker
to run Elasticsearch and the other services as well as the web container for the ruby on rails application itself.

    docker compose up

## Configuring morph development environment

    bundle install
    cp config/database.yml.example config/database.yml
    cp env-example .env

Edit `config/database.yml` with your database settings and `.env` with your local environment

Infrastructure as Code Development Environment
----------------------------------------------

### Requirements

Vagrant is used with the virtualbox provider to provide a local staging environment to develop
the ansible playbooks and roles used to provision the servers.

Install [Virtualbox](https://www.virtualbox.org/wiki/Downloads) as a provider

Install [Vagrant](https://developer.hashicorp.com/vagrant/install)

Install a Tool version manager, for example [mise](https://mise.jdx.dev/getting-started.html) to manage tool versions

Install python using your tool manager: `mise install python`

Check `python --version` matches the contents of `.python-version`

Install Ansible using `pip install -r provisioning/requirements.txt`

Install ansible roles as per the comment on `provisioning/requirements.yml` - note these roles are ignored (as per `.gitignore`)
and are thus **not** committed to the git repo
```
cd provisioning 
ansible-galaxy install -r requirements.yml -p roles
```

If you are using `mise` then ensure the idomatic version files for ruby (`.ruby-version`) and python (`.python-version`) are enabled.
Check `~/.config/mise/config.toml` contains:
```
[settings]
idiomatic_version_file_enable_tools = ["ruby","python"]
```

Ensure you are running the python version specified in `.python-version` using `python --version`

Install the required ansbile version using pip:
```bash
pip install -r provisioning/requirements.txt
```

Checkout the infrastructure repo as a sibling directory and generate ssl certificates as per its README.md.
This will move a self signed ssl certificate into this project for its use in development.

Repositories
------------

User-facing:

* [openaustralia/morph](https://github.com/openaustralia/morph) - Main application
* [openaustralia/morph-cli](https://github.com/openaustralia/morph-cli) - Command-line morph.io tool
* [openaustralia/scraperwiki-python](https://github.com/openaustralia/scraperwiki-python) - Fork of [scraperwiki/scraperwiki-python](https://github.com/scraperwiki/scraperwiki-python) updated to use morph.io naming conventions
* [openaustralia/scraperwiki-ruby](https://github.com/openaustralia/scraperwiki-ruby) - Fork of [scraperwiki/scraperwiki-ruby](https://github.com/scraperwiki/scraperwiki-ruby) updated to use morph.io naming conventions

Docker images:
* [openaustralia/buildstep](https://github.com/openaustralia/buildstep) - Base image for running scrapers in containers

### Tunnel GitHub webhook traffic back to your local development machine

We use "ngrok" a tool that makes tunnelling internet traffic to a local development machine easy. 
First [download ngrok](https://ngrok.com/download) if you don't have it already. Then,

    ngrok http 5100

Make note of the `http://*.ngrok.io` forwarding URL.

<!-- TODO: Add instructions for debugging and working with callbacks for the GitHub app in development with https://webhook.site -->

### Creating Github Application

You'll need to create an application on GitHub So that morph.io can talk to GitHub. 
We've pre-filled most of the important fields for a few different configurations below:

* [Create GitHub application on your personal account for use in development](https://github.com/settings/apps/new?name=Morph.io+(development)&description=Get+structured+data+out+of+the+web&url=http://127.0.0.1:5100&callback_urls[]=http://127.0.0.1:5100/users/auth/github/callback&setup_url=http://127.0.0.1:5100&setup_on_update=true&public=true&webhook_active=false&webhook_url=http://127.0.0.1:5100/github/webhook&administration=write&contents=write&emails=read)
* [Create GitHub application on your personal account for use in production](https://github.com/settings/apps/new?name=Morph.io&description=Get+structured+data+out+of+the+web&url=https://morph.io&callback_urls[]=https://morph.io/users/auth/github/callback&setup_url=https://morph.io&setup_on_update=true&public=true&webhook_active=false&webhook_url=https://morph.io/github/webhook&administration=write&contents=write&emails=read)
* [Create GitHub application on the openaustralia organization for use in production](https://github.com/organizations/openaustralia/settings/apps/new?name=Morph.io&description=Get+structured+data+out+of+the+web&url=https://morph.io&callback_urls[]=https://morph.io/users/auth/github/callback&setup_url=https://morph.io&setup_on_update=true&public=true&webhook_active=false&webhook_url=https://morph.io/github/webhook&administration=write&contents=write&emails=read)

You will need to add and change a few values manually:
* Disable "Expire user authorization tokens"
* Add an image - you can use the standard logo at `app/assets/images/logo.png` (you can add this after the app is created)
* If the webhooks are active and being used in production (currently not the case) then
  you'll also need to add a "Webhook secret" for security.

Next you'll need to fill in some values in the `.env` file which come from the GitHub App that you've just created.

* `GITHUB_APP_ID` - Look for "App ID" near the top of the page. This should be an integer
* `GITHUB_APP_NAME` - Look for "Public link". The name is what appears after "https://github.com/apps/". 
  It's essentially a url happy version of the name you gave the app.
* `GITHUB_APP_CLIENT_ID` - Look for "Client ID" near the top of the page.
* `GITHUB_APP_CLIENT_SECRET` - Go to "Generate a new client secret".

Also, a private key for the GitHub app is needed. 
This can be generated by clicking the "Generate a private key" button and will be automatically downloaded. 
Move and rename it to `config/morph-github-app.private-key.pem`.

Now setup the databases:

    bundle exec dotenv rake db:setup

Now you can start the server

    bundle exec dotenv foreman start

and point your browser at [http://127.0.0.1:3000](http://127.0.0.1:3000)

To get started, log in with GitHub. There is a simple admin interface
accessible at [http://127.0.0.1:3000/admin](http://127.0.0.1:3000/admin). To
access this, run the following to give your account admin rights:

    bundle exec rake app:promote_to_admin

## Running tests

If you're running guard (see above) the tests will also automatically run when you change a file.

By default, RSpec will skip tests that have been tagged as being slow. 
To change this behaviour, add the following to your `.env`:

    RUN_SLOW_TESTS=1

By default, RSpec will run certain tests against a running Docker server. 
These tests are quite slow, but not have been tagged as slow. 
To stop Rspec from running these tests, add the following to your `.env`:

    DONT_RUN_DOCKER_TESTS=1

### Guard Livereload

We use Guard and Livereload so that whenever you edit a view in development the web page gets automatically reloaded. 
It's a massive time saver when you're doing design or lots of work in the view. To make it work run

    bundle exec guard

Guard will also run tests when needed. Some tests do integration tests against a
running docker server. These particular tests are very slow. If you want to
disable them,

```
DONT_RUN_DOCKER_TESTS=1 bundle exec guard
```

### Mail in development

By default in development mails are sent to [Mailcatcher](http://mailcatcher.me/). To install

    gem install mailcatcher

## Deploying to production

This section will not be relevant to most people. It will however be relevant if you're deploying to a production server.

### Ansible Vault

We're using [Ansible Vault](https://docs.ansible.com/ansible/2.4/vault.html) to encrypt certain files, like the private key for the SSL certificate.

To make this work you will need to put the password in a file at `../.infrastructure_ansible_vault_pass.txt`.
This is the same password as used in the [openaustralia/infrastructure](https://github.com/openaustralia/infrastructure) GitHub repository.

## Restarting Discourse

Discourse runs in a container and should usually be restarted automatically by docker.

However, if the container goes away for some reason, it can be restarted:

```
root@morph:/var/discourse# ./launcher rebuild app
```

This will pull down the latest docker image, rebuild, and restart the container.

## Production devops development

> This method defaults to creating a 4Gb VirtualBox VM, which can strain an 8Gb Mac. 
> We suggest tweaking the Vagrantfile to restrict ram usage to 2Gb at first, or using a machine with at least 12Gb ram.

Install [Vagrant](http://www.vagrantup.com/), [VirtualBox](https://www.virtualbox.org) and [Ansible](http://www.ansible.com/).

Install a couple of Vagrant plugins: `vagrant plugin install vagrant-hostsupdater vagrant-disksize`

Install a Ruby Version Manager, for example (from latest to oldest):
- [mise](https://mise.jdx.dev/) - modern, polyglot and fast (includes language installer)
- [chruby](https://github.com/postmodern/chruby) and [ruby-install](https://github.com/postmodern/ruby-install) - a lightweight alternative
- [rbenv](https://github.com/rbenv/rbenv) and [ruby-build](https://github.com/rbenv/ruby-build#readme) - the leader between 2015 and 2020
- [rvm](https://rvm.io/) - used on production and staging, the old faithful and well known ruby version manager

If on Ubuntu, install libreadline-dev: `sudo apt install libreadline-dev libsqlite3-dev`

Install the required ruby version: `rbenv install`

Install capistrano: `gem install capistrano`

Run `make roles` to install some required ansible roles.

Run `vagrant up local`. This will build and provision a box that looks and acts like production at `dev.morph.io`.

## Production devops development

> This method defaults to creating a 4Gb VirtualBox VM, which can strain an 8Gb Mac. 
> We suggest tweaking the Vagrantfile to restrict ram usage to 2Gb at first, or using a machine with at least 12Gb ram.

Install [Vagrant](http://www.vagrantup.com/), [VirtualBox](https://www.virtualbox.org) and [Ansible](http://www.ansible.com/).

Install a couple of Vagrant plugins: `vagrant plugin install vagrant-hostsupdater vagrant-disksize`

Install a Ruby Version Manager, for example (from latest to oldest): 
- [mise](https://mise.jdx.dev/) - modern, polyglot and fast (includes language installer)
- [chruby](https://github.com/postmodern/chruby) and [ruby-install](https://github.com/postmodern/ruby-install) - a lightweight alternative
- [rbenv](https://github.com/rbenv/rbenv) and [ruby-build](https://github.com/rbenv/ruby-build#readme) - the leader between 2015 and 2020
- [rvm](https://rvm.io/) - used on production and staging, the old faithful and well known ruby version manager

If on Ubuntu, install libreadline-dev: `sudo apt install libreadline-dev libsqlite3-dev`

Install the required ruby version: `rbenv install`

Install capistrano: `gem install capistrano`

Run `make roles` to install some required ansible roles.

Run `vagrant up local`. This will build and provision a box that looks and acts like production at `dev.morph.io`.

Once the box is created and provisioned, deploy the application to your Vagrant box:

    cap local deploy

Now visit https://dev.morph.io/

## Production provisioning and deployment

To deploy morph.io to production, normally you'll just want to deploy using Capistrano:

    cap production deploy

When you've changed the Ansible playbooks to modify the infrastructure you'll want to run:

    make ansible

## SSL certificates

We're using Let's Encrypt for SSL certificates. It's not 100% automated.
On a completely fresh install (with a new domain) as root:
```
certbot --nginx certonly -m contact@oaf.org.au --agree-tos
```

It should show something like this:
```
Which names would you like to activate HTTPS for?
-------------------------------------------------------------------------------
1: morph.io
2: api.morph.io
3: faye.morph.io
4: help.morph.io
```

Leave your answer your blank which will install the certificate for all of them

### Installing certificates for local vagrant build

    sudo certbot certonly --manual -d dev.morph.io --preferred-challenges dns -d api.dev.morph.io -d faye.dev.morph.io -d help.dev.morph.io

### Scraper<->mitmdump SSL

Scrapers talk out to the internet by being routed through the mitmdump2
proxy container. The default container you'll get on a devops install
has no SSL certificates. This makes it easy for traffic to get out,
but means we can't replicate some problems that occur when the SSL
validation fails.

To work around this, you'll have to rebuild the mitmdump container. Look in `/var/www/current/docker_images/morph-mitmdump`; 
there's a `Makefile` that will aid in building the new image.

Once that's done, you'll need to build a new version of the `openaustralia/buildstep`:

```bash
cd
git clone https://github.com/openaustralia/buildstep.git`
cd buildstep
cp /var/www/current/docker_images/morph-mitmdump/mitmproxy/mitmproxy-ca-cert.pem .
ln -s Dockerfile.heroku-24  Dockerfile
docker image build -t openaustralia/buildstep:latest .
```

You should now be able to see in `docker image list --all` that your new image is ready. 
The next time you run a scraper it will be rebuilt using the new buildstep image.

# How to contribute

If you find what looks like a bug:

* Check the [GitHub issue tracker](http://github.com/openaustralia/morph/issues/)
  to see if anyone else has reported issue.
* If you don't see anything, create an issue with information on how to reproduce it.

If you want to contribute an enhancement or a fix:

* Fork the project on GitHub.
* Make your changes with tests.
* Commit the changes without making changes to any files that aren't related to your enhancement or fix.
* Send a pull request.

We maintain a list of [issues that are easy fixes](https://github.com/openaustralia/morph/issues?labels=easy+fix&milestone=&page=1&state=open). 
Fixing one of these is a great way to get started while you get familiar with the codebase.

# Copyright & License

Copyright OpenAustralia Foundation Limited. Licensed under the Affero GPL. See LICENSE file for more details.
