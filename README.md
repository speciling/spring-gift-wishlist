# spring-gift-wishlist
## 기능 요구사항
### STEP 1
1. 사용자 입력의 유효성을 검사해야 한다
   - 상품명
      - 공백을 포함하여 최소 1자 이상, 최대 15자 이하의 문자를 입력해야 한다.
      - 특수 문자
         - 가능: ( ), [ ], +, -, &, /, _
         - 그 외 특수 문자 사용 불가
   - 상품 가격
      - 0이상의 정수를 입력해야 한다.
      - null 이 되어선 안된다
2. 유효성 검사 오류에 대한 예외처리가 이루어져야 한다
   - 상태코드는 400으로 한다
   - 유효하지 않은 parameter의 이름과 그 이유를 반환해야 한다
      ```- json
      HTTP/1.1 400 Bad Request
      Content-Type: application/json
      {
         "price": "0 이상이어야 합니다",
         "name": "영문, 한글, 숫자와 공백, 특수문자 ()[]+-&/_ 1자 이상 15자 미만으로 입력해야 합니다."
      }
      ```
3. 상품명에 "카카오"가 포함된 문구는 담당 MD와 협의한 경우에만 사용할 수 있다. 
   - "카카오"가 포함된 상품은 등록할 권한이 없으므로 403 forbidden 에러를 반환한다
      ```- json
      HTTP/1.1 403 Forbidden
      Content-Type: application/json
      {
         "message": "'카카오'가 포함된 문구는 담당 MD와 협의한 경우에만 사용할 수 있습니다."
      }
      ```

### STEP 2
1. 사용자는 id, email, password, 이름, 권한(일반 사용자, 판매자, 관리자) 정보를 갖는다.
2. 사용자로부터 email, password, 이름, 권한(일반 사용자 or 판매자)을 입력받아 회원가입을 할 수 있다.
    - 가입한 사용자의 정보 중 password는 안전한 해시 알고리즘을 사용하여 단방향 암호화하여 저장한다
    - 관리자 계정은 회원가입 api로는 가입이 불가능하고, db에 직접 sql 쿼리를 입력하여야만 등록할 수 있다.
      - 관리자 권한을 갖도록 회원가입을 시도하는 경우 400 에러를 반환한다
3. 사용자로부터 email, password를 입력받아 로그인을 할 수 있다.
    - email과 password가 모두 일치하는 계정이 존재하면, 그 계정의 정보로 JWT 토큰을 만들어 사용자에게 반환한다
    - 일치하는 계정이 존재하지 않으면, 403 에러를 반환한다
4. 상품은 id, 상품명, 가격, 이미지 url, 등록한 사용자의 id  정보를 갖는다.
5. 상품 등록은 판매자, 관리자 권한을 가진 사용자만 사용할 수 있다.
    - 인증되지 않은 사용자가 상품 정보 수정을 시도하면 401 에러를 반환한다.
    - 올바른 권한을 갖지 않은 사용자가 상품 정보 수정을 시도하면 403 에러를 반환한다.
6. 상품 조회는 인증 유무에 관계없이 모두가 사용할 수 있다.
7. 상품 수정은 판매자, 관리자 권한을 가진 사용자만 사용할 수 있다.
    - 판매자 권한을 가진 사용자는 자신이 등록한 상품만 수정할 수 있다.
    - 관리자 권한을 가진 사용자는 모든 상품을 수정할 수 있다.
    - 인증되지 않은 사용자가 상품 정보 수정을 시도하면 401 에러를 반환한다.
    - 올바른 권한을 갖지 않은 사용자가 상품 정보 수정을 시도하면 403 에러를 반환한다.
8. 상품 삭제는 판매자, 관리자 권한을 가진 사용자만 사용할 수 있다.
    - 판매자 권한을 가진 사용자는 자신이 등록한 상품만 삭제할 수 있다.
    - 관리자 권한을 가진 사용자는 모든 상품을 삭제할 수 있다.
    - 인증되지 않은 사용자가 상품 정보 삭제를 시도하면 401 에러를 반환한다.
    - 올바른 권한을 갖지 않은 사용자가 상품 정보 삭제를 시도하면 403 에러를 반환한다.

### STEP 3
1. 위시 리스트에 상품을 추가할 수 있다.
    - 위시 리스트에 사용자 id, 상품 id, 상품 수량을 저장한다.
    - 해당 사용자 id, 상품 id를 가진 항목이 있다면 400 에러를 반환한다.
    - 인증된 사용자가 아닌 경우 401 에러를 반환한다
    - 존재하지 않는 상품 id인 경우 400 에러를 반환한다
2. 위시 리스트에 등록된 상품 목록을 조회할 수 있다.
    - 위시 리스트 테이블에서 현재 로그인한 사용자의 id를 가진 항목들의 리스트를 조회할 수 있다.
    - 인증된 사용자가 아닌 경우 401 에러를 반환한다
3. 위시 리스트에 담긴 상품 수량을 변경할 수 있다.
    - 위시 리스트 테이블에서 현재 로그인한 사용자의 id와 변경할 상품 id를 가진 항목을 찾아 수량을 변경할 수 있다.
    - 해당 항목이 존재하지 않는 경우 400 에러를 반환한다.
    - 인증된 사용자가 아닌 경우 401 에러를 반환한다.
4. 위시 리스트에 담긴 상품을 삭제할 수 있다.
    - 위시 리스트 테이블에서 현재 로그인한 사용자의 id와 삭제할 상품 id를 가진 항목을 삭제할 수 있다.
    - 해당 항목이 존재하지 않는 경우 400 에러를 반환한다.
    - 인증된 사용자가 아닌 경우 401 에러를 반환한다.

