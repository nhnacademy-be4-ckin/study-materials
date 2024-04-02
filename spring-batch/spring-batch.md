# Spring Batch

: spring batch는 로깅/추적, 트랜잭션 관리, 작업 처리 통계, 작업 재시작, 건너뛰기, 리소스 관리 등 대용량 레코드 처리에 필수적인 기능을 제공한다.

1. **pulgin - spring batch install**
2. **pom.xml 의존성 추가**

```xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-batch</artifactId>
</dependency>
```

1. **@EnableBatchProcessing**
    - @EnableBatchProcessing를 선언을 하면 스프링 배치를 작동시켜줍니다.
    - 스프링 배치의 모든 초기화 및 실행을 이루고 총 4개의 설정 클래스를 실행합니다.
    - 스프링 부트 배치의 자동 설정 클래스가 실행됨으로 빈으로 등록된 Job을 조회해서 초기화와 동시에 Job를 수행하도록 구성되어있습니다.
    - spring boot v3.0부터는 필요하지 않음.

## **Spring Batch 기본 구조**

![Untitled (1)](https://github.com/nhnacademy-be4-ckin/study-materials/assets/93987703/9d1cbe30-4acf-4925-9e28-7c5b16e498ff)


- 스프링 배치는 최소 한 개 이상의 Job을 생성할 수 있고,
- 하나의 Job은 여러개의 step으로 구성될 수 있다.
- Step은 tasklet단위로 처리되고, 그 중에서 ChunkOrientedTasklet을 통해 Chunk를 처리하며 이를 구성하는 3 요소로 ItempReader, ItempProcessor, ItemWriter가 있다.

## Meta Table

<aside>
💡 Spring Batch Meta-Data Tables
스프링 배치에서는 각 작업이 실행될 때마다 Job 또는 Step에 대한 상태와 이력을 기록한다.
따라서 스프링 배치를 구성하기 위해서는 메타데이터 테이블을 구성해야 한다. 
- 스프링 배치 메타데이터 테이블을 자동 생성할 수 있는 설정이 있다. 
spring.batch.jdbc.initialize-schema=always

</aside>

![Untitled (2)](https://github.com/nhnacademy-be4-ckin/study-materials/assets/93987703/e6f433ad-3b53-4d60-9f30-f910ce0a6023)


1. Spring Batch의 메타 데이터는 다음과 같은 내용들을 담고 있다.
- 이전에 실행한 Job이 어떤 것들이 있는지
- 최근 실패한 Batch Parameter가 어떤 것들이 있고, 성공한 Job은 어떤 것들이 있는지
- 다시 실행한다면 어디서부터 시작하면 될지
- 어떤 Job에 어떤 Step들이 있었고, Step들 중 성공한 Step과 실패한 Step들은 어떤 것들이 있는지

1. 테이블 생성시 충돌을 막기 위해 DDL로 미리 생성한다. 

```sql
# Meta Table DDL

CREATE TABLE `BATCH_JOB_EXECUTION` (
  `JOB_EXECUTION_ID` bigint NOT NULL,
  `VERSION` bigint DEFAULT NULL,
  `JOB_INSTANCE_ID` bigint NOT NULL,
  `CREATE_TIME` datetime(6) NOT NULL,
  `START_TIME` datetime(6) DEFAULT NULL,
  `END_TIME` datetime(6) DEFAULT NULL,
  `STATUS` varchar(10) DEFAULT NULL,
  `EXIT_CODE` varchar(2500) DEFAULT NULL,
  `EXIT_MESSAGE` varchar(2500) DEFAULT NULL,
  `LAST_UPDATED` datetime(6) DEFAULT NULL,
  `JOB_CONFIGURATION_LOCATION` varchar(2500) DEFAULT NULL,
  PRIMARY KEY (`JOB_EXECUTION_ID`),
  KEY `JOB_INST_EXEC_FK` (`JOB_INSTANCE_ID`),
  CONSTRAINT `JOB_INST_EXEC_FK` FOREIGN KEY (`JOB_INSTANCE_ID`) REFERENCES `BATCH_JOB_INSTANCE` (`JOB_INSTANCE_ID`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci

CREATE TABLE `BATCH_JOB_EXECUTION_CONTEXT` (
  `JOB_EXECUTION_ID` bigint NOT NULL,
  `SHORT_CONTEXT` varchar(2500) NOT NULL,
  `SERIALIZED_CONTEXT` text,
  PRIMARY KEY (`JOB_EXECUTION_ID`),
  CONSTRAINT `JOB_EXEC_CTX_FK` FOREIGN KEY (`JOB_EXECUTION_ID`) REFERENCES `BATCH_JOB_EXECUTION` (`JOB_EXECUTION_ID`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci

CREATE TABLE `BATCH_JOB_EXECUTION_PARAMS` (
  `JOB_EXECUTION_ID` bigint NOT NULL,
  `TYPE_CD` varchar(6) NOT NULL,
  `KEY_NAME` varchar(100) NOT NULL,
  `STRING_VAL` varchar(250) DEFAULT NULL,
  `DATE_VAL` datetime(6) DEFAULT NULL,
  `LONG_VAL` bigint DEFAULT NULL,
  `DOUBLE_VAL` double DEFAULT NULL,
  `IDENTIFYING` char(1) NOT NULL,
  KEY `JOB_EXEC_PARAMS_FK` (`JOB_EXECUTION_ID`),
  CONSTRAINT `JOB_EXEC_PARAMS_FK` FOREIGN KEY (`JOB_EXECUTION_ID`) REFERENCES `BATCH_JOB_EXECUTION` (`JOB_EXECUTION_ID`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci

CREATE TABLE `BATCH_JOB_EXECUTION_SEQ` (
  `ID` bigint NOT NULL,
  `UNIQUE_KEY` char(1) NOT NULL,
  UNIQUE KEY `UNIQUE_KEY_UN` (`UNIQUE_KEY`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci

CREATE TABLE `BATCH_JOB_INSTANCE` (
  `JOB_INSTANCE_ID` bigint NOT NULL,
  `VERSION` bigint DEFAULT NULL,
  `JOB_NAME` varchar(100) NOT NULL,
  `JOB_KEY` varchar(32) NOT NULL,
  PRIMARY KEY (`JOB_INSTANCE_ID`),
  UNIQUE KEY `JOB_INST_UN` (`JOB_NAME`,`JOB_KEY`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci

CREATE TABLE `BATCH_JOB_SEQ` (
  `ID` bigint NOT NULL,
  `UNIQUE_KEY` char(1) NOT NULL,
  UNIQUE KEY `UNIQUE_KEY_UN` (`UNIQUE_KEY`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci

CREATE TABLE `BATCH_STEP_EXECUTION` (
  `STEP_EXECUTION_ID` bigint NOT NULL,
  `VERSION` bigint NOT NULL,
  `STEP_NAME` varchar(100) NOT NULL,
  `JOB_EXECUTION_ID` bigint NOT NULL,
  `START_TIME` datetime(6) NOT NULL,
  `END_TIME` datetime(6) DEFAULT NULL,
  `STATUS` varchar(10) DEFAULT NULL,
  `COMMIT_COUNT` bigint DEFAULT NULL,
  `READ_COUNT` bigint DEFAULT NULL,
  `FILTER_COUNT` bigint DEFAULT NULL,
  `WRITE_COUNT` bigint DEFAULT NULL,
  `READ_SKIP_COUNT` bigint DEFAULT NULL,
  `WRITE_SKIP_COUNT` bigint DEFAULT NULL,
  `PROCESS_SKIP_COUNT` bigint DEFAULT NULL,
  `ROLLBACK_COUNT` bigint DEFAULT NULL,
  `EXIT_CODE` varchar(2500) DEFAULT NULL,
  `EXIT_MESSAGE` varchar(2500) DEFAULT NULL,
  `LAST_UPDATED` datetime(6) DEFAULT NULL,
  PRIMARY KEY (`STEP_EXECUTION_ID`),
  KEY `JOB_EXEC_STEP_FK` (`JOB_EXECUTION_ID`),
  CONSTRAINT `JOB_EXEC_STEP_FK` FOREIGN KEY (`JOB_EXECUTION_ID`) REFERENCES `BATCH_JOB_EXECUTION` (`JOB_EXECUTION_ID`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci

CREATE TABLE `BATCH_STEP_EXECUTION_CONTEXT` (
  `STEP_EXECUTION_ID` bigint NOT NULL,
  `SHORT_CONTEXT` varchar(2500) NOT NULL,
  `SERIALIZED_CONTEXT` text,
  PRIMARY KEY (`STEP_EXECUTION_ID`),
  CONSTRAINT `STEP_EXEC_CTX_FK` FOREIGN KEY (`STEP_EXECUTION_ID`) REFERENCES `BATCH_STEP_EXECUTION` (`STEP_EXECUTION_ID`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci

CREATE TABLE `BATCH_STEP_EXECUTION_SEQ` (
  `ID` bigint NOT NULL,
  `UNIQUE_KEY` char(1) NOT NULL,
  UNIQUE KEY `UNIQUE_KEY_UN` (`UNIQUE_KEY`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci
```

1. meta table과 서비스 database의 분리

@Primary 어노테이션을 이용하여 메타 테이블을 저장할 데이터베이스를 분리한다.

```java
package store.ckin.batch.config;

import lombok.RequiredArgsConstructor;
import org.apache.commons.dbcp2.BasicDataSource;
import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import store.ckin.batch.keymanager.KeyManager;

import javax.sql.DataSource;

/**
 * DataSourceConfig
 *
 * @author : 이가은
 * @version : 2024. 02. 20
 */
@Configuration
@RequiredArgsConstructor
public class DataSourceConfig {

    private final ApplicationContext applicationContext;
    private final DbProperties dbProperties;
    private final KeyManager keyManager;

    @Bean(name = "dataSource")
    public DataSource dataSource() {
        return getDataSource(keyManager.keyStore(dbProperties.getUrl()));
    }

    @Primary
    @Bean(name = "defaultDataSource")
    public DataSource defaultDataSource() {
        return getDataSource("jdbc:mysql://133.186.241.167:3306/ckin_coupon_batch");
    }

    private DataSource getDataSource(String url) {
        BasicDataSource basicDataSource = new BasicDataSource();

        basicDataSource.setDriverClassName(keyManager.keyStore(dbProperties.getDriver()));
        basicDataSource.setUrl(url);
        basicDataSource.setUsername(keyManager.keyStore(dbProperties.getUserName()));
        basicDataSource.setPassword(keyManager.keyStore(dbProperties.getPassword()));

        basicDataSource.setInitialSize(dbProperties.getInitialSize());
        basicDataSource.setMaxTotal(dbProperties.getMaxTotal());
        basicDataSource.setMaxIdle(dbProperties.getMaxIdle());
        basicDataSource.setMinIdle(dbProperties.getMinIdle());

        basicDataSource.setTestOnBorrow(true);
        basicDataSource.setValidationQuery("SELECT 1");

        basicDataSource.setMaxWaitMillis(dbProperties.getMaxWaitMillis());
        return basicDataSource;
    }

    @Bean
    public SqlSessionFactory sqlSessionFactory(@Qualifier("dataSource") DataSource dataSource) throws Exception {
        SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
        sqlSessionFactoryBean.setDataSource(dataSource);
        sqlSessionFactoryBean.setMapperLocations(applicationContext.getResources("classpath:mappers/*.xml"));

        return sqlSessionFactoryBean.getObject();
    }
}

```

### Chunk 프로세싱

    100개의 로우 데이터를 한 번에 처리하는 것은 가능하지만, 처리해야할 데이터의 개수가 100만 개라고 했을때는 어떨까? 
    100만 개의 데이터를 한 번에 처리하는 것은 불가능하다.  (여러 문제; 커넥션이 길어지고 메모리에 모든 데이터를 올려야 하는 문제 등)
    그렇기 때문에 큰 데이터의 경우, chunk단위로 데이터를 분할해서 처리하는 것이 바람직하다. 
    100만 개의 데이터를 1,000개씩 잘라서 처리하는 것이 청크 프로세싱이라고 할 수 있다. 

![Untitled](https://github.com/nhnacademy-be4-ckin/study-materials/assets/93987703/81d6758f-8d24-4f4a-9fca-0f2dac680175)

![img1 daumcdn](https://github.com/nhnacademy-be4-ckin/study-materials/assets/93987703/03f66aed-3245-41f7-bd29-7a3c0e9da59f)

    청크 단위로 읽고, 처리하고, 쓰기를 반복한다. 이는 하나의 트랜잭션으로 구성됨.

### **Batch vs Scheduler의 차이**

: 배치와 스케쥴러는 다른 개념이다. 

- **배치(Batch) : 일괄(대량)처리**사용자와 상호작용 없이 `여러 개의 작업`을 미리 정해진 순서에 따라 중단없이 처리하는 것> Spring Batch. 대량의 유저회원들에게 알림 메시지 보내기
- **스케쥴러(Scheduler)**
    
    `특정한 시간에` 등록한 작업을 자동으로 실행시키는 것
    
    - > Spring Scheduler, Quarts 등
    
    ex. 새벽 12시에 쿠폰 만료(expired)시간 체크하기
    
- **스케쥴러를 사용하여 배치의 작업을 수행! (주기적인 잡 실행)**
    
    

### Spring Scheduler

- **사용법**
    - Spring Boot Starter에 기본 제공됨 (별도의 의존성 추가 필요 없음)
    - 사용하기
        - main 함수가 포함된 클래스에 `@EnableScheduling` 어노테이션 붙인다.
            - 예시
                
                ```java
                @EnableScheduling
                @SpringBootApplication
                public class Application() {
                    public static void main(String[] args) {
                        SpringApplication.run(Application.class, args);
                    }
                }
                ```
                
        - 스케줄링을 원하는 메서드에 `@Scheduled`어노테이션을 붙여주면 된다. 스케줄링을 할 메서드는 아래 두 개의 조건을 만족해야 한다
            1. return type이 void일 것
            2. parameter가 없을 것
            - 예시
                
                ```java
                @Component
                public class BirthScheduler {
                    @Autowired
                    private JobLauncher jobLauncher;
                    @Autowired
                    private CouponBatchConfig couponBatchConfig;
                
                    @Scheduled(cron = "15 * * * * *")
                    public void birthJob() {
                        Map<String, JobParameter> confMap = new HashMap<>();
                
                        Date date = Calendar.getInstance().getTime();
                        confMap.put("time", new JobParameter(date));
                        JobParameters jobParameters = new JobParameters(confMap);
                
                        try {
                            jobLauncher.run(couponBatchConfig.giveBirthCoupon(), jobParameters);
                            System.out.println("run!!!");
                        } catch (Exception e) {
                            e.printStackTrace();
                            SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
                            System.out.println(String.format("ERRER TIME : %s", format.format(date) ));
                        }
                    }
                }
                
                ```
                
- **동작 방식**
    - 기본적으로 thread 1개를 이용하여 동기 방식으로 실행
        - 1번 스케줄이 끝나지 않으면 2번 스케줄 시작 시간이 되었다고 해도 시작되지 않음
        - 비동기 방식으로 실행하고 싶으면 `@EnableAsync` 어노테이션을 이용
    - cron표현식으로 주기적인 실행
      
        ![img1 daumcdn](https://github.com/nhnacademy-be4-ckin/study-materials/assets/93987703/ae48d8da-0297-4ecd-bfa0-e6445dcd7dc1)
        

## MyBaits

1) MyBatis와 JPA 차이

    데이터베이스 접속을 편하게 사용하기 위해 SQL Mapper 기술과, ORM(Object Relational Mapping) 기술을 제공한다. 둘 다 DB와의 연동, 저장을 위한 기술이며, SQL Mapper는 ‘**개발자가 작성한 SQL 실행 결과를 객체에 매핑**’시켜주는 프레임워크이며, ORM은 객체와 DB의 데이터를 ‘**자동으로 매핑**’시켜주는 프레임워크를 말한다.
    **SQL Mapper 기술을 제공하는 것이 ‘MyBatis’이며, ORM 기술을 제공하는 것이 ‘JPA(Java Persistence [Api](https://www.elancer.co.kr/blog/view?seq=199))’이다.** 두 가지 기술은 모두 데이터를 관계형 데이터베이스에 저장, 즉 영속화 (Persistence) 시킨다는 측면에서는 동일하지만, 서로 다른 접근 방식을 채택하고 있다.

- MyBatis는 SQL 문을 JAVA와 분리하여 별도 파일로 관리할 수 있어서 개발과 유지 보수에 용이하고, JPA는 SQL문을 아예 만들 필요가 없기 때문에 더욱 자동화되고 반복작업을 더욱 줄여준다.

## 중요한 것은 예외 처리!

스프링 배치 애플리케이션에서 시작할 때와 처리 중에 또는 결과를 기록할 때 문제가 발생할 수 있다. 

1. 레코드 건너뛰기
    1. 입력에서 레코드를 읽는 중에 에러가 발생하면 몇 가지 선택지 중에서 한 가지는 예외를 던져 처리를 멈추는 것이다. 그러나 얼마나 많은 레코드를 처리해야 하는가와 에러가 발생한 레코드 한 개를 처리하지 않았을 때의 영향도에 따라 에러를 던져 처리를 멈추는 것은 극단적인 방법일 수 있다. 
    2. 스프링 배치는 그 대신 특정 예외가 발생했을 때 레코드를 건너뛰는 skip기능을 제공한다. 
    3. 그렇다면 레코드를 건너뛰고자 할 때에는 어떤 예외/조건에서 건너뛸 것인지, 실패했다고 결정하기 전에 얼마나 많은 레코드를 건너뛸 수 있게 할 것인가에 대해 고려해야 한다. 

<aside>
💡 예를 들어, 백만 건의 렠코드 중  한 두건만 건너뛴다면 대수롭지 않은 일일 수도 있다. 그러나 백만 건 중 오십만 건을 건너뛴다면 뭔가 잘못된 것이다

</aside>

```java
@Bean
public Step copyFileStep() {
	return this.stepBuilderFactory.get("copyFileStep")
	.<Customer, Customer>chunk(10)
	.reader(customItemReader())
	.writer(itemWriter())
	.faultTolerant() //fault를 허용
	.skip(ParseException.class) //ParseException예외를
	.skipLimit(10) // 10번까지는 건너뛰도록
	.noSkip(IllegalArgumentException.class) //건너뛰지 말아야 할 대상 지정
	.build();
```

1. 잘못된 레코드 로그 남기기
    1. 문제가 있는 레코드를 건너뛰는 것은 유용한 방법이긴 하지만, 건너뛰는 것 자체가 문제가 될 수도 있다. 예를 들어 거래 내역을 처리하는 레코드라면 단순히 건너뛰는 것은 제대로 된 해결책이 아닐 것이다. 
    2. 이럴 때에는 에러를 일으킨 레코드의 로그를 남기는 것이 도움이 된다. 
    3. ItemReaderListener를 이용하여 로그를 남길 수 있다. 이 리스너는 beforeRead, afterRead, onReadError와 같은 메서드 세 개로 구성되어 있다. 
    4. 잘못된 레코드를 읽었을 때 로그를 남기려면 ItemListenerSupport를 사용하고 onReadError 메서드를 오버라이드해서 발생한 에러를 기록한다.

```java
public CustomerItemListener{
	
    private static final Log logger = LogFactory.getLog(CustomerItemListener.class);
    
    @OnReadError
    public void onReadError(Exception e){
    	if(e instanceof FlatFileParseException){
        	FlatFileParseException ffpe = (FlatFileParseException) e;
            StringBuilder errorMessage = new StringBuilder();
            errorMessage.append("An error occured while processing the " + 
            	ffpe.getLineNumber() +
                " line of the file.Below was the faulty " + 
                "input.\n");
            errorMessage.append(ffpe.getInput() + "\n");
            
            logger.error(errorMessage.toString(), ffpe);            
        }else{
        	logger.error("An error has occurred", e);            
        }
    }
	
}
```

리스너를 구성하려면 파일을 조회하는 스텝을 수정해야 한다. 

```java
@Bean
public CustomerItemListener customerListener(){
	return new CustomerItemListener();
}

@Bean
public Step copyFileStep(){
	return this.stepBuilderFactory.get("copyFileStep")
    		.<Customer, Customer> chunk(10)
            .reader(customerItemReader())
            .writer(itemWriter())
            .faultTolerant()
            .skipLimit(100)
            .skip(Exception.class)
            .listener(customerListener())
            .build();
}
```

1. 입력이 없을 때의 처리
    1. SQL쿼리가 빈 결과를 반환하는 것은 흔한 일이다. 빈 파일이 존재할 때도 많다. 그러나 이런 상황은 조회를 시도할 때 보통 정상 처리한다. 
    2. 입력을 읽지 못했을 때 스텝을 실패로 처리하거나 이메일을 보내는 것 같은 다른 처리를 하려면 StepListener를 사용한다. 
    3. 다음 예제의 StepListener의 @AfterStep메서드를 사용해서 조회한 레코드 수를 확인한 뒤에 레코드 수에 따라 적절히 처리한다. 

```java
public class EmptyInputStepFailer{
	
    @AfterStep
    public ExitStatus afterStep(StepExecuttion execution){
    	if(execution.getReadCount() > 0){
        	return execution.getExitStatus();
        }else{
        	return ExitStatus.FAILED;
        }
    }
    
}
```

```java
@Bean
public EmptyInputStepFailer emptyFileFailer(){
	return new EmptyInputStepFailer();
}

@Bean
public Step copyFileStep(){
	return this.stepBuilderFactory.get("copyFileStep")
    .<Customer, Customer>chunk(10)
    .reader(customerFileReader(null))
    .writer(outputWriter(null))
    .listener(emptyFileFailer())
    .build();
}
```

## 성능 테스트

- **1000건**
    - primary key가 있는 경우
        
        ![스크린샷 2024-03-04 오후 1 20 16](https://github.com/nhnacademy-be4-ckin/study-materials/assets/93987703/22b798cd-37c0-4720-8d15-7cd11c6cd184)

    - primary key가 없는 경우
        
        ![스크린샷 2024-03-04 오후 1 22 45](https://github.com/nhnacademy-be4-ckin/study-materials/assets/93987703/9afc0809-6099-42b2-b8e3-5f644b9695e2)

- **10000건**
    - primary key가 있는 경우
        
        ![스크린샷 2024-03-04 오후 2 07 28](https://github.com/nhnacademy-be4-ckin/study-materials/assets/93987703/03e3deee-8e2a-46a0-95fb-d48f9f58f1e8)

    - primary key가 없는 경우
        
        ![스크린샷 2024-03-04 오후 2 01 42](https://github.com/nhnacademy-be4-ckin/study-materials/assets/93987703/da109f62-cbef-419e-8c05-efc2fb8e1643)

- **100000건**
    - 결과
        
        ![스크린샷 2024-03-05 오전 12 44 17](https://github.com/nhnacademy-be4-ckin/study-materials/assets/93987703/3fcc1c4f-9915-4f5a-8570-87d8dd23c9e6)


---

💡PK유무에 따른 성능 차이

→ 왜 차이가 별로 안 나지? 제대로 테스트가 안 됐나?

- 차이가 나지 않는 이유는 테스트 값이 적기 때문! 테스트 크기가 100만건, 1000만건으로 늘어난다면 PK가 없는 테이블이 유리하다.

💡단일 insert vs bulk insert

- 이 부분에서는 mybatis의 foreach문을 활용한 Bulk Insert가 무려 1082배의 성능 차이로 우세했다.
