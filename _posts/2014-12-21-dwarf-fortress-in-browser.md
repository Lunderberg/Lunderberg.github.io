---
layout: post
title: Yet Another Dwarf Fortress in Browser
tags: [dwarf-fortress, linux]
---

The title says it all.
I want to be able to play Dwarf Fortress in a browser.
For maximal silliness, I also want to be able to play with other people as well.
The code itself is available from my [github account](https://github.com/Lunderberg/YADFIB).

To do so, there are four main steps.

1. Dwarf Fortress can be played in a terminal.
2. Terminals can be shared with `tmux`.
3. Terminal applications can be accessed over SSH.
4. SSH can be opened in a browser.

First, Dwarf Fortress can be played in a terminal.
This is the easy part.
In the `data/init/init.txt`, setting the option `[PRINT_MODE:TEXT]`
  will start the game in the terminal.

Next, sharing the terminal with tmux.
If a session already exists, it can be joined with `tmux attach`.
Otherwise, a session can be started with `tmux new-session path/to/df`.
This session will automatically close when the game of dwarf fortress closes.
We want to prevent the players from opening up a new bash shell,
  so we want to disable most of the keyboard shortcuts.
In addition, we want to make sure that the user doesn't connect to the wrong tmux session.
Putting this all together, the following command can be used.

```bash
tmux attach -t DwarfFortress || tmux -f tmux.conf new-session -s DwarfFortress df_linux/df
```

Since this will be open to the internet,
  we want to ensure that only the above command can be used.
This can be done using SSH keys.
In the authorized_keys file, a command can be listed to be run automatically.
This is the only command that can be run using that particular SSH key.

```
command="COMMAND",no-port-forwarding,no-X11-forwarding,no-agent-forwarding SSH_KEY
```

Finally, providing this over the browser.
[GateOne](https://github.com/liftoff/GateOne) is a Web-based SSH client,
  free for personal use.
By setting up a saved Favorite to connect through SSH using the previously generated SSH key,
  new players can quickly connect.

As a final note, whenever services are opened up to the public,
  security
Is this safe?
Possibly, possibly not.
Does this open up security holes?
Again, possibly.
I have done my best to prevent players from gaining access to anything other than Dwarf Fortress,
  but I am not an expert, and may have overlooked a means of attack.
I take no responsibility for any security holes that this may open, should you choose to use it.
Whenever I am running this, I run it on a dedicated user account,
  with limited file permissions.
