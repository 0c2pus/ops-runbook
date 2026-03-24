# AppArmor - Application Security Profiles

AppArmor is a Linux kernel security module that restricts what programs can do - which files they can read, which network connections they can open. It works independently of standard Linux file permissions. Even if a file has permissions `777`, AppArmor can still block access.

## 1. Checking AppArmor Status
* `sudo aa-status` - Show all loaded profiles and their mode (enforce/complain).
* `sudo aa-status | grep <program>` - Check if a specific program has an AppArmor profile.

## 2. Reading Logs
* `sudo journalctl -k | grep apparmor` - Show kernel AppArmor denials.
* `sudo grep apparmor /var/log/syslog` - Find AppArmor blocks in syslog.

_When you see `permission denied` in app logs but file permissions look correct - always check AppArmor logs._

## 3. Profile Files
* Profiles are stored in `/etc/apparmor.d/`
* File name matches the program path with `/` replaced by `.` Example: `/usr/local/bin/webapp` → `usr.local.bin.webapp`

**Profile structure:**
```bash
/usr/local/bin/webapp {
  include <abstractions/base>
  network inet stream,
  /path/to/directory/ r,     # read access to directory
  /path/to/directory/* r,    # read access to all files in directory
  /path/to/file rw,          # read+write access to specific file
}
```

**Permission flags:** `r` = read, `w` = write, `x` = execute, `k` = lock

## 4. Managing Profiles
* `sudo apparmor_parser -r /etc/apparmor.d/<profile>` - Reload a profile after editing.
* `sudo aa-enforce /etc/apparmor.d/<profile>` - Set profile to enforce mode.
* `sudo aa-complain /etc/apparmor.d/<profile>` - Set to complain mode (logs but does not block).
* `sudo aa-disable /etc/apparmor.d/<profile>` - Disable a profile entirely.

## 5. Debugging Workflow
When an app fails with `permission denied` despite correct file permissions:
1. Check AppArmor logs: `sudo journalctl -k | grep apparmor`
2. Find the profile: `ls /etc/apparmor.d/ | grep <program-name>`
3. Edit the profile to add missing rules
4. Reload: `sudo apparmor_parser -r /etc/apparmor.d/<profile>`