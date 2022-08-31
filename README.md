# FileGazer - deep file analysing


![FileGazer](filegazer_small.png)

## Table of Content

- [FileGazer - deep file analysing](#filegazer---deep-file-analysing)
  * [Intro](#intro)
  * [Inspectors](#inspectors)
    + [Base](#base)
    + [Magicnumber/Mimetype](#magicnumber-mimetype)
    + [Hashcode](#hashcode)
    + [Barcode](#barcode)
    + [Tika](#tika)
    + [Content Analyse](#content-analyse)
      - [Aufbau der FileGazerContentAnalyse.xml](#aufbau-der-filegazercontentanalysexml)
        * [\<Classification>](#--classification-)
        * [\<FindeClasses>](#--findeclasses-)
        * [\<Scripts>](#--scripts-)
        * [Beispiel](#beispiel)
  * [Interfaces](#interfaces)
    + [Web-Application](#web-application)
    + [ReST Services](#rest-services)
      - [Inspect a file](#inspect-a-file)
        * [Curl-example:](#curl-example-)
      - [Execute scripts](#execute-scripts)
      - [Search files](#search-files)
  * [Filegazer Scripting](#filegazer-scripting)
    + [FileGazerScripts.xml](#filegazerscriptsxml)
      - [FileGazerScript-Tag](#filegazerscript-tag)
  * [Installation, setup and starting](#installation--setup-and-starting)
    + [Setup FileGazer](#setup-filegazer)
      - [Start-Batch (Example)](#start-batch--example-)
      - [Install a windows service](#install-a-windows-service)
      - [application.properties](#applicationproperties)
    + [Install tesseract (for OCR)](#install-tesseract--for-ocr-)
      - [Ubuntu](#ubuntu)
      - [macOS](#macos)
      - [Microsoft Windows](#microsoft-windows)
      - [Docker](#docker)
  * [Links and further information](#links-and-further-information)
    + [Tika](#tika-1)
    + [Tesseract](#tesseract)
    + [magic number](#magic-number)
  * [Beyond](#beyond)
  * [Help & contact...](#help---contact)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>


## Intro


**FileGazer** 'collect' all available information for a certain file. This is done with the help of the *inspectors* (currently we have 6 inspectors). You can use FileGazes as a Web Application to get all these informations from a file (or search files) or you use FileGazer REST API.

Current inspector:
- Base (Filename, Size, Timestamps...)
- MagicNumber (determ filetype based on the *magic number*)
- Hashcode (determ a list of well known Hascodes)
- Barcode (determ available barcodes - on documents: pdf, tif, png, jpg)
- Tika (use Apache Tika to get all available attribute/metadata and also content. If tesseract is install also do a OCR)
- Content analysing, Based on the file content and your rules Filegazer categorize the file (document) and extract indexdata

(Please find a detail description for all inspectors in this document)


**FileGazer** basily offers to function: 

- Inspect a file: Execute all Inspectors and give all available information.
- Search a file: Search for a regExpr.-Phrase in a list of files - search in all inspector-results
- REST Interface: to call Filegazer from other application
- Scripting: define your own groovy script to manage your files/documents

**Feature overview**

- Search in all available Metadata (GPS-infos of your pictures, content, barcodes, OCR text, ....)
- Do a full and powerfull OCR (with help of tesseract)
- determ more meta data you ever expect
- easy to use Web-app
- REST Service
- Scripting
- available for linux, windows and Mac
- classify your documents by its content 


## Inspectors

A inspector is a piece of code which invesigate the given file(s). This chapter descripes the available inspectors in detail. 

### Base

This inspector reads the basic attributes of the file: 

- Filename
- Pathname
- Size 
- Hidden flag
- Symbolic link flag
- Timestamps (create, modify, access) 

### Magicnumber/Mimetype

A magic number is a numeric or string constant that indicates the file type. This number is in the first 512 bytes of the file.
This inspector read that byte sequence and try to determ the filetype/mimetype. In somes cases it offer a link to the file specification.

### Hashcode

The hascode inspector determ a list of well known hascode: GOST3411, MD2 , MD4, MD5, RIPEMD128, RIPEMD160, RIPEMD256, RIPEMD320, SHA1, SHA224, SHA256, SHA384, SHA512, SM3, Tiger, Whirlpool

### Barcode

This inspector reads all available barcodes. For this it may be necessary to convert the file (for example pdf to image). This may take a while; which means using this inspector in a search can slow down the search.

Here the list of Barcode which this inspector can read: 

| 1D Barcode | 2D Barcode |
| ------| ------| 
| UPC-A | QR Code |
| UPC-E | Data Matrix |
| EAN-8 | Aztec |
| EAN-13 | PDF 417 |
| UPC/EAN | Maxi Code |
| Code 39 | RSS-14 | 
| Code 93 | RSS-Expanded |
| Code 128 | 
| Codabar | 
| ITF |

### Tika

Based on Apache Tika this inspector reads all available attributes/metadata and the file content. In case there is no tesseract installed it can read content from PDF-files (if there is a content), all office documents, text files and so on. 

If FileGazer find a local instance of tesseract (https://github.com/tesseract-ocr) this inspector run a OCR to get the file content. Please check the install instructions in this document. 

For detailed information about Apache Tika check this: https://tika.apache.org/

### Content Analyse

With the help of content analysis, the text of the file or document (if available) can be categorised. Based on the categorisation rules 
are used to define what kind - what class - of document is involved. 
In a further step, data can be extracted from the document via the class-specific 'finders'. 

For example, invoices, delivery notes, incoming mail, etc. can be classified and indexed. 

The rules for analysis and extraction are defined in the file **FileGazerContentAnalyse.xml**. 

#### Structure of the FileGazerContentAnalyse.xml

The file *FileGazerContentAnalyse.xml* is essentially divided into three sections: 

* **\<Classification>** - Here, several \<Classificationrule> are used to determine what type or class of document it is.
* **\<FinderClasses>** - For each class defined in the \<Classification>, a corresponding \<FinderClass> is defined here, which in turn contains one to n \<Finders>. Each finder extract exactly one index value (invoice number, customer number, indicator....). In addition to the specific finder classes, there are also the classes "_unknown" and "_default". "_unknown" is executed if the classification could not determine a result. "_default" is always executed (if a finder class was determined, "_default" is executed afterwards).
* **\<Scripts>** - This area is divided into \<Prepare> - scripts that adjust the content or the read value/index and \<Validate> - scripts that check whether a found value is correct and can be accepted. 

The scripts are groovy scripts that can be as complex as desired. 

##### \<Classification>

| Attribute | Description | 
| ----- | ----- | 
| undefMethod | If no clear result is determined (several classification rules were triggered), the result to be used is determined here. Possible values: FIRST, LAST, BEST |
| takeFirst | If this value is set to true, the evaluation of the individual classification rules is terminated as soon as a positive result is achieved. In this case, the attribute "undefMethod" is irrelevant or set to FIRST. This setting makes sense in order to achieve faster processing. |

**Classificationrule**
| Attribute | Description | 
| ----- | ----- | 
| name | Name of the classification rule. This name is helpful in an analysis to find out which rule has "fired". |
| classname | Name of the assigned class. For the extraction of index values, the FinderClass is then called up with exactly this name. |
| contentScript | Name of the script (or scripts separated by commas) to be executed on the content before(!) the search/evaluation. In this way, the content to be examined can be prepared, processed or cleaned. | 
| regExpr | RegExpr which determines whether it is a document of the defined class. |

##### \<FindeClasses>

The assignment of a Finderclass to the corresponding class name is done via the attribute "name". A finder class contains 1... n "finders". Each Finder 
searches the content for index values and assigns them to an ItemName: 

**Finder** 
| Attribute | Description | 
| ----- | ----- | 
| name | Name of the finder. Can be chosen arbitrarily, but should be unique for better analysis.  |
| contentScript | Script to be executed on the content before the search. In this way, the content to be examined can be prepared, edited or cleaned. |
| regexpr | Regular expression to search for the desired data. The first grouping expression ($1) or the first () is extracted. |
| itemScript | Script that is executed to post-process the read result. |
| validateScript | This script can check whether the value found is correct (for example, whether the "customer number" can be found in the database). The value is only taken over if the validateScript returns "true"|
| itemName | Name of the index/item to which the value is to be assigned. If a Finder has already made an assignment, it will be overwritten. |


##### \<Scripts>

These scripts are divided into "**Prepare**" scripts and "**Validate**" scripts. 
When calling up a script, several scripts can also be specified one after the other (separated by commas).

"Prepare" scripts take the supplied content (in the script this is the variable "**content**"), change this content and return it to the caller. If it is an "itemScript", it is the found text. In all other cases, it is the content of the document.
A prepare script can also be given an argument. This argument can be queried in the script in the variable "**argv**".

"Validate" scripts also receive the text to be checked in the variable "content" and must always return a boolean value.

##### Beispiel

    <!-- 

        Simple Example for a contentAnalyse configuration file.

    -->
    <FileGazerFinderConfig>

        <Classification undefMethod="BEST" takeFirst="true">

            <Classificationrule name="Incoice-1"  classname="Invoice"  contentScript="" regExpr="(?s).*Invoice"/> 
            <Classificationrule name="Incoice-2"  classname="Invoice"  contentScript="" regExpr="(?s).*Rechnung"/> 		

        </Classification>
        
        <FinderClasses>
            
            <FinderClass name="Invoice">
                
                <!-- set a static value -->
                <Finder name="TYPE"             
                        contentScript="static(Invoice)" 
                        regExpr="^(.*)$"  
                        itemScript="" 
                        validateScript="" 
                        itemName="TYPE"/>

                <Finder name="Invoicedate" 
                        contentScript="" 
                        regExpr="Invoicedate:[ 	]*(\d\d\.\d\d\.\d{4})" 
                        itemScript="" 
                        validateScript="" 
                        multiselect="FIRST"
                        itemName="Invoicedate"/>	
                        
            </FinderClass>

            <!-- no classification -->
            <FinderClass name="_unknown">
            </FinderClass>

            <!-- run this class in any case!!! -->
            <FinderClass name="_default">
            </FinderClass>

        </FinderClasses>

        <Scripts>

            <Prepare>

                <!-- prepare Content for search/finder -->
                <Script name="removeWhiteSpace">
                    <code>
                        <![CDATA[
                            content.replaceAll("[\\t ]", "")
                        ]]>
                    </code>
                </Script>
                <Script name="removeNL">
                    <code>
                        <![CDATA[
                            content.replaceAll("[\\n\\t ]", " ")
                        ]]>
                    </code>
                </Script>
                <Script name="subContent">
                    <code>
                        <![CDATA[
                        if( content != null ) {
                            if( content.length() > 1900 ) {
                                content.substring(1,1900)
                            } else {
                                content
                            }
                        }
                        ]]>
                    </code>
                </Script>

                <!-- helper to set a static value -->
                <Script name="static">
                    <code>
                        <![CDATA[
                            if( argv != null ) {
                                argv
                            } else {
                                ""
                            }
                        ]]>
                    </code>
                </Script>

            </Prepare>

            <Validate>
                <Script name="example">
                    <![CDATA[
                            return true;
                        ]]>
                </Script>
            </Validate>

        </Scripts>
        
    </FileGazerFinderConfig>




## Interfaces

Currently FileGazer offers a WebApplication and a simple REST-Service.

### Web-Application 

The Web-Frontend offers to basic function: inspect a file by select a local file or search for files. This interface is currently just for local use.

### ReST Services

FileGazer provide three REST endpoint. One to inspect a file, one to trigger a script execution and the last to search for files. 

#### Inspect a file

You can inspect a file with this endpoint. 

    http://myServer:Port/filegazer/file/inspect

use

    http://myServer:Port/filegazer/file/inspect/local

to inspect a file which filegazer can find localy

or 

    http://myServer:Port/filegazer/file/inspect/upload

if you want to upload a file to filegazer for inspection. 

The following parameters are required with the corresponding end point: 

| Parameter | Description | Example |
| ----- | ----- | ----- |
| inspector | Which inspectors should examine the file. The names of the inspectors are separated by commas and are usually given in capital letters | file/inspect/local?inspector=TIKA,CONTENTANALYSE,BARCODE |
| format | In which format should the result be returned? XML or JSON | - | 
| transformation | If it is an XML return, the xslt script can be specified here that transforms the result of the inspector accordingly. | transformation=getClassification |

In the case of a "local" call, the name (plus path) of the file will be specified as POST workload. In the case of an upload, a MultipartFile is expected. 

| | |
| ----- | ----- |
| Method | POST |
| Endpoint | http://localhost:8080/filegazer/file/inspect/{local or upload}? | 
| Parameter [format] |  Result format "xml" or "json"     |
| Parameter [inspector] |  list (seperator is ",") of inspectornames to inspect the given file. BASE,BARCODE,HASHCODE,MAGICNUMBER,TIKA,CONTENTANALYSE  |
| Parameter [transformation] |  xslt script (in ./etc/xlst) to transform the inspector-result |
| Workload/Body | Filename (inlude path) to inspect or MultipartFile |


FileGazer expect the fileName of the xslt stylesheet without the .xslt suffix and search the file in the current directory (of FileGazer)

    curl --location --request POST 'http://localhost:8080/filegazer/file/inspect/local/inspector=TIKA,CONTENTANALYSE,BARCODE&transformation=inspect2docProcessing' --data-raw '/home/filegazer/ExampleFiles/Picture.jpg'

Filegazer search a file named "/home/filegazer/ExampleFiles/Picture.jpg" run the inspectors TIKA, CONTENTANALYSE and BARCODE and transform the xml output with the script inspect2docProcessing.xslt (which you can find in ./etc/xslt).


##### Curl-example:

get all available information for a file (default format is xml)

    curl --location --request POST 'http://localhost:8080/filegazer/file/inspect/local --data-raw '/home/filegazer/ExampleFiles/Picture.jpg'

just read the barcode

    curl --location --request POST 'http://localhost:8080/filegazer/file/inspect/local?inspector=BARCODE --data-raw '/home/filegazer/ExampleFiles/Picture.jpg'

read hascodes and content (Tika) and transform the result by a xslt script

    curl --location --request POST 'http://localhost:8080/filegazer/file/inspect/local?inspector=HASHCODE,TIKA&transformation=myXSLTScript --data-raw '/home/filegazer/ExampleFiles/Picture.jpg'


#### Execute scripts

This endpoint is only used to execute a FilegazerScript (these are listed in the ./etc/FileGazerScripts.xml file). 
A script can only be executed through this endpoint if the attribute "isRestExecute" in FileGazerScripts.xml has been set to true. 

|  |   |
| ----- | ----- |
| Method | GET |
| Endpoint | http://localhost:8080/filegazer/execute/{scriptname} | 

"scriptname" is the name of the script which you want to execute - it must be listed in "FileGazerScripts.xml" and set to isRestExecute=true

#### Search files

With 

     http://localhost:8080/filegazer/search/directory/template

you get the template to run a search (searchrequest): 

    {
        "searchText": null,
        "maxFound": -1,
        "dirName": null,
        "fileFilter": null,
        "dirFilter": null,
        "recursive": false,
        "stopAfterFirst": false,
        "regSearch": false,
        "ignoreCase": false,
        "justValues": false
    }

| item | description |
| ----- | ------ | 
| searchText | the phrase you searching for. You can use a regexpr if it is activated! |
| maxFound | after n findings the search stops. -1 means "no limit" | 
| dirName | pathname where to start the search |
| fileFilter  | regexpr to identify the files | 
| dirFilter | regexpr to select the directories - in case you run a recursive search | 
| recursive | run the search just in the given directory (false) or do a recursive search (true) |
| stopAfterFirst | stop the search as soon as we found sonething - like maxFound=1 |
| regsearch | should we use the searchText as a regexpr? |
| ignoreCase | ignore lower or upper case | 
| justValues | espacially for the Tika metadata: per default we search in the metadata-name AND in the value. You can switch that of | 


| | |
| ----- | ----- |
| Method | POST |
| Endpoint | http://localhost:8080/filegazer/search/directory |
| Workload/Body | Searchrequest  |

## Filegazer Scripting

With the FileGazer scripts, the functions of FileGazer can be combined in scripts and executed locally. These scripts are formulated in groovy and can be as complex as you like. By default, the scripts are stored in ./etc/scripts. Any java libraries that may be required should be stored under ./lib. 

### FileGazerScripts.xml

In this file, all scripts that are prepared for execution are defined. 

Example: 
    <FileGazerScriptsConfig>

        <FileGazerScripts>

            <FileGazerScript name="startupDone" description="a simple example how to start a script after application startup" onStartup="true" onExit="false" onTimer="" isFileLink="false" isRestExecute="true">
                <![CDATA[
                            println "##################"
                            println "#  Startup done! #"
                            println "##################"
                        ]]>
            </FileGazerScript>


            <FileGazerScript name="shutDown" description="a simple example how to start a script before application shutdown" onStartup="false" onExit="true" onTimer="" isFileLink="false" isRestExecute="true">
                <![CDATA[
                            println "##################################"
                            println "#  about to shutdown... bye bye  #"
                            println "##################################"
                        ]]>
            </FileGazerScript>

            <!-- 
                    Example scripts ... 
            -->
            <FileGazerScript name="getTheAnswer" description="This script return the answer to life the universe and everything " onStartup="false" onExit="false" onTimer="* * * 12-14" isFileLink="true" isRestExecute="true">
                TheAnswer.groovy
            </FileGazerScript>

            <FileGazerScript name="Doc2txt" description="Pick up all files in /Example/IN, get content from filegazer and create textfile" onStartup="false" onExit="false" onTimer="" isFileLink="true" isRestExecute="true">
                Doc2txt.groovy
            </FileGazerScript>

            <FileGazerScript name="DocCategorization" description="Pick up all filesin /Example/IN, ask filegazer for the documenttype and move this file to ./Example/in /Example/IN/{documenttype}" onStartup="false" onExit="false" onTimer="" isFileLink="true" isRestExecute="true">
                DocCategorization.groovy
            </FileGazerScript>

            <FileGazerScript name="CreateIndexControlFile" description="Pick up all filesin /Example/IN, ask filegazer for detailinformation and create a html file to view the metadata and the file as well (just for pdf documents)" onStartup="false" onExit="false" onTimer="" isFileLink="true" isRestExecute="true">
                CreateIndexControlFile.groovy
            </FileGazerScript>

        </FileGazerScripts>

    </FileGazerScriptsConfig>

#### FileGazerScript-Tag

Each script is defined in a FileGazerScript tag. This tag can either contain the groovy code directly (for example, in "startupDone") or - for more complex scripts - the tag contains the name of the file that contains the code. This source code file must be provided under ./etc/scripts. 

| Attribut | description |
| ----- | ------ | 
| name | Any (and unique) name for the script |
| description | Descripton |
| onStartup | {true,false} - should the script be executed automatically when FileGazer starts up? |
| onExit | {true,false} - should the script be executed automatically when FileGazer shuts down? |
| onTimer | cron String (0 6,18 * * *) that specifies when the script should be executed. |
| isLink | {true,false} - is it an embedded code (false) or the name of a script (true)? |
| isRestExecute | {true,false} - may the script be called via the Rest API? Example: curl http://localhost:8080/filegazer/execute/Doc2txt |


## Installation, setup and starting

### Download 

The current version of FileGazer is available for download under 

    https://filegazer.eichelsheimer.de/filegazer/filegazer-0.1.17.jar

### Setup FileGazer 

Just copy the current version of Filegazer to your host and run

    java -jar filegazer-0.1.17.jar 
	
After the first start FileGazer create a new etc-directory (if not exists) and store some example configurationsfiles there. 

#### Start-Batch (Example)

    @echo off
    rem
    rem Filegazer starter....
    rem

    mode con:cols=250 

    rem may set your own java Version
    rem set JAVA_HOME=X:\FileGazer\java\jdk-18.0.1.1

    set VERSION=0.1.16
    set FILEGAZER_JAR=filegazer-%VERSION%.jar
    set FILEGAZER_MAIN=de.samoak.filegazer.FilegazerApplication

    set DIR_ROOT=X:\FileGazer
    set DIR_LIB=%DIR_ROOT%/lib

    set MEMORY=-Xms1024m -Xmx2048m

    rem set etc dirs - if nessecary 
    set ETC=--filegazer.etc=%DIR_ROOT%/etc --filegazer.etc.xslt=%DIR_ROOT%/etc/xslt --filegazer.etc.scripts=%DIR_ROOT%/etc/scripts

    rem Set port to 9090 and set OCR to active and limit barcode detection to the first 3 pages 
    rem (check application.propperties for more parameter)
    set PARAMETER=--server.port=9090 --filegazer.inspector.tika.skipocr=false --filegazer.inspector.barcode.max.pages=3

    cd %DIR_ROOT%
    echo . 
    echo starting...
    echo . 
    java  %MEMORY% -cp %FILEGAZER_JAR% -Dloader.path=%DIR_LIB% -Dloader.main=%FILEGAZER_MAIN% org.springframework.boot.loader.PropertiesLauncher %PARAMETER% %ETC% 

I think it is easy to "translate" this script to a unix-shell script. 

#### Install a windows service

There are various methods for installing Filegazer or a Java application as a service under Windows. I refer here to "WinSW" (https://github.com/winsw/winsw).

Cookbook:

* Copy WinSW-x64.exe into the directory where filegazer-xxxx.jar is located. For example: D:\filegazer
* Rename "WinSW-x64.exe" to "FileGazer.exe".
* Create the Filegazer.xml (example below)
* Install the service: Filegazer install
* Start the service: FileGazer start
* Stop the service: FileGazer stop
* Remove the service: Filegazer uninstall

Example of a WinSW configuration file:

    <service>
        <id>Filegazer</id>
        <name>FileGazer</name>
        <description>FileGazer Service</description>
        <workingdirectory>D:\filegazer</workingdirectory>
        <priority>normal</priority>
        <executable>c:\Program Files\Java\jdk-17.0.1\bin\java.exe</executable>
        <arguments>-Xrs -Xms1024m -Xmx2048m -jar filegazer-0.1.16.jar -Dloader.path=D:\filegazer\lib -Dloader.main=de.samoak.filegazer.FilegazerApplication org.springframework.boot.loader.PropertiesLauncher</arguments>
        <log mode="roll"></log>
    </service>

In this case, all parameters are read from the application.properties. See next chapter

#### application.properties

May you want to adjust some application properties: 

You can change these parameters by: 

1) Extract application.properties from the jar. Do your changes. Pack the file back to the jar-file.
That's great, because you just have one file for your application and it is not that easy to change something. On the other hand, it is not that flexible. I think it is a good way for a production environment.

2) Extract application.properties from the jar. Do your changes. Store the file in the same directory like your jar-File.
This great because you can easily change your setting or you prepare a couple of different profiles.

3) Passing a parameter when starting the application:
java -jar FileGazer-xxxxx.jar --server.port=9090
This is a good way if you have just a few parameters to change.

Please check the content of application.propieties - you will find a detailed description for all parameters.

### Install tesseract (for OCR)

This section gives a short description "how to install tesseract".
A small wiki for Tika and tesseract here: https://cwiki.apache.org/confluence/display/tika/TikaOCR

#### Ubuntu
        
    sudo apt-get update
    sudo apt-get install tesseract-ocr
    
May you have to install a language-pack: 

    sudo apt-get install tesseract-ocr-deu

(Replace "deu" with you country code)

#### macOS

    brew install tesseract tesseract-lang

#### Microsoft Windows

You find a windows setup and further instructions under: https://github.com/UB-Mannheim/tesseract/wiki

#### Docker

....

## Links and further information 

### Tika 
    
- https://tika.apache.org/

### Tesseract 

- https://github.com/tesseract-ocr/tesseract

### magic number

- https://en.wikipedia.org/wiki/List_of_file_signatures
- https://www.garykessler.net/library/file_sigs.html

## Beyond

What is the issues I'll work on next ? 

- Bufixing (may with your help)
- sceduling for scripts (not implemnted yet)
- add results of the content analyse to the search 


## Help & contact...

- If you think FileGazer is helpful, please let me know :-) ... 
- If have idea for a new inspector, please let mew know :-) ...
- If have a problem or a question,  please let mew know :-) ...
- If you found an error,  please let mew know :-) ...

In one word: feel free to contact me... I am looking forward

Sven@Eichelsheimer.de