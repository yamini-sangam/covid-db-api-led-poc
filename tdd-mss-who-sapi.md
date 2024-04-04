# <p align="center">Raml Code</p>
# mss-who-sapi 
- ## examples
  - ### 400.json
    <details>
    <summary></summary>

    ```json
    {
       "code":400,
       "message":"Bad request",
       "description":"required key caseType not found",
       "dateTime":"2021-05-29T05:58:02Z",
       "transactionId":"44b32520-61ee-47b4-907d-fa15869f3c4d"
    }

  - ### 500.json
    <details>
    <summary></summary>

    ```json
    {
       "code":500,
       "message":"Internal server error",
       "description":"The server encountered an unexpected condition which prevented it from fulfilling the request",
       "dateTime":"2021-05-29T05:58:02Z",
       "transactionId":"44b32520-61ee-47b4-907d-fa15869f3c4d"
    }

  - ### 503.json
    <details>
    <summary></summary>
  
    ```json
    {
       "code":503,
       "message":"Service unavailable",
       "description":"The (upstream) service is temporarily not available",
       "dateTime":"2021-05-31T06:18:02Z",
       "transactionId":"48n32920-69ne-47b4-907d-fa15869f3c4d"
    }

  - ### report-covid-case-request.json
    <details>
    <summary></summary>
  
    ```json
    {
       "caseType":"positive",
       "firstName":"John",
       "lastName":"Nixon",
       "phone":"541-754-3010",
       "email":"john@gmail.com",
       "dateOfBirth":"1989-04-26",
       "country":"USA"
    }

  - ### report-covid-case-response.json
    <details>
    <summary></summary>
  
    ```json
    {
       "code": 200,
       "message": "Success",
       "description": "Case reported successfully",
       "dateTime": "2021-07-29T05:58:02Z",
       "transactionId": "44b32520-61ee-47b4-907d-fa15869f3c4d"
    }

- ## who-sapi-types
  - ### who-sapi-types.raml
    <details>
    <summary></summary>
    
    ```raml
    #%RAML 1.0 Library
    usage: Types for who-sapi
    types:
      report-covid-case-request-type:
        type: object
        description: This type used to register and update covid case
        properties:
          caseType: 
            type: string
            required: true
            example: "positive"
          firstName: 
            type: string
            required: true
            example: "John"
          lastName: 
            type: string
            required: true
            example: "Nixon"
          phone: 
            type: string
            required: true
          email: 
            type: string
            required: false
            example: "john@gmail.com"
          dateOfBirth:
            type: date-only
            required: true
            example: "1989-04-26"
          country: 
            type: string
            required: true
            example: "USA"
      report-covid-case-response-type:
        description: This type used to respond back the success status
        type: object
        properties:
          code:
            type: integer
            description: status code
            required: true 
          message:
            description: A human readable message that describes the status
            type: string
            required: true
          description:
            description: A human readable, comprehensive explanation of the status
            type: string
            required: false
          dateTime:
            description: The date-time stamp of the response
            type: datetime
            required: true
          transactionId:
            description: Internal identifier of the transaction
            type: string
            required: true

- ## who-sapi.raml
  <details>
  <summary></summary>

  ```raml
  #%RAML 1.0
  title: who-sapi
  version: v1
  baseUri: http://{environment}/covid/{version}/
  baseUriParameters:
    environment:
      description: DEV, TEST, UAT, PROD
      enum: ["uho-dev-who-sapi.us-e2.cloudhub.io","uho-test-who-sapi.us-e2.cloudhub.io", "uho-uat-who-sapi.us-e2.cloudhub.io", "uho-prod-who-sapi.us-e2.cloudhub.io"]
  traits:
    client-id-header: !include exchange_modules/d896f4a8-62ed-4bab-9db4-5db7ac45ebd4/client-id-required/1.0.0/client-id-required.raml
    transaction-header: !include exchange_modules/d896f4a8-62ed-4bab-9db4-5db7ac45ebd4/transaction-header/1.0.0/transaction-header.raml
    correlation-id-header: !include exchange_modules/d896f4a8-62ed-4bab-9db4-5db7ac45ebd4/correlation-id-header/1.0.0/correlation-id-header.raml
  uses:
    resource-types: exchange_modules/d896f4a8-62ed-4bab-9db4-5db7ac45ebd4/resource-types/1.0.0/resource-types.raml
    common-data-types: exchange_modules/d896f4a8-62ed-4bab-9db4-5db7ac45ebd4/common-data-types/1.0.0/common-data-types.raml
    who-sapi-types: who-sapi-types/who-sapi-types.raml
  /case:
    post:
      description: To register covid case
      is: [client-id-header,correlation-id-header,transaction-header]
      body:
        application/json:
          type: who-sapi-types.report-covid-case-request-type
          example: !include examples/report-covid-case-request.json
      responses:
        200:
          body:
            application/json:
              type: who-sapi-types.report-covid-case-response-type
              example: !include examples/report-covid-case-response.json
        400:
          body:
            application/json:
              type: common-data-types.errorType
              example: !include examples/400.json
        503:
          body:
            application/json:
              type: common-data-types.errorType
              example: !include examples/503.json
        500:
          body:
            application/json:
              type: common-data-types.errorType
              example: !include examples/500.json
  /health-check:
    get:
      type:
        resource-types.all: 
          getResponseType: string


# <p align="center">Implementation of mss-who-sapi</p>  
# mss-who-sapi
- ## src\main
  - ### mule
    - #### implementations
      - ##### health-check.xml
        <details>
        <summary></summary>
        
        ```xml
        <mule xmlns="http://www.mulesoft.org/schema/mule/core" xmlns:doc="http://www.mulesoft.org/schema/mule/documentation" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.mulesoft.org/schema/mule/core http://www.mulesoft.org/schema/mule/core/current/mule.xsd">
            <flow name="get:\health-check:who-sapi-config">
                <logger level="INFO" message="get:\health-check:who-sapi-config"/>
            </flow>
        </mule>

      - ##### report-case-to-who.xml
        <details>
        <summary></summary>
        
        ```xml
        <mule xmlns:jms="http://www.mulesoft.org/schema/mule/jms" xmlns:ee="http://www.mulesoft.org/schema/mule/ee/core" xmlns="http://www.mulesoft.org/schema/mule/core" xmlns:doc="http://www.mulesoft.org/schema/mule/documentation" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.mulesoft.org/schema/mule/core http://www.mulesoft.org/schema/mule/core/current/mule.xsd http://www.mulesoft.org/schema/mule/ee/core http://www.mulesoft.org/schema/mule/ee/core/current/mule-ee.xsd http://www.mulesoft.org/schema/mule/jms http://www.mulesoft.org/schema/mule/jms/current/mule-jms.xsd">
            <flow name="post:\case:application\json:who-sapi-config">
                <set-variable value="#[attributes.headers.'x-correlation-id' default ""]" doc:name="Set correlationId" doc:id="0933d8e4-faf1-4209-aa91-95fd092d4f27" variableName="correlationId"/>
                <logger level="INFO" doc:name="Start Log" doc:id="75638d13-7720-4098-9a89-ef44ec362f23" message="transactionID: #[vars.transactionId]], correlationID: #[vars.correlationID], message: &quot;Started report covid case to who flow&quot;, payload: #[payload]"/>
                <ee:transform doc:name="Prepare WHO Case Payload" doc:id="3ff8b02e-9db2-4697-8acf-ff477f00c12d">
                    <ee:message>
                        <ee:set-payload>
                            <![CDATA[ %dw 2.0 output application/xml --- { COVID_CASE: { FIRST_NAME: payload.firstName, LAST_NAME: payload.lastName, PHONE: payload.phone, EMAIL: payload.email, DATE_OF_BIRTH: payload.dateOfBirth, COUNTRY: payload.country } } ]]>
                        </ee:set-payload>
                    </ee:message>
                    <ee:variables>
                        <ee:set-variable variableName="dynamicProperties">
                            <![CDATA[ %dw 2.0 output application/java --- { CASE_TYPE: payload.caseType } ]]>
                        </ee:set-variable>
                    </ee:variables>
                </ee:transform>
                <!--  		<flow-ref doc:name="publish-covid-case-to-who" doc:id="69da617a-9262-4d14-ae22-1b3734d219fc" name="publish-covid-case-to-who" />
                  -->
                <ee:transform xmlns:ee="http://www.mulesoft.org/schema/mule/ee/core" xsi:schemaLocation="http://www.mulesoft.org/schema/mule/ee/core http://www.mulesoft.org/schema/mule/ee/core/current/mule-ee.xsd" doc:name="Set Response">
                    <ee:message>
                        <ee:set-payload>
                            <![CDATA[ %dw 2.0 output application/json --- { code: 200, message: &quot;Success&quot;, description: &quot;Case reported successfully&quot;, dateTime: now() as String { format: &quot;yyyy-MM-dd'T'HH:mm:ss'Z'&quot; }, transactionId: vars.transactionId } ]]>
                        </ee:set-payload>
                    </ee:message>
                </ee:transform>
                <logger level="INFO" doc:name="End Log" doc:id="efa235bc-a40c-4854-8a50-d68903f85a34" message="transactionID: #[vars.transactionId]], correlationID: #[vars.correlationID], message: &quot;Started report covid case to who flow&quot;, payload: #[payload]"/>
            </flow>
        </mule>

    - #### subflows
      - ##### who-jms.xml
        <details>
        <summary></summary>
          
        ```xml
        <mule xmlns:jms="http://www.mulesoft.org/schema/mule/jms" xmlns="http://www.mulesoft.org/schema/mule/core" xmlns:doc="http://www.mulesoft.org/schema/mule/documentation" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.mulesoft.org/schema/mule/core http://www.mulesoft.org/schema/mule/core/current/mule.xsd http://www.mulesoft.org/schema/mule/jms http://www.mulesoft.org/schema/mule/jms/current/mule-jms.xsd">
            <sub-flow name="publish-covid-case-to-who" doc:id="09541aa1-9856-4338-aa92-4e071d92e3e5">
                <logger level="DEBUG" doc:name="Before Backend Call" doc:id="033ac45d-ca9f-428a-95c7-014d39cbb139" message="transactionID: #[vars.transactionId]], correlationID: #[vars.correlationID], message: &quot;Before publish case to who JMS&quot;, payload: #[payload], attributes: #[vars.dynamicProperties]"/>
                <jms:publish doc:name="Publish Case To WHO" doc:id="0bb6b7f0-8467-47ca-9660-8eed0a856a07" destination="${secure::jms.who.covid.case.queue}" config-ref="JMS_UHO_Config">
                    <jms:message>
                        <jms:properties>
                            <![CDATA[ #[vars.dynamicProperties] ]]>
                        </jms:properties>
                    </jms:message>
                </jms:publish>
                <logger level="DEBUG" doc:name="After Backend Call" doc:id="f73a05aa-ec74-4ca3-8b43-cd9146e2e83a" message="transactionID: #[vars.transactionId]], correlationID: #[vars.correlationID], message: &quot;After publish case to who JMS&quot;, payload: #[payload]"/>
            </sub-flow>
        </mule>

    - #### global.xml
      <details>
      <summary></summary>
      
      ```xml
      <mule xmlns:api-gateway="http://www.mulesoft.org/schema/mule/api-gateway" xmlns:jms="http://www.mulesoft.org/schema/mule/jms" xmlns:apikit="http://www.mulesoft.org/schema/mule/mule-apikit" xmlns:http="http://www.mulesoft.org/schema/mule/http" xmlns:secure-properties="http://www.mulesoft.org/schema/mule/secure-properties" xmlns="http://www.mulesoft.org/schema/mule/core" xmlns:doc="http://www.mulesoft.org/schema/mule/documentation" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation=" http://www.mulesoft.org/schema/mule/api-gateway http://www.mulesoft.org/schema/mule/api-gateway/current/mule-api-gateway.xsd http://www.mulesoft.org/schema/mule/jms http://www.mulesoft.org/schema/mule/jms/current/mule-jms.xsd http://www.mulesoft.org/schema/mule/mule-apikit http://www.mulesoft.org/schema/mule/mule-apikit/current/mule-apikit.xsd http://www.mulesoft.org/schema/mule/http http://www.mulesoft.org/schema/mule/http/current/mule-http.xsd http://www.mulesoft.org/schema/mule/core http://www.mulesoft.org/schema/mule/core/current/mule.xsd http://www.mulesoft.org/schema/mule/secure-properties http://www.mulesoft.org/schema/mule/secure-properties/current/mule-secure-properties.xsd">
          <secure-properties:config name="Secure_Properties_Config" doc:name="Secure Properties Config" doc:id="9eaa6648-18f6-442b-a849-80a7045cc186" file="${env}.yaml" key="${enc.key}"/>
          <http:listener-config name="who-sapi-httpListenerConfig" basePath="covid">
              <http:listener-connection host="0.0.0.0" port="${http.port}"/>
          </http:listener-config>
          <apikit:config name="who-sapi-config" api="who-sapi.raml" outboundHeadersMapName="outboundHeaders" httpStatusVarName="httpStatus"/>
          <jms:config name="JMS_UHO_Config" doc:name="JMS Config" doc:id="53bbf26c-bb8a-4559-823e-a164020e418f">
              <jms:active-mq-connection username="${secure::jms.who.username}" password="${secure::jms.who.password}">
                  <jms:caching-strategy>
                      <jms:no-caching/>
                  </jms:caching-strategy>
                  <jms:factory-configuration brokerUrl="${secure::jms.who.broker.url}"/>
              </jms:active-mq-connection>
          </jms:config>
          <!--  	<api-gateway:autodiscovery apiId="${instance.id}" ignoreBasePath="true" doc:name="API Autodiscovery" doc:id="56252fd6-2124-4103-a5db-a2c888963480" flowRef="who-sapi-main" />
            -->
      </mule>

    - #### global-error-handler.xml
        <details>
        <summary></summary>
          
        ```xml
      <mule xmlns:ee="http://www.mulesoft.org/schema/mule/ee/core"
          xmlns="http://www.mulesoft.org/schema/mule/core"
          xmlns:doc="http://www.mulesoft.org/schema/mule/documentation"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://www.mulesoft.org/schema/mule/ee/core http://www.mulesoft.org/schema/mule/ee/core/current/mule-ee.xsd
          http://www.mulesoft.org/schema/mule/core http://www.mulesoft.org/schema/mule/core/current/mule.xsd">
      
          <!-- Global Error Handler -->
          <error-handler name="global-error-handler" doc:id="77f85497-bc5a-4406-9826-4708e19ce808">
      
              <!-- Handle APIKIT:BAD_REQUEST error -->
              <on-error-propagate type="APIKIT:BAD_REQUEST" enableNotifications="true" logException="true" doc:name="On Error Propagate" doc:id="ef3cf218-baa9-4253-9657-bc8168fb8815">
                  <set-variable value="#[400]" doc:name="Set HTTP Status - 400" doc:id="5e7ccc4e-d9ea-4e3d-8442-c14ecb513309" variableName="httpStatus"/>
                  <set-variable value="Bad request" doc:name="set Error Message" doc:id="3cdb7dd5-31bf-4c24-b910-101a9242ea20" variableName="errorMessage"/>
                  <set-variable value="#[(((error.description default "" replace "[" with "") replace "]" with "") splitBy "\n")]" doc:name="Set Error Description" doc:id="e094d07d-c5fb-4083-8f00-6b1a40369929" variableName="errorDescription"/>
                  <flow-ref doc:name="global-prepare-error-response-sub-flow" doc:id="eb11b457-b296-4d7e-9a27-6d2bdaca019a" name="global-prepare-error-response-sub-flow"/>
              </on-error-propagate>
      
              <!-- Handle APIKIT:METHOD_NOT_ALLOWED error -->
              <on-error-propagate type="APIKIT:METHOD_NOT_ALLOWED" enableNotifications="true" logException="true" doc:name="On Error Propagate" doc:id="2dc25069-521b-4119-b9d9-8a2811417ff1">
                  <set-variable value="#[405]" doc:name="Set HTTP Status - 405" doc:id="3a96b68f-77cf-4b1e-a185-d4384e1bf5b8" variableName="httpStatus"/>
                  <set-variable value="Method Not Allowed" doc:name="Set Error Message" doc:id="b5b32a4e-135b-42b7-85fd-ef756edd3471" variableName="errorMessage"/>
                  <set-variable value="The method specified in the request is not allowed for this resource" doc:name="Set Error Description" doc:id="294531c7-7f83-4fef-a1ae-520bd7f2b25b" variableName="errorDescription"/>
                  <flow-ref doc:name="global-prepare-error-response-sub-flow" doc:id="7521a9ad-5006-43b3-acae-46284c39aac7" name="global-prepare-error-response-sub-flow"/>
              </on-error-propagate>
      
              <!-- Handle APIKIT:NOT_ACCEPTABLE error -->
              <on-error-propagate type="APIKIT:NOT_ACCEPTABLE" enableNotifications="true" logException="true" doc:name="On Error Propagate" doc:id="df815d08-9957-4e68-b3ca-f864a5181115">
                  <set-variable value="#[406]" doc:name="Set HTTP Status - 406" doc:id="08259cfe-fbf3-441e-951f-ab431c5f57b0" variableName="httpStatus"/>
                  <set-variable value="Not Acceptable" doc:name="Set Error Message" doc:id="ca6b5800-1bfb-43d9-9ada-7d372d4aa0e7" variableName="errorMessage"/>
                  <set-variable value="The resource identified by the request is not capable of generating response entities according to the request accept headers" doc:name="Set Error Description" doc:id="4e301aef-84af-4c43-923d-abc306306058" variableName="errorDescription"/>
                  <flow-ref doc:name="global-prepare-error-response-sub-flow" doc:id="0a0586a1-749f-4b09-8ff1-ed299bbaeace" name="global-prepare-error-response-sub-flow"/>
              </on-error-propagate>
      
              <!-- Handle APIKIT:NOT_FOUND error -->
              <on-error-propagate type="APIKIT:NOT_FOUND" enableNotifications="true" logException="true" doc:name="On Error Propagate" doc:id="68d40e8d-5285-41e2-b299-01d614c1b656">
                  <set-variable value="#[404]" doc:name="Set HTTP Status - 404" doc:id="de8bb9db-b4f8-4cd3-9f0a-348869331950" variableName="httpStatus"/>
                  <set-variable value="Not found" doc:name="Set Error Message" doc:id="23037b6a-29f1-41d7-867f-58af686326e4" variableName="errorMessage"/>
                  <set-variable value="The server has not found anything matching the Request-URI" doc:name="Set Error Description" doc:id="e6d889a0-2b53-455d-8d05-150e5d3bd91e" variableName="errorDescription"/>
                  <flow-ref doc:name="global-prepare-error-response-sub-flow" doc:id="42d77664-1fb8-4900-b13d-62d7fc8d11c3" name="global-prepare-error-response-sub-flow"/>
              </on-error-propagate>
      
              <!-- Default error handler -->
              <on-error-propagate enableNotifications="true" logException="true" doc:name="On Error Propagate" doc:id="4b526b96-51b5-4f28-9db7-3347f666b930">
                  <set-variable value="#[500]" doc:name="Set HTTP Status - 500" doc:id="11abeb41-b20d-4dc1-af49-b1a9fb36686a" variableName="httpStatus"/>
                  <set-variable value="Internal Server Error" doc:name="Set Error Message" doc:id="e44bbfb1-e303-477b-a28d-6050bca1a513" variableName="errorMessage"/>
                  <set-variable value="#[error.description ?: error.cause.errorType.errorMessage default 'An error occurred']" doc:name="Set Error Description" doc:id="bfc1d460-e83e-4d9e-a7f9-10817111836d" variableName="errorDescription"/>
                  <flow-ref doc:name="global-prepare-error-response-sub-flow" doc:id="fe45e5cf-d154-45ec-a81f-39418081e805" name="global-prepare-error-response-sub-flow"/>
              </on-error-propagate>
      
          </error-handler>
      
          <!-- Sub-flow to prepare error response -->
          <sub-flow name="global-prepare-error-response-sub-flow" doc:id="ccda8235-9c1e-4640-b725-eb71331df2ff">
              <set-payload value="#[{'code':vars.httpStatus, 'message':vars.errorMessage, 'description':vars.errorDescription}]" doc:name="Set Error Response Payload" doc:id="c66668bb-170b-43b2-8134-9dfc081ac2a3"/>
              <ee:transform doc:name="Transform Payload to JSON" doc:id="52a06212-b8f0-4031-8080-1d2732c9b26f">
                  <ee:message>
                      <ee:set-payload><![CDATA[%dw 2.0
                  output application/json
                  ---
                  payload]]></ee:set-payload>
                  </ee:message>
              </ee:transform>
          </sub-flow>
      
      </mule>

    - #### who-sapi.xml
      <details>
      <summary></summary>
      
      ```xml
      <mule xmlns:doc="http://www.mulesoft.org/schema/mule/documentation" xmlns="http://www.mulesoft.org/schema/mule/core" xmlns:apikit="http://www.mulesoft.org/schema/mule/mule-apikit" xmlns:http="http://www.mulesoft.org/schema/mule/http" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.mulesoft.org/schema/mule/core http://www.mulesoft.org/schema/mule/core/current/mule.xsd http://www.mulesoft.org/schema/mule/http http://www.mulesoft.org/schema/mule/http/current/mule-http.xsd http://www.mulesoft.org/schema/mule/mule-apikit http://www.mulesoft.org/schema/mule/mule-apikit/current/mule-apikit.xsd ">
          <flow name="who-sapi-main">
              <http:listener config-ref="who-sapi-httpListenerConfig" path="/v1/*">
                  <http:response statusCode="#[vars.httpStatus default 200]">
                      <http:headers>
                          <![CDATA[ #[vars.outboundHeaders default {}] ]]>
                      </http:headers>
                  </http:response>
                  <http:error-response statusCode="#[vars.httpStatus default 500]">
                      <http:body>
                          <![CDATA[ #[payload] ]]>
                      </http:body>
                      <http:headers>
                          <![CDATA[ #[vars.outboundHeaders default {}] ]]>
                      </http:headers>
                  </http:error-response>
              </http:listener>
              <set-variable value="#[attributes.headers.'x-transaction-id' default uuid()]" doc:name="Set transactionId" doc:id="b5700bd3-acd1-495e-b329-fe69a3272184" variableName="transactionId"/>
              <apikit:router config-ref="who-sapi-config"/>
              <error-handler ref="global-error-handler"/>
          </flow>
          <flow name="who-sapi-console">
              <http:listener config-ref="who-sapi-httpListenerConfig" path="/console/*">
                  <http:response statusCode="#[vars.httpStatus default 200]">
                      <http:headers>#[vars.outboundHeaders default {}]</http:headers>
                  </http:response>
                  <http:error-response statusCode="#[vars.httpStatus default 500]">
                      <http:body>#[payload]</http:body>
                      <http:headers>#[vars.outboundHeaders default {}]</http:headers>
                  </http:error-response>
              </http:listener>
              <apikit:console config-ref="who-sapi-config"/>
              <error-handler>
                  <on-error-propagate type="APIKIT:NOT_FOUND">
                      <ee:transform xmlns:ee="http://www.mulesoft.org/schema/mule/ee/core" xsi:schemaLocation="http://www.mulesoft.org/schema/mule/ee/core http://www.mulesoft.org/schema/mule/ee/core/current/mule-ee.xsd">
                          <ee:message>
                              <ee:set-payload>
                                  <![CDATA[ %dw 2.0 output application/json --- {message: "Resource not found"} ]]>
                              </ee:set-payload>
                          </ee:message>
                          <ee:variables>
                              <ee:set-variable variableName="httpStatus">404</ee:set-variable>
                          </ee:variables>
                      </ee:transform>
                  </on-error-propagate>
              </error-handler>
          </flow>
      </mule>

  - ### resources
    - #### dev.yaml
      <details>
      <summary></summary>
      
      ```yaml
      http:
        port: "8081"
      
      jms:
        who.broker.url: "tcp://localhost:61616"
        who.username: "admin"
        who.password: "![qbNVHoyilMguV3wOS45CFQ==]"
        who.covid.case.queue: "Q.WHO.COVID.CASE.REPORT"

    - #### prod.yaml
      <details>
      <summary></summary>
      
      ```yaml
      http:
        port: "8091"

    - #### test.yaml
      <details>
      <summary></summary>
      
      ```yaml
      http:
        port: "8091"

    - #### uat.yaml
      <details>
      <summary></summary>
      
      ```yaml
      http:
        port: "8091"
- ## mule-artifact.json
  <details>
  <summary></summary>

  ```json
  {
    "minMuleVersion": "4.3.0",
    "secureProperties": ["enc.key", "anypoint.platform.client_id", "anypoint.platform.client_secret"]
  }

- ## pom.xml
  <details>
  <summary></summary>

  ```xml
  <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.mycompany</groupId>
    <artifactId>who-sapi</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>mule-application</packaging>
    <name>who-sapi</name>
    <properties>
      <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
      <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
      <app.runtime>4.3.0-20210322</app.runtime>
      <mule.maven.plugin.version>3.5.1</mule.maven.plugin.version>
    </properties>
    <build>
      <plugins>
        <plugin>
          <groupId>org.apache.maven.plugins</groupId>
          <artifactId>maven-clean-plugin</artifactId>
          <version>3.0.0</version>
        </plugin>
        <plugin>
          <groupId>org.mule.tools.maven</groupId>
          <artifactId>mule-maven-plugin</artifactId>
          <version>${mule.maven.plugin.version}</version>
          <extensions>true</extensions>
          <configuration>
            <sharedLibraries>
              <sharedLibrary>
                <groupId>org.apache.activemq</groupId>
                <artifactId>activemq-all</artifactId>
              </sharedLibrary>
            </sharedLibraries>
          </configuration>
        </plugin>
      </plugins>
    </build>
    <dependencies>
      <dependency>
        <groupId>org.mule.connectors</groupId>
        <artifactId>mule-http-connector</artifactId>
        <version>1.5.24</version>
        <classifier>mule-plugin</classifier>
      </dependency>
      <dependency>
        <groupId>org.mule.connectors</groupId>
        <artifactId>mule-sockets-connector</artifactId>
        <version>1.2.1</version>
        <classifier>mule-plugin</classifier>
      </dependency>
      <dependency>
        <groupId>org.mule.modules</groupId>
        <artifactId>mule-apikit-module</artifactId>
        <version>1.5.1</version>
        <classifier>mule-plugin</classifier>
      </dependency>
      <dependency>
        <groupId>com.mulesoft.modules</groupId>
        <artifactId>mule-secure-configuration-property-module</artifactId>
        <version>1.2.3</version>
        <classifier>mule-plugin</classifier>
      </dependency>
      <dependency>
        <groupId>org.mule.connectors</groupId>
        <artifactId>mule-jms-connector</artifactId>
        <version>1.7.3</version>
        <classifier>mule-plugin</classifier>
      </dependency>
      <dependency>
        <groupId>org.apache.activemq</groupId>
        <artifactId>activemq-all</artifactId>
        <version>5.15.15</version>
      </dependency>
    </dependencies>
    <repositories>
      <repository>
        <id>anypoint-exchange-v2</id>
        <name>Anypoint Exchange</name>
        <url>https://maven.anypoint.mulesoft.com/api/v2/maven</url>
        <layout>default</layout>
      </repository>
      <repository>
        <id>mulesoft-releases</id>
        <name>MuleSoft Releases Repository</name>
        <url>https://repository.mulesoft.org/releases/</url>
        <layout>default</layout>
      </repository>
    </repositories>
    <pluginRepositories>
      <pluginRepository>
        <id>mulesoft-releases</id>
        <name>MuleSoft Releases Repository</name>
        <layout>default</layout>
        <url>https://repository.mulesoft.org/releases/</url>
        <snapshots>
          <enabled>false</enabled>
        </snapshots>
      </pluginRepository>
    </pluginRepositories>
  </project>
  
