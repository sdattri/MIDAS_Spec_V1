# Appendix A - Uploading to MIDAS

Only CEC approved entities can upload to MIDAS. To request upload capabilities, after establishing a MIDAS account by registering through the `registration` API endpoint and verifying your email by clicking on the link, then send an email to midas@energy.ca.gov from your LSE email account. CEC staff will review your request and respond.

## Rate and Holiday Upload Support

LSEs are encouraged to reach out to CEC staff at midas@energy.ca.gov with any questions or issues with MIDAS. Staff will do our best to answer any questions and address issues quickly.

The rate upload XML schema is available through the MIDAS API. The version from the API is the canonical one. However, we are also providing the XML schema document as part of this documentation. [XML Schema Document](support-docs/MIDAS-upload-XML-schema.xsd)

## Rate Definitions

MIDAS is designed to provide energy users with the electricity price information they need to optimize when they use energy. While it would be useful for electricity users to be able to use the data from MIDAS to try to estimate electricity bill totals, some billing structures such as tiered rates make this impossible without private customer-specific data. The decision to exclude private data from MIDAS has an upside&mdash;inclusion of just the time-dependent portions of rates, as required by the CEC Load Management Standards. This reduces the number of different rates that need to be uploaded to MIDAS while still providing electricity users with the information they need to optimize their energy use over periods of hours to days.

In order for energy users to effectively optimize energy use, they need information on the time-dependent portions of their rates. As of spring 2023, this usually means that only the generation and distribution portions of rates are required to be included since few transmission rates in California are currently time-dependent.

## Upload Rates

Uploading rates needs to happen prior to any changes in those rates.

### Rate Examples

Here are some example rate upload XML documents to help with formatting rate uploads. Each example XML file only contains a few days worth of data (March 1-3, 2023) for readability. 

* **TOU rate** An example of a TOU rate with hourly values. A full year of this rate would have 8760 ValueData blocks
* **Streaming rate** This file is the same format as the TOU rate, but prices vary hourly
* **TOU rate with time-varying demand charges** This file contains the same rate as the TOU file, but also contains ValueData blocks for time-dependent demand charges.
% TODO: Add these example rates to the support-docs folder

### Assigning a RIN

The first step to uploading a rate to MIDAS is to determine the Rate Identification Number (RIN). See Figure 1 above for the RIN structure. If a rate already exists in MIDAS and you don't have the RIN available, refer to the [Get RIN LIST](#get-rin-list) section above to download the list of RINs, and make sure to use the existing RIN. If the rate has never been previously uploaded to MIDAS, you will need to determine the RIN before uploading.

For California-based LSEs, the first four characters of the RIN will always be **USCA**, showing that the LSE is in the United States (US) and California (CA). For utility users in other countries and states/provinces, refer to the **Country** and **State** lookup tables to find your country and state code.

The next step is to determine the distribution and energy company codes that make up the next four characters of the RIN. Refer to the **Distribution** and **Energy** lookup tables to find the two character codes that correspond to your distribution and energy companies. For example, Marin Clean Energy would use the Pacific Gas and Electric distribution company code **PG** and the Marin Clean Energy energy code **MC**, yielding **PGMC** for the second four characters.

The next four characters are open for the LSE to define. These four alphanumeric characters (containing only uppercase english letters and the numeric digits 0-9) should, to the extent possible, reflect the rate. For example, a commercial TOU rate could have the four characters **CTOU**, or a critical-peak rate could have the characters **CPP2**.

The final 4 to 10 characters are the location code. For rates with no specific location, use the four character code **0000**. The list of allowable location codes are available in the **Location** lookup table. If your LSE has location codes to add to that table, contact the MIDAS team at midas@energy.ca.gov to request the addition of those codes.

Putting this together for a full example, the full RIN for a rate at Marin Clean Energy could be **USCA-PGMC-CTOU-0000**.

### Rate Upload Data Structure

The canonical requirements for the rate upload structure is the XML rate upload schema available through the API at the `/ValueData` endpoint. You can use this schema verify that your rate upload is structured correctly prior to uploading to MIDAS using XML validation software.

**Endpoint:** `/ValueData`

**HTTP Request:** `POST https://midasapi.energy.ca.gov/api/ValueData`

**Authorization:** Bearer

**Query Parameters:** None

**Body Parameters:** None

**Python Example**

```python
import requests
import xml.etree.ElementTree as ET

headers = {'accept': 'application/json', 'Authorization': "Bearer " + token}
url = 'https://midasapi.energy.ca.gov/api/valuedata'
pricing_response = requests.get(url, headers = headers)

element = ET.XML(json.loads(pricing_response.text))
ET.indent(element)
print(ET.tostring(element, encoding='unicode'))
```

#### Streaming Rate Structure
All rates uploaded to MIDAS should be in a "streaming" structure. This is a time-series structure where every hour (or sub-hourly period) has an entry. There needs to be at least one value for every hour, if the rate changes with a frequency higher than hourly, it needs to have one entry for each period where the rate could change, even when it does not change.

The `DateStart` and `TimeStart`, and `DateEnd` and `TimeEnd` fields in the rate must be in UTC. Combining `DateStart` and `TimeStart` will yield a UTC datetime, as will combining `DateEnd` and `TimeEnd`. For example, for the first hour of March 1st in California (UTC-8), we would convert an interval start date of "2023-01-01" and start time of "00:00:00" to UTC, yielding a `DateStart` of "2023-01-01" and `TimeStart` of "08:00:00".

One day of a streaming rate would include the information Table 1. The table shows data for March 1, 2023 in the "America/Los_Angeles", also known as "PST/PDT" time zone. Note that the dates and times are all in UTC:

Table 1: _Example_ Hourly Rate Information for 2023-03-01 <br>
|DateStart|TimeStart|DateEnd|TimeEnd|DayStart|DayEnd|ValueName|Value|Unit|
|---------|---------|-------|-------|--------|------|---------|-----|----|
|2023-03-01|08:00:00|2023-03-01|08:59:59|3|3|winter off peak|0.1006|$/kWh|
|2023-03-01|09:00:00|2023-03-01|09:59:59|3|3|winter off peak|0.1006|$/kWh|
|2023-03-01|10:00:00|2023-03-01|10:59:59|3|3|winter off peak|0.1006|$/kWh|
|2023-03-01|11:00:00|2023-03-01|11:59:59|3|3|winter off peak|0.1006|$/kWh|
|2023-03-01|12:00:00|2023-03-01|12:59:59|3|3|winter off peak|0.1006|$/kWh|
|2023-03-01|13:00:00|2023-03-01|13:59:59|3|3|winter off peak|0.1006|$/kWh|
|2023-03-01|14:00:00|2023-03-01|14:59:59|3|3|winter off peak|0.1006|$/kWh|
|2023-03-01|15:00:00|2023-03-01|15:59:59|3|3|winter off peak|0.1006|$/kWh|
|2023-03-01|16:00:00|2023-03-01|16:59:59|3|3|winter off peak|0.1006|$/kWh|
|2023-03-01|17:00:00|2023-03-01|17:59:59|3|3|winter off peak|0.1006|$/kWh|
|2023-03-01|18:00:00|2023-03-01|18:59:59|3|3|winter off peak|0.1006|$/kWh|
|2023-03-01|19:00:00|2023-03-01|19:59:59|3|3|winter off peak|0.1006|$/kWh|
|2023-03-01|20:00:00|2023-03-01|20:59:59|3|3|winter off peak|0.1006|$/kWh|
|2023-03-01|21:00:00|2023-03-01|21:59:59|3|3|winter off peak|0.1006|$/kWh|
|2023-03-01|22:00:00|2023-03-01|22:59:59|3|3|winter off peak|0.1006|$/kWh|
|2023-03-01|23:00:00|2023-03-01|23:59:59|3|3|winter off peak|0.1006|$/kWh|
|2023-03-02|00:00:00|2023-03-01|00:59:59|3|3|winter off peak|0.1006|$/kWh|
|2023-03-02|01:00:00|2023-03-01|01:59:59|3|3|winter on peak|0.1388|$/kWh|
|2023-03-02|02:00:00|2023-03-01|02:59:59|3|3|winter on peak|0.1388|$/kWh|
|2023-03-02|03:00:00|2023-03-01|03:59:59|3|3|winter on peak|0.1388|$/kWh|
|2023-03-02|04:00:00|2023-03-01|04:59:59|3|3|winter off peak|0.1006|$/kWh|
|2023-03-02|05:00:00|2023-03-01|05:59:59|3|3|winter off peak|0.1006|$/kWh|
|2023-03-02|06:00:00|2023-03-01|06:59:59|3|3|winter off peak|0.1006|$/kWh|
|2023-03-02|07:00:00|2023-03-01|07:59:59|3|3|winter off peak|0.1006|$/kWh|

#### Rate Information

Uploaded rates must also contain rate information that makes up the header section of the XML. Many of these are optional, but including all of those applicable to the rate will substantially help users. 

* **RateID** _required_ This is the Rate Identification Number (RIN)
* **RateName** _required_ The LSE’s name for the rate plan, consistent with the CEC’s Interval Meter Database as required by California Code of Regulations, Title 20, section 134¬4
* **AltRateName1** _optional_ An alternative name for the rate
* **AltRateName2** _optional_ A second alternative name for the rate
* **SignupCloseDate** _optional_ The last day a customer may sign up for the rate
* **RatePlan_Url** _optional_ A valid URL that directs to the utility webpage describing the rate plan
* **RateType** _optional_ The applicable rate type. Must be one of those in the RateType lookup table
* **Sector** _optional_ The sector that the rate applies to. Must be one of those in the Sector lookup table
* **EndUse** _optional_ The end use that the rate applies to. Must be one of those in the EndUse lookup table
* **API_Url** _optional_ A valid uniform resource locator (URL) that specifies the API that provides the values

### POST Rate Information

_This function is available to authorized LSE accounts only._ <br>
Populating the RateInfo and Value tables requires a call to the ValueData endpoint using the XML schema (see section 3). Acceptable data entries are catalogued in supporting MIDAS lookup tables listed in the Appendix. Each time rate data is uploaded to MIDAS, the previously uploaded rates under that Distribution or Energy Company are deleted from the Value table and only the newly uploaded values will be available to users. At the time the rate data is uploaded, it is also added to the HistoricalData table. See [Archiving Data to HistoricalData](#archived-data-in-the-historicaldata-table) above for more information. Please note, each RIN may only store a total of 50,000 values in the Value table.

**Endpoint:** `/ValueData`

**HTTP Request:** `POST https://midasapi.energy.ca.gov/api/ValueData`

**Authorization:** Bearer

#### Query Parameters

**None**

#### Python Example

```python
import os 
import sys
import requests

# File on the local filesystem that is correctly formatted against the XML schema definition (XSD)
priceFileName = 'XML_streaming_upload.xml'

headers = {'accept': 'application/json', 'Content-Type': 'text/xml', 'Authorization': "Bearer " + token}
url = 'https://midasapi.energy.ca.gov/api/valuedata'
priceFile = open(priceFileName)
xml = priceFile.read()
priceFile.close()
pricing_response = requests.post(url, data = xml, headers = headers)

print(pricing_response.text)
```

## Upload Holidays

Uploading the list of holidays for your LSE should only need to happen on occasion, perhaps once per year.

### POST Holiday Values

_This function is available to authorized LSE accounts only._ <br>
LSEs can upload the holidays that apply to their rates. This supports MIDAS users by making it clear which days are holidays on the rates they download. Populate the Holiday table by POSTing to the Holiday endpoint. The XML schema document is available as part of this documentation at [Holiday Upload XML Schema](support-docs/MIDAS-Holiday-XML-schema.xsd.xml).

**Endpoint:** `/Holiday`

**HTTP Request:** <br> `POST https://midasapi.energy.ca.gov/api/Holiday`

**Authorization:** Bearer

#### Query Parameters

**None**

#### Body Parameters

When uploading to the Holiday table, the body of the uploaded XML or JSON has the following fields:

| Name                              | Description                                              | Example | Type |
|-----------------------------------|----------------------------------------------------------|---------|------|
| EnergyCode <br> _required_        | Two letter code for LSE from the Energy lookup table     | "MC" | string |
| EnergyDescription <br> _required_ | Full name of the LSE from the Energy lookup table        | "Marin Clean Energy" | string |
| DateOfHoliday <br> _required_     | Date of holiday in local time. <br>For California PST format: "YYYY-MM-DDTHH:MM:SS-07:00" | "2023-12-25T00:00:00-07:00" | date   |
| HolidayDescription                | Full name of holiday                                     | "Christmas 2023" | string |

#### Response



**Python Example**

```python
import os 
import sys
import requests

holidayFileName = 'MIDAS_Test_Holidays.json'
headers = {'accept': 'application/json', 'Content-Type': 'text/json', 'Authorization': "Bearer " + token}
url = 'https://midasapi.energy.ca.gov/api/holiday'
holidayFile = open(holidayFileName)
holidays = holidayFile.read()
holidayFile.close()
print(holidays)
holiday_response = requests.post(url, data = holidays, headers = headers)

print(holiday_response.text)
```
