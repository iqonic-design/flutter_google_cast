<!-- the title of doc is FlutterGoogleCast -->
```FlutterGoogleCast``` is a Flutter plugin for Google Cast SDK. It supports both iOS and Android.

## Getting Started
add ```google_cast``` as a dependency in your pubspec.yaml file.

## Usage
### Import the library
```dart
import 'package:google_cast/lib.dart';
```

### Initialize the library
```dart
GoogleCast.instance.initialize();
Future<void> initPlatformState() async {
    const appId = GoogleCastDiscoveryCriteria.kDefaultApplicationId;
    GoogleCastOptions? options;
    if (Platform.isIOS) {
      options = IOSGoogleCastOptions(
        GoogleCastDiscoveryCriteriaInitialize.initWithApplicationID(appId),
      );
    } else if (Platform.isAndroid) {
      options = GoogleCastOptionsAndroid(
        appId: appId,
      );
    }
    GoogleCastContext.instance.setSharedInstanceWithOptions(options!);
  }
```

### Start Discovery
```dart
//the discovery is even run in background when app is in foreground consuming low power
// if you want a hard discover like when you open a devices dialog/modal is recommended to use
GoogleCast.instance.startDiscovery();
/// don`t forget to stop discovery when you don`t need it anymore
GoogleCast.instance.stopDiscovery();
```

### listen to devices found
```dart
StreamBuilder<List<GoogleCastDevice>>(
                stream: GoogleCastDiscoveryManager.instance.devicesStream,
                builder: (context, snapshot) {
                  final devices = snapshot.data ?? [];
                  return Column(
                    children: [
                      Expanded(
                        child: ListView(
                          children: [
                            ...devices.map((device) {
                              return ListTile(
                                title: Text(device.friendlyName),
                                subtitle: Text(device.modelName ?? ''),
                                onTap: () => _loadQueue(device),
                              );
                            })
                          ],
                        ),
                      ),
                    ],
                  );
                },
              ))
```

### connect to a device
```dart
    await GoogleCastSessionManager.instance.startSessionWithDevice(device);
```

### load items
```dart 
await GoogleCastRemoteMediaClient.instance.queueLoadItems(
      [
        GoogleCastQueueItem(
          activeTrackIds: [0],
          mediaInformation: GoogleCastMediaInformationIOS(
            contentId: '0',
            streamType: CastMediaStreamType.BUFFERED,
            contentUrl: Uri.parse(
                'http://commondatastorage.googleapis.com/gtv-videos-bucket/sample/ElephantsDream.mp4'),
            contentType: 'video/mp4',
            metadata: GoogleCastMovieMediaMetadata(
              title: 'The first Blender Open Movie from 2006',
              studio: 'Blender Inc',
              releaseDate: DateTime(2011),
              subtitle:
                  'Song : Raja Raja Kareja Mein Samaja\nAlbum : Raja Kareja Mein Samaja\nArtist : Radhe Shyam Rasia\nSinger : Radhe Shyam Rasia\nMusic Director : Sohan Lal, Dinesh Kumar\nLyricist : Vinay Bihari, Shailesh Sagar, Parmeshwar Premi\nMusic Label : T-Series',
              images: [
                GoogleCastImage(
                  url: Uri.parse(
                      'https://i.ytimg.com/vi_webp/gWw23EYM9VM/maxresdefault.webp'),
                  height: 480,
                  width: 854,
                ),
              ],
            ),
            tracks: [
              GoogleCastMediaTrack(
                trackId: 0,
                type: TrackType.TEXT,
                trackContentId: Uri.parse(
                        'https://raw.githubusercontent.com/felnanuke2/flutter_cast/master/example/assets/VEED-subtitles_Blender_Foundation_-_Elephants_Dream_1024.vtt')
                    .toString(),
                trackContentType: 'text/vtt',
                name: 'English',
                language: RFC5646_LANGUAGE.PORTUGUESE_BRAZIL,
                subtype: TextTrackType.SUBTITLES,
              ),
            ],
          ),
        ),
        GoogleCastQueueItem(
          preLoadTime: const Duration(seconds: 15),
          mediaInformation: GoogleCastMediaInformationIOS(
            contentId: '1',
            streamType: CastMediaStreamType.BUFFERED,
            contentUrl: Uri.parse(
                'http://commondatastorage.googleapis.com/gtv-videos-bucket/sample/BigBuckBunny.mp4'),
            contentType: 'video/mp4',
            metadata: GoogleCastMovieMediaMetadata(
              title: 'Big Buck Bunny',
              releaseDate: DateTime(2011),
              studio: 'Vlc Media Player',
              images: [
                GoogleCastImage(
                  url: Uri.parse(
                      'https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/images/BigBuckBunny.jpg'),
                  height: 480,
                  width: 854,
                ),
              ],
            ),
          ),
        ),
      ],
      options: GoogleCastQueueLoadOptions(
        startIndex: 0,
        playPosition: const Duration(seconds: 30),
      ),
    );
```


### listen connection state changes
```dart
 StreamBuilder(
                    stream:
                        GoogleCastSessionManager.instance.currentSessionStream,
                    builder: (context, snapshot) {
                      final isConnected =
                          GoogleCastSessionManager.instance.connectionState ==
                              GoogleCastConnectState.ConnectionStateConnected;
                      return Visibility(
                        visible: isConnected,
                        child: FloatingActionButton(
                          onPressed: _insertQueueItemAndPlay,
                          child: const Icon(Icons.add),
                        ),
                      );
                    })
```

### listen session events
```dart
 StreamBuilder<GoogleCastSession?>(
                      stream: GoogleCastSessionManager
                          .instance.currentSessionStream,
                      builder: (context, snapshot) {
                        final bool isConnected =
                            GoogleCastSessionManager.instance.connectionState ==
                                GoogleCastConnectState.ConnectionStateConnected;
                        return IconButton(
                            onPressed: GoogleCastSessionManager
                                .instance.endSessionAndStopCasting,
                            icon: Icon(isConnected
                                ? Icons.cast_connected
                                : Icons.cast));
                      })
```

### some utils functionalities
```dart

/// seel to a specific time
 void _changeCurrentTime(double value) {
    final seconds = GoogleCastRemoteMediaClient
            .instance.mediaStatus?.mediaInformation?.duration?.inSeconds ??
        0;
    final position = (value * seconds).floor();
    GoogleCastRemoteMediaClient.instance
        .seek(GoogleCastMediaSeekOption(position: Duration(seconds: position)));
  }

/// pause or play
  void _togglePLayPause() {
    final isPlaying =
        GoogleCastRemoteMediaClient.instance.mediaStatus?.playerState ==
            CastMediaPlayerState.playing;
    if (isPlaying) {
      GoogleCastRemoteMediaClient.instance.pause();
    } else {
      GoogleCastRemoteMediaClient.instance.play();
    }
  }
  
  void _loadMedia(GoogleCastDevice device) async {
    await GoogleCastSessionManager.instance.startSessionWithDevice(device);

    GoogleCastRemoteMediaClient.instance.loadMedia(
      GoogleCastMediaInformationIOS(
        contentId: '',
        streamType: CastMediaStreamType.BUFFERED,
        contentUrl: Uri.parse(
            'http://commondatastorage.googleapis.com/gtv-videos-bucket/sample/ElephantsDream.mp4'),
        contentType: 'video/mp4',
        metadata: GoogleCastTvShowMediaMetadata(
          episode: 1,
          season: 2,
          seriesTitle: 'Big Buck Bunny',
          originalAirDate: DateTime.now(),
          images: [
            GoogleCastImage(
              url: Uri.parse(
                  'https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/images/BigBuckBunny.jpg'),
              height: 480,
              width: 854,
            ),
          ],
        ),
        tracks: [
          GoogleCastMediaTrack(
            trackId: 0,
            type: TrackType.TEXT,
            trackContentId: Uri.parse(
                    'https://raw.githubusercontent.com/felnanuke2/flutter_cast/master/example/assets/VEED-subtitles_Blender_Foundation_-_Elephants_Dream_1024.vtt')
                .toString(),
            trackContentType: 'text/vtt',
            name: 'English',
            language: RFC5646_LANGUAGE.PORTUGUESE_BRAZIL,
            subtype: TextTrackType.SUBTITLES,
          ),
        ],
      ),
      autoPlay: true,
      playPosition: const Duration(seconds: 0),
      playbackRate: 2,
      activeTrackIds: [0],
    );
  }

/// load queue
  _loadQueue(GoogleCastDevice device) async {
    await GoogleCastSessionManager.instance.startSessionWithDevice(device);
    await GoogleCastRemoteMediaClient.instance.queueLoadItems(
      [
        GoogleCastQueueItem(
          activeTrackIds: [0],
          mediaInformation: GoogleCastMediaInformationIOS(
            contentId: '0',
            streamType: CastMediaStreamType.BUFFERED,
            contentUrl: Uri.parse(
                'http://commondatastorage.googleapis.com/gtv-videos-bucket/sample/ElephantsDream.mp4'),
            contentType: 'video/mp4',
            metadata: GoogleCastMovieMediaMetadata(
              title: 'The first Blender Open Movie from 2006',
              studio: 'Blender Inc',
              releaseDate: DateTime(2011),
              subtitle:
                  'Song : Raja Raja Kareja Mein Samaja\nAlbum : Raja Kareja Mein Samaja\nArtist : Radhe Shyam Rasia\nSinger : Radhe Shyam Rasia\nMusic Director : Sohan Lal, Dinesh Kumar\nLyricist : Vinay Bihari, Shailesh Sagar, Parmeshwar Premi\nMusic Label : T-Series',
              images: [
                GoogleCastImage(
                  url: Uri.parse(
                      'https://i.ytimg.com/vi_webp/gWw23EYM9VM/maxresdefault.webp'),
                  height: 480,
                  width: 854,
                ),
              ],
            ),
            tracks: [
              GoogleCastMediaTrack(
                trackId: 0,
                type: TrackType.TEXT,
                trackContentId: Uri.parse(
                        'https://raw.githubusercontent.com/felnanuke2/flutter_cast/master/example/assets/VEED-subtitles_Blender_Foundation_-_Elephants_Dream_1024.vtt')
                    .toString(),
                trackContentType: 'text/vtt',
                name: 'English',
                language: RFC5646_LANGUAGE.PORTUGUESE_BRAZIL,
                subtype: TextTrackType.SUBTITLES,
              ),
            ],
          ),
        ),
        GoogleCastQueueItem(
          preLoadTime: const Duration(seconds: 15),
          mediaInformation: GoogleCastMediaInformationIOS(
            contentId: '1',
            streamType: CastMediaStreamType.BUFFERED,
            contentUrl: Uri.parse(
                'http://commondatastorage.googleapis.com/gtv-videos-bucket/sample/BigBuckBunny.mp4'),
            contentType: 'video/mp4',
            metadata: GoogleCastMovieMediaMetadata(
              title: 'Big Buck Bunny',
              releaseDate: DateTime(2011),
              studio: 'Vlc Media Player',
              images: [
                GoogleCastImage(
                  url: Uri.parse(
                      'https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/images/BigBuckBunny.jpg'),
                  height: 480,
                  width: 854,
                ),
              ],
            ),
          ),
        ),
      ],
      options: GoogleCastQueueLoadOptions(
        startIndex: 0,
        playPosition: const Duration(seconds: 30),
      ),
    );
  }

/// Skip to the previous item in the queue
  void _previous() {
    GoogleCastRemoteMediaClient.instance.queuePrevItem();
  }



  /// Skip to the next item in the queue
  void _next() {
    GoogleCastRemoteMediaClient.instance.queueNextItem();
  }

/// Insert a new item to the queue
  void _insertQueueItem() {
    GoogleCastRemoteMediaClient.instance.queueInsertItems(
      [
        GoogleCastQueueItem(
          preLoadTime: const Duration(seconds: 15),
          mediaInformation: GoogleCastMediaInformationIOS(
            contentId: '3',
            streamType: CastMediaStreamType.BUFFERED,
            contentUrl: Uri.parse(
                'http://commondatastorage.googleapis.com/gtv-videos-bucket/sample/ForBiggerBlazes.mp4'),
            contentType: 'video/mp4',
            metadata: GoogleCastMovieMediaMetadata(
              title: 'For Bigger Blazes',
              subtitle:
                  'Song : Raja Raja Kareja Mein Samaja\nAlbum : Raja Kareja Mein Samaja\nArtist : Radhe Shyam Rasia\nSinger : Radhe Shyam Rasia\nMusic Director : Sohan Lal, Dinesh Kumar\nLyricist : Vinay Bihari, Shailesh Sagar, Parmeshwar Premi\nMusic Label : T-Series',
              releaseDate: DateTime(2011),
              studio: 'T-Series Regional',
              images: [
                GoogleCastImage(
                  url: Uri.parse(
                      'https://i.ytimg.com/vi/Dr9C2oswZfA/maxresdefault.jpg'),
                  height: 480,
                  width: 854,
                ),
              ],
            ),
          ),
        )
      ],
      beforeItemWithId: 2,
    );
  }




  /// Insert a queue item and play it
  void _insertQueueItemAndPlay() {
    GoogleCastRemoteMediaClient.instance.queueInsertItemAndPlay(
      GoogleCastQueueItem(
        preLoadTime: const Duration(seconds: 15),
        mediaInformation: GoogleCastMediaInformationIOS(
          contentId: '3',
          streamType: CastMediaStreamType.BUFFERED,
          contentUrl: Uri.parse(
              'http://commondatastorage.googleapis.com/gtv-videos-bucket/sample/ForBiggerBlazes.mp4'),
          contentType: 'video/mp4',
          metadata: GoogleCastMovieMediaMetadata(
            title: 'For Bigger Blazes',
            subtitle:
                'Song : Raja Raja Kareja Mein Samaja\nAlbum : Raja Kareja Mein Samaja\nArtist : Radhe Shyam Rasia\nSinger : Radhe Shyam Rasia\nMusic Director : Sohan Lal, Dinesh Kumar\nLyricist : Vinay Bihari, Shailesh Sagar, Parmeshwar Premi\nMusic Label : T-Series',
            releaseDate: DateTime(2011),
            studio: 'T-Series Regional',
            images: [
              GoogleCastImage(
                url: Uri.parse(
                    'https://i.ytimg.com/vi/Dr9C2oswZfA/maxresdefault.jpg'),
                height: 480,
                width: 854,
              ),
            ],
          ),
        ),
      ),
      beforeItemWithId: 2,
    );
  }

/// This is the code that I used to get the image from the metadata
  String? _getImage(GoogleCastMediaMetadata? metadata) {
    if (metadata == null) {
      return null;
    }
    if (metadata.images?.isEmpty ?? true) {
      return null;
    }
    return metadata.images!.first.url.toString();
  }
```


### remote media client
    
```dart
//call
GoogleCastRemoteMediaClient instance;


    abstract class GoogleCastRemoteMediaClientPlatformInterface
    implements PlatformInterface {
  GoggleCastMediaStatus? get mediaStatus;

  Stream<GoggleCastMediaStatus?> get mediaStatusStream;

  Duration get playerPosition;

  Stream<Duration> get playerPositionStream;

  Stream<List<GoogleCastQueueItem>> get queueItemsStream;

  List<GoogleCastQueueItem> get queueItems;

  bool get queueHasNextItem;

  bool get queueHasPreviousItem;

  Future<void> loadMedia(
    GoogleCastMediaInformation mediaInfo, {
    bool autoPlay = true,
    Duration playPosition = Duration.zero,
    double playbackRate = 1.0,
    List<int>? activeTrackIds,
    String? credentials,
    String? credentialsType,
  });

  Future<void> queueLoadItems(
    List<GoogleCastQueueItem> queueItems, {
    GoogleCastQueueLoadOptions? options,
  });

  Future<void> setPlaybackRate(double rate);

  Future<void> setActiveTrackIDs(List<int> activeTrackIDs);

  Future<void> setTextTrackStyle(TextTrackStyle textTrackStyle);

  Future<void> pause();

  Future<void> play();

  Future<void> stop();

  Future<void> queueNextItem();

  Future<void> queuePrevItem();

  Future<void> seek(GoogleCastMediaSeekOption option);

  Future<void> queueInsertItems(
    List<GoogleCastQueueItem> items, {
    int? beforeItemWithId,
  });
  Future<void> queueInsertItemAndPlay(
    GoogleCastQueueItem item, {
    required int beforeItemWithId,
  });

  Future<void> queueRemoveItemsWithIds(
    List<int> itemIds,
  );

  Future<void> queueJumpToItemWithId(int itemId);

  Future<void> queueReorderItems({
    required List<int> itemsIds,
    required int? beforeItemWithId,
  });
}
```