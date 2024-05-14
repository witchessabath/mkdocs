# Arch Linux Configuration

A video of my complete Arch Install will be found here soon.

You can also view my config files here: www.github.com/witchessabath/linuxdotfiles
I used and configured the following software:
- i3-gaps Window manager
- i3statusbar
- rofi App Launcher
- zsh shell
- kitty terminal
- dunst Notification daemon
- Neovim with AstroNvim as Text Editor


> A quick section to explain some things I had to setup myself when using Arch:

## How to display AppImages in Rofi
I use rofi as an app launcher. It works great, but by default only displays and starts apps installed by package manager.
After installing AppImages, I wanted them to be accessible as well.
To do this:
- `chmod +x Example.AppImage` to make the AppImage executable
- `sudo nvim /usr/share/applications/Example.desktop` to create a desktop file for the AppImage
    Content:
    ```
    [Desktop Entry]
    Type=Appication
    Name=Example
    Exec=/path/to/Example.AppImage
    ```
Now rofi can also launch the AppImage.

## Scripting notfications for battery usage
I noticed when writing a script to notify me when battery is low, that notifications can't be delivered.
The script is:
```
#!/bin/bash
bat_files="/sys/class/power_supply/BAT1"
bat_status=$(cat "${bat_files}/status")
capacity=$(cat "${bat_files}/capacity")
echo "${capacity}"
if [[ "${bat_status}"=="Discharging" && ${capacity} -le 25 ]]; then
    echo "Battery alert - ${capacity}%"
    notify-send \
        "Warning: Battery! Only ${capacity}% battery remaining!"
fi 
if [[ "${bat_status}"=="Discharging" && ${capacity} -le 10 ]]; then
    echo "Battery alert - ${capacity}%"
    notify-send \
        "Urgent Warning! Only ${capacity}% battery remaining! Feed me!"
fi
```
Everything was working but the `notify-send` command.
I tried making the notification daemon a D-Bus service by configuring a `org.freedesktop.Notifications.service` file in the D-Bus service directory, but it still didn't reliably start.
I then realized I could simply autostart dunst via my windows manager (`exec-always --no-startup-id dunst`).
