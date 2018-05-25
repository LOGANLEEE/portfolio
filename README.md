mysql을 수동으로 깔았고 그 학원에서의 설정을 그대로 가져왔기 떄문에 소스가 엉켜 개판이었음.

그래서 export query로 제대로 안됬는지는 모르지만 일단 정리가 제대로 안되고 꼬일대로 꼬여서.

아예 완전히 밀기로 결심 그리고 brew로 install하기로 결정.

- brew search mysql
brew install mysql 후에 문제점은 서버 시작이 mysql.server start 던 brew servcies start mysql이던
오류를 내뱉는다는거 전자는 에러가 명시되고 후자는 시작했다고 말만한다.
https://stackoverflow.com/questions/44433052/mysql-on-osx-sierra-cant-start-the-server-quit-without-updating-pid-file 여기에서 나와 비슷한 증상을 겪는 글을 발견.

실험에 들어간다.

sudo /usr/local/mysql/bin/mysqld_safe
mysqld_safe Logging to '/usr/local/mysql/data/My-iMac.local.err'.
Starting mysqld daemon with databases from /usr/local/mysql/data
mysqld_safe mysqld from pid file /usr/local/mysql/data/My-iMac.local.pid ended

구문을 실행하고 나서는 brew services start mysql로 실행이 된다. 그리고 mysql.server start로 된다
근데 나는 brew로 install하였으니 왠만하면 brew 명령어를 사용하고자한다.

아나...
ERROR 1290 (HY000): The MySQL server is running with the --secure-file-priv option so it cannot execute this statement

이 에러가 날 죽인다.

/usr/local/Cellar/mysql/5.7.22/.bottle/etc/my.cnf
수정해서 secure-file-priv =“” 혹은 경로를 /Users/Logna/dataexpo/mysql_export/로 경로를 줬음에도 불구하고 도저히 들어먹질 않는다.

SELECT @@GLOBAL.secure_file_priv; 를 쓰면 주구장창 NULL만 뱉는다.

LoganLeeui-MacBook-Pro:/ Logan$ sudo find . -name *mysql.plist 
Password:
./usr/local/Cellar/mysql/5.7.22/homebrew.mxcl.mysql.plist
./System/Library/DirectoryServices/DefaultLocalDB/Default/groups/_mysql.plist
./System/Library/DirectoryServices/DefaultLocalDB/Default/users/_mysql.plist
./private/var/db/dslocal/nodes/Default/groups/_mysql.plist
./private/var/db/dslocal/nodes/Default/users/_mysql.plist
./Users/Logan/Library/LaunchAgents/homebrew.mxcl.mysql.plist

이렇게 결과가 있는데 plist들을 하나씩 까볼려고 마음먹었지만 vi로 plist를 열면 깨져버린다. 방법이 있겠지.


별지랄지랄을 다 떨다가 결국 sudo mysqld —secure-file-priv=/Users/Logan
이거 한방으로 해결했다....아무리 어떤 my.cnf던 plist를 고치던 안되더라 아오..
https://dev.mysql.com/doc/refman/5.7/en/server-options.html
여기서 읽고 해결함

깨닫은점 일단은 정식 홈페이지 레퍼런스부터 보고와야겠다.

일단 mapper랑 parser를 더 만져야하는데.. 한국어 자연어처리를 고민해야봐야될것같다.

희소식은 구글 자연어처리 api가 있다는것이나 그에 대한 개념을 하나도 모른다는게 문제이다. 파서도 개판이고 맵퍼 좀 다듬고 리듀서도 전반적으로만 다듬으면 
괜찮은값을 얻을 수 있을거라 생각하며 결과를 R에 뿌려 시각화자료를 만들고 chart.js를 통해서도 시각화하여 웹에 뿌려줄수만 있다면 
충분히 훌륭한 포폴이 되리라 믿고.
6월달 이내로 얼른 취직하고싶다.

======
export 버튼을 추가해야한다.
Error Pasing a record : For input string: “;90745;" 이런 에러가 뜨는데 맵퍼를 바꾸고나서 계속 뜬다.
그리고 lines terminated by ‘\n’로 진행하는데 
export된 데이터값에는 NULL대신  \N이 뜬다 허참..

흠 parser를 맵퍼에 추가하고나서는 에러가 뜨는데 
하둡 로그를 살펴보자. parser를 새로 만들었으니 에러가 나느게아닐까. 라는 생각이 든다.
그 이유는 내가 새로 export한 파일을 hadoop에 올리지않아 생기는 어이없는 에러였다.

parser,mapper를 refactroing했고, 처음 만난 에러는 숫자가 있어야될 자리에 string이 위치했던건데 131번줄에 \ 문자를 사용해 잘못인식됬던것이다.
crawling과정에서 유효성검사를 심어놓아야할거같다
java.lang.Exception: java.lang.ArrayIndexOutOfBoundsException: 1
라는 에러가 발생. 원인이 어딘지 알려주지는 않으나 array가 들어간건 reg_date랑 date랑 time밖에 없는데 이것 인덱스수가 모자를 수가없는데 왜 났을까.
ArrayIndexOutOfBoundsException이 나는 이유는 Thrown to indicate that an array has been accessed with an illegal index. The index is either negative or greater than or equal to the size of the array.

Date's indexs = 2018
Date's indexs = 05
Date's indexs = 09
로 정상적으로 잘 쪼개져서 출력이되는데 왜 mapreduce만 돌리면 Month에서 터지는지.

계속 5월9일자 글들만 가져오길레 이상해서보니 야갤주소가 이전되었음
DcController에서 새주소로 변경후 크롤링을 진행하니 이상하게 2017-11-11 17시로 글등록날짜가 저장됨.
코드상으로 태그 위치는 틀림이없음. 어디서부터 뭐가 잘못된건지 이클립스 사파리 mvn패키징까지 다해봤으나 증상은 동일.
** 해결함. 메인주소는 new7으로 바꿧으나 title_no를 받아오는 코드에는 여전히 6으로 되어있어 수정함.

Exported file doesn't exist in local. Please Check it again.라는 에러로 파일을 못찾고있음.
이 부분을 만지면 웹상에서 버튼으로만 조작가능.
Path를 수정해야함.

=====
만약에 내 패턴을 빅데이터 분석을 통해 결과를 도출해낼수있는 포폴을 만들어 제출하는것도 괜찮을것같긴하다.

==== 

filesystem을 local용으로 하나 hdfs용으로 만들어놓고 local에서 유효성검사후 copy메소드는 hdfs용으로 실행하니 잘된다. 물론 conf에 core-site.xml을 물려줬음.
if에서 local로 작업을 하고 나서 hdfs가 들어가버리니까 local로만 인식을해 hdfs로 접근이 안됬던것같다.

이제 결과값을 챠트나 알에 넣어서 시각화시켜야됨.
날짜단위로 짤라서 언제 작성을 많이했는지로 시간별로 분류해보자.

일단 hdfs 메소드를 실행할때 local에서 hdfs로 업로드하는 작업후에 hdfs에서 part-r-00000를 local로 내리는 메소드가 두개가 있는데 잘못된게 mapreduce된 part-r-00000을 가져와야하는데
이전에 작업된 part-r-00000가 로컬에 옴.
그래서 hadoop컨트롤러에 붙힐려고했는데 또 Wrong FS 에러가 나와주심 
