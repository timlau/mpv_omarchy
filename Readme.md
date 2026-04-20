## mpv OSC for Omarchy

[mpv](https://github.com/mpv-player/mpv) is a free (as in freedom) media player for the command line. It supports a wide variety of media file formats, audio and video codecs, and subtitle types.

The On Screen Controller (short: OSC) is a minimal GUI integrated with mpv to offer basic mouse-controllability. It is intended to make interaction easier for new users and to enable precise and direct seeking.

This OSC adapt colors from Omarchy theme, so then a new theme is enabled, then this OSC will use the colors for the Theme.

## Attribution

The OSC is based on [ModernZ](https://github.com/Samillion/ModernZ) that is based on the following projects. 
 - [mpv-osc-modern](https://github.com/maoiscat/mpv-osc-modern)
 - [cyl0/ModernX](https://github.com/cyl0/ModernX)
 - [dexeonify/ModernX](https://github.com/dexeonify/mpv-config/blob/main/scripts/modernx.lua)
 - the original mpv [osc.lua](https://github.com/mpv-player/mpv/blob/master/player/lua/osc.lua) script
 
 # License
 This project is derived from code licensed under LGPLv2.1+, so the omarcy.lua file is licenced under LGPLv2.1+ see [LICENCE.LGPL](LICENSES/LICENCE.LGPL) for details.
 Other files in the project is licenced under the MIT License, there is compatible with the LGPLv2.1+ Licensed code.
 Check [LICENCE.MIT](LICENSES/LICENCE.MIT) for details

 ## Installation
 
 ```
 git clone git@github.com:timlau/mpv_omarchy.git ~/.config/mpv
 cp ~/.config/mpv/omarchy/defaults/themed/mpv_colors.conf.tpl ~/.config/omarchy/themed/mpv_colors.conf.tpl
 # Optional remove the .git directory
 rm -rf ~/.config/mpv/.git
 ```
 
  - Switch the theme to another one and the `~/.config/omarchy/current/theme/mpv_colors.conf` will be created.
  - This file with be used for the skin to set the matching colors of the current theme.
