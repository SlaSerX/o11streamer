# o11 OTT Streamer v2

## Prerequisites

- Linux server running Ubuntu 20.04 LTS (64-bit)
- Access to a terminal with root permissions

## Installation

### Step 1: Download the Archive

Download the `o11_OTT_Streamer_v2.zip` archive from this secure link:

🔒 [Secure Download Link](https://drive.proton.me/urls/5YB8XBZGFC#y7c94Y4d45nb)

### Step 2: Upload to Your Server

Upload the `o11_OTT_Streamer_v2.zip` archive to your Linux server.

### Step 3: Unzip and Install

\`\`\`bash
cd /path/to/folder
unzip o11_OTT_Streamer_v2.zip
chmod +x install.sh
./install.sh
\`\`\`

### Step 4: Start the Service

\`\`\`bash
sudo systemctl start o11OTTStreamer
sudo systemctl stop o11OTTStreamer
sudo systemctl reload o11OTTStreamer
\`\`\`

## Usage

Navigate to `http://<your-server-ip>:8000`.


Description

O11 purpose is to convert LIVE and VOD streams to a friendly protocol such as HLS and/or direct http MpegTS.

The following input formats are currently supported

    DASH
    HLS
    Microsoft Smooth Streaming

Dependencies

    FFmpeg (HLS FMp4 output does not require it though)

Executing program

You can list command line options using ./o11 -h

  Provider selection page

This page is the default one, it’s a clickable list of the available providers.

To setup a provider logo, create a png file in logos/ which must be named [providerid].png. If not forced using ForcedId in the provider config, the provider Id is the provider name with lower case, no space and only alphanumericable chars (example: My Provider becomes myprovider)

  Channels monitoring page

The FILTER select box allows to select the type of streams to filter.

Click on a stream to display it in the internal player (it must be already in running state or the OnDemand option should be selected). Note that playback will mostly fail under Chrome is the audio track is AC3 or the video track is not H264.

  Providers and streams configuration page

Providers

O11 supports multiple providers. Each provider will have its own set of streams and its own network configuration.

Go to Config tab, select a provider name and click on NEW PROVIDER. Provider configuration files will be stored into providers/ by default. This path can be changed using -providers [path] from command-line.

Clicking on RESCAN PROVIDERS will trigger a full scan of the providers/ directory for new provider configuration files.

The DELETE button will delete the currently selected provider. Its configuration file will only be deleted after clicking on SAVE ALL CONFIGS.

You can export a selection of streams or a full provider’s configuration using EXPORT SELECTION or EXPORT PROVIDER.

You can also import a new proviser using the IMPORT button.
Channels

To add a new channel to a provider, ho to Config tab and press ADD STREAM after entering its name in the field.

To delete a stream, click on its checkbox and press DELETE SELECTION. If the stream was running, it will be stopped first, then deleted.
Stream configuration

    START/STOP: to start or stop the stream.
    Name: the name of the stream. This is also a clickable link that will start the internal player. You can also copy this link (HLS) to use in another player.
    Status: the status of the stream.
    Mode: Live, Replay or VOD. Live mode is for a live channel. Replay will trigger a fetch of all the fragments from the manifest and generate a HLS output stream. VOD can be used to get VOD streamd and generate mp4 file.
    Manifest: setup the manifest url and/or manifest script used to get a dynamic manifest (the SessionManifest option must be selected, please check the Scripts section for more details). Network ovverride fields will also appear in the box when the NetworkOverride option is selected. Press the R button to parse the manifest and refresh the tracks list.
    Tracks: list of selectable Video/Audio/Subtitles tracks. Only one Video track can be selected. The Best quality option is seletected by default and let O11 take the best resolution/bandwidth/framrate. Multiple audio and subtiltes tracks can be selected.
    Keys or CDM Script: set the decryption keys for the stream. On line per kid:key. If UseCdm, O11 will try to call the selected Cdm script to get keys. Please check the Scripts section for more detail.
    Options: please see the Stream options section.

Stream options

Several options can be selected for each stream.

    Autostart: auto start the stream when starting O11. It is possible to override this option globally using the -noautostart command-line option.
    Autoreplay: when a Live stream reaches the end of the broadcast, switch mode to Replay and record the whole stream.
    OnDemand: automatically start a stream when the internal or a remote player request for it. The stream is then automatically stops when it is not used anymore. Use this option to avoid having all streams running for nothing 24/24.
    KeepFiles: don’t delete unused segments (mostly for debugging).
    SessionManifest: use a session manifest script to automatically get the manifest URL. Usefull when the manifest is changing (check the Scripts paragraph for more details).
    SpeedUp: run all segments download in parallel. It speeds up OnDemand more start.
    NetworkOverride: add new fields to override the global network configuration (check the Scripts paragraph for more details).
    IgnoreUpdate: don’t autoremove/modify this channel when using the channels or events script (check the Scripts section for more details).
    FixIvSize: force decruption IV size to be on 16 bytes.
    TimeRange: select a time range to extract from a manifest. Only works with DASH using times and not numbers.

Network configuration

Network options can be configured globally for a provider and then overriden per stream.

    Use for media files: also use the following network configuration for media (fragments) files download. If not checked, only the manifest will use the network configuration.
    HTTP Proxy: coma separated list of proxies. Format is http://user:pass@ip:port
    User Agent: User Agent to use for HTTP connections.
    Bind: coma separated list of ips or interface names.
    DNS over HTTPS: coma separated list of DOH servers.

All these network parameters can be overriden per channel, selecting the NetworkOverride option.

    Manifest Param: global list of parameters to add to manifest script call (param1 param2…).

    Cdm Param: global list of parameters to add to cdm script call (param1 param2…).

    Max concurrency: no limit (0), by default. Else limit the maximum number of streams that can run per provider.

Scripts

Scripts play a big role for O11. Each provider should have each own script. It can be either a shell script (.sh) or a python3 script (.py).
When a script is configured, only it’s name body must be specify. O11 will first try to start the .sh extension and then .py one if not available.
Scripts must be stored in the scripts/ directory.

Let say we have a provider call provider1 which script name is provider.py, stored in scripts/provider.py.
This script must then be referenced as provider1 param1 param2 … inside the ManifestScript/CdmScript/EventsScript/ChannelsScript fields.

Depending on the type of script, it will automatically receive the following parameters, additionnaly to the ones configured by user.

When called, a script will receive, in addition to user selected ones, the following parameters:

    Common to all scripts: bind=[bind addresses] proxy=[proxy urls] doh=[doh urls] action=[action type]. Action type can be one of: manifest, cdm, events, channels, heartbeat.
    Manifest script: action=nanifest
    Cdm script: action=cdm drm=[widevine or playready] pssh=[pssh used to extract needed keys]
    Events: action=events
    Channels: action=channels
    Heartbeat: action=heartbeat heartbeaturl=[heartbeat url returned by manifest script] heartbeatparams=[heartbeat parms returned by manifest script]

An example of script can be found in scripts/example.py. It explains how to manage all actions.
Manifest script specifics

A manifest script should return a json formatted as followed:

{
    "ManifestUrl" : [manifest url],
    "Headers": {
	"Manifest": {
	    'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/107.0.0.0 Safari/537.36',
	    'Custom-Header': 'It's me',
	    ...
	},
	"Media": {
	    'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/107.0.0.0 Safari/537.36'
	    'Custom-Header': 'It's me',
	    ...
	}
    },
    "Heartbeat": {
	"Url" : [heartbeat url],
	"Params": [param1,param2,...],
	"PeriodMs" : 5*60*1000
    }
}
		

Cdm script specifics

A CDM script should return a list of kid:key formatted as followed:

kid1:key1
kid2:key2
...
		

Events/Channels script specifics

A events or channels script should return a list of streams following the same format as the config files:

{
  "[Channels or Events]": [
    {
      "Name": [name]",
      "ForcedId": "stream%auto%",
      "Mode": "live",
      "SessionManifest": true,
      "ManifestScript": "provider1 id=123 country=fr",
      "CdmType": "widevine",
      "UseCdm": true,
      "Cdm": "provider1 id=123 country=fr",
      "Video": "best",
      "OnDemand": true,
      "TsHls": true,
      "SpeedUp": true
    },
    {
      "Name": [name]",
      "ForcedId": "stream%auto%",
      "Mode": "live",
      "SessionManifest": true,
      "ManifestScript": "provider1 id=123 country=fr",
      "CdmType": "widevine",
      "UseCdm": true,
      "Cdm": "provider1 id=123 country=fr",
      "Video": "best",
      "OnDemand": true,
      "TsHls": true,
      "SpeedUp": true
    },
    ...
   ]
}
		

You will notice the "ForcedId": "stream%auto%", field that uses the special placeholed %auto%.
This placeholder will be automatically replaced by an incremental number by O11.

  Users configuration page

This page is used to manage users access.

When adding a new user, the following options are available:

    Password: the password for the selected user.
    Providers: the list of providers the user has access to. If the is Admin option is seletected, the user has access to all providers
    Networks: allowed network for user. If nothing is selected, the user can access O11 from any ip.
    Is Admin: set user as admin. Only admin users can access the Users and Config pages.


  Streams recording management page

  Playlist including channels available to user
