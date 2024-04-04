# <p align="center">Raml Code</p>
# mss-covid-papi
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

  - ### covid-case-request.json
    <details>
    <summary></summary>
      
    ```json
    {
       "source":"Hospital1",
       "caseType":"positive",
       "firstName":"John",
       "lastName":"Nixon",
       "phone":"541-754-3010",
       "email":"john@gmail.com",
       "dateOfBirth":"1989-04-26",
       "nationalID":"209-49-6193",
       "nationalIDType":"SSN",
       "address":{
          "streetAddress":"1600 Pennsylvania Avenue NW",
          "city":"Washington",
          "state":"DC",
          "postal":"20500",
          "country":"USA"
       },
       "iDoc":"iVBORw0KGgoAAAANSUhEUgAAAOIAAACRCAYAAADepmS4AAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8"
    }

  - ### covid-case-response.json
    <details>
    <summary></summary>
      
    ```json
    {
        "registrationID": "3673673"
    }

- ## covid-papi-types
  - ### covid-papi-types.raml
    <details>
    <summary></summary>

    ```raml
    #%RAML 1.0 Library
    usage: Types for uho sapi
    types:
      covid-case-request-type:
        type: object
        description: This type used to register and update covid case
        properties:
          source: 
            type: string
            required: true
            minLength: 1
            maxLength: 60
            example: "The johns hopkins hospital"
          caseType: 
            type: string
            required: true
            minLength: 1
            maxLength: 20
            example: "positive"
          firstName: 
            type: string
            required: true
            minLength: 1
            maxLength: 20
            example: "John"
          lastName: 
            type: string
            required: true
            minLength: 1
            maxLength: 20
            example: "Nixon"
          phone: 
            type: string
            required: true
            minLength: 1
            maxLength: 12
          email: 
            type: string
            required: false
            minLength: 1
            maxLength: 50
            example: "john@gmail.com"
          dateOfBirth:
            type: date-only
            required: true
            example: "1989-04-26"
          nationalID: 
            type: string
            required: true
            minLength: 1
            maxLength: 20
            example: "209-49-6193"
          nationalIDType:
            type: string
            required: true
            minLength: 1
            maxLength: 20
            example: "SSN"
          address:
            type: object
            properties:
              streetAddress: 
                type: string
                required: true
                minLength: 1
                maxLength: 30
                example: "1600 Pennsylvania Avenue NW"
              city: 
                type: string
                required: true
                minLength: 1
                maxLength: 30
                example: "Washington"
              state: 
                type: string
                required: true
                minLength: 1
                maxLength: 30
                example: "DC"
              postal: 
                type: string
                required: true
                minLength: 1
                maxLength: 30
                example: "20500"
              country: 
                type: string
                required: true
                minLength: 1
                maxLength: 30
                example: "USA"
          iDoc: 
            type: string
            required: true
      covid-case-response-type:
        type: object
        properties:
          registrationID: 
            type: string      
            example: "38774"

- ## covid-papi.raml
  <details>
  <summary></summary>

  ```raml
  #%RAML 1.0
  title: covid-papi
  version: v1
  baseUri: http://{environment}/covid/{version}/
  baseUriParameters:
    environment:
      description: DEV, TEST, UAT, PROD
      enum: ["uho-dev-covid-papi.us-e2.cloudhub.io","uho-test-covid-papi.us-e2.cloudhub.io", "uho-uat-covid-papi.us-e2.cloudhub.io", "uho-prod-covid-papi.us-e2.cloudhub.io"]
  traits:
    client-id-header: !include exchange_modules/d896f4a8-62ed-4bab-9db4-5db7ac45ebd4/client-id-required/1.0.0/client-id-required.raml
    transaction-header: !include exchange_modules/d896f4a8-62ed-4bab-9db4-5db7ac45ebd4/transaction-header/1.0.0/transaction-header.raml
    correlation-id-header: !include exchange_modules/d896f4a8-62ed-4bab-9db4-5db7ac45ebd4/correlation-id-header/1.0.0/correlation-id-header.raml
  uses:
    resource-types: exchange_modules/d896f4a8-62ed-4bab-9db4-5db7ac45ebd4/resource-types/1.0.0/resource-types.raml
    common-data-types: exchange_modules/d896f4a8-62ed-4bab-9db4-5db7ac45ebd4/common-data-types/1.0.0/common-data-types.raml
    covid-papi-types: covid-papi-types/covid-papi-types.raml
  /cases:
    post:
      description: To register covid case
      is: [client-id-header,correlation-id-header,transaction-header]
      body:
        application/json:
          type: covid-papi-types.covid-case-request-type
          example: !include examples/covid-case-request.json
      responses:
        201:
          body:
            application/json:
              type: covid-papi-types.covid-case-response-type
              example: !include examples/covid-case-response.json
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
    /{nationalID}:
        get:
          description: To get covid case by national id.
          is: [client-id-header,correlation-id-header,transaction-header]
          responses:
            200:
              body:
                application/json:
                  type: covid-papi-types.covid-case-response-type
                  example: !include examples/covid-case-response.json
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
  /cases-qa:
    post:
      description: To register covid case
      is: [client-id-header,correlation-id-header,transaction-header]
      body:
        application/json:
          type: covid-papi-types.covid-case-request-type
          example: !include examples/covid-case-request.json
      responses:
        201:
          body:
            application/json:
              type: covid-papi-types.covid-case-response-type
              example: !include examples/covid-case-response.json
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


# <p align="center">Implementation of mss-covid-papi</p>  
# mss-covid-papi
- ## src\main
  - ### mule
    - #### implementations
      - ##### get-covid-case.xml
        <details>
        <summary></summary>
        
        ```xml
        <mule xmlns:ee="http://www.mulesoft.org/schema/mule/ee/core" xmlns="http://www.mulesoft.org/schema/mule/core" xmlns:doc="http://www.mulesoft.org/schema/mule/documentation" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.mulesoft.org/schema/mule/core http://www.mulesoft.org/schema/mule/core/current/mule.xsd http://www.mulesoft.org/schema/mule/ee/core http://www.mulesoft.org/schema/mule/ee/core/current/mule-ee.xsd">
            <flow name="get:\cases\(nationalID):covid-papi-config" doc:id="c7e78ed0-b1c0-4235-9d8a-aaf3cf46af13">
                <set-variable value="#[attributes.headers.'x-correlation-id' default ""]" doc:name="Set correlationId" doc:id="d7b458f7-2fc0-4c59-8571-55df3a40e569" variableName="correlationId"/>
                <logger level="INFO" doc:name="Start Log" doc:id="47fbb5b5-4e1b-46e7-8a1b-bd51be2cebe8" message="transactionID: #[vars.transactionId]], correlationID: #[vars.correlationID], message: Started register covid case flow, payload: #[payload]"/>
                <ee:transform doc:name="Create Variables" doc:id="373d15e7-2149-44f3-a8b0-c32beef0c947">
                    <ee:message/>
                    <ee:variables>
                        <ee:set-variable variableName="uhubHeaderParameters">
        <![CDATA[ %dw 2.0 output application/java --- { "client_id": Mule::p('ehub.sapi.client_id'), "client_secret": Mule::p('ehub.sapi.client_secret'), "x-transaction-id": vars.transactionId, "x-correlation-id": vars.correlationId } ]]>
                        </ee:set-variable>
                        <ee:set-variable variableName="covidCasePayload">
        <![CDATA[ { nationalID: attributes.uriParams.nationalID } ]]>
                        </ee:set-variable>
                    </ee:variables>
                </ee:transform>
                <flow-ref doc:name="check-case-exist" doc:id="fbade97d-d1ee-4882-b7db-3de3f1fa6519" name="check-case-exist"/>
                <ee:transform doc:name="Set Response" doc:id="a7806b8b-5912-4ef6-86c3-5f3cc2cd0820">
                    <ee:message>
                        <ee:set-payload>
        <![CDATA[ %dw 2.0 output application/json --- if(payload.code != 404) payload else { code : 200, message : "Case registration is in progress", description: "Case registration is in progress, please reach out to support team in case the registration is taking more then 12 hours", dateTime : now() as String { format: "yyyy-MM-dd'T'HH:mm:ss'Z'" }, transactionId : vars.transactionId } ]]>
                        </ee:set-payload>
                    </ee:message>
                </ee:transform>
            </flow>
        </mule>

      - ##### health-check.xml
        <details>
        <summary></summary>
        
        ```xml
        <mule xmlns:custom-logger="http://www.mulesoft.org/schema/mule/custom-logger" xmlns:ee="http://www.mulesoft.org/schema/mule/ee/core" xmlns="http://www.mulesoft.org/schema/mule/core" xmlns:doc="http://www.mulesoft.org/schema/mule/documentation" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.mulesoft.org/schema/mule/core http://www.mulesoft.org/schema/mule/core/current/mule.xsd http://www.mulesoft.org/schema/mule/ee/core http://www.mulesoft.org/schema/mule/ee/core/current/mule-ee.xsd http://www.mulesoft.org/schema/mule/custom-logger http://www.mulesoft.org/schema/mule/custom-logger/current/mule-custom-logger.xsd">
            <flow name="get:\health-check:covid-papi-config">
                <ee:transform doc:name="Set Response" doc:id="3996a32d-7737-48b6-83fc-8e7ff8ac2071">
                    <ee:message>
                        <ee:set-payload>
        <![CDATA[ %dw 2.0 output application/json --- { message: app.name ++ " is alive and kicking" } ]]>
                        </ee:set-payload>
                    </ee:message>
                </ee:transform>
            </flow>
        </mule>

      - ##### register-covid-case.xml
        <details>
        <summary></summary>

        ```xml
        <mule xmlns:http="http://www.mulesoft.org/schema/mule/http" xmlns:ee="http://www.mulesoft.org/schema/mule/ee/core" xmlns="http://www.mulesoft.org/schema/mule/core" xmlns:doc="http://www.mulesoft.org/schema/mule/documentation" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.mulesoft.org/schema/mule/core http://www.mulesoft.org/schema/mule/core/current/mule.xsd http://www.mulesoft.org/schema/mule/ee/core http://www.mulesoft.org/schema/mule/ee/core/current/mule-ee.xsd http://www.mulesoft.org/schema/mule/http http://www.mulesoft.org/schema/mule/http/current/mule-http.xsd">
            <flow name="post:\cases:application\json:covid-papi-config">
                <set-variable value="#[attributes.headers.'x-correlation-id' default ""]" doc:name="Set correlationId" doc:id="af0fa660-a42d-4aa4-902a-cba31699779e" variableName="correlationId"/>
                <logger level="INFO" doc:name="Start Log" doc:id="516856d5-9900-4abf-a8fa-d79f999f0bb9" message="transactionID: #[vars.transactionId]], correlationID: #[vars.correlationID], message: Started register covid case flow, payload: #[payload]"/>
                <ee:transform doc:name="Create Variables" doc:id="2dfb85bd-d3fc-46c2-bdfe-3d8bff0820e4">
                    <ee:message> </ee:message>
                    <ee:variables>
                        <ee:set-variable variableName="covidCasePayload">
        <![CDATA[ %dw 2.0 output application/json --- { source: payload.source, caseType: payload.caseType, firstName: payload.firstName, lastName: payload.lastName, phone: payload.phone, email: payload.email, dateOfBirth: payload.dateOfBirth, nationalID: payload.nationalID, nationalIDType: payload.nationalIDType, address: { streetAddress: payload.address.streetAddress, city: payload.address.city, state: payload.address.state, postal: payload.address.postal, country: payload.address.country } } ]]>
                        </ee:set-variable>
                        <ee:set-variable variableName="identityDocument">
        <![CDATA[ payload.iDoc ]]>
                        </ee:set-variable>
                        <ee:set-variable variableName="uhubHeaderParameters">
        <![CDATA[ %dw 2.0 output application/java --- { "client_id": Mule::p('ehub.sapi.client_id'), "client_secret": Mule::p('ehub.sapi.client_secret'), "x-transaction-id": vars.transactionId, "x-correlation-id": vars.correlationId } ]]>
                        </ee:set-variable>
                    </ee:variables>
                </ee:transform>
                <flow-ref doc:name="check-case-exist" doc:id="f942eb7d-0c04-43d6-b05a-2b16d8432686" name="check-case-exist"/>
                <flow-ref doc:name="upsert-upload-covid-case" doc:id="afa17e1f-d8db-4d23-b109-5f5ec726cee1" name="upsert-upload-covid-case"/>
                <ee:transform doc:name="Report WHO Payload" doc:id="2a6790e0-9bd7-48cf-9b36-bab8c6e4443f">
                    <ee:message>
                        <ee:set-payload>
        <![CDATA[ %dw 2.0 output application/json --- { "caseType": vars.covidCasePayload.caseType, "firstName": vars.covidCasePayload.firstName, "lastName": vars.covidCasePayload.lastName, "phone": vars.covidCasePayload.phone, "email": vars.covidCasePayload.email, "dateOfBirth": vars.covidCasePayload.dateOfBirth, "country": vars.covidCasePayload.address.country } ]]>
                        </ee:set-payload>
                    </ee:message>
                    <ee:variables>
                        <ee:set-variable variableName="whoHeaderParameters">
        <![CDATA[ %dw 2.0 output application/java --- { "client_id": Mule::p('who.sapi.client_id'), "client_secret": Mule::p('who.sapi.client_secret'), "x-transaction-id": vars.transactionId, "x-correlation-id": vars.correlationId } ]]>
                        </ee:set-variable>
                    </ee:variables>
                </ee:transform>
                <flow-ref doc:name="report-case-to-who-request" doc:id="f97ead04-c594-4c9c-b276-b19c6653cfc1" name="report-case-to-who-request"/>
                <ee:transform xmlns:ee="http://www.mulesoft.org/schema/mule/ee/core" xsi:schemaLocation="http://www.mulesoft.org/schema/mule/ee/core http://www.mulesoft.org/schema/mule/ee/core/current/mule-ee.xsd" doc:name="Set Response">
                    <ee:message>
                        <ee:set-payload>
        <![CDATA[ %dw 2.0 output application/json --- { registrationID: vars.registrationId } ]]>
                        </ee:set-payload>
                    </ee:message>
                </ee:transform>
            </flow>
            <sub-flow name="check-case-exist" doc:id="1fa566f3-c3eb-46dd-9d92-42ca39ddbf81">
                <flow-ref doc:name="get-covid-case-by-national-id" doc:id="5ebebe9d-5668-47c2-959d-64ab2ed6def6" name="get-covid-case-by-national-id"/>
                <ee:transform doc:name="Set uhubResponse Variables" doc:id="eb58d01c-aaae-4e5d-9401-f48704dffaf9">
                    <ee:message> </ee:message>
                    <ee:variables>
                        <ee:set-variable variableName="uhubResponse">
        <![CDATA[ %dw 2.0 output application/java var recentCase = if ( payload.code != 404 ) (payload orderBy (item) -> item.updateDate)[0] else {} var lastVaccinationDate = if ( payload.code != 404 ) (payload filter (item) -> item.caseType == "vaccinated").createDate[0] else {} --- if ( payload.code != 404 ) { caseId: recentCase.caseID, recentCaseType: recentCase.caseType, vaccinatedFlag: if ( vars.covidCasePayload.caseType == "vaccinated" and lastVaccinationDate != null ) (daysBetween(lastVaccinationDate, now()) > 30) else (false) } else "" ]]>
                        </ee:set-variable>
                    </ee:variables>
                </ee:transform>
            </sub-flow>
            <sub-flow name="upsert-upload-covid-case" doc:id="8f6992d9-0be1-4c61-9a61-589471977628">
                <choice doc:name="Case Exist ?" doc:id="370c7bb4-355d-4b9b-b9c1-4206e450920b">
                    <when expression="#[(payload.code == 404) or (vars.recentCaseType == &quot;invalid&quot; or vars.recentCaseType == &quot;recovered&quot; or vars.vaccinatedFlag == false)]">
                        <flow-ref doc:name="post-covid-case-request" doc:id="c71de8b9-8be2-4536-b597-a1a367a8defd" name="post-covid-case-service-request"/>
                        <set-variable value="#[payload.caseID]" doc:name="Set registrationId" doc:id="2208de93-2445-4213-9e61-ea52da7896b0" variableName="registrationId"/>
                        <ee:transform doc:name="Prepare Upload Document Payload" doc:id="91e84cdd-2ded-4ace-be2b-95fada530296">
                            <ee:message>
                                <ee:set-payload>
        <![CDATA[ %dw 2.0 output application/json --- { bucketName: Mule::p('secure::aws.s3.bucket.name'), folderPath: Mule::p('secure::aws.s3.folder.path'), fileName: vars.registrationId as String ++ &quot;.png&quot;, document: vars.identityDocument } ]]>
                                </ee:set-payload>
                            </ee:message>
                            <ee:variables>
                                <ee:set-variable variableName="awsHeaderParameters">
        <![CDATA[ %dw 2.0 output application/java --- { &quot;client_id&quot;: Mule::p('aws.sapi.client_id'), &quot;client_secret&quot;: Mule::p('aws.sapi.client_secret'), &quot;x-transaction-id&quot;: vars.transactionId, &quot;x-correlation-id&quot;: vars.correlationId } ]]>
                                </ee:set-variable>
                            </ee:variables>
                        </ee:transform>
                        <flow-ref doc:name="post-upload-document-service-request" doc:id="ecda1d4b-15f6-4f51-a9fe-324072663507" name="post-upload-document-service-request"/>
                    </when>
                    <otherwise>
                        <ee:transform doc:name="Update Covid Case Payload" doc:id="06fcec34-0384-4362-84b0-3b1cdfa1e61e">
                            <ee:message>
                                <ee:set-payload>
        <![CDATA[ %dw 2.0 output application/json --- { caseID: vars.uhubResponse.caseId } ++ vars.covidCasePayload as Object ]]>
                                </ee:set-payload>
                            </ee:message>
                        </ee:transform>
                        <flow-ref doc:name="update-covid-case-service-request" doc:id="016b1184-9f73-4f9f-879d-3a8e1f73f509" name="update-covid-case-service-request"/>
                        <set-variable value="#[payload.caseID]" doc:name="Set registrationId" doc:id="361d5b58-f154-4085-9f61-2f939aba515f" variableName="registrationId"/>
                    </otherwise>
                </choice>
            </sub-flow>
        </mule>

    - #### subflows
      - ##### uhub-sapi-rest-calls.xml
        <details>
        <summary></summary>
        
        ```xml
        <mule xmlns:http="http://www.mulesoft.org/schema/mule/http" xmlns="http://www.mulesoft.org/schema/mule/core" xmlns:doc="http://www.mulesoft.org/schema/mule/documentation" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.mulesoft.org/schema/mule/core http://www.mulesoft.org/schema/mule/core/current/mule.xsd http://www.mulesoft.org/schema/mule/http http://www.mulesoft.org/schema/mule/http/current/mule-http.xsd">
            <sub-flow name="report-case-to-who-request" doc:id="68e471bc-f6e9-4ba3-be31-c2f0ae8cee6e">
                <logger level="DEBUG" doc:name="Before Backend Call" doc:id="1534e02d-8af9-4517-94e8-00ea1225981a" message="transactionID: #[vars.transactionId]], correlationID: #[vars.correlationID], message: &quot;Before report case to who service call&quot;, payload: #[payload]"/>
                <http:request method="POST" doc:name="Call Report Case To WHO Service" doc:id="a9093685-f82c-48dc-b512-2aac44409861" config-ref="HTTP_WHO_Request_configuration" path="/${secure::who.sapi.subpath}">
                    <http:headers>
                        <![CDATA[ #[vars.whoHeaderParameters] ]]>
                    </http:headers>
                </http:request>
                <logger level="DEBUG" doc:name="After Backend Call" doc:id="7273a341-a0b7-4f84-94c9-03dbc46b7397" message="transactionID: #[vars.transactionId]], correlationID: #[vars.correlationID], message: &quot;After update case uhub sapi service call&quot;, payload: #[payload]"/>
            </sub-flow>
            <sub-flow name="update-covid-case-service-request" doc:id="61f5f2a1-14dc-418c-bf08-5ace1cb7449f">
                <logger level="DEBUG" doc:name="Before Backend Call" doc:id="40b4149e-4277-40fc-94a3-f5e3ceb8be8b" message="transactionID: #[vars.transactionId]], correlationID: #[vars.correlationID], message: &quot;Before update case uhub sapi service call&quot;, payload: #[payload]"/>
                <http:request method="PUT" doc:name="Update Covid Case " doc:id="8ef2d136-a47d-4a38-a50f-9ec919276104" config-ref="HTTP_UHUB_SAPI_Request_configuration" path="/${secure::ehub.sapi.subpath.update.case}">
                    <http:headers>
                        <![CDATA[ #[vars.uhubHeaderParameters] ]]>
                    </http:headers>
                </http:request>
                <logger level="DEBUG" doc:name="After Backend Call" doc:id="94e7242c-fd40-4dfb-971d-aa996f5e3612" message="transactionID: #[vars.transactionId]], correlationID: #[vars.correlationID], message: &quot;After report case to who service call&quot;, payload: #[payload]"/>
            </sub-flow>
            <sub-flow name="post-covid-case-service-request" doc:id="12343dae-90cd-4952-b68f-ba502d7fb54e">
                <logger level="DEBUG" doc:name="Before Backend Call" doc:id="2a84da65-40f1-44c5-9131-138a0f4d4d01" message="transactionID: #[vars.transactionId]], correlationID: #[vars.correlationID], message: &quot;Before add case uhub sapi service call&quot;, payload: #[payload]"/>
                <http:request method="POST" doc:name="Add Covid Case " doc:id="52b0c630-33cf-40fd-8203-3961a30a094c" config-ref="HTTP_UHUB_SAPI_Request_configuration" path="/${secure::ehub.sapi.subpath.register.case}">
                    <http:body>
                        <![CDATA[ #[vars.covidCasePayload] ]]>
                    </http:body>
                    <http:headers>
                        <![CDATA[ #[vars.uhubHeaderParameters] ]]>
                    </http:headers>
                </http:request>
                <logger level="DEBUG" doc:name="After Backend Call" doc:id="15cf6be4-0b85-4c79-a587-9685dadfb535" message="transactionID: #[vars.transactionId]], correlationID: #[vars.correlationID], message: &quot;After add case uhub sapi service call&quot;, payload: #[payload]"/>
            </sub-flow>
            <sub-flow name="get-covid-case-by-national-id" doc:id="1b0fd322-c4f2-4a06-87af-aee298269d2f">
                <logger level="DEBUG" doc:name="Before Backend Call" doc:id="912bec8e-3dbb-4965-bbec-ba0586d6fbbe" message="transactionID: #[vars.transactionId]], correlationID: #[vars.correlationID], message: &quot;Before get case uhub sapi service call&quot;, payload: #[payload]"/>
                <http:request method="GET" doc:name="Get Covid Case By NationalID" doc:id="47e22801-5d3e-4526-bd0a-956731483829" config-ref="HTTP_UHUB_SAPI_Request_configuration" path="/${secure::ehub.sapi.subpath.get.case}/{nationalId}">
                    <http:headers>
                        <![CDATA[ #[vars.uhubHeaderParameters] ]]>
                    </http:headers>
                    <http:uri-params>
                        <![CDATA[ #[{ &quot;nationalId&quot;: vars.covidCasePayload.nationalID }] ]]>
                    </http:uri-params>
                    <http:response-validator>
                        <http:success-status-code-validator values="200,404"/>
                    </http:response-validator>
                </http:request>
                <logger level="DEBUG" doc:name="After Backend Call" doc:id="ecdda797-65e5-47c0-8e1b-f98ec9ce6069" message="transactionID: #[vars.transactionId]], correlationID: #[vars.correlationID], message: &quot;After get case uhub sapi service call&quot;, payload: #[payload]"/>
            </sub-flow>
            <sub-flow name="post-upload-document-service-request" doc:id="ae9d59be-4ef7-4f29-896b-1d842af54c5a">
                <logger level="DEBUG" doc:name="Before Backend Call" doc:id="74ead57b-5a32-463b-8bc2-97f8d7aac01d" message="transactionID: #[vars.transactionId]], correlationID: #[vars.correlationID], message: &quot;Before upload document aws sapi service call&quot;, payload: #[payload]"/>
                <http:request method="POST" doc:name="Call Upload Document Service" doc:id="afb881eb-19a8-4874-8652-c62d39f31a68" config-ref="HTTP_AWS_SAPI_Request_configuration" path="/${secure::aws.sapi.subpath}">
                    <http:headers>
                        <![CDATA[ #[vars.awsHeaderParameters] ]]>
                    </http:headers>
                </http:request>
                <logger level="DEBUG" doc:name="After Backend Call" doc:id="c7aa5736-0426-4322-bfff-3be1aa354a13" message="transactionID: #[vars.transactionId]], correlationID: #[vars.correlationID], message: &quot;After upload document aws sapi service call&quot;, payload: #[payload]"/>
            </sub-flow>
        </mule>

    - #### global.xml
      <details>
      <summary></summary>
    
      ```xml
      <mule xmlns:apikit="http://www.mulesoft.org/schema/mule/mule-apikit" xmlns:http="http://www.mulesoft.org/schema/mule/http" xmlns:secure-properties="http://www.mulesoft.org/schema/mule/secure-properties" xmlns="http://www.mulesoft.org/schema/mule/core" xmlns:doc="http://www.mulesoft.org/schema/mule/documentation" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation=" http://www.mulesoft.org/schema/mule/mule-apikit http://www.mulesoft.org/schema/mule/mule-apikit/current/mule-apikit.xsd http://www.mulesoft.org/schema/mule/http http://www.mulesoft.org/schema/mule/http/current/mule-http.xsd http://www.mulesoft.org/schema/mule/core http://www.mulesoft.org/schema/mule/core/current/mule.xsd http://www.mulesoft.org/schema/mule/secure-properties http://www.mulesoft.org/schema/mule/secure-properties/current/mule-secure-properties.xsd">
          <http:listener-config name="covid-papi-httpListenerConfig" basePath="covid">
              <http:listener-connection host="0.0.0.0" port="${http.port}"/>
          </http:listener-config>
          <apikit:config name="covid-papi-config" api="covid-papi.raml" outboundHeadersMapName="outboundHeaders" httpStatusVarName="httpStatus"/>
          <secure-properties:config name="Secure_Properties_Config" doc:name="Secure Properties Config" doc:id="8a51007e-1d2b-4ce9-94b1-03ae40fd2935" file="${env}.yaml" key="${enc.key}"/>
          <http:request-config name="HTTP_UHUB_SAPI_Request_configuration" doc:name="HTTP Request configuration" doc:id="9295e8d3-1453-458a-bbd4-4fd9dc2c5d45" basePath="/${secure::ehub.sapi.basepath}">
              <http:request-connection host="${secure::ehub.sapi.host}" port="${secure::ehub.sapi.port}"/>
          </http:request-config>
          <http:request-config name="HTTP_AWS_SAPI_Request_configuration" doc:name="HTTP Request configuration" doc:id="35ab1b26-b2b1-4135-9e25-ab98229303f7" basePath="/${secure::aws.sapi.basepath}">
              <http:request-connection host="${secure::aws.sapi.host}" port="${secure::aws.sapi.port}"/>
          </http:request-config>
          <http:request-config name="HTTP_WHO_Request_configuration" doc:name="HTTP Request configuration" doc:id="80aec699-9c3d-46e9-be9c-3d887b5327aa" basePath="/${secure::who.sapi.basepath}">
              <http:request-connection host="${secure::who.sapi.host}" port="${secure::who.sapi.port}"/>
          </http:request-config>
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
              <on-error-propagate type="APIKIT:UNSUPPORTED_MEDIA_TYPE" enableNotifications="true" logException="true" doc:name="On Error Propagate" doc:id="0a5707e2-e14f-4f47-8972-cc6d3679b5d1">
                  <set-variable value="#[415]" doc:name="Set HTTP Status - 415" doc:id="6e0cf929-8d7d-469e-b0a9-3962dcbb2aa2" variableName="httpStatus"/>
                  <set-variable value="Unsupported media type" doc:name="Set Error Message" doc:id="c729dfc6-7154-44ff-bf47-efb9a35df92c" variableName="errorMessage"/>
                  <set-variable value="The server is refusing to service the request because the entity of the request is in a format not supported by the requested resource for the requested method" doc:name="Set Error Description" doc:id="e4b1ff27-98a9-4c28-a0c1-b95c13bfc57e" variableName="errorDescription"/>
                  <flow-ref doc:name="global-prepare-error-response-sub-flow" doc:id="5df3ad13-498a-4035-a84a-1bcf9c170364" name="global-prepare-error-response-sub-flow"/>
              </on-error-propagate>
              <on-error-propagate type="APIKIT:UNSUPPORTED_MEDIA_TYPE" enableNotifications="true" logException="true" doc:name="On Error Propagate" doc:id="30d0545c-5077-4906-855d-35e96c1dd1cf">
                  <set-variable value="#[415]" doc:name="Set HTTP Status - 415" doc:id="e5b75090-6027-46a5-a4ad-87517d5e3d49" variableName="httpStatus"/>
                  <set-variable value="Unsupported media type" doc:name="Set Error Message" doc:id="ea78063e-f1bc-40aa-b7b8-e9e91a7d5f7d" variableName="errorMessage"/>
                  <set-variable value="The server is refusing to service the request because the entity of the request is in a format not supported by the requested resource for the requested method" doc:name="Set Error Description" doc:id="65d5a4fb-29f0-4477-bdb1-b7e584363320" variableName="errorDescription"/>
                  <flow-ref doc:name="global-prepare-error-response-sub-flow" doc:id="0158cbb0-48bc-46d9-bc39-0265a3cc2068" name="global-prepare-error-response-sub-flow"/>
              </on-error-propagate>
              <on-error-propagate type="APIKIT:UNSUPPORTED_MEDIA_TYPE" enableNotifications="true" logException="true" doc:name="On Error Propagate" doc:id="b1a01e94-5a6a-487b-9f8c-2cf98d373448">
                  <set-variable value="#[415]" doc:name="Set HTTP Status - 415" doc:id="5b43b1eb-fc18-46d0-a84b-514c78b35c3a" variableName="httpStatus"/>
                  <set-variable value="Unsupported media type" doc:name="Set Error Message" doc:id="084c8c43-18f6-4968-acef-f9ed8fd6c5a3" variableName="errorMessage"/>
                  <set-variable value="The server is refusing to service the request because the entity of the request is in a format not supported by the requested resource for the requested method" doc:name="Set Error Description" doc:id="2d7ad191-3640-4f4c-9c30-1d2ef3e2e64e" variableName="errorDescription"/>
                  <flow-ref doc:name="global-prepare-error-response-sub-flow" doc:id="c23638a6-bfa2-4f5c-864e-598ef5b49633" name="global-prepare-error-response-sub-flow"/>
              </on-error-propagate>
              <on-error-propagate type="APIKIT:UNSUPPORTED_MEDIA_TYPE" enableNotifications="true" logException="true" doc:name="On Error Propagate" doc:id="7b8b11b5-c3fd-4b26-a219-f0cc5e1be863">
                  <set-variable value="#[415]" doc:name="Set HTTP Status - 415" doc:id="b034ab4d-89a5-462d-b8f5-c2676f4f9782" variableName="httpStatus"/>
                  <set-variable value="Unsupported media type" doc:name="Set Error Message" doc:id="f041cb10-2094-4043-b4d2-f8230ed79211" variableName="errorMessage"/>
                  <set-variable value="The server is refusing to service the request because the entity of the request is in a format not supported by the requested resource for the requested method" doc:name="Set Error Description" doc:id="6dfeea24-2a19-46e5-8cc9-b75798a5cb62" variableName="errorDescription"/>
                  <flow-ref doc:name="global-prepare-error-response-sub-flow" doc:id="7ad15d15-363b-4a9d-bf82-55e35812353c" name="global-prepare-error-response-sub-flow"/>
              </on-error-propagate>
              <on-error-propagate type="APIKIT:UNSUPPORTED_MEDIA_TYPE" enableNotifications="true" logException="true" doc:name="On Error Propagate" doc:id="31d15a3e-c865-4483-908d-f9c1a973bb2b">
                  <set-variable value="#[415]" doc:name="Set HTTP Status - 415" doc:id="8580f9a2-2b02-4c8b-bb30-76a4f7ff7fd7" variableName="httpStatus"/>
                  <set-variable value="Unsupported media type" doc:name="Set Error Message" doc:id="2d7dbde1-ec8d-42e0-8242-1ec7d230d5a3" variableName="errorMessage"/>
                  <set-variable value="The server is refusing to service the request because the entity of the request is in a format not supported by the requested resource for the requested method" doc:name="Set Error Description" doc:id="f4a2f8c6-500b-4c3b-a98b-b8625a8a498f" variableName="errorDescription"/>
                  <flow-ref doc:name="global-prepare-error-response-sub-flow" doc:id="77c93a25-2488-4699-a02b-6bfb9f1df11a" name="global-prepare-error-response-sub-flow"/>
              </on-error-propagate>
              <on-error-propagate type="APIKIT:UNSUPPORTED_MEDIA_TYPE" enableNotifications="true" logException="true" doc:name="On Error Propagate" doc:id="5fb21d8b-6058-4642-a1a7-34b6b1a6fc4e">
                  <set-variable value="#[415]" doc:name="Set HTTP Status - 415" doc:id="1075a16a-eeba-47ee-bcd4-bb45f5da9d8b" variableName="httpStatus"/>
                  <set-variable value="Unsupported media type" doc:name="Set Error Message" doc:id="0c7978c2-57e1-4b0e-a29a-3eb88b8f2e3c" variableName="errorMessage"/>
                  <set-variable value="The server is refusing to service the request because the entity of the request is in a format not supported by the requested resource for the requested method" doc:name="Set Error Description" doc:id="1d501d0e-e916-4fd4-9469-1815d535f5c3" variableName="errorDescription"/>
                  <flow-ref doc:name="global-prepare-error-response-sub-flow" doc:id="29c1f8b8-6b49-4451-aa45-56f5fd06efb8" name="global-prepare-error-response-sub-flow"/>
              </on-error-propagate>
              <on-error-propagate type="APIKIT:UNSUPPORTED_MEDIA_TYPE" enableNotifications="true" logException="true" doc:name="On Error Propagate" doc:id="ff0b9f9c-4f35-4e5f-8851-17f784e06f8a">
                  <set-variable value="#[415]" doc:name="Set HTTP Status - 415" doc:id="bf2cb712-b642-49dd-b2f4-53b854c3284e" variableName="httpStatus"/>
                  <set-variable value="Unsupported media type" doc:name="Set Error Message" doc:id="7b29a876-f8e4-4b77-b816-107cdd69ee8d" variableName="errorMessage"/>
                  <set-variable value="The server is refusing to service the request because the entity of the request is in a format not supported by the requested resource for the requested method" doc:name="Set Error Description" doc:id="eb84d961-b517-46b1-b19c-b9a9398c0142" variableName="errorDescription"/>
                  <flow-ref doc:name="global-prepare-error-response-sub-flow" doc:id="db06216b-c001-4b7e-8ef0-b77a80e5d4bf" name="global-prepare-error-response-sub-flow"/>
              </on-error-propagate>
          </error-handler>
      </flow>

    - #### covid-papi.xml
      <details>
      <summary></summary>
      
      ```xml
      <mule xmlns:doc="http://www.mulesoft.org/schema/mule/documentation" xmlns="http://www.mulesoft.org/schema/mule/core" xmlns:apikit="http://www.mulesoft.org/schema/mule/mule-apikit" xmlns:http="http://www.mulesoft.org/schema/mule/http" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.mulesoft.org/schema/mule/core http://www.mulesoft.org/schema/mule/core/current/mule.xsd http://www.mulesoft.org/schema/mule/http http://www.mulesoft.org/schema/mule/http/current/mule-http.xsd http://www.mulesoft.org/schema/mule/mule-apikit http://www.mulesoft.org/schema/mule/mule-apikit/current/mule-apikit.xsd ">
          <flow name="covid-papi-main">
              <http:listener config-ref="covid-papi-httpListenerConfig" path="/v1/*">
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
              <set-variable value="#[attributes.headers.'x-transaction-id' default uuid()]" doc:name="Set transactionId" doc:id="58261580-8891-4810-a69e-4120749c9eb8" variableName="transactionId"/>
              <apikit:router config-ref="covid-papi-config"/>
              <error-handler ref="global-error-handler"/>
          </flow>
          <flow name="covid-papi-console">
              <http:listener config-ref="covid-papi-httpListenerConfig" path="/console/*">
                  <http:response statusCode="#[vars.httpStatus default 200]">
                      <http:headers>#[vars.outboundHeaders default {}]</http:headers>
                  </http:response>
                  <http:error-response statusCode="#[vars.httpStatus default 500]">
                      <http:body>#[payload]</http:body>
                      <http:headers>#[vars.outboundHeaders default {}]</http:headers>
                  </http:error-response>
              </http:listener>
              <apikit:console config-ref="covid-papi-config"/>
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
          get.case: "cases"
          register.case: "cases"
          update.case: "cases"
        
      aws.sapi:
        host: "uho-dev-aws-sapi.us-e2.cloudhub.io"
        port: "80"
        basepath: "covid/v1"
        client_id: ""
        client_secret: ""
        subpath: "document"
        
      aws.s3:
        bucket.name: "uho-covid-docs"
        folder.path: "identity"
       
      who.sapi:
        host: "uho-dev-who-sapi.us-e2.cloudhub.io"
        port: "80"
        basepath: "covid/v1"
        client_id: ""
        client_secret: ""
        subpath: "case"

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
    <artifactId>covid-papi</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>mule-application</packaging>
    <name>covid-papi</name>
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
      <dependency>
        <groupId>org.mule.connectors</groupId>
        <artifactId>mule-jms-connector</artifactId>
        <version>1.7.5</version>
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
  
