# Quickstart to test the Password Hashing Serve #
This document shall act as a quickstart guide to understand about testing the Password hashing service in a Windows 10 local server.

## Tool dependencies ##
The testing of the application is done using various freeware tools available in the market.
1. Postman v7.19.1
2. Cygwin 3.1.1
3. curl 7.66.0 (x86_64-pc-cygwin) libcurl/7.66.0
4. xargs (GNU findutils) 4.6.0

## Testing overview ##
The testing of the application was a multi-faceted approach which included manual as well as simple automation efforts. The test coverage choice could be categorised as follows.

### Functional ###
Functional testing was mostly performed using Postman javascript assertions as well as manually using cURLs. 

The test scenarios included are the following:

Method name  | Method Type | Test Scenario | Reference | Test Result
------------- | ------------- | ------------- | ------------- | -------------
/hash | POST | Validate for successful response code | Postman: hashingJob/validatePasswordHashEndpointFunctionality | <span style="color:green">*Pass*</span>
/hash | POST | Status code is '415 Unsupported Media Type' when the 'Content-Type' Header value is not 'application/json' | Postman: hashingJob/validatePasswordHashEndpointForUnsupportedMediaType | <span style="color:red">*Fail*</span>
/hash | POST | Validate if the job identifier value is getting returned immediately | ``` curl 'http://127.0.0.1:8088/hash' --header 'Content-Type: application/json' --header 'Accept: */*' --data-raw '{"password": "angrymonkey"}' --compressed -s -o /dev/null -w  "\ntime_namelookup:  %{time_namelookup}\ntime_connect:  %{time_connect}\ntime_appconnect:  %{time_appconnect}\ntime_pretransfer:  %{time_pretransfer}\ntime_redirect:  %{time_redirect}\ntime_starttransfer:  %{time_starttransfer}\n__________\ntime_total:  %{time_total}\n" ``` | <span style="color:red">*Fail*</span>
/hash | GET | Validate for successful response code | Postman: retrievePasswordHash/validateHashedPasswordRetrievalEndpointFunctionality | <span style="color:green">*Pass*</span>
/hash | GET | Response time is less than 10ms | Postman: retrievePasswordHash/validateHashedPasswordRetrievalEndpointFunctionality | <span style="color:green">*Pass*</span>
/hash | GET | Response body text is the Base64 encoded value of the password string after SHA-512 hashing | Postman: retrievePasswordHash/validateHashedPasswordRetrievalEndpointFunctionality | <span style="color:red">*Fail*</span>
/stats | GET | Validate for successful response code | Postman: hashStatistics/valiateHashingStatsEndpointFunctionality | <span style="color:green">*Pass*</span>
/stats | GET | Response time is less than 10ms | Postman: hashStatistics/valiateHashingStatsEndpointFunctionality | <span style="color:green">*Pass*</span>
/stats | GET | Validate if the response Content-Type is of type JSON | Postman: hashStatistics/valiateHashingStatsEndpointFunctionality | <span style="color:red">*Fail*</span>
/stats | GET | Verify the response value of total hash requests since server started | Postman: hashStatistics/valiateHashingStatsEndpointFunctionality | <span style="color:green">*Pass*</span>
/stats | GET | Verify the average response time for a hash request in milliseconds after including the hash calculation | Postman: hashStatistics/valiateHashingStatsEndpointFunctionality | <span style="color:red">*Fail*</span>
/hash (shutdown) | POST | Verify if the software supports a graceful shutdown - status code is 200 | Postman: tearDown/validateShutdownEndpointResponseCode | <span style="color:green">*Pass*</span>
/hash | POST | Verify if no additional password requests are allowed when the shutdown is pending | <code>cat validate-shutdown-functionality &#124; while read n; do printf "%q\n" "$n"; done &#124; xargs --max-procs=2 -I LC bash -c LC</code> | <span style="color:green">*Pass*</span>

### Performance ###
Performance impacts on the service during concurrency was tested manually using cURLs. The usage of jmeter or any other performace specific tools has not been considered at this time as we don't have specific performance requirements documented at this point of time. The following are the scenario(s) tested.

Method name  | Method Type | Test Scenario | Reference | Test Result
------------- | ------------- | ------------- | ------------- | -------------
/hash | POST | Validate if the software is able to process multiple connections simultaneously | ``` xargs --max-procs=3 -I curl --verbose 'http://127.0.0.1:8088/hash' --header 'Accept: */*' --header 'Content-Type: application/json' --data '{"password":"angrymonkey"}' < <(printf '%s\n' {1..3}) ``` | <span style="color:green">*Pass*</span>

### Security ###
It could be idenfied from the manual verification of the Password Hashing Serve application that it is insecure currently as there is no authentication involved in accessing the service and hence a third party could easily snoop in to the application and manipulate the results.

### Issues Identified ###
1. The ***job identifier*** is getting returned from the ***/hash POST call*** only after 5 seconds wait and the computing the hash time, even though it's expected immediately.
2. Status code for the ***/hash POST call*** is 200 despite Content-Type header is being supplied as ***test/plain***, even though a ***415: Unsupported Media type*** response is expected.
3. Response body text for the ***/hash GET call*** is not the Base64 encoded value of the password string after SHA-512 hashing, as expected.
4. The response Content-Type  for the ***/stats GET call*** is of the type ***text/plain*** instead of ***application/json***.
5. The average response time value caculation in the ***/stats GET call*** response body is incorrect.

## References
* __xargs:__  http://man7.org/linux/man-pages/man1/xargs.1.html
* __cURL:__ https://curl.haxx.se/docs/manpage.html
