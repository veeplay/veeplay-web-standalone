# Veeplay.js
### Javascript SDK for web-based Veeplay implementations

## Installation

Veeplay is available as a JS bundle that can be retrieved from this repository, as well as from the Veeplay CDN:

`https://cdn.veeplay.com/veeplay-web/1.1.15/mp.bundle.js`

Npm package coming soon.

#### Getting an `invalidLicense` error?

Usage under the `localhost` domain is open. To use with a different domain name, go to [your Veeplay panel](https://panel.veeplay.com/) and register a new application.

## Basic usage

See all available [configuration options](https://veeplay.github.io/json-docs/).

````
<div id="player"></div>
<script src="<CDN_URL>"></script>
<script>
(async () => {
    try {
        // instantiate main player object
        const player = new MediaPlayer('player');
        // instantiate a MediaBuilder object used for generating MediaUnits for configuring the main content and any abreaks
        const builder = new MediaBuilder(player);

        await builder.configureFromJSON({
        // Main content unit configuration
        "content": [
            {
            "url": "<MAIN CONTENT URL>",
            "autoplay": true,
            "minimumAdInitialOffset": 0, // minimum offset at which midrolls can start
            "minimumAdFinalOffset": 0, // maximum offset at which midrolls can start
            "minimumAdSpacing": 0, // minimum duration between midroll adbreaks
            "seekAdHandling": "last", // when skipping multiple adbreaks, which adbreak should run
            "controls": {
            }
            }
        ],
        // Ads configuration
        "ads": {
            "adBreaks": {
            // insert a preroll AdBreak. Other possible values include midContent and postContent. Offset is inferred for prerolls and postrolls
            "preContent": [
                {
                // Format "HH:MM:SS"
                "offset": "00:00:05",
                // One or more VAST or VPAID tag URLs
                "urls": [
                    "<TAG_URL>"
                ]
                }
            ]
            },
            // should the ad be skipped if the user clicks through?
            "afterVideoAdTapped": "doNothing",
            "countdown": {
            }
        }
        });

        // Retrieve the generated MediaUnits (the main class for encapsulating each piece of content that needs to be played)
        const units = await builder.mediaUnits();

        // Instruct the player to start playback
        player.playMediaUnits(units);
        player.setMute();
    } catch (error) {
        console.error(error);
    }    
    })();
</script>
````

## Classes and terminology

- `MediaEvent` - Base class for encapsulating player playlist configuration. All objects in the playlist are classes that derive from MediaEvent. This basically provides the infrastructure for encapsulating logic that needs to be executed at some point in the playlist.
- `MediaUnit` - Class derived from `MediaEvent` that represents a single video that needs to be rendered and all associated metadata (for example if the video is an ad, the initial position in the stream, etc)
- `MediaOverlay` - Class derived from `MediaEvent` encapsulating any overlays that need to be displayed on top of the video, such as the controls bar, the clickthrough handler for ads or any banners or images
- `MediaBuilder` - Orchestrator class that helps construct the array of `MediaEvent` derived objects composing a Veeplay playlist. A MediaBuilder instance will use a JSON configuration object to generate MediaUnits and MediaEvents as necessary depending on the main content units as well as any ad tags that are configured, as in the example above.

## Event tracking

The player uses an EventEmitter instance for propagating player events. The instance is unique per each player object and can be accessed using the following method:

`const tracker = player.getEventTracker()`

In order to subscribe to events (player state updates), use the following snippet:

````
tracker.emitter.on('trackedEvent', (e) => {
    if (e.event === EVENTS.PAUSE) {
    console.log('paused');
    } else if (e.event === EVENTS.RESUME) {
    console.log('resumed');
    }
});
````

In order to subscribe to notifications (player UI updates), use the following example:

`tracker.emitter.on(NOTIFICATIONS.ENTER_FULLSCREEN, () => console.log('fullscreen mode on'));`

The full list of events and notifications supported by the player are exported in the SDK bundle, alongside the MediaPlayer and MediaBuilder classes:

`EVENTS` :
````
CLICK: "click" // an ad was clicked
CLOSE: "close" // user stopped playback before completion
CLOSE_LINEAR: "closeLinear" // synonym for CLOSE, part of the IAB VAST spec
COLLAPSE: "collapse" // a VPAID unit was collapsed
COMPLETE: "complete" // the MediaUnit playback was completed succesfully
CREATIVE_VIEW: "creativeView" // a VAST creative was displayed
ERROR: "error" // playback error or VAST tracking error
EXIT_FULLSCREEN: "exitFullscreen" // fullscreen was disabled
EXPAND: "expand" // a VPAID ad was expanded
FINISH: "finish" // a MediaUnit has ended and is being cleared from the player
FORWARD: "forward" // seeking forward
FULLSCREEN: "fullscreen" // entering fullscreen
ICON_VIEW: "iconView" // a VAST icon creative was displayed
IMPRESSION: "impression" // a VAST ad has been displayed
IMPRESSION_NOT_VIEWABLE: "notViewable" // the VAST ad is not visible on screen
IMPRESSION_UNDETERMINED: "viewUndetermined" // VAST ad Viewability could not be determined
IMPRESSION_VIEWABLE: "viewable" // the VAST ad is currently visible on screen
LAUNCH: "launch" // a new MediaUnit is starting playback
MUTE: "mute" // player was muted
PAUSE: "pause" // player was paused
RESUME: "resume" // playback was resumed from a paused state
REWIND: "rewind" // seeking backwards in the stream
SEEKED: "seeked" // a seek was performed
SKIP: "skip" // an ad was skipped by the user
START: "start" // an ad has started playing
UNMUTE: "unmute" // player was unmuted
UPDATE: "update" // emitted every second the player is rendering a stream, unless playback is paused.
````

`NOTIFICATIONS`

````
CONTROLS_DISPLAYED: "controlsDisplayed" // the playback control bar has been shown on screen
CONTROLS_HIDDEN: "controlsHidden" // the playback control bar has been hidden
DURATION_AVAILABLE: "durationAvailable" // the duration of the currently playing stream is now available
ENTER_FULLSCREEN: "enterFullscreen" // fullscreen was entered
ERROR: "error" // a playback or VAST error has been encountered
EXIT_FULLSCREEN: "exitFullscreen" // fullscreen mode was exited
INVALID_LICENSE: "invalidLicense" // the Veeplay SDK was not able to verify a valid license for this application
LOAD_STATE_CHANGED: "loadStateChanged" // the underlying video renderer has updated it's progress in loading the requested stream
PLAYBACK_DID_FINISH: "playbackFinished" // playback ended for any reason
PLAYBACK_STATE_CHANGED: "playbackStateChanged" // playback state changed between idle / playing / paused / seeking
PLAYER_MOUSE_MOVE: "playerMouseMove" // the player registered a mouse hover or a touch move event over it's container
PLAYER_TAPPED: "playerTapped" // the player surfaced was tapped or clicked
PLAYER_UPDATE: "playerUpdate" // this event is fired every second while playback is active
PLAYLIST_FINISH: "playlistFinish" // the full array of MediaUnits was processed
STATUS_CHANGED: "statusChanged"
TOGGLE_FULLSCREEN: "toggleFullscreen" // fullscreen mode was toggled
TRACKED_EVENT: "trackedEvent" // a VAST tracking event was fired
UNIT_FINISHED: "unitFinished" // a MediaUnit has finished playback and is currently being cleared (regardless of the finish reason)
VOLUME_CHANGED: "volumeChanged" // the system audio volume was updated
````

## Interacting with the player object

- `resetPlayer()`
Stops playback and unloads the MediaUnit
- `pause()`
Pause playback
- `play()`
Resume playback
- `stop()`
Unload the current MediaUnit and stop processing the playlist.
Will signal that the user exited playback before completion.
- `skip()`
Skip to the next MediaUnit in the playlist
- `next()`
Trigger the next MediaUnit in the playlist without signaling a skipped MediaUnit
- `previous()`
Unload current MediaUnit and load the previous MediaUnit in the playlist
- `setMediaUnits(units)`
Overwrites the current MediaUnit playlist. Takes an array of MediaUnit or MediaEvent objects.
- `loadState()`
   * Returns the current state of the media loaded into the underlying video renderer.
   * Returned value will be a member of the constants.LOAD_STATE dictionary.
   * Possible values are UNKNOWN / PLAYABLE / PLAYTHROUGH_OK
- `playbackState()`
   * Returns the current playback state of the underlying video renderer.
   * Returned value will be a member of the constants.PLAYBACK_STATE dictionary.
   * Possible values are INTERRUPTED / PAUSED / SEEKING / STOPPED / PLAYING
- `duration()`
Retrieve the duration of the currently playing stream
- `playableDuration()`
Retrieve the currently buffered duration
- `showActivityIndicator()`
Display the loading animation
- `hideActivityIndicator()`
Hide the loading animation
- `isStreamingLive()`
Return a boolean value stating if the current stream is VOD or Livestream
- `setVolume(volume)`
   * Update the player volume. Takes a float number between 0 and 1.
   * 0 - Muted
   * 1 - Set volume at 100%
- `getVolume()`
Retrieve the current player volume. Will return a float between 0 and 1
- `setMute(bool)`
Mute or unmute the player based on the parameter value (true/false)
- `getMute()`
Will return a boolean representing whether the player is currently muted.
- `enterFullscreen()`
Switch to fullscreen mode
- `exitFullscreen()`
Switch to inline mode
- `toggleFullscreen()`
Toggle between fullscreen and inline mode
- `currentPlaybackTime()`
Return the current playback position
- `getEventTracker()`
Return the `EventTracker` object associated with the player instance
- `audioTracks()`
Return a list of AudioTrack objects associated with current video. Set the `enabled` attribute to true on any AudioTrack to switch between multiple audio tracks.

## Custom overlays

After you included mp.bundle.js in your webpage create a subclass of `OverlayController` and implement the `load` method:

````
class CustomOverlayController extends OverlayController {
    load() {
        // Set the overlay's width and height
        this.overlay.width = 'auto';
        this.overlay.height = 'auto';

        // Create an image
        this.imageView = document.createElement('img');
        this.imageView.src = 'https://veeplay.com/wp-content/themes/veeplay/images/logo_veeplay.png';

        // Add the image to the overlay's element
        this.view.appendChild(this.imageView);
    }
}
````

Register your custom overlay controller:
````
player.controllerRegistry.registerOverlayController(CustomOverlayController, 'customc');
````
where `customc` is an identifier to be used as your overlay "type".

## Adding additional HTTP/S headers to HLS HTTP requests

In some cases, you might want to add some additional HTTP headers to the requests made for retrieving HLS manifests or .ts chunks.

In order to do so, pass an `options` dictionary when constructing the player object, like so:

````
const options = {
    hlsHeaders: {
        'X-USES-VEEPLAY': 'true',
        'X-TEST-HEADER': 'value',
    }
}
const player = new MediaPlayer('player', options);
````

Notes:
- the CORS configuration on the server needs to allow these custom headers as well.
- the headers are only applied for HLS requests, HTTP requests for MP4 streams or VAST tags will not be affected by this setting.


## Google IMA SDK integration

If you'd like to use Google's IMA SDK instead of the internal VAST support, add a script tag pointing to the official JavaScript library:
````
<script type="text/javascript" src="//imasdk.googleapis.com/js/sdkloader/ima3.js"></script>
````

then set to `true` the `preferGoogleIma` property on the `MediaPlayer` instance your using, as soon as possible after instantiaton.
