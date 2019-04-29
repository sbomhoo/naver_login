# naver_login
naver 로그인 api (네아로)
- war / zip 둘다 같은 파일 

## 네이버 Developer 사이트에 API 설정한 내용
**서비스 URL**
- http://localhost:8080/login_project/login

**Callback URL**
- http://localhost:8080/tst/callback


## 모두의 주방_ UserController.java  
- 나만의 네아로 로그인 , 로그아웃 구현 방법  (이 파일에 없는 부분 , 모두의 주방 프로젝트에만 추가하는 내용)
- 컨트롤러 수정한 부분
 1. 첫 로그인form 요청 받았을 때 loginForm로직에 naverAuthUrl 만들어 놓기
 2. 기존의 모두의 주방 회원으로 로그인하는 로직(loginAction)은 그대로 둔다. 
 3. 네이버 로그인 성공 시 로직(callback)을 새로 작성, 여기서 String형식의 json데이터인 'apiResult'를 파싱하여 세션에 저장

```java 
	//로그인
	@RequestMapping(value="/login.userdo",method=RequestMethod.GET)
	public String loginForm(Model model, HttpSession session) throws IOException {   //사용자 입력값을 request가 아닌 command가 받게 하기 위해 boardVO를 매개변수로 선언
		System.out.println("로그인 폼");
		
		/* 네이버아이디로 인증 URL을 생성하기 위하여 naverLoginBO클래스의 getAuthorizationUrl메소드 호출 */
		String naverAuthUrl = naverLoginBO.getAuthorizationUrl(session);
		System.out.println("네이버:" + naverAuthUrl);
		
		//네이버 
		model.addAttribute("url", naverAuthUrl);

		/* 생성한 인증 URL을 View로 전달 */
		return "loginForm";  //글 입력 완성 후 게시글리스트로 가기 위
	}
	
	//네이버 로그인 성공시 callback호출 메소드
	@RequestMapping(value = "/callback.userdo", method = { RequestMethod.GET, RequestMethod.POST })
	public String callback(Model model, @RequestParam String code, @RequestParam String state, HttpSession session)
			throws IOException, ParseException {
		System.out.println("여기는 callback");
		OAuth2AccessToken oauthToken;
        oauthToken = naverLoginBO.getAccessToken(session, code, state);

        //1. 로그인 사용자 정보를 읽어온다.
		apiResult = naverLoginBO.getUserProfile(oauthToken);  //String형식의 json데이터
		
		/** apiResult json 구조
		{"resultcode":"00",
		 "message":"success",
		 "response":{"id":"33666449","nickname":"shinn****","age":"20-29","gender":"M","email":"shinn0608@naver.com","name":"\uc2e0\ubc94\ud638"}}
		**/
		
		//2. String형식인 apiResult를 json형태로 바꿈
		JSONParser parser = new JSONParser();
		Object obj = parser.parse(apiResult);
		JSONObject jsonObj = (JSONObject) obj;
		
		//3. 데이터 파싱 
		//Top레벨 단계 _response 파싱
		JSONObject response_obj = (JSONObject)jsonObj.get("response");
		//response의 nickname값 파싱
		String nickname = (String)response_obj.get("nickname");

		//System.out.println(nickname);
		
		//4.파싱 닉네임 세션으로 저장
		session.setAttribute("sessionId",nickname); //세션 생성
		
		model.addAttribute("result", apiResult);
	     
		/* 네이버 로그인 성공 페이지 View 호출 */
		return "redirect:index.jsp";
		}

	//모두의 주방 회원 로그인 성공시
	@RequestMapping(value="/login.userdo",method=RequestMethod.POST)
	public String loginAction(UserVO vo,HttpSession session) throws IOException {   //사용자 입력값을 request가 아닌 command가 받게 하기 위해 boardVO를 매개변수로 선언
		System.out.println("로그인 처리");
		
		vo.getPwd(); //폼에서 보낸 비번 
		userService.loginAction(vo);//디비에서 가져온 비번
		
		//로그인 성공
		if(vo.getPwd().equals(userService.loginAction(vo))) {	
			session.setAttribute("sessionId", vo.getId()); //세션 생성
			return "redirect:index.jsp";
		}
		session.setAttribute("message", "로그인에 실패하였습니다. ID와 PW를 확인해주세요!");
		return "loginForm";  //글 입력 완성 후 게시글리스트로 가기 위해
	}
	
	@RequestMapping(value="/logout.userdo",method=RequestMethod.GET)
	public String logoutAction(HttpSession session) throws IOException {   //사용자 입력값을 request가 아닌 command가 받게 하기 위해 boardVO를 매개변수로 선언
		System.out.println("로그아웃 처리");
		
		session.invalidate();	
		return "redirect:index.jsp";  //글 입력 완성 후 게시글리스트로 가기 위해
	}
```
