---
layout: post
title: "[Spring] Spring-graphql 라이브러리 리뷰"
author: qwlake
categories: Spring
tags: graphql spring-graphql
---

# spring-graphql

[Introducing Spring GraphQL](https://spring.io/blog/2021/07/06/introducing-spring-graphql)

[spring-projects/spring-graphql](https://github.com/spring-projects/spring-graphql)

[Spring GraphQL 1.0.0-M1 API](https://docs.spring.io/spring-graphql/docs/1.0.0-M1/api/)

[](https://mvnrepository.com/artifact/org.springframework.graphql/spring-graphql/1.0.0-M1)

## 패키지

- org.springframework.graphql - GraphQL request를 처리하기 위한 최상위 추상화 단계. `GraphQlService`로 request를 처리하고 `RequestInput`로 input을 표현한다
- org.springframework.graphql.data - Spring Data 지원. Querydsl인 `DataFetcher`를 구현하고 있다
- org.springframework.graphql.execution - GraphQL request 실행을 지원. `GraphQL`을 구성하고 호출하기 위한 추상화 포함
- org.springframework.graphql.security - Spring Security 지원
- org.springframework.graphql.test.tester - GraphQL 클라이언트 테스트 지원
- org.springframework.graphql.web
- org.springframework.graphql.web.webflux - Spring WebFlux 애플리케이션에서 사용하기 위한 HTTP, WebSocket 핸들러
- org.springframework.graphql.web.webmvc - Spring WebMvc 애플리케이션에서 사용하기 위한 HTTP, WebSocket 핸들러

(org.springframework.graphql.web.webmvc 위주로 알아볼 예정)

### org.springframework.graphql.web.webmvc

- GraphiQlHandler - GraphiQl UI 페이지를 렌더링하기 위한 Spring MVC functional handler
- GraphQlHttpHandler - `RouterFunctions`를 통해 WebMvc.fn 엔드포인트를 노출하기 위한 GraphQL 핸들러
- GraphQlWebSocketHandler - `GraphQL Over WebSocket Protocol`에 기반을 둔 WebSocketHandler. spring-websocket과 함께 서블릿 컨테이너에서도 사용
- SchemaHandler - `SchemaPrinter`를 통해 `GraphQLSchema`를 출력하는 Spring MVC functional handler

    **GraphQlHttpHandler 가 생성되는 과정**

    1. DefaultGraphQlSourceBuilder에서 GraphQL 인스턴스를 생성하고 GraphQlSource와 함께 래핑한 후 반환

        ```java
        // org/springframework/graphql/execution/DefaultGraphQlSourceBuilder.java

        class DefaultGraphQlSourceBuilder implements GraphQlSource.Builder {

        	private final List<Resource> schemaResources = new ArrayList<>();

        	private RuntimeWiring runtimeWiring = RuntimeWiring.newRuntimeWiring().build();

        	private final List<DataFetcherExceptionResolver> exceptionResolvers = new ArrayList<>();

        	private final List<GraphQLTypeVisitor> typeVisitors = new ArrayList<>();

        	private final List<Instrumentation> instrumentations = new ArrayList<>();

        	private Consumer<GraphQL.Builder> graphQlConfigurers = (builder) -> {
        	};

        	DefaultGraphQlSourceBuilder() {
        		this.typeVisitors.add(ContextDataFetcherDecorator.TYPE_VISITOR);
        	}

        	...

        	@Override
        	public GraphQlSource build() {
        		TypeDefinitionRegistry registry = this.schemaResources.stream()
        				.map(this::parseSchemaResource).reduce(TypeDefinitionRegistry::merge)
        				.orElseThrow(() -> new IllegalArgumentException("'schemaResources' should not be empty"));

        		GraphQLSchema schema = new SchemaGenerator().makeExecutableSchema(registry, this.runtimeWiring);
        		for (GraphQLTypeVisitor visitor : this.typeVisitors) {
        			schema = SchemaTransformer.transformSchema(schema, visitor);
        		}

        		GraphQL.Builder builder = GraphQL.newGraphQL(schema);
        		builder.defaultDataFetcherExceptionHandler(new ExceptionResolversExceptionHandler(this.exceptionResolvers));
        		if (!this.instrumentations.isEmpty()) {
        			builder = builder.instrumentation(new ChainedInstrumentation(this.instrumentations));
        		}
        		this.graphQlConfigurers.accept(builder);
        		GraphQL graphQl = builder.build();

        		return new CachedGraphQlSource(graphQl, schema);
        	}

        	...

        }
        ```

    2. 반환된 GraphQlSource를 ExecutionGraphQlService에 주입 (GraphQL 인스턴스를 얻고 쿼리를 실행하기 위해서)
        - ExecutionGraphQlService는 GraphQlService를 구현하고 있다

        ```java
        // org/springframework/graphql/execution/ExecutionGraphQlService.java

        public class ExecutionGraphQlService implements GraphQlService {

        	private final GraphQlSource graphQlSource;

        	public ExecutionGraphQlService(GraphQlSource graphQlSource) {
        		this.graphQlSource = graphQlSource;
        	}

        	...

        }
        ```

    3. ExecutionGraphQlService(GraphQlService)를 DefaultWebGraphQlHandlerBuilder에 주입

        ```java
        // org/springframework/graphql/web/DefaultWebGraphQlHandlerBuilder.java

        class DefaultWebGraphQlHandlerBuilder implements WebGraphQlHandler.Builder {

        	private final GraphQlService service;

        	...

        	DefaultWebGraphQlHandlerBuilder(GraphQlService service) {
        		Assert.notNull(service, "GraphQlService is required");
        		this.service = service;
        	}

        	...

        }
        ```

    4. DefaultWebGraphQlHandlerBuilder를 통해 WebGraphQlHandler 생성

        ```java
        // org/springframework/graphql/web/DefaultWebGraphQlHandlerBuilder.java

        class DefaultWebGraphQlHandlerBuilder implements WebGraphQlHandler.Builder {

        	private final GraphQlService service;

        	@Nullable
        	private List<WebInterceptor> interceptors;

        	@Nullable
        	private List<ThreadLocalAccessor> accessors;

        	...

        	@Override
        	public WebGraphQlHandler build() {
        		List<WebInterceptor> interceptorsToUse =
        				(this.interceptors != null) ? this.interceptors : Collections.emptyList();

        		WebGraphQlHandler targetHandler = (webInput) ->
        				this.service.execute(webInput).map((result) -> new WebOutput(webInput, result));

        		WebGraphQlHandler interceptionChain = interceptorsToUse.stream()
        				.reduce(WebInterceptor::andThen)
        				.map((interceptor) -> (WebGraphQlHandler) (input) -> interceptor.intercept(input, targetHandler))
        				.orElse(targetHandler);

        		return (CollectionUtils.isEmpty(this.accessors) ? interceptionChain
        				: new ThreadLocalExtractingHandler(interceptionChain, ThreadLocalAccessor.composite(this.accessors)));
        	}

        	...

        }
        ```

    5. GraphQlHttpHandler에 WebGraphQlHandler 주입
        - webflux가 아닌 webmvc지만 Mono를 사용해서 ServerResponse를 받는 모습을 볼 수 있다.

        ```java
        // org/springframework/graphql/web/webmvc/GraphQlHttpHandler.java

        public class GraphQlHttpHandler {

        	private static final Log logger = LogFactory.getLog(GraphQlHttpHandler.class);

        	private static final ParameterizedTypeReference<Map<String, Object>> MAP_PARAMETERIZED_TYPE_REF =
        			new ParameterizedTypeReference<Map<String, Object>>() {};

        	private final WebGraphQlHandler graphQlHandler;

        	/**
        	 * Create a new instance.
        	 * @param graphQlHandler common handler for GraphQL over HTTP requests
        	 */
        	public GraphQlHttpHandler(WebGraphQlHandler graphQlHandler) {
        		Assert.notNull(graphQlHandler, "WebGraphQlHandler is required");
        		this.graphQlHandler = graphQlHandler;
        	}

        	/**
        	 * Handle GraphQL requests over HTTP.
        	 * @param request the incoming HTTP request
        	 * @return the HTTP response
        	 * @throws ServletException may be raised when reading the request body, e.g.
        	 * {@link HttpMediaTypeNotSupportedException}.
        	 */
        	public ServerResponse handleRequest(ServerRequest request) throws ServletException {
        		WebInput input = new WebInput(request.uri(), request.headers().asHttpHeaders(), readBody(request), null);
        		if (logger.isDebugEnabled()) {
        			logger.debug("Executing: " + input);
        		}
        		Mono<ServerResponse> responseMono = this.graphQlHandler.handle(input).map((output) -> {
        			if (logger.isDebugEnabled()) {
        				logger.debug("Execution complete");
        			}
        			ServerResponse.BodyBuilder builder = ServerResponse.ok();
        			if (output.getResponseHeaders() != null) {
        				builder.headers((headers) -> headers.putAll(output.getResponseHeaders()));
        			}
        			return builder.body(output.toSpecification());
        		});
        		return ServerResponse.async(responseMono);
        	}

        	private static Map<String, Object> readBody(ServerRequest request) throws ServletException {
        		try {
        			return request.body(MAP_PARAMETERIZED_TYPE_REF);
        		}
        		catch (IOException ex) {
        			throw new ServerWebInputException("I/O error while reading request body", null, ex);
        		}
        	}

        }
        ```

# spring-graphql-starter

## schema

`shcema.graphqls` 파일에 스키마를 정의한다. 스키마를 정의하고 이 스키마에 어떻게 접근할 것인지에 대한 쿼리를 Query 항목에 다음과 같이 정의한다.

```graphql
type Query {
    greeting: String
    artifactRepositories : [ArtifactRepository]
    artifactRepository(id : ID!) : ArtifactRepository
    project(slug: ID!): Project
}
```

## endpoint

[https://github.com/spring-projects/spring-graphql/blob/f1bc239b57/graphql-spring-boot-starter/src/main/java/org/springframework/graphql/boot/GraphQlWebMvcAutoConfiguration.java](https://github.com/spring-projects/spring-graphql/blob/f1bc239b57/graphql-spring-boot-starter/src/main/java/org/springframework/graphql/boot/GraphQlWebMvcAutoConfiguration.java)

위 링크 보면 graphQlRouterFunction() 에서 다음과 같이 작성

```java
// graphql-spring-boot-starter/src/main/java/org/springframework/graphql/boot/GraphQlWebMvcAutoConfiguration.java

RouterFunctions.Builder builder = RouterFunctions.route()
		.GET(graphQLPath, request ->
				ServerResponse.status(HttpStatus.METHOD_NOT_ALLOWED)
						.headers(headers -> headers.setAllow(Collections.singleton(HttpMethod.POST)))
						.build())
		.POST(graphQLPath,
				contentType(MediaType.APPLICATION_JSON).and(accept(MediaType.APPLICATION_JSON)),
				handler::handleRequest);
```

따라서 서버의 graphql 자원을 호출할 경우 아래와 같이 사용할 수 있다.

```
GET /graphql/schema
GET /graphql -> Not allowd
POST /graphql body-type: json
```

### 호출 예제

- raw-json으로 해서 아래처럼 보내기

```json
# request body
{
    "query":"{artifactRepositories {id name} }"
}

# response body
{
    "data": {
        "artifactRepositories": [
            {
                "id": "spring-releases",
                "name": "Spring Releases"
            },
            {
                "id": "spring-milestones",
                "name": "Spring Milestones"
            },
            {
                "id": "spring-snapshots",
                "name": "Spring Snapshots"
            }
        ]
    }
}
```

- PostMan에서 GraphQL 선택하고 아래처럼 보내기 (content-type은 여전히 json으로 보내짐)

```graphql
# request body
{
    artifactRepositories {
        id 
        name
    } 
}

#response body
{
    "data": {
        "artifactRepositories": [
            {
                "id": "spring-releases",
                "name": "Spring Releases"
            },
            {
                "id": "spring-milestones",
                "name": "Spring Milestones"
            },
            {
                "id": "spring-snapshots",
                "name": "Spring Snapshots"
            }
        ]
    }
}
```

# ISSUE

- Could not find org.springframework.graphql:spring-graphql:1.0.0-M1.

    ```
    Execution failed for task ':compileKotlin'.
    > Error while evaluating property 'filteredArgumentsMap' of task ':compileKotlin'
       > Could not resolve all files for configuration ':compileClasspath'.
          > Could not find org.springframework.graphql:spring-graphql:1.0.0-M1.
            Required by:
                project :

    Possible solution:
     - Declare repository providing the artifact, see the documentation at https://docs.gradle.org/current/userguide/declaring_repositories.html
    ```

    ~~왜 에러가 나는지..~~

    → 레포지토리([https://repo.spring.io/libs-milestone/](https://repo.spring.io/libs-milestone/)) 별도로 추가해야함

- 윈도우에서는 [https://github.com/spring-projects/spring-graphql](https://github.com/spring-projects/spring-graphql) 가 build에 실패하는 현상 (해당 프로젝트 하위에 존재하는 samples 역시 마찬가지로 실패)

    ```
    Execution failed for task ':buildSrc:checkFormatMain'.
    	> Formatting violations found in the following files:
    	   * src\main\java\org\springframework\graphql\build\ConventionsPlugin.java
    ```

- PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target 에러

    ```
    Execution failed for task ':kaptGenerateStubsKotlin'.
    > Could not resolve all files for configuration ':compileClasspath'.
       > Could not resolve org.springframework.experimental:graphql-spring-boot-starter:1.0.0-SNAPSHOT.
         Required by:
             project :
          > Could not resolve org.springframework.experimental:graphql-spring-boot-starter:1.0.0-SNAPSHOT.
             > Unable to load Maven meta-data from https://repo.spring.io/milestone/org/springframework/experimental/graphql-spring-boot-starter/1.0.0-SNAPSHOT/maven-metadata.xml.
                > Could not get resource 'https://repo.spring.io/milestone/org/springframework/experimental/graphql-spring-boot-starter/1.0.0-SNAPSHOT/maven-metadata.xml'.
                   > Could not GET 'https://repo.spring.io/milestone/org/springframework/experimental/graphql-spring-boot-starter/1.0.0-SNAPSHOT/maven-metadata.xml'.
                      > The server may not support the client's requested TLS protocol versions: (TLSv1.2, TLSv1.3). You may need to configure the client to allow other protocols to be used. See: https://docs.gradle.org/6.8/userguide/build_environment.html#gradle_system_properties
                         > PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target
    ```

    → [https://www.lesstif.com/system-admin/java-validatorexception-keystore-ssl-tls-import-12451848.html](https://www.lesstif.com/system-admin/java-validatorexception-keystore-ssl-tls-import-12451848.html)