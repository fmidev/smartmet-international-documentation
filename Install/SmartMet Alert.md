#SmartMet Alert Installation Guide 

SmartMet Alert client is usually installed to one of the SmartMet Workstations. It is possible to 
install it to other computers as well.  

1. Download and install proper Java-version.  Recommended version is Bellsoft Liberica 
(based on openJDK) LTS version 8. It can be found from: https://bell-sw.com 

2. Make sure that you download FULL JDK, instead of proposed Standard JDK-package. 
Please also note that version is 8. (e.g. https://download.bell-
sw.com/java/8u292+10/bellsoft-jdk8u292+10-windows-amd64-full.msi) 
3. Download the newest software package from the address provided by FMI. (e.g. 
https://cdn.fmi.fi/b/smartmet-alert-kyrgyzstan-2021-05-27.zip) 
4. Check that the SmartMet server is connected and mapped as an drive S: in the computer 
where you want to do the installation.  
5. Unzip the downloaded SmartMet Alert installation package to the desired destination in 
your computer. 
6. Change the destination folder of the SmartMet Alert CAP messages to create the files 
into SmartMet server (or other wanted destination): 
1. From the installed SmartMet Alert package find the folder:smartmet-alert-
country/conf  
2. Open the file with notepad etc:application-configuration.xml 
3. Find the following configuration snippet (ca from rows 6-8) 4.  <bean id="repositoryOutputFolder" class="java.lang.String"> 
5.      <constructor-arg 
value="#{systemProperties['user.home']}/SmartMet-Alert-Country" 
/> 
</bean> 
 
6. Change the value attribute followingly (please note that the destination path may 
vary depending on the country. Default is S:\smartalert).  7.  <bean id="repositoryOutputFolder" class="java.lang.String"> 
8.      <constructor-arg value="S:\smartalert" /> 
</bean> 
 
9. In case test environment is set up and the produced CAP files are wanted to be 
produced there change the value attribute to point into the test folder. For 
example: S:\smartalert_test 
7. Start the program double clicking smartmet-alert-country.jar -file in the folder.  
1. If program does not start, check the used Java version. Program can be launched 
also from command line with command "java -jar smartmet-alert-country.jar"  
2. Known bug and Workaround: If map does not show up when starting the 

