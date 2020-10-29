[TOC]

# Learn WebRTC

## connect
1. ```java
    new WebSocketClient(wssUri);
    SSLContext.getInstance("TLS").init(null, new TrustManager[]{new TrustManagerTest()}, new SecureRandom());
    WebSocketClient.setSocketFactory(sslContext.getSocketFactory());
    WebSocketClient.connect();
    ```
2. ```java
new PeerConnectionHelper(WebSocketClient, iceServers);
ArrayList<PeerConnection.IceServer> mICEServers = new ArrayList<>();
   foreach IceServer:
   	PeerConnection.IceServer iceServer = PeerConnection.IceServer
           .builder(myIceServer.uri)
           .setUsername(myIceServer.username)
           .setPassword(myIceServer.password)
           .createIceServer();
   	mICEServers.add(iceServer);
   ```

## connect success -> join room
```java
WebSocketClient.send(json: {"data":{"room":"232343"},"eventName":"__join"});
第一个加入的：
WebSocketClient.onMessage(json: {"eventName":"_peers","data":{"connections":[],"you":"93fb0c51-448a-4f34-8fe3-05b782ca3653"}})
第二个加入的：
WebSocketClient.onMessage {"eventName":"_peers","data":{"connections":["93fb0c51-448a-4f34-8fe3-05b782ca3653"],"you":"3b3a1b90-096b-4171-bcbd-e0d04bc88e16"}}
```
## join success
```java
PeerConnectionHelper.onJoinToRoom(ArrayList<String> connections, String myId);
ArrayList<String> mConnections = new ArrayList<>();
mConnections.addAll(connections);
```
### 1.createConnectionFactory();
```java
PeerConnectionFactory.initialize(PeerConnectionFactory.InitializationOptions.builder(context).createInitializationOptions());
PeerConnectionFactory.Options options = new PeerConnectionFactory.Options();
PeerConnectionFactory peerConnectionFactory. = PeerConnectionFactory.builder()
	.setOptions(options).setAudioDeviceModule(
		JavaAudioDeviceModule.builder(context).createAudioDeviceModule())
	.createPeerConnectionFactory();
```
### 2.createLocalStream();
```java
MediaStream mediaStream = peerConnectionFactory.createLocalMediaStream("ARDAMS");
AudioSource audioSource = peerConnectionFactory
    .createAudioSource(createAudioConstraints());
		MediaConstraints.mandatory.add:
			googEchoCancellation:true;
			googAutoGainControl:false;
			googHighpassFilter:true;
			googNoiseSuppression:true.
AudioTrack audioTrack = peerConnectionFactory
    .createAudioTrack("ARDAMSa0", audioSource);
mediaStream.addTrack(audioTrack);
```

### 3.createPeerConnections();

```java
foreach connections:
	Peer peer = new Peer("connection id"); // 比如new Peer("93fb0c51-448a-4f34-8fe3-05b782ca3653")
		PeerConnection.RTCConfiguration rtcConfig = 
            new PeerConnection.RTCConfiguration(mICEServers);
		PeerConnection peerConnection = 
            peerConnectionFactory.createPeerConnection(rtcConfig, this);
```

### 4.addStreams();

```java
foreach Peer:
	peer.peerConnection.addStream(mediaStream);
```

### 5.createOffers();
  ```java
foreach Peer:
	peer.peerConnection.createOffer(peer, createReceiveConstraint());
		MediaConstraints.mandatory.add:
			OfferToReceiveAudio:true;
        	OfferToReceiveVideo:false.
  ```

## receiver

### 加入房间，当房间里已经有其他用户时：

```flow
st=>start: start
op1=>operation: onRenegotiationNeeded
op2=>operation: onCreateSuccess type: OFFER
sub2=>subroutine: peerConnection.setLocalDescription(Peer.this, 
		new SessionDescription(origSdp.type, origSdp.description));
op3=>operation: onSignalingChange: HAVE_LOCAL_OFFER
op4=>operation: onSetSuccess state: HAVE_LOCAL_OFFER, role: Caller
sub4=>subroutine: WebSocketClient.sendOffer
op5=>operation: onIceGatheringChange: GATHERING
op6=>operation: onIceCandidate: audio:0:candidate:1989069592 1 udp 2122260223 192.168.53.59 40190 typ host generation 0 ufrag gENR network-id 3 network-cost 10:
sub6=>subroutine: WebSocketClient.sendIceCandidate
op7=>operation: onIceCandidate: audio:0:candidate:559267639 1 udp 2122202367 ::1 47938 typ host generation 0 ufrag gENR network-id 2:
op8=>operation: onIceCandidate: audio:0:candidate:1510613869 1 udp 2122129151 127.0.0.1 45519 typ host generation 0 ufrag gENR network-id 1:
op9=>operation: onIceCandidate: audio:0:candidate:1876313031 1 tcp 1518222591 ::1 44616 typ host tcptype passive generation 0 ufrag gENR network-id 2:
op10=>operation: onIceCandidate: audio:0:candidate:344579997 1 tcp 1518149375 127.0.0.1 38179 typ host tcptype passive generation 0 ufrag gENR network-id 1:
op11=>operation: onIceCandidate: audio:0:candidate:842163049 1 udp 1686052607 210.22.179.242 2997 typ srflx raddr 192.168.53.59 rport 40190 generation 0 ufrag gENR network-id 3 network-cost 10:stun:108.177.125.127:19302
op12=>operation: onConnectionChange: CONNECTING
op13=>operation: onSignalingChange: STABLE
op14=>operation: onIceConnectionChange: CHECKING
op15=>operation: onAddTrack receiver id: ARDAMSa0, mediaStreams: [[ARDAMS:A=1:V=0]]
op16=>operation: onAddStream: [ARDAMS:A=1:V=0]
op17=>operation: onIceGatheringChange: COMPLETE
op18=>operation: onConnectionChange: CONNECTED
op19=>operation: onIceConnectionChange: CONNECTED
op20=>operation: onIceConnectionChange: COMPLETED
op21=>operation: onSetSuccess state: STABLE, role: Caller
op22=>operation: onIceConnectionChange: CONNECTED
e=>end: end

st->op1->op2->sub2->op3->op4->sub4->op5->op6->sub6->op7->op8->op9->op10->op11->op12->op13->op14->op15->op16->op17->op18->op19->op20->op21->op22->e
```



### 已在房间里的用户，收到新用户加入：

```flow
st=>start: start
op1=>operation: onRenegotiationNeeded
op2=>operation: onSignalingChange: HAVE_REMOTE_OFFER
op3=>operation: onAddTrack receiver id: ARDAMSa0, mediaStreams: [[ARDAMS:A=1:V=0]]
op4=>operation: onAddStream: [ARDAMS:A=1:V=0]
op5=>operation: onSetSuccess state: HAVE_REMOTE_OFFER, role: Receiver
sub5=>subroutine: peerConnection.createAnswer(CustomedPeer.this, offerOrAnswerConstraint());
op6=>operation: onCreateSuccess type: ANSWER
sub6=>subroutine: peerConnection.setLocalDescription(Peer.this, 
		new SessionDescription(origSdp.type, origSdp.description));
op7=>operation: onSignalingChange: STABLE
op8=>operation: onIceConnectionChange: CHECKING
op9=>operation: onSetSuccess state: STABLE, role: Receiver
sub9=>subroutine: WebSocketClient.sendAnswer
op10=>operation: onIceGatheringChange: GATHERING
op11=>operation: onConnectionChange: CONNECTING
op12=>operation: onIceCandidate: audio:0:candidate:4078321699 1 udp 2122260223 192.168.53.80 37469 typ host generation 0 ufrag sQ5h network-id 1:
op13=>operation: onIceCandidate: audio:0:candidate:559267639 1 udp 2122202367 ::1 55351 typ host generation 0 ufrag sQ5h network-id 3:
op14=>operation: onIceCandidate: audio:0:candidate:1510613869 1 udp 2122129151 127.0.0.1 52880 typ host generation 0 ufrag sQ5h network-id 2:
op15=>operation: onIceGatheringChange: COMPLETE
op16=>operation: onConnectionChange: CONNECTED
op17=>operation: onIceConnectionChange: CONNECTED
e=>end: end
st->op1->op2->op3->op4->op5->sub5->op6->sub6->op7->op8->op9->sub9->op10->op11->op12->op13->op14->op15->op16->op17->e
```





## enable H264

HardwareVideoEncoderFactory.java


```java
private boolean isHardwareSupportedInCurrentSdkH264(MediaCodecInfo info) {
        // First, H264 hardware might perform poorly on this model.
        if (H264_HW_EXCEPTION_MODELS.contains(Build.MODEL)) {
            return false;
        }
        String name = info.getName();
        // QCOM H264 encoder is supported in KITKAT or later.
        return (name.startsWith(QCOM_PREFIX) && Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) ||
                // Exynos H264 encoder is supported in LOLLIPOP or later.
                (name.startsWith(EXYNOS_PREFIX) && Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) ||
                // fengyi add: enable h264 for vhub
                (name.startsWith("OMX.google.") && Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP)
                ;
    }
```













