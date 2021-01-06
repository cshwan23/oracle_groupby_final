# oracle_groupby_final
oracle groupby_final


    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm
    -- Q.116 [직원명], [출생년도](년-월-일), [나이]를 츨력하면?
    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm
    select
    EMP_NAME
    ,case when substr(JUMIN_NUM,7,1) in('1','2') then '19'
              else '20' end || substr(JUMIN_NUM,1,2)||'년'||
                                substr(JUMIN_NUM,3,2)||'월'||
                                substr(JUMIN_NUM,5,2)||'일'
        "출생년도"
    ,extract(year from sysdate) -
          to_number(
              case when substr(JUMIN_NUM,7,1) in('1','2') then '19'
              else '20' end || substr(JUMIN_NUM,1,2)
              )+1 ||'살' "나이"

    from EMPLOYEE;
    -- [직원명], [생일](년-월-일), [나이]를 츨력하면?
    select
    EMP_NAME
    ,to_char(
        to_date(
           case when substr(JUMIN_NUM,7,1) in('1','2') then '19'
              else '20' end || substr(JUMIN_NUM,1,6)
        ,'YYYYMMDD')
        ,'YYYY-MM-DD')
        "생일"
    ,to_number(to_char(sysdate,'YYYY')) -
          to_number(
              case when substr(JUMIN_NUM,7,1) in('1','2') then '19'
              else '20' end || substr(JUMIN_NUM,1,2)
              )+1 ||'살' "나이"

    from EMPLOYEE;

    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm
    -- Q.117 부서별로 [부서번호], [평균근무년수]를 츨력하면? (근년수 소수2째자리에서 반올림)
    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm
    select
    DEP_NO "부서명"
    ,to_char(avg((sysdate-HIRE_DATE)/365),'99.9')||'년' "평균근무년수"
    from EMPLOYEE
    group by DEP_NO;
    ----------------------------------
    --round(특정숫자 , n ) => 소수점 n번째 자리까지 보여줘라(반올림된다)/즉째 소수점 n+1번째에서 반올림한다.
    ----------------------------------
    select
    DEP_NO "부서명"
    ,round(avg((sysdate-HIRE_DATE)/365),1)||'년' "평균근무년수"
    from EMPLOYEE
    group by DEP_NO;


    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm
    -- Q.118 입사분기별로 [입사분기], [인원수]를 츨력하면?
    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm
    select
    to_char(HIRE_DATE,'Q')||'/4분기' "입사분기"
    ,count(*)||'명'                "인원수"
    from EMPLOYEE
    group by to_char(HIRE_DATE,'Q')
    order by 1 asc;
        -------------------------------
        --Q.118 새끼문제) 10번부서의 입사분기별로 [입사분기], [인원수]를 츨력하면?
        -------------------------------
        -- group by 이전에 where 을 쓸 수 있다.
        select
               to_char(HIRE_DATE, 'Q') || '/4분기' "입사분기"
             , count(*) || '명'                   "인원수"
        from EMPLOYEE
        where DEP_NO = 10
        group by to_char(HIRE_DATE, 'Q')
        order by 1 asc;
        -- group by로 그룹진 이후에 inline view/having 다 안 된다.
        -- group by 뒤에 group by 결과물로 행을 골라낼 땐 having 가능
        -- group by로 그룹짓기 전에 where 를 써서 먼저 10번 부서를 걸러야한다.


    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm
    -- Q.119 입사분기별로 [입사분기], [인원수]를 츨력하면? 단 1분기만 보여라.
    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm
    select
    to_char(HIRE_DATE,'Q')||'분기' "입사분기"
    ,count(*)||'명'                "인원수"
    from EMPLOYEE
    group by to_char(HIRE_DATE,'Q')
    having to_char(HIRE_DATE,'Q') = 1;
    -----------------------
    select *
    from
    (
        select to_char(HIRE_DATE, 'Q') ||'분기' "QUARTER"
             , count(*) || '명'                 "CNT"
        from EMPLOYEE
        group by to_char(HIRE_DATE, 'Q')
    )
    where QUARTER = 1 ||'분기';


    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm
    -- Q.120 입사연대별, 성별로 [입사연대], [성별], [연대별입사자수]를 츨력하면?
    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm
    select
    substr(to_char(HIRE_DATE,'YYYY'),1,3)||'0년대' "입사연대"
    , case when substr(JUMIN_NUM,7,1) in('1','3') then '남' else '여' end    "성별"
    ,count(*) ||'명' "연대별입사자수"

    from EMPLOYEE
    group by substr(to_char(HIRE_DATE,'YYYY'),1,3)
           ,case when substr(JUMIN_NUM,7,1) in('1','3') then '남' else '여' end
    order by 1 asc;


    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm
    -- Q.121 [직원명], [입사일](년-월-일/~/4분기~요일), [퇴직일](년~월~일)을 츨력하면?
    -- 조건 - 퇴직일은 입사 후 20년 5개월 10일 후
    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm
        select
            EMP_NAME                                                                "직원명"
            ,to_char(HIRE_DATE,'YYYY-MM-DD-Q')||'/4분기-'||to_char(HIRE_DATE,'DY')  "입사일"
            ,to_char(add_months(HIRE_DATE,5+(12*20))+10,'YYYY-MM-DD')               "퇴직일"
        from EMPLOYEE;

    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm
    -- Q.122 직원이 있는 부서별로 [부서번호], [부서위치], [직원수]를 출력하면?
    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm
        select
            DEP_NO                                               "부서번호"
            ,(select d.LOC from DEPT d where e.DEP_NO=d.DEP_NO ) "부서위치"
            ,count(*) || '명'                                     "직원수"
        from EMPLOYEE e
        group by DEP_NO
    order by 1 asc;
    --------------------------
        select
            d.DEP_NO            "부서번호"
            ,d.LOC              "부서위치"
            ,count(e.EMP_NAME)  "직원수"
        from EMPLOYEE e, DEPT d
        where e.DEP_NO = d.DEP_NO
        group by d.DEP_NO, d.LOC
        order by 1 asc;
    --------------
        select
            d.DEP_NO    "부서번호"
            ,d.LOC  "부서위치"
            ,(select count(*) from EMPLOYEE e where e.DEP_NO=d.DEP_NO) "직원수"
        from DEPT  d
        where(select count(*) from EMPLOYEE e where e.DEP_NO=d.DEP_NO)>0
        order by 1 asc;
    --------------
        select *
        from(
            select
                d.DEP_NO "DEP_NO"
                ,d.loc  "LOC"
                ,(select count(*) from EMPLOYEE e where e.DEP_NO=d.DEP_NO) "EMP_CNT"
            from DEPT d
        )
        where emp_cnt>0
    order by 1 asc;

    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm
    -- Q.123 월별로 [입사월], [인원수]를 출력하면? (입사월 오름차순 유지
    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm
    select
        extract(month from HIRE_DATE) ||'월'  "입사월"
        ,count(*)                     ||'명'  "인원수"
    from EMPLOYEE
    group by extract(month from HIRE_DATE) ||'월'
    order by 1 asc;
    --------------------------------------------
    select
        to_char(HIRE_DATE,'MM')||'월'         "입사월"
        ,count(*)                     ||'명'  "인원수"
    from EMPLOYEE
    group by to_char(HIRE_DATE,'MM') ||'월'
    order by 1 asc;
        --------------------------------------------
        --Q.123(새끼문제) 위 결과에서 2월, 9월은 없어서 빠진다. 없는 월도 포함시키고 인원수는 0으로 포함하려면?
        --------------------------------------------
         select
            m.MONTH||'월'    "입사월"
            ,count(e.EMP_NAME)||'명'  "입사인원수"
        from(
            select '01' "MONTH" from dual
            union select '02' from dual
            union select '03' from dual
            union select '04' from dual
            union select '05' from dual
            union select '06' from dual
            union select '07' from dual
            union select '08' from dual
            union select '09' from dual
            union select '10' from dual
            union select '11' from dual
            union select '12' from dual
        ) m, EMPLOYEE e
        where m.MONTH = to_char(e.HIRE_DATE(+),'MM')
        group by m.MONTH||'월'
        order by "입사월" asc;
        --------------------------------------------
        select
            m.MONTH ||'월'                                       "입사월"
            ,(select count(*)
              from EMPLOYEE e
              where m.MONTH = to_char(e.HIRE_DATE,'MM')) ||'명'  "입사인원수"
        from(
                select '01' "MONTH" from dual
                union select '02' from dual union select '03' from dual
                union select '04' from dual union select '05' from dual
                union select '06' from dual union select '07' from dual
                union select '08' from dual union select '09' from dual
                union select '10' from dual union select '11' from dual
                union select '12' from dual
            )m ;
        --------------------------------------------
        select
        m.MONTH ||'월'
        ,count(e.EMP_NAME) ||'명'
        from(
            select case when rownum<10 then '0' else '' end||ROWNUM "MONTH" from EMPLOYEE where rownum <=12
            )m , EMPLOYEE e
        where m.MONTH=to_char(e.HIRE_DATE(+),'MM')
        group by m.MONTH ||'월'
        order by 1 asc;
        --------------------------------------------
        select
        m.MONTH ||'월' "입사월"
        ,(select count(*) from EMPLOYEE e where m.MONTH=to_char(e.HIRE_DATE,'MM')) ||'명' "입사원수"
        from(
            select case when rownum<10 then '0' else '' end||ROWNUM "MONTH" from EMPLOYEE where rownum <=12
            )m
        order by 1 asc;

    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm
    -- Q.124 employee 테이블에서 직급순서대로 정렬하여
    --      직급별로 [직급], [직급평균연봉], [인원수] 를 검색하면?
    --      단, 높은 직급이 먼저 나와야함.
    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm
    select
        JIKUP                           "직급"
        ,round(avg(salary),0) ||'만원'   "직급평균연봉"
        ,count(*) ||'명'                 "인원수"
    from EMPLOYEE
    group by JIKUP
    order by case JIKUP when '사장' then 1 when '부장' then 2 when '과장' then 3 when '대리' then 4 else 5 end asc;

    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm
    -- Q.125 부서별로 [부서번호], [부서명], [직원수], [관리고객수] 를 검색하면?
    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm
    -----------------------------------------------
    --상관쿼리 사용
    -----------------------------------------------
    select
        d.DEP_NO as "부서번호"
        ,d.DEP_NAME as "부서명"
        ,(select count(*)||'명' from EMPLOYEE e where e.DEP_NO = d.DEP_NO) as "직원수"
        ,(select count(*)||'명' from EMPLOYEE e, CUSTOMER c where e.EMP_NO = c.EMP_NO and e.DEP_NO= d.DEP_NO) as "담당고객수"
    from dept d
    order by 1 asc;
    -----------------------------------------------
    -- group by 사용
    -----------------------------------------------
    select
        d.DEP_NO                          "부서번호"
        ,d.DEP_NAME                       "부서명"
        ,count(distinct e.EMP_NO) ||'명'  "직원수"
        ,count(c.EMP_NO)          ||'명'  "담당고객수"
    from EMPLOYEE e , DEPT d, CUSTOMER c
    where d.DEP_NO = e.DEP_NO(+) and c.EMP_NO(+) = e.EMP_NO
    group by d.DEP_NO ,d.DEP_NAME
    order by 1 asc;
    -----------------------------------------------
    -- ansi 방식
    -----------------------------------------------
    select
        d.DEP_NO                         "부서번호"
        ,d.DEP_NAME                      "부서명"
        ,count(distinct e.EMP_NO) ||'명' "직원수"
        ,count(c.EMP_NO) ||'명'          "담당고객수"
    from dept d left outer join EMPLOYEE e on e.DEP_NO = d.DEP_NO
                left outer join CUSTOMER c on c.EMP_NO = e.EMP_NO
    group by d.DEP_NO, d.DEP_NAME
    order by 1 asc;

    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm
    -- Q.126 오늘부터 10일까지 날짜 중에 토,일,월을 제외한 날의 개수를 구하면?
    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm
    -- 내 코드
    select
            case to_number(to_char(sysdate,'D')) when 1 then 0 when 2 then 0 when 7 then 0 else 1 end +
            case to_number(to_char(sysdate+1,'D')) when 1 then 0 when 2 then 0 when 7 then 0 else 1 end +
            case to_number(to_char(sysdate+2,'D')) when 1 then 0 when 2 then 0 when 7 then 0 else 1 end +
            case to_number(to_char(sysdate+3,'D')) when 1 then 0 when 2 then 0 when 7 then 0 else 1 end +
            case to_number(to_char(sysdate+4,'D')) when 1 then 0 when 2 then 0 when 7 then 0 else 1 end +
            case to_number(to_char(sysdate+5,'D')) when 1 then 0 when 2 then 0 when 7 then 0 else 1 end +
            case to_number(to_char(sysdate+6,'D')) when 1 then 0 when 2 then 0 when 7 then 0 else 1 end +
            case to_number(to_char(sysdate+7,'D')) when 1 then 0 when 2 then 0 when 7 then 0 else 1 end +
            case to_number(to_char(sysdate+8,'D')) when 1 then 0 when 2 then 0 when 7 then 0 else 1 end +
            case to_number(to_char(sysdate+9,'D')) when 1 then 0 when 2 then 0 when 7 then 0 else 1 end +
            case to_number(to_char(sysdate+10,'D')) when 1 then 0 when 2 then 0 when 7 then 0 else 1 end

    from dual;
    --------------------------------------------
    --샘코드
    select count(*) from
         (
         select sysdate+1 "XDAY" from dual union select sysdate+2 from dual
         union select sysdate+3 from dual union select sysdate+4 from dual
         union select sysdate+5 from dual union select sysdate+6 from dual
         union select sysdate+7 from dual union select sysdate+8 from dual
         union select sysdate+9 from dual union select sysdate+10 from dual
             )d
    where to_char(d.XDAY, 'dy')!='토'
    and to_char(d.XDAY, 'dy')!='일'
    and to_char(d.XDAY, 'dy')!='월';

    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm
    -- Q.127 이번달 중에 토요일, 일요일을 제외한 날의 개수를 구하면?
    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm
    --샘코드
    select count(*) from
         (

         select to_date(to_char(sysdate,'YYYY-MM')||'-01','YYYY-MM-DD')+no-1 "XDAY"
         from(select ROWNUM "NO" from EMPLOYEE union  select ROWNUM+20 from EMPLOYEE)xxx
             where no<=31
             )d
        where to_char(d.XDAY, 'dy')!='토'
        and to_char(d.XDAY, 'dy')!='일'
        and xday <= last_day(sysdate);

    ---------------------------------------------
    --내코드
    select count(*) from
         (
         select to_date(to_char(sysdate,'YYYY-MM')||'-01')+1 "XDAY" from dual union select to_date(to_char(sysdate,'YYYY-MM')||'-01')+2 from dual
         union select to_date(to_char(sysdate,'YYYY-MM')||'-01')+3 from dual union select to_date(to_char(sysdate,'YYYY-MM')||'-01')+4 from dual
         union select to_date(to_char(sysdate,'YYYY-MM')||'-01')+5 from dual union select to_date(to_char(sysdate,'YYYY-MM')||'-01')+6 from dual
         union select to_date(to_char(sysdate,'YYYY-MM')||'-01')+7 from dual union select to_date(to_char(sysdate,'YYYY-MM')||'-01')+8 from dual
         union select to_date(to_char(sysdate,'YYYY-MM')||'-01')+9 from dual union select to_date(to_char(sysdate,'YYYY-MM')||'-01')+10 from dual
         union select to_date(to_char(sysdate,'YYYY-MM')||'-01')+11 from dual union select to_date(to_char(sysdate,'YYYY-MM')||'-01')+12 from dual
         union select to_date(to_char(sysdate,'YYYY-MM')||'-01')+13 from dual union select to_date(to_char(sysdate,'YYYY-MM')||'-01')+14 from dual
         union select to_date(to_char(sysdate,'YYYY-MM')||'-01')+15 from dual union select to_date(to_char(sysdate,'YYYY-MM')||'-01')+16 from dual
         union select to_date(to_char(sysdate,'YYYY-MM')||'-01')+17 from dual union select to_date(to_char(sysdate,'YYYY-MM')||'-01')+18 from dual
         union select to_date(to_char(sysdate,'YYYY-MM')||'-01')+19 from dual union select to_date(to_char(sysdate,'YYYY-MM')||'-01')+20 from dual
         union select to_date(to_char(sysdate,'YYYY-MM')||'-01')+21 from dual union select to_date(to_char(sysdate,'YYYY-MM')||'-01')+22 from dual
         union select to_date(to_char(sysdate,'YYYY-MM')||'-01')+23 from dual union select to_date(to_char(sysdate,'YYYY-MM')||'-01')+24 from dual
         union select to_date(to_char(sysdate,'YYYY-MM')||'-01')+25 from dual union select to_date(to_char(sysdate,'YYYY-MM')||'-01')+26 from dual
         union select to_date(to_char(sysdate,'YYYY-MM')||'-01')+27 from dual union select to_date(to_char(sysdate,'YYYY-MM')||'-01')+28 from dual
         union select to_date(to_char(sysdate,'YYYY-MM')||'-01')+29 from dual union select to_date(to_char(sysdate,'YYYY-MM')||'-01')+30 from dual
         union select to_date(to_char(sysdate,'YYYY-MM')||'-01')+31 from dual
             )d
    where to_char(d.XDAY, 'dy')!='토'
    and to_char(d.XDAY, 'dy')!='일';
    ------------------
    select *
    from (
             select xxx.*,
                    rownum "RNUM"
             from (
                      select *
                      from EMPLOYEE
                      order by SALARY desc
                  ) xxx
             where ROWNUM <= 5
         ) where RNUM >=3;

    select e.* ,rownum from EMPLOYEE e order by SALARY desc
