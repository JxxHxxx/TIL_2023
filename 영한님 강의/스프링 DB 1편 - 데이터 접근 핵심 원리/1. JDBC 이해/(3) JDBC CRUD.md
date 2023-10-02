

### INSERT

쿼리를 한 번 작성하기 위해 매우 복잡한 절차를 거쳐야 함을 느끼면 된다.

- `쿼리 작성` > `쿼리 파라미터 바인딩` > `ResultSet` 종료 > `PreparedStatement` 종료 >  `Connection` 종료

이 과정을 순서대로 진행해야 한다. 그래서 `finally` 문이 사용되기도 한다. 또한 `checked` 예외가 발생하기 때문에 `try/catch` 문이 여러 번 작성되게 된다.


```
public class MemberRepositoryV0 {  
  
    public Member save(Member member) throws SQLException {  
        String sql = "insert into member(member_id, money) values (?, ?)";  
  
        Connection con = null;  
        PreparedStatement pstmt = null;  
  
        try {  
            con = getConnection();  
            pstmt = con.prepareStatement(sql);  
            pstmt.setString(1, member.getMemberId());
            pstmt.setInt(2, member.getMoney());  
            pstmt.executeUpdate(); // 준비된 쿼리 실행
            return member;  
        } catch (SQLException e) {  
            log.error("db error", e);  
            throw e;  
        } finally {  
            close(con, pstmt, null); // 연결 닫는게 보장되도록 finally 구문에 사용되어야 함  
        }  
    }  
}
```


*분리한 close 메서드*

```
    private void close(Connection con, Statement stmt, ResultSet rs) {  
        if (rs != null) {  
            try {  
                rs.close();  
            } catch (SQLException e) {  
                log.error("Error" , e);  
            }  
        }  
  
        if (stmt != null) {  
            try {  
                stmt.close(); // Exception 이 터지면 아래 con.close() 호출이 안됨  
            } catch (SQLException e) {  
                log.error("Error" , e);  
            }  
        }  
        if (con != null) {  
            try {  
                con.close(); // 히카리 풀 같은 곳에 있는 커넥션과의 연결을 종료한다고 보면 된다.  
            } catch (SQLException e) {  
                log.error("Error" , e);  
            }  
        }  
    }  
```


### SELECT

SELECT 도 INSERT 와 크게 다르지 않다. 대신 SELECT은 결과를 가지기 때문에 `ResultSet` 이 활용된다.

```
public Member findById(String memberId) throws SQLException {  
    String sql = "select * from member where member_id = ?";  
  
    Connection con = null;  
    PreparedStatement pstmt = null;  
    ResultSet rs = null;  
  
    try {  
        con = getConnection();  
        pstmt = con.prepareStatement(sql);  
        pstmt.setString(1, memberId);  
  
        rs = pstmt.executeQuery(); // select -> executeQuery  
        if (rs.next()) { // cursor 라는게 존재하는데 next를 한 번은 호출해야 실제로 select 된 데이터들이 들어가있는듯  
            Member member = new Member();  
            member.setMemberId(rs.getString("member_id"));  
            member.setMoney(rs.getInt("money"));  
            return member;  
        }  
        else {  
            throw new NoSuchElementException("member not found memberId=" + memberId);  
        }  
  
    } catch (SQLException e) {  
        log.error("db error", e);  
        throw e;  
    } finally {  
        close(con, pstmt, rs);  
    }  
}
```


*ResultSet*

`ResultSet` 은 테이블 구조와 같은 자료 구조로 `SELECT` 에 대한 결과를 객체로 담고 있다. 최초에 `next()` 메서드를 호출하면 `SELECT` 쿼리 결과에 대한 첫 번째 레코드에 접근할 수 있다.

위 코드에서는 `pk` 로 조회하기 때문에 `ResultSet` 에 단 하나의 레코드만 존재하게 된다.



### UPDATE/DELETE

사실 INSERT 와 별 차이가 없으나 `excuteUpdate()` 메서드가 반환하는 값에 대해서 가볍게 설명하도록 하겠다.

```
public void update(String memberId, int money) throws SQLException {  
    String sql = "update member set money=? where member_id=?";  
  
    Connection con = null;  
    PreparedStatement pstmt = null;  
  
    try {  
        con = getConnection();  
        pstmt = con.prepareStatement(sql);  
        pstmt.setInt(1, money);  
        pstmt.setString(2, memberId);  
        int resultSize = pstmt.executeUpdate();  
        log.info("resultSize = {}", resultSize);  
  
    } catch (SQLException e) {  
        log.error("db error", e);  
        throw e;  
    } finally {  
        close(con, pstmt, null);   
    }  
}  
  
public void delete(String memberId) throws SQLException {  
    String sql = "delete from member where member_id=?";  
  
    Connection con = null;  
    PreparedStatement pstmt = null;  
  
    try {  
        con = getConnection();  
        pstmt = con.prepareStatement(sql);  
        pstmt.setString(1, memberId);  
        int resultSize = pstmt.executeUpdate();  
        log.info("resultSize = {}", resultSize);  
  
    } catch (SQLException e) {  
        log.error("db error", e);  
        throw e;  
    } finally {  
        close(con, pstmt, null);   
    }  
}
```


*executeUpdate()* 

해당 메서드는 INSERT, DELETE, UPDATE  같이 `ResultSet` 을 사용하지 않는 쿼리를 실행할 때 사용하고 `int` 타입을 반환한다. 반환 값은 INSERT, DELETE, UPDATE  작업에 영향을 받은 레코드 수를 의미한다.