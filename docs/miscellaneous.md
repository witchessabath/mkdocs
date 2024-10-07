# Miscellaneous

Here I will share some miscellaneous projects that don't really have anything to do with the rest of my homelab.

## Arduino Weather Station

I made a little 'Weather Station' to display the current weather in my city using an Arduino, a LCD display and the OpenWeatherMap API.
I did the following steps to make it work:

- Install the Arduino IDE on my PC
- Connect the Arduino to my PC and select the correct COM port, so the code can be loaded from the IDE to the Arduino
- Connect the Arduino to the LCD display with jumper cables
- Retrieve an API Key from <a href="https://openweathermap.org/" target="_blank">OpenWeatherMap</a>
- Write the code, which you can find <a href="https://github.com/witchessabath/misc/blob/main/WeatherDisplay.ino" target="_blank">here</a>.

!!! note
    Make sure to enter the correct data types for the JSON values (e.g. double for Temperature) - the Arduino_JSON.h library doesn't allow the `Convert.toDouble` method.<br />
    Also, note that OpenWeatherMap display Temperature in Kelvin - hence the line `double temperatureC = (temperatureK - 273.15);` to convert it to Celsius.<br />
    Another tip: In the Arduino IDE, go to 'Tools -> Serial Monitor' to view the `Serial.println` lines. Make sure you have the correct Baud rate selected.

## Arch Linux Configuration


> A quick section to explain some things I had to setup myself when using Arch:

### How to display AppImages in Rofi
I use rofi as an app launcher. It works great, but by default only displays and starts apps installed by the package manager.
After installing AppImages, I wanted them to be accessible from rofi as well.
To enable this, I did the following:
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

### Scripting notfications for battery usage
I noticed when writing a script to notify me when battery is low, that notifications can't be delivered.
The script is:
```bash
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
I then realized I hadn't enabled dunst via `sudo systemctl enable dunst.service`. Only using the `systemctl start` command will start the service in that session, but to be able to automatically a service, the `enable` command is needed. Then, I could simply autostart dunst via my window manager i3 (`exec-always --no-startup-id dunst`).


## Video Game

In 2023, I used Game Maker Studio to make a 2D RPG computer game as a birthday gift.
The engine uses its own language called Game Maker Studio, and I can highly recommend it, as I had a lot of fun and a great learning experience.
I chose this engine because  it was used by Toby Fox to make 'Undertale', a great game that majorly inspired my project.
I can highly recommend <a href="https://www.youtube.com/@peytonburnham4316" target="_blank">Peyton Burnham's YouTube videos</a> on the engine.
I also uploaded a tiny bit of my code into <a href="https://github.com/witchessabath/misc" target="_blank">this GitHub repo</a>, so you can have a sneak peek at what the Game Maker Script Language looks like if you wish.