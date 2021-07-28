---
layout: post
title:  "도메인 주도 설계"
date:   2020-10-15 12:00
author: chyusee
---

## 도메인?

소프트웨어로 해결하고자 하는 문제영역
도메인은 하위 도메인으로 나눌 수 있음
예: 도메인 = 쇼핑몰
하위 도메인 = 회원, 주문, 결재, 배송, 정산 등등

## 도메인 모델 패턴
####아키텍처 구성
<table style="width:100%;border:1px solid #000;text-align:center;">
    <tr>
        <th>계층</th>
        <th>설명</th>
    </tr>
    <tr>
        <td>표현(UI)</td>
        <td>
            사용자의 요청을 처리 및 정보 제공<br/>
            여기서 사용자는 유저뿐만 아니라 외부 시스템도 의미함
        </td>
    </tr>
    <tr>
        <td>응용(Application)</td>
        <td>
            사용자가 요청한 기능을 실행<br/>
            도메인 계층을 조합해서 기능을 실행
        </td>
    </tr>
    <tr>
        <td>도메인</td>
        <td>시스템이 제공할 도메인 규칙을 구현</td>
    </tr>
    <tr>
        <td>인프라스트럭처(Infrastructure)</td>
        <td>DB 나 메세징 같은 외부 시스템과의 연동 처리</td>
    </tr>
</table>

도메인 계층에서 주괸 도직이 구현됨
도메인 모델은 크게 엔티티와 벨류로 구분


