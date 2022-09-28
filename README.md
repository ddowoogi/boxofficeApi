# boxofficeApi
boxofficeApi call
Jquery 박스오피스 불러오기

HTML에서 input과 button 태그를 만든 뒤 자바스크립트에서 Date 객체를 만들고 get 방식으로 JSON 데이터를 받는 과정까지는 이전 포스팅과 동일하게 진행한다.

HTML
<body>
<input type="date" id="date"><button id="mybtn">확인</button>
<div id="boxoffice">
    박스 오피스 순위<br>
</div>
</body>
JavaScript
<script src="http://code.jquery.com/jquery-3.6.0.min.js"></script>
<script>
            $(function() {
                let y = new Date();
                y.setDate(y.getDate()-1);
                let str = y.getFullYear() + "-"
                + ("0" + (y.getMonth() + 1)).slice(-2) + "-"
                + ("0" + y.getDate()).slice(-2);
                $("#date").attr("max",str);


                // 버튼의 클릭 이벤트
                $("#mybtn").click(function() {
                    let d = $("#date").val();//YYYY-MM-dd
                    const regex = /-/g;
                    let d_str = d.replace(regex,"")//YYYYMMdd 

                    let url = "http://kobis.or.kr/kobisopenapi/webservice/rest/boxoffice/searchDailyBoxOfficeList.json?key=f5eef3421c602c6cb7ea224104795888&targetDt="+d_str

                     $.getJSON(url, function(data) {
                         let movieList = data.boxOfficeResult.dailyBoxOfficeList;
                         $("#boxoffice").empty();
                         $("#boxoffice").append(d+" 박스 오피스 순위<br>");
                         for(let i in movieList){
                             $("#boxoffice").append("<div class='movie' id="+movieList[i].movieCd+">"+(parseInt(i)+1)+". "+movieList[i].movieNm+" / "+movieList[i].audiAcc+"명</div><hr>");
                         }
                        });
                });//button click
            });//ready
        </script>
이제 ready function 사이에 클릭 이벤트 발생 시 기능을 구현하는 코드를 추가해주도록 한다. 해당 API로 가져올 수 있는 정보는 무지 많지만 개봉일, 감독, 주연 세 항목만 가져오도록 한다. 주연은 3명까지만 출력한다.

JavaScript
$("#boxoffice").on("click",".movie", function(){
                    let d = $(this);
                    let movieCd = d.attr("id");
                    let url = "http://www.kobis.or.kr/kobisopenapi/webservice/rest/movie/searchMovieInfo.json?key=f5eef3421c602c6cb7ea224104795888&movieCd="+movieCd;
                    $.getJSON(url,function(res){
                        let movie = res.movieInfoResult.movieInfo;
                        d.append("<hr>");
                        d.append("개봉일 : "+movie.openDt+"<br>");
                        d.append("감독 : "+movie.directors[0].peopleNm+"<br>");
                        d.append("주연 : "+movie.actors[0].peopleNm+", "+movie.actors[1].peopleNm+", "+movie.actors[2].peopleNm);
                        d.append("<hr>");

                    })
                })
$("#boxoffice").on("click",".movie", function() : 클래스명이 movie인 태그에 대해 클릭 이벤트 발생시 id가 boxoffice인 태그에 정의한 function 수행
let d = $(this); : $.getJSON 블록 {} 안에서는 this를 사용할 수 없기 때문에 밖에 선언하고 사용
각 태그별 movieCd 값을 가져오는 원리 : 앞의 코드에서 id 값을 movieCd로 설정해 놨기 때문에 this 연산자를 통해 각 태그의 movieCd을 찾고 이를 통해 알맞은 JSON 데이터를 가져옴
JSON 데이터 구조 쉽게 파악하는 법

JSON 데이터를 직접 보면 된다. 이건 크롬에서 URL로 직접 JSON 데이터에 접근한 거다(JSON 데이터 링크). 원래는 이렇게 보기 좋은 형태가 아니지만, 크롬 확장 앱 중 JSONviewer를 설치하면 이런 식으로 구조를 보기 좋게 제공해 준다. 잘 보면 movieInfoResult로 시작해서 movieInfo >> openDt 등으로 이어지는 구조가 보일 것이다.

 

{"이름" : 값} 형태의 데이터는 movieInfoResult.movieInfo.openDt 처럼 .을 이용하면서 접근한다. 배열[] 안에 있는 데이터는 인덱스 번호로 접근해 준다. 예를 들어 김창주라는 값을 얻기 위해 peopleNm에 접근하려면 movieInfoResult.movieInfo.directors[0].peopleNm로 접근한다
