---
layout: post
title: lombok 의 Custom Builder 이슈
excerpt_separator:  <!--more-->
---

lombok 의 커스텀 builder 사용 시  이슈

```java
package com.payco.shopping.api.product.response;

import lombok.Builder;
import lombok.Getter;
import lombok.Setter;

/**
 * @author hanjin lee
 */
@Getter
@Setter
@Builder(builderMethodName = "hiddenBuilder", builderClassName = "ProductDetailUrlResponseBuilder" )
public class ProductDetailUrlResponse {
    private String url;


    public static ProductDetailUrlResponseBuilder builder(String a, String b){
        return hiddenBuilder().url(a+b);
    }

}

```
이와 같이 커스텀 빌더를 사용하려 할때 querydsl과 함께 사용하면 컴파일 순서 때문에 해당 커스텀빌더를 찾지 못하는 일이 발생한다.

로그를 보면
```
16:59:54: Executing task 'bootRun'...

:common:initQuerydslSourcesDir
:common:compileQuerydsl UP-TO-DATE
:common:compileJava UP-TO-DATE
:common:processResources UP-TO-DATE
:common:classes UP-TO-DATE
:common:jar UP-TO-DATE
:web.front.api:initQuerydslSourcesDir
Note: Running JPAAnnotationProcessor
Note: Serializing Entity types
Note: Generating com.payco.shopping.api.board.entity.QBoardArticle for [com.payco.shopping.api.board.entity.BoardArticle]
Note: Generating com.payco.shopping.api.display.entity.QBannerSectionView for [com.payco.shopping.api.display.entity.BannerSectionView]
Note: Generating com.payco.shopping.api.board.entity.QBoard for [com.payco.shopping.api.board.entity.Board]
Note: Generating com.payco.shopping.api.display.entity.QSection for [com.payco.shopping.api.display.entity.Section]
Note: Generating com.payco.shopping.api.inquiry.entity.QInquiryAnswer for [com.payco.shopping.api.inquiry.entity.InquiryAnswer]
Note: Generating com.payco.shopping.api.inquiry.entity.QInquiry for [com.payco.shopping.api.inquiry.entity.Inquiry]
Note: Generating com.payco.shopping.common.mall.entity.QMall for [com.payco.shopping.common.mall.entity.Mall]
Note: Generating com.payco.shopping.common.mall.entity.QMallAppkey for [com.payco.shopping.common.mall.entity.MallAppkey]
Note: Generating com.payco.shopping.api.inquiry.entity.QInquiryType for [com.payco.shopping.api.inquiry.entity.InquiryType]
Note: Generating com.payco.shopping.api.profile.entity.QRecentProduct for [com.payco.shopping.api.profile.entity.RecentProduct]
Note: Generating com.payco.shopping.api.display.entity.QBannerSectionAccountView for [com.payco.shopping.api.display.entity.BannerSectionAccountView]
Note: Generating com.payco.shopping.common.admin.entity.QAdmin for [com.payco.shopping.common.admin.entity.Admin]
Note: Generating com.payco.shopping.api.board.entity.QBoardCategory for [com.payco.shopping.api.board.entity.BoardCategory]
Note: Generating com.payco.shopping.api.display.entity.QBannerSectionAccountImageView for [com.payco.shopping.api.display.entity.BannerSectionAccountImageView]
Note: Generating com.payco.shopping.api.profile.entity.QLikeProduct for [com.payco.shopping.api.profile.entity.LikeProduct]
Note: Generating com.payco.shopping.api.product.entity.QDisplayCategoryView for [com.payco.shopping.api.product.entity.DisplayCategoryView]
Note: Running JPAAnnotationProcessor
/Users/hanjin/Repositories/payco_shopping/web.front.api/src/main/java/com/payco/shopping/api/product/response/ProductDetailUrlResponse.java:17: error: cannot find symbol
    public static ProductDetailUrlResponseBuilder builder(String a, String b){
                  ^
  symbol:   class ProductDetailUrlResponseBuilder
  location: class ProductDetailUrlResponse
Note: Running JPAAnnotationProcessor
1 error
:web.front.api:compileQuerydsl FAILED

FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':web.front.api:compileQuerydsl'.
> Compilation failed; see the compiler error output for details.

* Try:
Run with --stacktrace option to get the stack trace. Run with --info or --debug option to get more log output. Run with --scan to get full insights.

* Get more help at https://help.gradle.org

BUILD FAILED in 4s
7 actionable tasks: 3 executed, 4 up-to-date
Compilation failed; see the compiler error output for details.
16:59:58: Task execution finished 'bootRun'.

```

이경우

gradle 설정에

```groovy
project.afterEvaluate {
    project.tasks.compileQuerydsl.options.compilerArgs = [
            "-proc:only",
            "-processor", project.querydsl.processors() +
                    ',lombok.launch.AnnotationProcessorHider$AnnotationProcessor'
    ]
}

``` 
를 추가 해주면 됨

참조 : https://github.com/ewerk/gradle-plugins/issues/59
