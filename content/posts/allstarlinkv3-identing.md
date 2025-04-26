+++

title = "Fixing Repeater IDing on AllStarLink 3 (the Right Way?)"
description = "Why the default settings don’t cut it—A workaround for AllStarLink nodes that don't ID without on-going transmissions and preventing it double IDing, using a custom cron job and custom Bash script."
date = "2025-04-26"
preview = ""
draft = "false"
tags = [ "AllStar", "AllStarLink", "Amateur Radio", "Asterisk", "Bash", "Cron", "Ham Radio" ]
categories = [ "Amateur Radio" ]
type = "posts"
series = []

+++

A couple of months back, I switched from [HamVoIP](https://www.hamvoip.org/) over to [AllStarLink v3](https://www.allstarlink.org/). Main reason? I wanted updated software to reduce the risk of compromise—especially since I'm port forwarding IAX from the open internet. Makes sense, right?

Now, this post isn’t about installing AllStarLink 3 from scratch (that’s already covered [here](https://allstarlink.github.io/user-guide/pi-detailed/). Instead, I want to talk about getting identing working properly. Why? Because here in VK (Australia, for those outside the ham world), we’re required to ID every 10 minutes when transmitting, or when running a repeater or simplex "hotspot."

Problem is... I thought it was working the same way it does in HamVoIP. Turns out, it wasn’t.

---

## First Stop: `rpt.conf`

Let’s start where I would with HamVoIP: checking the `rpt.conf` settings. While this is a much more up-to-date application, it is still built on what is essentially the same stack. `rpt.conf` is the core configuration file that controls how your node behaves, including when and how it announces its ID. It’s where you'd normally configure automatic identing.

```INI
idrecording = |iHI HI DE VK6CIA     ; ID recording or morse string
idtalkover  = |iVK6CIA              ; Talkover ID (optional)
idtime      = 600000                ; ID interval (ms), default 5 mins (300000)
politeid    = 30000                 ; Time before expiry to ID in the tail (default 30000)
```

Perfect! Looks like all I need to do is enable and bump `idtime` up to 600000 (10 minutes), and we’re golden. I'll set `politeid` just to avoid it IDing over the top of transmissions. At the same time I also set the `idtalkover` to just be a shorter version of the normal ID so it wouldn't be so intrusive during conversations.

Yes, it works—but only *if* there’s active transmission since the last ident. If no one’s talking, the timer doesn’t trigger. Not ideal and not meeting the regulations here in VK.

As a side note for those potentially wondering, it would work *if* I was running full duplex, but as I'm only using duplex mode "1" (Half duplex with telemetry tones and hang time), it doesn't.

---

## Plan B: Cron It

Since the built-in timers weren’t cutting it, I figured why not lean on something that does run no matter what—cron. I already had a simple time announcement script scheduled for the top of every hour, so adding a second cron job to handle IDing felt like a natural workaround.

Here’s what I started with:

```Bash
#! /usr/bin/bash

/usr/bin/sudo asterisk -rx "rpt cmd 60349 status 11 xxx"
```

Simple. The script fires off a status 11 command (Force ID) to the node via the Asterisk CLI.

And the cron entries:

```Text
00 * * * * /etc/asterisk/scripts/saytime.sh
00,10,20,30,40,50 * * * * /etc/asterisk/scripts/sayid.sh
```

This worked well... except when both scripts ran at the same time—like at the top of the hour. Then it was chaos. The ID and the time would stomp on each other, playing simultaneously, turning what should’ve been clean into a cluttered mess.

### Plan B: And a Half

To fix the overlapping audio, I tweaked the saytime.sh script to include the ID right after the time announcement. That way, I could remove the sayid.sh entry for the top of the hour entirely.

Keeping the same ident script as the snip-it above, here was the new time script:

```Bash
#! /usr/bin/bash

/usr/bin/sudo asterisk -rx "rpt cmd 60349 status 12 xxx"
/usr/bin/sudo asterisk -rx "rpt cmd 60349 status 11 xxx"
```

And the quick modification to to remove the `sayid.sh` script from running at the top of the hour in the cron entries:

```Text
00 * * * * /etc/asterisk/scripts/saytime.sh
10,20,30,40,50 * * * * /etc/asterisk/scripts/sayid.sh
```

It worked, technically—but it still felt a bit fragile. If something went wrong or if I wanted to troubleshoot later, there’d be no logging, no context—just a silent failure waiting to happen. Plus, having two scripts doing essentially the same thing? Not elegant.

---

## Final Version: Combined Script with Logging

Time to refine. I wanted to pull all of the above into one script. One script to rule them all!

The goal:
* announces the time and ID at the top of the hour,
* just the ID every 10 minutes in between,
* and logs every action it takes so I’m not debugging in the dark.

After refreshing my Bash scripting skills here is the updated script:

```Bash
#! /usr/bin/bash
# This script announces the current time and repeater ID at the top of each hour.
# At any other minute, it announces only the repeater ID.
# It uses the Asterisk command line interface to send commands to the repeater.

# Log function: writes a timestamped message to stdout
log_message() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1"
}

# Log script start
log_message "Script execution started"

# Get the current minute (e.g., 00, 10, 20, etc.)
CURRENT_MINUTE=$(date +%M)
log_message "Current minute is $CURRENT_MINUTE"

# Check if it's exactly on the hour (minute == 00)
# If it is then announce the current time
if [ "$CURRENT_MINUTE" = "00" ]; then
    log_message "Running time announcement"
    /usr/bin/sudo asterisk -rx "rpt cmd 60349 status 12 xxx"
    log_message "Time announcement completed"
fi

# Run repeater ID regardless of minute
log_message "Running repeater ID"
/usr/bin/sudo asterisk -rx "rpt cmd 60349 status 11 xxx"
log_message "Repeater ID completed"

# Log script end
log_message "Script execution completed"
```

And the appropriate matching cron entry:

Little side note, make be sure to remove the other two cron entries if you have setup the same previously, else all 3 scripts would be running causing more overlapped audio.

```Text
*/10 * * * * /etc/asterisk/scripts/sayidandtime.sh >> /var/log/asterisk/sayidandtime.log 2>&1
```

This way:
* You get consistent 10-minute identing.
* The top of the hour gets the time + ID combo.
* Everything’s logged in one place, which is a lifesaver when something misbehaves.

No more audio overlap, no more manual juggling between two cron entries. Just one script, doing its job, quietly and reliably. Everything is running like clockwork...

---

## But Wait... One More Thing

After about a week of smooth sailing, everything seemed to be working great. I’d hopped on the [F-troop](https://groups.io/g/ftroop) net this Saturday morning and was happily listening in when I noticed something odd—every so often, the repeater was identifying **twice** in quick succession during active transmissions. Not cool.

At first, I thought maybe the cron job had somehow fired twice, or maybe there was some funky bug in the script I missed. So I went back through the logs—nothing out of the ordinary there. The script was doing exactly what it was told to: ID every 10 minutes, with the time announcement on the hour.

That’s when it hit me.

I’d completely forgotten to disable the default ID handling in `rpt.conf`. Remember those lines?

```INI
idtime = 600000
politeid = 30000
```

Yep—those were *still* active. So the repeater was IDing automatically based on the built-in timer **and** my cron job was triggering its own ID, right on schedule. Two IDs, back to back.

I commented those lines out:

```INI
;idtime = 600000
;politeid = 30000
```

Restarted Asterisk, waited 10 minutes, and boom—just one clean ID, exactly when I expected it.

### Moral of the story?

If you're adding your own IDing mechanism via cron, **make sure you’re not doubling up with the default system**. It’s easy to overlook, especially since the defaults seem harmless at first. But when they overlap, it’s messy and sounds broken.

Also worth noting—this isn’t a “compliance” issue, it’s purely a user experience thing. You’re still technically IDing, just not gracefully.

---

## The Wrap-up

If you’re using AllStarLink 3 and want reliable and predictable identing (even when the repeater is idle), a custom cron job is the way to go currently. Just remember to clean up your `rpt.conf` to avoid the dreaded duplicate IDs. This setup ticks the compliance box, avoids annoying overlaps, and keeps things simple as they can be.

**Done and dusted.**

*Until I can figure out a way to implement `politeid` into the mix* 😉

73, VK6CIA