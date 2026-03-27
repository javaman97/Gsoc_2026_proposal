# GSoC 2026 : Integrate Remote Playback and Listening Now

## Contact Information

| Field | Details |
|-------|---------|
| **Name** | Aman Nishad |
| **IRC nick** | javaman97 |
| **GitHub** | [javaman97](https://github.com/javaman97) |
| **Email** | [javaman0512@gmail.com](mailto:javaman0512@gmail.com) |
| **LinkedIn** | [Aman Nishad](https://www.linkedin.com/in/java-aman/) |
| **Time Zone** | UTC+05:30 |

I am a first-year student at **Sikkim Manipal University**, currently pursuing a **Master of Computer Applications (MCA)**. I developed an interest in Android development during my Bachelor of Computer Applications and have since built applications that address real-world problems and deliver meaningful user experiences.

---

## Project Overview

**ListenBrainz Android** is an open-source, privacy-focused app that lets users track their music listening habits. The app ships three independent playback subsystems — **BrainzPlayer** (local playback), **Remote Playback** (Spotify SDK + YouTube), and **Listening Now** (real-time track display from server) — but they operate in complete isolation.

The Scaffold front layer only surfaces BrainzPlayer. Remote playback ends after a single track. Listening Now isn't wired into the UI at all. This project unifies all three behind a single state machine in a new **Kotlin Multiplatform module**, revamps the Scaffold front layer, and enables continuous playlist playback through Spotify and YouTube.

### My Contributions

I have been a contributor to ListenBrainz since **January 2026** and have worked on implementing the migration of **compose-rating bar**, Android logger to Kermit, compatible with Compose Multiplatform along with reporting and fixing some bugs.

- [My PRs](https://github.com/metabrainz/listenbrainz-android/pulls?q=is%3Apr+author%3Ajavaman97)
- [My commits](https://github.com/metabrainz/listenbrainz-android/commits/cmp?author=javaman97)

---

## Problem Statement — Current State

| Area | Problem | Existing Code |
|------|---------|---------------|
| Scaffold front layer | Hard-coded to `BrainzPlayerViewModel` in `ui/screens/main/` — Listening Now and Remote Playback are invisible here | [`MainActivity.kt`](https://github.com/metabrainz/listenbrainz-android/blob/main/app/src/main/java/org/listenbrainz/android/ui/screens/main/MainActivity.kt) builds the Scaffold and passes `BrainzPlayerViewModel` directly to the front layer. The [`BrainzPlayerBackLayerContent`](https://github.com/metabrainz/listenbrainz-android/blob/main/app/src/main/java/org/listenbrainz/android/ui/screens/brainzplayer/BrainzPlayerBackLayerContent.kt) composable is the only playback surface wired to the Scaffold — no equivalent exists for Listening Now or Remote Playback. |
| Listening Now | API endpoint exists in `ListensViewModel` but is never wired to the Scaffold | [`ListensViewModel.kt`](https://github.com/metabrainz/listenbrainz-android/blob/main/app/src/main/java/org/listenbrainz/android/viewmodel/ListensViewModel.kt) calls the Listening Now API via [`ListensRepository`](https://github.com/metabrainz/listenbrainz-android/blob/main/app/src/main/java/org/listenbrainz/android/repository/listens/ListensRepository.kt), but the result is only consumed within the Listens screen — it never reaches the Scaffold front layer or any mini-player surface. |
| Remote Playback | `RemotePlaybackHandlerImpl` supports single-track only (`playOnSpotify(trackName, artistName)`); no queue, no background continuity | [`RemotePlaybackHandlerImpl.kt`](https://github.com/metabrainz/listenbrainz-android/blob/main/app/src/main/java/org/listenbrainz/android/repository/remoteplayer/RemotePlaybackHandlerImpl.kt) exposes `playOnSpotify()` and `playOnYoutube()` — both fire-and-forget single-track calls. There is no queue model, no track-ended callback, and no mechanism to advance to the next track. The [`RemotePlaybackHandler`](https://github.com/metabrainz/listenbrainz-android/blob/main/app/src/main/java/org/listenbrainz/android/repository/remoteplayer/RemotePlaybackHandler.kt) interface defines only these single-track methods. |
| State conflicts | No arbiter: BrainzPlayer and Listening Now can produce conflicting "now playing" signals simultaneously | [`BrainzPlayerServiceConnection.kt`](https://github.com/metabrainz/listenbrainz-android/blob/main/app/src/main/java/org/listenbrainz/android/service/BrainzPlayerServiceConnection.kt) emits player state independently of [`ListensViewModel`](https://github.com/metabrainz/listenbrainz-android/blob/main/app/src/main/java/org/listenbrainz/android/viewmodel/ListensViewModel.kt), which also provides "now playing" data. No shared state machine arbitrates between them — both can claim the "now playing" slot simultaneously. |

---

## Deliverables Summary

1. **`playback-shared` KMP module** — `PlaybackStateManager`, `PlaybackState`, `PlaybackAction` models, `expect` repository interfaces, Android `actual` implementations, iOS no-op stubs.
2. **`AndroidDeviceDetection`** — local vs. remote device resolution via [`ListenSubmissionService`](https://github.com/metabrainz/listenbrainz-android/blob/main/app/src/main/java/org/listenbrainz/android/service/ListenSubmissionService.kt) and [`ListenServiceManager`](https://github.com/metabrainz/listenbrainz-android/blob/main/app/src/main/java/org/listenbrainz/android/service/ListenServiceManager.kt), 10s detection window.
3. **`ListeningNowSocketRepository`** — real-time WebSocket Listening Now events (extending the existing [`repository/socket/`](https://github.com/metabrainz/listenbrainz-android/tree/main/app/src/main/java/org/listenbrainz/android/repository/socket) package) with REST polling fallback via [`ListensService`](https://github.com/metabrainz/listenbrainz-android/blob/main/app/src/main/java/org/listenbrainz/android/service/ListensService.kt).
4. **`RemotePlaybackQueue` + `RemoteQueuePlayer`** — continuous multi-track queue wrapping [`RemotePlaybackHandlerImpl`](https://github.com/metabrainz/listenbrainz-android/blob/main/app/src/main/java/org/listenbrainz/android/repository/remoteplayer/RemotePlaybackHandlerImpl.kt) for Spotify and YouTube with SDK-driven auto-advance, Room persistence (alongside the existing [`AppDatabase`](https://github.com/metabrainz/listenbrainz-android/blob/main/app/src/main/java/org/listenbrainz/android/model/AppDatabase.kt)), and foreground service background continuity.
5. **`UnifiedPlaybackFrontLayer`** — animated Scaffold front layer rendering the correct mini-player for each `PlaybackState`, replacing the current hard-coded BrainzPlayer front layer in [`MainActivity.kt`](https://github.com/metabrainz/listenbrainz-android/blob/main/app/src/main/java/org/listenbrainz/android/ui/screens/main/MainActivity.kt). Existing `BrainzPlayerMiniPlayer` preserved unchanged.
6. **`UnifiedPlaybackViewModel`** — single state owner combining all input streams via `combine`, replacing the current split between [`BrainzPlayerViewModel`](https://github.com/metabrainz/listenbrainz-android/blob/main/app/src/main/java/org/listenbrainz/android/viewmodel/BrainzPlayerViewModel.kt) and [`ListensViewModel`](https://github.com/metabrainz/listenbrainz-android/blob/main/app/src/main/java/org/listenbrainz/android/viewmodel/ListensViewModel.kt) for playback state.
7. **Full test suite** — `commonTest` unit tests (kotlin.test + Turbine), `androidTest` integration tests, Compose UI tests, ≥ 90% coverage via JaCoCo + Kover.

---

## Technical Design
![end_to_end_data_flow](https://github.com/user-attachments/assets/a5efbe6f-eab3-4e9c-b906-65c9404823ab)



### 1. KMP `playback-shared` Module

#### Module Layout

The new module sits alongside the existing [`shared/`](https://github.com/metabrainz/listenbrainz-android/tree/main/shared) KMP module on the `cmp` branch:

```
playback-shared/
  src/
    commonMain/kotlin/org/listenbrainz/playback/
      PlaybackStateManager.kt              ← pure state machine
      models/
        PlaybackState.kt
        PlaybackAction.kt
        Track.kt
        Listen.kt
        PlaybackSource.kt                  // BRAINZ | SPOTIFY | YOUTUBE | OTHER
        RemoteService.kt                   // SPOTIFY | YOUTUBE_MUSIC
      repository/
        PlaybackRepository.kt              // expect
        DeviceDetectionRepository.kt       // expect
        RemoteQueueRepository.kt           // expect
      utils/
        StateResolver.kt

    androidMain/kotlin/org/listenbrainz/playback/
      AndroidPlaybackRepository.kt         // actual — wraps BrainzPlayerServiceConnection
      AndroidDeviceDetection.kt            // actual — wraps ListenServiceManager
      AndroidRemoteQueueRepository.kt      // actual — wraps RemotePlaybackHandlerImpl

    iosMain/kotlin/org/listenbrainz/playback/
      IOSPlaybackRepository.kt             // stub actual — compiles, no-op, post-GSoC impl
      IOSDeviceDetection.kt                // stub actual — compiles, no-op, post-GSoC impl
      IOSRemoteQueueRepository.kt          // stub actual — compiles, no-op, post-GSoC impl

    commonTest/kotlin/
      PlaybackStateManagerTest.kt
      StateResolutionTest.kt

    androidTest/kotlin/
      DeviceDetectionIntegrationTest.kt
      RemoteQueueIntegrationTest.kt
```

The `expect/actual` pattern is the key architectural guarantee: `commonMain` contains zero Android or iOS imports, so the state machine is fully portable. Android implementations wrap existing services — [`BrainzPlayerServiceConnection`](https://github.com/metabrainz/listenbrainz-android/blob/main/app/src/main/java/org/listenbrainz/android/service/BrainzPlayerServiceConnection.kt), [`ListenServiceManager`](https://github.com/metabrainz/listenbrainz-android/blob/main/app/src/main/java/org/listenbrainz/android/service/ListenServiceManager.kt), and [`RemotePlaybackHandlerImpl`](https://github.com/metabrainz/listenbrainz-android/blob/main/app/src/main/java/org/listenbrainz/android/repository/remoteplayer/RemotePlaybackHandlerImpl.kt) — without modifying them. iOS stubs satisfy the `expect` contract so the module compiles for iOS on the `cmp` branch immediately.

#### Unified State — Sealed Class

```kotlin
// commonMain — PlaybackState.kt

sealed class PlaybackState {

    object Idle : PlaybackState()

    data class BrainzPlayerActive(
        val song: Song,
        val queue: List<Song>,
        val isPlaying: Boolean,
        val progress: Float,           // 0f..1f
    ) : PlaybackState()

    data class ListeningNowActive(
        val listen: Listen,
        val source: PlaybackSource,
        val isLocalDevice: Boolean,
        val remoteQueue: List<Listen>? = null,
        val timestamp: Long,
    ) : PlaybackState()

    data class RemotePlaybackActive(
        val service: RemoteService,
        val track: Track,
        val queue: List<Track>,
        val isPlaying: Boolean,
        val progress: Float,
    ) : PlaybackState()
}
```

#### Actions

```kotlin
// commonMain — PlaybackAction.kt

sealed class PlaybackAction {
    object PlayPause      : PlaybackAction()
    object SkipNext       : PlaybackAction()
    object SkipPrevious   : PlaybackAction()
    data class Seek(val position: Float)           : PlaybackAction()
    data class PlayQueue(
        val tracks: List<Track>,
        val service: RemoteService,
    )                                              : PlaybackAction()
    object StopRemote     : PlaybackAction()
}
```

#### State Resolution — Pure Function

`PlaybackStateManager.resolve()` is a pure function with no Android dependencies — fully testable in `commonTest`. All input streams are serialised onto a single `Mutex`-guarded coroutine to prevent race conditions when two sources update simultaneously.

```kotlin
// commonMain — PlaybackStateManager.kt

class PlaybackStateManager {

    fun resolve(
        brainzPlayerState: BrainzPlayerInputState?,
        listeningNow: ListeningNowEvent?,
        isLocalDevice: Boolean,
        remotePlayback: RemotePlaybackInputState?,
    ): PlaybackState = when {

        // Priority 1: BrainzPlayer is playing → always wins
        brainzPlayerState?.isPlaying == true ->
            PlaybackState.BrainzPlayerActive(
                song      = brainzPlayerState.song,
                queue     = brainzPlayerState.queue,
                isPlaying = true,
                progress  = brainzPlayerState.progress,
            )

        // Priority 2: Listening Now event present
        listeningNow != null -> when {

            // 2a. Local device + remote playback active → show queue
            isLocalDevice && remotePlayback != null ->
                PlaybackState.RemotePlaybackActive(
                    service   = remotePlayback.service,
                    track     = remotePlayback.currentTrack,
                    queue     = remotePlayback.queue,
                    isPlaying = remotePlayback.isPlaying,
                    progress  = remotePlayback.progress,
                )

            // 2b. Local device, remote playback not yet started
            isLocalDevice ->
                PlaybackState.ListeningNowActive(
                    listen        = listeningNow.listen,
                    source        = listeningNow.source,
                    isLocalDevice = true,
                    remoteQueue   = null,
                    timestamp     = listeningNow.timestamp,
                )

            // 2c. Remote device → read-only, no playlist shown
            else ->
                PlaybackState.ListeningNowActive(
                    listen        = listeningNow.listen,
                    source        = listeningNow.source,
                    isLocalDevice = false,
                    timestamp     = listeningNow.timestamp,
                )
        }

        // Priority 3: Nothing active
        else -> PlaybackState.Idle
    }
}
```
![state_resolution_flowchart](https://github.com/user-attachments/assets/6bc12910-179c-41da-aeb9-5963b89e8f23)


---

### 2. Listening Now + Device Detection
![listening_now_device_detection_flow](https://github.com/user-attachments/assets/6801da3e-6c40-477b-9f88-a2a505619051)



#### WebSocket Integration

The existing [`repository/socket/`](https://github.com/metabrainz/listenbrainz-android/tree/main/app/src/main/java/org/listenbrainz/android/repository/socket) package already handles WebSocket connections — specifically, [`SocketRepository`](https://github.com/metabrainz/listenbrainz-android/blob/main/app/src/main/java/org/listenbrainz/android/repository/socket/SocketRepository.kt) manages connection lifecycle and [`SocketRepositoryImpl`](https://github.com/metabrainz/listenbrainz-android/blob/main/app/src/main/java/org/listenbrainz/android/repository/socket/SocketRepositoryImpl.kt) provides the concrete implementation. A dedicated `ListeningNowSocketRepository` with REST fallback will be added, building on this existing infrastructure:

```kotlin
// androidMain — ListeningNowSocketRepository.kt

class ListeningNowSocketRepository(
    private val socketRepository: SocketRepository,    // existing interface
    private val listensService: ListensService,        // existing Retrofit service
) {
    val listeningNowFlow: Flow<ListeningNowEvent?> = flow {
        socketRepository.connect(LISTENING_NOW_CHANNEL)
            .onEach { raw -> emit(raw.toListeningNowEvent()) }
            .catch {
                // WebSocket failed — fall back to REST polling every 30s
                // using the existing ListensService endpoint
                while (true) {
                    emit(listensService.getListeningNow().toEvent())
                    delay(30_000)
                }
            }
            .collect()
    }.shareIn(scope, SharingStarted.WhileSubscribed(), replay = 1)
}
```

#### Device Detection via Listen Submission Service

[`ListenSubmissionService`](https://github.com/metabrainz/listenbrainz-android/blob/main/app/src/main/java/org/listenbrainz/android/service/ListenSubmissionService.kt) extends `NotificationListenerService` and captures metadata from all media notifications on this device. It delegates to [`ListenServiceManager`](https://github.com/metabrainz/listenbrainz-android/blob/main/app/src/main/java/org/listenbrainz/android/service/ListenServiceManager.kt), which processes the raw notification data and maintains a record of recently submitted listens. Together, they form a ground-truth log of what this specific device has submitted — which we use for device detection:

```kotlin
// androidMain — AndroidDeviceDetection.kt

class AndroidDeviceDetection(
    private val listenServiceManager: ListenServiceManager,  // existing service manager
) : DeviceDetectionRepository {

    /**
     * Compares a Listening Now event against recent submissions tracked by
     * ListenServiceManager. If the track/artist match within a 10s window,
     * the event originated from this device.
     */
    override fun isLocalDevice(event: ListeningNowEvent): Boolean {
        val recentSubmissions = listenServiceManager.getRecentSubmissions()
        return recentSubmissions.any { sub ->
            sub.trackName.equals(event.listen.trackName, ignoreCase = true) &&
            sub.artistName.equals(event.listen.artistName, ignoreCase = true) &&
            abs(sub.timestamp - event.timestamp) < DETECTION_WINDOW_MS
        }
    }

    companion object {
        private const val DETECTION_WINDOW_MS = 10_000L
    }
}
```

The `expect` interface in `commonMain`:

```kotlin
// commonMain — DeviceDetectionRepository.kt

expect interface DeviceDetectionRepository {
    fun isLocalDevice(event: ListeningNowEvent): Boolean
}
```

---

### 3. Remote Playback Queue + Background Continuity
![remote_playback_queue_flow](https://github.com/user-attachments/assets/15ba7531-a521-4acd-8e61-79584c140be6)

The current [`RemotePlaybackHandler`](https://github.com/metabrainz/listenbrainz-android/blob/main/app/src/main/java/org/listenbrainz/android/repository/remoteplayer/RemotePlaybackHandler.kt) interface defines only single-track methods: `playOnSpotify(trackName, artistName)` and `playOnYoutube(trackName, artistName)`. The implementation in [`RemotePlaybackHandlerImpl`](https://github.com/metabrainz/listenbrainz-android/blob/main/app/src/main/java/org/listenbrainz/android/repository/remoteplayer/RemotePlaybackHandlerImpl.kt) uses `SpotifyAppRemote` for Spotify playback and a YouTube WebView/IFrame for YouTube. This project wraps that existing handler in a queue layer — **without modifying the handler itself** — to enable continuous multi-track playback.

#### PlaybackQueue Model

```kotlin
// commonMain — RemotePlaybackQueue.kt

data class RemotePlaybackQueue(
    val tracks: List<Track>,
    val cursor: Int = 0,
) {
    val currentTrack: Track?   get() = tracks.getOrNull(cursor)
    val hasNext: Boolean       get() = cursor < tracks.lastIndex
    val remaining: List<Track> get() = tracks.drop(cursor + 1)

    fun advance(): RemotePlaybackQueue = copy(cursor = cursor + 1)
    fun isEmpty(): Boolean = tracks.isEmpty()
}
```

#### Queue Playback Handler

```kotlin
// androidMain — RemoteQueuePlayer.kt

class RemoteQueuePlayer(
    private val remotePlaybackHandler: RemotePlaybackHandler,  // existing interface — not modified
    private val scope: CoroutineScope,
) {
    private val _queueState = MutableStateFlow<RemotePlaybackQueue?>(null)
    val queueState: StateFlow<RemotePlaybackQueue?> = _queueState.asStateFlow()

    fun playQueue(queue: RemotePlaybackQueue, service: RemoteService) {
        _queueState.value = queue
        playCurrentTrack(service)
    }

    fun onTrackEnded(service: RemoteService) {
        val current = _queueState.value ?: return
        if (current.hasNext) {
            _queueState.value = current.advance()
            playCurrentTrack(service)
        } else {
            onQueueExhausted()
        }
    }

    private fun playCurrentTrack(service: RemoteService) {
        val track = _queueState.value?.currentTrack ?: return
        scope.launch {
            when (service) {
                RemoteService.SPOTIFY       ->
                    remotePlaybackHandler.playOnSpotify(track.name, track.artist)
                RemoteService.YOUTUBE_MUSIC ->
                    remotePlaybackHandler.playOnYoutube(track.name, track.artist)
            }
        }
    }

    private fun onQueueExhausted() {
        _queueState.value = null
    }
}
```

**SDK-level auto-advance:**

- **Spotify:** The existing [`RemotePlaybackHandlerImpl`](https://github.com/metabrainz/listenbrainz-android/blob/main/app/src/main/java/org/listenbrainz/android/repository/remoteplayer/RemotePlaybackHandlerImpl.kt) already holds a reference to `SpotifyAppRemote`. We subscribe to `spotifyAppRemote.playerApi.subscribeToPlayerState()`. When `playerState.isPaused && playerState.playbackPosition == 0L` after a track was playing → call `onTrackEnded()`.
- **YouTube:** The existing YouTube playback in `RemotePlaybackHandlerImpl` uses the IFrame API. We hook into the `onStateChange` callback with state `YT.PlayerState.ENDED` → call `onTrackEnded()`.

**Background continuity** is handled by a foreground service that holds a `WakeLock` and posts a persistent notification with track title, artist, and queue position. Playlist tracks are sourced from the existing [`repository/playlists/`](https://github.com/metabrainz/listenbrainz-android/tree/main/app/src/main/java/org/listenbrainz/android/repository/playlists) data layer (specifically [`PlaylistRepository`](https://github.com/metabrainz/listenbrainz-android/blob/main/app/src/main/java/org/listenbrainz/android/repository/playlists/PlaylistRepository.kt)) and persisted in Room via the existing [`AppDatabase`](https://github.com/metabrainz/listenbrainz-android/blob/main/app/src/main/java/org/listenbrainz/android/model/AppDatabase.kt) so the queue survives process death.

#### Edge Cases

| Scenario | Handling |
|----------|----------|
| Spotify token expiry mid-queue | Exponential-backoff re-auth via the existing Spotify auth flow in [`RemotePlaybackHandlerImpl`](https://github.com/metabrainz/listenbrainz-android/blob/main/app/src/main/java/org/listenbrainz/android/repository/remoteplayer/RemotePlaybackHandlerImpl.kt); UI shows retry affordance |
| YouTube IFrame latency | 500ms buffer before advancing; fallback to 2s polling |
| Network loss | Queue persisted in Room via [`AppDatabase`](https://github.com/metabrainz/listenbrainz-android/blob/main/app/src/main/java/org/listenbrainz/android/model/AppDatabase.kt); resume on reconnect |
| SDK disconnection | Reconnect attempt × 3 using existing `SpotifyAppRemote` connection logic, then degrade to `ListeningNowActive` |
| Empty queue from playlist | Transition directly to `ListeningNowActive` with `remoteQueue = emptyList()` |

---

### 4. Revamped Scaffold Front Layer
![scaffold_front_layer_flow](https://github.com/user-attachments/assets/fd73abe9-cb17-42ce-b0e6-96cbdbf306af)

Currently, [`MainActivity.kt`](https://github.com/metabrainz/listenbrainz-android/blob/main/app/src/main/java/org/listenbrainz/android/ui/screens/main/MainActivity.kt) builds the Scaffold and wires the front layer exclusively to [`BrainzPlayerViewModel`](https://github.com/metabrainz/listenbrainz-android/blob/main/app/src/main/java/org/listenbrainz/android/viewmodel/BrainzPlayerViewModel.kt). The existing [`BrainzPlayerBackLayerContent`](https://github.com/metabrainz/listenbrainz-android/blob/main/app/src/main/java/org/listenbrainz/android/ui/screens/brainzplayer/BrainzPlayerBackLayerContent.kt) serves as the only playback composable in the Scaffold. The existing `BrainzPlayerMiniPlayer` is **preserved and reused as-is** — it is simply wrapped inside the new `UnifiedPlaybackFrontLayer` alongside the new variants:

```kotlin
// UnifiedPlaybackFrontLayer.kt

@Composable
fun UnifiedPlaybackFrontLayer(
    state: PlaybackState,
    onAction: (PlaybackAction) -> Unit,
    modifier: Modifier = Modifier,
) {
    AnimatedContent(
        targetState = state,
        transitionSpec = {
            slideInVertically { it } togetherWith slideOutVertically { -it }
        },
        modifier = modifier,
    ) { target ->
        when (target) {
            is PlaybackState.BrainzPlayerActive   ->
                BrainzPlayerMiniPlayer(target, onAction)   // existing composable, unchanged
            is PlaybackState.ListeningNowActive   ->
                ListeningNowMiniPlayer(target, onAction)
            is PlaybackState.RemotePlaybackActive ->
                RemotePlaybackMiniPlayer(target, onAction)
            PlaybackState.Idle ->
                Spacer(Modifier.height(0.dp))
        }
    }
}
```

```kotlin
@Composable
fun ListeningNowMiniPlayer(
    state: PlaybackState.ListeningNowActive,
    onAction: (PlaybackAction) -> Unit,
) {
    Column {
        TrackRow(
            artUrl   = state.listen.coverArtUrl,
            title    = state.listen.trackName,
            subtitle = buildString {
                append(state.listen.artistName)
                if (!state.isLocalDevice) append(" · ${state.listen.deviceName}")
            },
            badge    = if (state.isLocalDevice) LocalBadge() else ListeningNowBadge(),
            controls = if (state.isLocalDevice) {
                { PlayPauseSkipControls(onAction) }
            } else null,
        )

        if (state.isLocalDevice && !state.remoteQueue.isNullOrEmpty()) {
            QueueSection(
                tracks = state.remoteQueue,
                label  = "Up next (${state.remoteQueue.size} tracks)",
            )
        }
    }
}
```

The feature is gated behind a `FeatureFlag.UNIFIED_PLAYBACK` flag (following the existing feature flag pattern in the codebase) so it can be merged to `cmp` without breaking existing behaviour during development.

#### Mini-player Variant Summary

| Variant | Album Art | Badge | Controls | Queue |
|---------|-----------|-------|----------|-------|
| `BrainzPlayerMiniPlayer` (existing) | ✓ | — | Play/Pause · Skip · Seek | — |
| `ListeningNowMiniPlayer` (remote device) | ✓ | "Listening now" + device name | None — read-only | — |
| `ListeningNowMiniPlayer` (local device) | ✓ | "Local" | Play/Pause · Skip | ✓ Remote queue |
| `RemotePlaybackMiniPlayer` | ✓ | Service logo | Play/Pause · Skip | ✓ Full queue |

---

### 5. ViewModel Wiring

Currently, playback state is split across [`BrainzPlayerViewModel`](https://github.com/metabrainz/listenbrainz-android/blob/main/app/src/main/java/org/listenbrainz/android/viewmodel/BrainzPlayerViewModel.kt) (which owns local player state via [`BrainzPlayerServiceConnection`](https://github.com/metabrainz/listenbrainz-android/blob/main/app/src/main/java/org/listenbrainz/android/service/BrainzPlayerServiceConnection.kt)) and [`ListensViewModel`](https://github.com/metabrainz/listenbrainz-android/blob/main/app/src/main/java/org/listenbrainz/android/viewmodel/ListensViewModel.kt) (which holds Listening Now data). The new `UnifiedPlaybackViewModel` combines all streams into one:

```kotlin
// UnifiedPlaybackViewModel.kt

class UnifiedPlaybackViewModel(
    private val playbackStateManager: PlaybackStateManager,
    private val brainzPlayerConnection: BrainzPlayerServiceConnection,  // existing
    private val listeningNowRepo: ListeningNowSocketRepository,        // new
    private val deviceDetection: DeviceDetectionRepository,             // new
    private val remoteQueuePlayer: RemoteQueuePlayer,                   // new
) : ViewModel() {

    val playbackState: StateFlow<PlaybackState> = combine(
        brainzPlayerConnection.playerState,
        listeningNowRepo.listeningNowFlow,
        remoteQueuePlayer.queueState,
    ) { bp, ln, rq ->
        val isLocal = ln?.let { deviceDetection.isLocalDevice(it) } ?: false
        playbackStateManager.resolve(
            brainzPlayerState = bp,
            listeningNow      = ln,
            isLocalDevice     = isLocal,
            remotePlayback    = rq?.toInputState(),
        )
    }
        .stateIn(viewModelScope, SharingStarted.WhileSubscribed(5_000), PlaybackState.Idle)

    fun onAction(action: PlaybackAction) {
        viewModelScope.launch {
            when (action) {
                is PlaybackAction.PlayPause  -> handlePlayPause()
                is PlaybackAction.SkipNext   -> handleSkipNext()
                is PlaybackAction.PlayQueue  ->
                    remoteQueuePlayer.playQueue(action.tracks.toQueue(), action.service)
                is PlaybackAction.StopRemote -> remoteQueuePlayer.stop()
                else -> { /* delegate to BrainzPlayerViewModel */ }
            }
        }
    }
}
```

DI registration follows the existing Koin module pattern used throughout the app (e.g., in [`di/`](https://github.com/metabrainz/listenbrainz-android/tree/main/app/src/main/java/org/listenbrainz/android/di)):

```kotlin
// di/PlaybackModule.kt

val playbackModule = module {
    viewModel {
        UnifiedPlaybackViewModel(
            playbackStateManager   = get(),
            brainzPlayerConnection = get(),
            listeningNowRepo       = get(),
            deviceDetection        = get(),
            remoteQueuePlayer      = get(),
        )
    }
}
```

---

### 6. Testing Strategy

#### Unit Tests — `commonTest`

```kotlin
// PlaybackStateManagerTest.kt

class PlaybackStateManagerTest {
    private val manager = PlaybackStateManager()

    @Test
    fun `brainzPlayer active overrides listening now`() {
        val state = manager.resolve(
            brainzPlayerState = BrainzPlayerInputState(isPlaying = true, song = fakeSong()),
            listeningNow      = fakeListeningNowEvent(),
            isLocalDevice     = true,
            remotePlayback    = null,
        )
        assertIs<PlaybackState.BrainzPlayerActive>(state)
    }

    @Test
    fun `remote device listening now is read-only`() {
        val state = manager.resolve(
            brainzPlayerState = null,
            listeningNow      = fakeListeningNowEvent(),
            isLocalDevice     = false,
            remotePlayback    = null,
        )
        assertIs<PlaybackState.ListeningNowActive>(state)
        assertFalse((state as PlaybackState.ListeningNowActive).isLocalDevice)
    }

    @Test
    fun `local device with remote queue produces RemotePlaybackActive`() {
        val state = manager.resolve(
            brainzPlayerState = null,
            listeningNow      = fakeListeningNowEvent(),
            isLocalDevice     = true,
            remotePlayback    = fakeRemotePlaybackState(),
        )
        assertIs<PlaybackState.RemotePlaybackActive>(state)
    }

    @Test
    fun `idle when nothing active`() {
        val state = manager.resolve(null, null, false, null)
        assertEquals(PlaybackState.Idle, state)
    }
}
```

Flow emissions tested with **Turbine**:

```kotlin
@Test
fun `state transitions from Idle to BrainzPlayerActive`() = runTest {
    val viewModel = buildTestViewModel()
    viewModel.playbackState.test {
        assertEquals(PlaybackState.Idle, awaitItem())
        fakePlayerConnection.emit(activeBrainzPlayerState())
        assertIs<PlaybackState.BrainzPlayerActive>(awaitItem())
    }
}
```

#### Integration Tests — `androidTest`

- Queue advancement across Spotify `PlayerState` callbacks with a fake `SpotifyAppRemote` (mirroring the real connection in [`RemotePlaybackHandlerImpl`](https://github.com/metabrainz/listenbrainz-android/blob/main/app/src/main/java/org/listenbrainz/android/repository/remoteplayer/RemotePlaybackHandlerImpl.kt)).
- Device detection accuracy using [`ListenServiceManager`](https://github.com/metabrainz/listenbrainz-android/blob/main/app/src/main/java/org/listenbrainz/android/service/ListenServiceManager.kt) with injected fake submissions.
- WebSocket reconnect → REST fallback transition via [`SocketRepository`](https://github.com/metabrainz/listenbrainz-android/blob/main/app/src/main/java/org/listenbrainz/android/repository/socket/SocketRepository.kt).

#### UI Tests — Compose Test

```kotlin
@Test
fun `front layer renders read-only view for remote device`() {
    composeTestRule.setContent {
        UnifiedPlaybackFrontLayer(
            state    = fakeListeningNowRemote(),
            onAction = {},
        )
    }
    composeTestRule.onNodeWithText("Listening now").assertIsDisplayed()
    composeTestRule.onNodeWithContentDescription("Play").assertDoesNotExist()
}
```

**Coverage target:** ≥ 90% line coverage enforced via **JaCoCo** (Android instrumented tests) + **Kover** (KMP `commonTest`) in CI.

---

## Timeline

### Phase 1 — Foundation + Core Integration (pre-midterm)

**Exit criteria:** `playback-shared` compiles and all state machine unit tests pass. Listening Now flows through `PlaybackStateManager` with correct device detection. Remote playback supports continuous queue playback via Spotify and YouTube. BrainzPlayer correctly overrides Listening Now when active.

| Week | Focus | Priority | Deliverables |
|------|-------|----------|--------------|
| May 1–24 | Community bonding + spike | Medium | Finalize scope with mentor. Deep-dive into [`ListenSubmissionService`](https://github.com/metabrainz/listenbrainz-android/blob/main/app/src/main/java/org/listenbrainz/android/service/ListenSubmissionService.kt), [`ListenServiceManager`](https://github.com/metabrainz/listenbrainz-android/blob/main/app/src/main/java/org/listenbrainz/android/service/ListenServiceManager.kt), and [`RemotePlaybackHandlerImpl`](https://github.com/metabrainz/listenbrainz-android/blob/main/app/src/main/java/org/listenbrainz/android/repository/remoteplayer/RemotePlaybackHandlerImpl.kt). Prototype `PlaybackStateManager` in isolation. Draft module Gradle config. |
| May 25–31 | KMP module scaffolding | High | Create `playback-shared` alongside existing [`shared/`](https://github.com/metabrainz/listenbrainz-android/tree/main/shared) on `cmp`. Define `PlaybackState`, `PlaybackAction`, `Track`, `Listen`, `PlaybackSource` in `commonMain`. Add iOS stubs. Set up JaCoCo + Kover. |
| Jun 1–7 | State machine + BrainzPlayer | High | Implement `PlaybackStateManager.resolve()`. Full unit test suite in `commonTest` (kotlin.test + Turbine). Wire [`BrainzPlayerServiceConnection`](https://github.com/metabrainz/listenbrainz-android/blob/main/app/src/main/java/org/listenbrainz/android/service/BrainzPlayerServiceConnection.kt) as first input stream. |
| Jun 8–14 | Listening Now + device detection | Critical | Extend [`repository/socket/`](https://github.com/metabrainz/listenbrainz-android/tree/main/app/src/main/java/org/listenbrainz/android/repository/socket) with `ListeningNowSocketRepository` (WebSocket + REST fallback via [`ListensService`](https://github.com/metabrainz/listenbrainz-android/blob/main/app/src/main/java/org/listenbrainz/android/service/ListensService.kt)). Implement `AndroidDeviceDetection` wrapping [`ListenServiceManager`](https://github.com/metabrainz/listenbrainz-android/blob/main/app/src/main/java/org/listenbrainz/android/service/ListenServiceManager.kt). Refactor [`ListensViewModel`](https://github.com/metabrainz/listenbrainz-android/blob/main/app/src/main/java/org/listenbrainz/android/viewmodel/ListensViewModel.kt). |
| Jun 15–21 | Listening Now UI + feature flag | High | Build `ListeningNowMiniPlayer` (both variants). Wire to `UnifiedPlaybackFrontLayer` behind `FeatureFlag.UNIFIED_PLAYBACK`. |
| Jun 22–28 | Remote Playback queue | Critical | Implement `RemotePlaybackQueue` + `RemoteQueuePlayer` wrapping [`RemotePlaybackHandlerImpl`](https://github.com/metabrainz/listenbrainz-android/blob/main/app/src/main/java/org/listenbrainz/android/repository/remoteplayer/RemotePlaybackHandlerImpl.kt). Integrate Spotify `PlayerState` subscription and YouTube `onStateChange(ENDED)`. Persist queue in Room via [`AppDatabase`](https://github.com/metabrainz/listenbrainz-android/blob/main/app/src/main/java/org/listenbrainz/android/model/AppDatabase.kt). |
| Jun 29–Jul 5 | Remote Playback UI + background | High | `RemotePlaybackMiniPlayer` with queue visualization. Foreground service for background continuity. Integration tests for queue exhaustion. Pre-midterm stabilization. |

### Midterm Evaluation (Jul 6–10)

Submit midterm evaluation, apply mentor feedback, fix regressions.

### Phase 2 — Scaffold Integration + Polish (post-midterm)

| Week | Focus | Priority | Deliverables |
|------|-------|----------|--------------|
| Jul 11–17 | Midterm feedback | Critical | Apply mentor feedback. Fix regressions. Lock remaining scope. |
| Jul 18–24 | Unified Scaffold front layer | High | Replace existing front layer in [`MainActivity.kt`](https://github.com/metabrainz/listenbrainz-android/blob/main/app/src/main/java/org/listenbrainz/android/ui/screens/main/MainActivity.kt) with `UnifiedPlaybackFrontLayer`. `AnimatedContent` transitions. Surface queue for local-device remote playback. |
| Jul 25–31 | End-to-end wiring + edge cases | High | Full flow: BrainzPlayer start overrides LN; stop restores previous state. Handle Spotify token expiry (via existing auth in [`RemotePlaybackHandlerImpl`](https://github.com/metabrainz/listenbrainz-android/blob/main/app/src/main/java/org/listenbrainz/android/repository/remoteplayer/RemotePlaybackHandlerImpl.kt)), YouTube IFrame latency, network loss, SDK disconnection. |
| Aug 1–7 | Full test suite + performance | Critical | All unit, integration, and UI tests. Verify ≥ 90% coverage. Profile startup, memory, battery impact. |
| Aug 8–14 | Accessibility + docs | High | TalkBack audit. KDoc for all public APIs. Architecture Decision Record. Migration guide. Address final review feedback. |
| Aug 15–17 | Final submission + handover | Critical | Prepare final evaluation deliverables: detailed technical report. Submit final GSoC evaluation. |

---

## iOS Compatibility Note

The `commonMain` layer contains zero Android imports. All platform services are accessed only through `expect` interfaces. iOS `actual` files are provided as no-op stubs so `playback-shared` compiles for iOS targets on the `cmp` branch from day one. Full iOS implementations (`AVAudioPlayer`, `CoreMotion`) are deferred to post-GSoC — the architecture requires no changes to accommodate them when ready.

---

## Extras (time permitting)

- Full iOS `actual` implementations in `iosMain` (`AVAudioPlayer`, `CoreMotion`).
- Rich foreground service notification — album art, progress bar, queue count.
- Analytics — state transition frequency and device detection accuracy metrics.

---

## About Me

### What type of music do you listen to?

I mostly enjoy listening to **Hip-hop** while coding. Some of my favorite artists include **Panther, Raftaar, and Fotty Seven**.

Favorite MBIDs:
- `f49eebbf-2afe-4a61-91b5-79200c996a13`
- `5d1d23d9-b4ca-4ab7-b917-259b74313d8c`
- `f2c6d65e-527a-4c99-9cfd-94e638d1057b`

### When did you first start programming?

I first started programming in **9th grade**, learning the basics of **Java**. What hooked me early was seeing how a few lines of code could turn an idea into something real and useful.

### Have you contributed to other open-source projects? If so, which projects and can we see some of your code?

**ListenBrainz Android** has been my first significant open-source experience, where I actively contributed to the project.

### What sorts of programming projects have you done on your own time?

I have worked on several Android apps including migration of [Screen Recorder Facecam Audio](https://play.google.com/store/apps/details?id=com.nbow.screenrecorder&hl=en-US) from legacy Java and Kotlin codebase to Compose following MVVM and integrating libraries like Koin, Firebase, and ExoPlayer 2.

### What computer(s) do you have available for working on your SoC project?

Lenovo IdeaPad Gaming 3 — AMD Ryzen 7 5800H, 24 GB RAM, 512 GB SSD, NVIDIA GeForce RTX 3050

### How much time do you have available per week?

I will dedicate **20-25 hours per week** with no academic obligations during this time, I will be fully focused on development. If I have additional bandwidth, I would be glad to take on extra tasks and contribute to other areas of the project as well.
