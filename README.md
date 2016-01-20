# tcg-ocr-scanner

Trading Card Game card scanner using OCR (using Tesseract and OpenCV)

Currently only supports a sub set of Magic: The Gathering™ cards, but the design is generic enough to support any Trading Card Game that uses the card name as the main identifier of the card.

Picked up from https://github.com/starstuffharvestingstarlight/tcg-ocr-scanner and updated

Current hardware setup
  - Samgsung Galaxy s4 running droidCam
  - Coffee box to hold phone up

I hope to get develop a hardware solution that will automatically flip through a pile of cards and scan them into the csv

## Description

### Conceptual data flow

![Data flow chart](https://raw.github.com/starstuffharvestingstarlight/tcg-ocr-scanner/master/docs/application_data_flow.png)

### Demos

| Date | Functionality  | Comments | Videos  | 
| ---- | -------------  | -------- | ------  |
| 2014-03-10 | Clipboard      | This is the first demo of the app, the detection calculation wasn't correct as it included the UI rendering times. This mode is only useful if the system is always on, say, in a games shop selling singles. | [![Clipboard functionality](http://img.youtube.com/vi/xH1hempwqMk/1.jpg)](http://youtu.be/xH1hempwqMk) |
| 2014-03-11 | Export to deckbox.org  | One of the output formats available is the deckbox.org csv format. From there, it is possible to manage the collection appropriately. A lot of the metadata on the card is still being worked on. Note that this detector captured both a split card (a hard one at that too) and a sleeved card. As long as the lighting is good, it will read the card, regardless of whether it is sleeved, in a binder or on its own. The main challenge is the alignment, since there is no card detector (yet).  | [![deckbox.org import](http://img.youtube.com/vi/27QnTu-SrWQ/1.jpg)](http://youtu.be/27QnTu-SrWQ) |

### Detection system description

The system will guess the closest card, provided the poll threshold is high enough. 
It will only output anything if it has a good candidate, which means it will have very few false positives. 
Depending on the dictionary, the total guess space can be reduced as well. 
For example, if one is trying to detect in a space of 250 cards, then the dictionary should only have those and any 'random' card will either fit one of those or not get detected at all. 
There is no upper bound on how long the system takes to identify a card (though one might be added in the future).
As it is now, it takes about 12s to detect a card name.
Detection is highly dependant on image quality, lighting and camera focal distance.
The current setup was optimised for 800x600 webcams, but the detection doesn't seem to depend on the resolution as much as it depends on image quality (nice focus, contrasting text).

## Roadmap

### Main software goals

- support scanning card by card at an acceptable speed (currently at about 12s)
- output to csv formats for importing into other applications
- call optional web service with the detected card
- import card list in other formats (csv, xml) 

### Main hardware goals

- design frame for scanning cards with an off-the-shelf webcam (OpenSCAD)
- design container for piling up cards in the scanner (OpenSCAD)
- design automated card switching mechanism (OpenSCAD + Arduino/RaspberriPi/WhateverWorks)

## Dependencies

### Required Python packages

- opencv (debian package `python-opencv`)
- numpy `sudo pip install numpy`
- [hunspell](https://github.com/smathot/pyhunspell) `git clone git@github.com:smathot/pyhunspell.git && cd pyhunspell/ && sudo ./setup.py install`
- [xerox](https://github.com/kennethreitz/xerox) `sudo pip install xerox`
- pygame `sudo apt-get install python-pygame`

### Required software

- [tesseract](https://code.google.com/p/tesseract-ocr/)
- [xclip](http://sourceforge.net/projects/xclip/) or equivalent for your OS (see xerox)
- [GNU make](https://www.gnu.org/software/make/) (to build dictionary file)

On debian for example: `sudo apt-get install make xclip tesseract`

### Required data

`cards.xml` with the cards you'd like to recognise. Current format included below (currently what the Cockatrice Oracle Downloader generates).

```xml
<?xml version="1.0" encoding="UTF-8"?>
<whatevertagyoudlike>
  <card>
    <name>Soilshaper</name>
    <set muId="79177">CHK</set>
    <color>G</color>
    <manacost>1G</manacost>
    <cmc>2</cmc>
    <type>Creature — Spirit</type>
    <pt>1/1</pt>
    <tablerow>2</tablerow>
    <text>Whenever you cast a Spirit or Arcane spell, target land becomes a 3/3 creature until end of turn. It's still a land.</text>
  </card>
</whatevertagyoudlike>

```

### Required Hardware

- Webcam with reasonable resolution and lighting (tested 1280x720)
- Optional rig to place the card in the best place for matching and prevent it from moving around

## Installing 

- Get a copy of `cards.xml` and put it in `./db/`
- Run `make` to generate the database and the dictionary

## Running

### Command line options

```
$ ./main.py -h
usage: main.py [-h] [--nobeep] [--clipboard]
               [--batchfile BATCHFILE [BATCHFILE ...]] [-v int]
               [-d DICTIONARY] [--min-suggestions int] [--min-length int]

Scan MTG cards using OCR (tesseract).

optional arguments:
  -h, --help            show this help message and exit
  --nobeep              disable beep when scan happens
  --clipboard           save card name to clipboard
  --batchfile BATCHFILE [BATCHFILE ...]
                        save data to batch file
  -v int, --verbosity int
                        verbosity level (0: quiet, 1: feedback, 2: debug)
  -d DICTIONARY, --dictionary DICTIONARY
                        dictionary to use (e.g., dict/INN for dict/INN.dic +
                        dict/INN.aff)
  --min-suggestions int
                        minimum number of suggestions polled (e.g., 5 means it
                        is considered a correct guess after 5 equal guesses)
  --min-length int      minimum length for a detected card name
  --webcam int          webcam number (usually, the biggest number is the one
                        you plugged in)
```

### Example run

Using a `cards.xml` file with Magic: The Gathering™ card names

```bash
$ ./main.py -v 1
```

#### Output #TODO update

```
Detected: Rapid Hybridization (12.84 secs)
Detected: Rapid Hybridization (10.45 secs)
Detected: Hands of Binding (12.42 secs)
Detected: Simic Charm (14.22 secs)
Detected: Riot Control (15.22 secs)
Detected: Mind Grind (9.07 secs)
Detected: Spark Trooper (6.47 secs)
Detected: Vorel of the Hull Clade (11.54 secs)
Detected: Varolz, the Scar-Striped (14.62 secs)
Detected: Teysa, Envoy of Ghosts (13.34 secs)
Detected: Kessig Wolf Run (12.89 secs)
Detected: Fireshrieker (18.82 secs)
Detected: Megantic Sliver (9.38 secs)
Detected: Elvish Mystic (14.29 secs)
Detected: Bane Alley Blackguard (9.46 secs)
Detected: Doom Blade (12.16 secs)
Detected: Bronzebeak Moa (9.38 secs)
```

Notice: The card names listed in this example are from Magic: The Gathering™, a trademark of Wizards of the Coast, Inc., a subsidiary of Hasbro, Inc. © 2014 Wizards. All Rights Reserved. This project is not affiliated with, endorsed, sponsored, or specifically approved by Wizards of the Coast LLC. Please contact me if there is a problem.

### Test results

Test date  | Dictionary Size | Cards tested | Cards matched | Average T (s) | Max T (s) | Min T (s) | Median T (s) | Error rate
---------- | --------------- | ------------ | ------------- | ------------- | --------- | --------- | ------------ | ----------
2014-03-09 | 14123           | 57           | 51            | 22.8          | 103.5     | 7.39      | 16.19        | 10.5%
2014-03-09 | 14123           | 94           | 90            | 20.8          | 100.49    | 6.04      | 12.82        | 4.26%
2014-03-13 | 14123           | 102           | 99            | 7.96          | 75.48    | 1.27      | 5.28        | 2.94%

