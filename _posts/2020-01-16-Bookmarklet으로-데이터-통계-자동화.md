---
title: "Bookmarklet으로 데이터 통계 자동화"
date: 2020-04-13 17:32:00
categories: Chrome Bookmarklet
---

들어가기 앞서 `Bookmarklet` 이란? 

브라우저에 새로운 기능을 추가하는 JavaScript 명령이 포함 된 웹 브라우저에 저장된 북마크이다 라고 정의가 되어 있다.

[Bookmarklet](https://en.wikipedia.org/wiki/Bookmarklet)

우리가 설정하는 웹 페이지 북 마크 표시가 곧 브라우저의 기능이 될 수도 있다는 건데, 간단한 예시를 들자면

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6b173947-ded9-4507-b52f-2add82090422/sdaf.png](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F6b173947-ded9-4507-b52f-2add82090422%2Fsdaf.png?table=block&id=7d795e05-6b24-40d2-9bf7-40434ca303be&width=2110&cache=v2)

Chrome 브라우저에 등록된 북마크 중 간단한  javascript 가 포함된 북마크를 실행하게되면 위 사진과 같이 알람을 표시할 수 있는 기능을 구현할 수 있게된다.

이를 이용해 노가다 자동화 처리를 진행 할 프로젝트가 생겼다. 초기 버전의 관리자 페이지가 납품되어 업데이트가 없었던 바람에 `Excel 저장 기능` 혹은 `통계 기능`이 빠져 작년 기록들을 모두 수기로 조사해야 할 상황이 생겼다. 이를 해결하기 위해 `Bookmarklet` 을 사용해보기로 하였는데 이유는 다음과 같다.

- 사람이 수기로 조사하기엔 데이터 양이 너무 많음
- 관리자 페이지의 소스 관리가 되지 않아 추가 기능 개발이 어려움
- 원격 솔루션이 없으며, 회사에서 현장까지 방문하기엔 거리가 너무 멂
- 기간이 넉넉치 않아 간단하고 빠르게 구현을 해야 함

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a215bc37-647d-447f-b5eb-9833ae6691e5/fdsafsdafasdf.png](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fa215bc37-647d-447f-b5eb-9833ae6691e5%2Ffdsafsdafasdf.png?table=block&id=d7bf9d44-222e-40ae-9627-18a972b7ca69&width=2330&cache=v2)

위와 같이 특정 날짜를 기반으로 조회시 나오는 데이터들을 통계를 얻고자 하는 조건에 맞게 계산하여 결과를 알려주면 되는 것이 목적이다.

사용자가 검색하고자 하는 날짜 범위를 입력받아야 했으므로 

    var startDate = prompt("검색 시작 날짜\n입력 포멧: yyyy-MM-dd\n예시: 2020-01-16", "2020-01-16");
    var endDate = prompt("검색 마감 날짜\n입력 포멧: yyyy-MM-dd\n예시: 2020-01-16", "2020-01-16");

같이 입력을 받았고, 위 입력받은 내용을 토데로 `HTTP` 를 통해 데이터를 불어와야 했으므로 `ajax` 를 사용하였다. `ajax` 가 가능했던 이유는 본 페이지가 `jquery` 를 사용하고 있었기 때문에 `jquery` 라이브러리를 따로 `HTML injection` 을 할 필요는 없었다.

    $.ajax({
                url: '<API PATH>',
                method: "POST",
                dataType: 'json',
                headers: {
                    'Content-Type': 'application/x-www-form-urlencoded; charset=UTF-8',
                },
                data: {
                    startDate: startDate, endDate: endDate, 
                },
                success: function(data) {

`HTTP` 요청 후 받은 결과는 `JS` 코드로 사용자가 원하는 방식데로 잘 짜주면 되었다

일단 소스를 짰으니 배포를 해야한다. 이 부분에서 `Bookmarklet` 이 진가를 발휘하는데, 기존에 시스템을 업그레이드 작업할 필요 없이, 간단하게 `Chrome` 브라우저에서 북마크를 추가하는 방식만으로도 배포가 가능하다.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9982d014-fab3-4e7f-967d-42ee0754a230/1111111111.png](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F9982d014-fab3-4e7f-967d-42ee0754a230%2F1111111111.png?table=block&id=563eff8f-4daa-43e3-8b65-e2f3ad1db00d&width=1490&cache=v2)

`Chrome` 의 우상단 `:` 버튼 클릭 → 북마크 → 북마크 관리자

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/de39600b-a5d0-4402-9440-60b2fe0fe12d/11111111121.png](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fde39600b-a5d0-4402-9440-60b2fe0fe12d%2F11111111121.png?table=block&id=f468c17a-db9e-481d-a877-e2d9efff4104&width=1000&cache=v2)

북마크 관리자 페이지에서 우상단 `:` 버튼 클릭 → 새 북마크 추가

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/870554aa-13ad-4ca7-bbb3-698aeb59e5c1/111111111221.png](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F870554aa-13ad-4ca7-bbb3-698aeb59e5c1%2F111111111221.png?table=block&id=c577f08d-1132-48cd-9e9c-eaf7adecbcac&width=1060&cache=v2)

이름은 자유롭게, URL 에는 북마크 소스 작성한 내용을 넣어 주면 완료

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a1367ad1-3d3b-48f5-b956-d1130cb396b7/ezgif-7-04c9a6f664d2.gif](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/a1367ad1-3d3b-48f5-b956-d1130cb396b7/ezgif-7-04c9a6f664d2.gif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20200413%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20200413T084642Z&X-Amz-Expires=86400&X-Amz-Signature=f24bd2c075110d35d9515d2863b5c13ce181ab63369ac75dd3f6159bb29c4351&X-Amz-SignedHeaders=host)

그리하여 완성된 모습은 위와 같다.

여기서 또 다른 예기치 못한 문제가 나오게 되는데, 

    alert('');

자체의 함수는 문자열 길이 제한이 없지만 브라우저에서 제한을 한다는 것이었다.

[How many characters allowed in an alert box - javascript](https://stackoverflow.com/a/6864674/7270469)

제기랄, 이 때문에 다른 방법으로 결과 내용을 `.csv` 파일을 다운로드 할 수 있게끔 만들어 주기로 하였다. `.csv` 파일 자체는 `,` 와 `\n` 으로 이루어져 있기 때문에 간단히 만들어 낼 수 있었고, 다운로드는 jquery 를 이용하여 `<a>` 태그 생성 후 누르기 방식을 사용하였다.

    $('<a></a>')
    	.attr('id','downloadFile')
    	.attr('href','data:text/csv;charset=utf8,' + encodeURIComponent(csv))
    	.attr('download',`filename.csv`)
    	.appendTo('body');
    
    	$('#downloadFile').ready(function() {
    		$('#downloadFile').get(0).click();
    		$('#downloadFile').get(0).remove();
    	});

부가적으로 `.csv` 를 엑셀 프로그램에서 읽으려면 한글이 깨져서 보이는 경우가 있는데 이는

[엑셀 CSV파일 한글깨짐 현상 해결방법](https://m.blog.naver.com/PostView.nhn?blogId=bonnyborn&logNo=221217991960&proxyReferer=https%3A%2F%2Fwww.google.com%2F)

위 링크데로 조치하면 된다

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e930d748-e021-45f5-917c-2d863cc7c53e/Image_from_iOS.jpg](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fe930d748-e021-45f5-917c-2d863cc7c53e%2FImage_from_iOS.jpg?table=block&id=25e04b21-6df5-46a0-b200-02fa0888e808&width=2020&cache=v2)

`Bookmarklet` 참조 링크

[Making Bookmarklets](https://gist.github.com/caseywatts/c0cec1f89ccdb8b469b1)

다만 `Bookmarklet` 기능을 구현할 땐 주의해야 할 사항이 있는데,  `IDE` 에서 작성할 땐 각 줄 끝에 `;` 를 작성하지 않아도 동작하였지만, `Bookmarklet` 에 배포 후 실행하려 하면 JS 코드가 모두 한줄로 바뀌므로 문법 오류가 발생할 수 있으니 꼭 각 줄 끝엔 `;` 를 추가하도록 하자 또한 중간에 `//` 과 같은 주석처리가 있었다면 삭제를 하거나 `/**/` 로 변경하도록 하자
