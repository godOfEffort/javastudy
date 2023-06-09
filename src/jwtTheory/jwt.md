# 1. 토큰 기반 인증

## 토큰기반 인증

1. 토큰 기반인증은 토큰을 사용하는 방법. 토큰은 서버에서 클라이언트를 구분하기 위한 유일한 값. 서버가 토큰을 생성해서 클라이언트에게 제공하면, 클라이언트는 이 토큰을 갖고 있다가
여러 요청을 이 토큰과 함께 신청
-------
2. 토큰을 전달하고 인증받는 과정

```
1) 클라이언트가 아이디와 비밀번호를 서버에게 전달하면서 인증 요청
2) 서버는 아이디와 비밀번호를 확인해 유효한 사용자인지 검증. 유효하면 토큰을 생성해서 응답
3) 클라이언트는 서버에서 준 토큰을 저장
4) 이후 인증이 필요한  API를 사용할 때 토큰을 함께 보낸다
5) 서버는 토큰이 유효한지 검증
6) 토큰이 유효하다면 클라이언트의 요청 처리
```

------- 
3. 토큰 기반 인증의 특징

    1) 무상태성
        - 사용자의 인증 정보가 담겨 있는 토큰이 서버가 아닌 클라이언트에 있으므로 서버에 저장할 필요가 없다. 토큰기반 인증에서는 클라이언트에서 인증 정보가 담긴
    토큰을 생성하고 인증한다. 서버 입장에서는 클라이언트의 인증 정보를 저장하거나 유지하지 않아도 된다.
    2) 확장성
        - 물건을 파는 서비스가 / 결제 서비스가 다른 서버에 있다고 가정. 세션 인증 기반은 각각 API에서 인증을 해야 하지만 토큰기반 인증에서는 토큰을 가지는
    주체는 서버가 아니라 클라이언트기 때문에 가지고 있는 하나의 토큰으로 결제 서버와 주문 서버에게 요청을 보낼 수 있다.
    3) 무결성
        - 토큰을 발급한 이후에는 토큰 정보를 변경하는 행위를 할 수 없다. 즉, 토큰의 무결성이 보장된다.
------- 
## JWT
4. JWT 
 발급받은 JWT를 이용해 인증을 하려면 HTTP 요청 헤더 중에 Authorization 키값에 Bearer + JWT 토큰값을 넣어 보내야 한다.
 JWT는 헤더, 내용(payload), 서명으로 이루어져 있다. 
 
    1) 헤더
        - 토큰의 타입과 해싱 알고리즘을 지정
        ```
            {
                "typ": "JWT",
                "alg": "HS256"
            }
        ```
    2) 내용
        - 토큰관련 정보를 담으며, 내용 한 덩어리를 클레임이라고 부른다. 클레임은 키, 값으로 이루어져 있다. 등록된 클레임, 공개 클레임, 비공개 클레임으로 나뉜다.
        - 공개클레임은 공개되도 상관없는 클레임. 보통 클레임 이름을 URI로 짓는다. 비공개 클레임은 클라이언트와 서버 간의 통신에 사용
        - iss, iat, exp는 JWT 자체에서 등록된 클레임
        ```
            {
                "iss" : "test@gmail.com", // 등록된 클레임
                "iat" : 1622370878,       // 등록된 클레임  
                "exp" : 1622370878,       // 등록된 클레임
                "https://test/test/is_admin" : true, // 공개 클레임
                "email" : "test@gmail.com", //비공개 클레임
                "hello" : "안녕하세요!"       //비공개 클레임
            }
        ```
    3) 서명
        - 해당 토큰이 조작/변경 됐는지 확인하는 용도. 헤더의 인코딩 값과 내용의 인코딩값을 합친 후에 주어진 비밀키를 사용해 해시값 생성

-------
5. 토큰 유효기간
토큰 자체가 노출되면 어떻게 될까? 토큰은 이미 발급되면 그 자체로 인증 수단이 되므로 서버는 토큰과 함께 들어온 요청이
토큰을 탈취한 사람의 요청인지 확인할 수 없다. 만약 리프레시 토큰이 있다면? 리프레시 토큰은 사용자를 인증하기 위한 용도가 아닌 액세스 토큰이
만료되었을 때 새로운 액세스 토큰을 발급하기 위해 사용한다. 
```
1) 클라이언트가 서버에게 인증 요청
2) 서버는 클라이언트에서 전달한 정보를 바탕으로 인증정보 유효한지 확인 후, 액세스 토큰과 리프레시 토큰을 생성해서 클라이언트에게 전달
3) 서버에서 생성한 리프레시 토큰을 DB에도 저장
4) 인증을 필요로 하는 API를 호출할 때 클라이언트에서 저장된 액세스 토큰과 함께 API 요청
5) 서버에서 전달받은 액세스 토큰이 유효한지 검사한 뒤에 유요하다면 클라이언트 요청 처리
6) 시간이 지나고 액세스 토큰이 만료된 뒤에 클라이언트에서 서버에 API요청 
7) 서버에서 액세스 토큰 유효한지 검증 후 만료된 토큰이라면 토큰 만료되었다는 에러 전달
8) 클라이언트에서는 이 응답을 받고 저장해준 리프레시 토큰과 함께 새로운 액세스 토큰을 발급하는 요청 전송
9) 서버에서 전달받은 리프레시 토큰이 유효한지, DB에서 리프레시 토큰을 조회한 후 저장해둔 리프레시 토큰과 같은지 확인
10 유효한 리프레시 토큰이라면 새로운 액세스 토큰 생성한 뒤 응답
```   
    

