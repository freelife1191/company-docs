## OSS 서비스 요청처리 FLOW
```uml
participant 사용자
participant OSS웹
participant "OSS-API" as ossApi
participant "OSS-DB" as ossDb
participant "IPARKING-LINK-API" as iparkingLinkApi
participant "IPARKING-DB" as iparkingDb
participant "DOORAY-MESSENGER" as doorayMessenger

사용자 -> OSS웹: OSS 서비스 요청처리

activate OSS웹
OSS웹 -> ossApi: POST /v1/service/메인코드/상세코드\nOSS 서비스 요청처리
activate ossApi
ossApi -> ossDb: OSS 요청 서비스 검색
activate ossDb
ossDb -[#0000FF]-> ossApi: OSS 요청 서비스 검색결과
deactivate ossDb

ossApi -> doorayMessenger: OSS 서비스 요청 Dooray Messenger 알림

alt OSS 요청 서비스 롤백 가능여부가 true 이면
ossApi -> iparkingLinkApi: OSS 서비스 롤백 데이터 조회 요청
activate iparkingLinkApi
iparkingLinkApi -> ossApi: OSS 서비스 롤백 데이터 조회 응답
deactivate iparkingLinkApi
end

ossApi -> ossDb: OSS 서비스 요청처리 이력등록(JSON)
activate ossDb
ossDb -[#0000FF]-> ossApi: OSS 서비스 요청처리 이력등록완료(JSON)
deactivate ossDb

ossApi -> iparkingLinkApi: iparking 서비스 처리요청
activate iparkingLinkApi
iparkingLinkApi -> iparkingDb: iparking 서비스 처리요청
activate iparkingDb
iparkingDb -[#0000FF]-> iparkingLinkApi: iparking 서비스 처리완료
deactivate iparkingDb
iparkingLinkApi -[#0000FF]-> ossApi: iparking 서비스 처리완료
deactivate iparkingLinkApi

ossApi -[#0000FF]-> OSS웹: OSS 서비스 처리완료
deactivate ossApi

OSS웹 -[#0000FF]-> 사용자: OSS 서비스 처리완료
deactivate OSS웹
```

## OSS 서비스 롤백 요청처리 FLOW
```uml
participant 사용자
participant OSS웹
participant "OSS-API" as ossApi
participant "OSS-DB" as ossDb
participant "IPARKING-LINK-API" as iparkingLinkApi
participant "IPARKING-DB" as iparkingDb
participant "DOORAY-MESSENGER" as doorayMessenger

사용자 -> OSS웹: OSS 서비스 롤백 요청처리

activate OSS웹
OSS웹 -> ossApi: POST /v1/service/메인코드/상세코드/rollback\nOSS 서비스 롤백 요청처리
activate ossApi
ossApi -> ossDb: OSS 요청 서비스 롤백 데이터 검색
activate ossDb
ossDb -[#0000FF]-> ossApi: OSS 요청 서비스 롤백 데이터 검색결과
deactivate ossDb

alt 롤백 처리 여부: false, 요청 서비스 롤백 데이터 처리내역 이후 2건 미만일 경우 진행

ossApi -> doorayMessenger: OSS 서비스 롤백 처리요청 Dooray Messenger 알림

ossApi -> iparkingLinkApi: iparking 서비스 롤백 처리요청
activate iparkingLinkApi
iparkingLinkApi -> iparkingDb: iparking 서비스 롤백 처리요청
activate iparkingDb
iparkingDb -[#0000FF]-> iparkingLinkApi: iparking 서비스 롤백 처리완료
deactivate iparkingDb
iparkingLinkApi -[#0000FF]-> ossApi: iparking 서비스 롤백 처리완료
deactivate iparkingLinkApi

ossApi -> ossDb: OSS 서비스 롤백 이력등록(JSON)
activate ossDb
ossDb -[#0000FF]-> ossApi: OSS 서비스 롤백 이력등록완료(JSON)
deactivate ossDb

end

ossApi -[#0000FF]-> OSS웹: OSS 서비스 롤백 처리완료
deactivate ossApi

OSS웹 -[#0000FF]-> 사용자: OSS 서비스 롤백 처리완료
deactivate OSS웹
```