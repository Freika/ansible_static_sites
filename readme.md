# What is this?

This is an automated script to help you setup new server and deploy Rails application along with database dump.

## Requirements:
Ubuntu 16.04 or newer

## Preparation

1. Execute within your rails app directory: `git clone git@github.com:Freika/rails_server.git server`
2. Create file `server/hosts` and set values as shown in an example below. User and database passwords should be [encrypted](http://docs.ansible.com/ansible/faq.html#how-do-i-generate-crypted-passwords-for-the-user-module). Don't forget to provide `yandex_disk_access_token` for backups, use this [guide](https://github.com/anjlab/yandex-disk). It requires to allow access to Yandex Disk REST API **AND** Yandex Disk WebDAB API.
3. Place you postgres database dump at `server/app_name.sql`. Dump must be created with following command: `pg_dump --no-owner app_name > app_name.sql -U DB_USERNAME`
4. Update IP address of your server in `config/deploy/production.rb` and set user value to `deploy`.
5. Install roles from ansible-galaxy:
```
ansible-galaxy install manala.git manala.zsh mashimom.oh-my-zsh pgolm.monit geerlingguy.passenger DavidWittman.redis rvm_io.rvm1-ruby ANXS.postgresql kamaln7.swapfile
```

## hosts file
Create file `hosts` in root directory of ansible script. Here is an example:

```
[rails]
IP_ADDRESS                      # Your server IP address

[rails:vars]
app_name=APP_NAME               # Your application name
app_domain=example.com          # Your app domain (for nginx configuration)
user_password=PASSWORD          # Password for user deploy, encrypted in md5

monit_enabled=true              # Enable or disable monit installation
monit_user=MONIT_USERNAME       # Username for monit
monit_password=MONIT_PASSWORD   # Password for monit, plain text
monit_allowed_ip=0.0.0.0        # Allowed IP for monit sign in

ruby_version=2.3.3              # Ruby version

restore_backup=true             # Make it false for fresh deploy
postgres_password=00000000      # Your app database password (for database.yml)

aws_key=xxxx
aws_key_secret=xxxx
aws_region=xxxx
aws_bucket_name=xxxx

slack_webhook_url=https://hooks.slack.com/services/SOME_GENERATED_VALUE

ansible_ssh_user=vagrant        # For local testing
ansible_ssh_private_key_file=~/.vagrant.d/insecure_private_key
```

## Usage

### Server

Run
```
ansible-playbook server/python.yml server/server.yml server/app.yml -i server/hosts
```

This script goes through full server configuration process. For now it does next things (in order of applying):

#### python.yml

- Installs Python2 in order to allow Ansible do all the work
- Installs pip, python packet manager

#### server.yml

- Creates `deploy` user which is our main user for application deployments
- Configures `zsh` and `oh-my-zsh` for both `root` and `deploy` users
- Installs `vim` and `htop`
- Installs `monit` for server state monitoring and notifications (RAM, HDD, etc)
- Installs `nginx` with `passenger` to serve your Rails applications
- Installs `Redis`
- Installs `Ruby`
- Installs `PostgreSQL`

#### app.yml

- Configures Nginx to server your Rails application
- Creates PostgreSQL database and user for your Rails application
- Uploads database dump and restores it on server
- Setups daily DB backups with uploading to Yandex Disk

### Application
Run `cap production deploy` to start deployment process. If it failed because if not found active support 5, just run it one more time.

### That's all!

## Deploying application to already configured server

Just run `ansible-playbook server/app.yml -i server/hosts`. You'll also want to update files from Preparation as appropriate.

## Local testing

Install Vagrant, run `vagrant up` and then provision it as if it's just your production server.

## Todo

- Setup Fail2ban
