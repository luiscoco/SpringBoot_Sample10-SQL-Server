# Spring Boot + SQL Server example: CRUD Operations Rest API

For more detail, please visit:
> [Spring Boot CRUD Operations example with SQL Server](https://www.bezkoder.com/spring-boot-sql-server/)

## 1. Run a SQL Server docker container

It is mandatory to **run Docker Desktop** before pulling and running the SSMS docker container

We open a command prompt window and run the following command

```
docker run ^
  -e "ACCEPT_EULA=Y" ^
  -e "MSSQL_SA_PASSWORD=Luiscoco123456" ^
  -p 1433:1433 ^
  -d mcr.microsoft.com/mssql/server:2022-latest
```

## 2. Connect to the SQL Server container from SSMS

Download and install SQL Server Management Studio (SSMS) from this site: https://learn.microsoft.com/es-es/sql/ssms/download-sql-server-management-studio-ssms

![image](https://github.com/luiscoco/spring-boot-sql-server-master/assets/32194879/48ac3b8a-9a95-41b3-a446-942f73739d53)

We create the database **bezkoder_db** and the table **tutorials**

```sql
CREATE DATABASE bezkoder_db
GO

CREATE TABLE tutorials (
  id BIGINT IDENTITY(1,1) PRIMARY KEY,
  title NVARCHAR(255),
  description NVARCHAR(MAX),
  published BIT
);
```

## 3. Source Code explanation

### 3.1. Project folders and files structure

![image](https://github.com/luiscoco/spring-boot-sql-server-master/assets/32194879/653978f9-8a13-424c-b77d-416043bcc6b9)

### 3.2. Project dependencies

These are the project dependencies:

- spring-boot-starter-actuator
- **spring-boot-starter-data-jpa**
- spring-boot-starter-web
- **mssql-jdbc**
- **springdoc-openapi-starter-webmvc-ui**
- spring-boot-starter-test

**pom.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>3.1.5</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.bezkoder</groupId>
	<artifactId>spring-boot-sql-server</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>spring-boot-sql-server</name>
	<description>Spring Boot + SQL Server (MSSQL) example with JPA</description>
	<properties>
		<java.version>21</java.version>
	</properties>
	<dependencies>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-jpa</artifactId>
		</dependency>
		
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>com.microsoft.sqlserver</groupId>
			<artifactId>mssql-jdbc</artifactId>
			<scope>runtime</scope>
		</dependency>

		<dependency>
			<groupId>org.springdoc</groupId>
			<artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
			<version>2.0.3</version>
		</dependency>
		
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>
```

### 3.3. Application Main entry point

**SpringBootSqlServerApplication.java**

```java
package com.bezkoder.spring.mssql;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringBootSqlServerApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringBootSqlServerApplication.class, args);
	}

}
```

### 3.4. Data Model

**Tutorial.java**

```java
package com.bezkoder.spring.mssql.model;

import jakarta.persistence.*;

@Entity
@Table(name = "tutorials")
public class Tutorial {

  @Id
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  private long id;

  @Column(name = "title")
  private String title;

  @Column(name = "description")
  private String description;

  @Column(name = "published")
  private boolean published;

  public Tutorial() {

  }

  public Tutorial(String title, String description, boolean published) {
    this.title = title;
    this.description = description;
    this.published = published;
  }

  public long getId() {
    return id;
  }

  public String getTitle() {
    return title;
  }

  public void setTitle(String title) {
    this.title = title;
  }

  public String getDescription() {
    return description;
  }

  public void setDescription(String description) {
    this.description = description;
  }

  public boolean isPublished() {
    return published;
  }

  public void setPublished(boolean isPublished) {
    this.published = isPublished;
  }

  @Override
  public String toString() {
    return "Tutorial [id=" + id + ", title=" + title + ", desc=" + description + ", published=" + published + "]";
  }

}
```

### 3.5. Repository

**TutorialRepository.java**

```java
package com.bezkoder.spring.mssql.repository;

import java.util.List;

import org.springframework.data.jpa.repository.JpaRepository;

import com.bezkoder.spring.mssql.model.Tutorial;

public interface TutorialRepository extends JpaRepository<Tutorial, Long> {
  List<Tutorial> findByPublished(boolean published);
  List<Tutorial> findByTitleContaining(String title);
}
```

### 3.6. Controller

We also configure the Swagger Open API docs for each action inside the controller

**TutorialController**

```java
package com.bezkoder.spring.mssql.controller;

import java.util.ArrayList;
import java.util.List;
import java.util.Optional;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.PutMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import com.bezkoder.spring.mssql.model.Tutorial;
import com.bezkoder.spring.mssql.repository.TutorialRepository;

import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.Parameter;
import io.swagger.v3.oas.annotations.media.Content;
import io.swagger.v3.oas.annotations.media.Schema;
import io.swagger.v3.oas.annotations.responses.ApiResponse;
import io.swagger.v3.oas.annotations.tags.Tag;

// @CrossOrigin(origins = "http://localhost:8081")
@RestController
@RequestMapping("/api")
@Tag(name = "Tutorial", description = "The Tutorial API")
public class TutorialController {

	@Autowired
	TutorialRepository tutorialRepository;

	@Operation(summary = "Test the API", description = "Test endpoint to verify the API is working", responses = {
		@ApiResponse(description = "Success", responseCode = "200", content = @Content(mediaType = "text/plain")) })
	@GetMapping("/test")
	public ResponseEntity<String> test() {
    	return ResponseEntity.ok("Test endpoint response");	
	}

	@Operation(summary = "Get all tutorials", description = "Retrieve all tutorials or filter by title", responses = {
		@ApiResponse(description = "Successful Operation", responseCode = "200", content = @Content(mediaType = "application/json", schema = @Schema(implementation = Tutorial.class))),
		@ApiResponse(description = "Not Found", responseCode = "404") })
	@GetMapping("/tutorials")
	public ResponseEntity<List<Tutorial>> getAllTutorials(@RequestParam(required = false) String title) {
		try {
			List<Tutorial> tutorials = new ArrayList<Tutorial>();
			if (title == null) {
				tutorialRepository.findAll().forEach(tutorials::add);
			} else {
				tutorialRepository.findByTitleContaining(title).forEach(tutorials::add);
			}
			
			if (tutorials.isEmpty()) {
				// Add logging here
				System.out.println("No tutorials found");
				return new ResponseEntity<>(HttpStatus.NO_CONTENT);
			}
			
			// Additional logging here if needed
			System.out.println("Found tutorials: " + tutorials.size());
			return new ResponseEntity<>(tutorials, HttpStatus.OK);
		} catch (Exception e) {
			// Log the exception details here
			e.printStackTrace();
			return new ResponseEntity<>(null, HttpStatus.INTERNAL_SERVER_ERROR);
		}
	}

	@Operation(summary = "Get a tutorial by ID", description = "Retrieve a single tutorial by its ID", responses = {
		@ApiResponse(description = "Found", responseCode = "200", content = @Content(mediaType = "application/json", schema = @Schema(implementation = Tutorial.class))),
		@ApiResponse(description = "Not Found", responseCode = "404") })
	@GetMapping("/tutorials/{id}")
	public ResponseEntity<Tutorial> getTutorialById(@PathVariable("id") long id) {
		Optional<Tutorial> tutorialData = tutorialRepository.findById(id);

		if (tutorialData.isPresent()) {
			return new ResponseEntity<>(tutorialData.get(), HttpStatus.OK);
		} else {
			return new ResponseEntity<>(HttpStatus.NOT_FOUND);
		}
	}

	@Operation(summary = "Create a new tutorial", description = "Add a new tutorial to the database", responses = {
		@ApiResponse(description = "Created", responseCode = "201", content = @Content(mediaType = "application/json", schema = @Schema(implementation = Tutorial.class))),
		@ApiResponse(description = "Internal Server Error", responseCode = "500") })
	@PostMapping("/tutorials")
	public ResponseEntity<Tutorial> createTutorial(@RequestBody Tutorial tutorial) {
		try {
			Tutorial _tutorial = tutorialRepository
					.save(new Tutorial(tutorial.getTitle(), tutorial.getDescription(), false));
			return new ResponseEntity<>(_tutorial, HttpStatus.CREATED);
		} catch (Exception e) {
			return new ResponseEntity<>(null, HttpStatus.INTERNAL_SERVER_ERROR);
		}
	}

	@Operation(summary = "Update a tutorial", description = "Update an existing tutorial by ID", responses = {
		@ApiResponse(description = "Successful update", responseCode = "200", content = @Content(schema = @Schema(implementation = Tutorial.class))),
		@ApiResponse(description = "Not found", responseCode = "404")
})
	@PutMapping("/tutorials/{id}")
	public ResponseEntity<Tutorial> updateTutorial(@PathVariable("id") long id, @RequestBody Tutorial tutorial) {
		Optional<Tutorial> tutorialData = tutorialRepository.findById(id);

		if (tutorialData.isPresent()) {
			Tutorial _tutorial = tutorialData.get();
			_tutorial.setTitle(tutorial.getTitle());
			_tutorial.setDescription(tutorial.getDescription());
			_tutorial.setPublished(tutorial.isPublished());
			return new ResponseEntity<>(tutorialRepository.save(_tutorial), HttpStatus.OK);
		} else {
			return new ResponseEntity<>(HttpStatus.NOT_FOUND);
		}
	}

	@Operation(summary = "Delete a tutorial", description = "Delete a tutorial by ID", responses = {
		@ApiResponse(description = "Successful deletion", responseCode = "204"),
		@ApiResponse(description = "Internal server error", responseCode = "500")
})
	@DeleteMapping("/tutorials/{id}")
	public ResponseEntity<HttpStatus> deleteTutorial(@PathVariable("id") long id) {
		try {
			tutorialRepository.deleteById(id);
			return new ResponseEntity<>(HttpStatus.NO_CONTENT);
		} catch (Exception e) {
			return new ResponseEntity<>(HttpStatus.INTERNAL_SERVER_ERROR);
		}
	}

	@Operation(summary = "Delete all tutorials", description = "Delete all tutorials from the database", responses = {
		@ApiResponse(description = "Successful deletion", responseCode = "204"),
		@ApiResponse(description = "Internal server error", responseCode = "500")
})
	@DeleteMapping("/tutorials")
	public ResponseEntity<HttpStatus> deleteAllTutorials() {
		try {
			tutorialRepository.deleteAll();
			return new ResponseEntity<>(HttpStatus.NO_CONTENT);
		} catch (Exception e) {
			return new ResponseEntity<>(HttpStatus.INTERNAL_SERVER_ERROR);
		}

	}
	
	@GetMapping("/tutorials/published")
	public ResponseEntity<List<Tutorial>> findByPublished() {
		try {
			List<Tutorial> tutorials = tutorialRepository.findByPublished(true);

			if (tutorials.isEmpty()) {
				return new ResponseEntity<>(HttpStatus.NO_CONTENT);
			}
			return new ResponseEntity<>(tutorials, HttpStatus.OK);
		} catch (Exception e) {
			return new ResponseEntity<>(HttpStatus.INTERNAL_SERVER_ERROR);
		}
	}

}
```

### 3.7. application.properties

We configure the database connection string and Swagger options:

```
spring.datasource.url= jdbc:sqlserver://localhost:1433;encrypt=true;trustServerCertificate=true;databaseName=bezkoder_db
spring.datasource.username= sa
spring.datasource.password= Luiscoco123456

spring.jpa.properties.hibernate.dialect= org.hibernate.dialect.SQLServerDialect
spring.jpa.hibernate.ddl-auto= update

#springdoc.api-docs.enabled=false
#springdoc.swagger-ui.enabled=false
#springdoc.packages-to-scan=com.bezkoder.spring.swagger.controller
springdoc.swagger-ui.path=/bezkoder-documentation
springdoc.api-docs.path=/bezkoder-api-docs
#springdoc.swagger-ui.operationsSorter=method
#springdoc.swagger-ui.tagsSorter=alpha
springdoc.swagger-ui.tryItOutEnabled=true
springdoc.swagger-ui.filter=true

bezkoder.openapi.dev-url=http://localhost:8080
bezkoder.openapi.prod-url=https://bezkoder-api.com
```

### 3.8. Swagger configuration

**OpenAPIConfig.java**

```java
package com.bezkoder.spring.mssql.config;

import java.util.List;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import io.swagger.v3.oas.models.OpenAPI;
import io.swagger.v3.oas.models.info.Contact;
import io.swagger.v3.oas.models.info.Info;
import io.swagger.v3.oas.models.info.License;
import io.swagger.v3.oas.models.servers.Server;

@Configuration
public class OpenAPIConfig {

  @Value("${bezkoder.openapi.dev-url}")
  private String devUrl;

  @Value("${bezkoder.openapi.prod-url}")
  private String prodUrl;

  @Bean
  public OpenAPI myOpenAPI() {
    Server devServer = new Server();
    devServer.setUrl(devUrl);
    devServer.setDescription("Server URL in Development environment");

    Server prodServer = new Server();
    prodServer.setUrl(prodUrl);
    prodServer.setDescription("Server URL in Production environment");

    Contact contact = new Contact();
    contact.setEmail("bezkoder@gmail.com");
    contact.setName("BezKoder");
    contact.setUrl("https://www.bezkoder.com");

    License mitLicense = new License().name("MIT License").url("https://choosealicense.com/licenses/mit/");

    Info info = new Info()
        .title("Tutorial Management API")
        .version("1.0")
        .contact(contact)
        .description("This API exposes endpoints to manage tutorials.").termsOfService("https://www.bezkoder.com/terms")
        .license(mitLicense);

    return new OpenAPI().info(info).servers(List.of(devServer, prodServer));
  }
}
```

## 4. Run Spring Boot application

We run the application in VSCode with the following command

```
mvn spring-boot:run
```

These are the application endpoints defined in the Controller

Methods	Urls	           Actions

POST	**/api/tutorials** create new Tutorial

GET	**/api/tutorials** retrieve all Tutorials

GET	**/api/tutorials/:id** retrieve a Tutorial by :id

PUT	**/api/tutorials/:id** update a Tutorial by :id

DELETE	**/api/tutorials/:id** delete a Tutorial by :id

DELETE	**/api/tutorials** delete all Tutorials

GET	**/api/tutorials/published** find all published Tutorials

GET	**/api/tutorials?title=[keyword]** find all Tutorials which title contains keyword

We navigate to the Swagger OpenAPI documentation: http://localhost:8080/swagger-ui/index.html

![image](https://github.com/luiscoco/spring-boot-sql-server-master/assets/32194879/be4f0eb0-2ce8-40f4-9546-16ded7f8b502)

We can sent a GET request to see all the tutorials stored in the SQL Sever database

![image](https://github.com/luiscoco/spring-boot-sql-server-master/assets/32194879/6b65f9b8-d201-4b67-a177-07d5e42c3ccd)

We confirm with the information obtained from the database

![image](https://github.com/luiscoco/spring-boot-sql-server-master/assets/32194879/866d38b8-9ccf-454a-b909-24001025a96a)
