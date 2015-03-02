---
layout: post
title: "Thucydides+JBehave+Maven run tests in fixed order"
date: 2014-01-19 13:59:07 +0200
comments: true
categories: Thucydides, Jbehave, Maven
---
In my previous [post](/blog/2014/01/17/thucydides-plus-jbehave-plus-maven-run-tests-in-parallel), I discribed how to run tests in parallel mode using Thucycdides,Jbehave and Maven Failsafe Plugin. But sometimes you need to run tests in fixed order, for example when you:

* make some preconditions
* clear data
* import new configuration
* test applications when some services are shutdown
* etc.
<!-- more -->
Maven failsafe plugin has ability to add different executions in fixed order.
This feature can be used by adding *executions* sections into failsafe configuration, for each test suite. They will run in the same order as discribed in pom.xml file.

For example you have several **.story* files and one of them named Preconditions.story. And you know that according to business logic of your application all stories except Preconditions can be run in parallel order. But Preconditions.story should be executed before others. Then you can use configuration that shown below.
``` xml pom.xml
        <plugin>
            <artifactId>maven-failsafe-plugin</artifactId>
            <version>2.16</version>
            <executions>
                <execution>
                	<id>preconditions</id>
                    <goals>
                    	<goal>integration-test</goal>
                    </goals>
                    <configuration>
                    	<systemPropertyVariables>
                        	<webdriver.driver>${webdriver.driver}</webdriver.driver>
                        </systemPropertyVariables>
						<includes>
						   <include>**/acceptancetest/Preconditions.java</include>
						</includes>
                        <summaryFile>target/failsafe-reports/failsafe-summary-preconditions.xml</summaryFile>
                    </configuration>
				</execution>
			    <execution>
			       <id>acceptance-test-parallel-execution</id>
			       <goals>
			           <goal>integration-test</goal>
			       </goals>
			       <configuration>
			           <systemPropertyVariables>
			               <webdriver.driver>${webdriver.driver}</webdriver.driver>
			           </systemPropertyVariables>
			           <includes>
			               <include>**/acceptancetest/*.java</include>
			           </includes>
			           <excludes>
			               <exclude>**/acceptancetest/Preconditions.java</exclude>
			           </excludes>
			           <forkCount>4</forkCount>
			           <reuseForks>true</reuseForks>
			           <summaryFile>target/failsafe-reports/failsafe-summary-acceptance-test-parallel-execution.xml</summaryFile>
			       </configuration>
			   </execution>
			   <execution>
			       <id>verify</id>
			       <goals>
			           <goal>verify</goal>
			       </goals>
			       <configuration>
			           <summaryFiles>
			               <summaryFile>target/failsafe-reports/failsafe-summary-acceptance-test-preconditions.xml</summaryFile>
			               <summaryFile>target/failsafe-reports/failsafe-summary-acceptance-test-parallel-execution.xml</summaryFile>
			           </summaryFiles>
			       </configuration>
			   </execution>
			</executions>
        </plugin>
``` 
You can create *execution* sections as many as you want but this is bad practice, because your tests become dependent and execution time will rise. That's why I suggest to launch most of the tests in one run with parallel mode.