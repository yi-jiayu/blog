---
title: "Singapore COVID-19 cases API"
date: 2020-03-31T22:23:56+08:00
---

The [MOH website](https://www.moh.gov.sg/covid-19) is the authoritative source
of information about the current COVID-19 situation in Singapore, but it's not
exactly machine-readable.

It doesn't seem like there's any official API either. For those trying to build
their own dashboard or crunch some data, it might look like you have no choice
but to do some data entry, scrape HTML or get data from a third-party source.

However, the MOH site also links to a more interactive [COVID-19 Situation
Dashboard](https://experience.arcgis.com/experience/7e30edc490a5441a874f9efe67bd8b89)
hosted on ArcGIS Online which is a JavaScript application that makes
requests to a JSON API:

{{< figure src="dashboard.png" alt="Singapore COVID-19 Situation Dashboard" >}}

After a bit of poking around, I was able to find this:

```
$ curl 'https://services6.arcgis.com/LZwBmoXba0zrRap7/arcgis/rest/services/COVID_19_Prod_feature/FeatureServer/0/query?f=json&where=1%3D1&returnGeometry=false&spatialRel=esriSpatialRelIntersects&outFields=*&orderByFields=Case_ID%20desc&resultOffset=0&resultRecordCount=1' | jq '.features[0].attributes'
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  6362  100  6362    0     0  22167      0 --:--:-- --:--:-- --:--:-- 22167
{
  "Case_ID": 879,
  "Case_Count": "Case 879",
  "Cluster": "0",
  "Current_Lo": "CGH",
  "PHI": "0",
  "Imported_o": "Local",
  "Place": "Singapore",
  "Age": 26,
  "Gender": "M",
  "Nationalit": "Singapore Citizen",
  "Status": "Hospitalised",
  "Date_of_Co": 1585526400000,
  "Date_of_Di": null,
  "Region_201": "0",
  "PlanningAr": "0",
  "POST_CODE": 0,
  "LONG": 0,
  "LAT": 0,
  "TOT_COUNT": 0,
  "UPD_AS_AT": "As of 30 March 2020, 1200h",
  "CASE_PENDG": 0,
  "CASE_NEGTV": 0,
  "Confirmed": 879,
  "DISCHARGE": 228,
  "DEATH": 3,
  "Suspct_Cas": 0,
  "Tot_NonICU": 401,
  "Tot_ICU": 19,
  "Tot_Impotd": 487,
  "Tot_local": 392,
  "PLAC_VISTD": "0",
  "RES_LOC": "0",
  "RESPOSTCOD": 228,
  "DT_CAS_TOT": 1585526400000,
  "CNFRM_CLR": "royalblue",
  "Case_total": 420,
  "Prs_rl_URL": "http://moh.gov.sg/news-highlights/details/16-more-cases-discharged-35-new-cases-of-covid-19-infection-confirmed",
  "ObjectId": 5752
}
```

Each element in the `features` array represents a COVID-19 case in Singapore. By
changing the value of the `resultRecordCount` query parameter, it's possible to
fetch data for all cases.

For some reason, every element also contains the latest value of key statistics
such as total, active, recovered, imported and local cases:

* Active cases: `Case_total`
* Hospitalised stable: `Tot_NonICU`
* Hospitalised critical: `Tot_ICU`
* Discharged to isolation: `RESPOSTCOD`
* Deaths: `DEATH`
* Discharged: `DISCHARGE`
* Total imported: `Tot_Impotd`
* Total local: `Tot_local`
* Confirmed cases: `Confirmed`

I wonder how the naming scheme came about.
