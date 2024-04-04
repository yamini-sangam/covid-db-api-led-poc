# <p align="center">Raml Code</p>
# mss-who-sapi 
- ## examples
  - ### 400.json
     <details>
      <summary></summary>
    
        ```json
        
        ```
      </details>
  - ### 404.json
  - ### 500.json
  - ### 503.json
  - ### get-reports-reponse.json
- ## covid-eapi-types
  - ### covid-eapi-types.raml
  - ### state-query-param.raml
- ## covid-eapi.raml

# <p align="center">Implementation of mss-covid-eapi</p>  
# mss-covid-eapi
- ## src\main
  - ### mule
    - #### implementations
      - ##### health-check.xml
      - ##### get-covid-reports.xml
    - #### subflows
      - ##### uhub-sapi-rest-calls.xml
    - #### global.xml
    - #### global-error-handler.xml
    - #### covid-eapi.xml
  - ### resources
    - #### dev.yaml
    - #### prod.yaml
