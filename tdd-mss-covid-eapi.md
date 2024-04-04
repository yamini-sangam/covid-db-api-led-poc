# <p align="center">Raml Code</p>
# mss-who-sapi 
- ## examples
  - ### 400.json
     <details>
      <summary></summary>
      
      ```json
      {
        "code": 400,
        "message": "Bad request",
        "description": "required key caseType not found",
        "dateTime": "2021-05-29T05:58:02Z",
        "transactionId": "44b32520-61ee-47b4-907d-fa15869f3c4d"
      }
  - ### 404.json
     <details>
      <summary></summary>
      
      ```json
      {
        "code": 404,
        "message": "No cases Found",
        "description": "No cases found for givrn national id",
        "dateTime": "2021-05-29T05:58:02Z",
        "transactionId": "44b32520-61ee-47b4-907d-fa15869f3c4d"
      }


  - ### 500.json
    <details>
    <summary></summary>
    
    ```json
    {
      "code": 500,
      "message": "Internal Server Error",
      "description": "The (upstream) service is temporarily not available",
      "dateTime": "2021-05-31T06:18:02Z",
      "transactionId": "48n32920-69ne-47b4-907d-fa15869f3c4d"
    }

  - ### 503.json
    <details>
    <summary></summary>
    
    ```json
    {
      "code": 503,
      "message": "Service unavailable",
      "description": "The (upstream) service is temporarily not available",
      "dateTime": "2021-05-31T06:18:02Z",
      "transactionId": "48n32920-69ne-47b4-907d-fa15869f3c4d"
    }
  - ### get-reports-reponse.json
    <details>
    <summary></summary>
    
    ```json
    {
      "reports": [
        {
          "state": "texas",
          "report": {
            "total": 46744,
            "positive": 436744,
            "recovered": 433744,
            "death": 300,
            "vaccinated": 7847848
          }
        }
      ]
    }

- ## covid-eapi-types
  - ### covid-eapi-types.raml
    <details>
    <summary></summary>
    
    ```raml
    #%RAML 1.0 Library
    usage: Types for uho sapi
    types:
      get-reports-response-type:
        reports:
          type: array
          items:
            type: object
            properties:
              state:
                type: string
                example: "texas"
              report:
                type: object
                properties:
                  total:
                    type: number
                    example: 46744
                  positive:
                    type: number
                    example: 436744
                  recovered:
                    type: number
                    example: 433744
                  death:
                    type: number
                    example: 300
                  vaccinated:
                    type: number
                    example: 7847848

  - ### state-query-param.raml
    <details>
    <summary></summary>
    
    ```raml
    #%RAML 1.0 Trait
    queryParameters:
      state: 
        type: string
        displayName: state
        required: false
        example: "texas"

- ## covid-eapi.raml
  <details>
  <summary></summary>
  
  ```raml
  #%RAML 1.0
  title: covid-eapi
  version: v1
  baseUri: http://{environment}/covid/{version}/
  baseUriParameters:
    environment:
      description: DEV, TEST, UAT, PROD
      enum: ["uho-dev-covid-eapi.us-e2.cloudhub.io","uho-test-covid-eapi.us-e2.cloudhub.io", "uho-uat-covid-eapi.us-e2.cloudhub.io", "uho-prod-covid-eapi.us-e2.cloudhub.io"]
  traits:
    client-id-header: !include exchange_modules/d896f4a8-62ed-4bab-9db4-5db7ac45ebd4/client-id-required/1.0.0/client-id-required.raml
    transaction-header: !include exchange_modules/d896f4a8-62ed-4bab-9db4-5db7ac45ebd4/transaction-header/1.0.0/transaction-header.raml
    correlation-id-header: !include exchange_modules/d896f4a8-62ed-4bab-9db4-5db7ac45ebd4/correlation-id-header/1.0.0/correlation-id-header.raml
    state-query-param: !include covid-eapi-types/state-query-param.raml
  uses:
    resource-types: exchange_modules/d896f4a8-62ed-4bab-9db4-5db7ac45ebd4/resource-types/1.0.0/resource-types.raml
    common-data-types: exchange_modules/d896f4a8-62ed-4bab-9db4-5db7ac45ebd4/common-data-types/1.0.0/common-data-types.raml
    covid-eapi-types: covid-eapi-types/covid-eapi-types.raml
  /reports:
    get:
      description: To get covid reports
      is: [client-id-header,correlation-id-header,transaction-header, state-query-param]
      responses:
        200:
          body:
            application/json:
            application/xml:
              type: covid-eapi-types.get-reports-response-type
              example: !include examples/get-reports-response.json
        503:
            body:
              application/json:
              application/xml:
                type: common-data-types.errorType
                example: !include examples/503.json     
        500:
          body:
              application/json:
              application/xml:
                type: common-data-types.errorType
                example: !include examples/500.json
  /health-check:
    get:
    type:
      resource-types.all: 
        getResponseType: string


# <p align="center">Implementation of mss-covid-eapi</p>  
# mss-covid-eapi
- ## src\main
  - ### mule
    - #### implementations
      - ##### health-check.xml
        <details>
        <summary></summary>
        
        ```xml
        <mule xmlns:custom-logger="http://www.mulesoft.org/schema/mule/custom-logger" xmlns:ee="http://www.mulesoft.org/schema/mule/ee/core" xmlns="http://www.mulesoft.org/schema/mule/core" xmlns:doc="http://www.mulesoft.org/schema/mule/documentation" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.mulesoft.org/schema/mule/core http://www.mulesoft.org/schema/mule/core/current/mule.xsd http://www.mulesoft.org/schema/mule/ee/core http://www.mulesoft.org/schema/mule/ee/core/current/mule-ee.xsd http://www.mulesoft.org/schema/mule/custom-logger http://www.mulesoft.org/schema/mule/custom-logger/current/mule-custom-logger.xsd">
            <flow name="get:\health-check:uhub-sapi-config">
                <ee:transform doc:name="Set Response" doc:id="bb1cca78-967f-4df7-a628-58b320bf27b0">
                    <ee:message>
                        <ee:set-payload>
                            <![CDATA[ %dw 2.0 output application/json --- { message: app.name ++ " is alive and kicking" } ]]>
                        </ee:set-payload>
                    </ee:message>
                </ee:transform>
            </flow>
        </mule>

      - ##### get-covid-reports.xml
        <details>
        <summary></summary>
        
        ```xml
        <mule xmlns:db="http://www.mulesoft.org/schema/mule/db" xmlns:ee="http://www.mulesoft.org/schema/mule/ee/core" xmlns="http://www.mulesoft.org/schema/mule/core" xmlns:doc="http://www.mulesoft.org/schema/mule/documentation" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.mulesoft.org/schema/mule/core http://www.mulesoft.org/schema/mule/core/current/mule.xsd http://www.mulesoft.org/schema/mule/ee/core http://www.mulesoft.org/schema/mule/ee/core/current/mule-ee.xsd http://www.mulesoft.org/schema/mule/db http://www.mulesoft.org/schema/mule/db/current/mule-db.xsd">
            <flow name="get:\reports:covid-eapi-config">
                <ee:transform doc:name="Set Variables" doc:id="2ef24f9a-2d5c-4377-90b6-716269cf3e1d">
                    <ee:message> </ee:message>
                    <ee:variables>
                        <ee:set-variable variableName="state"><![CDATA[ attributes.queryParams.state default "" ]]></ee:set-variable>
                        <ee:set-variable variableName="acceptType"><![CDATA[ attributes.headers.Accept ]]></ee:set-variable>
                        <ee:set-variable variableName="correlationId"><![CDATA[ attributes.headers.'x-correlation-id' default "" ]]></ee:set-variable>
                        <ee:set-variable variableName="uhubHeaderParameters"><![CDATA[ %dw 2.0 output application/java --- { "client_id": Mule::p('ehub.sapi.client_id'), "client_secret": Mule::p('ehub.sapi.client_secret'), "x-transaction-id": vars.transactionId, "x-correlation-id": attributes.headers.'x-correlation-id' default "" } ]]></ee:set-variable>
                    </ee:variables>
                </ee:transform>
                <logger level="INFO" doc:name="Start Log" doc:id="a8cf8ef8-fb2e-436e-b913-d30020a95c44" message="transactionId: #[vars.transactionId]], correlationId: #[vars.correlationId], message: Started get reports flow and received query parameter is - #[attributes.queryParams.state]""/>
                <flow-ref doc:name="get-covid-case-reports-by-state" doc:id="4310de60-a857-456a-a60e-7a847a5ccb2d" name="get-covid-case-reports-by-state"/>
                <choice doc:name="Choice" doc:id="bc1052fe-768d-4fda-a6c5-b3ce362b3af0">
                    <when expression="#[vars.acceptType == &quot;application/xml&quot;]">
                        <ee:transform xmlns:ee="http://www.mulesoft.org/schema/mule/ee/core" xsi:schemaLocation="http://www.mulesoft.org/schema/mule/ee/core http://www.mulesoft.org/schema/mule/ee/core/current/mule-ee.xsd" doc:name="Set XML Response">
                            <ee:message>
                                <ee:set-payload>
                                    <![CDATA[ %dw 2.0 output application/xml --- { reports: payload map { "state": $.state, "report": { "total": $.reports.total default 0, "positive": $.reports.positive default 0, "recovered": $.reports.recovered default 0, "death": $.reports.death default 0, "vaccinated": $.reports.vaccinated default 0 } } } ]]>
                                </ee:set-payload>
                            </ee:message>
                        </ee:transform>
                    </when>
                    <otherwise>
                        <ee:transform doc:name="Set JSON Response" doc:id="e113cb27-fea5-4914-ba5c-68dfb90f9bb7" xsi:schemaLocation="http://www.mulesoft.org/schema/mule/ee/core http://www.mulesoft.org/schema/mule/ee/core/current/mule-ee.xsd">
                            <ee:message>
                                <ee:set-payload>
                                    <![CDATA[ %dw 2.0 output application/json --- { reports: payload map { "state": $.state, "report": { "total": $.reports.total default 0, "positive": $.reports.positive default 0, "recovered": $.reports.recovered default 0, "death": $.reports.death default 0, "vaccinated": $.reports.vaccinated default 0 } } } ]]>
                                </ee:set-payload>
                            </ee:message>
                        </ee:transform>
                    </otherwise>
                </choice>
                <logger level="INFO" doc:name="End Log" doc:id="a78c29b1-fc4f-49c9-922d-0207a62d8143" message="transactionID: #[vars.transactionId]], correlationID: #[vars.correlationID], message: Completed get reports flow, payload: #[payload]""/>
            </flow>
        </mule>

    - #### subflows
      - ##### uhub-sapi-rest-calls.xml
       <details>
        <summary></summary>
        
        ```xml
        <mule xmlns:http="http://www.mulesoft.org/schema/mule/http" xmlns="http://www.mulesoft.org/schema/mule/core" xmlns:doc="http://www.mulesoft.org/schema/mule/documentation" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.mulesoft.org/schema/mule/core http://www.mulesoft.org/schema/mule/core/current/mule.xsd http://www.mulesoft.org/schema/mule/http http://www.mulesoft.org/schema/mule/http/current/mule-http.xsd">
            <sub-flow name="get-covid-case-reports-by-state" doc:id="1b0fd322-c4f2-4a06-87af-aee298269d2f">
                <logger level="DEBUG" doc:name="Before Backend Call" doc:id="912bec8e-3dbb-4965-bbec-ba0586d6fbbe" message="transactionID: #[vars.transactionId]], correlationID: #[vars.correlationID], message: &quot;Before get reports uhub sapi service call&quot;"/>
                <http:request method="GET" doc:name="Get Covid Case By NationalID" doc:id="47e22801-5d3e-4526-bd0a-956731483829" config-ref="HTTP_UHUB_SAPI_Request_configuration" path="/${secure::ehub.sapi.subpath.reports}">
                    <http:headers>
                        <![CDATA[ #[vars.uhubHeaderParameters] ]]>
                    </http:headers>
                    <http:query-params>
                        <![CDATA[ #[{ &quot;state&quot;: vars.state }] ]]>
                    </http:query-params>
                    <http:response-validator>
                        <http:success-status-code-validator values="200,404"/>
                    </http:response-validator>
                </http:request>
                <logger level="DEBUG" doc:name="After Backend Call" doc:id="ecdda797-65e5-47c0-8e1b-f98ec9ce6069" message="transactionID: #[vars.transactionId]], correlationID: #[vars.correlationID], message: &quot;After get reports uhub sapi service call&quot;"/>
            </sub-flow>
        </mule>

    - #### global.xml
      <details>
      <summary></summary>
      
      ```xml
      <mule xmlns:api-gateway="http://www.mulesoft.org/schema/mule/api-gateway" xmlns:db="http://www.mulesoft.org/schema/mule/db" xmlns:secure-properties="http://www.mulesoft.org/schema/mule/secure-properties" xmlns:apikit="http://www.mulesoft.org/schema/mule/mule-apikit" xmlns:http="http://www.mulesoft.org/schema/mule/http" xmlns="http://www.mulesoft.org/schema/mule/core" xmlns:doc="http://www.mulesoft.org/schema/mule/documentation" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.mulesoft.org/schema/mule/core http://www.mulesoft.org/schema/mule/core/current/mule.xsd http://www.mulesoft.org/schema/mule/api-gateway http://www.mulesoft.org/schema/mule/api-gateway/current/mule-api-gateway.xsd http://www.mulesoft.org/schema/mule/mule-apikit http://www.mulesoft.org/schema/mule/mule-apikit/current/mule-apikit.xsd http://www.mulesoft.org/schema/mule/http http://www.mulesoft.org/schema/mule/http/current/mule-http.xsd http://www.mulesoft.org/schema/mule/secure-properties http://www.mulesoft.org/schema/mule/secure-properties/current/mule-secure-properties.xsd http://www.mulesoft.org/schema/mule/db http://www.mulesoft.org/schema/mule/db/current/mule-db.xsd">
          <http:listener-config name="covid-eapi-httpListenerConfig" basePath="covid">
              <http:listener-connection host="0.0.0.0" port="${http.port}"/>
          </http:listener-config>
          <apikit:config name="covid-eapi-config" api="covid-eapi.raml" outboundHeadersMapName="outboundHeaders" httpStatusVarName="httpStatus"/>
          <secure-properties:config name="Secure_Properties_Config" doc:name="Secure Properties Config" doc:id="aaa8e37c-89a0-48d0-bdf3-60132f8b13d4" file="${env}.yaml" key="${enc.key}"/>
          <http:request-config name="HTTP_UHUB_SAPI_Request_configuration" doc:name="HTTP Request configuration" doc:id="18f84cf5-477b-476b-afd9-eb714dd1d961" basePath="/${secure::ehub.sapi.basepath}">
              <http:request-connection host="${secure::ehub.sapi.host}" port="${secure::ehub.sapi.port}"/>
          </http:request-config>
          <!--  	<api-gateway:autodiscovery apiId="${instance.id}" ignoreBasePath="true" doc:name="API Autodiscovery" doc:id="50f841cd-bc73-4655-aa43-3aba9ef19e51" flowRef="covid-eapi-main" />
            -->
      </mule>

    - #### global-error-handler.xml
      <details>
      <summary></summary>
    
      ```xml
      <mule xmlns:ee="http://www.mulesoft.org/schema/mule/ee/core" xmlns="http://www.mulesoft.org/schema/mule/core" xmlns:doc="http://www.mulesoft.org/schema/mule/documentation" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.mulesoft.org/schema/mule/ee/core http://www.mulesoft.org/schema/mule/ee/core/current/mule-ee.xsd http://www.mulesoft.org/schema/mule/core http://www.mulesoft.org/schema/mule/core/current/mule.xsd">
      <error-handler name="global-error-handler" doc:id="77f85497-bc5a-4406-9826-4708e19ce808">
          <on-error-propagate type="APIKIT:BAD_REQUEST" enableNotifications="true" logException="true" doc:name="On Error Propagate" doc:id="ef3cf218-baa9-4253-9657-bc8168fb8815">
              <set-variable value="#[400]" doc:name="Set HTTP Status - 400" doc:id="5e7ccc4e-d9ea-4e3d-8442-c14ecb513309" variableName="httpStatus"/>
              <set-variable value="Bad request" doc:name="set Error Message" doc:id="3cdb7dd5-31bf-4c24-b910-101a9242ea20" variableName="errorMessage"/>
              <set-variable value="#[(((error.description default "" replace "[" with "") replace "]" with "") splitBy "\n")]" doc:name="Set Error Description" doc:id="e094d07d-c5fb-4083-8f00-6b1a40369929" variableName="errorDescription"/>
              <flow-ref doc:name="global-prepare-error-response-sub-flow" doc:id="eb11b457-b296-4d7e-9a27-6d2bdaca019a" name="global-prepare-error-response-sub-flow"/>
          </on-error-propagate>
          <on-error-propagate type="APIKIT:METHOD_NOT_ALLOWED" enableNotifications="true" logException="true" doc:name="On Error Propagate" doc:id="2dc25069-521b-4119-b9d9-8a2811417ff1">
              <set-variable value="#[405]" doc:name="Set HTTP Status - 405" doc:id="3a96b68f-77cf-4b1e-a185-d4384e1bf5b8" variableName="httpStatus"/>
              <set-variable value="Method Not Allowed" doc:name="Set Error Message" doc:id="b5b32a4e-135b-42b7-85fd-ef756edd3471" variableName="errorMessage"/>
              <set-variable value="The method specified in the request is not allowed for this resource" doc:name="Set Error Description" doc:id="294531c7-7f83-4fef-a1ae-520bd7f2b25b" variableName="errorDescription"/>
              <flow-ref doc:name="global-prepare-error-response-sub-flow" doc:id="7521a9ad-5006-43b3-acae-46284c39aac7" name="global-prepare-error-response-sub-flow"/>
          </on-error-propagate>
          <on-error-propagate type="APIKIT:NOT_ACCEPTABLE" enableNotifications="true" logException="true" doc:name="On Error Propagate" doc:id="df815d08-9957-4e68-b3ca-f864a5181115">
              <set-variable value="#[406]" doc:name="Set HTTP Status - 406" doc:id="08259cfe-fbf3-441e-951f-ab431c5f57b0" variableName="httpStatus"/>
              <set-variable value="Not Acceptable" doc:name="Set Error Message" doc:id="ca6b5800-1bfb-43d9-9ada-7d372d4aa0e7" variableName="errorMessage"/>
              <set-variable value="The resource identified by the request is not capable of generating response entities according to the request accept headers" doc:name="Set Error Description" doc:id="4e301aef-84af-4c43-923d-abc306306058" variableName="errorDescription"/>
              <flow-ref doc:name="global-prepare-error-response-sub-flow" doc:id="0a0586a1-749f-4b09-8ff1-ed299bbaeace" name="global-prepare-error-response-sub-flow"/>
          </on-error-propagate>
          <on-error-propagate type="APIKIT:NOT_FOUND" enableNotifications="true" logException="true" doc:name="On Error Propagate" doc:id="68d40e8d-5285-41e2-b299-01d614c1b656">
              <set-variable value="#[404]" doc:name="Set HTTP Status - 404" doc:id="de8bb9db-b4f8-4cd3-9f0a-348869331950" variableName="httpStatus"/>
              <set-variable value="Not found" doc:name="Set Error Message" doc:id="23037b6a-29f1-41d7-867f-58af686326e4" variableName="errorMessage"/>
              <set-variable value="The server has not found anything matching the Request-URI" doc:name="Set Error Description" doc:id="e6d889a0-2b53-455d-aadf-0ba15fb6bf91" variableName="errorDescription"/>
              <flow-ref doc:name="global-prepare-error-response-sub-flow" doc:id="4d71d4db-ea53-4c48-ad7f-4d6ce2981972" name="global-prepare-error-response-sub-flow"/>
          </on-error-propagate>
          <on-error-propagate type="APIKIT:UNSUPPORTED_MEDIA_TYPE" enableNotifications="true" logException="true" doc:name="On Error Propagate" doc:id="d21e6866-91a6-459d-9d9c-bf2eb2be61d1">
              <set-variable value="#[415]" doc:name="Set HTTP Status - 415" doc:id="f51535ac-dcb1-4bc0-a299-1f754a1d26d2" variableName="httpStatus"/>
              <set-variable value="Unsupported Media Type" doc:name="Set Error Message" doc:id="3da66f8b-e537-4dab-af8d-c1e8a5e1f8eb" variableName="errorMessage"/>
              <set-variable value="The server is refusing to service the request because the entity of the request is in a format not supported by the requested resource for the requested method" doc:name="Set Error Description" doc:id="80b94b71-8bb0-45e5-9c18-6a0533fd4d92" variableName="errorDescription"/>
              <flow-ref doc:name="global-prepare-error-response-sub-flow" doc:id="169c0ef3-77e3-4d16-8d61-2f3c10f874ff" name="global-prepare-error-response-sub-flow"/>
          </on-error-propagate>
          <on-error-propagate type="APIKIT:UNSUPPORTED_URI" enableNotifications="true" logException="true" doc:name="On Error Propagate" doc:id="bb26b017-d319-4e5c-a1d7-dfa15d17b289">
              <set-variable value="#[404]" doc:name="Set HTTP Status - 404" doc:id="7b330f8e-9b0c-45a7-8b84-0817ef4b5d9a" variableName="httpStatus"/>
              <set-variable value="Not Found" doc:name="Set Error Message" doc:id="b5b5c4d3-0047-43dc-860b-e48abf6d27fc" variableName="errorMessage"/>
              <set-variable value="The URI requested by the client is invalid" doc:name="Set Error Description" doc:id="55a54f47-c4d2-47a9-8a6d-c34857fe464d" variableName="errorDescription"/>
              <flow-ref doc:name="global-prepare-error-response-sub-flow" doc:id="a1437fa1-9e55-4a6e-8cd4-cbd08b22f4a9" name="global-prepare-error-response-sub-flow"/>
          </on-error-propagate>
          <on-error-propagate type="APIKIT:REQUEST_ENTITY_TOO_LARGE" enableNotifications="true" logException="true" doc:name="On Error Propagate" doc:id="ba2583d3-1d45-4b13-b4d4-cd922c9e9d90">
              <set-variable value="#[413]" doc:name="Set HTTP Status - 413" doc:id="268bae5b-4746-48d1-94f1-d4789f6c0979" variableName="httpStatus"/>
              <set-variable value="Request Entity Too Large" doc:name="Set Error Message" doc:id="a4c7c30e-35b2-4209-8f3c-7dca7ee77450" variableName="errorMessage"/>
              <set-variable value="The server is refusing to process a request because the request payload is larger than the server is willing or able to process" doc:name="Set Error Description" doc:id="67c22d26-f1fb-4c89-85fb-56f55cb4e44b" variableName="errorDescription"/>
              <flow-ref doc:name="global-prepare-error-response-sub-flow" doc:id="7232e4bc-6b7c-4582-93c2-79dbbf3ab1e0" name="global-prepare-error-response-sub-flow"/>
          </on-error-propagate>
          <on-error-propagate type="APIKIT:UNSUPPORTED_MEDIA_TYPE" enableNotifications="true" logException="true" doc:name="On Error Propagate" doc:id="a316a858-03c5-43f8-b0a0-49008b9936c5">
              <set-variable value="#[415]" doc:name="Set HTTP Status - 415" doc:id="1ab3035b-8f38-4d94-9330-535891fdcb95" variableName="httpStatus"/>
              <set-variable value="Unsupported Media Type" doc:name="Set Error Message" doc:id="7d04d4d8-e8af-4eeb-8d42-1ee59e1d4a5d" variableName="errorMessage"/>
              <set-variable value="The server is refusing to service the request because the entity of the request is in a format not supported by the requested resource for the requested method" doc:name="Set Error Description" doc:id="6deab318-3b65-486b-aad7-24ee5a0819cd" variableName="errorDescription"/>
              <flow-ref doc:name="global-prepare-error-response-sub-flow" doc:id="05e5d126-4498-4fd0-8624-42370a924b65" name="global-prepare-error-response-sub-flow"/>
          </on-error-propagate>
          <on-error-propagate type="APIKIT:UNSUPPORTED_URI" enableNotifications="true" logException="true" doc:name="On Error Propagate" doc:id="3f31711e-31e1-49a7-811f-02b4b81a5e14">
              <set-variable value="#[404]" doc:name="Set HTTP Status - 404" doc:id="99fa108b-ae18-444d-b9cc-8d53686fb884" variableName="httpStatus"/>
              <set-variable value="Not Found" doc:name="Set Error Message" doc:id="9ec6e2c2-03da-4d4a-82f3-b7f64fc9612e" variableName="errorMessage"/>
              <set-variable value="The URI requested by the client is invalid" doc:name="Set Error Description" doc:id="4c1c4f43-31de-499d-9a01-dcc7485f3694" variableName="errorDescription"/>
              <flow-ref doc:name="global-prepare-error-response-sub-flow" doc:id="c90654d7-104e-48de-a9d0-5b2a47643c87" name="global-prepare-error-response-sub-flow"/>
          </on-error-propagate>
          <on-error-propagate type="APIKIT:BAD_REQUEST" enableNotifications="true" logException="true" doc:name="On Error Propagate" doc:id="e860437f-c81b-4ef0-a9eb-c9dd0be944d7">
              <set-variable value="#[400]" doc:name="Set HTTP Status - 400" doc:id="2a74794e-2317-4f6b-88dd-292736174af0" variableName="httpStatus"/>
              <set-variable value="Bad Request" doc:name="Set Error Message" doc:id="e79cf8b5-2e0d-46c4-b07f-2d1e4bb8bb21" variableName="errorMessage"/>
              <set-variable value="The request could not be understood by the server due to malformed syntax. The client SHOULD NOT repeat the request without modifications" doc:name="Set Error Description" doc:id="c44d8c1a-0ac5-4ab7-a2b2-f1e80e1e3546" variableName="errorDescription"/>
              <flow-ref doc:name="global-prepare-error-response-sub-flow" doc:id="1e140670-90f2-4ae9-b362-30a8b614dcb1" name="global-prepare-error-response-sub-flow"/>
          </on-error-propagate>
          <on-error-propagate type="APIKIT:BAD_REQUEST" enableNotifications="true" logException="true" doc:name="On Error Propagate" doc:id="8c1cc015-58b8-4d02-8592-78f2229071cf">
              <set-variable value="#[400]" doc:name="Set HTTP Status - 400" doc:id="e5efda35-bc00-4e84-a0fc-ee369b448b6a" variableName="httpStatus"/>
              <set-variable value="Bad Request" doc:name="Set Error Message" doc:id="b1687251-d8fb-45b3-bb5f-c84709db4dbb" variableName="errorMessage"/>
              <set-variable value="The request could not be understood by the server due to malformed syntax. The client SHOULD NOT repeat the request without modifications" doc:name="Set Error Description" doc:id="61bca292-8f21-4953-853e-e73bbf7ac2bc" variableName="errorDescription"/>
              <flow-ref doc:name="global-prepare-error-response-sub-flow" doc:id="0ebc2452-64e0-4290-9a70-bf15f8b58926" name="global-prepare-error-response-sub-flow"/>
          </on-error-propagate>
          <on-error-propagate type="APIKIT:BAD_REQUEST" enableNotifications="true" logException="true" doc:name="On Error Propagate" doc:id="5261f1e2-4d53-44e0-8262-d6e19779809f">
              <set-variable value="#[400]" doc:name="Set HTTP Status - 400" doc:id="2d5f2ff3-6aa9-4dc1-b623-4f26824d3c6d" variableName="httpStatus"/>
              <set-variable value="Bad Request" doc:name="Set Error Message" doc:id="cfd4a936-0af5-4051-982e-4f6ff390c462" variableName="errorMessage"/>
              <set-variable value="The request could not be understood by the server due to malformed syntax. The client SHOULD NOT repeat the request without modifications" doc:name="Set Error Description" doc:id="b95d4d72-18e1-4948-bdbf-ff066170cb4e" variableName="errorDescription"/>
              <flow-ref doc:name="global-prepare-error-response-sub-flow" doc:id="df48ee4e-e7c1-4d68-9884-4f7e935f02a0" name="global-prepare-error-response-sub-flow"/>
          </on-error-propagate>
          <on-error-propagate type="APIKIT:UNSUPPORTED_URI" enableNotifications="true" logException="true" doc:name="On Error Propagate" doc:id="50b9f878-82f5-4b2d-90c9-af81a33f9275">
              <set-variable value="#[404]" doc:name="Set HTTP Status - 404" doc:id="c6be9f59-7c3d-4e44-af34-d4d93a98b8b4" variableName="httpStatus"/>
              <set-variable value="Not Found" doc:name="Set Error Message" doc:id="c7619241-e8a7-4a69-83d5-63dd7a9b3927" variableName="errorMessage"/>
              <set-variable value="The URI requested by the client is invalid" doc:name="Set Error Description" doc:id="3b30fb0e-8327-4c6f-bd77-07bcbae3bde2" variableName="errorDescription"/>
              <flow-ref doc:name="global-prepare-error-response-sub-flow" doc:id="6a4f3946-ba95-41f5-a429-00e93fd0e43c" name="global-prepare-error-response-sub-flow"/>
          </on-error-propagate>
          <on-error-propagate type="APIKIT:UNSUPPORTED_MEDIA_TYPE" enableNotifications="true" logException="true" doc:name="On Error Propagate" doc:id="372af52d-fa0f-44d1-8774-8e0125e81ae7">
              <set-variable value="#[415]" doc:name="Set HTTP Status - 415" doc:id="4bdc80ec-c9c2-452e-b37a-42e4f7b7c2de" variableName="httpStatus"/>
              <set-variable value="Unsupported Media Type" doc:name="Set Error Message" doc:id="950e02cb-0308-44af-9384-3d0e04de3611" variableName="errorMessage"/>
              <set-variable value="The server is refusing to service the request because the entity of the request is in a format not supported by the requested resource for the requested method" doc:name="Set Error Description" doc:id="26a28d47-7cf0-4a24-93e2-fa780fb9b2b7" variableName="errorDescription"/>
              <flow-ref doc:name="global-prepare-error-response-sub-flow" doc:id="99e04c35-ded0-4f6d-85d8-3740a85ba308" name="global-prepare-error-response-sub-flow"/>
          </on-error-propagate>
          <on-error-propagate type="APIKIT:UNSUPPORTED_URI" enableNotifications="true" logException="true" doc:name="On Error Propagate" doc:id="8f65f014-89e3-49e3-b73a-6281c27c1748">
              <set-variable value="#[404]" doc:name="Set HTTP Status - 404" doc:id="ce44dc24-e1a0-4b82-b011-3b494a5f0437" variableName="httpStatus"/>
              <set-variable value="Not Found" doc:name="Set Error Message" doc:id="3b5d748a-fc65-4d3d-93cb-fc8554f3d0f7" variableName="errorMessage"/>
              <set-variable value="The URI requested by the client is invalid" doc:name="Set Error Description" doc:id="ef300b34-55a4-4701-8604-4c1f225d8e02" variableName="errorDescription"/>
              <flow-ref doc:name="global-prepare-error-response-sub-flow" doc:id="d9931262-d6bc-4fe5-8550-6c04b8c585ff" name="global-prepare-error-response-sub-flow"/>
          </on-error-propagate>
          <on-error-propagate type="APIKIT:UNSUPPORTED_MEDIA_TYPE" enableNotifications="true" logException="true" doc:name="On Error Propagate" doc:id="bfab1c18-6c5b-42d9-bcc8-cba88b600499">
              <set-variable value="#[415]" doc:name="Set HTTP Status - 415" doc:id="fd9ebc9f-00e2-460a-8448-09d34509c61d" variableName="httpStatus"/>
              <set-variable value="Unsupported Media Type" doc:name="Set Error Message" doc:id="7b5c540e-79e4-4d7a-8376-eb4d226bcb95" variableName="errorMessage"/>
              <set-variable value="The server is refusing to service the request because the entity of the request is in a format not supported by the requested resource for the requested method" doc:name="Set Error Description" doc:id="b1118f6c-5a29-4149-8b8a-3e6a38086806" variableName="errorDescription"/>
              <flow-ref doc:name="global-prepare-error-response-sub-flow" doc:id="f0a65a7b-238a-46f5-80d2-9d2d5e0d8687" name="global-prepare-error-response-sub-flow"/>
          </on-error-propagate>
          <on-error-propagate type="APIKIT:NOT_ACCEPTABLE" enableNotifications="true" logException="true" doc:name="On Error Propagate" doc:id="190dc8f7-65b0-4a43-a44c-19ffda5de911">
              <set-variable value="#[406]" doc:name="Set HTTP Status - 406" doc:id="0a890cad-72c4-49d8-9dbd-dcb0d7f797d1" variableName="httpStatus"/>
              <set-variable value="Not Acceptable" doc:name="Set Error Message" doc:id="cd7fd126-f2c1-4c1d-9c54-11cb3150c49a" variableName="errorMessage"/>
              <set-variable value="The resource identified by the request is not capable of generating response entities according to the request accept headers" doc:name="Set Error Description" doc:id="504a1253-2051-4768-8802-af9dd8b7b100" variableName="errorDescription"/>
              <flow-ref doc:name="global-prepare-error-response-sub-flow" doc:id="b3d08fe8-7579-4c08-ae8a-067a06314c17" name="global-prepare-error-response-sub-flow"/>
          </on-error-propagate>
      </error-handler>
  </mule>

    - #### covid-eapi.xml
      <details>
      <summary></summary>
      
      ```xml
      <mule xmlns:db="http://www.mulesoft.org/schema/mule/db" xmlns:doc="http://www.mulesoft.org/schema/mule/documentation" xmlns="http://www.mulesoft.org/schema/mule/core" xmlns:apikit="http://www.mulesoft.org/schema/mule/mule-apikit" xmlns:http="http://www.mulesoft.org/schema/mule/http" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.mulesoft.org/schema/mule/core http://www.mulesoft.org/schema/mule/core/current/mule.xsd http://www.mulesoft.org/schema/mule/http http://www.mulesoft.org/schema/mule/http/current/mule-http.xsd http://www.mulesoft.org/schema/mule/mule-apikit http://www.mulesoft.org/schema/mule/mule-apikit/current/mule-apikit.xsd http://www.mulesoft.org/schema/mule/db http://www.mulesoft.org/schema/mule/db/current/mule-db.xsd">
          <flow name="covid-eapi-main">
              <http:listener config-ref="covid-eapi-httpListenerConfig" path="/v1/*">
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
              <set-variable value="#[attributes.headers.'x-transaction-id' default uuid()]" doc:name="Set transactionId" doc:id="4809d9ca-ee5b-45d0-beed-341d87950924" variableName="transactionId"/>
              <apikit:router config-ref="covid-eapi-config"/>
              <error-handler ref="global-error-handler"/>
          </flow>
          <flow name="covid-eapi-console">
              <http:listener config-ref="covid-eapi-httpListenerConfig" path="/console/*">
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
              <apikit:console config-ref="covid-eapi-config"/>
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
      
      ehub.sapi:
        host: "uho-dev-uhub-sapi.us-e2.cloudhub.io"
        port: "80"
        basepath: "covid/v1"
        client_id: ""
        client_secret: ""
        subpath:
          reports: "reports"

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
    "minMuleVersion": "4.3.0"
  }

- ## pom.xml
  <details>
  <summary></summary>
  
  ```xml
  <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
      <modelVersion>4.0.0</modelVersion>
      <groupId>com.mycompany</groupId>
      <artifactId>covid-eapi</artifactId>
      <version>1.0.0-SNAPSHOT</version>
      <packaging>mule-application</packaging>
      <name>covid-eapi</name>
      <properties>
          <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
          <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
          <app.runtime>4.3.0-20210609</app.runtime>
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
              </plugin>
          </plugins>
      </build>
      <dependencies>
          <dependency>
              <groupId>org.mule.connectors</groupId>
              <artifactId>mule-http-connector</artifactId>
              <version>1.5.25</version>
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
