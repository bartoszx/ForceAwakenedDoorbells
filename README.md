# ForceAwakenedDoorbells
## Manual: Preparing and Sending Star Wars-Themed Audio Files

### Requirements
- Dahua VTO (e.g. VTO4202F-P)
- Contactron, for example from [this link](https://a.aliexpress.com/_mNvNBtk)

### YT

https://youtu.be/k64-3JSmAEI
https://youtu.be/s1z7GmH05ak

### 1. Preparing Texts

Prepare various greeting and thank you messages in the style of Star Wars.

### 2. Convert Texts to Audio
Use the service [elevenlabs.io/speech-synthesis](https://elevenlabs.io/speech-synthesis) to convert the prepared texts into MP3 audio files. Save them in the directories `ha-sound/gate` (for greetings) and `ha-sound/mail` (for thanks).

### 3. Convert MP3 to AL
Perform the MP3 files conversion to the AL format using ffmpeg:

```bash
for input in *.mp3; do if [ $(stat -f%z "$input") -lt 102400 ]; then /opt/homebrew/bin/ffmpeg -i "$input" -c:a pcm_alaw -ac 1 -ar 8000 -sample_fmt s16 "${input%.*}.al"; fi; done
```

### 4. Adding Prefixes
Add appropriate prefixes to the file names depending on the directory:

In the `gate` directory:

```bash
for file in *.al; do mv "$file" "welcome-$file"; done
```

In the `mail` directory:

```bash
for file in *.al; do mv "$file" "thankyou-$file"; done
```

### 5. Send Files to the Server
Send the files to HA:

For files in the `gate` directory:

```bash
scp ha-sound/gate/welcome_*.al root@your-ha:/config/mp3/
```

For files in the `mail` directory:

```bash
scp ha-sound/mail/thankyou_*.al root@your-ha0:/config/mp3/
```

Invocation in HA:

```yaml
shell_command:
   play_thankyou_for_mail: >-
        /bin/bash -c "file=$(find /config/mp3 -name 'thankyou-*.al' | shuf -n 1); echo $file; /usr/bin/curl -vvv -F "file=@$file;type=Audio/G.711A" -H "Content-Type: Audio/G.711A" "http://u:p@10.6.0.12/cgi-bin/audio.cgi?action=postAudio&httptype=singlepart&channel=1" --speed-limit 1 --speed-time 1;"
   play_welcome: >-
        /bin/bash -c "file=$(find /config/mp3 -name 'welcome-*.al' | shuf -n 1); echo $file; /usr/bin/curl -vvv -F "file=@$file;type=Audio/G.711A" -H "Content-Type: Audio/G.711A" "http://u:p@10.6.0.12/cgi-bin/audio.cgi?action=postAudio&httptype=singlepart&channel=1" --speed-limit 1 --speed-time 1;"
```

### 6. Create automation 

Add automation to your ha as usual and call script. Check scripts and automation in example folder.
When someone presses the doorbell, I trigger this script through automation, which interrupts the ringing and plays the greeting.


### Notes
- Remember to adjust the paths in the above commands to your specific directory structure.
