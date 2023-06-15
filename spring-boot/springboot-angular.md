# Integrate Angular in a Spring Boot Project

* Prerequisites
    * Java
    * Nodejs
    * npm
    * Angular cli

## Add the backend project
* Go to the start.spring.io [page](https://start.spring.io/ "spring boot project generator")
* Select
    * **Build Tool**: Gradle - Groovy
    * **Language**: Java
    * **Spring Boot Version**: 3.1.0
    * **Packaging**: Jar
    * **Java Version**: 17
* Fill the metadata part as your own
* Select Dependecies
    * **Spring WEB**: Contains Spring MVC and embedded Tomcat container
* Click the **GENERATE** Button
* Copy the compressed file to a desired location
* Unzip it
* Write a rest controller in a new java file. I've created the TicketController.java
* Add this code
    ```java
    @RestController
    @RequestMapping("/ticket")
    public class TicketController {

        @GetMapping
        public ResponseEntity<Ticket> getTicket() {
            String unique = UUID.randomUUID().toString();

            return new ResponseEntity<>(new Ticket(unique), HttpStatus.OK);
        }
    }
    ```
* Now run the application
    ```bash
    ./gradlew bootRun
    ```
* Check it in the browser. Enter http://localhost:8080/ticket
## Add the frontend part
* Navigate to the path 'src/main' using command line tool
* Run this command
    ```bash
    ng new frontent
    ```
    A new folder 'frontend' will be created. It contains the full angular project.
* Create a proxy file(proxy.conf.json) under 'src/main/frontend/src/'
* Add this code
    ```json
    {
        "/ticket": {
        "target": "http://localhost:8080",
        "secure": false
        }
    }
    ```
    Here, '/ticket' is the path of our api. And the target is the domain. The value(false) of the 'secure' means that SSL isn't enabled.
    Here, every request mapping is prefixed with '/ticket'. We have to only call this prefix '/ticket' with other urls.
* Go to the angular.json file and navigate projects -> architect -> serve. In server modify the "development" section. Add this line.
    ```json
       "proxyConfig": "src/proxy.conf.json" 
    ```
* Start the application
    ```bash
    ng serve
    ```
* Import the HttpClientModule in app.module.ts
    ```typescript
        imports: [
            BrowserModule,
            HttpClientModule
        ]
    ```
* Generate a service in command line
    ```bash
    ng g s ticket
    ```
* 2 new files will be created
    ```bash
    CREATE src/app/ticket.service.spec.ts (357 bytes)
    CREATE src/app/ticket.service.ts (135 bytes)
    ```
* Add the rest api calls in the ticket.service.ts class
    ```typescript
    import { Injectable } from '@angular/core';
    import { HttpClient } from '@angular/common/http';

    import {Ticket} from './ticket';
    import { Observable } from 'rxjs';

    @Injectable({
        providedIn: 'root'
    })
    export class TicketService {

        constructor(private http: HttpClient) { }

        getTicket() : Observable<Ticket> {
            return this.http.get<Ticket>('/ticket');
        }
    
    }
    ```
* Here I've used the Ticket interface as response object. This is the code of the interface in the Ticket.ts file.
    ```typescript
    export interface Ticket {
        readonly ticketNo: string;
    }
    ```
* Finally the wrap everything in the app.component.ts
    ```typescript
    import { Component } from '@angular/core';
    import { TicketService } from './ticket.service';

    @Component({
    selector: 'app-root',
        templateUrl: './app.component.html',
        styleUrls: ['./app.component.css']
    })
    export class AppComponent {

        ticketNo: string = 'click the button';

        constructor(private ticketService: TicketService) {}

        getTicket() {
            this.ticketService
                .getTicket()
                .subscribe(res => {
                this.ticketNo = res.ticketNo;
                })
        }
    }
    ```
* And the app.component.html
    ```html
    <h1>Your ticket is <span>{{ticketNo}}</span></h1>
    <button (click)="getTicket()">Click to generate</button>
    ```
## Building frontend while building the jar
* Go to the build.gradle file and create a task like below
    ```groovy
    task buildFrontend(type: Exec) {
        OperatingSystem os = DefaultNativePlatform.currentOperatingSystem;

        workingDir 'src/main/frontend'

        if(os.isWindows()) {
            commandLine 'cmd', '/c', 'ng build'
        } else {
            commandLine 'sh', '-c', 'ng build'
        }
    }
    ```
* Import the implementation DefaultNativePlatform for the OperatingSystem interface
    ```groovy
    import org.gradle.nativeplatform.platform.internal.DefaultNativePlatfo
    ```
* Copy the frontend assets to the static path
    ```groovy
    task copyFrontendFiles(type: Copy) {
        from 'src/main/frontend/dist/frontend'
        into 'src/main/resources/static'
    }
    ```
* Chain both tasks with bootJar
    ```groovy
    bootJar.configure {dependsOn(copyFrontendFiles)}
    copyFrontendFiles.configure {dependsOn(buildFrontend)}
    ```
* Still Spring boot willn't find the index.html. We need to resolve the view. For simplicity, we can add the template engine thymeleaf as a dependency. It will configure all the things by default.
    ```groovy
    implementation 'org.springframework.boot:spring-boot-starter-thymeleaf:3.1.0'
    ```
* Build the jar
    ```bash
    ./gradlew bootJar
    ```
* Run the jar
    ```bash
    java -jar spring-angular-1.0.jar
    ```
