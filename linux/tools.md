# *nix tools

## dd

> The dd command does not give any information of its progress and so may appear to have frozen; it could take more than five minutes to finish writing to the card. If your card reader has an LED it may blink during the write process. To see the progress of the copy operation you can run pkill -USR1 -n -x dd in another terminal, prefixed with sudo if you are not logged in as root. The progress will be displayed in the original window and not the window with the pkill command; it may not display immediately, due to buffering.
> 
> Instead of dd you can use dcfldd; it will give a progress report about how much has been written.
>
> -- <cite>RaspberryPi.org (https://www.raspberrypi.org/documentation/installation/installing-images/linux.md)</cite>
