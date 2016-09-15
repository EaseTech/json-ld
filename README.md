# Summary of where we are at
There currently exists a tension in the CTI TC community between parties on one side desiring to define and limit STIX/CybOX to a single JSON serialization in the interests of lowering the bar as far as possible for developers to drive adoption and on the other side desiring to define the STIX and CybOX languages in a model-based, semantically-consistent, technology-agnostic form that would provide inherent consistency and maintainability, would support advanced forms of analysis and would losslessly support a range of different serialization choices leaving the decision up to the implementer based on their context.
 
After much discussion it appears clear that the supporters of the model-based approach believe that it is possible to support both desires at the same time but the supporters of the JSON-based approach believe that the model-based approach will force implementers and end users to deal with model and semantic complexities even when they don't wish to.
 
## JSON-LD as a potential solution
JSON-LD has been raised as a potential option offering a solution to support both a full semantic approach for those who desire one and a limited JSON approach for those who do not. For those new to JSON-LD and its ontologically traceable approach this option has been somewhat confusing. Some have found themselves focused on a specific use case supported by JSON-LD (linked data across the internet) that they do not see as relevant to typical STIX/CybOX usage rather than the more general capabilities JSON-LD offers for general information representation, analysis, transformation and exchange. Others look at the default expanded RDF-graph-based form shown in many JSON-LD examples and do not see how it could be used in a simpler "pure" JSON fashion.
 
This brief writeup is an attempt to clarify some misconceptions and help illustrate how JSON-LD can be used for those who desire to represent STIX/CybOX as "pure" JSON without worrying about models or RDF or other serialization formats. This writeup does not attempt to present or discuss the details of how those desiring to take advantage of the myriad benefits of the richer semantic capabilities (model-based derivations, ontological mapping with 3rd-party ontologies, advanced graph-based analysis, lossless transformation between serializations) could do so. There appears to be less contention on those points so that conversation is reserved for another venue.
 
# Clearing up some misconceptions
**Myth:** JSON vs JSON-LD is an either-or choice.
 
**Fact:** JSON-LD IS JSON. It is just JSON whose content is conformant with a specified vocabulary for types and properties. 

* As a test to validate that the JSON-LD is valid JSON, you can run the examples through the following command: python –m json.tool <example_file>
 
***
 
**Myth:** Supporting JSON-LD in STIX/CybOX would put an undue burden on those interested in using only "pure" JSON and not in leveraging any of the advantages offered by the semantics of JSON-LD
 
**Fact:** JSON-LD can be supported and available for those who desire to leverage its semantic advantages without adding any complexity for producers or consumers wishing to only use "pure" JSON. 
 
The only additional requirements can be viewed as stylistic ones for JSON: 

* each field name would contain a short prefix
* **it should be noted that this is only a unique name for the field and does not involve any of the complex canonicalization required with XML namespace prefixes and places NO processing burden and only very minimal cognitive burden on "pure" JSON implementers** 
* each use of the field names 'id', 'type' and 'value' would contain a leading '@' (e.g. @id). 

**Fact:**  JSON-LD annotations can be specified in an external file so that only an '@context' line need be supplied:

* aliases for 'id', 'type', and 'value' can be defined so that no other portion of the JSON document need contain JSON-LD specific annotation.

```json
"@context" : {
	...
	"id" : "@id",
	"type": "@type",
	"value": "@value",
	...
}
```
This removes the need to add a statement for '@id', '@type', or '@value' completely:

* a context file can be completely externally defined by supplying the following at the beginning of the JSON document

```json
{
	"@context: "http://docs.oasis-open.org/cti/ns/stix/campaign-2.jsonld",
	...
	"id": "example:Indicator-33fe3b22-0201-47cf-85d0-97c02164528d",
	"type": "Campaign",
	"name": ...
}
```
This use of @context above references an external jsonld file that contains only a context and serves as a default vocabulary in much the same manner as we use XSDs in XML.  This would not only allow the ability define the vocabulary external to the JSON file, but would avoid people having to supply short IRIs (ala XML prefix) to key names.  It appears a similar approach could be possible by using the @vocab term as such:

```json
{
	“@context” : {
		“@vocab” : “https://docs.oasis-open.org/cti/ns/stix/stix-2.jsonld”,
		“databaseId” : null
	},
	“@id” : “”,
	“@type” : “Campaign”,
	“name” : “”
	...
}
```

What is nice about the @vocab approach is that you can specify in the context any terms that SHOULD NOT be expanded per the vocabulary by specifying the term with a value of null, as shown above with databaseId.

**Fact:** If a JSON doucment is sent to a JSON-LD consumer, a reference to the appropriate context can be specified in the HTTP header as a Link header relationshp using the value http://www.w3.org/ns/json-ld#context.  For example:

```html
Link: <http://json-ld.org/contexts/person.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="applicaton/ld+json"
```
**Fact:** If a JSON-LD document is sent to a JSON consumer the '@context' can be ignored and it would have no other impact if the context contained an alias for '@id', '@type', and '@value'.
 
That's it. 
 
There is no requirement for any JSON producer or consumer to do anything more than this. 
 
The only additional requirement for those wishing to leverage the semantic advantages of JSON-LD is availability of a [@context specification mapping structures and field names within the JSON content to specific elements of the RDF/OWL ontology](https://gist.github.com/sbarnum/e162ff4cf4e1c68f1854). 
 
In JSON-LD we have three main options for how to handle this @context: 

* it can be defined inline within the content (obviously high-impact on "pure" JSON users)

* it can be defined in a separate file and explicitly referenced with a single line at the beginning of a content file (very minimal impact on "pure" JSON users)
* it can be defined in a separate file and implicitly referenced and leveraged external to the content itself and based on the specification for a given version of STIX/CybOX (zero impact on "pure" JSON users). The examples below are leveraging this approach.
 
So, with a use of JSON-LD in this fashion the decision on whether to support only "pure" JSON or the richer semantic dimensions of JSON-LD is left solely to each producer and consumer. This means that the TC could still choose to specify an MTI serialization of JSON and not prevent those wishing to leverage higher-order semantic capabilities from doing so. The key is that the actual JSON Schema specified as the MTI should derive from JSON stylistic rules applied to a model-based field-traceable specification of the language.
 
# Examples
 
Below are three simple examples from our existing idioms of what this simple JSON-LD could look like. It should be noted that the current form of these examples is not intended to explicitly dictate JSON stylistic choices or modeling structures under discussion (ordering of fields, embedding vs referencing of fields, hierarchy vs flattening, graph vs tree, current structure for relationships, etc.). 

The examples are based on a mapping of STIX/XML 1.2 and Cybox/XML 2.1 to a set of ontologies that I developed for STIX, CybOX, and others that can be found at [https://github.com/daedafusion/cyber-ontology](https://github.com/daedafusion/cyber-ontology).  The examples were built by first loading the examples through a utility I wrote the converts the ingest STIX/XML into W3C-compliant N-Triples, which are then fed into [RDFLib configured with the rdflib_jsonld plugin](https://github.com/RDFLib/rdflib-jsonld) to do the conversion to JSON-LD.

**NOTE:**  I've not had the opportunity to play as much with the Framing Processor that can be used to control the format of the output.  This would allow us to more closely map the output of the JSON to that currently being specified by the STIX 2.0 Community draft specification.

 
A few quick notes on some of the stylistic and structural choices used in the examples:

* There is clear consensus support for breaking relationships out to separate top-level objects. The examples here illustrate one possible approach for how to represent them. This came for free from using RDF in the backplane. This is not intended to be a formal assertion that this should be the form used.
* Controlled vocabularies in the example leverage a very terse form proposed by Cory Casanave. 
* The examples leverage a unique form for representing CybOX patterns on object properties. The use of this form is not intended to assert that it should be the form used going forward.
* Many hierarchical structures have been flattened (using arrays, dot.notation, etc.) but it's recognized that there still remains opportunity for additional stylistic tweaking.
 
 
## Example #1: [IP Watchlist](https://github.com/STIXProject/schemas/blob/master/samples/STIX_IP_Watchlist.xml)
 
```json
{
"@id": "example:STIXPackage-33fe3b22-0201-47cf-85d0-97c02164528d",
"@type": "stix:Package",
"stix:indicators": [
{
"@id": "example:Indicator-33fe3b22-0201-47cf-85d0-97c02164528d",
"@type": "ind:Indicator",
"ind:indicatorType": "stixVocabs:IndicatorTypeVocab-1.1:IP_Watchlist",
"ind:observable": {"@id": "example:Observable-1c798262-a4cd-434d-a958-884d6980c459"},
"stixc:description": {"@value": "Sample IP Address Indicator for this watchlist. This contains one indicator with a set of three IP addresses in the watchlist."},
"stixc:timestamp": "2014-05-08T09:00:00Z"
}
],
"cybox:observables": {
"cybox:observable": [
{
"@id": "example:Observable-1c798262-a4cd-434d-a958-884d6980c459",
"@type": "cybox:Observable",
"cybox:observedEntity": {
"@id": "example:Object-1980ce43-8e03-490b-863a-ea404d12242e",
"@type": "addressObj:IPv4Address",
"addressObj:addressCategory": "ipv4-addr",
"addressObj:addressValue": {
"@type": "cyboxc:pattern",
"@value": "Equals( '10.0.0.0 OR 10.0.0.1 OR 10.0.0.2' )"
}
}
}
]
},
"stix:packageIntent": "stixVocabs:PackageIntentVocab-1.0:Indicators_-_Watchlist",
"stixc:timestamp": "2014-05-08T09:00:00Z",
"stixc:title": "Example watchlist that contains IP information."
}
```
 
 
[Gist for JSON-LD version of IP Watchlist example](https://gist.github.com/sbarnum/81054bc4ee3e118f00a3)
 
Gist for EclecticIQ JSON version of IP Watchlist example
 
[Gist for XML version of IP Watchlist example](https://gist.github.com/sbarnum/4eafbd96b42b45482236)
 
## Example #2: [Idiom : Incident Essentials - Who, What, When](http://stixproject.github.io/documentation/idioms/simple-incident/)
 
```json
{
"@id": "example:Package-ec96d2a6-5a95-48f2-93c0-b3b2198633ca",
"@type": "stix:Package",
"stix:incidents": [
{
"@id": "example:incident-8236b4a2-abe0-4b56-9347-288005c4bb92",
"@type": "inc:Incident",
"inc:impactEffects": "stixVocabs:IncidentEffectVocab-1.0:Financial_Loss",
"inc:reporter": {
"@id": "example:4cdb58ee3d18966146434adc2ce80d0972b2e991",
"@type": "stixc:InformationSource",
"cyboxc:time.producedDateTime": "2014-03-11T00:00:00",
"stixc:infoSource.description": {"@value": "The person who reported it"},
"stixc:infoSource.identity": {
"@id": "example:Identity-cd64aaa6-b1c0-4026-8ea1-14ff5a19e5fb",
"@type": "stixc:CIQIdentity",
"xnl:partyName": "Sample Investigations, LLC"
}
},
"inc:timeSequence.incidentDiscoveryDateTime": "2012-05-10T00:00:00",
"inc:timeSequence.incidentReportedDateTime": "2012-12-10T00:00:00",
"inc:timeSequence.initialCompromiseDateTime": "2012-01-30T00:00:00",
"inc:timeSequence.restorationAchievedDateTime": "2012-08-10T00:00:00",
"inc:victim": {
"@id": "example:Identity-dd8637b7-51b4-48f0-9e3c-a2b23b3a2dd7",
"@type": "stixc:CIQIdentity",
"xnl:partyName": "Cyber Tech Dynamics"
},
"stixc:confidence": {
"stixc:timestamp": "2014-11-18T23:40:08.061379Z",
"stixc:value": "stixVocabs:HighMediumLowVocab-1.0:High"
},
"stixc:description": {"@value": "Intrusion into enterprise network"},
"stixc:timestamp": "2014-11-18T23:40:08.061362Z",
"stixc:title": "Breach of Cyber Tech Dynamics"
}
]
}
```
 
 
[Gist for JSON-LD version of Idiom : Incident Essentials - Who, What, When](https://gist.github.com/sbarnum/5870c32f452959fc39df)
 
Gist for EclecticIQ JSON version of Idiom : Incident Essentials - Who, What, When
 
[Gist for XML version of Idiom : Incident Essentials - Who, What, When](https://gist.github.com/sbarnum/e3ea5e45fb8ca26c852f)
 
 
## Example #3: [Idiom : Malicious E-mail Indicator with Attachment](http://stixproject.github.io/documentation/idioms/malicious-email-attachment/)
 
```json
{
"@id": "example:Package-8b8ed1c1-f01d-4393-ac65-97017ed15876",
"@type": "stix:Package",
"stix:indicators": [
{
"@id": "example:indicator-8cf9236f-1b96-493d-98be-0c1c1e8b62d7",
"@type": "ind:Indicator",
"ind:indicatorType": "stixVocabs:IndicatorTypeVocab-1.1:Malicious_E-mail",
"ind:observable": {"@id": "example:Observable-437f0c20-ab26-4400-9f6a-fc395da3ddd9"},
"stixc:confidence": {
"stixc:timestamp": "2014-10-31T15:52:13.127950Z",
"stixc:value": "stixVocabs:HighMediumLowVocab-1.0:High"
},
"stixc:timestamp": "2014-10-31T15:52:13.127931Z",
"stixc:title": "Malicious E-mail"
}, {
"@id": "example:indicator-b06b0eb7-61dd-4338-a094-0290c380fbd8",
"@type": "ind:Indicator",
"ind:indicatorType": "stixVocabs:IndicatorTypeVocab-1.1:Malicious_E-mail",
"ind:observable": {"@id": "example:Observable-e9926796-6b52-463c-8be1-0ab66e9adb1c"},
"stixc:confidence": {
"stixc:timestamp": "2014-10-31T15:52:13.127225Z",
"stixc:value": "stixVocabs:HighMediumLowVocab-1.0:Low"
},
"stixc:timestamp": "2014-10-31T15:52:13.126999Z",
"stixc:title": "Malicious E-mail Subject Line"
}, {
"@id": "example:indicator-2e17f6fe-3a4d-438a-911a-e509ba1b9933",
"@type": "ind:Indicator",
"ind:indicatorType": "stixVocabs:IndicatorTypeVocab-1.1:Malicious_E-mail",
"ind:observable": {"@id": "example:Observable-9c9869a2-f822-4682-bda4-e89d31b18704"},
"stixc:confidence": {
"stixc:timestamp": "2014-10-31T15:52:13.127775Z",
"stixc:value": "stixVocabs:HighMediumLowVocab-1.0:Low"
},
"stixc:timestamp": "2014-10-31T15:52:13.127668Z",
"stixc:title": "Malicious E-mail Attachment"
}
],
"stix:TTPs": [
{
"@id": "example:ttp-d7b066aa-4091-4276-a142-29d5d81c3484",
"@type": "ttp:TTP",
"stixc:timestamp": "2014-10-31T15:52:13.126765Z",
"stixc:title": "Phishing"
}
],
"cybox:observables": {
"cybox:observable": [
{
"@id": "example:Observable-9c9869a2-f822-4682-bda4-e89d31b18704",
"@type": "cybox:Observable",
"cybox:observedEntity": {
"@id": "example:EmailMessage-9d56af8e-5588-4ed3-affd-bd769ddd7fe2",
"@type": "emailMsgObj:EmailMessage",
"emailMsgObj:attachments": [{"@id": "example:File-c182bcb6-8023-44a8-b340-157295abc8a6"}]
}
}, {
"@id": "example:Observable-437f0c20-ab26-4400-9f6a-fc395da3ddd9",
"@type": "cybox:Observable",
"cybox:observedEntity": {
"@id": "example:EmailMessage-0dc3478e-153a-412f-8718-7e9ee65b8084",
"@type": "emailMsgObj:EmailMessage",
"emailMsgObj:attachments": [{"@id": "example:File-c182bcb6-8023-44a8-b340-157295abc8a6"}],
"emailMsgObj:header.subject": {
"@type": "cyboxc:pattern",
"@value": "StartsWith( '[IMPORTANT] Please Review Before' )"
}
}
}, {
"@id": "example:Observable-e9926796-6b52-463c-8be1-0ab66e9adb1c",
"@type": "cybox:Observable",
"cybox:observedEntity": {
"@id": "example:EmailMessage-38afa5c9-ef26-4948-928b-0230521c67b7",
"@type": "emailMsgObj:EmailMessage",
"emailMsgObj:header.subject": {
"@type": "cyboxc:pattern",
"@value": "StartsWith( '[IMPORTANT] Please Review Before' )"
}
}
}
],
"cybox:ObjectPool": [
{
"@id": "example:File-c182bcb6-8023-44a8-b340-157295abc8a6",
"@type": "fileObj:File",
"fileObj:fileExtension": {
"@type": "cyboxc:pattern",
"@value": "Equals( 'doc.exe' )"
},
"fileObj:fileName": {
"@type": "cyboxc:pattern",
"@value": "StartsWith( 'Final Report' )"
}
}
]
},
"stix:relationships": [
{
"@id": "example:Relationship-91a98baca7528928aca2bd3c3afb89c5c063580b",
"@type": "ind:indicatedTTP",
"stixc:to": "example:ttp-d7b066aa-4091-4276-a142-29d5d81c3484",
"stixc:from": "example:indicator-8cf9236f-1b96-493d-98be-0c1c1e8b62d7"
}, {
"@id": "example:Relationship-fd03727b704b20f0f32f59a2a024daf39a56b1d9",
"@type": "ind:indicatedTTP",
"stixc:to": "example:ttp-d7b066aa-4091-4276-a142-29d5d81c3484",
"stixc:from": "example:indicator-b06b0eb7-61dd-4338-a094-0290c380fbd8"
}, {
"@id": "example:Relationship-a7142d45bc1d641ffb231abae34a7d3245573a16",
"@type": "ind:indicatedTTP",
"stixc:to": "example:ttp-d7b066aa-4091-4276-a142-29d5d81c3484",
"stixc:from": "example:indicator-2e17f6fe-3a4d-438a-911a-e509ba1b9933"
}, {
"@id": "example:Relationship-1b2e978554d80a2295747becfb4b67ffe3df0d27",
"@type": "cybox:relatedObjects",
"cyboxc:objectRelationshipType": "cyboxVocabs:ObjectRelationshipVocab-1.1:Contains",
"stixc:to": "example:File-c182bcb6-8023-44a8-b340-157295abc8a6",
"stixc:from": "example:EmailMessage-9d56af8e-5588-4ed3-affd-bd769ddd7fe2"
}
]
}
```
 
 
[Gist for JSON-LD version of Idiom : Malicious E-mail Indicator with Attachment](https://gist.github.com/sbarnum/b97a089e9b8fd67866a0)
 
[Gist for EclecticIQ JSON version of Idiom : Malicious E-mail Indicator with Attachment](https://gist.github.com/sbarnum/1fba5b26b6dd3dd28563)
 
[Gist for XML version of Idiom : Malicious E-mail Indicator with Attachment](https://gist.github.com/sbarnum/efeafc831f335e00ce62)
 
# Summary
 
This approach of leveraging JSON-LD (and appropriate stylistic serializations) as a bridge between explicit semantic model-based specifications for STIX/CybOX and support for very simple "pure" JSON serializations for those who desire them offers unique potential in enabling a substantial and diverse community (both within and intersecting with the CTI TC) of players without disenfranchising anyone or imposing inappropriate adoption overhead.
