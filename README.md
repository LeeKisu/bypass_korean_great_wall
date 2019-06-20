# HTTPS 차단 우회 프로젝트 : bypass_korean_great_wall
## 1. 개요
2019년 상반기 정부는 기존의 유해사이트 차단 방법(warning.or.kr)에 HTTPS 차단기술 까지 도입하였다. 이에 온,오프라인 상에서 국가에의한 인터넷 검열이라는 의견과 정당한 유해사이트 차단이라는 의견이 맞섰다. 필자는 이 상황를 보면서 한명의 시민으로서 
옳고 그름에 관한 가치판단과 함께, 한명의 개발자로서 정부의 HTTPS 차단 방법과 그 우회 방법에 관한 궁금증이 들었다. 이에 전반적인 웹의 구조를 배우고, 
더 나아가 HTTPS 차단 기술과 그 파훼 방법을 알아보고자 한다.

## 2. 웹의 동작 방식
HTTPS 차단 우회 방법을 찾기에 앞서, 보다 근본적인 웹의 동작방식에 대해 알아보고자 한다. 웹은 클라이언트단에서부터 서버단에 이르기까지 수많은 하드웨어, 소프트웨어 기술이 관련되어있기 때문에 모든 과정을 완변히 이해하는 것은 불가능에 가깝다. 따라서 먼저 간략한 웹의 동작 방식을 알아 보고, 그 과정에 있는 핵심 요소를 하나씩 살펴보겠다.


**그럼 이제 가상의 웹서버에 접속하는 과정을 통해 간단히 웹의 동작방식을 알아보자.**
  1. 브라우저 주소창에 test.com을 입력한다.
  2. 브라우저는 DNS서버를 통해 test.com의 실제 주소를 찾고 이 과정에서 재귀적 질의(Recursive Query) 또는 반복적 질의(Iterative Query)가 이루어진다.
  3. DNS서버를 통해 알아낸 test.com의 주소에 접속을 시도한다. 이 과정에서 핸드쉐이크(Handshake)가 이루어지면서 안정적인 통신을 위한 사전 준비가 이루어진다.
  4. 이후, 본격적으로 웹서버에 request를 요청한다.
  5. test.com의 웹서버는 request를 받고 그에 대한 response를 보낸다. 이때 웹서버는 일반적으로 Web Server - WAS - Database Server의 구조를 가진다.
  6. 브라우저는 웹서버의 response를 받고 이를 해석하여 test.com의 페이지를 띄운다.


### 2-1. 도메인 주소를 통해 서버의 IP주소를 알아내는 과정(1~2)
브라우저 주소창에 test.com을 입력하면, 제일 먼저 해당 도메인의 IP 주소가 브라우저와 운영체제의 dns 캐시에 저장되어 있는지 확인한다. dns 캐시에서 해당 도메인을 찾지 못할 경우, 운영체제의 hostname 파일을 확인해본다. (이 과정의 경우 브라우저와 OS마다 조금씩 다를 수 있다. Windows의 경우 dns 캐시와 hostname 파일을 같이 처리한다.)


이렇게 컴퓨터 내부에서 먼저 찾아본 이후, 외부의 dns 서버에 요청을 보낸다. 컴퓨터에 설정된 local dns 서버로 요청을 보내는데, local dns는 일반적으로 isp의 dns 서버로 설정되어있다. local dns는 요청을 받으면 자신에게 test.com의 IP 주소가 있는지 확인 해본다. 해당 도메인의 IP 주소가 있다면 바로 값을 반환해주고, 없다면 Local dns가  root dns로 요청을 보낸다. 이때, 재귀적 질의(Recursive Query) 또는 반복적 질의(Iterative Query)가 이루어진다. 재귀적 질의는 Local dns가 root dns로 요청을 보내면, root dns가 com dns로, com dns가 naver.com dns로 내려가면서 도메인의 IP주소를 질의하는 것을 말한다. 반복적 질의는 Local dns가 root dns로 요청을 보내고, Local dns가 com dns로, Local dns가 naver.com dns로 반복해서 질의하는 것을 말한다.


이제 실제 동작을 통해 위까지의 2-1의 내용을 확인해보자. cmd창에서 ipconfig/all을 통해 현재 컴퓨터의 local dns를 확인해 볼 수 있다.


![local dns 확인](https://i.ibb.co/2ndCPbp/image.png)


필자의 local dns는 182.172.255.180으로 인터넷 제공업체인 Dlive의 dns로 연결되어있다.


실제로 브라우져에 naver.com을 입력하고 일어나는 일을 패킷 캡쳐 프로그램인 wireshark를 통해 보면, dns query가 local dns인 182.172.255.180으로 날라가고 그에 대한 응답이 돌아오는 것을 볼 수 있다.


![dns 통신 확인](https://i.ibb.co/9r1rqmq/2.png)


그럼 여기서 운영체제의 hostname 파일에 naver.com을 넣어보면 어떻게 될까. windows의 hostname 파일인 hosts파일에 naver.com 도메인을 127.0.0.1로 맵핑 시켜 보았다.


![host 파일 수정](https://i.ibb.co/h8QND6d/3.png)


그리고 브라우져에 naver.com을 입력하고 wireshark를 보면, 아까와는 다르게 local dns와 dns query가 오고가지 않는 다는 사실을 확인할 수 있다. hosts 파일에서 해당 도메인을 찾았으므로 그 이후 과정이 일어나지 않은 것이다.


![host 파일 수정](https://i.ibb.co/Ltggnjs/4.png)

