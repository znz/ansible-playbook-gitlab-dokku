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
