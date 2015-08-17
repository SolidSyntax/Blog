title: Howto use JAXB class customization to avoid XSD conflicts
tags:
  - JAXB
  - xsd
id: 188
categories:
  - Lessons learned
date: 2013-12-07 12:46:59
---

While generating binding classes with JAXB I've recently encountered the following problem:

{% codeblock lang:propertie %}
parsing a schema...
compiling a schema...
[ERROR] A class/interface with the same name "net.solidsyntax.examples.jaxb.oxm.AnXSDType" is already in use. Use a class customization to resolve this conflict.
  line 15 of file:/C:/xsd/CommonTypes.xsd
[ERROR] (Relevant to above error) another "AnXSDType" is generated from here.
  line 10 of file:/C:/xsd/EvenMoreCommonTypes.xsd
[ERROR] Two declarations cause a collision in the ObjectFactory class.
  line 15 of file:/C:/xsd/CommonTypes.xsd
[ERROR] (Related to above error) This is the other declaration.
  line 10 of file:/C:/xsd/EvenMoreCommonTypes.xsd
Failed to produce code.
{% endcodeblock %}

The problem occurred while trying to generate binding classes from an xsd (MySchema.xsd) which included CommonTypes.xsd and EvenMoreCommonTypes.xsd. Both CommonTypes.xsd and EvenMoreCommonTypes.xsd contain a type called "AnXSDType". By default JAXB generates binding files with the same name as the type. Two source files with the same name in the same package is not possible so JAXB throws generates an exception.

How can we fix this ? 
We can use JAXB class customization to specify a custom name for the binding. In this case we want the "AnXSDType" from CommonTypes.xsd to generate a binding file named "AnXSDTypeFromCommonTypes" and the one from EvenMoreCommonTypes.xsd should generate a file called "AnXSDTypeFromEvenMoreCommonTypes".
In order to do so we need to create a custom bindings file (here called binding.xml)

{% codeblock lang:xml %}
<jxb:bindings 
    xmlns:xs="http://www.w3.org/2001/XMLSchema"
    xmlns:jxb="http://java.sun.com/xml/ns/jaxb"
    version="2.1">

    <jxb:bindings schemaLocation="xsd/CommonTypes.xsd">
        <jxb:bindings node="//xs:complexType[@name='AnXSDType']">
            <jxb:class name="AnXSDTypeFromCommonTypes"/>
        </jxb:bindings>
    </jxb:bindings>
    <jxb:bindings schemaLocation="xsd/EvenMoreCommonTypes.xsd">
        <jxb:bindings node="//xs:complexType[@name='AnXSDType']">
            <jxb:class name="AnXSDTypeFromEvenMoreCommonTypes"/>
        </jxb:bindings>
    </jxb:bindings>

</jxb:bindings>
{% endcodeblock %}

Then we tell xjc to use a class customization file to generate binding files with the -b argument:
{% codeblock lang:propertie %}
  xjc -d outputDir -b binding.xml MySchema.xsd
{% endcodeblock %}

This fixes the problem by generating two different files for the "AnXSDType".