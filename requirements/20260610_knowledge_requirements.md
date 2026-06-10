나는 knowledge 기능을 구체화 하려고 한다.

<developer>
1. developer는 knowledge기능을 사용여부를 결정해야한다. 근데 어디서 할지 논의가 필요하다.
2. developer는 agent를 만들떄 knowledge기능을 사용한다면 airflow로 데이터를 처리하는 dag을 생성해야하고 end-user는 해당 dag을 통해서 자신의 문서를 처리해야한다.
3. developer는 end-user가 업로드한 문서를 처리할지 말지 승인/반려 권한을 가지고 있고, end-user는 developer의 승인 받아야만 자신의 문서를 developer가 만든 dag을 통해서 처리하여 vectorDB에 넣을 수 있다.


<end-User>
1. 만약 developer가 knowledge기능을 사용한다면, end-user는 구독한 agent에 자신의 문서를 업로드하고 지식화 할 수 있다.


UI적으로 다음의 기능들이 어떤 화면에서 이루어지게 할지 정의가 필요하다. docs의 requirements.html을 기준으로 case별로 정리가 필요하다.

1. 먼저, developer가 knowledge기능을 사용할지 말지 여부를 선택하는 화면을 어디 화면에서 할지 결정해야한다. case 별로 정리해줘

2. developer는 agent만을 만들기도 하지만 데이터처리가 필요한 경우 airflow의 dag을 만들어 제공한다. 해당 agent를 만들때 S3자원을 받아서 사용자가 업로드한 문서를 일시보관하고 승인/반려에 따라서 해당 S3에서 문서를 우리가 만드는 플랫폼에서 관리할지 여부를 선택해야한다.
아니면 내가 만드는 플랫폼의 S3에 end-user가 업로드한 문서를 보관하고 developer가 만든 airflow dag에서 가져다 쓸 수 있게 해야한다. 이렇게 만들기 위해서는 developer가 dag을 만들때 해당 S3경로, end-user가 올린 문서의 경로를 플랫폼 백엔드 api 호출을 통해 전달받아서 dag에서 해당 경로를 사용하여 문서를 처리할 수 있게 해야한다.
이렇게 하려면 우리 쪽에서는 developer의 승인/반려에 따라서 end-user가 올린문서를 dag에 전송, 삭제를 컨트롤해야한다.
또한 developer가 dag을 만들때 사용할 template code를 우리가 제공해서 플랫폼 백엔드와 airflow dag과의 인터페이스 규격을 정해야한다.


이러한 case를 case별로 정리하여 고객이 선택하게 해야한다.


developer는 마켓에 agent를 신청할때 다음을 적는다.
step 1.
과제번호, 부문, agent agent 이름(중복확인), 카테고리, agent 설명, 팀이름(접속 할떄 이미 developer의 정보가 있어서 자동 표시), 신청자(팀이름과 마찬가지로) 담당,
협업 developer, 시스템계정, 보안성 검토 내영 신청 ID, 승인여부 

step2. Skills/Tags설정

step3. Step 3. 확인 절차

step4. 신청 완료(등록 대기) -> 추후 관리자가 승인여부에 따라서 승인 페이지로 전환

각 step이 ui 상단에 단계별로 표시된다. 여기에 knowledge 기능 선택 토글이 존재 