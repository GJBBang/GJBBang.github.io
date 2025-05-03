---
layout: post

title: swagger 정리

categories: java

tag: []

---



팀 내에서 EAI 구축을 위한 기획 단계에 있는데,, 최차장님께서 스웨거에 관심을 가지게 되었다.<br/>
지금 내가 하고 있는 프로젝트에서 스웨거 쓰는걸 실제로 보시고 흥미가 많이 생기신 듯 하다..
(항상 최근 기술, 새로운 것을 학습하시려는 차장님 존경합니다..)

아무튼 이참에 스웨거를 정리하려고 한다.



### 환경 세팅 ( Spring Boot 3.x 버전 )

- Gradle 의존성 추가

```java
// build.gradle

dependencies {
	...
	implementation 'org.springdoc:springdoc-openapi-starter-webmvc-ui:2.6.0'
	...
}
```


Spring Boot 2.x 버전의 경우 springdoc-openapi v1.6.x 사용 권장. (v1.7.x 이상부터는 Spring Boot 3.x 이상을 대상으로 설계)

❗️**주의**
Springfox (2.x or 3.x): 과거 많이 사용됐지만, Spring Boot 2.6+ 이후로 호환성 문제가 자주 발생함 (PathPatternParser 도입으로)
또한, Springfox는 사실상 유지보수가 중단된 상태이므로 (마지막 릴리즈: 2020.10) springdoc-openapi 사용이 권장된다.



### Config 설정

- 기본적인 설정

```java
package com.project.abook.common.config;

import io.swagger.v3.oas.models.Components;
import io.swagger.v3.oas.models.OpenAPI;
import io.swagger.v3.oas.models.info.Info;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class SwaggerConfig {

    @Bean
    public OpenAPI openAPI() {
        return new OpenAPI()
                .components(new Components())
                .info(apiInfo());
    }

    private Info apiInfo() {
        return new Info()
                .version("1.0")
                .title("Api TEST")
                .description("Api description");
    }
}
```

~~(메모: 추후 인증 추가)~~



### 어노테이션 정리

#### 1. `@Tag`

```java
@Tag(name = "ExTracker TEST", description = "TEST CRUD")
```

- **API 그룹 설명** (Swagger UI에서 좌측 카테고리로 표시됨)
- `name`: 그룹 이름 (예: "ExTracker TEST")
- `description`: 그룹 설명

🔎 예: Swagger UI에서 이 API는 “ExTracker TEST”라는 제목 아래 묶입니다.

------

#### 2. `@Operation`

```java
@Operation(summary = "해당 일자 가계부 정보", description = "해당 일자의 가계부 정보를 가져옵니다.")
```

- 하나의 API Operation(엔드포인트) 설명
- `summary`: 요약 제목 (Swagger UI에 굵게 표시됨)
- `description`: 상세 설명

------

#### 3. `@ApiResponses`

```java
@ApiResponses(value = {
    @ApiResponse(...),
    @ApiResponse(...)
})
```

- 이 API에서 발생할 수 있는 **HTTP 응답 코드들에 대한 설명**을 나열

------

#### 4. `@ApiResponse`

```java
@ApiResponse(
    responseCode = "200",
    description = "성공",
    content = @Content(
        mediaType = "application/json",
        schema = @Schema(implementation = ExTracker.class)
    )
)
```

- **특정 응답 코드**에 대한 설명
- `responseCode`: HTTP 상태 코드 (`"200"`, `"500"` 등)
- `description`: 해당 상태 코드에 대한 설명
- `content`: 반환 타입과 미디어 타입

------

#### 5. `@Content`

```java
@Content(
    mediaType = "application/json",
    schema = @Schema(implementation = ExTracker.class)
)
```

- API 응답의 **미디어 타입과 스키마 정보**
- `mediaType`: 응답 형식 (`application/json`, `text/plain`, 등)
- `schema`: 반환 데이터 구조 설명 → `@Schema`

------

#### 6. `@Schema`

```java
@Schema(implementation = ExTracker.class)
```

- **응답 또는 요청 데이터 구조를 명시**
- `implementation`: 반환할 DTO/엔티티 클래스 (Swagger 문서에 구조가 표시됨)

------

#### 7. `@ExampleObject`

```java
@ExampleObject(
    name = "ServerErrorExample",
    summary = "서버 에러",
    description = "서버 내부 오류 발생 시 반환되는 메시지 형식",
    value = "{ ... }"
)
```

- 응답 예시 데이터(JSON)를 Swagger 문서에 표시
- `value`: 예시 JSON 문자열
- `summary`, `description`: 예시 설명
- 예시 하나만 넣을 경우 `examples = @ExampleObject(...)`

------

#### 8. `@Parameter`

```java
@Parameter(name = "id", description = "가계부 id", required = true, example = "1")
```

- **요청 파라미터 설명** (`@RequestParam`, `@PathVariable` 등에 붙여 사용)
- `name`: 파라미터 이름
- `description`: 설명
- `required`: 필수 여부
- `example`: 예시 값

~~(메모: 추후 DTO 스키마 추가)~~



#### 예시

```java
package com.project.abook.extracker.controller;

import com.project.abook.extracker.domain.ExTracker;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.Parameter;
import io.swagger.v3.oas.annotations.media.Content;
import io.swagger.v3.oas.annotations.media.ExampleObject;
import io.swagger.v3.oas.annotations.media.Schema;
import io.swagger.v3.oas.annotations.responses.ApiResponse;
import io.swagger.v3.oas.annotations.responses.ApiResponses;
import io.swagger.v3.oas.annotations.tags.Tag;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestBody;

@Tag(name = "ExTracker TEST", description = "TEST CRUD")
public interface ExTrackerRestControllerDocs {

    @Operation(summary = "해당 일자 가계부 정보", description = "해당 일자의 가계부 정보를 가져옵니다.")
    @ApiResponses(value = {
            @ApiResponse(responseCode = "200", description = "성공",
                    content = @Content(
                            mediaType = "application/json",
                            schema = @Schema(implementation = ExTracker.class)
                    )),
            @ApiResponse(responseCode = "500", description = "서버에러 발생",
                    content = @Content(
                            mediaType = "application/json",
                            examples = @ExampleObject(
                                    name = "ServerErrorExample",
                                    summary = "서버 에러",
                                    description = "서버 내부 오류 발생 시 반환되는 메시지 형식",
                                    value = "{ \"timestamp\": \"2025-05-01T10:00:00\", \"status\": 500, \"error\": \"Internal Server Error\", \"message\": \"예기치 못한 오류가 발생했습니다.\", \"path\": \"/api/v1/exTracker/1\" }"
                            )
                    ))
    })
    public ResponseEntity<String> getExTracker(
            @Parameter(name = "id", description = "가계부 id", required = true, example = "1")
            @PathVariable Long id
    );

    @Operation(summary = "해당 일자 가계부 정보 저장", description = "해당 일자의 가계부 정보를 저장합니다.")
    @ApiResponses(value = {
            @ApiResponse(responseCode = "200", description = "성공",
                    content = @Content(
                            mediaType = "application/json",
                            examples = @ExampleObject(
                                    name = "",
                                    summary = "성공",
                                    description = "가계부 저장 성공",
                                    value = "{ \"sucessYN\": \"Y\", \"message\": \"가계부 저장 성공\" }"
                            )
                    )),
            @ApiResponse(responseCode = "500", description = "서버에러 발생",
                    content = @Content(
                            mediaType = "application/json",
                            examples = @ExampleObject(
                                    name = "ServerErrorExample",
                                    summary = "서버 에러",
                                    description = "서버 내부 오류 발생 시 반환되는 메시지 형식",
                                    value = "{ \"timestamp\": \"2025-05-01T10:00:00\", \"status\": 500, \"error\": \"Internal Server Error\", \"message\": \"예기치 못한 오류가 발생했습니다.\", \"path\": \"/api/v1/exTracker/save\" }"
                            )
                    ))
    })
    public ResponseEntity<Void> saveExTracker(
            @RequestBody @io.swagger.v3.oas.annotations.parameters.RequestBody(
                    description = "생성할 가계부 정보", required = true,
                    content = @Content(
                            mediaType = "application/json",
                            examples = @ExampleObject(
                                    name = "default Object",
                                    summary = "",
                                    description = "",
                                    value = "{ \"id\": 1, \"title\": \"title ex\", \"price\": 3000 }"
                            )
                    )
            ) ExTracker exTracker
    );
}

```

번외로, 이렇게 스웨거를 작성할 시 컨트롤러가 굉장히(?) 지저분해지는데 위 예시와 같이 인터페이스로 따로 빼서 작성하는게 좋다고 생각한다.



### 기본 접속 경로

```java
http://localhost:8080/swagger-ui/index.html
```



#### 🔧 접속 경로 변경 방법

`application.yml` 또는 `application.properties`에 아래 설정 추가:

#### ▸ `application.yml`

```yaml
springdoc:
  swagger-ui:
    path: /docs # 원하는 경로로 변경
```

#### ▸ `application.properties`

```properties
springdoc.swagger-ui.path=/docs
```

####  변경 후 접속 URL 예

```bash
http://localhost:8080/docs
```