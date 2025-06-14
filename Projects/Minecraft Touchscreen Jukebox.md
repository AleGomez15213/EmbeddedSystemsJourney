This project will be a Jukebox that will display have a circular touchscreen on the top. It will use Spotify to service music and will need Wi-Fi capabilities.
# Parts Needed
- Microcontroller (ESP, Rasberry, etc.)
	- Needs to connect to wifi to use Spotify
- Speaker(s)
- 3D casing
# Milestones
1) Display screen on touch display
2) Control Raspberry Pi functions with touch controls
3) Design app to be used as scratch-table interface
4) Add rewind feature

# Concept
![[Pasted image 20250508092710.png|400]]![[Pasted image 20250508092744.png]]

# Design Planning
Most Raspberry Pi Spotify devices work either as headless devices (no GUI) or install an OS like Volumio. I want to design my own program/GUI and have the Pi interact with Spotify's API. This will allow me to have the record player controls and play music natively from Pi.

The app can be written in C/C++, run a custom GUI, and use https://github.com/smaltby/spotify-api-plusplus to connect to the Spotify API. I can probably use https://wiki.qt.io/Qt_for_Beginners for the GUI implementation. It will primarily need a rotating disk to present the album art, and basic controls (play/pause, next song, previous song). Later on, I can add a draggable line to speed up a song and maybe basic UI to select specific songs/playlists.