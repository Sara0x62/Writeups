"What information can you possibly get with just one photo?"

Alright, let's start with this-

The Image we've been given:

![](../attachments/1b06d405e3dfa9c34b5c688cec0c7f84.jpg)

First of, lets see if there's anything interest in its metadata with `exiftool`:

![](../attachments/0a330838305880fe0961820ac3ae7f83.png)

Interesting parts in here seem to be the GPS and copyright
```
GPS Latitude                    : 54 deg 17' 41.27" N
GPS Longitude                   : 2 deg 15' 1.33" W
Copyright                       : OWoodflint
```

## GPS Location
Coords: `54° 17' 41.27" N, 2° 15' 1.33" W`
Searching these in Google Maps puts us here.

![](../attachments/e0320dd5f5a4c14becd6953021fcf578.png)

## Copyright name:
`OWoodflint`

Googling it we end up with 3 links that look interesting (`-tryhackme` to try and avoid spoilers T_T )
![](../attachments/de0c35ad8c1fe535d1642c4c46b91d88.png)

![](../attachments/74baecfbd907398e7dfbcd84c538c68b.png)

## The BSSID
Using [Wigle](https://www.wigle.net/). with the BSSID: `B4:5D:50:AA:86:41`
We end up here (see the partial circle on London)

![](../attachments/7e09c27b8c1ff6f5097ad06ff1f14383.png)

After zooming all the way in and clicking on it we get some more detailed network info;

![](../attachments/de3d8c4a83d99d68f099709e23f86934.png)

## Their GitHub
At first glance, their GitHub profile ( https://github.com/OWoodfl1nt ) doesn't have anything visibly interesting, however their repository ( https://github.com/OWoodfl1nt/people_finder ) has some interesting info in the `README.md` mainly an email address and a link to a WordPress website!

With this we should have all our info except 1 thing, their password

![](../attachments/f247c57b201e41c0e430dd40b7173ecd.png)

After lots of scrolling around the source code of their website, I finally notice it, right under their post!
sneakily hidden text..

![](../attachments/98ea9dafd95920655c42d657daf5c696.png)

With this we solved all the questions and completed this room!