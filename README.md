# captureSystemAudio
Capture system audio (["What-U-Hear"](https://wiki.archlinux.org/index.php/PulseAudio/Examples#ALSA_monitor_source)) 

> To be able to record from a monitor source (a.k.a. "What-U-Hear", "Stereo Mix"), use `pactl` list to find out the name of the source in PulseAudio (e.g.  alsa_output.pci-0000_00_1b.0.analog-stereo.monitor`). 
<h5>Background</h5>

- https://lists.w3.org/Archives/Public/public-speech-api/2017Jun/0000.html
- https://github.com/whatwg/html/issues/2823
- https://github.com/guest271314/SpeechSynthesisRecorder/issues/14
- https://github.com/WICG/native-file-system/issues/72
- https://github.com/WICG/native-file-system/issues/97
- https://github.com/w3c/mediacapture-main/issues/629
- https://github.com/WICG/speech-api/issues/69
- https://github.com/web-platform-tests/wpt/issues/23084
- https://github.com/w3c/mediacapture-main/issues/650
- https://github.com/w3c/mediacapture-main/issues/654
- https://gist.github.com/guest271314/59406ad47a622d19b26f8a8c1e1bdfd5
- https://github.com/guest271314/requestNativeScripts
- https://github.com/web-platform-tests/wpt/issues/23084
- https://github.com/w3c/mediacapture-screen-share/issues/140

<h5>Motivation</h5>

Specify and implement web compatible system audio capture, e.g., from `window.speechSynthesis.speak()`, `ffplay blade_runner.mkv`, `<video>` playing in an HTML document, `mpv output.amr sound.caf`.

<h5>Synopsis</h5>

Open local files watched by `inotifywait` from [inotify-tools](https://github.com/inotify-tools/inotify-tools) to capture system audio monitor device at Linux, write output to a local file, stop system audio capture, get the resulting local file in the browser.

<h5>Dependencies</h5>

`pacat`, `inotify-tools`.

<h6>Optional</h6>

`opus-tools`, `mkvtoolnix`, `ffmpeg` or other native code application or to convert WAV to Opus or different codec and write track to Matroska, WebM, or other media container supported at the system. `opus-tools`, `mkvtoolnix` are included in the code by default to reduce resulting file size of captured stream by converting to Opus codec from audio from WAV, written to WebM container for usage with `MediaSource`. TODO: [Pipe captured audio to WebRTC `MediaStreamTrack` instead of local file](https://github.com/guest271314/captureSystemAudio/projects/1).

<h5>Usage</h5>

<h6>Command line, Chromium launcher</h6>

Create a local folder in `/home/user/`, `localscripts` containing the files in this repository, `cd` to the folder and, or create executable to run the command

`$HOME/notify.sh & chromium-browser --enable-experimental-web-platform-features <Chromium flags> && killall -q -9 inotifywait`

To start system audio capture at the browser open the local file `captureSystemAudio.txt`, to stop capture by open the local file `stopSystemAudioCapture.txt`, where each file contains one space character, then get the captured audio from local filesystem using `<input type="file">` or where implemented Native File System `chooseFileSystemEntries()`.

```
captureSystemAudio()
.then(async requestNativeScript => {
  // system audio is being captured, wait 10 seconds
  await requestNativeScript.get('wait')(10000);
  // stop system audio capture
  await requestNativeScript.get('stop').arrayBuffer(); 
  // avoid Native File System ERR_UPLOAD_FILE_CHANGED error
  let output;
  // can be executed thousands of times
  FILE_EXISTS: do {
    try {
      if (output = await requestNativeScript.get('dir').getFile('output.webm', {create:false})) {
        // wait 50 milliseconds again here to avoid File size 0 reference collision 
        // where getFile() executed exactly at creation, open of file at native code
        await requestNativeScript.get('wait')();
        break FILE_EXISTS;
      };
      // function returning a Promise after default parameter 50, passed to setTimeout()
      requestNativeScript.get('wait')();
      // handle DOMException: A requested file or directory could not be found at the time an operation was processed.
    } catch (e) {
      console.error(e);
    }
  } while (!output);
  // resulting File object
  // potentially only file metadata reference, not reference to underlying content of file now
  const file = await output.getFile(); 
  const type = file.type;
  // read file to get underlying content instead of file metadata reference
  // store file as ArrayBuffer in memory, alternatively use file.stream() read/write File then remove
  const ab = await file.arrayBuffer();
  // remove file containing captured audio from local filesystem
  await requestNativeScript.get('dir').removeEntry('output.webm');
  // do stuff with captured system audio as WAV, Opus, other codec and container the system supports
  console.log(output, file, URL.createObjectURL(new Blob([ab], {type})));
})
.catch(e => console.error(e));
```





