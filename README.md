# webhacking.kr(old) write-ups 
<br>

## old-01

__쿠키 인젝션__

```php
<?php  
    if(!is_numeric($_COOKIE['user_lv'])) $_COOKIE['user_lv']=1;  
    if($_COOKIE['user_lv']>=6) $_COOKIE['user_lv']=1;  
    if($_COOKIE['user_lv']>5) solve(1);  
    echo "<br>level : {$_COOKIE['user_lv']}";  
?>  
```

쿠키 user_lv의 값을 조건에 맞게 바꿔야 한다.
쿠키는 5 초과 6 미만인 값이어야 하므로, 그 사이의 값(ex 5.5)을 넣으면 성공.  

![pwned](./pwned/old-01.PNG)  
<br>
  
## old-02*

__블라인드 SQL 인젝션__  

`<!-- if you access admin.php i will kick your ass -->` 따라 들어가보자.  

![02-input](./pic/02-input.PNG)  

input 필드가 나온다. admin에 접근하려면 password를 알아내야 한다.  
페이지의 소스코드 주석의 시간과 time 쿠키를 단서로 하여  쿠키의 값을 몇 번 고쳐보니,
쿠키에 넣은 논리값이 거짓일 때는 `2070-01-01 09:00:00`로, 참일 때는  `2070-01-01 09:00:01`로 바뀐다.

<br>

## old-03

__SQL 인젝션__

![03-nono](./pic/03-nono.PNG)  

..? 노노그램? 답을 입력하니 다음과 같은 input 필드가 나온다. 

![03-input](./pic/03-input.PNG)

소스코드를 확인해보니 answer에 들어가는 hidden 필드가 있다.  
`<form method=post action=index.php><input type=hidden name=answer value=1010100000011100101011111>`   
여기에 '를 넣어보니 쿼리 에러가 발생해 SQL 인젝션으로 값이 항상 참이 되게 하면 성공.

![03-SQL](./pic/03-SQL.PNG)

![pwned](./pwned/old-03.PNG)

<br>

## old-04

__레인보우테이블__

![04-hash](./pic/04-hash.PNG)

해시 값을 디코딩해서 패스워드 값을 찾는 것 같다.

```php+HTML
<?php
	sleep(1); // anti brute force
	if((isset($_SESSION['chall4'])) && ($_POST['key'] == $_SESSION['chall4'])) solve(4);
	$hash = rand(10000000,99999999)."salt_for_you";
	$_SESSION['chall4'] = $hash;
	for($i=0;$i<500;$i++) $hash = sha1($hash);
?>
...
<tr><td colspan=3 style=background:silver;color:green;><b><?=$hash?></b></td></tr>
```

세션값은 10000000~99999999사이의 임의의 값 + salt_for_you, 출력된 값은  이를 500번 sha1 인코딩 한 값이다.  
그러나 sha1은 단방향 암호화 알고리즘으로 500번이나 디코딩할 수 없다.  
브루트 포스 공격도 불가능하므로 파이썬으로 레인보우 테이블을 만들어주면 될듯하다.

__exploit__: [old-04.py](./code/old-04.py)

![04-rainbowtable](./pic/04-rainbowtable.PNG)

(ㅎㄷㄷ)

![pwned](./pwned/old-04.PNG)

<br>

## old-05

__SQL 절단 공격__

login으로 들어가면 다음과 같이 mem 디렉터리가 노출되며 디렉터리 인덱싱이 가능하다.



![05-login](./pic/05-login.PNG)

/mem으로 접속하면 접속이 거부되었던 join.php에 접근할 수 있게 된다.

![05-mem](./pic/05-mem.PNG)

join.php의 소스코드를 보면 암호화가 되어 있다.
```javascript
<script>
l='a';ll='b';lll='c';llll='d';lllll='e';llllll='f';lllllll='g';llllllll='h';lllllllll='i';llllllllll='j';lllllllllll='k';llllllllllll='l';lllllllllllll='m';llllllllllllll='n';lllllllllllllll='o';llllllllllllllll='p';lllllllllllllllll='q';llllllllllllllllll='r';lllllllllllllllllll='s';llllllllllllllllllll='t';lllllllllllllllllllll='u';llllllllllllllllllllll='v';lllllllllllllllllllllll='w';llllllllllllllllllllllll='x';lllllllllllllllllllllllll='y';llllllllllllllllllllllllll='z';I='1';II='2';III='3';IIII='4';IIIII='5';IIIIII='6';IIIIIII='7';IIIIIIII='8';IIIIIIIII='9';IIIIIIIIII='0';li='.';ii='<';iii='>';lIllIllIllIllIllIllIllIllIllIl=lllllllllllllll+llllllllllll+llll+llllllllllllllllllllllllll+lllllllllllllll+lllllllllllll+ll+lllllllll+lllll;
lIIIIIIIIIIIIIIIIIIl=llll+lllllllllllllll+lll+lllllllllllllllllllll+lllllllllllll+lllll+llllllllllllll+llllllllllllllllllll+li+lll+lllllllllllllll+lllllllllllllll+lllllllllll+lllllllll+lllll;if(eval(lIIIIIIIIIIIIIIIIIIl).indexOf(lIllIllIllIllIllIllIllIllIllIl)==-1) {alert('bye');throw "stop";}if(eval(llll+lllllllllllllll+lll+lllllllllllllllllllll+lllllllllllll+lllll+llllllllllllll+llllllllllllllllllll+li+'U'+'R'+'L').indexOf(lllllllllllll+lllllllllllllll+llll+lllll+'='+I)==-1){alert('access_denied');throw "stop";}else{document.write('<font size=2 color=white>Join</font><p>');document.write('.<p>.<p>.<p>.<p>.<p>');document.write('<form method=post action='+llllllllll+lllllllllllllll+lllllllll+llllllllllllll+li+llllllllllllllll+llllllll+llllllllllllllll
+'>');document.write('<table border=1><tr><td><font color=gray>id</font></td><td><input type=text name='+lllllllll+llll+' maxlength=20></td></tr>');document.write('<tr><td><font color=gray>pass</font></td><td><input type=text name='+llllllllllllllll+lllllllllllllllllllllll+'></td></tr>');document.write('<tr align=center><td colspan=2><input type=submit></td></tr></form></table>');}
</script>
```

코드를 예쁘게 정리해 보자. https://beautifier.io/  
js beautifulier와 개발자 도구를 이용해서 해석해보면 다음과 같다.
```javascript
<script>
    if (document.cookie.indexOf(oldzombie) == -1) {
        alert('bye');
        throw "stop";
    }
    if (document.URL.indexOf(mode=1) == -1) {
        alert('access_denied');
        throw "stop";
    } else {
...
</script>
```
oldzombie 쿠키가 있어야 하며 url에 mode=1이 있어야 한다는 조건을 맞춰주니 회원가입하는 창이 나타났다.

![05-join](./pic/05-join.PNG)

admin으로 로그인해야 하는데 이는 이미 있는 id라고 나온다. max length를 20보다 크게 바꿔줘 아무 문자나 입력하면 이후 문자열을 자르고 admin만 받아들인다.

![pwned](./pwned/old-05.PNG)

## old-06

___base64___

소스코드를 요약하면,20번 base64 인코딩하고 규칙대로 replace한 값인 user 쿠키와 password 쿠키의 값을 거꾸로 replace하고 20번 base64 디코딩한 값이 각각 admin, nimda와 같으면 풀린다.   
쿠키에 들어갈 값을 만들어보자.  

__exploit__: [old-06.py](./code/old-06.py)

변환한 값을 쿠키에 넣어주면 성공.  
![pwned](./pwned/old-06.PNG)

<br>

## old-07

___SQL 인젝션___

```php
<?php
$go=$_GET['val'];
...
$db = dbconnect();
$rand=rand(1,5);
if($rand==1){
  $result=mysqli_query($db,"select lv from chall7 where lv=($go)") or die("nice try!");
}
if($rand==2){
  $result=mysqli_query($db,"select lv from chall7 where lv=(($go))") or die("nice try!");
}
...
$data=mysqli_fetch_array($result);
if(!$data[0]) { echo("query error"); exit(); }
if($data[0]==1){
  echo("<input type=button style=border:0;bgcolor='gray' value='auth' onclick=\"alert('Access_Denied!')\"><p>");
}
elseif($data[0]==2){
  echo("<input type=button style=border:0;bgcolor='gray' value='auth' onclick=\"alert('Hello admin')\"><p>");
  solve(7);
}
?>
```

소스코드를 확인해보면 GET방식으로 받은 값을 정규표현식을 통해 필터링을 거쳐 5분의 1확률로 성공하는 SQL 쿼리문에 넣는 것 같다.  
마지막 부분을 보면 $data[0]을 2로 만들어야 문제가 해결되므로 UNION을 이용해 앞의 쿼리문을 거짓으로 만들고 뒤의 쿼리문을 참으로 만들어야 한다.  

따라서 `val=0)UNION SELECT 2` 와 같이 쓰고 싶었는데... 다음 정규표현식이 이를 필터링해 두가지 문제점이 있었다.  
`if(preg_match("/2|-|\+|from|_|=|\\s|\*|\//i",$go)) exit("Access Denied!");`

1. 띄어쓰기가 안된다. 정규표현식이 공백(\s)을 막고 있으므로 괄호를 사용해야 할듯하다.
2. 2를 사용하지 못한다. 정규표현식이 사칙연산과 2를 막고 있으므로 2를 직접적으로 입력하지 못한다. 따라서 char(50)등의 방법으로 우회해봐야 할듯하다.  
   
결국 `val=0)UNION(SELECT(CHAR(50))` 과 같이 작성해서 5번정도 새로고침했더니 성공.  

![pwned](./pwned/old-07.PNG)

<br>

## old-08

___SQL 인젝션___

```PHP
<?php
...
$result = mysqli_query($db,"select id from chall8 where agent='".addslashes($_SERVER['HTTP_USER_AGENT'])."'");
$ck = mysqli_fetch_array($result);

if($ck){
  echo "hi <b>".htmlentities($ck[0])."</b><p>";
  if($ck[0]=="admin"){
    mysqli_query($db,"delete from chall8");
    solve(8);
  }
}

if(!$ck){
  $q=mysqli_query($db,"insert into chall8(agent,ip,id) values('{$agent}','{$ip}','guest')") or die("query error");
  echo("<br><br>done!  ({$count_ck[0]}/70)");
}
?>
```
소스코드를 확인하니 HTTP_USER_AGENT 값을 이용해서 SQL 쿼리문을 작성하였다.
id에 admin이 있으면 solve, 없으면 agent를 이용해 INSERT를 하므로, 프록시 도구(burp suite)를 이용해 User-Agent 값을 `dummyagent', 'dummyip', 'admin'), ('0` 으로 바꿔주었다.

![08-insert](./pic/08-insert.PNG)

이는 실제 INSERT문에서는 다음과 같이 동작할 것이다.

```SQL 
INSERT INTO chall8(agent,ip,id) VALUES('dummyagent', 'dummyip', 'admin'), ('0', '{$ip}', 'guest')
```

따라서 뒷부분의 원래 쿼리는 '0'으로 추가되지 않고, 앞부분의 (dummyagent, dummyip, admin)만 성공적으로 추가된다.

![08-done](./pic/08-done.PNG)

이다음 agent를 insert한 값인 dummyagent로 바꿔주면 성공.

![08-select](./pic/08-select.PNG)

![pwned](./pwned/old-08.PNG)

<br>

# old-09*

___Blind SQL 인젝션___

![main](./pic/09-main.PNG)

1,2를 눌러보면 각각 Apple, Banana가 출력된다.

![secret](./pic/09-secret.PNG)

3을 눌러보면 다음과 같이 출력된다.

SQL 인젝션이 잘 막혀있어서 Blind SQL 인젝션을 시도해야 할 것 같다.

<br>

# old-10
___개발자도구___

![main](./pic/10-main.PNG)

(?!?!?!) ~~오바~~

O 를 클릭해보면 1px씩 앞으로 간다.
일일이 다 클릭해서 Goal 로 보낼 순 없으니 개발자도구를 이용해 위치를 바꿔보자.

![hackme](./pic/10-hackme.PNG)

다음과 같이 hackme 태그가 Goal을 넘어가게 되면 성공.

![hacked](./pic/10-hacked.PNG)

<br>

# old-11

___정규표현식___

```PHP
<?php
  $pat="/[1-3][a-f]{5}_.*$_SERVER[REMOTE_ADDR].*\tp\ta\ts\ts/";
  if(preg_match($pat,$_GET['val'])){
    solve(11);
  }
  else echo("<h2>Wrong</h2>");
  echo("<br><br>");
?>
```

GET 형식으로 전달받는 `$val`이 `$pat` 정규표현식과 일치하면 solve된다.

정규표현식의 동작을 확인하자. https://regex101.com/

|Code |Meaning|Example|
|------|:----:|----:|
|`[1-3]`|1-3의 숫자 1개|1|
|`[a-f]{5}`|a-f의 문자 5개|aaaaa|
|`.*x`|x 식이 0번이상 반복|x|
|`$_SERVER[REMOTE_ADDR]`|접속자의 ip주소|218.146.29.164|
|`\tp\ta\ts\ts`|\t는 tab이므로 URL 인코딩으로 치환|%09p%09a%09s%09s|

(해당 ip주소는 문제풀이시점의 ip)

![backslash](./pic/11-backslash.PNG)

`1aaaaa_(ip주소)  p a s s`와 같은 값이 전달되어야 하므로 tab을 소스코드 그대로 `\t` 로 처리해보았더니 그대로 Wrong이었다. 이유는 문제에서 우리는 GET 방식으로 값을 전달하므로 tab을 URL 인코딩하여 전달해야 하기 때문이다. 

![url](./pic/11-url.PNG)

따라서 `?val=1aaaa_(ip주소)%09p%09a%09s%09s` 와 같이 전달해주면 성공.

![pwned](./pwned/old-11.PNG)

<br>

# old-12

___javascript___

```javascript
<html>
<head>
<title>Challenge 12</title>
<style type="text/css">
body { background: black; color:white; font-size:10pt; }
</style>
</head>
<body>
<script>
ﾟωﾟﾉ= /｀ｍ´）ﾉ ~┻━┻   //*´∇｀*/ ['_']; o=(ﾟｰﾟ)  =_=3; c=(ﾟΘﾟ) =(ﾟｰﾟ)-(ﾟｰﾟ); (ﾟДﾟ) =(ﾟΘﾟ)= (o^_^o)/ (o^_^o);(ﾟДﾟ)={ﾟΘﾟ: '_' ,ﾟωﾟﾉ : ((ﾟωﾟﾉ==3) +'_') [ﾟΘﾟ] ,ﾟｰﾟﾉ :(ﾟωﾟﾉ+ '_')[o^_^o -(ﾟΘﾟ)] ,ﾟДﾟﾉ:((ﾟｰﾟ==3) +'_')[ﾟｰﾟ] }; (ﾟДﾟ) [ﾟΘﾟ] =((ﾟωﾟﾉ==3) +'_') [c^_^o];(ﾟДﾟ) ['c'] = ((ﾟДﾟ)+'_') [ (ﾟｰﾟ)+(ﾟｰﾟ)-(ﾟΘﾟ) ];(ﾟДﾟ) ['o'] = ((ﾟДﾟ)+'_') [ﾟΘﾟ];(ﾟoﾟ)=(ﾟДﾟ) ['c']+(ﾟДﾟ) ['o']+(ﾟωﾟﾉ +'_')[ﾟΘﾟ]+ ((ﾟωﾟﾉ==3) +'_') [ﾟｰﾟ] + ((ﾟДﾟ) +'_') [(ﾟｰﾟ)+(ﾟｰﾟ)]+ ((ﾟｰﾟ==3) +'_') [ﾟΘﾟ]+((ﾟｰﾟ==3) +'_') [(ﾟｰﾟ) - (ﾟΘﾟ)]+(ﾟДﾟ) ['c']+((ﾟДﾟ)+'_') [(ﾟｰﾟ)+(ﾟｰﾟ)]+ (ﾟДﾟ) ['o']+((ﾟｰﾟ==3) +'_') [ﾟΘﾟ];(ﾟДﾟ) ['_'] =(o^_^o) [ﾟoﾟ] [ﾟoﾟ];(ﾟεﾟ)=((ﾟｰﾟ==3) +'_') [ﾟΘﾟ]+ (ﾟДﾟ) .ﾟДﾟﾉ+((ﾟДﾟ)+'_') [(ﾟｰﾟ) + (ﾟｰﾟ)]+((ﾟｰﾟ==3) +'_') [o^_^o -ﾟΘﾟ]+((ﾟｰﾟ==3) +'_') [ﾟΘﾟ]+ (ﾟωﾟﾉ +'_') [ﾟΘﾟ]; (ﾟｰﾟ)+=(ﾟΘﾟ); (ﾟДﾟ)[ﾟεﾟ]='\\'; (ﾟДﾟ).ﾟΘﾟﾉ=(ﾟДﾟ+ ﾟｰﾟ)[o^_^o -(ﾟΘﾟ)];(oﾟｰﾟo)=(ﾟωﾟﾉ +'_')[c^_^o];(ﾟДﾟ) [ﾟoﾟ]='\"';(ﾟДﾟ)
...
</body>
</script> 
```

구글링을 통해 ___aaencode___ 인코딩으로 난독화되어 있다는 것을 알아냈다.

여기서 디코딩 할 수 있다. https://cat-in-136.github.io/2010/12/aadecode-decode-encoded-as-aaencode.html

디코딩한 소스코드는 다음과 같다.

```javascript
<script>
    var enco = '';
    var enco2 = 126;
    var enco3 = 33;
    var ck = document.URL.substr(document.URL.indexOf('='));
    for (i = 1; i < 122; i ++) {
        enco = enco + String.fromCharCode(i, 0);
    }
    function enco_(x) {
        return enco.charCodeAt(x);
    }
    if (ck == "=" +
    String.fromCharCode(enco_(240)) + 
    String.fromCharCode(enco_(220)) +
    String.fromCharCode(enco_(232)) + 
    String.fromCharCode(enco_(192)) + 
    String.fromCharCode(enco_(226)) + 
    String.fromCharCode(enco_(200)) + 
    String.fromCharCode(enco_(204)) + 
    String.fromCharCode(enco_(222 - 2)) + 
    String.fromCharCode(enco_(198)) + 
    "~~~~~~" + 
    String.fromCharCode(enco2) + 
    String.fromCharCode(enco3)) {
        location.href = "./" + ck.replace("=", "") + ".php";
    }
</script>
```

fromCharCode(x, y)함수는 객체의 ASCII코드를 인수로 하여 xy 문자열을 구성해주는 함수이다.

소스코드에서는 이를 이용하여 공백을 추가한다.

`$ck`에는 짝수 인덱스마다 NULL값이 추가되어 `{ 1,NULL,2,NULL,3,NULL, ...,122, NULL }` 와 같이 저장되므로 원하는 index에 저장된 ASCII코드는 `(index + 2) / 2` 이다.

따라서 위 조건문을 만족시키려면 URL에서 =뒤에 오는 값이 youaregod~~~~~~~!이 되어야 한다.

![youaregod](./pic/12-youaregod.PNG)

다음과 같이 URL에 파라미터를 붙여주면 성공.

![pwned](./pwned/old-12.PNG)

<br>

# old-13

___블라인드 SQL 인젝션___


<br>

# old-14 

___javascript___

![14-field](./pic/14-field.PNG)

다음과 같은 필드만 덩그러니 있다.

```javascript
...
<form name=pw><input type=text name=input_pwd><input type=button value="check" onclick=ck()></form>
<script>
function ck(){
  var ul=document.URL;
  ul=ul.indexOf(".kr");
  ul=ul*30;
  if(ul==pw.input_pwd.value) { location.href="?"+ul*pw.input_pwd.value; }
  else { alert("Wrong"); }
}
</script>
...
```

소스코드를 확인해보니 필드에 입력되는 값이 ul과 똑같아야 Wrong이 나오지 않을 것 같다. 
URL에서 ".kr"의 인덱스는 18이고, 여기에 30을 곱하면 540이므로, 이를 필드에 넣어주면 성공.

![pwned](./pwned/old-14.PNG)

<br>

# old-15

___javascript___

접속하면 다음과 같은 alert와 함께 webhacking.kr Index page로 이동한다.

![access_denied](./pic/access_denied.PNG)

사용하는 브라우저 설정에서 자바스크립트를 차단하면 정상적으로 접속할 수 있다.

![javascript](./pic/javascript.PNG)

```javascript
...
<script>
  alert("Access_Denied");
  location.href='/';
  document.write("<a href=?getFlag>[Get Flag]</a>");
</script>
...
```

소스코드를 확인해 보니 javascript가 /?getFlag로 접속하는 a태그를 만들어 주는 간단한 코드였다.

직접 해당 경로로 접속하면 성공.

![pwned](./pwned/old-15.PNG)

<br>

# old 16

______
