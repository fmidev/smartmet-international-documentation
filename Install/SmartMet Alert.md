# SmartMet Alert Installation Guide 

SmartMet Alert client is usually installed to one of the SmartMet Workstations. It is possible to 
install it to other computers as well.  

1. Download and install proper Java-version.  Recommended version is Bellsoft Liberica 
(based on openJDK) LTS version 8. It can be found from: https://bell-sw.com/pages/downloads/#jdk-8-lts

2. Make sure that you download FULL JRE, instead of proposed Standard JDK-package. 
Please also note that version is 8. Do NOT download standard version which is the default link.

3. Download the newest software package from the address provided by FMI. (e.g. 
https://cdn.fmi.fi/b/smartmet-alert-yourcountry-yyyy-mm-dd.zip) 

4. Check that the SmartMet server is connected and mapped as an drive S: in the computer 
where you want to do the installation.  

5. Unzip the downloaded SmartMet Alert installation package to c://smartmet folder in 
your computer. 

6. Change the destination folder of the SmartMet Alert CAP messages to create the files 
into s://smartalert (or other wanted destination):

7. Go to folder c://smartmet/smartmet-alert-
country/conf  
8. Open the file with notepad application-configuration.xml 
9. Find the following configuration snippet (ca from rows 6-8)
```xml
    <bean id="repositoryOutputFolder" class="java.lang.String"> 
    <constructor-arg value="#{systemProperties['user.home']}/SmartMet-Alert-Country" /> 
    </bean>
```

10. Change the value attribute followingly (please note that the destination path may vary depending on the country. Default is S:\smartalert).
```xml
     <bean id="repositoryOutputFolder" class="java.lang.String"> 
     <constructor-arg value="S:\smartalert" /> 
     </bean>
```
 
11. In case test environment is set up and the produced CAP files are wanted to be 
produced there change the value attribute to point into the test folder. For 
example: S:\smartalert_test 

12. Start the program double clicking smartmet-alert-country.jar -file in the folder.  

⚠️ If program does not start, check the used Java version. Program can be launched 
also from command line with command "java -jar smartmet-alert-country.jar"  

⚠️ Known bug and Workaround: If map does not show up when starting the 
program, resize SmartMet Alert window
