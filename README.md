# Ansible Playbook for gitlab omnibus, gitlab-ci, and dokku

## Usage

### Add /etc/hosts entries

Add following lines to `/etc/hosts`:

    10.0.0.10 gitlab.example.test gitlab0 registry.example.test mattermost.example.test
    10.0.0.11 gitlab-runner1.example.test gitlab-runner1
    10.0.0.12 gitlab-runner2.example.test gitlab-runner2
    10.0.0.9 dokku0.example.test dokku0

### Initialize

    $ git clone https://github.com/znz/ansible-playbook-gitlab-dokku
    $ cd ansible-playbook-gitlab-dokku
    $ vagrant up

- Open `http://gitlab.example.test/`
- Set password of root user
- Log in as root with your password
- Open `http://gitlab.example.test/admin/runners`
- Copy 'Registration token'
- `export GITLAB_RUNNER_REGISTRATION_TOKEN=${copied registration token}`
- `vagrant provision`
- Open `http://mattermost.example.test/` for mattermost

### Unregister runner

- Login gitlab-runnerX
- Run `sudo gitlab-runner list`
- Copy URL and Token
- Run `sudo gitlab-runner unregister -u $URL -t $TOKEN` with replacing `$URL` and `$TOKEN` with copied URL and Token

### Setting up Mattermost as a Slack project service integration

see http://docs.gitlab.com/omnibus/gitlab-mattermost/#setting-up-mattermost-as-a-slack-project-service-integration

## Production Usage

### letsencrypt

This step cannot do without valid domains.
You must replace valid domains.

Run following commands on gitlab0.

- `sudo certbot certonly --webroot --webroot-path=/var/www/letsencrypt -d gitlab.example.test`
- `sudo certbot certonly --webroot --webroot-path=/var/www/letsencrypt -d registry.example.test`
- `sudo certbot certonly --webroot --webroot-path=/var/www/letsencrypt -d mattermost.example.test`
- `sudo gitlab-ctl reconfigure`

### Register runner

- Login as admin to gitlab
- Open Admin Area (by clicking the wrench icon in the upper right corner)
- Open Runners tab in Overview tab
- Copy Registration token
- Run `ansible-playbook -i provision/inventory/production -K -e gitlab_runner_registration_token=$GITLAB_RUNNER_REGISTRATION_TOKEN provision/playbook.yml` with replacing `$GITLAB_RUNNER_REGISTRATION_TOKEN` with copied registration token

### Setup SMTP

Create `/etc/gitlab/local.rb` (eval in last of my `/etc/gitlab/gitlab.rb`).

For example:

    gitlab_rails['smtp_address'] = 'smtp.example.com'
    gitlab_rails['smtp_user_name'] = 'username'
    gitlab_rails['smtp_password'] = 'password'
    mattermost['email_smtp_username'] = 'username'
    mattermost['email_smtp_password'] = 'password'
    mattermost['email_smtp_server'] = 'smtp.example.com'
    mattermost['email_feedback_email'] = 'email@example.com'
    mattermost['support_email'] =  'support@example.com'

See https://docs.gitlab.com/omnibus/settings/smtp.html for more examples.
