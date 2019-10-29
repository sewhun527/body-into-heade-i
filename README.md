<?xml version="1.0" encoding="UTF-8"?>

<mule xmlns:dw="http://www.mulesoft.org/schema/mule/ee/dw" xmlns:metadata="http://www.mulesoft.org/schema/mule/metadata" xmlns:http="http://www.mulesoft.org/schema/mule/http" xmlns:mulexml="http://www.mulesoft.org/schema/mule/xml" xmlns:tracking="http://www.mulesoft.org/schema/mule/ee/tracking" xmlns="http://www.mulesoft.org/schema/mule/core" xmlns:doc="http://www.mulesoft.org/schema/mule/documentation"
	xmlns:spring="http://www.springframework.org/schema/beans" 
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-current.xsd
http://www.mulesoft.org/schema/mule/core http://www.mulesoft.org/schema/mule/core/current/mule.xsd
http://www.mulesoft.org/schema/mule/http http://www.mulesoft.org/schema/mule/http/current/mule-http.xsd
http://www.mulesoft.org/schema/mule/ee/dw http://www.mulesoft.org/schema/mule/ee/dw/current/dw.xsd
http://www.mulesoft.org/schema/mule/xml http://www.mulesoft.org/schema/mule/xml/current/mule-xml.xsd
http://www.mulesoft.org/schema/mule/ee/tracking http://www.mulesoft.org/schema/mule/ee/tracking/current/mule-tracking-ee.xsd">
    <http:listener-config name="HTTP_Listener_Configuration" host="0.0.0.0" port="8081" doc:name="HTTP Listener Configuration"/>
    <flow name="set-headersFlow">
        <http:listener config-ref="HTTP_Listener_Configuration" path="/test" doc:name="HTTP"/>
        <mulexml:dom-to-xml-transformer doc:name="DOM to XML"/>
        <logger message="#[payload]" level="INFO" doc:name="Logger"/>
        <choice doc:name="Choice">
            <when expression="#[message.inboundProperties.XBOAUserid !=null ]">
                <logger message="#[payload]" level="INFO" doc:name="existing flow continoues"/>
                <flow-ref name="existing-flow" doc:name="Call existing flow continoues"/>
            </when>
            <otherwise>
                <dw:transform-message doc:name="Transform Message" metadata:id="435a4ba9-e085-4bc0-89ea-685ade9628bf">
                    <dw:input-payload mimeType="application/xml"/>
                    <dw:set-payload><![CDATA[%dw 1.0
%output application/java
%namespace ns0 http://schemas.xmlsoap.org/soap/envelope/
%namespace ns1 http://soap.sforce.com/2005/09/outbound
%namespace ns2 urn:sobject.enterprise.soap.sforce.com
---
{
	ns0#Envelope: {
		ns0#Body: {
			ns1#notifications: {
				ns1#Notification: {
					ns1#sObject: {
						ns2#Id: payload.ns0#Envelope.ns0#Body.ns1#notifications.ns1#Notification.ns1#sObject.ns2#Id,
						ns2#XBOAPassword: payload.ns0#Envelope.ns0#Body.ns1#notifications.ns1#Notification.ns1#sObject.ns2#XBOAPassword,
						ns2#XBOAUserid: payload.ns0#Envelope.ns0#Body.ns1#notifications.ns1#Notification.ns1#sObject.ns2#XBOAUserid
					}
				}
			}
		}
	}
}]]></dw:set-payload>
                    <dw:set-property propertyName="XBOAPassword"><![CDATA[%dw 1.0
%output application/java
---
{
	XBOPASS: payload.Envelope.Body.notifications.Notification.sObject.XBOAPassword
	
}]]></dw:set-property>
                    <dw:set-property propertyName="XBOAUserId"><![CDATA[%dw 1.0
%output application/java
---
payload.Envelope.Body.notifications.Notification.sObject.XBOAUserid]]></dw:set-property>
                </dw:transform-message>
                <logger level="INFO" doc:name="Logger"/>
                <message-properties-transformer doc:name="Message Properties">
                    <add-message-property key="xboapassw" value="#[message.outboundProperties.XBOAUserId]"/>
                </message-properties-transformer>
                <flow-ref name="existing-flow" doc:name="Call existing flow continoues"/>
            </otherwise>
        </choice>
    </flow>
    <flow name="existing-flow">
        <logger level="INFO" doc:name="Logger"/>
    </flow>
</mule>
