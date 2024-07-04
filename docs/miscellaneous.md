# Misc

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

## Video Game

In 2023, I used Game Maker Studio to make a 2D RPG computer game as a birthday gift.
The engine uses its own language called Game Maker Studio, and I can highly recommend it, as I had a lot of fun and a great learning experience.
I chose this engine because  it was used by Toby Fox to make 'Undertale', a great game that majorly inspired my project.
I can highly recommend Peyton Burnham's YouTube videos on the engine.
I also uploaded a tiny bit of my code into <a href="https://github.com/witchessabath/misc" target="_blank">this GitHub repo</a>, so you can have a sneak peek at what the Game Maker Script Language looks like if you wish.