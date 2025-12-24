- `PathParam` : 리소스 식별에 필요한 값을 넣는다. 필수적이다.
- `QueryParam` : 필터링/정렬/페이징 같이 선택적이고 다중화될 수 있는 값을 넣는다.
- `RequestBody` : 복잡한 값 구조 / 민감한 값을 넣는다.
- 예를 들어
	- 34번 게시물의 댓글 중, 답글이 달린 댓글만 보여주세요.
	  : `/posts/34/comments?status=responsed`
	- 34번 게시물의 댓글 중, 답글이 달린 댓글에만 '하이' 라는 답글을 달아주세요.
	  : `/posts/34/comments/response?commentStatus=responsed {"content" : "하이"}`
- 근데 그러면 ID/PW 로그인을 할 때 ID를 PathParam으로 전달해주는게 낫지 않나? 왜 보통 IDPW 모두 바디로 전달하지... -> ID도 민감정보로 인정하는듯!