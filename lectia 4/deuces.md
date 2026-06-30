# deuces

The clue is in the attacker-edited logout configuration for the beachhead user. In `/uac/[root]/home/ritabeakerlab/.bash_logout:6`, the attacker added a cleanup routine that runs every time the user logs out:

```
# ~/.bash_logout: executed by bash(1) when login shell exits.
# when leaving the console clear the screen to increase privacy
if [ "$SHLVL" = 1 ]; then
    [ -x /usr/bin/clear_console ] && /usr/bin/clear_console -q
fi
# app cleanup routine executed at each logout
rm -rf /tmp/khsolutions-app-cache-$USER
losetup -d /dev/loop0p4
losetup -d /dev/loop1
losetup -d /dev/loop2p4
losetup -d /dev/loop2
losetup -d /dev/loop3p4
# ~/.bash_logout
```

That line directly answers the question. The challenge asks what file the adversary consistently deletes to clean up her tracks, and the logout hook shows the exact target path: /tmp/khsolutions-app-cache-$USER.

The timeline in `/uac/[root]/var/log/auth.log:172` confirms this was deliberate attacker behavior. The adversary edited .bash_logout twice with sudo vim:

- lesson04/uac/[root]/var/log/auth.log:172 COMMAND=/usr/bin/vim .bash_logout
- lesson04/uac/[root]/var/log/auth.log:191 COMMAND=/usr/bin/vim .bash_logout

That is strong evidence the cleanup logic was intentionally planted. Because .bash_logout executes on logout, the deletion happens consistently whenever the attacker disconnects, which matches the wording of the challenge.
