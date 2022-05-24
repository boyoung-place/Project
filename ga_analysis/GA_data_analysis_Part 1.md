# Google Analytics sample 데이터 분석 -Part 1. 탐색적 데이터 분석(EDA)

GA 데이터 분석 Part 1. 에서는 탐색적 데이터 분석(EDA)를 통해 데이터셋에 대한 탐구를 진행합니다.

## 프로젝트 목적

```markdown
1. GA 데이터셋에 대한 탐구
2. python을 통한 데이터 시각화
3. 가설 설정과 검증을 위한 데이터 분석
4. 가설과 데이터 탐구를 기반으로 한 Action Item 기획
```

## 목차

```markdown
- 데이터의 출처 및 설명
- 데이터 정의와 기초 source code
**- EDA (탐색적 데이터 분석)**
- 데이터 추출 및 현황 파악 (AS-IS 분석)
- 추가 분석과 리서치
- 결론과 Action Item (TO-BE 제안)
- 프로젝트 회고
```

## 사용 기술

```markdown
- 사용 언어: bigquery, sql, python
- 작업 툴: google spreadsheet, jupyter notebook
```

## EDA (탐색적 데이터 분석)

1. 진입 정보 중 **source/medium/channel grouping**에 대한 기본적인 정보 확인
    - 어떤 채널에서 가장 많이 유입 되었는지 살펴봅니다.
        
        ![2017년 1월 채널 당 유입 횟수.png](Google%20Analytics%20sample%20%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%20%E1%84%87%E1%85%AE%E1%86%AB%E1%84%89%E1%85%A5%E1%86%A8%20-Part%201%20%E1%84%90%E1%85%A1%E1%86%B7%E1%84%89%204ab3cb94cccf445a852dc412af3e9c77/2017%EB%85%84_1%EC%9B%94_%EC%B1%84%EB%84%90_%EB%8B%B9_%EC%9C%A0%EC%9E%85_%ED%9A%9F%EC%88%98.png)
        
        ![2017년 1월 채널 당 유입 횟수(직접유입 제외).png](Google%20Analytics%20sample%20%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%20%E1%84%87%E1%85%AE%E1%86%AB%E1%84%89%E1%85%A5%E1%86%A8%20-Part%201%20%E1%84%90%E1%85%A1%E1%86%B7%E1%84%89%204ab3cb94cccf445a852dc412af3e9c77/2017%EB%85%84_1%EC%9B%94_%EC%B1%84%EB%84%90_%EB%8B%B9_%EC%9C%A0%EC%9E%85_%ED%9A%9F%EC%88%98(%EC%A7%81%EC%A0%91%EC%9C%A0%EC%9E%85_%EC%A0%9C%EC%99%B8).png)
        
        - 2017년 1월 채널 당 유입 횟수 상위 10개 테이블
            
            
            | Source | 유입 세션 수 | 유입 비율(%) |
            | --- | --- | --- |
            | (direct) | 28207 | 81.75% |
            | google | 2425 | 7.03% |
            | youtube | 2276 | 6.60% |
            | Partners | 446 | 1.29% |
            | dfa | 403 | 1.17% |
            | facebook | 158 | 0.46% |
            | siliconvalley.about.com | 109 | 0.32% |
            | reddit | 80 | 0.23% |
            | quora.com | 74 | 0.21% |
            | qiita.com | 68 | 0.20% |
        - 2017년 1월 채널 당 유입 횟수 상위 10개 테이블 (직접 유입 제외)
            
            
            | Source | 유입 세션 수 | 유입 비율(%) |
            | --- | --- | --- |
            | google | 2425 | 38.51% |
            | youtube | 2276 | 36.14% |
            | Partners | 446 | 7.08% |
            | dfa | 403 | 6.40% |
            | facebook | 158 | 2.51% |
            | siliconvalley.about.com | 109 | 1.73% |
            | reddit | 80 | 1.27% |
            | quora.com | 74 | 1.18% |
            | qiita.com | 68 | 1.08% |
            | yahoo | 50 | 0.79% |
        - sql code
            
            ```sql
            with source_set as (select case when t1.source like '%google%' then 'google'
                        when t1.source like '%facebook%' then 'facebook'
                        when t1.source like '%youtube%' then 'youtube'
                        when t1.source like '%reddit%' then 'reddit'
                        else t1.source
                        end as source_clean
                        , count(distinct concat(t1.fullVisitorId, t1.visitId)) as session_count
            from basic_set as t1
            group by 1
            order by 2 desc)
            
            select source_clean, session_count, sum(session_count) over() as session_total
            from source_set
            -- (직접 유입 제외) where source_clean != '(direct)'
            order by 2 desc
            limit 10
            ```
            
        - 2017년 1월 한 달간 가장 유입이 많은 채널은 직접 유입인 **direct**가 압도적이며, 유입 횟수 28,207 건으로 전체 유입의 81.75%를 차지하고 있습니다.
        - 직접 유입을 대상에서 제외할 경우, 1월간 가장 유입이 많은 채널은 **google** 입니다. 유입 전체의 38.51%를 차지하고 있습니다.
        - 주요 유입 채널, Source는 **direct / google / youtube/ Partners** 으로 볼 수 있습니다.
    
    - 어떤 채널에서 “언제” 가장 많이 유입 되었는지 살펴봅니다.
        - Source&date별 유입 상위 3개
            
            
            | Source | date | 유입 세션 수 |
            | --- | --- | --- |
            | (direct) | 20170124 | 1737 |
            | (direct) | 20170125 | 1452 |
            | (direct) | 20170130 | 1124 |
        - Source&date별 유입 상위 3개 (직접 유입 제외)
            
            
            | Source | date | 유입 세션 수 |
            | --- | --- | --- |
            | google | 20170118 | 712 |
            | google | 20170108 | 590 |
            | google | 20170101 | 475 |
        - 직접 유입인 direct 채널에서 1월 24일 발생한 유입이 1,737건으로, 2017년 1월 중 가장 높습니다.
        - 직접 유입을 제외한 채널 중에서는 google 채널에서 1월 18일일 발생한 유입이 712건으로, 2017년 1월 중 가장 높습니다.
        
    - 일별 추이를 확인합니다.
        
        ![2017년 1월 일별 유입 추이 (1).png](Google%20Analytics%20sample%20%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%20%E1%84%87%E1%85%AE%E1%86%AB%E1%84%89%E1%85%A5%E1%86%A8%20-Part%201%20%E1%84%90%E1%85%A1%E1%86%B7%E1%84%89%204ab3cb94cccf445a852dc412af3e9c77/2017%EB%85%84_1%EC%9B%94_%EC%9D%BC%EB%B3%84_%EC%9C%A0%EC%9E%85_%EC%B6%94%EC%9D%B4_(1).png)
        
        ![2017년 1월 유입 일별 추이 - 유입 상위 채널 4개.png](Google%20Analytics%20sample%20%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%20%E1%84%87%E1%85%AE%E1%86%AB%E1%84%89%E1%85%A5%E1%86%A8%20-Part%201%20%E1%84%90%E1%85%A1%E1%86%B7%E1%84%89%204ab3cb94cccf445a852dc412af3e9c77/2017%EB%85%84_1%EC%9B%94_%EC%9C%A0%EC%9E%85_%EC%9D%BC%EB%B3%84_%EC%B6%94%EC%9D%B4_-_%EC%9C%A0%EC%9E%85_%EC%83%81%EC%9C%84_%EC%B1%84%EB%84%90_4%EA%B0%9C.png)
        
        ![2017년 1월 유입 일별 추이 - youtube & Partners.png](Google%20Analytics%20sample%20%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%20%E1%84%87%E1%85%AE%E1%86%AB%E1%84%89%E1%85%A5%E1%86%A8%20-Part%201%20%E1%84%90%E1%85%A1%E1%86%B7%E1%84%89%204ab3cb94cccf445a852dc412af3e9c77/2017%EB%85%84_1%EC%9B%94_%EC%9C%A0%EC%9E%85_%EC%9D%BC%EB%B3%84_%EC%B6%94%EC%9D%B4_-_youtube__Partners.png)
        
        - sql code
            
            ```sql
            select t1.date
            , count(distinct concat(t1.fullVisitorId, t1.visitId)) as session_count -- 유입 횟수
            from basic_set as t1
            group by 1
            order by 1
            ```
            
        - 전체 추이를 보았을 때에는 일주일 간격으로 week 초반에 상승하였다 주말에 하락하는 패턴이 지속적으로 보이며, 이러한 전체 추이는 (direct)채널의 유입 패턴의 영향을 많이 받는 것으로 보입니다. 1월 24~25일 기간에 평소의 약 1.5배 정도로 유입이 늘었는데 (direct) 채널에서의 유입의 영향으로 보입니다.
        - google 채널은 전체 추이와는 다르게, 대략 7일에서 10일정도의 기간을 두고 유입이 순간적으로 이전 기간 대비 50배 가량 급증하는 패턴을 보입니다.
        - 유입 횟수의 규모 차이로 인해 youtube 와 Partners의 유입 추이가 잘 보이지 않아 따로 차트를 다시 그려보았는데요, 일관된 유입 패턴은 보이지 않는 것 같습니다.
    
    *(여기서부터는 sql code를 생략하겠습니다.)*
    
    - 어떤 channelGrouping이 많이 사용되었을까?
        
        
        | channelGrouping | 유입 세션 수 | 사용 비율(%) |
        | --- | --- | --- |
        | Organic Search | 18744 | 54.33% |
        | Direct | 6545 | 18.97% |
        | Referral | 4469 | 12.95% |
        | Affiliates | 445 | 1.29% |
        | Paid Search | 1205 | 3.49% |
        | Display | 403 | 1.17% |
        | Social | 2687 | 7.79% |
        | (Other) | 1 | 0.00% |
        - 자연 검색 유입(웹문서에서 키워드 검색을 통한 유입)인 **Organic Search**가 1월 한 달간의 데이터 전체에서 **54.33%**로 제일 많은 사용량을 보입니다.
        - Organic Search > Direct > Referral
    
2. **사용자**의 **기기 정보**에 대한 기초적 통계
    - 사용자들은 어떤 브라우저를 많이 사용할까?
        
        
        | browser | 사용자 수 | 사용자 비율(%) |
        | --- | --- | --- |
        | Chrome | 19,687 | 67.97% |
        | Safari | 5,548 | 19.15% |
        | Firefox | 1,260 | 4.35% |
        | Internet Explorer | 741 | 2.56% |
        | Android Webview | 492 | 1.70% |
        - 사용자들이 가장 많이 사용하는 브라우저 상위 5개를 추려보았습니다.
        - Chrome이 67.97%로, 상당 수의 사용자가 크롬을 사용한다는 것을 알 수 있으며 그 다음으로 Safari가 19.15%의 사용 비율을 보입니다.
    - 사용자들은 어떤 기기를 많이 사용할까?
        
        
        | deviceCategory | 사용자 수 | 사용자 비율(%) |
        | --- | --- | --- |
        | desktop | 18,547 | 64.04% |
        | mobile | 9,001 | 31.08% |
        | tablet | 1,413 | 4.88% |
        - desktop이 유입에 가장 많이 사용되는 기기로 확인(사용 비율 64.04%)되며, desktop > mobile > tablet 순으로 사용되고 있습니다.
    - 사용자들은 어떤 **기기**에서 어떤 **브라우저**를 많이 사용할까?
        
        
        | deviceCategory | browser | 사용자 수 | 사용자 비율(%) |
        | --- | --- | --- | --- |
        | desktop | Chrome | 14,875 | 51.36% |
        | mobile | Chrome | 4,388 | 15.15% |
        | mobile | Safari | 3,427 | 11.83% |
        | desktop | Safari | 1,246 | 4.30% |
        | desktop | Firefox | 1,212 | 4.18% |
        - 유입시 **desktop**을 이용하는 사용자가 많고, desktop으로 **Chrome** 브라우저를 사용하여 유입되는 사용자가 많음(약 51%)을 확인할 수 있습니다. 또한, desktop ****사용 시에도, mobile 사용 시에도 **Chrome** 브라우저를 사용하는 사용자들이 많습니다.
    - 브라우저의 관점으로 다시 한번 볼까요?
        
        
        | browser | deviceCategory | 사용자 수 |
        | --- | --- | --- |
        | Chrome | desktop | 14,875 |
        | Chrome | mobile | 4,388 |
        | Safari | mobile | 3,427 |
        | Safari | desktop | 1,246 |
        - 위의 표와 동일한 결과를 브라우저의 기준으로 다시한번 살펴보면, Chrome 브라우저의 사용자들은 주로 desktop 이용을 선호하고, Safari 브라우저의 사용자들은 mobile 이용을 선호한다는 것을 확인할 수 있습니다. 이러한 사용자의 특징은 추후 개발 단계에서의 resource 분배 등에 활용될 수 있을 것 같네요.
    
     
    
3. 회사의 제품에 대한 간단 요약
    
    현재 제품에 대해서 아는 것이 아무 것도 없습니다. 이러한 문제 상황을 기술 통계를 기반하여 해결하는데 목적이 있습니다.
    
    - 회사에 어떤 제품 카테고리들이 있는지
        
        
        | 제품 카테고리(v2ProductCategory) | 제품 종류 수(productSKU unique count) |
        | --- | --- |
        | (not set) | 826 |
        | Apparel | 403 |
        | ${escCatTitle} | 343 |
        | Home/Shop by Brand/Google/ | 83 |
        | Home/Apparel/ | 71 |
        | Home/Apparel/Men's/ | 58 |
        | Home/Apparel/Men's/Men's-T-Shirts/ | 58 |
        | Home/Apparel/Women's/ | 55 |
        | Home/Shop by Brand/ | 55 |
        | Home/Apparel/Women's/Women's-T-Shirts/ | 49 |
        |  |  |
        - 회사에는 총 64개의 제품 카테고리가 존재하며, Apparel에서 상품 종류가 가장 다양합니다. (not set 제외)

- 카테고리별 상품 라인업
    
    ![A10E9803-D895-4BB5-9878-C55D5891ACEF_4_5005_c.jpeg](Google%20Analytics%20sample%20%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%20%E1%84%87%E1%85%AE%E1%86%AB%E1%84%89%E1%85%A5%E1%86%A8%20-Part%201%20%E1%84%90%E1%85%A1%E1%86%B7%E1%84%89%204ab3cb94cccf445a852dc412af3e9c77/A10E9803-D895-4BB5-9878-C55D5891ACEF_4_5005_c.jpeg)
    
    - 가장 상품 종류가 다양한 Apparel 카테고리만을 따로 뽑아 상품 관심도를 확인하였습니다. (그래프의 x축은 유입 수 입니다.)
    - 해당 카테고리에서는 Google Men's Zip Hoodie 가 가장 많은 유입 수를 보이며, 이 상품에 대한 사용자들의 관심도가 가장 많다고 볼 수 있습니다.
    - Google Men's 100% Cottom Short Sleeve Hero Tee 상품은 컬러 옵션별로 두루두루 관심도가 높은 것으로 확인됩니다.
    - 컬러 옵션을 제외하고 생각해 본다면, Google Men's 100% Cottom Short Sleeve Hero Tee 상품이 사용자들에게 제일 많은 관심을 받았습니다.

- 상세 제품별 가격대의 분포
    1. 먼저, 주요 카테고리의 상품 가격대 분포를 보았습니다. 이 때, 상품 식별 기준이 되는 ['v2ProductCategory','v2ProductName','productSKU','productVariant','productListName', 'productPrice'] 항목들을 기준으로 중복 제거를 하여 상품정보가 겹치는 부분이 없도록 했습니다. 'productPrice'를 중복 기준에 넣은 이유는, 시간의 변화에 따라 할인 등의 이유로 동일한 상품이더라도 가격에 변화가 일어날 수 있다고 보았기 때문입니다.
        
        이 중, 상품 라인 및 가격이 가장 다양한 "Apparel" 카테고리만 먼저 보았습니다.
        
        - **<Apparel 카테고리 상품 기초 통계>**
            
            
            | Category | stats | 상품 가격(productPrice) |
            | --- | --- | --- |
            | Apparel | count | 715 |
            | Apparel | mean | 42577053.15 |
            | Apparel | std | 45791394.06 |
            | Apparel | min | 1249000 |
            | Apparel | 25% | 16990000 |
            | Apparel | 50% | 23990000 |
            | Apparel | 75% | 55990000 |
            | Apparel | max | 671880000 |
            - Apparel 카테고리의 제품들은 평균 가격 약 42577053, 상품 중 최소 가격은 1249000이며 최대 가격은 671880000으로 확인됩니다.

1. 이어, 사용자들의 관심도가 높은 것으로 확인된 Google Men's 100% Cottom Short Sleeve Hero Tee 상품의 가격 정보를 확인해보았습니다.
    - **<"Google Men's 100% Cottom Short Sleeve Hero Tee" 상품 가격의 기초 통계표>**
        
        ![85F6DA3F-5F75-4CA9-B718-8BADE8ABA9E0_4_5005_c.jpeg](Google%20Analytics%20sample%20%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%20%E1%84%87%E1%85%AE%E1%86%AB%E1%84%89%E1%85%A5%E1%86%A8%20-Part%201%20%E1%84%90%E1%85%A1%E1%86%B7%E1%84%89%204ab3cb94cccf445a852dc412af3e9c77/85F6DA3F-5F75-4CA9-B718-8BADE8ABA9E0_4_5005_c.jpeg)
        
    - 해당 상품은 컬러 옵션에 따라 가격이 다소 상이한 모양을 보이는데, 전체적으로 15000000 ~ 18000000 범위의 가격대를 보입니다.
    - 최소 가격이 0으로 나오는 상품 옵션들이 있는데, 해당 데이터들은 이상치로 보입니다. 실제 운영에서 노출을 제외시킬 필요가 있어보입니다. 현재로서는 최소 가격이 발생한 날은 상품이 품절되었거나 판매가 중지된 경우로 추측됩니다.

Part 2. 에서는 Python을 이용한 데이터의 시각화를 통해 데이터의 현황 파악 및 인사이트를 도출을 진행합니다.