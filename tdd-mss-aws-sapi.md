# <p align="center">Raml Code</p>
# mss-aws-sapi 
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

  - ### upload-document-request.json
    <details>
    <summary></summary>
    
    ```json
    {
      "bucketName": "uho-covid-docs",
      "folderPath": "identity",
      "fileName": "35653.png",
      "document": "iVBORw0KGgoAAAANSUhEUgAAAOIAAACRCAYAAADepmS4AAAAAXNSR0IArs4c6QAAAARnQU1BAACxjw"
    }

  - ### upload-document-response.json
    <details>
    <summary></summary>
    
    ```json
    {
      "code": 200,
      "message": "Success",
      "description": "Document uploaded successfully",
      "dateTime": "2021-07-29T05:58:02Z",
      "transactionId": "44b32520-61ee-47b4-907d-fa15869f3c4d"
    }

- ## aws-sapi-types
  - ### aws-sapi-types.raml
    <details>
    <summary></summary>
    
    ```raml
    #%RAML 1.0 Library
    usage: Tyoes for aws-sapi
    types:
      upload-document-request-type:
        type: object
        description: This type used to upload document request
        properties:
          bucketName: 
            type: string
            required: true
            example: "uho-covid-docs"
          folderPath: 
            type: string
            required: true
            example: "identity"
          fileName: 
            type: string
            required: true
            example: "35653.png"
          document: 
            type: string
            required: true
            example: "iVBORw0KGgoAAAANSUhEUgAAAOIAAACRCAYAAADepmS4AAAAAXNSR0IArs4c6QAAAARnQU1BAACxjw"
      upload-document-response-type:
        description: This type used to respond back the status of upload document request
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

- ## aws-sapi.raml
  <details>
  <summary></summary>

  ```raml
  #%RAML 1.0
  title: aws-sapi
  version: v1
  baseUri: http://{environment}/covid/{version}/
  baseUriParameters:
    environment:
      description: DEV, TEST, UAT, PROD
      enum: ["uho-dev-aws-sapi.us-e2.cloudhub.io","uho-test-aws-sapi.us-e2.cloudhub.io", "uho-uat-aws-sapi.us-e2.cloudhub.io", "uho-prod-aws-sapi.us-e2.cloudhub.io"]
  traits:
    client-id-header: !include exchange_modules/d896f4a8-62ed-4bab-9db4-5db7ac45ebd4/client-id-required/1.0.0/client-id-required.raml
    transaction-header: !include exchange_modules/d896f4a8-62ed-4bab-9db4-5db7ac45ebd4/transaction-header/1.0.0/transaction-header.raml
    correlation-id-header: !include exchange_modules/d896f4a8-62ed-4bab-9db4-5db7ac45ebd4/correlation-id-header/1.0.0/correlation-id-header.raml
  uses:
    resource-types: exchange_modules/d896f4a8-62ed-4bab-9db4-5db7ac45ebd4/resource-types/1.0.0/resource-types.raml
    common-data-types: exchange_modules/d896f4a8-62ed-4bab-9db4-5db7ac45ebd4/common-data-types/1.0.0/common-data-types.raml
    aws-sapi-types: aws-sapi-types/aws-sapi-types.raml
  /document:
    post:
      description: To register covid case
      is: [client-id-header,correlation-id-header,transaction-header]
      body:
        application/json:
          type: aws-sapi-types.upload-document-request-type
          example: !include examples/upload-document-request.json
      responses:
        200:
          body:
            application/json:
              type: aws-sapi-types.upload-document-response-type
              example: !include examples/upload-document-response.json
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


# <p align="center">Implementation of mss-aws-sapi</p>  
# mss-aws-sapi
- ## src\main
  - ### mule
    - #### implementations
      - ##### health-check.xml
        <details>
        <summary></summary>
        
        ```xml
        <mule xmlns:custom-logger="http://www.mulesoft.org/schema/mule/custom-logger" xmlns:ee="http://www.mulesoft.org/schema/mule/ee/core" xmlns="http://www.mulesoft.org/schema/mule/core" xmlns:doc="http://www.mulesoft.org/schema/mule/documentation" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.mulesoft.org/schema/mule/core http://www.mulesoft.org/schema/mule/core/current/mule.xsd http://www.mulesoft.org/schema/mule/ee/core http://www.mulesoft.org/schema/mule/ee/core/current/mule-ee.xsd http://www.mulesoft.org/schema/mule/custom-logger http://www.mulesoft.org/schema/mule/custom-logger/current/mule-custom-logger.xsd">
          <flow name="get:\health-check:aws-sapi-config">
            <ee:transform doc:name="Set Response" doc:id="3996a32d-7737-48b6-83fc-8e7ff8ac2071">
              <ee:message>
                <ee:set-payload>
                  <![CDATA[ %dw 2.0 output application/json --- { message: app.name ++ " is alive and kicking" } ]]>
                </ee:set-payload>
              </ee:message>
            </ee:transform>
          </flow>
        </mule>

      - ##### upload-document-to-aws.xml
        <details>
        <summary></summary>
        
        ```xml
        <mule xmlns:s3="http://www.mulesoft.org/schema/mule/s3" xmlns:ee="http://www.mulesoft.org/schema/mule/ee/core" xmlns="http://www.mulesoft.org/schema/mule/core" xmlns:doc="http://www.mulesoft.org/schema/mule/documentation" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.mulesoft.org/schema/mule/core http://www.mulesoft.org/schema/mule/core/current/mule.xsd http://www.mulesoft.org/schema/mule/ee/core http://www.mulesoft.org/schema/mule/ee/core/current/mule-ee.xsd http://www.mulesoft.org/schema/mule/s3 http://www.mulesoft.org/schema/mule/s3/current/mule-s3.xsd">
          <flow name="post:\document:application\json:aws-sapi-config">
            <set-variable value="#[attributes.headers.'x-correlation-id' default ""]" doc:name="Set correlationId" doc:id="44e7bbf0-6f2c-47a9-87f8-7496c6c0ddeb" variableName="correlationId"/>
            <logger level="INFO" doc:name="Start Log" doc:id="d952703c-6885-46e9-ae37-047669cda048" message="transactionID: #[vars.transactionId]], correlationID: #[vars.correlationID], message: &quot;Started upload document to amazon S3 flow&quot;"/>
            <ee:transform doc:name="Transform Base64 And Set inputPayload" doc:id="efc6cc34-5846-4c57-8a58-d77e9f7d0331">
              <ee:message>
                <ee:set-payload>
                  <![CDATA[ %dw 2.0 import fromBase64 from dw::core::Binaries output application/octet-stream --- fromBase64(payload.document) ]]>
                </ee:set-payload>
              </ee:message>
              <ee:variables>
                <ee:set-variable variableName="keyPath">
                  <![CDATA[ payload.folderPath as String ++ "/" ++ payload.fileName as String ]]>
                </ee:set-variable>
                <ee:set-variable variableName="bucketName">
                  <![CDATA[ payload.bucketName ]]>
                </ee:set-variable>
              </ee:variables>
            </ee:transform>
            <!-- <flow-ref doc:name="upload-s3-document" doc:id="df5557e0-72c5-47b9-bf90-eb1df5fa0ea3" name="upload-s3-document" /> -->
            <flow-ref doc:name="upload-s3-document" doc:id="a2916e28-994b-41c9-a22d-1ece73b77716" name="upload-s3-document"/>
            <ee:transform xmlns:ee="http://www.mulesoft.org/schema/mule/ee/core" xsi:schemaLocation="http://www.mulesoft.org/schema/mule/ee/core http://www.mulesoft.org/schema/mule/ee/core/current/mule-ee.xsd" doc:name="Set Response">
              <ee:message>
                <ee:set-payload>
                  <![CDATA[ %dw 2.0 output application/json --- { code: 200, message: &quot;Success&quot;, description: &quot;Document uploaded successfully&quot;, dateTime: now() as String { format: &quot;yyyy-MM-dd'T'HH:mm:ss'Z'&quot; }, transactionId: vars.transactionId } ]]>
                </ee:set-payload>
              </ee:message>
            </ee:transform>
            <logger level="INFO" doc:name="End Log" doc:id="d66e7d59-8ed0-495f-9eab-e560f946063b" message="transactionID: #[vars.transactionId]], correlationID: #[vars.correlationID], message: Completed upload document to amazon S3 flow, payload: #[payload]&quot;"/>
            <error-handler>
              <on-error-continue enableNotifications="true" logException="true" doc:name="On Error Continue" doc:id="f556e5da-49c6-466f-b921-2427e2a5ed47" type="ANY">
                <set-variable value="#[500]" doc:name="Set HTTP Status - 500" doc:id="eed15cb3-51cf-42a8-bf61-e824330d8c28" variableName="httpStatus"/>
                <set-variable value="Internal server error" doc:name="Set errorMessage" doc:id="af011973-3262-4b9e-bb6e-a9c0f642b20b" variableName="errorMessage"/>
                <set-variable value="#[error.description]" doc:name="Set errorDescription" doc:id="57c06d2b-3032-42f9-abce-54ce3b6192d8" variableName="errorDescription"/>
                <flow-ref doc:name="global-prepare-error-response-sub-flow" doc:id="a037d323-cb98-4d4a-9768-1f24cb936a6b" name="global-prepare-error-response-sub-flow"/>
              </on-error-continue>
            </error-handler>
          </flow>
        </mule>

    - #### subflows
      - ##### aws.xml
        <details>
        <summary></summary>
        
        ```xml
        <mule xmlns:s3="http://www.mulesoft.org/schema/mule/s3" xmlns="http://www.mulesoft.org/schema/mule/core" xmlns:doc="http://www.mulesoft.org/schema/mule/documentation" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.mulesoft.org/schema/mule/core http://www.mulesoft.org/schema/mule/core/current/mule.xsd http://www.mulesoft.org/schema/mule/s3 http://www.mulesoft.org/schema/mule/s3/current/mule-s3.xsd">
          <sub-flow name="upload-s3-document" doc:id="14b38c61-b1c6-40f2-8c06-47e371c0139d">
            <logger level="DEBUG" doc:name="Before Backend Call" doc:id="1fdbcdd7-e538-4ed4-b9dc-5d081f7df6ec" message="transactionID: #[vars.transactionId]], correlationID: #[vars.correlationID], message: &quot;Before document upload to S3&quot;, bucketName: #[vars.bucketName], keyPath: #[vars.keyPath]"/>
            <s3:create-object doc:name="Upload Document To Amazon S3" doc:id="ec42e25b-3133-4d86-9392-d3a18288d3b7" bucketName="#[vars.bucketName]" key="#[vars.keyPath]" config-ref="Amazon_S3_Configuration"/>
            <logger level="DEBUG" doc:name="After Backend Call" doc:id="b343a819-829c-43b8-8667-1a1ce5dadc08" message="transactionID: #[vars.transactionId]], correlationID: #[vars.correlationID], message: &quot;After document upload to S3&quot;, payload: #[payload]"/>
          </sub-flow>
        </mule>

    - #### global.xml
      <details>
      <summary></summary>
      
      ```xml
      <mule xmlns:api-gateway="http://www.mulesoft.org/schema/mule/api-gateway" xmlns:s3="http://www.mulesoft.org/schema/mule/s3" xmlns:db="http://www.mulesoft.org/schema/mule/db" xmlns:secure-properties="http://www.mulesoft.org/schema/mule/secure-properties" xmlns:apikit="http://www.mulesoft.org/schema/mule/mule-apikit" xmlns:http="http://www.mulesoft.org/schema/mule/http" xmlns="http://www.mulesoft.org/schema/mule/core" xmlns:doc="http://www.mulesoft.org/schema/mule/documentation" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation=" http://www.mulesoft.org/schema/mule/mule-apikit http://www.mulesoft.org/schema/mule/mule-apikit/current/mule-apikit.xsd http://www.mulesoft.org/schema/mule/http http://www.mulesoft.org/schema/mule/http/current/mule-http.xsd http://www.mulesoft.org/schema/mule/core http://www.mulesoft.org/schema/mule/core/current/mule.xsd http://www.mulesoft.org/schema/mule/secure-properties http://www.mulesoft.org/schema/mule/secure-properties/current/mule-secure-properties.xsd http://www.mulesoft.org/schema/mule/db http://www.mulesoft.org/schema/mule/db/current/mule-db.xsd http://www.mulesoft.org/schema/mule/s3 http://www.mulesoft.org/schema/mule/s3/current/mule-s3.xsd http://www.mulesoft.org/schema/mule/api-gateway http://www.mulesoft.org/schema/mule/api-gateway/current/mule-api-gateway.xsd">
        <secure-properties:config name="Secure_Properties_Config" doc:name="Secure Properties Config" doc:id="aaa8e37c-89a0-48d0-bdf3-60132f8b13d4" file="${env}.yaml" key="${enc.key}"/>
        <apikit:config name="aws-sapi-config" api="aws-sapi.raml" outboundHeadersMapName="outboundHeaders" httpStatusVarName="httpStatus"/>
        <http:listener-config name="HTTP_Listener_config" doc:name="HTTP Listener config" doc:id="5fc4c544-f374-494e-94ec-4d08dee7cccd" basePath="covid">
          <http:listener-connection host="0.0.0.0" port="${http.port}"/>
        </http:listener-config>
        <s3:config name="Amazon_S3_Configuration" doc:name="Amazon S3 Configuration" doc:id="5d492931-c7c4-4e74-8c90-1f7d2009f928">
          <s3:basic-connection accessKey="${secure::amazon.access_key}" secretKey="${secure::amazon.secret_key}" region="${secure::amazon.region}"/>
        </s3:config>
        <!--  	<api-gateway:autodiscovery apiId="${instance.id}" ignoreBasePath="true" doc:name="API Autodiscovery" doc:id="a69de83e-523e-4ced-8833-a43c477ce658" flowRef="aws-sapi-main" />
          -->
      </mule>

    - #### global-error-handler.xml
      <details>
      <summary></summary>
  
      ```xml
      <mule xmlns:ee="http://www.mulesoft.org/schema/mule/ee/core" xmlns="http://www.mulesoft.org/schema/mule/core" xmlns:doc="http://www.mulesoft.org/schema/mule/documentation" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation=" http://www.mulesoft.org/schema/mule/ee/core http://www.mulesoft.org/schema/mule/ee/core/current/mule-ee.xsd http://www.mulesoft.org/schema/mule/core http://www.mulesoft.org/schema/mule/core/current/mule.xsd">
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
      <on-error-propagate type="APIKIT:UNSUPPORTED_MEDIA_TYPE" enableNotifications="true" logException="true" doc:name="On Error Propagate" doc:id="9119834d-7fc5-4d44-b804-abf5665dbb0a">
      <set-variable value="#[415]" doc:name="Set HTTP Status - 415" doc:id="3f2dec04-da5b-47ea-8edc-2de7b1208ea8" variableName="httpStatus"/>
      <set-variable value="Unsupported media type" doc:name="Set Error Message" doc:id="43f81baa-635d-4067-9393-1829f9c56bf3" variableName="errorMessage"/>
      <set-variable value="The server is refusing to service the request because the entity of the request is in a format not supported by the requested resource for the requested method" doc:name="Set Error Description" doc:id="b2b0ca06-735c-4931-957a-ad108931f613" variableName="errorDescription"/>
      <flow-ref doc:name="global-prepare-error-response-sub-flow" doc:id="c38521fa-7243-496c-81de-28a016e0fc89" name="global-prepare-error-response-sub-flow"/>
      </on-error-propagate>
      <!--  DB Related issues  -->
      <!--  HTTP Requster Related error handling  -->
      <on-error-propagate type="HTTP:BAD_REQUEST" enableNotifications="true" logException="true" doc:name="On Error Propagate" doc:id="20a62f4c-aac0-4cd8-8de9-aac3b46e2fe0">
      <set-variable value="#[400]" doc:name="Set HTTP Status - 400" doc:id="159ecece-2f3d-47cb-992c-219beb0c7ed0" variableName="httpStatus"/>
      <set-payload value="#[error.muleMessage.payload]" doc:name="Set Payload" doc:id="6af37201-6407-4091-9f88-2a3f52144c45"/>
      </on-error-propagate>
      <on-error-propagate type="HTTP:FORBIDDEN" enableNotifications="true" logException="true" doc:name="On Error Propagate" doc:id="8b572e2b-ce7f-4226-a4ad-60395e9dff12">
      <set-variable value="#[403]" doc:name="Set HTTP Status - 403" doc:id="b384641e-9dbc-4719-8ee7-203537d4555e" variableName="httpStatus"/>
      <set-variable value="Access to the upstream service is forbidden." doc:name="Set Error Message" doc:id="1d21f86b-1169-45c4-aa8f-8ef1933046f8" variableName="errorMessage"/>
      <flow-ref doc:name="global-prepare-error-response-sub-flow" doc:id="2f14711f-7193-4cf6-b352-dd8a3733673d" name="global-prepare-error-response-sub-flow"/>
      </on-error-propagate>
      <on-error-propagate type="HTTP:CLIENT_SECURITY" enableNotifications="true" logException="true" doc:name="On Error Propagate" doc:id="2bbaa7cc-1bcb-43d1-9abf-2f8779a6196e">
      <set-variable value="#[401]" doc:name="Set HTTP Status - 401" doc:id="1ba828a4-67e2-462c-98e0-c69d7298f442" variableName="httpStatus"/>
      <set-payload value="#[error.muleMessage.payload]" doc:name="Set Payload" doc:id="00b99b85-c3be-4ca9-9b1c-8387418d7378"/>
      </on-error-propagate>
      <on-error-propagate type="HTTP:CONNECTIVITY" enableNotifications="true" logException="true" doc:name="On Error Propagate" doc:id="a632b2ae-b8bb-4711-8c24-cb94a4b02091">
      <set-variable value="#[503]" doc:name="Set HTTP Status - 503" doc:id="589d97c5-1dfb-46af-9207-9632e1e1f5ff" variableName="httpStatus"/>
      <set-variable value="Service unavailable" doc:name="Set Error Message" doc:id="bf79f24a-fdb4-4418-b0aa-e1cfe01323ee" variableName="errorMessage"/>
      <set-variable value="The (upstream) service is temporarily not available " doc:name="Set errorDescription" doc:id="012bd74a-78be-4d0f-938a-88d50d665992" variableName="errorDescription"/>
      <flow-ref doc:name="global-prepare-error-response-sub-flow" doc:id="f498eea1-ea09-46cc-80c5-7e05119d8009" name="global-prepare-error-response-sub-flow"/>
      </on-error-propagate>
      <on-error-propagate type="HTTP:INTERNAL_SERVER_ERROR" enableNotifications="true" logException="true" doc:name="On Error Propagate" doc:id="c83785d4-8d95-491b-b425-88c8777302c6">
      <set-variable value="#[500]" doc:name="Set HTTP Status - 500" doc:id="9fecab5c-7e63-40e4-8f22-d2e2634e792c" variableName="httpStatus"/>
      <logger level="INFO" doc:name="Logger" doc:id="5752a3ec-4c46-457c-a370-80e02450a7fc" message="kom ik hier"/>
      <set-variable value="Upstream service unable to fulfil request." doc:name="Set Error Message" doc:id="bf22ebec-a5f5-4e3a-bbb4-0355d46eb81d" variableName="errorMessage"/>
      <flow-ref doc:name="global-prepare-error-response-sub-flow" doc:id="08db75d1-ed88-440c-a2f1-034d89841a68" name="global-prepare-error-response-sub-flow"/>
      </on-error-propagate>
      <on-error-propagate type="HTTP:METHOD_NOT_ALLOWED" enableNotifications="true" logException="true" doc:name="On Error Propagate" doc:id="65a70755-86ca-4592-a08b-a0dbd4c6cb27">
      <set-variable value="#[405]" doc:name="Set HTTP Status - 405" doc:id="5c03da34-0cb6-4f38-941c-f38d90781810" variableName="httpStatus"/>
      <set-variable value="The method specified in the request is not allowed for this resource" doc:name="Set Error Message" doc:id="c16676d6-bc56-46ad-b07e-19241b7e39fb" variableName="errorMessage"/>
      <flow-ref doc:name="global-prepare-error-response-sub-flow" doc:id="1d8a2975-abf8-4f38-bd1f-c95d96f052db" name="global-prepare-error-response-sub-flow"/>
      </on-error-propagate>
      <on-error-propagate type="HTTP:NOT_ACCEPTABLE" enableNotifications="true" logException="true" doc:name="On Error Propagate" doc:id="b030e0c2-eb46-4971-b3d2-85d5aedac2a0">
      <set-variable value="#[406]" doc:name="Set HTTP Status - 406" doc:id="ff0bb5b4-d8a1-43cb-b164-c675c760104f" variableName="httpStatus"/>
      <flow-ref doc:name="global-prepare-error-response-sub-flow" doc:id="7cbb4604-1521-455b-8ce3-a1157a8f1605" name="global-prepare-error-response-sub-flow"/>
      </on-error-propagate>
      <on-error-propagate type="HTTP:NOT_FOUND" enableNotifications="true" logException="true" doc:name="On Error Propagate" doc:id="b05536b9-a15e-4767-ad72-987f50d81abc">
      <set-variable value="#[404]" doc:name="Set HTTP Status - 404" doc:id="8ee6696e-2d18-49e7-a9b1-0f0c784c7a6e" variableName="httpStatus"/>
      <set-variable value="The server has not found anything matching the Request-URI" doc:name="Set Error Message" doc:id="2fcbf13c-8c96-42a7-8068-e94e7b1681c4" variableName="errorMessage"/>
      <flow-ref doc:name="global-prepare-error-response-sub-flow" doc:id="f5396841-e8fa-4012-b3da-33c6cd9b0786" name="global-prepare-error-response-sub-flow"/>
      </on-error-propagate>
      <on-error-propagate type="HTTP:PARSING" enableNotifications="true" logException="true" doc:name="On Error Propagate" doc:id="f44feae6-4ac5-40fd-b8a7-bf716cdaeb74">
      <set-variable value="#[400]" doc:name="Set HTTP Status - 400" doc:id="d1b01c4c-9d76-4ffe-bd5a-af864a9f4237" variableName="httpStatus"/>
      <flow-ref doc:name="global-prepare-error-response-sub-flow" doc:id="c8fbbb9f-882d-476d-bf53-ab4489b89db0" name="global-prepare-error-response-sub-flow"/>
      </on-error-propagate>
      <on-error-propagate type="HTTP:RETRY_EXHAUSTED" enableNotifications="true" logException="true" doc:name="On Error Propagate" doc:id="f0735fb8-5577-4fe1-9f9c-e83f8e3d3f5a">
      <set-variable value="#[503]" doc:name="Set HTTP Status - 503" doc:id="2c51c6c9-73d0-4298-a1c6-a7e27d1b09c2" variableName="httpStatus"/>
      <flow-ref doc:name="global-prepare-error-response-sub-flow" doc:id="fd95ae29-be93-4222-bd7f-e5cff9fac8fb" name="global-prepare-error-response-sub-flow"/>
      </on-error-propagate>
      <on-error-propagate type="HTTP:SECURITY" enableNotifications="true" logException="true" doc:name="On Error Propagate" doc:id="c4485094-070c-49b1-a397-703931d3dadf">
      <set-variable value="#[401]" doc:name="Set HTTP Status - 401" doc:id="e24240a4-34ba-4b5f-abcf-732b699a4285" variableName="httpStatus"/>
      <flow-ref doc:name="global-prepare-error-response-sub-flow" doc:id="5dddc4f4-7833-404e-9ee0-b83fd060bec3" name="global-prepare-error-response-sub-flow"/>
      </on-error-propagate>
      <on-error-propagate type="HTTP:TIMEOUT" enableNotifications="true" logException="true" doc:name="On Error Propagate" doc:id="8a1b7d8c-4662-4f9e-ae2b-d525f031add9">
      <set-variable value="#[504]" doc:name="Set HTTP Status - 504" doc:id="d6f738ed-09a0-49a7-9b0a-4c45b49fe06f" variableName="httpStatus"/>
      <flow-ref doc:name="global-prepare-error-response-sub-flow" doc:id="cc3b61ea-be8a-4a1d-92db-44beb4aa0ff4" name="global-prepare-error-response-sub-flow"/>
      </on-error-propagate>
      <on-error-propagate type="HTTP:TOO_MANY_REQUESTS" enableNotifications="true" logException="true" doc:name="On Error Propagate" doc:id="a8362c81-5d52-443d-a3ec-e4dbef726846">
      <set-variable value="#[429]" doc:name="Set HTTP Status - 429" doc:id="bee953a0-cc4b-466b-b46e-eea1cd69126c" variableName="httpStatus"/>
      <flow-ref doc:name="global-prepare-error-response-sub-flow" doc:id="c75dc3f0-a6ef-406f-aa48-4d91da77d65b" name="global-prepare-error-response-sub-flow"/>
      </on-error-propagate>
      <on-error-propagate type="HTTP:UNAUTHORIZED" enableNotifications="true" logException="true" doc:name="On Error Propagate" doc:id="32ac5482-5658-4cc5-90e0-12a6d0f8b538">
      <set-variable value="#[403]" doc:name="Set HTTP Status - 403" doc:id="a02ffebb-f103-4cf4-af38-576802d6b9e5" variableName="httpStatus"/>
      <flow-ref doc:name="global-prepare-error-response-sub-flow" doc:id="032dff29-e97d-4dc9-98ef-84ce3731c26f" name="global-prepare-error-response-sub-flow"/>
      </on-error-propagate>
      <on-error-propagate type="HTTP:UNSUPPORTED_MEDIA_TYPE" enableNotifications="true" logException="true" doc:name="On Error Propagate" doc:id="15e7ac10-fc0c-4fb4-ac51-e2313818ad99">
      <set-variable value="#[415]" doc:name="Set HTTP Status - 415" doc:id="f55e7c18-10d4-49dd-8604-3d4d7a795e6a" variableName="httpStatus"/>
      <flow-ref doc:name="global-prepare-error-response-sub-flow" doc:id="c429e1e0-1393-45f3-8d72-231623ee4d2b" name="global-prepare-error-response-sub-flow"/>
      </on-error-propagate>
      <!--  Streaming related exception  -->
      <on-error-propagate type="STREAM_MAXIMUM_SIZE_EXCEEDED" enableNotifications="true" logException="true" doc:name="On Error Continue" doc:id="3f3b0277-d20d-47d5-8d6f-a09f3967dd07">
      <set-variable value="#[500]" doc:name="Set HTTP Status - 500" doc:id="e1c47fbf-35b9-4caf-86f6-197385ad9afa" variableName="httpStatus"/>
      <flow-ref doc:name="global-prepare-error-response-sub-flow" doc:id="a34d2aef-93d8-4b72-9be9-5497b4c27092" name="global-prepare-error-response-sub-flow"/>
      </on-error-propagate>
      <!--  Generic CONNECTIVITY Related Exception handling start. Order matters  -->
      <on-error-propagate type="RETRY_EXHAUSTED" enableNotifications="true" logException="true" doc:name="On Error Continue" doc:id="8d8dd45e-bbec-41fe-9c74-34d0d6dc2f46">
      <set-variable value="#[503]" doc:name="Set HTTP Status - 503" doc:id="9ae9d4fa-5b33-4666-ae12-87a26da06078" variableName="httpStatus"/>
      <flow-ref doc:name="global-prepare-error-response-sub-flow" doc:id="58373f24-9a39-47dc-82b4-61420b55ccab" name="global-prepare-error-response-sub-flow"/>
      </on-error-propagate>
      <on-error-propagate type="REDELIVERY_EXHAUSTED" enableNotifications="true" logException="true" doc:name="On Error Continue" doc:id="4e592ce9-5b77-4bab-bd59-f72201eff9be">
      <set-variable value="#[503]" doc:name="Set HTTP Status - 503" doc:id="72723718-14f6-4c0c-bf9e-b78d8f309dbc" variableName="httpStatus"/>
      <flow-ref doc:name="global-prepare-error-response-sub-flow" doc:id="24e4b4aa-0707-4d07-a1ce-da2b34cb650b" name="global-prepare-error-response-sub-flow"/>
      </on-error-propagate>
      <on-error-propagate type="CONNECTIVITY" enableNotifications="true" logException="true" doc:name="On Error Continue" doc:id="69011dd7-61df-400d-bea9-996c16b7b13f">
      <set-variable value="#[503]" doc:name="Set HTTP Status - 503" doc:id="b1fb71c1-2b82-4895-99a6-345aa296b438" variableName="httpStatus"/>
      <set-variable value="Service unavailable" doc:name="Set vErrorMessage" doc:id="cb0edef4-22c1-4a67-870d-4f5adbf4c8f7" variableName="errorMessage"/>
      <set-variable value="The (upstream) service is temporarily not available " doc:name="Set vErrorDescription" doc:id="4f40f822-fb59-4fb8-992c-d301b60a7c69" variableName="errorDescription"/>
      <flow-ref doc:name="global-prepare-error-response-sub-flow" doc:id="b8881611-b3d8-4c70-b1d0-a1a00a34fd83" name="global-prepare-error-response-sub-flow"/>
      </on-error-propagate>
      <on-error-propagate type="TIMEOUT" enableNotifications="true" logException="true" doc:name="On Error Continue" doc:id="7be38373-eadd-4402-b75f-35995651f5a1">
      <set-variable value="#[504]" doc:name="Set HTTP Status - 504" doc:id="2743ef34-7a1b-4b6e-8903-0debf1b7e1d0" variableName="httpStatus"/>
      <flow-ref doc:name="global-prepare-error-response-sub-flow" doc:id="eef8760e-b921-4bc2-92c3-d904b4b9a9ad" name="global-prepare-error-response-sub-flow"/>
      </on-error-propagate>
      <!--  Generic CONNECTIVITY Exception handling end  -->
      <on-error-propagate type="TRANSFORMATION" enableNotifications="true" logException="true" doc:name="On Error Continue" doc:id="e41f63e9-acb9-4809-ae14-8c9877445c20">
      <set-variable value="#[400]" doc:name="Set HTTP Status - 400" doc:id="5ee07bcf-f33a-4854-9933-7b7a6a2d2679" variableName="httpStatus"/>
      <flow-ref doc:name="global-prepare-error-response-sub-flow" doc:id="293d3eda-9058-4ec3-bc10-e815fabd7908" name="global-prepare-error-response-sub-flow"/>
      </on-error-propagate>
      <on-error-propagate type="EXPRESSION" enableNotifications="true" logException="true" doc:name="On Error Continue" doc:id="ca605652-be5e-4e23-93d6-db210d805390">
      <set-variable value="#[500]" doc:name="Set HTTP Status - 500" doc:id="b8507ebb-1cde-4d56-be4e-f73a422ffe20" variableName="httpStatus"/>
      <flow-ref doc:name="global-prepare-error-response-sub-flow" doc:id="7772693a-7da6-4109-b0e2-aa4ae0b762eb" name="global-prepare-error-response-sub-flow"/>
      </on-error-propagate>
      <on-error-propagate type="ROUTING" enableNotifications="true" logException="true" doc:name="On Error Continue" doc:id="4fd8be22-522c-4f68-bcef-306f6435d2f0">
      <set-variable value="#[400]" doc:name="Set HTTP Status - 400" doc:id="efcd5c89-bc45-452b-ae9e-33429f5ef55a" variableName="httpStatus"/>
      <flow-ref doc:name="global-prepare-error-response-sub-flow" doc:id="61d1c95f-d953-40e2-9654-9c556b97ecb7" name="global-prepare-error-response-sub-flow"/>
      </on-error-propagate>
      <on-error-propagate type="SECURITY" enableNotifications="true" logException="true" doc:name="On Error Continue" doc:id="56b5b92e-8891-47dd-9c9d-57046d480c21">
      <set-variable value="#[401]" doc:name="Set HTTP Status - 401" doc:id="f7bc0ea4-c502-4df1-af51-3e6e9ec7fb3a" variableName="httpStatus"/>
      <flow-ref doc:name="global-prepare-error-response-sub-flow" doc:id="e0129274-370a-4515-81df-b99c3f7116f1" name="global-prepare-error-response-sub-flow"/>
      </on-error-propagate>
      <!--  If none of the above matches then handle a the exception using generic handler  -->
      <on-error-propagate type="ANY" enableNotifications="true" logException="true" doc:name="On Error Continue" doc:id="4aa15648-61ab-4f57-84e7-2a917c1c0155">
      <set-variable value="#[500]" doc:name="Set HTTP Status - 500" variableName="httpStatus"/>
      <set-variable value="Internal server error" doc:name="Set Error Message" doc:id="c787b78c-3afd-4e4d-8ae2-41e8127843b4" variableName="errorMessage"/>
      <set-variable value="The server encountered an unexpected condition which prevented it from fulfilling the request" doc:name="errorDescription" doc:id="a371c0b5-912b-4461-af7c-d20f531c447d" variableName="errorDescription"/>
      <flow-ref doc:name="global-prepare-error-response-sub-flow" doc:id="e155d644-3845-4c66-9b34-a73023916fec" name="global-prepare-error-response-sub-flow"/>
      </on-error-propagate>
      </error-handler>
      <sub-flow name="global-prepare-error-response-sub-flow" doc:id="4d88dcc6-e89d-4c57-88b3-b5d46d7f3500">
      <ee:transform doc:name="Init Variables" doc:id="1094c585-818e-4238-972b-5e6d72d3cb60">
      <ee:message> </ee:message>
      <ee:variables>
      <ee:set-variable variableName="errorRaised">
      <![CDATA[ %dw 2.0 output application/java --- true ]]>
      </ee:set-variable>
      <ee:set-variable variableName="errorDescription">
      <![CDATA[ %dw 2.0 output application/java --- if(vars.errorDescription?) vars.errorDescription else error.exception.detailMessage ]]>
      </ee:set-variable>
      <ee:set-variable variableName="logCategory">
      <![CDATA[ %dw 2.0 output application/java --- 'Exception' ]]>
      </ee:set-variable>
      <ee:set-variable variableName="logLevel">
      <![CDATA[ %dw 2.0 output application/java --- 'ERROR' ]]>
      </ee:set-variable>
      </ee:variables>
      </ee:transform>
      <ee:transform doc:name="Error Response" doc:id="1e9b9ba3-0d9f-42aa-b0e4-fe1e5a0351c1">
      <ee:message>
      <ee:set-payload>
      <![CDATA[ %dw 2.0 output application/json encoding="UTF-8", skipNullOn="everywhere" var errors = (((error.description default "" replace "Error validating JSON. Error: - " with "") replace "- " with "") splitBy "\n") --- { code : vars.httpStatus, message : if(vars.errorMessage != null) vars.errorMessage else (error.errorType.identifier), description: if(vars.errorDescription != null) vars.errorDescription else error.description, dateTime : now() as String { format: "yyyy-MM-dd'T'HH:mm:ss'Z'" }, transactionId : vars.transactionId } ]]>
      </ee:set-payload>
      </ee:message>
      </ee:transform>
      <logger level="INFO" doc:name="Error Log" doc:id="ee9dae01-a2f0-4fd1-a01a-8f3c4fec5a4a" message="Transaction [#[vars.transactionId]] - Error Code [#[vars.httpStatus]] - Error Message [#[error.errorType.identifier default '']] - Error Description [#[error.description default '']]"/>
      </sub-flow>
      </mule>

    - #### aws-sapi.xml
      <details>
      <summary></summary>
        
      ```xml
      <mule xmlns:tracking="http://www.mulesoft.org/schema/mule/ee/tracking" xmlns:doc="http://www.mulesoft.org/schema/mule/documentation" xmlns="http://www.mulesoft.org/schema/mule/core" xmlns:apikit="http://www.mulesoft.org/schema/mule/mule-apikit" xmlns:http="http://www.mulesoft.org/schema/mule/http" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.mulesoft.org/schema/mule/core http://www.mulesoft.org/schema/mule/core/current/mule.xsd http://www.mulesoft.org/schema/mule/http http://www.mulesoft.org/schema/mule/http/current/mule-http.xsd http://www.mulesoft.org/schema/mule/mule-apikit http://www.mulesoft.org/schema/mule/mule-apikit/current/mule-apikit.xsd http://www.mulesoft.org/schema/mule/ee/tracking http://www.mulesoft.org/schema/mule/ee/tracking/current/mule-tracking-ee.xsd">
          <flow name="aws-sapi-main">
              <http:listener path="/v1/*" config-ref="HTTP_Listener_config">
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
              <set-variable value="#[attributes.headers.'x-transaction-id' default uuid()]" doc:name="Set transactionId" doc:id="c4485bd8-89d4-46f4-a99e-559e712334dc" variableName="transactionId"/>
              <apikit:router config-ref="aws-sapi-config"/>
              <error-handler ref="global-error-handler"/>
          </flow>
          <flow name="aws-sapi-console">
              <http:listener path="/console/*" config-ref="HTTP_Listener_config">
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
              <apikit:console config-ref="aws-sapi-config"/>
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
        
      amazon:
        access_key: "![1tEzW95n06UcBa9iC6tA5v79PJfFGfSlVxYj1enF8JA=]"
        secret_key: "![RWxlryu7/0pQxtecd3R121KitzxXty7qxNMwW4Oc+dUh9r0ZusPORojP3+q1Otml]"
        region: "ap-south-1"

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
    <groupId>com.uho</groupId>
    <artifactId>aws-sapi</artifactId>
    <version>1.0.0</version>
    <packaging>mule-application</packaging>
    <name>aws-sapi</name>
    <description>aws sapi</description>
    <properties>
      <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
      <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
      <app.runtime>4.3.0-20210622</app.runtime>
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
            <classifier>mule-application</classifier>
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
        <groupId>com.mulesoft.modules</groupId>
        <artifactId>mule-secure-configuration-property-module</artifactId>
        <version>1.2.3</version>
        <classifier>mule-plugin</classifier>
      </dependency>
      <dependency>
        <groupId>org.mule.modules</groupId>
        <artifactId>mule-apikit-module</artifactId>
        <version>1.5.1</version>
        <classifier>mule-plugin</classifier>
      </dependency>
      <dependency>
        <groupId>com.mulesoft.connectors</groupId>
        <artifactId>mule-amazon-s3-connector</artifactId>
        <version>5.7.0</version>
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
  
