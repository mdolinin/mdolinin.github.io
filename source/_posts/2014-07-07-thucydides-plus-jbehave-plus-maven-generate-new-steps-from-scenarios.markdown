---
layout: post
title: "Thucydides+JBehave+Maven Generate new Steps from Scenarios"
date: 2014-07-07 10:30:39 +0300
comments: true
categories: 
- Thucydides 
- Jbehave 
- Maven 
---
If you are using Jbehave framework, you know that it has very simple structure of *.story* file content.
Which consists of steps - Given,When,Then. And only steps are required part of the story. But this framework provides us to add meta-information to our tests for better understanding. One of the way to do this is use *Scenario:* sections.
``` gherkin
Scenario: log in as autorized user
 
Given user on authorization page
When he logs in as John with password 123
Then opens user's page
```
It is very useful feature, which divide story into sections and makes it more readable/understandable.
But you can use *Scenario* to create your own steps library automatically.
<!-- more -->
To do this you need *maven-thucydides-jbehave-plugin* [src](https://github.com/mdolinin/maven-thucydides-jbehave-plugin). Add it to your pom.xml build section with goal *generate-steps*. See example below:
```xml pom.xml
             <plugin>
                <groupId>net.thucydides.maven.plugin</groupId>
                <artifactId>maven-thucydides-jbehave-plugin</artifactId>
                <version>0.9.223</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>generate-steps</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
``` 
This plugin starts working on phase *process-test-classes* (by default).

And as result will *generate new **When** steps for each **Scenario** section*.

Let's see his work on example. If you have APassingBehavior.story file with scenario shown above inside,
then plugin execution create class with two new steps.
```java APassingBehaviorSteps.java
package net.thucydides.maven.plugin.test;

import net.thucydides.maven.plugin.test.LoginSteps;
import java.lang.String;
import net.thucydides.core.annotations.Steps;
import net.thucydides.core.pages.Pages;
import net.thucydides.core.steps.ScenarioSteps;
import org.jbehave.core.annotations.When;

public class APassingBehaviorSteps extends ScenarioSteps {

    private static final long serialVersionUID = 1L;

    @Steps
    private LoginSteps loginSteps;

    public APassingBehaviorSteps(Pages pages) {
        super(pages);
    }

    @When("log in as autorized user $login $pass")
    public void logInAsAutorizedUser(String login, String pass) {
        loginSteps.givenUserOnAuthorizationPage();
        loginSteps.whenHeLogsInWithPassword(login, pass);
        loginSteps.thenOpensUsersPage();
    }

    @When("log in as autorized user")
    public void logInAsAutorizedUser() {
        loginSteps.givenUserOnAuthorizationPage();
        loginSteps.whenHeLogsInWithPassword("John", "123");
        loginSteps.thenOpensUsersPage();
    }

}
```
In result:

* steps will have name as scenario title
* first step will require to enter all parameters from inside steps explicitly
* second step will use parameters from scenario
* you can rewrite scenario to one step
``` gherkin
When log in as autorized user
```
* or
``` gherkin
When log in as autorized user John 123
```

Also you can create *parameterized Scenario Steps* using '$' sign, like in Jbehave.
Add parameters from any inside step to scenario title.
``` gherkin
Scenario: log in as $login user
 
Given user on authorization page
When he logs in as John with password 123
Then opens user's page
```
And rerun plugin.
```java APassingBehaviorSteps.java
package net.thucydides.maven.plugin.test;

import net.thucydides.maven.plugin.test.LoginSteps;
import java.lang.String;
import net.thucydides.core.annotations.Steps;
import net.thucydides.core.pages.Pages;
import net.thucydides.core.steps.ScenarioSteps;
import org.jbehave.core.annotations.When;

public class APassingBehaviorSteps extends ScenarioSteps {

    private static final long serialVersionUID = 1L;

    @Steps
    private LoginSteps loginSteps;

    public APassingBehaviorSteps(Pages pages) {
        super(pages);
    }

    @When("log in as $login $pass")
    public void logInAslogin(String login, String pass) {
        loginSteps.givenUserOnAuthorizationPage();
        loginSteps.whenHeLogsInWithPassword(login, pass);
        loginSteps.thenOpensUsersPage();
    }

    @When("log in as $login")
    public void logInAslogin(String login) {
        loginSteps.givenUserOnAuthorizationPage();
        loginSteps.whenHeLogsInWithPassword(login, "123");
        loginSteps.thenOpensUsersPage();
    }

}
```
So now you can use your login step.
``` gherkin
When log in as John
```

I hope this will be usefull for someone.