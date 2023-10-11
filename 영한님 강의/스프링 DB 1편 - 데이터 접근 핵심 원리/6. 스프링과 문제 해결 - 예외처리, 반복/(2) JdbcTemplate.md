
`JdbcTemplate` 을 템플릿 콜백 패턴을 통해 지금까지 고민했던 트랜잭션 추상화, 커넥션 동기화 및 관리, 예외 발생시 스프링 예외 변환기까지 자동으로 등록하고 실행해준다.

이제 `try ~ catch ~ fianlly` 구문, 커넥션 관리 등 모든 로직이 템플릿 콜백 패턴을 통해 처리되었다.

```
public class MemberRepository implements MemberRepository {  
  
    private final JdbcTemplate template;  
  
    public MemberRepository(DataSource dataSource) {  
        this.template = new JdbcTemplate(dataSource);  
    }  
  
    @Override  
    public Member save(Member member) {  
        String sql = "insert into member(member_id, money) values(?, ?)";  
        template.update(sql, member.getMemberId(), member.getMoney());  
        return member;  
    }  
  
    @Override  
    public Member findById(String memberId) {  
        String sql = "select * from member where member_id = ?";  
        //한건 조회  
        return template.queryForObject(sql, memberRowMapper(), memberId);  
    }
```
