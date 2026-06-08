
## What Is Spring?

Spring framework provides infrastructure support for developing Java applications.

Spring Boot is basically an extension of the Spring framework, which eliminates the boilerplate configurations required for setting up a Spring application.

It takes an opinionated view of the Spring platform, which paves the way for a faster and more efficient development ecosystem.

SB already has Tomcat server (no need to install)

@SpringBootApplication - annotation on main class. This replaces the need for setting up a manual Spring application context

SpringApplication.run()- start application (SB configures the embedded web server and other things, no need to do them manually)

Bean- reusable object (once created, can be used anywhere in codebase)



![[2026-06-05_18-02-28.png]]

![[2026-06-05_18-02-58.png]]

![[2026-06-05_18-03-17.png]]

![[2026-06-05_18-08-35.png]]


Spring web for APIs (no ext tomcat server required)

Maven is a course for dependencies. In maven repo, we can get the pom.xml code for the dependency
Maven is also a build automation tool 

![[2026-06-05_18-14-31.png]]

Maven wrapper is provided by SB btw (mvnw)

![[2026-06-05_18-16-19.png]]

Maven puts dependencies under the .m2 folder, so they can be reused when needed

pom.xml

![[2026-06-05_18-20-30.png]]

for inheriting some starter dependencies

![[2026-06-05_18-22-13.png]]

for building the project

.jar vs .jar.original

.jar -> has the dependencies and code (self contained jar, fat jar)
.jar.original -> only code

These files are created due to the build plugin

## Maven in SB:

build automation tool
manages dependencies

maven repository maintains jar libraries (dependencies) 
you can type "maven library-name" and get the dependency code, which u put in pom.xml

to use maven you can:
1. install it
2. use the maven wrapper (mvnw) 

you can do several things with maven like: test, package the app into a .jar, validate, 'install' your package into .m2 etc

to run a .jar: java -jar filename.jar 

maven stores the .jar files in .m2 repository. If a library is required and its avaliable in .m2, maven gets the library from there 

the jar package made is stored in target directory 
the jar is self-contained, it contains all required dependencies and everything else needed to run the program
to run: java -jar package.jar 

the .original file just has the compiled code 


## Springboot internal working



1) **IOC (inversion of control)**: It is possible to let SB handle object creation 
when we wanted to create object A, we did A a=new A() but we can make SB do this, hence we are inverting the control to SB. Object creation is being externalised to SB 

![[2026-06-05_18-31-06.png]]

in SB, we would actually go to an IOC container (think Spring itself) and ask to create objects 

IOC container is a container that contains a list of our classes. When we want something, we can just ask the container and have it supply the object to us

**ApplicationContext** is a way to **achieve IOC container** 

IOC container scans through a list of class (we supply the package name) 
it wont keep every class! only the ones which have @Component annotation 

![[2026-06-05_18-33-44.png]]

@Component <- add this to register the class in IOC container
public class A{
	...
}

^^^ A is registered as a Spring bean

**Bean is just an object. Its a object in IOC container** 

2) **@SpringBootApplication**- what does it mean?

It defines the entry point of the app 
put on the main class (with main method)

Does the work of 3 annotations:

![[2026-06-05_18-37-18.png]]

-> **@Configuration**

used usually with @Bean  

Bean is made with RestController, Component 
we can also use Bean, but @Bean is only put on **functions** 

-> **@EnableAutoConfiguration**

Configuration is made automatic (duh)

suppose we want to use mongodb, we just put the dependency in pom.yaml and it just works 


-> **@ComponentsScan**- Spring scans the classes which have @Component and adds them 

base package of the project is: com.rawmeat.myproject (Basically where main class is)

ComponentsScan will treat com.rawmeat.myproject as the base package and ONLY look for classes inside this package.
IF you create a class outside com.rawmeat.myproject, it will not be scanned!

@RestController is also like @Component with some extra things 

@Autowired- dependency injection (in code, Car depends on Dog)

![[2026-06-05_18-40-40.png]]

note that we didn't do dog=new Dog() <-- when we did private Dog dog; Spring already had supplied the object as Dog is inside IOC container (has @Component) 

![[2026-06-05_18-41-20.png]]

this is technically a field injection (field = object variable)

without @Autowired, dog is null and it wont work!

advantage: if many classes want to use Dog, they can do so easily   


3) **Annotation**- can be attached to classes, functions, interfaces, etc. tells some information about the thing 


## REST API (Representational State transfer API)

![[2026-06-05_18-51-52.png]]

To communicate with netflix server at  172.17.18.19:8080, you need URL + HTTP verb

/netflix/plans is the API endpoint

![[2026-06-05_18-54-33.png]]

(project structure)

![[2026-06-05_18-55-50 1.png]]

Always add a health check!


![[2026-06-05_18-57-07.png]]

![[2026-06-05_18-59-19.png]]

@RequestMapping adds an annotation to the whole class.
If we added GetMapping("/abc") to getAll(), the function will live at /journal/abc

How to send a json Journal entry to java code? - @RequestBody

![[2026-06-05_19-02-04.png]]

![[2026-06-05_19-04-17.png]]

**** Notes:** 



Make the get/post functions public as Swing needs to access them

![[2026-06-05_19-07-24.png]]

name is a Request parameter

![[2026-06-05_19-07-57.png]]

'vip' is path variable

![[2026-06-05_19-09-17.png]]

How to capture path variable ^^^

![[2026-06-05_19-10-07.png]]


![[2026-06-05_19-10-17.png]]


![[2026-06-05_19-10-49.png]]

![[2026-06-05_19-12-55.png]]



