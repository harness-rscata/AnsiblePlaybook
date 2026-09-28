# SSH password-auth + decoy file playbook

Pairs with the `main.tf` GCP setup (instance `rss-iacm-instance-0`, ports
22/80/443/4440 open to `0.0.0.0/0`).

## Files
- `site.yml` — the playbook
- `inventory.ini.example` — copy to `inventory.ini` and fill in the instance IP

## Usage

1. Apply the terraform config and grab the public IP:
   ```bash
   tofu apply
   tofu output instance_public_ips
   ```

2. Fill in `inventory.ini` with that IP (Debian 12 image → `ansible_user=debian`,
   or whatever user your SSH key was provisioned for).

3. Run the playbook:
   ```bash
   ansible-playbook -i inventory.ini site.yml --ask-become-pass
   ```
   (`--ask-become-pass` only if your SSH user isn't already passwordless sudo.)

## What it does
- Creates a `labuser` account with a password (`ChangeMe123!` by default —
  change `lab_password` in `site.yml` or move it into `group_vars`/Ansible
  Vault before using this for anything beyond a quick test).
- Sets `PasswordAuthentication yes` and `KbdInteractiveAuthentication yes`
  in `sshd_config` so password logins work (Debian cloud images ship with
  these disabled by default).
- Writes `/opt/backup/passwords.txt`, a sample file with obviously fake
  credentials, useful as bait/decoy content or as filler data for a demo.

## Note
This intentionally weakens SSH auth. Only run it against a disposable
lab/demo instance like the one `main.tf` creates — not anything you
actually want to keep secure.
