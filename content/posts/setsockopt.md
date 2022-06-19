---
title: "setsockopt 함수 사용 시 주의점"
date: 2022-06-19T13:09:44+09:00

---
***
해당 장비에 이더넷이 활성화 되어있는지 확인하기 위하여 해당 장비가 보내는 신호를 주기별로 받을 필요가 생겨서 소켓 옵션으로 SO_RCVTIMEO를 주어 일정 시간이 넘을 경우 비활성화 처리해야 하는 경우가 생겼다. 당연히 timeout이니 timeval_t가 옵션 인자로 들어갈 줄 알고 timeval_t 구조체를 사용하여 인자를 주었는데, 동작하지 않는 것이었다. 사용했던 코드는 아래와 같다.

```
timeval_t tv = {5, 0};
setsockopt(soc, SOL_SOCKET, SO_RCVTIMEO, reinterpret_cast<char*>(&tv), sizeof(tv));
```

그러나 찾아보니 윈도우에서는 인자가 timeval_t가 아닌 DWORD로 ms단위로 지정하여 보내줘야 했던 것이었다. 그래서 아래의 코드로 수정하고 적용하였더니 시간 초과 후 수신 실패가 발생하여 정상적으로 알려줄 수 있게 되었다.

```
DWORD nTimeout = 3000;
setsockopt(soc, SOL_SOCKET, SO_RCVTIMEO, reinterpret_cast<char*>&nTimeout, sizeof(nTimeout));
```

가끔 윈도우즈와 그 외 플랫폼 간의 인자가 다를 경우가 있으니, 맞는 것 같은데 동작이 안된다면 해당 함수의 MSDN 페이지를 참고하자.

[https://docs.microsoft.com/en-us/windows/win32/api/winsock/nf-winsock-setsockopt](https://docs.microsoft.com/en-us/windows/win32/api/winsock/nf-winsock-setsockopt)