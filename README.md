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
- Run `ansible-playbook -i provisioning/inventory/production -K -e gitlab_runner_registration_token=$GITLAB_RUNNER_REGISTRATION_TOKEN provisioning/playbook.yml` with replacing `$GITLAB_RUNNER_REGISTRATION_TOKEN` with copied registration token
