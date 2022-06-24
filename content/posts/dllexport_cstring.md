---
title: "DLL Export 시 MFC / 표준 Windows Library 간의 CString 빌드 대한 주의점"
date: 2022-06-25T00:55:48+09:00
---
***

구 버전의 호환을 위하여 MFC DLL로 작성된 클래스를 재 Wrapping 후 배포해야했다.  
기존 라이브러리에는 *표준 Windows 라이브러리*로 설정하여 *공유 MFC 사용* 플래그를 꺼두었다. 그러나 구 버전 클래스 헤더에 `CString`을 사용하기 위하여 `<atlstr.h>`를 참조하여 `CString`을 사용했다. 빌드에는 이상이 없었으나, 구 프로그램에 라이브러리를 교체하고 재빌드 시에 링킹 에러가 발생했다.  
발생 원인은 `CString` 클래스가 표준 *Windows 라이브러리*와 *MFC 라이브러리간*의 `StrTrait` 차이였다.  
`<atlstr.h>`를 참조하고 `표준 Windows 라이브러리`로 빌드한 결과물에서는 아래의 클래스로 빌드 되었다.

```
StrTraitATL<wchar_t | char | TCHAR, ChTraitsCRT<wchar_t | char | TCHAR>>
```
 `<afx.h>`를 참조하고 MFC 라이브러리로 빌드한 결과물에서는 아래의 클래스로 빌드 되었다.
 ```
StrTraitMFC<wchar_t | char | TCHAR, ChTraitsCRT<wchar_t | char | TCHAR>>
```
따라서 링크된 클래스가 달랐기 때문에 *dllimport*시 링크 에러가 났다.  
위 클래스들의 타입에 따라 Multibyte(`char`), Unicode(`wchar_t`)의 구분이 같이 들어가기 때문에 결국 빌드 결과물에 *Multibyte/Unicode* 구분이 들어가게 되었으며, 기본 배포시 *x86/x64*, *Debug/Release*의 구분이 들어가서 결과적으로는 8개의 바이너리 파일이 추가로 생성되게 되었다. 그렇기 때문에 구성 관리자에서 MFC 빌드 시에만 해당 Wrapper 클래스를 참조하여 빌드 하도록 수정하여 구 버전 호환/신 버전 으로 나누어 배포하게 되었다.
