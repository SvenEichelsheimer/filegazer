<p align="right">
  <img src="Documents/img/filegazer_small.png" alt="FileGazer" width="150"/>
</p>

# [FileGazer - deep file analysing 🔗](https://filegazer.eichelsheimer.de)

## Table of Content

- [FileGazer - deep file analysing](#filegazer---deep-file-analysing)
  * [Table of Content](#table-of-content)
  * [Intro](#intro)
  * [Inspectors](#inspectors)
    + [Base](#base)
    + [Magicnumber/Mimetype](#magicnumber-mimetype)
    + [Hashcode](#hashcode)
    + [Barcode](#barcode)
    + [Tika](#tika)
    + [Content Analyse](#content-analyse)
      - [Structure of the FileGazerContentAnalyse.xml](#structure-of-the-filegazercontentanalysexml)
        * [\<Classification>](#--classification-)
        * [\<FinderClasses>](#--finderclasses-)
        * [\<Scripts>](#--scripts-)
        * [Example](#example)
    + [PDF/A validation](#pdf-a-validation)
    + [PDF Convert](#pdf-convert)
  * [Interfaces](#interfaces)
    + [Web-Application](#web-application)
    + [ReST Services](#rest-services)
      - [Inspect a file](#inspect-a-file)
        * [Curl-example:](#curl-example-)
      - [Execute scripts](#execute-scripts)
      - [Search files](#search-files)
      - [REST Services for configuration](#rest-services-for-configuration)
        * [contentanalyse](#contentanalyse)
        * [scripts](#scripts)
        * [path](#path)
        * [path/filename](#path-filename)
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
    + [Gotenberg](#gotenberg)
    + [Docker](#docker)
      - [pull](#pull)
      - [start](#start)
  * [Links and further information](#links-and-further-information)
    + [Tika](#tika-1)
    + [Tesseract](#tesseract)
    + [magic number](#magic-number)
    + [cron expression](#cron-expression)
    + [PDF/A](#pdf-a)
    + [Gotenberg project](#gotenberg-project)
  * [Beyond](#beyond)
  * [Help & contact...](#help---contact)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>


## Intro

**FileGazer** 'collect' all available information for a certain file. This is done with the help of the *inspectors* (currently we have 8 inspectors). 
FileGazer is a RestService with a very simple webUI - just for testing. 

Current inspector:
- Base (Filename, Size, Timestamps...)
- MagicNumber (determ filetype based on the *magic number*)
- Hashcode (determ a list of well known Hascodes for the file)
- Barcode (determ available barcodes - on documents: pdf, tif, png, jpg)
- Tika (use Apache Tika to get all available attribute/metadata and also content. If tesseract is install also do a OCR)
- Content analysing, Based on the file content and your rules FileGazer categorize the file (document) and extract indexdata
- PDF/A analysing: determ if the given (PDF)file follows the PDF/A-1a specification
- PDF Convert: Convert non PDF documents to PDF/A

(Please find a detail description for all inspectors in this document)

You can also write your own (groovy) scripts and schedule these script based on a timer or a event. 

The REST Service returns a XML. If you need to transform this xml (may you need just aone or a few informations) FileGazer can do this for you: just store a xslt file in ./etc/xslt and reference this file when you call FileGazer. He will then do the transformation job for you.

**Feature overview**

- Search in all available Metadata (GPS-infos of your pictures, content, barcodes, OCR text, ....)
- Do a full and powerfull OCR (with help of tesseract)
- determ more meta data you ever expect
- easy to use Web-app
- REST Service
- Scripting
- available for linux, windows and Mac
- classify your documents by its content 
- convertion files to PDF/A (with the help from Gotenberg :-))
- If you inspect a PDF file: determ if this PDF is a PDF/A


## Inspectors

A inspector is a piece of code which investigate the given file(s). This chapter descripes the available inspectors in detail. 

### Base

This inspector reads the basic attributes of the file: 

- Filename
- Pathname
- Size 
- Hidden flag
- Symbolic link flag
- Timestamps (create, modify, access) 
- FileContent as a base64 stream for later use

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

**Note:** Every defined classname need a correspondending Finderclass. If not FileGazer will not run a ContentAnalyse.

##### \<FinderClasses>
If true, the web frontend can be used.
The assignment of a Finderclass to the corresponding class name is done via the attribute "name". A finder class contains 1... n "finders". Each Finder 
searches the content for index values and assigns them to an ItemName: 

**FinderClass**
| Attribute | Description | 
| ----- | ----- | 
| name | Name of the finderclass. |
| postScript | Name or list of scripts which are execute after all finder finished. You get the variable indexList and can change this list (LinkedHashMap<String,String>) |


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

if the script is a FinderClass postScript you can find all indexname/indexvalues in the LinkedHashMap named indexList.

"Validate" scripts also receive the text to be checked in the variable "content" and must always return a boolean value.

##### Example

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
                    <code>
                        <![CDATA[
                            return true;
                        ]]>
                    <code>
                </Script>
                
            </Validate>

        </Scripts>
        
    </FileGazerFinderConfig>

### PDF/A validation

If the file is a PDF document, the document is checked against the PDF/A specification. 
in Addition the inspector checks, if the PDF is linerized PDF (Fast-Web-View).


### PDF Convert

This inspector convert the given file/document to PDF/A. This is based on the service of Gotenberg, which is not part of filegazer and must be available as a service (by IP/port). 
Please find more informations about Goteneberg here: https://gotenberg.dev/
Thanks to that great peace of software :-) 

files taht can be converted by this inspector: 

.123 .602 .abw .bib .bmp .cdr .cgm .cmx .csv .cwk .dbf .dif .doc .docm .docx .dot .dotm .dotx .dxf .emf .eps .epub .fodg .fodp .fods .fodt .fopd .gif .htm .html .hwp .jpeg .jpg .key .ltx .lwp .mcw .met .mml .mw .numbers .odd .odg .odm .odp .ods .odt .otg .oth .otp .ots .ott .pages .pbm .pcd .pct .pcx .pdb .pdf .pgm .png .pot .potm .potx .ppm .pps .ppt .pptm .pptx .psd .psw .pub .pwp .pxl .ras .rtf .sda .sdc .sdd .sdp .sdw .sgl .slk .smf .stc .std .sti .stw .svg .svm .swf .sxc .sxd .sxg .sxi .sxm .sxw .tga .tif .tiff .txt .uof .uop .uos .uot .vdx .vor .vsd .vsdm .vsdx .wb2 .wk1 .wks .wmf .wpd .wpg .wps .xbm .xhtml .xls .xlsb .xlsm .xlsx .xlt .xltm .xltx .xlw .xml .xpm .zabw


By setting applicationparameter you can activate this service by setting "filegazer.inspector.pdfconvert.active" to "true"
The service is set to active by default! If do not need/use the service please set this parameter to false to increase processing performance. 


Other Parameter to control that service: 

    # PDF convert: url for the convertion serice (this setting for a docker-compose setup)
    filegazer.inspector.pdfconvert.url=http://gotenberg:3000


    # possible: PDF/A-1b PDF/A-2b PDF/A-3b
    filegazer.inspector.pdfconvert.pdfa.format=PDF/A-1b

    # https://de.wikipedia.org/wiki/PDF/UA
    filegazer.inspector.pdfconvert.pdfua=true


## ReST Services

FileGazer provide three REST endpoint. One to inspect a file, one to trigger a script execution and the last to search for files. 

### Inspect a file

You can inspect a file with this endpoint. 

    http://myServer:Port/file/inspect

use

    http://myServer:Port/file/inspect/local

to inspect a file which filegazer can find localy

or 

    http://myServer:Port/file/inspect/upload

if you want to upload a file to filegazer for inspection. 

The following parameters are required with the corresponding end point: 

| Parameter | Description | Example |
| ----- | ----- | ----- |
| inspector | Which inspectors should examine the file. The names of the inspectors are separated by commas and are usually given in capital letters | file/inspect/local?inspector=TIKA,CONTENTANALYSE,BARCODE |
| format | In which format should the result be returned? XML or JSON | - | 
| transformation | If it is an XML return, the xslt script can be specified here that transforms the result of the inspector accordingly. You can also specify several xx at this point, which are then executed in the specified order (like a Unix pipe). Seperation is [,] | transformation=getClassification |

In the case of a "local" call, the name (plus path) of the file will be specified as POST workload. In the case of an upload, a MultipartFile is expected. 

| | |
| ----- | ----- |
| Method | POST |
| Endpoint | http://localhost:8080/file/inspect/{local or upload}? | 
| Parameter [format] |  Result format "xml" or "json"     |
| Parameter [inspector] |  list (seperator is ",") of inspectornames to inspect the given file. BASE,BARCODE,HASHCODE,MAGICNUMBER,TIKA,CONTENTANALYSE,PDFAANALYSER  |
| Parameter [transformation] |  xslt script (in ./etc/xlst) to transform the inspector-result |
| Workload/Body | Filename (inlude path) to inspect or MultipartFile |


FileGazer expect the fileName of the xslt stylesheet without the .xslt suffix and search the file in the current directory (of FileGazer)

    curl --location --request POST 'http://localhost:8080/file/inspect/local/inspector=TIKA,CONTENTANALYSE,BARCODE&transformation=inspect2docProcessing' --data-raw '/home/filegazer/ExampleFiles/Picture.jpg'

Filegazer search a file named "/home/filegazer/ExampleFiles/Picture.jpg" run the inspectors TIKA, CONTENTANALYSE and BARCODE and transform the xml output with the script inspect2docProcessing.xslt (which you can find in ./etc/xslt).


#### Curl-example:

get all available information for a file (default format is xml)

    curl --location --request POST 'http://localhost:8080/file/inspect/local --data-raw '/home/filegazer/ExampleFiles/Picture.jpg'

just read the barcode

    curl --location --request POST 'http://localhost:8080/file/inspect/local?inspector=BARCODE --data-raw '/home/filegazer/ExampleFiles/Picture.jpg'

read hascodes and content (Tika) and transform the result by a xslt script

    curl --location --request POST 'http://localhost:8080/file/inspect/local?inspector=HASHCODE,TIKA&transformation=myXSLTScript --data-raw '/home/filegazer/ExampleFiles/Picture.jpg'


### Execute scripts

This endpoint is only used to execute a FilegazerScript (these are listed in the ./etc/FileGazerScripts.xml file). 
A script can only be executed through this endpoint if the attribute "isRestExecute" in FileGazerScripts.xml has been set to true. 

|  |   |
| ----- | ----- |
| Method | GET |
| Endpoint | http://localhost:8080/execute/{scriptname} | 

"scriptname" is the name of the script which you want to execute - it must be listed in "FileGazerScripts.xml" and set to isRestExecute=true

### Search files

With 

     http://localhost:8080/search/directory/template

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



| Method | POST |
| ----- | ----- |
| Endpoint | http://localhost:8080/search/directory |
| Workload/Body | Searchrequest  |

### REST Services for configuration

Filegazer has some REST services that make it possible to perform the setup/config even if no direct access to the host file system is available. 
Essentially, these are services that can be used to read and modify the two configuration files 
"FileGazerScripts.xml" and "FileGazerContentAnalyse.xml" and to manage the xslt files and script files.
You also can 'load/flush' a certain version of FileGazerContentAnalyse.xml before you call the service to run your individuell setup (analysing).

Please find the parameter "filegazer.config.rest.active" in appalication.properties to switch this service on or off. By default the service is active.

#### contentanalyse

Endpoint: http://localhost:8080/config/contentanalyse


| RequestType | Description |
| --- | --- |
| GET | read content of the file FileGazerContentAnalyse.xml  |
| PUT | set content of the file FileGazerContentAnalyse.xml |

#### scripts

Endpoint: http://localhost:8080/config/scripts

| RequestType | Description |
| --- | --- |
| GET | read content of file FileGazerScripts.xml |
| PUT | set content of file FileGazerScripts.xml |

#### path

Endpoint: http://localhost:8080/setup/{path}

| RequestType | Description |
| --- | --- |
| GET | List all files in these directories. path - xslt or scripts |

#### path/filename

Endpoint: http://localhost:8080/setup/{path}/{filename}

| RequestType | Description |
| --- | --- |
| GET | read content of specified file |
| PUT | set contents of the specified file |
| DELETE | delete file |

## Filegazer Scripting

With the FileGazer scripts, the functions of FileGazer can be combined in scripts and executed locally. These scripts are formulated in groovy and can be as complex as you like. By default, the scripts are stored in ./etc/scripts. Any java libraries that may be required should be stored under ./lib. 

### FileGazerScripts.xml

In this file, all scripts that are prepared for execution are defined. 

Example: 
<FileGazerScriptsConfig>

    <FileGazerScripts>

        <!-- 
            Startup and shutdown scripts
        -->
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
            Scheduled script
        -->
        <FileGazerScript name="getTheAnswer" description="This script return the answer to life the universe and everything " onStartup="false" onExit="false" onTimer="0 * * ? * *" isFileLink="true" isRestExecute="true">
            TheAnswer.groovy
        </FileGazerScript>



        <!-- 
                Example scripts ... 
        -->
        <FileGazerScript name="Doc2txt" description="Pick up all files in /Example, get content from filegazer and create textfile" onStartup="false" onExit="false" onTimer="" isFileLink="true" isRestExecute="true">
            Doc2txt.groovy
        </FileGazerScript>
        <FileGazerScript name="DocCategorization" description="Pick up all filesin /Example, ask filegazer for the documenttype and move this file to  ./Example/{documenttype}" onStartup="false" onExit="false" onTimer="" isFileLink="true" isRestExecute="true">
            DocCategorization.groovy
        </FileGazerScript>
        <FileGazerScript name="CreateIndexControlFile" description="Pick up all filesin /Example, ask filegazer for detailinformation and create a html file to view the metadata and the file as well (just for pdf documents)" onStartup="false" onExit="false" onTimer="" isFileLink="true" isRestExecute="true">
            CreateIndexControlFile.groovy
        </FileGazerScript>


        <!-- 
            A script which execute other scripts
        -->
        <FileGazerScript name="ScriptBatchExample" description="A groovy script, which run a list of script one by one " onStartup="false" onExit="false" onTimer="" isFileLink="true" isRestExecute="true">
            ExecuteAllScripts.groovy
        </FileGazerScript>
            
        <!-- 
            Scripts which define a onBefore and/or onAfter script - a script-chain. Be careful with circle definitions!!!!
        -->
        <FileGazerScript name="ScriptBefore" description="A 'Before' Example" onStartup="false" onExit="false" onTimer="" isFileLink="false" isRestExecute="true">
            <![CDATA[
                println "##################################"
                println "#  Before-Script                 #"
                println "##################################"
            ]]>
        </FileGazerScript>
        <FileGazerScript name="ChainScriptTest" description="This script shows, how to call a script before and after execution" 
                                        onStartup="false" 
                                        onExit="false"
                                        onBefore="ScriptBefore"
                                        onAfter="ScriptAfter1,ScriptAfter2"
                                        onTimer="" 
                                        isFileLink="false" 
                                        isRestExecute="true">

            <![CDATA[
                println "##################################"
                println "#  Chain-Script                  #"
                println "##################################"
            ]]>
        </FileGazerScript>            
        <FileGazerScript name="ScriptAfter1" description="This script is a called after execution " onStartup="false" onExit="false" onTimer="" isFileLink="false" isRestExecute="true">
            <![CDATA[
                println "##################################"
                println "#  After-Script I                #"
                println "##################################"
            ]]>
        </FileGazerScript>            
        <FileGazerScript name="ScriptAfter2" description="This script is a called after execution" onStartup="false" onExit="false" onTimer="" isFileLink="false" isRestExecute="true">
            <![CDATA[
                println "##################################"
                println "#  After-Script II               #"
                println "##################################"
            ]]>
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
| onBefore | list of scripts (separator is ',' oder ';') which are executed right before this script |
| onAfter | list of scripts (separator is ',' oder ';') which are executed after this script |
| onTimer | cron String (0 6,18 * * *) that specifies when the script should be executed. |
| isLink | {true,false} - is it an embedded code (false) or the name of a script (true)? |
| isRestExecute | {true,false} - may the script be called via the Rest API? Example: curl http://localhost:8080/execute/Doc2txt |

If a FileGazerScript is started by the scheduler, it is ensured that a script is only executed once at any one time. On the other hand, different scripts can be executed in parallel.

## Installation, setup and starting

In order to run FileGazer you need at least

* Windows 7...11 or Linux or MacOS or a Docker Host
* Java 21
* local Tesseract installation (see below)

**You can find more infos here: https://filegazer.eichelsheimer.de**

### Setup FileGazer 

Just copy the current version of Filegazer to your host and run

    java -jar filegazer-xxxx.jar 
	
After the first start FileGazer create a new etc-directory (if not exists) and store some example configurationsfiles there. 

#### Start-Batch (Example)

    @echo off
    mode con:cols=250 
    rem
    rem Filegazer starter....
    rem

    rem . 
    rem basic parameter :-) 
    rem .
    set JAVA_HOME=d:/Programme/filegazer/java/jdk-21
    set FILEGAZER_HOME=d:/Programme/filegazer
    set VERSION=1.0.1


    set FILEGAZER_JAR=filegazer-%VERSION%.jar
    set FILEGAZER_MAIN=de.samoak.filegazer.FilegazerApplication
    set DIR_LIB=%FILEGAZER_HOME%/bin/lib

    rem Java VM settings...
    set MEMORY=-Xms1024m -Xmx2048m

    rem FileGazer Script-Directory
    set ETC=--filegazer.etc.scripts=%FILEGAZER_HOME%/etc/scripts

    rem FileGazer parameter (overload from application.properties)
    set PARAMETER=--server.port=9090 --filegazer.inspector.tika.skipocr=false --filegazer.inspector.barcode.max.pages=3 --debug=false --logging.level.com=DEBUG

    cd %FILEGAZER_HOME%
    echo .
    echo java %MEMORY% -cp %FILEGAZER_JAR% -Dloader.path=%DIR_LIB% -Dloader.main=%FILEGAZER_MAIN% org.springframework.boot.loader.PropertiesLauncher
    echo . 
    echo starting...
    echo . 

    %JAVA_HOME%\bin\java  %MEMORY% -cp %FILEGAZER_JAR% -Dloader.path=%DIR_LIB% -Dloader.main=%FILEGAZER_MAIN% org.springframework.boot.loader.PropertiesLauncher %PARAMETER% %ETC%


I think it is easy to "translate" this script to a unix-shell script. 

#### Install a windows service

There are various methods for installing Filegazer or a Java application as a service under Windows. I refer here to "WinSW" (https://github.com/winsw/winsw).

Cookbook:

* Copy WinSW-x64.exe into the directory where filegazer-xxxx.jar is located. For example: D:\filegazer
* Rename "WinSW-x64.exe" to "FileGazer.exe".
* Create the Filegazer.xml (example below)
* Install the serviceuxileGazer stop
* Remove the service: Filegazer uninstall

Example of a WinSW configuration file:

    <service>
        <id>Filegazer</id>
        <name>FileGazer</name>
        <description>FileGazer Service</description>
        <workingdirectory>D:\filegazer</workingdirectory>
        <priority>normal</priority>
        <executable>c:\Program Files\Java\jdk-17.0.1\bin\java.exe</executable>
        <arguments>-Xrs -Xms1024m -Xmx2048m -jar filegazer-1.0.1.jar -Dloader.path=D:\filegazer\lib -Dloader.main=de.samoak.filegazer.FilegazerApplication org.springframework.boot.loader.PropertiesLauncher</arguments>
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



| Attribut | default | description |
| ----- | ------ | ----- | 
| filegazer.autosetup.active | true | If true, FileGazer checks whether the etc directory exists. If not, all directories are created, the (default) configuration files, the documentation and the examples are extracted | 
| filegazer.ui.open | true | If true, the web frontend can be used |
| filegazer.ui.show.open | true | Do you want to present the open (inspect) button? |
| filegazer.ui.show.search | true | Do you want to present the search button? |
| filegazer.config.rest.active | true | activate/deactivate the config Rest Services |
| filegazer.etc | ./etc | path for configurationfiles | 
| filegazer.etc.xslt | ./etc/xslt | Where to find all xslts?  |
| filegazer.etc.scripts | ./etc/scripts | If there is no path given in "FileGazerScripts.xml", FileGazer try to find find scripts here |
| filegazer.scripts.configfile | FileGazerScripts.xml | config file for scripting ... relative to "filegazer.etc" | 
| filegazer.transformation.filegazerfile2Json | FileGazer2Json.xsl | Transformationscript which convert the default xml result to a jsonformat |
| filegazer.inspector.barcode.max.pages | -1 | set to the number of pages (start at page 1) you want to inspect. -1 means: all pages | 
| filegazer.inspector.barcode.try.harder | true | set to true get better results. set to false for faster reading | 
| filegazer.inspector.barcode.content.just.ascii | true | if true just content with characters >=32 accepted | 
| filegazer.inspector.barcode.types | | Give a list with barcodes you want to read. empty means all. List:AZTEC,CODABAR,CODE_39,CODE_93,CODE_128,DATA_MATRIX,EAN_8,EAN_13,ITF,MAXICODE,PDF_417,QR_CODE,RSS_14,RSS_EXPANDED,UPC_A,UPC_E,UPC_EAN_EXTENSION |
| filegazer.inspector.base.filecontent.include | true | should be the uuencoded filecontent be part of the base inspector. if you do not need this and you need more performance switch this of | 
| filegazer.inspector.tika.contentsize | -1 | How much (bytes) Content should we read (-1 means "all") | 
| filegazer.inspector.tika.pdfbox.minsize | 200 | If PDFbox found content ... it must be minimum this value - otherwise it is "no content" and FileGazer runs a OCR |
| filegazer.inspector.tika.skipocr | false | Should FileGazer start a local installed OCR (tesseract) | 
| filegazer.inspector.tika.ocr.language | deu |
| filegazer.inspector.tika.ocr.density | 600 |
| filegazer.inspector.tika.ocr.timeout | 300 |
| filegazer.inspector.tika.ocr.pagesegmode | 1 |
| filegazer.inspector.tika.ocr.pageseparator |  |
| filegazer.inspector.tika.ocr.preserveinterwordspacing | false |
| filegazer.inspector.tika.ocr.depth | 4 |
| filegazer.inspector.tika.ocr.colorspace | gray |
| filegazer.inspector.tika.meta.filter.nonascii | true | if true all non ascii character eliminate |
| filegazer.inspector.contentanalyse.config | FileGazerContentAnalyse.xml | Config for (text)content analysing ... file found relativ to "etc" directory | 



### Install tesseract (for OCR)

This section gives a short description "how to install tesseract".
A small wiki for Tika and tesseract here: https://cwiki.apache.org/confluence/display/tika/TikaOCR

#### Ubuntu
        
    sudo apt-get update
    sudo apt-get install tesseract-ocr
    ux
May you have to install a language-pack: 

    sudo apt-get install tesseract-ocr-deu

(Replace "deu" with you country code)ux

#### macOS

    brew install tesseract tesseract-lang

#### Microsoft Windows

You find a windows setup and further instructions under: https://github.com/UB-Mannheim/tesseract/wiki

### Gotenberg

Gotenberg need a docker environment.

    docker run --rm -p 3000:3000 gotenberg/gotenberg:8

### Docker

The docker image is available on dockerhub

#### pull 

    docker pull samoak/filegazer:latest

#### start

    docker run -d -p 9090:8080 samoak/filegazer

#### Docker-compose

To start FileGazer and Gotenberg and future components please use the docker-compose-File. 

    version: '3.8'
    services:

    gotenberg:
        container_name: gotenberg
        image: docker.io/gotenberg/gotenberg:latest
        ports:
        - '3000:3000'
        restart: unless-stopped
        # The gotenberg chromium route is used to convert .eml files. We do not
        # want to allow external content like tracking pixels or even javascript.
        command:
        - "gotenberg"
        - "--chromium-disable-javascript=true"
        - "--chromium-allow-list=file:///tmp/.*"
        healthcheck:
        test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
        interval: 10s
        timeout: 5s
        retries: 5

    filegazer:
        depends_on:
        gotenberg:
            condition: service_healthy
        container_name: filegazer
        image: samoak/filegazer:latest
        ports:
        - '9090:8080'
        restart: unless-stopped
        healthcheck:
        test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
        interval: 10s
        timeout: 5s
        retries: 5

    networks:
    default:
        name: filegazer_net

## Links and further information 

### Tika 
    
- https://tika.apache.org/

### Tesseract 

- https://github.com/tesseract-ocr/tesseract

### magic number

- https://en.wikipedia.org/wiki/List_of_file_signatures
- https://www.garykessler.net/library/file_sigs.html

### cron expression

There are some online tools to create cron expressions. Here is an example

- https://www.freeformatter.com/cron-expression-generator-quartz.html

### PDF/A

https://www.pdfa.org

https://verapdf.org


### Gotenberg project

- https://gotenberg.dev/

## Beyond

What is the issues I'll work on next ? 

- Bufixing (may with your help)
- An AI inspector 

## Help & contact...

- If you think FileGazer is helpful, please let me know :-) ... 
- If have an idea for a new inspector, please let mew know :-) ...
- If have a problem or a question, please let mew know :-) ...
- If you found an error, please let mew know :-) ...

In one word: feel free to contact me... I am looking forward


