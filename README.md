# 🌙 중급 프로젝트 Moonshot: 강시연 개발 리포트

## ⚙️ 개발 환경 및 협업 도구

| 구분       | 내용                                  |
|------------|-------------------------------------|
| Backend    | Node.js (Express) + TypeScript       |
| Database   | PostgreSQL                          |
| API 문서화 | Swagger                            |
| 협업 도구   | GitHub, Notion, Discord             |
| 일정 관리   | GitHub Issues + Notion 타임라인      |
| 브랜치 전략 | Git-Flow + dev                      |
| 네이밍 컨벤션 | GitHub 위키 네이밍 컨벤션 참고       |
| 커밋 컨벤션 | GitHub 위키 커밋 컨벤션 참고         |
| PR 규칙    | GitHub 위키 PR 컨벤션 참고           |
| 데일리 스크럼 | 매일 오전 9시                      |

## 📋 프로젝트 개요

- 일정 관리와 협업을 지원하는 서비스 Moonshot의 백엔드 API 개발
- 사용자 인증, 프로젝트 및 멤버 관리, 권한 검증, 할 일 관리 기능 구현
- 데이터 무결성과 권한 관리의 안정성 확보를 최우선 목표로 설정


## 🛠 담당한 작업

- 사용자 정보 조회 및 수정 API 개발  
- 프로젝트 생성, 조회, 수정, 삭제 API 개발  
- 프로젝트 생성 시 프로젝트 멤버 자동 등록 및 최대 생성 개수 제한 구현  
- 프로젝트 삭제 시 멤버 이메일 수집 후 알림 메일 발송 기능 구현  
- 권한 검증을 위한 미들웨어(authorization) 구현  
- Prisma ORM을 활용한 데이터베이스 쿼리 및 트랜잭션 처리 구현  
- 에러 핸들링 미들웨어와 통합하여 일관된 예외 처리 구조 적용  


## 🚀 기술적 성과

- **TypeScript & Express** 기반 API 서버 구현  
- **Prisma ORM**을 통한 효율적인 DB 접근 및 트랜잭션 처리 (프로젝트 생성과 멤버 등록 동시 처리)  
- **JWT 토큰 기반 인증** 후 사용자 정보(req.user) 활용한 권한 검증 미들웨어 작성  
- 프로젝트별 멤버 수 및 상태별 할 일 수 조회를 위한 집계 쿼리 최적화  
- 프로젝트 삭제 시 참여 멤버 이메일 수집 후 비동기 이메일 알림 발송 기능 구현  
- 사용자 비밀번호 변경 시 bcrypt 해싱 및 현재 비밀번호 확인을 통한 보안 강화  
- Google OAuth 사용자의 비밀번호 변경 제한 로직 구현  
- API 레이어별 (Controller-Service-Repository) 책임 분리로 코드 가독성 및 유지보수성 향상  


## ⚠️ 문제점 및 해결 과정

| 문제                         | 해결 방안                                               | 결과 및 배운 점                             |
| -------------------------------- | ------------------------------------------------------ | ------------------------------------------ |
| 프로젝트 최대 생성 개수 제한 미적용 | 서비스 레이어에서 생성 개수 조회 후 5개 초과 시 에러 반환 | 불필요한 자원 낭비 방지 및 UX 개선         |
| 프로젝트 삭제 시 메일 발송 실패    | try-catch로 메일 발송 감싸 실패 시 로그 처리만 수행       | 서비스 안정성 강화, 메일 실패가 전체 프로세스 방해 안함 |
| 권한 체크 누락으로 인한 보안 취약 | 서비스 레이어와 미들웨어에서 소유자 및 멤버 권한 철저 검증 | 보안 강화 및 명확한 에러 반환               |
| 비밀번호 변경 시 현재 비밀번호 미검증 | 변경 시 현재 비밀번호 입력 필수 및 구글 로그인 예외 처리    | 보안 강화 및 로그인 유형별 로직 분리        |

## 💻 핵심 코드 스니펫
### 프로젝트 최대 생성 개수 제한 (project-service.ts)
```js
const projectCount = await projectRepository.countProjectsByCreator(params.creatorId);

if (projectCount >= 5) {
  return {
    error: true,
    status: statusCode.badRequest,
    message: errorMsg.maxProjectLimit,
  };
}
```
### 프로젝트 삭제 시 멤버 이메일 수집 후 메일 발송 (project-service.ts)
```js
const membersEmails = await projectRepository.getProjectMembersEmails(projectId);

await projectRepository.deleteProject(projectId);

try {
  await Promise.all(
    membersEmails.map(email =>
      sendMail({
        to: email,
        subject: `[MOONSHOT] "${project.data.name}" 프로젝트가 삭제되었습니다.`,
        text: `안녕하세요.\n참여 중이던 프로젝트 "${project.data.name}"가 삭제되었습니다.`,
      })
    )
  );
} catch (error) {
  console.error('메일 발송 실패:', error);
}

```
### 권한 검증 미들웨어 (authorization.ts)
```js
const userId = req.user?.id;
const projectId = Number(req.params.projectId);

if (!userId) {
  return handleError(next, null, errorMsg.loginRequired, statusCode.unauthorized);
}

if (isNaN(projectId)) {
  return handleError(next, null, errorMsg.invalidProjectId, statusCode.badRequest);
}

const isMember = await prisma.project_members.findFirst({
  where: { projectId, userId },
});

if (!isMember) {
  return handleError(next, null, errorMsg.accessDenied, statusCode.forbidden);
}

next();
```

## 🤝 협업 및 피드백

- PR 과정에서 팀원들과 코드 리뷰를 통해 서비스 계층에서의 권한 검증 강화 및 에러 메시지 통일에 대해 피드백을 받았습니다.  
- 프론트엔드와 API 요청/응답 명세를 정기적으로 공유하며, 쿼리 파라미터의 유효성 검사와 예외처리 기준에 대해 조율하였습니다.  
- GitHub PR 코멘트를 적극 수용하여 코드 품질 개선 및 주석 보완에 신경 썼습니다.  
- ESLint, Prettier, Husky 등의 도구 도입으로 코드 스타일을 통일하며 협업 효율성을 높였습니다.


## 🧹 코드 품질 및 최적화

- 컨트롤러는 요청 검증 및 서비스 호출에 집중, 비즈니스 로직은 서비스 레이어로 분리하여 관심사 분리 적용  
- Prisma 트랜잭션 활용으로 데이터 일관성과 원자성 확보  
- 에러 처리 미들웨어와 별도 상태 코드·메시지 관리 모듈 도입으로 유지보수 용이성 확보  
- 페이징, 정렬, 필터링 기능을 서비스 단에서 처리해 불필요한 DB 부하 최소화  
- **가독성 및 유지보수성을 높이기 위한 노력**:  
  - 명확하고 일관된 변수 및 함수명 사용  
  - 중복 코드 최소화 위한 공통 함수 및 DTO 활용  
  - 충분한 주석과 문서화로 동료가 쉽게 이해할 수 있도록 구현  


## 🔮 향후 개선 사항 및 제안

- 프로젝트 삭제 시 메일 발송 실패에 대비한 재시도 로직 또는 큐(Queue) 시스템 도입 검토  
- 권한 검증 미들웨어를 더욱 세분화하여 세부 권한(읽기, 쓰기, 삭제 등)별로 분리하면 보안 강화 가능  
- 구글 소셜 로그인 사용자의 회원정보 수정 기능 확장(프로필 이미지 등)  
- 프로젝트 및 할 일에 대한 캐시 처리 도입으로 조회 성능 향상  
- API 요청 파라미터 유효성 검사에 `Joi` 등 스키마 검증 라이브러리 적용 고려  
- 테스트 코드 보완 및 통합 테스트 환경 구축으로 안정성 강화  

