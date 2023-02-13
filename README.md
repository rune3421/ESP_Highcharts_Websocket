# ESP_Highcharts_Websocket
Plotting streaming data to a Highcharts server via Websocket.

This repository will have multiple codes, because I've pursued this target over multiple exercises, and I'll post the end result code of each attempt here...and the Readme will be a sequential narrative of the quest. 

Hello, and welcome to another random project with Karl. 
Today, I’m pursuing a Websocket interface between Highcharts and an ESP8266. 

I’ve recently found out that highcharts has a refresh rate limit of 13Hz when dealing with wifi sources over regular HTML (and that could be partly because of the Sonar chip I was using), and that the ESP8266 has some wifi speed limitations, so today I’m going to take the HTML out of the way with this websocket Highcharts project that I’m adapting to see how close we can get to charting an audio waveform. 

This code is based on the Arduino MKR1000, so that’ll be a thing to swap out, and since I’m only using one sensor I’ll need to change some of the website code and swap out their DHT sensor. 

I have the Elegoo Big Sound microphone already plugged into the ESP8266 ADC, with ground and 3.3v for the other pins. I’ve also connected the Digital Out to D0 just in case I want to mimic two separate sensors over websocket at the end of the project, but for now I just want to see how close to true waveform the ESP8266 can perform on a websocket. 

I’ll start out by repackaging the two parts of the code into a single page, by putting the HTML code into the PROGMEM portion of the Arduino code. I’m borrowing the structure from a prior exercise to do this. 

So, other than embedding the HTML code in the PROGMEM, I have the entire sample project loaded into my Arduino IDE. Next I need to make sure of a few things;

I’m sampling from the right sensor type
The WIFI credentials are correct
The Webpage is formatted correctly for one sensor (we might go back to two later)
The WIFI settings are compatible with the ESP8266
Documentation is correct, so I don’t get lost later while debugging

Those things should be enough to make sure that the initial trial runs on the page, and then we can work on debugging and optimization. Most of this can be done by comparing and replacing code from my previous exercise, and leaving the websocket portions intact. 

Aand, after some editing, I think I’ve got it figured, except for item 4. The libraries are the part I’m least sure of, because of potential incompatibilities or syntax differences, but nothing to it but to try it. Let’s see if it loads!

The code is called ESP_Websocket_Rev1
