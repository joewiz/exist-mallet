exist-mallet-topic-modeling
===========================

Integrates the Mallet Machine Learning and Topic Modeling library into eXist-db.

## Compile and install

1. clone the github repository: https://github.com/ljo/exist-mallet
2. edit local.build.properties and set exist.dir to point to your eXist-db install directory
3. call "ant" in the directory to create a .xar
4. upload the xar into eXist-db using the dashboard

## Functions

There are currently three main function groups:

### topics:create-instances-*
Processes resources in the provided collection hierarchy and creates a serialized instances document which can be used by nearly all Mallet sub-packages. Returns the path to the stored instances document.
The parameter $configuration gives the configuration, eg &lt;parameters&gt;&lt;param name='stopwords' value='false'/&gt;&lt;param name='langugage' value='en'/&gt;&lt;/parameters&gt;.  For polylingual you use the $collection-uris parameter with one or more URIs. If only one collection-uri is given, sub-collections for each language code are expected, otherwise one collection-uri for each of the languages in the same order are expected.

topics:create-instances-collection($instances-doc as xs:anyURI, $collection-uri as xs:anyURI, $qname as xs:QName?) as xs:string?

topics:create-instances-collection($instances-doc as xs:anyURI, $collection-uri as xs:anyURI, $qname as xs:QName?, $configuration as element()) as xs:string?

topics:create-instances-collection-polylingual($instances-doc as xs:anyURI, $collection-uri as xs:anyURI, $qname as xs:QName?, $languages as xs:string+) as xs:string?

topics:create-instances-collection-polylingual($instances-doc as xs:anyURI, $collection-uris as xs:anyURI+, $qname as xs:QName?, $languages as xs:string+, $configuration as element()) as xs:string+

topics:create-instances-node($instances-doc as xs:anyURI, $node as node()+) as xs:string?

topics:create-instances-node($instances-doc as xs:anyURI, $node as node()+, $configuration as element()) as xs:string?

topics:create-instances-string($instances-doc as xs:anyURI, $text as xs:string+) as xs:string?

topics:create-instances-string($instances-doc as xs:anyURI, $text as xs:string+, $configuration as element()) as xs:string?

### topics:topic-model-inference / topics:polylingual-topic-model-inference
Processes new instances and applies the stored topic model's inferencer(s). Returns the topic probabilities for the inferenced instances.

topics:topic-model-inference($instances-doc as xs:anyURI, $number-of-words-per-topic as xs:integer, $number-of-topics as xs:integer, $number-of-iterations as xs:integer?, $number-of-threads as xs:integer?, $alpha_t as xs:double?, $beta_w as xs:double?, $language as xs:string?, $instances-inference-doc as xs:anyURI) as node()+

topics:topic-model-inference($topic-model-doc as xs:anyURI, $instances-inference-doc as xs:anyURI, $number-of-iterations as xs:integer, $thinning as xs:integer?, $burn-in as xs:integer?) as node()+

topics:polylingual-topic-model-inference($instances-doc as xs:anyURI, $number-of-words-per
-topic as xs:integer, $number-of-topics as xs:integer, $number-of-iterations as xs:integer?, $number-of-threads as xs:integer?, $alpha_t as xs:double?, $beta_w as xs:double?, $languages as xs:string+, $instances-inference-doc as xs:anyURI) as node()+

topics:polylingual-topic-model-inference($topic-model-doc as xs:anyURI, $instances-inference
-doc as xs:anyURI, $number-of-iterations as xs:integer, $thinning as xs:integer?, $burn-in as xs:integer?, $languages as xs:string+) as node()+

### topics:topic-model / topics:polylingual-topic-model
Processes instances and creates either a monolingual topic model or a polylingual topic model (PLTM) which can be used for inference. Returns the specified number of top ranked words per topic.

Note: currently the language parameter does very little. It guides formating of numbers and use of stopwords (not for polylingual) though. For PLTM it is very important to tune the parameters otherwise it might even kill the whole jvm. 

topics:topic-model($instances-doc as xs:anyURI, $number-of-words-per-topic as xs:integer, $number-of-topics as xs:integer, $number-of-iterations as xs:integer?, $number-of-threads as xs:integer?, $alpha_t as xs:double?, $beta_w as xs:double?, $language as xs:string?) as node()+

topics:polylingual-topic-model($instances-doc as xs:anyURI, $number-of-words-per-topic as xs
:integer, $number-of-topics as xs:integer, $number-of-iterations as xs:integer?, $number-of-threads as xs:integer?, $alpha_t as xs:double?, $beta_w as xs:double?, $languages as xs:string+) as node()+

### topics:topic-model-sample
Processes instances and creates a topic model which can be used for inference. Returns the specified number of top ranked words per topic. All other parameters use default values. Runs the model for 50 iterations and stops (this is for testing only, for real applications, use 1000 to 2000 iterations).

topics:topic-model-sample($instances-doc as xs:anyURI) as node()+

topics:topic-model-sample($instances-doc as xs:anyURI, $number-of-words-per-topic as xs:integer, $language as xs:string?) as node()+


## Usage example, monolingual

```xquery
xquery version "3.0";
import module namespace tm="http://exist-db.org/xquery/mallet-topic-modeling";
declare namespace tei="http://www.tei-c.org/ns/1.0";

let $text := 
("This is a test in English for the eXist-db@XML Prague preconference day. 
A subject as good as any. So what subjects will be chosen to label this text? ", 
"Can eXist-db really tell the subjects? Let us see now when we give two strings as arguments. ")
let $text2 := (<text>{$text[1]}</text>, <text>{$text[2]}</text>)
let $text3 := xs:anyURI("/db/dramawebben/data/works")
let $instances-doc-suffix := ".mallet"
let $topic-model-doc-suffix := ".tm"
let $create-collections := (xmldb:create-collection('/db', 'temp'), xmldb:create-collection('/db/temp', 'topic-example'))
let $instances-doc-prefix := "/db/temp/topic-example"
let $instances-path := $instances-doc-prefix || $instances-doc-suffix
let $instances-path2 := $instances-doc-prefix || "2" || $instances-doc-suffix
let $instances-path3 := $instances-doc-prefix || "3" || $instances-doc-suffix

let $mode := 1
let $call-type := ("string", "node", "collection")[$mode]
let $instances-uri := xs:anyURI(($instances-path, $instances-path2, $instances-path3)[$mode])
let $topic-model-uri := xs:anyURI(($instances-path || $topic-model-doc-suffix, $instances-path2 || $topic-model-doc-suffix, $instances-path2 || $topic-model-doc-suffix)[$mode])

(: Make sure you create an instance for the mode you use. :)
let $create-instances-p := true()
(: Please note this is an example configuration. :)
let $config := 
    <parameters>
        <param name="stopwords" value="true"/>
        <param name="language" value="sv"/>
        <param name="useStored" value="true"/>
        <param name="showWordLists" value="true"/>
</parameters>
let $created := if ($create-instances-p) then 
    switch ($call-type)
        case "string" return tm:create-instances-string($instances-uri, $text, $config)
        case "node" return tm:create-instances-node($instances-uri, $text2, $config)
        case "collection" return tm:create-instances-collection($instances-uri, $text3, xs:QName("tei:body"), $config)
        default return tm:create-instances-string($instances-uri, $text)
    else ()
return 
    if ($create-instances-p) then
        tm:topic-model($instances-uri, 5, 15, 50, (), (), (), "sv")
	(: tm:topic-model-inference($instances-uri, 5, 15, 50, (), (), (), "sv", $instances-uri) :)
    else
        tm:topic-model-inference($topic-model-uri, $instances-uri, 50, (), ())
```

## Expected results, monolingual

```xml
<topics:topicModel xmlns:topics="http://exist-db.org/xquery/mallet-topic-modeling">
    <topics:topic n="0" alpha="0.01" totalTokens="0"/>
    <topics:topic n="1" alpha="0.01" totalTokens="3">
        <topics:token rank="1">give</topics:token>
        <topics:token rank="2">subjects</topics:token>
        <topics:token rank="3">exist-db</topics:token>
    </topics:topic>
    <topics:topic n="2" alpha="0.01" totalTokens="0"/>
    <topics:topic n="3" alpha="0.01" totalTokens="0"/>
    <topics:topic n="4" alpha="0.01" totalTokens="3">
        <topics:token rank="1">arguments</topics:token>
        <topics:token rank="2">good</topics:token>
        <topics:token rank="3">english</topics:token>
    </topics:topic>
    <topics:topic n="5" alpha="0.01" totalTokens="0"/>
    <topics:topic n="6" alpha="0.01" totalTokens="2">
        <topics:token rank="1">text</topics:token>
        <topics:token rank="2">subject</topics:token>
    </topics:topic>
    <topics:topic n="7" alpha="0.01" totalTokens="0"/>
    <topics:topic n="8" alpha="0.01" totalTokens="0"/>
    <topics:topic n="9" alpha="0.01" totalTokens="0"/>
    <topics:topic n="10" alpha="0.01" totalTokens="5">
        <topics:token rank="1">strings</topics:token>
        <topics:token rank="2">label</topics:token>
        <topics:token rank="3">subjects</topics:token>
        <topics:token rank="4">preconference</topics:token>
        <topics:token rank="5">prague</topics:token>
    </topics:topic>
    <topics:topic n="11" alpha="0.01" totalTokens="0"/>
    <topics:topic n="12" alpha="0.01" totalTokens="2">
        <topics:token rank="1">xml</topics:token>
        <topics:token rank="2">exist-db</topics:token>
    </topics:topic>
    <topics:topic n="13" alpha="0.01" totalTokens="3">
        <topics:token rank="1">chosen</topics:token>
        <topics:token rank="2">day</topics:token>
        <topics:token rank="3">test</topics:token>
    </topics:topic>
    <topics:topic n="14" alpha="0.01" totalTokens="0"/>
</topics:topicModel>

<topics:wordLists xmlns:topics="http://exist-db.org/xquery/mallet-topic-modeling">
    <topics:wordList n="0">
        <topics:token normalized-form="test" topic="13"/>
        <topics:token normalized-form="english" topic="4"/>
        <topics:token normalized-form="exist-db" topic="12"/>
        <topics:token normalized-form="xml" topic="12"/>
        <topics:token normalized-form="prague" topic="10"/>
        <topics:token normalized-form="preconference" topic="10"/>
        <topics:token normalized-form="day" topic="13"/>
        <topics:token normalized-form="subject" topic="6"/>
        <topics:token normalized-form="good" topic="4"/>
        <topics:token normalized-form="subjects" topic="10"/>
        <topics:token normalized-form="chosen" topic="13"/>
        <topics:token normalized-form="label" topic="10"/>
        <topics:token normalized-form="text" topic="6"/>
    </topics:wordList>
    <topics:wordList n="1">
        <topics:token normalized-form="exist-db" topic="1"/>
        <topics:token normalized-form="subjects" topic="1"/>
        <topics:token normalized-form="give" topic="1"/>
        <topics:token normalized-form="strings" topic="10"/>
        <topics:token normalized-form="arguments" topic="4"/>
    </topics:wordList>
</topics:wordLists>
```

## Usage example, polylingual topic modeling (PLTM)

```xquery
xquery version "3.0";
import module namespace tm="http://exist-db.org/xquery/mallet-topic-modeling";
declare namespace tei="http://www.tei-c.org/ns/1.0";
let $languages := ("sv", "en")
let $text-sv := 
("Det här är en text för förkonferensdagen@XML-Prag. 
Ett ämne gott som något. Så vilket ämne kommer att tilldelas som etikett för den här texten? ", 
"Kan eXist-db verkligen skilja på ämnena? Vad händer om vi ger två strängar som argument? ")
let $text-en := ("This is a test for the eXist-db@XML Prague preconference day. 
A subject as good as any. So what subjects will be chosen to label this text? ", 
"Can eXist-db really tell the subjects? Let us see now when we give two strings as arguments. ")
let $text2 := (<text><text>{$text-sv[1]}</text><text>{$text-sv[2]}</text></text>, 
               <text><text>{$text-en[1]}</text><text>{$text-en[2]}</text></text>)
let $text3 := xs:anyURI("/db/dramawebben/data/works")
let $instances-doc-suffix := ".mallet"
let $topic-model-doc-suffix := ".pltm"

let $create-collections := (xmldb:create-collection('/db', 'temp'), xmldb:create-collection('/db/temp', 'topic-example'))
let $instances-doc-prefix := "/db/temp/topic-example"
let $instances-path := $instances-doc-prefix || $instances-doc-suffix
let $instances-path2 := $instances-doc-prefix || "2" || $instances-doc-suffix
let $instances-path3 := $instances-doc-prefix || "3" || $instances-doc-suffix

(: Mode 3 specific instance creation is available for PLTM 
   tm:create-instances-collection-polylingual($instance-uri, $collection-uris, xs:QName("tei:body"), $languages)
   tm:create-instances-collection-polylingual($instance-uri, $collection-uris, xs:QName("tei:body"), $languages, $config)
   If only one collection-uri is given, sub-collections for each language code are expected,
   otherwise one collection-uri for each of the languages in the same order are expected.
:)
let $mode := 2
let $call-type := ("string", "node", "collection")[$mode]
let $instances-uri := xs:anyURI(($instances-path, $instances-path2, $instances-path3)[$mode])
let $instances-uris := for $lang in $languages return xs:anyURI(concat(($instances-path, $instances-path2, $instances-path3)[$mode], ".", $lang))
let $topic-model-uri := xs:anyURI(($instances-path || $topic-model-doc-suffix, $instances-path2 || $topic-model-doc-suffix, $instances-path2 || $topic-model-doc-suffix)[$mode])

(: Make sure you create instances for the mode you use at least once. :)
let $create-instances-p := true()
(: Please note this is an example configuration. :)
let $config := 
    <parameters>
        <param name="stopwords" value="true"/>
        <param name="useStored" value="false"/>
        <param name="showWordLists" value="true"/>
    </parameters>
let $created := if ($create-instances-p) then 
    switch ($call-type)
        case "string" return for $instance-uri at $pos in $instances-uris return tm:create-instances-string($instance-uri, ($text-sv, $text-en)[$pos], $config)
        case "node" return for $instance-uri at $pos in $instances-uris return tm:create-instances-node($instance-uri, $text2[$pos]/text, $config)
        case "collection" return tm:create-instances-collection-polylingual($instances-uri, $text3, xs:QName("tei:body"), $languages, $config)
        default return for $instance-uri at $pos in $instances-uri return tm:create-instances-string($instance-uri, ($text-sv, $text-en)[$pos], $config)
    else ()
return 
    if ($create-instances-p) then
        tm:polylingual-topic-model($instances-uri, 5, 5, 50, (), (), (), $languages)
        (:tm:polylingual-topic-model-inference($instances-uri, 10, 25, 750, (), (), (), $languages, $instances-uri) :)
    else
        tm:polylingual-topic-model-inference($topic-model-uri, $instances-uri, 750, (), (), $languages)
    (: tm:polylingual-topic-model($instances-uri, 5, 25, 50, (), (), (), ("sv")) :)
```

## Expected results, polylingual topic modeling (PLTM)

```xml
<topics:topicModel xmlns:topics="http://exist-db.org/xquery/mallet-topic-modeling">
    <topics:topic n="0" alpha="1269.651281779051">
        <topics:language name="sv" totalTokens="12" beta="0.05997365217342881">
            <topics:token rank="1">ämne</topics:token>
            <topics:token rank="2">här</topics:token>
            <topics:token rank="3">strängar</topics:token>
            <topics:token rank="4">vad</topics:token>
            <topics:token rank="5">ämnena</topics:token>
        </topics:language>
        <topics:language name="en" totalTokens="3" beta="0.13893142897441135">
            <topics:token rank="1">subjects</topics:token>
            <topics:token rank="2">prague</topics:token>
            <topics:token rank="3">test</topics:token>
        </topics:language>
    </topics:topic>
    <topics:topic n="1" alpha="1540.3322053077632">
        <topics:language name="sv" totalTokens="8" beta="0.05997365217342881">
            <topics:token rank="1">om</topics:token>
            <topics:token rank="2">verkligen</topics:token>
            <topics:token rank="3">den</topics:token>
            <topics:token rank="4">vilket</topics:token>
            <topics:token rank="5">något</topics:token>
        </topics:language>
        <topics:language name="en" totalTokens="4" beta="0.13893142897441135">
            <topics:token rank="1">strings</topics:token>
            <topics:token rank="2">text</topics:token>
            <topics:token rank="3">chosen</topics:token>
            <topics:token rank="4">preconference</topics:token>
        </topics:language>
    </topics:topic>
    <topics:topic n="2" alpha="976.447795420123">
        <topics:language name="sv" totalTokens="7" beta="0.05997365217342881">
            <topics:token rank="1">ger</topics:token>
            <topics:token rank="2">händer</topics:token>
            <topics:token rank="3">på</topics:token>
            <topics:token rank="4">skilja</topics:token>
            <topics:token rank="5">att</topics:token>
        </topics:language>
        <topics:language name="en" totalTokens="3" beta="0.13893142897441135">
            <topics:token rank="1">exist-db</topics:token>
            <topics:token rank="2">good</topics:token>
        </topics:language>
    </topics:topic>
    <topics:topic n="3" alpha="954.1483294895851">
        <topics:language name="sv" totalTokens="6" beta="0.05997365217342881">
            <topics:token rank="1">argument</topics:token>
            <topics:token rank="2">vi</topics:token>
            <topics:token rank="3">exist-db</topics:token>
            <topics:token rank="4">så</topics:token>
            <topics:token rank="5">xml-prag</topics:token>
        </topics:language>
        <topics:language name="en" totalTokens="3" beta="0.13893142897441135">
            <topics:token rank="1">subjects</topics:token>
            <topics:token rank="2">subject</topics:token>
            <topics:token rank="3">xml</topics:token>
        </topics:language>
    </topics:topic>
    <topics:topic n="4" alpha="1206.0944789971775">
        <topics:language name="sv" totalTokens="7" beta="0.05997365217342881">
            <topics:token rank="1">som</topics:token>
            <topics:token rank="2">två</topics:token>
            <topics:token rank="3">texten</topics:token>
            <topics:token rank="4">etikett</topics:token>
            <topics:token rank="5">kommer</topics:token>
        </topics:language>
        <topics:language name="en" totalTokens="4" beta="0.13893142897441135">
            <topics:token rank="1">arguments</topics:token>
            <topics:token rank="2">give</topics:token>
            <topics:token rank="3">label</topics:token>
            <topics:token rank="4">day</topics:token>
        </topics:language>
    </topics:topic>
</topics:topicModel>

<topics:documentTopics xmlns:topics="http://exist-db.org/xquery/mallet-topic-modeling">
    <topics:document n="0">
        <topics:topic ref="0" weight="0.29729729890823364"/>
        <topics:topic ref="1" weight="0.2432432472705841"/>
        <topics:topic ref="4" weight="0.18918919563293457"/>
        <topics:topic ref="3" weight="0.13513512909412384"/>
        <topics:topic ref="2" weight="0.13513512909412384"/>
    </topics:document>
    <topics:document n="1">
        <topics:topic ref="2" weight="0.25"/>
        <topics:topic ref="4" weight="0.20000000298023224"/>
        <topics:topic ref="3" weight="0.20000000298023224"/>
        <topics:topic ref="0" weight="0.20000000298023224"/>
        <topics:topic ref="1" weight="0.15000000596046448"/>
    </topics:document>
</topics:documentTopics>

<topics:wordLists xmlns:topics="http://exist-db.org/xquery/mallet-topic-modeling">
    <topics:language name="sv">
        <topics:wordList n="0">
            <topics:token normalized-form="det" topic="1"/>
            <topics:token normalized-form="här" topic="0"/>
            <topics:token normalized-form="är" topic="0"/>
            <topics:token normalized-form="en" topic="2"/>
            <topics:token normalized-form="text" topic="2"/>
            <topics:token normalized-form="för" topic="1"/>
            <topics:token normalized-form="förkonferensdagen" topic="0"/>
            <topics:token normalized-form="xml-prag" topic="3"/>
            <topics:token normalized-form="ett" topic="0"/>
            <topics:token normalized-form="ämne" topic="0"/>
            <topics:token normalized-form="gott" topic="1"/>
            <topics:token normalized-form="som" topic="4"/>
            <topics:token normalized-form="något" topic="1"/>
            <topics:token normalized-form="så" topic="3"/>
            <topics:token normalized-form="vilket" topic="1"/>
            <topics:token normalized-form="ämne" topic="0"/>
            <topics:token normalized-form="kommer" topic="4"/>
            <topics:token normalized-form="att" topic="2"/>
            <topics:token normalized-form="tilldelas" topic="0"/>
            <topics:token normalized-form="som" topic="4"/>
            <topics:token normalized-form="etikett" topic="4"/>
            <topics:token normalized-form="för" topic="3"/>
            <topics:token normalized-form="den" topic="1"/>
            <topics:token normalized-form="här" topic="0"/>
            <topics:token normalized-form="texten" topic="4"/>
        </topics:wordList>
        <topics:wordList n="1">
            <topics:token normalized-form="kan" topic="0"/>
            <topics:token normalized-form="exist-db" topic="3"/>
            <topics:token normalized-form="verkligen" topic="1"/>
            <topics:token normalized-form="skilja" topic="2"/>
            <topics:token normalized-form="på" topic="2"/>
            <topics:token normalized-form="ämnena" topic="0"/>
            <topics:token normalized-form="vad" topic="0"/>
            <topics:token normalized-form="händer" topic="2"/>
            <topics:token normalized-form="om" topic="1"/>
            <topics:token normalized-form="vi" topic="3"/>
            <topics:token normalized-form="ger" topic="2"/>
            <topics:token normalized-form="två" topic="4"/>
            <topics:token normalized-form="strängar" topic="0"/>
            <topics:token normalized-form="som" topic="4"/>
            <topics:token normalized-form="argument" topic="3"/>
        </topics:wordList>
    </topics:language>
    <topics:language name="en">
        <topics:wordList n="0">
            <topics:token normalized-form="test" topic="0"/>
            <topics:token normalized-form="exist-db" topic="2"/>
            <topics:token normalized-form="xml" topic="3"/>
            <topics:token normalized-form="prague" topic="0"/>
            <topics:token normalized-form="preconference" topic="1"/>
            <topics:token normalized-form="day" topic="4"/>
            <topics:token normalized-form="subject" topic="3"/>
            <topics:token normalized-form="good" topic="2"/>
            <topics:token normalized-form="subjects" topic="0"/>
            <topics:token normalized-form="chosen" topic="1"/>
            <topics:token normalized-form="label" topic="4"/>
            <topics:token normalized-form="text" topic="1"/>
        </topics:wordList>
        <topics:wordList n="1">
            <topics:token normalized-form="exist-db" topic="2"/>
            <topics:token normalized-form="subjects" topic="3"/>
            <topics:token normalized-form="give" topic="4"/>
            <topics:token normalized-form="strings" topic="1"/>
            <topics:token normalized-form="arguments" topic="4"/>
        </topics:wordList>
    </topics:language>
</topics:wordLists>
```
