> url 압축을 해보자
> [url 압축의 응용](https://www.youtube.com/watch?v=pCOBmmJARPE)


## 사전지식
### [http 301 - Permanently Moved](https://ko.wikipedia.org/wiki/HTTP_301)
> 영구적 이동 -> 브라우저가 해당 리다이렉트를 캐싱할 겁니다.
- 즉 한번 리다이렉트 이후로는, 서버에 리다이렉트 될 주소를 질의하지 않고 즉시, 목표 주소를 찾아갈 수가 있습니다.
- **영구적 이동** 이라는 키워드처럼 캐시된 와중에 주소가 바뀌어버린다면 안될거에요
- [RFC 문서](https://net-study.club/entry/RFC-Request-for-Comments%EB%9E%80-RFC%EC%9D%98-%EC%97%AD%EC%82%AC-RFC-%EC%A2%85%EB%A5%98-RFC-%ED%91%9C%EC%A4%80%ED%99%94-%EC%A0%88%EC%B0%A8) 에 따른 기준은 다음과 같아요:
	-   클라이언트가 링크 편집 기능이 있으면 요청 URL에 대한 모든 참조를 업데이트해야 한다. *(이게 뭔소리임?)*
	-   특별한 표시가 없으면 응답은 캐시가 가능할 것.
	-   요청 메소드가 HEAD가 아닌 경우 엔티티는 새로운 URL로 향하는 하이퍼링크와 함께 크기가 작은 하이퍼텍스트 참고사항을 포함해야 한다.
	-   GET이나 HEAD 외의 요청에 대해 301 상태 코드를 수신할 경우 클라이언트는 리다이렉트 전에 사용자에게 물어보아야 한다.'
	
### [http 302 - Found](https://developer.mozilla.org/ko/docs/Web/HTTP/Status/302)

> 캐싱을 하지 않아요. 매번 리다이렉션 url을 서버에게 받고, 재질의 해야해요.

- 또한 302 code를 받은 브라우저는 원본 요청 타입과 다른 타입의 요청으로 리다이렉트 요청을 할 수가 있어요 
	- 때문에, 만약 원본 요청과 동일한 타입으로 요청하고 싶다면, http 307 (Temporary Redirect)를 사용해야 해요
	- http 303 (See Other) 코드는 명시적으로 항상 GET 타입으로 리3
	- 다이렉트를 실행해요 캐싱 되지 않기 때문에, 301에 비해 **서버 부하**가 올 수 있지만, 사용자 트래픽 분석에는 이점이 있어요



### 결론
- 이번 장에서는, 단축된 url을 사용자가 요청하면, 이에 맞는 원본 url로 **리! 다이렉트! ** 시킬게요


# URL, 어떻게 줄여볼까?

## 1. 해싱을 써보자
- Hash를 쓰면 편한데, in-memory (WAS 내부에 HashMap 같은 자료구조를 사용한다고 하면,) 안정성도 떨어지고, 분산 아키텍처에서는 사용이 어려울 수 있어요.
	- 승환이가 A 서버에 단축된 url을 저장했는데, 그 다음 요청을 B 서버에 보내버리면, B서버 HashMap에서는 도무지 원본 url을 찾을 수가 없어요

- 그렇기 때문에, **db에 저장하는게** 좋을 거 같아요!

- 그런데 해싱을 하는데 문제가 생겼어요!

### 해싱 문제상황: 너무 길다
- CRC32 함수를 써도 8자, 책의 요구사항은 7글자만 있어도 충족이 가능해요
- **해결!** : 앞에 7 글자만 쓰고, 충돌나면 안날때까지 해싱을 다시 돌린다.  
	- 예상 문제점: 100번 충돌나면 101번 돌려야 해... 언제 이러고 자빠졌냐?
	- **해결2!**: db 대신 **블룸 필터**를 통해, 확인하자!
	- *아쉬운 점...*: 어쨌든 충돌이 안 날때까지 반복을 해야하는건 동일하다.


> 블룸필터: 확률론에 기초한...어쩌구. 아무튼 해당 데이터 존재 여부를 빠르게 확인 할 수 있다. (나무위키 참조)

## 2. 진법 변환(Base-64) 를 활용하자

- 그냥 단순하게 url에 배정된 id 값을 64진법으로 변환하는 방식이다. (가용한 문자가 64개이기 때문에 64진법을 활용한다)
- URL길이가 가변하게 된다.
- **문제점1**: id값이 꼬오오옥 유일해야 한다! (7장 참조)
- **문제점2**: id 값이 1씩 증가하는 로직이라면? 축약 url 유추가 쉬워진다 -> 아무튼 보안문제

## 결론
- 요구사항에 맞게 잘 짜자. 
- URL 특성상 읽기 연산이 많으니 캐싱을 가미해보자


