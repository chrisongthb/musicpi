# musicpi
Headless Music Jukebox

## What?
This is a step by step guide to install [raspotify](https://dtcooper.github.io/raspotify/) on a rasperry pi (or in my case a antique bananapi).

The _killer feature_ is that you can use a left over (wireless) mouse to control the volume and to stop the spotify playback.

## Why?
I have my bananapi already running with Ubuntu Server 16.04 for some other reasons, so I could not use the pre installed image of [pimusicbox](https://www.pimusicbox.com/).

## Future plans?
* install [mopidy](https://mopidy.com/) next to raspotify
* stop each other playback automatically
* multiroom feature, maybe with [snapcast](https://github.com/badaix/snapcast)

## How?
### Install Raspotify and Alsa
important: it works only, if pulseaudio is _not_ installed.
```bash
sudo apt install -y apt-transport-https curl
curl -sSL https://dtcooper.github.io/raspotify/key.asc | sudo apt-key add -v -
echo 'deb https://dtcooper.github.io/raspotify raspotify main' | sudo tee /etc/apt/sources.list.d/raspotify.list
sudo apt update
sudo apt install raspotify alsa-utils
```

### Configure Raspotify

Configure params for raspotify in `/etc/default/raspotify`. I use a good USB sound card, as the built-in sound card on the bananapi is very bad.

Get your device number with `aplay -l`. The syntax beneath is `--device hw:<card>,<device>`:

```bash
DEVICE_NAME="BananaPi"
BITRATE="320"
OPTIONS="--device hw:1,0"
VOLUME_ARGS="--linear-volume --initial-volume=100"
BACKEND_ARGS="--backend alsa"
```

Now (re)start raspotify:
```bash
sudo systemctl restart raspotify
```

And connect your device!

If you have problems getting sound working, see [here](https://dtcooper.github.io/raspotify/#troubleshooting), [here](https://github.com/librespot-org/librespot/wiki/Audio-Backends#alsa) or even [here](https://github.com/dtcooper/raspotify/issues?utf8=%E2%9C%93&q=is%3Aissue+is%3Aopen+sound)


### Configure your mouse as media remote

First we plug a mouse to your pi. Then we need to install the stone-age software [gpm](https://wiki.archlinux.org/index.php/General_purpose_mouse) and setup the event listener. Mind, that we cannot parse mouse wheel events with gpm (wheel up + down have the same event IDs). The left button will lower the volume, the right one raises the volume. The middle click (on the mouse wheel) will stop the playback (restart raspotify).
```
sudo apt install gpm
sudo curl -o /usr/local/sbin/gpm_spotify_media_control https://github.com/chrisongthb/musicpi/blob/master/resources/gpm_spotify_media_control
sudo curl -o /etc/systemd/system/gpm_spotify_media_control.service https://github.com/chrisongthb/musicpi/blob/master/resources/gpm_spotify_media_control.service
sudo systemctl daemon-reload
sudo systemctl enable gpm_spotify_media_control.service
sudo systemctl start gpm_spotify_media_control.service
sudo systemctl enable gpm
```

Now you're done!

Maybe you want to...

### Check setup

* `sudo systemctl is-enabled gpm raspotify gpm_spotify_media_control` should print `enabled` three (!) times.
* The command `sudo systemctl status gpm_spotify_media_control.service` should notice "active // running" and
* `sudo journalctl -fu gpm_spotify_media_control` should show you what is going on with your mouse, if you click one of the three configured buttons.
* You may also watch the volume (changes) with `sudo alsamixer`.
