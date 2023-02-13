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

Now, verifying on the IDE revealed that I’m missing some libraries, so we’ll add those real quick.

Libraries have been added, and it looks like the code isn’t exactly compatible with the libraries, so we’ll be swapping those parts out with the examples that go with the libraries for those callbacks. 

So I did some refactoring of the websocket handlers and callbacks, and debugged the compile a bit. Compile finished without errors, so now it’s time for an upload. Let’s see what happened. Well, the ESP connected, and posted the connection details to Serial, but the website times out. Let’s see if we can find out if there’s a missing call for uploading the website.

Yeah, there’s no matching call in the websocket code for sending the web page details. We’ll have to include that somewhere before the websocket starts up. Let’s see if we can find the spot. Found it. The Wifi is started and connected, then the server is begun, and then the websocket. If I put in the call to send the website details in the setup between the server and the websocket, it should load the web page. 

So I added a call after the server.begin() to client.print() a group of messages, including the file for the website, yet now instead of timing out, the server is refusing to connect. So…I gotta figure out what that is. 

And after some fenaggling and putsching around, I think I’ve understood the problem. The ESP sets up the websocket for the whole website, and then doesn’t understand when the website is asking for formatting. I think I need to layer this connection; one part for loading the website on an asynchronous callback, and then once the charts are ready, setting up websockets for updating their data. 

So, refactoring will look like:

//headers with the async and websocket, the WiFi credentials, the ports and the sensor variables

//website format stored in PROGMEM, with calls to start websocket

//WiFi connection function

//Setup for Async browser startup, with callback to load website

//Function to start websocket that starts when called by website

//Void loop for gathering and sending data over websocket

I think for this one, I’ll double back and start with the already working Async Highcharts code, swap the sensor code, check that, and then start working in the bits to update the highcharts through websocket inside that. 

Swapping out the sensors in the code for the microphone in the Async_Highcharts code worked. Now time for adding in the websocket pieces, starting with the library.

With the library and all websocket data included, the code compiles, and the information still flows via the async webserver to the Highcharts plotter. 

Now it’s time to refactor the website code to call for the websocket, and the Arduino code to send the data via the websocket. And, as a matter of principle, I went and did some research to find out how to write the call for a websocket server to a websocket client, and discovered a tutorial that writes the entire code, including splitting between an Async startup and a Websocket data stream for the chart. So I’ll be wrapping up this exercise and starting fresh with a new tutorial. 


Hello and Welcome to another episode of Karl Fiddles Around. Today I’m going to be using a compilation of tutorials to cobble together a websocket server plus Highcharts webpage through an ESP8266. Here are the tutorial sources: 
Websocket Server via Async Page
Highcharts Plotter via Async Webpage

There may be others as I go through, because there may be catches to it I don’t know about yet…but hopefully not. 

We’ll start with importing and compiling the tutorial code in two separate Arduino IDE windows to make sure they are whole. And, they are. 

Now it’s time to start hacking away. First things are the simple things; swapping out the sensor, and removing unnecessary pieces, maybe adding some comments. I’ve swapped the sensor out for the Elegoo Big Sound sensor, for this pinout. I've connected the analog to ADC and digital to D12, using the 3.3v for the voltage. 

Now it’s time to start comparing the codes, to see if we can get the highcharts container to accept websocket-sourced data. 

It looks like there may be a tricky bit in the middle, but I’ve identified the parts that need to be matched. It’s time to merge the codes into a single page, and compile. I’ll go through in the order that the functions are called when booting, so that I make sure I trace all process threads as I go. 

Just so I keep track, the order is:


Header info like libraries and globals, 
Setup 
	Connect Wifi
InitWebsocket
	ws.onEvent
Server.begin
Website load (server.on”/”)
	CSS
	Highcharts container
	Javascript websocket handler
		Highcharts data plotter
Loop
	Sensor reading
	Send data via websocket
	Plot data received via websocket


And I’ve gotten all of the pieces merged, except I can tell that I’m not matching up the data types between the client side and the server side, so I’ve added one more tutorial: this github to help me interpret how the data is formatted on each end of the websocket communication. 

And, with that, I’ve figured it out, and added back the notifyClients call in the void loop, with the function to send the data string. 

Now, so far I’ve been compiling this for the ESP8266, and I have a broken library link for the sdkconfig which makes the compile fail, so I won’t be able to test until I switch eventually to the ESP32, but let’s give it one more read-through, and switch to the ESP32 loading library. 

And it compiles! 

So, before we get into uploading it into the chip and seeing if it works, let’s see the code so far. 

This codebank is labeled Websocket_Server_Via_Async_Rev1

