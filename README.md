# RasPIano
headless Raspian with fluidsynth for Raspberry Pi 1 Model B

# Raspberry PI digital Piano

### Hardware

- Raspberry PI Model B Rev.2 (512 MB RAM, 700MHz)
- HiFi Berry DAC (Dac Sabre Es9023 Analog I2S 24 Bit 192 Khz Decoder Board)
- MIDI-Keyboard (MIDI-USB)



### Software

- Alsa 
- fluidsynth



## Einrichtung

### Raspberry Pi

aktuelles Raspian lite installieren

Für den ersten Zugriff über SSH eine leere Datei in /boot mit dem Namen "ssh" erstellen.

`$ sudo apt update && sudo apt -y upgrade`

`$ sudo install fluidsynth`

#### raspi-config

`$ sudo raspi-config`

overclocking auf high (950 MHz)

expand filesystem

autologin console

#### config.txt bearbeiten

die config.txt ist in /boot/ und stellt die Systemdienste ein

gpu_mem=16	// da wir keine GUI verwenden können wir der GPU den geringsten Wert zuschreiben

boot_delay=0	// verkürzt den Bootvorgang (muss noch getestet werden)

disable_splash=1 // schaltet den Splash-Screen aus, da wir sowieson kein Display haben

#### cmdline.txt bearbeiten

in der cmdline.txt werden boot parameter gesetzt. Die Parameter wurden aus der Samplerbox Init übernommen und müssen noch getestet werden

`root=/dev/mmcblk0p2 ro rootwait console=tty1 selinux=0 plymouth.enable=0 smsc95xx.turbo_mode=N dwc_otg.lpm_enable=0 elevator=noop`

### Raspberry PI DAC

Die Raspberry PI Audiokarte sorgt für eine bessere Soundausgabe und verringert die Latenz beim spielen.

Link zur verwendeten Karte:

https://www.ebay.de/itm/Audiophonics-I2S-DAC-ES9023-Sabre-to-Analog-24bit-192KHZ-fur-Raspberry-PI-/252228822345

**Descriptions:**

ES9023 is a 24bit stereo sound frequency mode conversion chip (DAC), the use of advanced SABRE digital-analog conversion technology.
The chip set the best sound quality, so that it's a good choice for a number of mode conversion.

**Features:**

ES9023 is a 24bit 192 KHz stereo sound frequency mode conversion chip (DAC).
Uses advanced SABRE digital-analog conversion technology.
Offering direct analog outputs, this module is great for all DIY projects.
Its high versatility enables it to be used directly.

##### Specifications:

Input: I2S
Output: RCA
Sampling Rates Supported: 16 / 24 bit 192KHz
Connectors Delivered
Remote Connections: 29cm
RCA Distance: 20mm
Spacing: 40 * 32mm
PCB Size: 50 * 42mm

#### DAC an den Raspberry PI anschließen

##### I2S Link Line:

Orange Line: MCLK
Yellow line: GND
White Line: DATA
Red Line: LRCK
Black Line: BCK

5V Power Input:
Black Line: GND
Red Line: +5V

from Amazon:

Final connections for your ease:
from DAC to Pi B+ GPIO;
5V to pin2
PWR GND to pin6 (gnd)
BCK to pin12
MCLK removed
LRCK to pin35
GND dac to pin39
DATA to pin40

#### Treiber im Raspberry PI anmelden

Die Audiokarte läuft mit dem HiFi Berry Treiber

Folgende Zeilen müssen in die config.txt eingefügt werden:

- `dtoverlay=hifiberry-dac`
- `device_tree_param=i2c_arm=on`	// optional (muss getestet werden)
- `disable_audio_dither=1`	// muss getestet werden
- `dtparam=audio=off`	// oder Zeile komplett löschen. Schaltet onboard audio aus



Ab hier testen ob alles läuft !!!

### fluidsynth

Für fluidsynth müssen je nach System die besten Einstellungen gefunden werden, damit es stabil und mit einer geringen Latenz läuft.

Dafür hat es gute Ergebnisse fluidsynth mit root Rechten auszuführen damit der Prozess mit einer höheren Priorität läuft und keine Warnung auf die Konsole ausgegeben wird.

fluidsynth best practice

```sh
# mit Samplerate kann mit der Latenz gespielt werden
$ fluidsynth \
				-is \ # start fluidsynth in server mode without interactive
				-a alsa \ # startet fluidsynth mit alsa driver
				-o audio.alsa.device=hw:0 \ # greift direkt auf Hardware (geringere Latenz)
				-g 1 \ # gain. Bei manchen SoundFonts kam es ab 2 zu Distortion
				-C0 # Chorus wird ausgeschaltet
				-R0 # Reverb wird ausgeschaltet
				/usr/share/sounds/sf2/FluidR3_GM.sf2 # Pfad zum SoundFont
```



#### fluidsynth settings

Die folgenden settings können mit den `-o` Switch angegeben werden:

audio.alsa.device        STR   [def='default']

audio.driver             STR   [def='' vals:'alsa','file','jack','oss','pulseaudio','sdl2']

audio.file.endian        STR   [def='auto' vals:'auto','big','cpu','little']

audio.file.format        STR   [def='s16' vals:'double','float','s16','s24','s32','s8','u8']

audio.file.name          STR   [def='fluidsynth.wav']

audio.file.type          STR   [def='auto' vals:'aiff','au','auto','avr','caf','flac','htk','iff','mat','mpc','oga','paf','pvf','raw','rf64','sd2','sds','sf','voc','w64','wav','wve','xi']

audio.jack.autoconnect   BOOL  [def=False]

audio.jack.id            STR   [def='fluidsynth']

audio.jack.multi         BOOL  [def=False]

audio.jack.server        STR   [def='']

audio.oss.device         STR   [def='/dev/dsp']

audio.period-size        INT   [min=64, max=8192, def=64]

audio.periods            INT   [min=2, max=64, def=16]

audio.pulseaudio.adjust-latency BOOL  [def=True]

audio.pulseaudio.device  STR   [def='default']

audio.pulseaudio.media-role STR   [def='music']

audio.pulseaudio.server  STR   [def='default']

audio.realtime-prio      INT   [min=0, max=99, def=60]

audio.sample-format      STR   [def='16bits' vals:'16bits','float']

audio.sdl2.device        STR   [def='default' vals:'default','snd_rpi_hifiberry_dac, HifiBerry DAC HiFi pcm5102a-hifi-0','vc4-hdmi, MAI PCM i2s-hifi-0']

midi.alsa.device         STR   [def='default']

midi.alsa_seq.device     STR   [def='default']

midi.alsa_seq.id         STR   [def='pid']

midi.autoconnect         BOOL  [def=False]

midi.driver              STR   [def='' vals:'alsa_raw','alsa_seq','jack','oss']

midi.jack.id             STR   [def='fluidsynth-midi']

midi.jack.server         STR   [def='']

midi.oss.device          STR   [def='/dev/midi']

midi.portname            STR   [def='']

midi.realtime-prio       INT   [min=0, max=99, def=50]

player.reset-synth       BOOL  [def=True]

player.timing-source     STR   [def='sample' vals:'sample','system']

shell.port               INT   [min=1, max=65535, def=9800]

shell.prompt             STR   [def='']

synth.audio-channels     INT   [min=1, max=128, def=1]

synth.audio-groups       INT   [min=1, max=128, def=1]

synth.chorus.active      BOOL  [def=True]

synth.chorus.depth       FLOAT [min=0.000, max=256.000, def=8.000]

synth.chorus.level       FLOAT [min=0.000, max=10.000, def=2.000]

synth.chorus.nr          INT   [min=0, max=99, def=3]

synth.chorus.speed       FLOAT [min=0.100, max=5.000, def=0.300]

synth.cpu-cores          INT   [min=1, max=256, def=1]

synth.default-soundfont  STR   [def='/usr/share/sounds/sf3/default-GM.sf3']

synth.device-id          INT   [min=0, max=126, def=0]

synth.dynamic-sample-loading BOOL  [def=False]

synth.effects-channels   INT   [min=2, max=2, def=2]

synth.effects-groups     INT   [min=1, max=128, def=1]

synth.gain               FLOAT [min=0.000, max=10.000, def=0.200]

synth.ladspa.active      BOOL  [def=False]

synth.lock-memory        BOOL  [def=True]

synth.midi-bank-select   STR   [def='gs' vals:'gm','gs','mma','xg']

synth.midi-channels      INT   [min=16, max=256, def=16]

synth.min-note-length    INT   [min=0, max=65535, def=10]

synth.overflow.age       FLOAT [min=-10000.000, max=10000.000, def=1000.000]

synth.overflow.important FLOAT [min=-50000.000, max=50000.000, def=5000.000]

synth.overflow.important-channels STR   [def='']

synth.overflow.percussion FLOAT [min=-10000.000, max=10000.000, def=4000.000]

synth.overflow.released  FLOAT [min=-10000.000, max=10000.000, def=-2000.000]

synth.overflow.sustained FLOAT [min=-10000.000, max=10000.000, def=-1000.000]

synth.overflow.volume    FLOAT [min=-10000.000, max=10000.000, def=500.000]

synth.polyphony          INT   [min=1, max=65535, def=256]

synth.reverb.active      BOOL  [def=True]

synth.reverb.damp        FLOAT [min=0.000, max=1.000, def=0.000]

synth.reverb.level       FLOAT [min=0.000, max=1.000, def=0.900]

synth.reverb.room-size   FLOAT [min=0.000, max=1.000, def=0.200]

synth.reverb.width       FLOAT [min=0.000, max=100.000, def=0.500]

synth.sample-rate        FLOAT [min=8000.000, max=96000.000, def=44100.000]

synth.threadsafe-api     BOOL  [def=True]

synth.verbose            BOOL  [def=False]



### ALSA

#### aplay

Die Konfiguration mit aplay testen ob die Karte Sound ausgibt

`$ aplay /usr/share/scratch/Media/Sounds/Noise.wav`

#### aconnect 

MIDI-USB Device mit fluidsynth verbinden

Die MIDI-Eingabe wird über ALSA mit `aconnect` auf fluidsynth geleitet. 

Input Geräte anzeigen:

`$ aconnect -i

Beispiel Ausgabe:

```cmd
client 0: 'System' [type=kernel]
    0 'Timer           '
    1 'Announce        '
client 64: 'External MIDI-0' [type=kernel]
    0 'MIDI 0-0        '
```

Output Geräte anzeigen:

`$ aconnect -o` 

Geräte verknüpfen:

xx = MIDI Controller - yy = fluidsynth

`$ aconnect xx:0 yy:0`

`$ aconnect 20:0 128:0`

alternative kann man auch die Namen der Geräte eingeben:

$ aconnect 'NAME MIDI DEVICE':0 'FLUIDSYNTH':0

Alle Verbindungen auflösen

`$ aconnect -x`

#### nice

nice erhöht die Priorität mit der das Programm ausgeführt wird gibt im mehr Speicher zur Verfügung. Um fluidsynth mit nice starten zu lassen, braucht man nur den nice Befehl davor setzen.

`$ nice -19 /usr/bin/fluidsynth`

## Einstellungen automatisieren für headless Systemstart

Bash-Script "run_fluid.sh" um den Programmstart zu automatisieren

```sh
#!/bin/bash
# run_fluid.sh

echo "Starting"
sudo nice -19 /usr/bin/fluidsynth \
				-is \ # start fluidsynth in server mode without interactive
				-a alsa \ # startet fluidsynth mit alsa driver
				-o audio.alsa.device=hw:0 \ # greift direkt auf Hardware (geringere Latenz)
				-g 1 \ # gain. Bei manchen SoundFonts kam es ab 2 zu Distortion
				-C0 # Chorus wird ausgeschaltet
				-R0 # Reverb wird ausgeschaltet
				/usr/share/sounds/sf2/FluidR3_GM.sf2 # Pfad zum SoundFont
echo "Fluidsynth started"
# wartet auf fluidsynth start
while true; do if [[ $(/usr/bin/aconnect -o ) = *FLUID* ]]; then break; fi; sleep 2; done
# wartet bis 'SL STUDIO' Keyboard angeschlossen ist
while true; do if [[ $(/usr/bin/aconnect -o ) = *STUDIO* ]]; then break; fi; sleep 2; done
# da sich die Nummer vom MIDI-Device ab und an ändert macht es Sinn den Namen des Keyboards
# anzugeben. Dann kann die Nummer wechseln
/usr/bin/aconnect 'SL STUDIO':0 128:0
echo "Connected"
```

Das Skript ausführbar machen:

`$ sudo chmod +x run_fluid.sh`

#### crontab

Um Programme bei einem Systemstart automatisch starten zu lassen gibt es verschiedene Möglichkeiten. Die meiner Meinung nach einfachste ist die Möglichkeit über crontab.

Crontab Datei öffnen mit:

`$ crontab -e`

und bearbeiten. Die Zeile sorgt dafür, dass das Skript bei jeden Start ausgeführt wird

`@reboot /home/pi/run_fluid.sh`



## ToDo und Test

- Den Befehl SHELL=/usr/bin/bash in die crontab Datei einfügen damit die while Schleife ausgeführt wird. Ansonsten noch einen sleep Befehl einfügen. (Testen ob das wirklich den Unterschied macht) [LINK](https://wiki.ubuntuusers.de/Cron/)
- das Filesystem read-only schalten, damit es keinen Crash auf der SD-Karte beim harten ausschalten gibt.
- - das Filesysten kann über fstab auf "read-only" gestellt werden
    - /etc/fstab --> 
    - <file system>            <mount point>   <type>       <options>                           <dump>  <pass> 
      - /dev/								/												ro											0			0
  - mit dem mount Befehl kann die Festplatte wieder in den Schreibmodus versetzt werden
    - `$ mount -n -o remount, rw /`
- Wpa_supplicant Service deaktivieren, da wir am Raspberry PI 1 kein WLAN haben
  - `$ sudo systemctl disable wpa_supplicant`
- Den scaling_governor der die CPU verwaltet auf performance setzen. Ansonsten macht die CPU performance scaling und das kann zu Aussetzern im Audio führen. der Befehl muss noch angepasst werden, da er noch nicht vom Start Skript übernommen wird
  - `$ echo -n performance | sudo tee /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor`
  - [OVERCLOCKING RASPBERRY PI: SET SCALING_GOVERNOR ON RASPBIAN BOOT](https://putokaz.wordpress.com/2015/03/01/overclocking-raspberry-pi-set-scaling_governor-on-raspbian-boot/)
- die Punkte aus dem start_skrip_example.sh überprüfen
- Raspi-config automatisieren ? 
- Cmdline.txt Einstellungen überprüfen
- eth0 hot plug Einstellungen

 

## Link Liste

- raspian base system
  - http://mirror.internode.on.net/pub/raspbian/raspbian/dists/stable/
- fluidsynth [GitHub](https://github.com/FluidSynth/fluidsynth/)
- [Qsynth and FluidSynth on Raspberry Pi: The basics](http://sandsoftwaresound.net/qsynth-fluidsynth-raspberry-pi/)
- [Building a MIDI Synth With a Raspberry Pi](http://andrewdotni.ch/blog/2015/02/28/midi-synth-with-raspberry-p/?ref=reddit)
- [Keyboard piano on the Raspberry Pi (with FluidSynth)](http://florisdriessen.nl/electronics/raspberry-pi-electronics/keyboard-piano-on-the-raspberry-pi-with-fluidsynth/)
- [Fedora musicians guide](https://docs.fedoraproject.org/en-US/Fedora/15/html/Musicians_Guide/index.html)
- [GNU solfege Musik Training](https://www.gnu.org/software/solfege/solfege.de.html)
- [BUILD A BUDGET MIDI SYNTHESIZER - FluidSynth on Raspberry Pi Tutorial! (Video)](https://www.youtube.com/watch?v=wzIkIQ7Hebc)
- [Fluidsynth low latency tips](https://github.com/FluidSynth/fluidsynth/wiki/LowLatency)
- [fluidsynth privileges](https://forums.raspberrypi.com/viewtopic.php?t=211327)
- [gist gitHub:fluidsynth service](https://gist.github.com/oostendo/b22a57aacc9439f80f74b0545d38148d)
- [Raspberry Pi Autostart von Programmen mit init.d](https://techgeeks.de/raspberry-pi-autostart-von-programmen/)
- [Autostart: systemd zum script starten als service mit systemctl](http://blog.wenzlaff.de/?p=15477)
- [Debian: FluidSynth als systemd-Service](https://blog.geierb.de/debian-fluidsynth-als-systemd-service/)
- [Linuxaudio.org: Raspberry Pi and realtime, low-latency audio](https://wiki.linuxaudio.org/wiki/raspberrypi)
- [Raspberry Pi Performance Tuning und Tweaks](http://raspberry.tips/raspberrypi-tutorials/raspberry-pi-performance-tuning-und-tweaks)
- [Daneil A. Russel: Hammer nonlinearity, dynamics and the piano sound](https://www.acs.psu.edu/drussell/Piano/Dynamics.html)



## SoundFonts

- [Ultimate Top List of 900+ FREE Soundfonts SF2 – (2021 Updated)](https://www.producersbuzz.com/downloads/download-free-soundfonts-sf2/ultimate-top-list-of-900-free-soundfonts-sf2-2020-updated/)
- https://sites.google.com/site/soundfonts4u/
- 
