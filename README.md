# Spring Boot + Thymeleaf + Tailwind CSS

This is an example of how to use Tailwind CSS in your next Spring Boot + Thymeleaf project. This should be similar for
whatever template engine you're using, I just happen to be using Thymeleaf here. 

You can just use the CDN if you're prototyping something out but if this is a real world project you don't want to do that. When you include the CDN you get all the Tailwind CSS utility classes even if you're not using them. 

```html
 <script src="https://cdn.tailwindcss.com"></script>
```

## Getting Started 

To follow along you will need to have node and npm installed. To check if they are installed and what version you're currently 
running you can run the following commands: 

```shell
node -v # v20.2.0
npm -v # 9.8.1
```

You can head over to https://start.spring.io and create a new project with the following dependenices or follow [this link](https://start.spring.io/#!type=maven-project&language=java&platformVersion=3.3.1&packaging=jar&jvmVersion=22&groupId=dev.danvega&artifactId=tailwind&name=tailwind&description=Demo%20project%20for%20Spring%20Boot&packageName=dev.danvega.tailwind&dependencies=web,thymeleaf,devtools).

- **Dependencies**
  - Spring Web
  - Thymeleaf 
  - Spring Boot Devtools

# Frontend 

Create a new directory `/src/main/frontend` and go into that directory using the command line. Run the following command to create a new 
`package.json` using the defaults:

```shell
npm init 
```

## Tailwind CSS

Now that you have a `package.json` you can run the following command to install Tailwind CSS as a dev dependency. The next command will create a new `tailwind.config.js`

```shell
npm install -D tailwindcss
npx tailwindcss init
```

Update the Tailwind config, so it knows where the templates are that will include Tailwind CSS styles.

```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: ["../resources/templates/**/*.{html,js}"],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

Add the `node_modules` folder to your `.gitignore` before you forget 🤦‍♂️

```gitignore
### Frontend ###

/src/main/frontend/node_modules/
```

Create a new file `/src/main/frontend/styles.css` and include the following: 

styles.css
```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

Create a script that will take the styles file you just created as input and send the build output to `/resources/static/main.css`: 

```json
{
  "name": "frontend",
  "version": "1.0.0",
  "description": "Spring Boot + Tailwind CSS Custom Login Form",
  "scripts": {
    "build": "tailwindcss -i ./styles.css -o ../resources/static/main.css",
    "watch": "tailwindcss --watch -i ./styles.css -o ../resources/static/main.css"
  },
  "author": "Dan Vega",
  "license": "ISC",
  "private": true,
  "devDependencies": {
    "tailwindcss": "^3.4.4"
  },
  "dependencies": {
    "@tailwindcss/forms": "^0.5.7"
  }
}
```

## Templates

Now that everything has been set up and configured you can create a new template `/src/main/resources/templates/index.html`. This Thymeleaf template will point to the `main.css` output that is now located in the static folder. This home page includes some basic Tailwind utility classes to ensure everything is working properly.

```html
<!DOCTYPE HTML>
<html
        lang="en"
        xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Home</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    <link th:href="@{/main.css}" rel="stylesheet" />
</head>
<body class="bg-slate-50">
    <header class="bg-white container mx-auto">
        <h1 class="text-4xl font-bold p-8">Home Page</h1>
    </header>
</body>
</html>
```

You will need to create a controller that has a mapping for the root and returns your new index template.

```java
@Controller
public class HomeController {

    @GetMapping("/")
    public String home() {
        return "index";
    }
    
}
```

You will need to run `npm run build` or `npm run watch` and then start the Spring Boot application. After running the npm command you should see a new file `main.css` in the `/src/main/resources/static` directory. This contains all the Tailwind CSS utility classes you are using instead of every single one the framework declares resulting in a much smaller size in production. 

## Maven Configuration 

What we did in the previous section is great for development but what happens when we want to build an artifact for another environment like production. In that scenario we need to automate this process so that the `npm run build` command is run and the output of the build creates a new `main.css` in the static folder in the event this wasn't done locally. 

For that we will be using the [frontend-maven-plugin](https://github.com/eirslett/frontend-maven-plugin). This plugin will download/install node and npm, run npm install or any other commands we need it to. First you will need to declare the versions of node and npm you wish to use. 

```xml
<properties>
    <java.version>22</java.version>
    <node.version>v20.2.0</node.version>
    <npm.version>6.14.3</npm.version>
</properties>
```

Next setup the plugin to install node and npm, run `npm install` and `npm run build`. The working directory in the configuration section is based on what we did in this tutorial. If your `package.json` is in a different directory please update that. 

```xml
<plugin>
  <groupId>com.github.eirslett</groupId>
  <artifactId>frontend-maven-plugin</artifactId>
  <version>1.15.0</version>
  <executions>

      <execution>
          <id>install node and npm</id>
          <goals>
              <goal>install-node-and-npm</goal>
          </goals>
          <phase>generate-resources</phase>
          <configuration>
              <nodeVersion>${node.version}</nodeVersion>
              <npmVersion>${npm.version}</npmVersion>
          </configuration>
      </execution>

      <execution>
          <id>npm install</id>
          <goals>
              <goal>npm</goal>
          </goals>
          <phase>generate-resources</phase>
          <configuration>
              <arguments>install</arguments>
          </configuration>
      </execution>

      <execution>
          <id>npm build</id>
          <goals>
              <goal>npm</goal>
          </goals>
          <phase>generate-resources</phase>
          <configuration>
              <arguments>run build</arguments>
          </configuration>
      </execution>

  </executions>
  <configuration>
      <nodeVersion>${node.version}</nodeVersion>
      <workingDirectory>src/main/frontend</workingDirectory>
      <installDirectory>target</installDirectory>
  </configuration>
</plugin>
```

To verify that this works go change something in `/src/main/resources/templates/index.html` like the background to `bg-blue-500`. You can now run `./mvnw clean package` to create an executable JAR. If that runs successfully you can run the application with the following command:

```shell
 java -jar target/tailwind-0.0.1-SNAPSHOT.jar
```

Visit http://localhost:8080/ and you should now see a blue background.

## Resources 

- [Tailwind CSS Docs](https://tailwindcss.com/docs)
- [Spring Security Docs](https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/form.html)
- [Maciej Walkowiak](https://maciejwalkowiak.com/blog/spring-boot-thymeleaf-tailwindcss/)
