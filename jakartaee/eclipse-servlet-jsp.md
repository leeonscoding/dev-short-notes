# Jakarta EE Servlet and JSP project using Eclipse IDE

## Tools
Install those tools in the machine
* Java 17
* Apache maven
* Eclipse IDE for Enterprise Java web development
* Jakarta EE 10
* Servlet
* JSP
* Tomcat Server(10)

## Initialize the project
* Start the Eclipse IDE
* Go to File menu. Select File->New->Dynamic Web Project
* Select Project Name, Project location
* Select Target runtime, I'm using Apache tomcat
* Click Next and keep the default source folders for build paths and default output folder
* Click Next and Select Context root(I've changed this) and Content directory. I don't want web.xml so I unchecked this opton
* Click Finish

## Add Maven
* Right click in the project from the Project Explorer
* Go to Configure and click 'Convert to Maven Project'
* A new Popup will appear. Modify Version and add Name and description and click Finish
* The pom.xml will be created and maven will build the project

## Crate a new JSP file
* Right click in the project from the Project Explorer
* Select New->JSP file
* Select src > main > webapp and give a File name. I've given the index.jsp
* Modify code
    ```html
    <%@ page language="java" contentType="text/html; charset=ISO-8859-1"
    pageEncoding="ISO-8859-1"%>
    <!DOCTYPE html>
    <html>
    <head>
    <meta charset="ISO-8859-1">
    <title>Insert title here</title>
    </head>
    <body>
        <h1>This is my first JSP page</h1>
        <form action="HelloServlet" method="post">
            <input type="text" name="myName">
            <button type="submit">Submit</button>
        </form>
    </body>
    </html>
    ```

## Create a new Servlet file
* Right click in the project from the Project Explorer
* Select New->Servlet
* A new popup will be appear. Write the 'Java package' and 'Class name'. I've choose 'com.leeonscoding' and 'HelloServlet'
* Click Next and I've keep the default 'Initialization parameters' and 'URL mappings'
* Click Next then select
    * Modifiers: public
    * Uncheck: Constructors from superclass
    * Check: doPost in 'Inherited abstract methods'
* Click Finish
* _Note: The URL Mapping between JSP post action and the Servlet should be same_
* Modify the code
    ```java
    @WebServlet("/HelloServlet")
    public class HelloServlet extends HttpServlet {
        private static final long serialVersionUID = 1L;

        
        protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
            String name = request.getParameter("myName");
            
            PrintWriter pw = response.getWriter();
            pw.println("<h1>"+ name +"</h1>");
            pw.close();
        }

    }
    ```
* Here I've received the input value from the index.jsp form and print it.

## Run the application
* Right click in the project from the Project Explorer
* Select Run As->Run on Server
* Select 'Choose an existing server'
* Go to localhost->Tomcat(what is installed in your machine)
* Click Finish
* Automatically the default browser will be opened with the application and the index.jsp will render
