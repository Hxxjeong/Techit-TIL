## [05/03] SQL JOIN, 서브쿼리



### JOIN

- Cartesian Join (Cross Join)

  한 테이블에 있는 모든 행이 다른 테이블에 있는 모든 행과 join

  ```sql
  select * from table1, table2;
  ```

  튜플의 개수: table1의 튜플 개수 * table2의 튜플 개수

- Equi Join

  컬럼에 있는 값이 정확하게 일치하는 경우 = 연산자 사용

  ```sql
  select employee_id, first_name, email, department_name
  from employees e, departments d
  where e.department_id = d.department_id;
  ```

  where절 or on절 사용

  join을 위한 최소한의 조건 = 테이블 개수 - 1

- Theta Join (Non-Equi Join)

  임의의 조건을 Join 조건으로 사용 가능

  ```sql
  select ename, sal, GRADE
  from emp e, salgrade s
  where sal between LOSAL and HISAL;
  ```

- Natural Join

  두 개의 테이블에 똑같은 컬럼 이름이 존재해야 함

- Inner

  using: 조인하고자 하는 두 컬럼의 이름이 같을 때 사용

  on: 조인 조건 기술

- Outer

  조건을 만족하지 않는 튜플의 경우 null로 출력

  Oracle: null이 올 수 있는 조건에 + 붙여서 조회 가능

  MySQL에서 full outer join이 제공이 되지 않으므로 조건을 주어 테이블을 출력

  ```sql
  select ename, dname
  from emp left join dept using(deptno)
  union
  select ename, dname
  from emp right join dept using(deptno);
  ```

- Self Join



### SubQuery

하나의 SQL문 안에 다른 SQL문이 포함된 형태

- Single-Row Subquery

  단일 비교 연산자 사용

- Multi-Row Query

  - in

    in의 조건이 여러 개라면 조건은 and로 묶임

  - any

    `>any`, `<any`, `=any (in과 동일)`

  - all

- Correlated Query

  Outer Query와 Inner Query가 서로 연관되어 있음

  ```sql
  -- 자신 부서의 평균 급여보다 급여가 많은 사원
  select * from emp o
  where sal > (select avg(sal)
               from emp i
               where i.deptno = o.deptno);
  ```



### Set Operator

- union, union all, intersect, except

  MySQL에서는 intersect와 except는 지원되지 않음

------

https://carami.tistory.com/18