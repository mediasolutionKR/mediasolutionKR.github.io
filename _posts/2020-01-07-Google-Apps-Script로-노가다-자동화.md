---
title: "Google Apps Script로 노가다 자동화"
date: 2020-04-13 17:42:00
categories: Google Apps Script
---

단순하지만 수기로 작업을 해왔던 자잘한 것들을 자동화 해보자

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/cb1ba0e9-768f-4676-bb32-b41cb701e024/s1.png](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fcb1ba0e9-768f-4676-bb32-b41cb701e024%2Fs1.png?table=block&id=7d909b00-48f2-4a62-ab24-24ec57e87dcc&width=3050&cache=v2)

위 사진은 납품된 서비스 유지보수 차원에서 문의 전화를 받을 때 마다 기록을 해왔던 일지이다.

이 부분에서 약간의 불만이 있었는데

1. 이런 사소한 작업들이 모이다보니 개발자가 정작 개발에 신경쓸 시간이 적어 진다.
2. 문의 접수를 대부분의 경우 접수를 하려는 사람이 접수를 하는데 그걸 접수 받는 사람이 대신 해주고 있다.

그래서 해결을 한 방식이 바로 구글 설문조사를 이용하는 것

일단 완성된 모습은

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/aa8316d4-4222-4f5f-b1ec-a7f8fa7704ff/s6.png](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Faa8316d4-4222-4f5f-b1ec-a7f8fa7704ff%2Fs6.png?table=block&id=e7fe407e-c3c4-43f6-9a93-364d07a9766a&width=2350&cache=v2)

전체 구조는 다음과 같다

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ba523776-f7f8-4c99-88b7-4bcc8c2faec3/__.jpg](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fba523776-f7f8-4c99-88b7-4bcc8c2faec3%2F__.jpg?table=block&id=59612681-cd9c-4383-9781-7d1a1526ecb3&width=2160&cache=v2)

본 솔루션을 구현하기 위해 Firebase를 사용하였는데 그 이유는 홈페이지 호스팅을 하는데 간단하고 빨랐으며 무료 가격 정책에도 준수한 성능을 갖추었기 때문이다.

문의 접수 시나리오는 다음과 같다.

1. 문의 접수 홈페이지에 사용자가 접속 한다.
2. 문의 접수 Google-Forms (설문조사) 에 응답한다.
3. 새로운 문의 접수가 Google-Forms에 연결된 Google Sheets 에 먼저 저장이 된다
4. Google Sheets 에 새 문의자 저장 되자마자 트리거가 발생되어 해당 트리거와 연결된 Google Apps Script 가 실행된다.
5. Google Apps Script 에서 기존에 쓰이던 문의 접수 일지 Sheets 에 새 문의 내용을 작성해준다.
6. Google Apps Script 에서 새 문의 내용을 메일, 슬랙에 전달한다.

Google Apps Script 가 어떻게 보면 제일 핵심 요소가 될 수 있지만 작업된 양은 그리 많지 않았다.

    function myFunction() {
      
    }
    
    function onRowAppend(e) {
      var sheet = SpreadsheetApp.openById('SPREAD_SHEEDT_ID').getSheetByName('2020')
      
      var data = {
        timestamp: e.namedValues['Timestamp'][0],
        site: e.namedValues['Site'][0],
        place: e.namedValues['Place'][0] == '' ? e.namedValues['Site'][0] : e.namedValues['Place'][0],
        reason: e.namedValues['Reason'][0],
        contact: e.namedValues['Contact'][0]
      }
      
      sheet.appendRow([data.timestamp, data.site, data.place, data.reason, data.contact])
      
      MailApp.sendEmail("test@example.com",
                       "--- 문의 접수 - " + data.site + " / " + data.place,
                        "접수 날짜: " + data.timestamp + "\n사유: " + data.reason + "\n연락처: " + data.contact)
      MailApp.sendEmail("test@example.com",
                       "--- 프로그램 문의 접수 - " + data.site + " / " + data.place,
                        "접수 날짜: " + data.timestamp + "\n사유: " + data.reason + "\n연락처: " + data.contact)
      
      
      var slackUrl = "https://example.com";
      var slackMsgForm = {
        "attachments": [
          {
            "color": "#36a64f",
            "pretext": "새로운 문의 접수 - " + data.site + " / " + data.place,
            "author_name": data.contact,
            "title": "문의 내용",
            "text": data.reason,
          }
        ]
      }
       var payload = JSON.stringify(slackMsgForm);
    
       var headers = { "Accept":"*", 
                  "Content-Type":"application/json", 
                 };
      var options = { "method":"POST",
                 "contentType" : "application/json",
                "headers": headers,
                "payload" : payload
               };
       var response = UrlFetchApp.fetch(slackUrl, options);
      Logger.log("slack request result: " + JSON.stringify(response))
    }

Javascript 기반으로 작동되니 코딩 난이도도 높진 않았지만 문제는 API 문서 해석이었다.

[Reference overview | Apps Script | Google Developers](https://developers.google.com/apps-script/reference)

겁나 많다. 모든 API를 보려 하지 않았고, 필요한 내용만 구글링으로 짜집었더니 그리 어렵지 않게 완성할 수 있었다.

소스는 완성했으니 그럼 저 스크립트는 언제 어떻게 실행을 해야 할까?

일단 조건은 설문조사용 Spread Sheets 에 값이 추가되었을 경우 트리거가 일어나면 되는 것이다.

그럼 트리거로 오는 데이터는 어떤걸까?

[Event Objects | Apps Script | Google Developers](https://developers.google.com/apps-script/guides/triggers/events#form-submit)

짜잔~

그럼 트리거는 어떻게 설정해야 할까?

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b96707cc-fe13-4d85-96b3-06011f92d24c/s2.png](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fb96707cc-fe13-4d85-96b3-06011f92d24c%2Fs2.png?table=block&id=fe8c30a4-4b2b-4cfd-9c28-1ea4e21bb404&width=2010&cache=v2)

해당 스크립트 편집기의 수정탭 → `현재 프로직트의 트리거` 로 들어가게 되면 트리거 설정을 할 수 있는 페이지가 열리게 되는데

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/428bd089-3cbf-4199-bbf1-8a18b1564c45/s3.png](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F428bd089-3cbf-4199-bbf1-8a18b1564c45%2Fs3.png?table=block&id=70dbfa00-522d-41f3-a241-163f23773d3a&width=1470&cache=v2)

트리거 추가 시 소스에 작성해두었던 원하는 함수를 `실행할 함수 선택` 에서 골라주면 되고, 이벤트 유형은 설문 양식을 제출하였을 때 발생되기를 원하므로 `양식 제출시` 를 선택해주고 저장을 누르면 이후로 설문 조사가 될 때마다 자동으로 호출이 된다.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/cbaa8d71-4872-4828-aac2-1f6fb580e0c0/s4.png](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fcbaa8d71-4872-4828-aac2-1f6fb580e0c0%2Fs4.png?table=block&id=c5c9b160-3aac-4d9c-82a2-9f8a134dfba6&width=1890&cache=v2)

자동으로 써진 모습

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9d8bba25-b710-443c-b3d7-882c4cdd01f1/s5.png](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F9d8bba25-b710-443c-b3d7-882c4cdd01f1%2Fs5.png?table=block&id=ba95bf13-9f69-468c-8037-ce9cd8871bfe&width=1150&cache=v2)

슬랙으로도 알림이 발송되는 모습을 볼 수 있다.
