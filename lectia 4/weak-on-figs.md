# weak-on-figs

The key point is that /home/ritabeakerlab/.bashrc was only the launcher, not the backdoor itself. In `/uac/[root]/home/ritabeakerlab/.bashrc:42` the attacker-added lines are:

```bash
# do not delete # admin #
(/tmp/.sysadmin/debian_chroot/sysrun.sh >/dev/null 2>&1 &)
```

That tells us every interactive shell for ritabeakerlab starts the real payload at `/tmp/.sysadmin/debian_chroot/sysrun.sh`. Since the challenge asks for the “initial backdoor” planted on the beachhead user, the correct answer is the payload path, not the shell startup file that invokes it.

The timeline in lesson04/uac/[root]/var/log/auth.log:116 supports this. After repeated SSH logins as ritabeakerlab, the attacker used sudo to edit .bashrc several times:

- lesson04/uac/[root]/var/log/auth.log:116 COMMAND=/usr/bin/vim .bashrc
- lesson04/uac/[root]/var/log/auth.log:150 COMMAND=/usr/bin/vim .bashrc
- lesson04/uac/[root]/var/log/auth.log:157 COMMAND=/usr/bin/vim .bashrc

Only later did the attacker edit .bash_logout, create extra users like sys-gnd, xfcenv, admin, and finally the UID 0 avahi-d account. That makes the sysrun.sh mechanism the earliest planted backdoor in the compromise chain.
