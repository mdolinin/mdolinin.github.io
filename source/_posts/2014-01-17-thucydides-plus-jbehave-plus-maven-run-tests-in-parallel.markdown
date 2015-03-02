---
layout: post
title: "Thucydides+JBehave+Maven run tests in parallel"
date: 2014-01-17 05:31:04 +0200
comments: true
categories: Thucydides, Jbehave, Maven
---
Thucydides is very cool test automation framework, with good built-in support of Selenium/WebDriver, understandable java API, simple architecture. This project also has integration with common BDD frameworks as JBehave and EasyB. Integration with JBehave is done by thucydides-jbehave project [src](https://github.com/thucydides-webtests/thucydides-jbehave).

JBehave allows you to write, store and run your tests in plain text files with *.story* extension.
``` gherkin
Scenario: trader is not alerted below threshold
 
Given a stock of symbol STK1 and a threshold of 10.0
When the stock is traded at 5.0
Then the alert status should be OFF
 
Scenario: trader is alerted above threshold
 
Given a stock of symbol STK1 and a threshold of 10.0
When the stock is traded at 11.0
Then the alert status should be ON
```
Each story file is same as one test suite. That's why it should be atomic and without dependecies from other tests.

The main goal of testing is to provide feedback to developers team as fast as possible. The most powerfull solution to reduce time spent for tests is run them in parallel processes. Now if you use Thucydides and Jbehave frameworks together out of the box, you can't run tests in parallel mode.
But exists severall solutions for this issue.
<!-- more -->
**First** is to divide automatically all stories on batches and then run them by some *build server*. [Here](http://wakaleo.com/index.php/blog/running-parallel-acceptance-tests-using-jbehave-thucydides-and-bamboo) is a good article written by Simeon Ross, which presents how to use this approach by using [Bamboo](https://www.atlassian.com/software/bamboo).

**Second** is to use [Maven Failsafe plugin parallel test execution ability](http://maven.apache.org/surefire/maven-failsafe-plugin/examples/fork-options-and-parallel-execution.html). Maven failsafe plugin has ability to run tests using *forks* by setting the parameter *forkCount* to a value higher than 1. The parameter *forkCount* defines the maximum number of JVM processes that Failsafe will spawn concurrently to execute the tests. Below you can find an example of this approach. 

First of all you should create JUnitStory test for each JBehave story. For example you have *use_calendar.story* file inside your *src/test/resources/stories* folder. That means you should create UseCalendar class in your *src/test/java* folder and extend it from ThucydidesJUnitStory class. Class name should be same as story name and match to java syntax (use "CamelCase" without spaces).
``` java
package com.mdolinin.acceptancetest;

import net.thucydides.jbehave.ThucydidesJUnitStory;

public class UseCalendar extends ThucydidesJUnitStory {}
```
Then make changes in configuration of your pom.xml. You should define where to find your JUnitStory classes and how many forks to use. For example you want to run all your stories in parallel mode. 
``` xml pom.xml
             <plugin>
                <artifactId>maven-failsafe-plugin</artifactId>
                <version>2.16</version>
                <configuration>
                    <systemPropertyVariables>
                        <webdriver.driver>${webdriver.driver}</webdriver.driver>
                    </systemPropertyVariables>
                    <includes>
                        <include>**/acceptancetest/*.java</include>
                    </includes>
                    <forkMode>perthread</forkMode>
                    <threadCount>4</threadCount>
                </configuration>
                <executions>
                    <execution>
                        <goals>
                            <goal>integration-test</goal>
                            <goal>verify</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
```
Now if you run your tests using maven then every story will run in one of four forks. 
``` bash
$ mvn integration-test
```
But it is boring to create new boilerplate class for each story file. That's why I created *maven-thucydides-jbehave-plugin* [src](https://github.com/mdolinin/maven-thucydides-jbehave-plugin) for this job. To use it add to your pom.xml build section with goal *generate-sources* and define *project.junit.stories.package*(where to put generated stubs) in properties section. See example below:
```xml pom.xml
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <webdriver.driver>firefox</webdriver.driver>
        <project.junit.stories.package>com.mdolinin.acceptancetest</project.junit.stories.package>
    </properties>
```
```xml pom.xml
             <plugin>
                <groupId>net.thucydides.maven.plugin</groupId>
                <artifactId>maven-thucydides-jbehave-plugin</artifactId>
                <version>0.9.223-SNAPSHOT</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>generate-sources</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
```
I hope this will be usefull for someone.