tmux_env.py
===========

When weechat runs simply in a terminal, weechat and its scripts inherit the
parent environment variables.  This allows scripts such as
[notify.py][] to access variables such as `DBUS_SESSION_BUS_ADDRESS` so
they can interact with the desktop.

The problem of running weechat directly in a terminal is that you can't
access it remotely with ssh, and it's not persistent if you log out and
back in.  Therefore it's common to run weechat inside [screen][] or
[tmux][] so you can freely disconnect and reconnect.

One advantage of this setup is that you can manage your presence indication
(awayness) with the [screen_away.py script][screen_away.py], which works
just as well for tmux as for screen.

However the setup becomes problematic when your weechat scripts rely on
environment variables.  For example [notify.py][] relies on
`DBUS_SESSION_BUS_ADDRESS` for sending notifications to the screen.  If
this variable is missing or stale, then notifications will stop working.

Tmux provides a solution to this problem by providing access to the client
environment variables, and propagating them to any new windows or panes
instantiated after the client connects.  Existing processes don't
automatically get the updates, though, and that's where `tmux_env.py` (this
script) bridges the gap by occasionally querying tmux and propagating
environment changes into the tmux process, including the script
interpreters.

What this means is that you can disconnect your ssh session and reconnect
on your desktop, and the proper values of `DBUS_SESSION_BUS_ADDRESS` will
appear in the weechat process environment, and therefore `notify.py` will
be able to provide desktop notifications.

How to use
----------

 1. Install this script and [notify.py][]

 2. Configure tmux to update `DBUS_SESSION_BUS_ADDRESS` in `.tmux.conf`

        set-option -g update-environment 'DISPLAY SSH_ASKPASS SSH_AUTH_SOCK SSH_AGENT_PID SSH_CONNECTION WINDOWID XAUTHORITY DBUS_SESSION_BUS_ADDRESS'

 3. When you disconnect tmux and connect from somewhere else, you should
    see messages like this in weechat:

        tmux_env: add DBUS_SESSION_BUS_ADDRESS=u'unix:abstract=/tmp/dbus-K7Sk2maQI7,guid=e41a4322998f0ec0b937d5ef525ae11d' (was None)
        tmux_env: add DISPLAY=u':0' (was None)
        tmux_env: add SSH_AGENT_PID=u'1707' (was None)
        tmux_env: add SSH_ASKPASS=u'/usr/bin/ksshaskpass' (was u'/usr/libexec/openssh/gnome-ssh-askpass')
        tmux_env: add SSH_AUTH_SOCK=u'/tmp/ssh-6DDLwlmL8Uvs/agent.1547' (was None)
        tmux_env: remove SSH_CONNECTION (was u'172.20.0.4 38969 172.20.0.11 22')
        tmux_env: add WINDOWID=u'73401727' (was None)
        tmux_env: add XAUTHORITY=u'/tmp/kde-aronGPPXUt/xauth-10208-_0' (was None)

[notify.py]: http://www.weechat.org/scripts/source/notify.py.html
[screen]: http://www.gnu.org/software/screen
[tmux]: http://tmux.sourceforge.net/
[screen_away.py]: http://www.weechat.org/scripts/source/screen_away.py.html
[notify.py]: http://www.weechat.org/scripts/source/notify.py.html
