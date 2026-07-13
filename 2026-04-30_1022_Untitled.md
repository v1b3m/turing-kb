Use a **simulated player**, not real video playback.

The clone should keep YouTube’s watch, shorts, and playback-related UI, but no actual media file should stream or decode. The system tracks playback state and time, while the visible player remains a controlled UI surface.

**How It Should Work**  
Each curated video or short has static metadata plus simulated playback state:
- `duration`
- `currentTime`
- `playbackState`: `idle | playing | paused | ended`
- `playbackRate`
- `isMuted`
- `isFullscreen`
- `isTheaterMode`
- optional state for captions, quality, transcript, chapters, queue, and miniplayer

When the agent presses play:
- the UI switches to `playing`
- `currentTime` advances deterministically
- the progress bar, timestamps, chapters, and transcript position update from state
- no real video or audio is played

When the agent pauses, seeks, changes speed, toggles captions, opens transcript, or changes quality:
- only app state changes
- the relevant controls and panels update normally
- no media transport behavior is involved

**What This Enables**  
Yes, this means you can test agents on playback-setting scenarios such as:
- changing playback speed
- toggling captions
- switching quality
- muting and unmuting
- entering theater mode or fullscreen
- seeking to a timestamp
- opening transcript or chapters
- moving the session into miniplayer
- managing queue behavior

These are still valid gym tasks because the agent is being tested on:
- finding the correct controls
- making the correct UI changes
- handling resulting state transitions
- completing multi-step watch-page workflows reliably

**What Stays In Scope**
- watch page
- shorts viewer
- play/pause
- seek bar and timestamps
- transcript panel
- chapters
- miniplayer
- queue
- watch history rules
- share/save/playlist/comment/subscription flows
- playback settings UI

**What Is Out Of Scope**
- real streaming
- buffering/network media behavior
- codec and true quality delivery
- actual audio/video playback
- real caption sync
- true live latency behavior

**Data Strategy**  
Use a **small curated catalog** shared by all users:
- fixed videos
- fixed shorts
- fixed durations
- fixed chapters/transcripts
- fixed recommendations

**Why This Is Good**  
- stable for autonomous agents
- easier to verify and test
- cheaper to run
- avoids licensing and hosting issues
- preserves most of YouTube’s UI value

**Main Limitation**  
Agents will not learn real media-delivery behavior. They will learn playback-related UI and state transitions instead.

**Recommended Scope Rule**  
Keep anything that depends on **player state**.  
Drop anything that depends on **actual media transport**.