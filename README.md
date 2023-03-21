Market Informed Demand Automation Server (MIDAS) Documentation
==============================================================

_Connecting to and Interacting with the MIDAS API_


# Introduction

The The California Energy Commission’s (CEC) Market Informed Demand Automation Server (MIDAS) is a database and application programming interface (API) that provides access to current, future, and historic time-varying rates, greenhouse gas (GHG) emissions associated with electrical generation, and California Flex Alert Signals. The database is populated by utilities and Community Choice Aggregators (CCAs), WattTime’s Self-Generation Incentive Program (SGIP) marginal GHG emissions API, the California Independent System Operator (California ISO), and other entities that are registered with the MIDAS system.

MIDAS is designed to provide energy users with the electricity price information they need to optimize when they use energy. While it would be useful for electricity users to be able to use the data from MIDAS to try to estimate electricity bill totals, some billing structures such as tiered rates make this impossible without private customer-specific data. MIDAS does not and will not contain any private information. The only non-public data in MIDAS is login information.

MIDAS is accessible through a public API at <https://midasapi.energy.ca.gov> in two standard machine-readable formats: extensible markup language (XML), and JavaScript Object Notation (JSON). MIDAS is public and accessible for all registered users to query and download information by interacting with the MIDAS API. CEC strongly encourages Load Serving Entity (LSE) users have programming skills and software to effectively upload and maintain rate information stored in the database. Non-LSE users should be able to retrieve information stored in MIDAS without extensive programming skills. Retrieving MIDAS-hosted data can be easily done through the code examples provided or through a user’s own code. For instructions on accessing the MIDAS database, see section 3.


# Database Structure

The MIDAS database supports retrieval of electric utility price schedules, California Flex Alert signals, and marginal GHG emissions from electrical generation. Flex Alerts and GHG emissions values – both forecasted and real-time - are continually retrieved from the California ISO Flex Alert site and WattTime’s SGIP API respectively. GHG realtime and forecasted values are cached, or temporarily stored, within MIDAS until new values are available while Flex Alerts are passed through MIDAS from the California ISO website, as a user queries MIDAS. Previous real-time values are moved to the HistoricalData table automatically. A record of previously active Flex Alerts and historic GHG emissions are stored in the HistoricalData table.

Pursuant to the Load Management Standards, the state’s largest utilities and CCAs are responsible for populating the MIDAS database Holiday Table, RateInfo Table, and the Value Table with all time-varying rate information and values offered to customers. For upload examples, please see [Appendix A](appendix-a.md). For instructions on how to retrieve the XML upload schema, please see section 3.

The primary lookup identification (ID) for the MIDAS database is a compound key comprised of six individual fields that make up a standardized rate identification number (RIN) as shown in Figure 1. RINs are assigned at the time rate information is first uploaded by the LSE through the MIDAS API. When an LSE uploads to an existing RIN, the correct RIN must be used at the time of upload. Figure 1 illustrates the six identifiers that comprise a RIN: Country, State, Distribution, Energy, Rate, and Location. The location portion of the RIN may consist of 1 to 10 characters depending on the specified location’s requirements.

Figure 1. Rate Identification Number Structure
![Rate Identifican Number Specification](img/RIN-structure.png)
Source: California Energy Commission

## Rate Information
% TODO: Add more detail about rate information tables RateInfo and Value

## SGIP GHG Emissions

WattTime estimates GHG emissions for 11 regions across the state of California providing real-time and forecasted values for each. MIDAS includes a total of 33 different GHG RINS for California - real-time, forecasted, and historic. RINs with the first 12 characters USCA-SGIP-SGRT-XXXX, USCA-SGIP-SGFC-XXXX, and USCA-SGIP-SGHT-XXXX allow access to the information retrieved from queries performed on the [WattTime.org SGIP API](https://sgipsignal.com) in MIDAS. The portion of the RIN specified as “SGRT” stands for “SGIP real-time”, “SGFC” stands for “SGIP forecast”, and “SGHT” stands for “SGIP historic”. The SGRT RIN will return only one data point that specifies the current CO2 level for the specified region. The SGFC RIN will return the forecasted CO2 levels, at 5-minute intervals, available from the WattTime API. SGHT RINs will return the historic information for each SGIP GHG emissions region. Once a real-time emissions value is replaced with a new current value, the information is moved to the HistoricalData table where it can be retrieved through the SGHT RIN. For a list of the regions and region abbreviations please see WattTime’s SGIP webpage at: <https://sgipsignal.com/grid-regions>.

## CAISO Flex Alerts

MIDAS includes 3 different Flex Alert RINS - real-time, forecasted, and historic values with dates and times of previous Flex Alerts. The California ISO maintains information as to whether there is an active Flex Alert and whether one is planned. MIDAS checks the California ISO Flex Alert web page for Flex Alerts and adds it to the database.

RINs of USCA-FLEX-FXRT-0000 and USCA-FLEX-FXFC-0000 check  at the moment a query is initiated through the MIDAS API. The data is passed through to the querying user directly and instantaneously. The “FXRT” portion of the first RIN stands for “Flex Alert real-time” and “FXFC” in the second RIN stands for “Flex Alert forecasted”. In the RIN USCA-FLEX-FXHT-0000, “FXHT” stands for “Flex Alert historical”. All previously active Flex Alerts will be reflected in the information retrieved with this RIN.
For more information on the XML schema and uploads, see [Appendix A](appendix-a.md).

## Archived Data in the HistoricalData Table

MIDAS stores uploaded data in the HistoricalData table to provide a durable record of rate information. Each time an LSE account, whether a distribution or energy company, uploads new rate information, the uploaded information is also added to the HistoricalData table. For example, if an LSE user designated as a distribution company uploads a new file with values for one RIN, that data will be added to the HistoricalData table at the same time it is added to the Value table.
The data in the HistoricalData table will always be up to date as well as providing a historical record. The information in the HistoricalData table can then be queried through the HistoricalData endpoint.


# Getting Information From MIDAS

The MIDAS API is a Representational State Transfer Application Programming Interface (RESTful API) accessible using any programming language able to create instances of a Hypertext Transfer Protocol (HTTP) Client, HTTP Request and HTTP Response classes. Users may develop their own in-house software to connect with the MIDAS RESTful API. The requests (calls) and responses should be executed asynchronously. Most software for interacting with APIs will allow easy interaction with the MIDAS API.

The MIDAS REST API and Database are protected by the CEC firewall and data throttling to prevent distributed denial of service (DDoS) attacks (see [Appendix D](appendix-d.md)). If an error occurs after an API call, the program will send a notification for CEC IT staff to fix the issue.

The MIDAS API is comprised of six endpoints:

* Registration: Use POST to create a new account.
* Token: Use GET to retrieve a temporary token for interacting with the MIDAS API. Tokens must be passed to every API call except Registration and Token.
* Holiday: Use GET to retreive the list of all holidays. Use POST to populate the Holiday table (LSE accounts only).
* ValueData: Use GET to retrieve a list of available rates (RINs), rate information, the XML schema, or lookup tables. Use POST to upload data to the RateInfo and Value tables (LSE accounts only).
* HistoricalData: Use GET to retrieve historic rate information.
* HistoricalList: Use GET to retreive the list of rates (RINs) available from HistoricalData.

The following are instructions for registering a MIDAS account and a description of the basic functions that can be used to upload and download MIDAS data. Here, we provide some examples in the Python programming language. Links to GitHub repositories wwith this and examples in other programming languages are in [Example Code](#example-code) below.

## Register

There are two types of accounts that can be used to interface with MIDAS: LSE accounts and user accounts. For security and accuracy of the system, only CEC-verified LSE accounts can upload (POST) data. Those wishing to retrieve data stored in the MIDAS database must register a user account and use that account to retrieve an access token to authenticate GET and POST requests.

User accounts can only query (GET) data from MIDAS, with the exception of registering for an account using the Registration endpoint.

1. User Accounts: User account registrants may include researchers, automation service providers, technology manufacturers, etc. Registration is done through the API by making a one-time call to the MIDAS registration endpoint with the required parameters. If no errors occur, this process will send an email to the email address specified as a parameter to the call. The user must then respond to the email before they can request a token. User account holders cannot post data.

2. LSE Accounts: New accounts for utilities and community-choice aggregators will follow the same registration process as User accounts. Following a successful registration, send an email to midas@energy.ca.gov from your LSE account to request access.  Accounts must be verified by a CEC staff member on the MIDAS support team to allow LSE-level access. An LSE account may only be registered under one distribution or energy company, allowing upload capabilities under only one company.

A successful registration for both User and LSE accounts will return: “User account for [your username] was successfully created. A verification email has been sent to [your email]. Please click the link in the email to start using the API.”

For forgotten passwords or usernames please follow the links below:

Password: <https://midasweb.energy.ca.gov/Pages/AccountMaint/ForgotPassword>

Username: <https://midasweb.energy.ca.gov/Pages/AccountMaint/ForgotUsername>

Example query:
```
https://midasapi.energy.ca.gov/registration/
```

## GET a Token

After registering, make a GET call to the token endpoint with username and password credentials to receive a token string. This token will expire after 10 minutes. During those 10 minutes, the token can be used to call all other endpoint actions as many times as desired up to the request rate limit. Pass the token with each call made to the system to allow access to the MIDAS database.

## GET RIN List

To receive a full list of all RINs in the MIDAS database, users may query the API using the GET RIN List call. This call is part of the ValueData endpoint with a parameter that identifies the signal type of the RINs being returned. The signal type parameter will return all the RINs of the requested type:

0.	All
1.	Electricity rates
2.	Greenhouse gas emissions
3.	California Independent System Operator Flex Alert

## GET Values

This call allows all accounts to receive values and other information on a specified RIN. Pass a RIN with parameter RealTime to return the current value, or AllData to the ValueData endpoint to return the full schedule in either XML or JSON, as indicated in the header.

## GET Rate Upload XML Schema

This call will allow all accounts to receive the full XML schema for uploading rates to the MIDAS database. This call is part of the ValueData endpoint and uses the GET verb with no parameters. It returns a string with the XML Schema Definition that the system uses to validate incoming XML Upload data.

## GET Lookup Table

To receive the values stored in each lookup table, all users may use the GET Lookup table call. This call is part of the ValueData endpoint with a parameter that identifies the relevant lookup table. Possible lookup tables include Country, Daytype, Distribution, Enduse, Energy, Location, Ratetype, Sector, State, and TimeZone. The returned data will have the upload code and description for the specified lookup table. See [Appendix C](appendix-c.md) for more detail on the lookup tables.

## GET Holiday Table

All accounts may query the MIDAS API for the information stored in the Holiday table. This call is part of the Holiday endpoint. There is no parameter to be specified, the requested data will be returned in either XML or JSON as specified in the header.

## GET Historical RIN List

This call will retrieve all RINs with information stored in the HistoricalData table. This call is part of the HistoricalList endpoint. All users can specify the distributor company, energy company or both that they would like to receive RIN information for. All Flex Alert and GHG Emission historical RINs are automatically returned when querying any distribution or energy company. If a user specifies a code that does not correspond to a RIN in the HistoricalData table, then only GHG and Flex Alert historic RINs will be returned.

## GET RIN History Data

To receive historical information, all accounts may query MIDAS using the GET RIN History Data call. This call is part of the HistoricalData endpoint. Pass a RIN with a parameter that identifies the “startdate” and “enddate” to return historic values for the specified RIN during that timeframe in XML or JSON as specified in the header. Please note, only historic RINs may be used in this call. For example, GHG and Flex Alert RINs have separate historic RINs.

# Posting Data to MIDAS

_**Only CEC approved accounts may upload data to MIDAS.**_ <br>
See [Appendix A](appendix-a.md) for in-depth details on uploading rates to MIDAS.


# Example Code

These are several sets of example code and an R package for working with the MIDAS API.

## Python

Repository of Python language example code for working with MIDAS: 
<https://github.com/morganmshep/MIDAS-Python-Repository>

## R

Repository of R language example code for working with MIDAS: <https://github.com/morganmshep/MIDAS-R-Repository>

Repository for installable R language package for easily and reliably working with MIDAS: <https://github.com/stefwayland/cecmidas>

## C-Sharp (C#)

Repository of C# language example code for working with MIDAS: 
<https://github.com/morganmshep/MIDAS-CSharp-Repository>


# Appendices

[Appendix A](appendix-a.md) discusses rate and holiday upload and links to example upload documents.

[Appendix B](appendix-b.md) contains a list of acronyms and a glossary.

[Appendix C](appendix-c.md) contains a list of all lookup tables available through MIDAS.

[Appendix D](appendix-d.md) contains a diagram of the MIDAS API service architecture.


