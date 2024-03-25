---
title: "[Nestjs + SocketIO + React Native] 실시간으로 위치 정보를 공유하는 서비스 만들기 (2)"
last_modified_at: '2024-03-26 09:20:00 +0900'
categories:
- React Native
tag:
- React Native
- nestjs
- nodejs
---

### 실시간으로 위치 정보를 지도에 표시하는 앱 개발기

![]({{ 'assets/images/post/2024/nestjs-socketio-share-location/preview.gif' | relative_url }}){: width="500"}

[이전글](https://kiss8981.github.io/nestjs/nestjs-socketio-share-location/)에서는 실시간 처리를 담당하는 백엔드를 만들었다.  
이번 글에서는 실시간으로 위치를 전달하는 앱을 만들어 보겠다.


실시간 통신을 위해서 `socket.io-client` 패키지를 이용하여 진행하였다.

우선 백그라운드, 포그라운드 실시간 위치정보를 받기 위한 권한을 받고  
위치 정보를 백그라운드에서 사용 중 이라는 상태를 표시하기 위해 알림 권한도 요청한다.
```ts
 useEffect(() => {
    (async () => {
      try {
        const { granted: foregroundGranted } =
          await Location.requestForegroundPermissionsAsync();
        const { granted: backgroundGranted } =
          await Location.requestBackgroundPermissionsAsync();

        const { granted: notificationGranted } =
          await Notifications.requestPermissionsAsync({
            ios: {
              allowAlert: true,
              allowBadge: true,
              allowSound: true,
            },
            android: {
              allowAlert: true,
              allowBadge: true,
              allowSound: true,
            },
          }); // foregroundGranted, backgroundGranted, notificationGranted 권한이 없을경우 예외처리
      } catch (e) {
        console.warn(e);
      }
    })();
  }, []);
```

유저 인증과 웹 소캣 연결
```ts
useEffect(() => {
    socket.on("authenticate", data => {
      const { data: response } = JSON.parse(data);
      if (typeof response == "boolean" && !response) {
        Toast.show({
          type: "error",
          text1: "인증 실패",
          text2: "유저 정보를 불러오지 못했습니다. 다시 로그인 해주세요.",
        });
        setIsConnected(true);
      } else {
        Toast.show({
          type: "success",
          text1: "연결성공",
          text2: "서버 연결에 성공했습니다.",
        });
      }
    });

    socket.emit("authenticate", {
      provider: user.provider,
      token: token.access,
    });

    socket.on("disconnect", () => {
      Toast.show({
        type: "error",
        text1: "서버 연결 끊김",
        text2: "서버와 연결이 끊겼습니다. 다시 연결합니다.",
      });
			setIsConnected(false);
    });

    socket.on("connect", () => {
      socket.emit("authenticate", {
        provider: user.provider,
        token: token.access,
      });
    });
  }, []);
	
	// 연결 끊김시 재연결 처리
	  useEffect(() => {
    if (!isConnected) {
      socket.connect();
    }
  }, [isConnected]);
```

운행시작 및 실시간 위치 전송 시작
```
const startBusRun = async () => {
    if (!route) {
      Toast.show({
        type: "error",
        text1: "운행 시작",
        text2: "운행할 노선을 선택해주세요.",
      });
      return;
    }
    setIsStart(true);
		
		// 운행 시작시 화면 항상 켜짐
    activateKeepAwakeAsync();

    await SecureStore.setItemAsync("route", route);
    await SecureStore.setItemAsync("providerId", user.provider!);

    // 앱에서 나갈경우 백그라운드에서도 위치정보 전송 활성화
    startLocationFetchBackground();
		
    const watcher = setInterval(async () => {
      const nowLocation = await Location.getCurrentPositionAsync({
        accuracy: Location.Accuracy.High,
      });

      const {
        coords: { latitude, longitude },
      } = nowLocation;
			
      socket.emit("locationupdate", {
        busId: route,
        location: {
          latitude,
          longitude,
        },
      });
    }, 4000);

    setLocationWatcher(watcher);

    Toast.show({
      type: "success",
      text1: "운행 시작",
      text2: "운행을 시작합니다.",
    });
  };
	
const stopBusRun = () => {
    if (locationWatcher) {
      clearInterval(locationWatcher);
    }
    setIsStart(false);
    deactivateKeepAwake();
    stopLocationFetchBackground();

    Toast.show({
      type: "success",
      text1: "운행 종료",
      text2: "운행이 종료되었습니다.",
    });
  };
```

백그라운드 TaskManager를 이용한 위치정보 전달  

```
const TASK_FETCH_LOCATION = "TASK_FETCH_LOCATION";

TaskManager.defineTask(
  TASK_FETCH_LOCATION,
  async ({
    data: { locations },
    error,
  }: {
    data: {
      locations: LocationObject[];
    };
    error?: TaskManagerError | null;
  }) => {
    if (error) {
      console.error(error);
      return;
    }
    const accessToken = await SecureStore.getItemAsync("accessToken");
    const route = await SecureStore.getItemAsync("route");
    const providerId = await SecureStore.getItemAsync("providerId");

    const [location] = locations;
    try {
      await fetcher.post(
        "/bus/location",
        {
          location: {
            latitude: location.coords.latitude,
            longitude: location.coords.longitude,
          },
          providerId: providerId,
          busId: route,
        },
        {
          headers: {
            Authorization: accessToken,
          },
        }
      );
    } catch (err) {
      const error = err as AxiosError;
      console.error(error);
    }
  }
);
```

```
  const startLocationFetchBackground = async () => {
    await Location.startLocationUpdatesAsync(TASK_FETCH_LOCATION, {
      accuracy: Location.Accuracy.Highest,
      timeInterval: 5000,
      distanceInterval: 10,
      foregroundService: {
        notificationTitle: "셔틀버스 위치 제공 중",
        notificationBody:
          "셔틀버스 위치를 제공 중입니다. 운행 종료 시 중지해주세요.",
        killServiceOnDestroy: true,
      },
    });
  };
```
