---
layout: post
title: Remove Instrusive Applications from MacOS 13
date: 2024-10-14 03:00 -0700
---

I recently (2023) had the pleasure of trying to uninstall the _Microsoft Teams_ [application](https://www.microsoft.com/en-us/microsoft-teams/download-app) from my Mac (M1 MacOS Ventura 13.6) desktop computer, which I had to install for a specific usage, but now had now further need for, and wished to remove.  Teams proved to be a particularly intrusive app which was difficult to fully remove.  Ironically, on Windows 11, there is a handy 'remove app' feature built right into the OS system settings, but on Mac, there isn't.  So it is Mac users who have to do irksome digging into deep, hidden layers of the OS to manually delete files.  The simple, approachable, easy-to-use, "it just works" Mac ethos does not apply to removing applications.

I of course started by finding the main app executable under `/Applications` and moving it to the Trash.  The actual file was named `Microsoft Teams.app`.

But this was not enough.  As per usual with Mac apps, I also looked under `/Library/Application\ Support`, `~/Library/Application\ Support`, and `~/Library/Preferences` for any matching subfolders or files to remove.  These are where settings files live which allow you to remove an app, reinstall it, and still have the saved configurations from the previous install (as in the case of an update).  I'm sorry that I didn't record any specific filenames there which I may have found and deleted in this debugging process.  But even if there was anything there, there was still another problem that persisted:

<img alt="all input sources" src="https://dev-blog-images-2024.s3.us-west-1.amazonaws.com/article-msteams/msteams-01.jpeg" width="600px" />

This app installs a system-wide _Audio Device_, which shows up on the left panel of the _Audio Midi Setup_ Utility app, which sticks around after removing the actual app, as described above.  This annoyed me, as I do actually make use of Audio Midi Setup when working on music and video production projects.  The listed device _should_ be removable with the minus icon `-` on the bottom of the window, but this was frustratingly grayed out.

Some searching led me to a helpful audio production [blog](https://www.production-expert.com/production-expert-1/how-to-remove-unwanted-mac-audio-devices#:~:text=If%20you%20want%20to%20remove,works%2C%20so%20try%20this%20first.), which advised searching under `/Library/Audio/Plug-Ins/HAL` which is where _Hardware Abstraction Layer_ files live.  I was pleased to find something there:

<img alt="all input sources" src="https://dev-blog-images-2024.s3.us-west-1.amazonaws.com/article-msteams/msteams-03.jpeg" width="600px" />

Zoom, OBS, and other A/V apps don't seem to need anything in this folder, but Teams does, and this is why I regard this piece of software as intrusive and over-engineeded.  Anyway, I trashed it.  I tried restarting Audio Midi Setup:

<img alt="all input sources" src="https://dev-blog-images-2024.s3.us-west-1.amazonaws.com/article-msteams/msteams-02.jpeg" width="600px" />

Sadly, it is still there, only now in an error state since I have removed the configuration files, and it was still undeletable.  I did more searching online, and was lucky to find this [StackExchange](https://apple.stackexchange.com/questions/375483/how-to-remove-a-sound-output-device-created-by-an-application) Q/A which excactly matched my needs.  Apparently the complete fix involves re-booting the `coreaudiod` daemon.  This can be done with:

```bash
sudo /bin/launchctl kill SIGTERM system/com.apple.audio.coreaudiod || /usr/bin/killall coreaudiod
```

Before trying this, I first tried a full system _restart_, which did resolve the problem:

<img alt="all input sources" src="https://dev-blog-images-2024.s3.us-west-1.amazonaws.com/article-msteams/msteams-04.jpeg" width="600px" />

I'm now back to normal and hopefully won't have to do this again.

UPDATE

Similarly, I had to remove another intrusive application, which goes beyond simply trashing the file found under
`/Applications`.  This application had installed a daemon that shows up under `🍎 > System Settings... > Login Items & Extensions > Allow in the Background`.  These items will likewise not be removed simply by removing the app file.

This app just so happened to be [SoundID Reference](https://www.sonarworks.com/soundid-reference), by Sonarworks, which actually does have legitimate reason for installing lower-level driver/system extension software, as it happens.  It's a fine piece of software that serves a very specific, niche, need, and I'm not trying to diss that product or the company.  I had simply installed the demo version, tried it briefly, and then determined that I no longer need that on this particular machine, although I may return to it in the future on other machines.  The [Steam](https://www.tomsguide.com/how-to/how-to-install-steam-on-mac) gaming platform is a similarly instrusive piece of software, which shouldn't really need such drivers, which I also noticed had also installed files in the same locations right alongside, and I will likely also remove that from this machine in the future.

As a helpful poster at the apple support forum [has said](https://discussions.apple.com/thread/254320918?sortBy=rank), Look under `/Library/LaunchDaemons` and `/Library/LaunchAgents`.  This is good, helpful, and correct, but not enough.  I also had to look under `~/Library/LaunchAgents`, that is, `Macintosh HD > User > [username] > Library > LaunchAgents` as well. Finally, a restart is required.