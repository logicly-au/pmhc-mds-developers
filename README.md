# PMHC-MDS Developers

This repository has been created as a public resource for developers working on supplying data to or otherwise integrating with the PMHC-MDS system.

## PMHC-MDS private API

The MDS is built around an (unpublished) API. The reports are rendered from an
intermediate JSON `dataPackage` based on the Frictionless Data specifications.
See https://frictionlessdata.io/ and their published specs.

The json data used to render the html report content can be fetched
automatically (with a little effort) for the reports. Using your browser
developer tools you can see the request to the API when you click
"Generate Report". The request will be similar to the following, which is for
report a1. The query params (roughly) match the form fields and may change
whenever a new release is made.

```
https://pmhc-mds.net/api/reports/a1?data_source=pmhc&level=R&option=PO&organisation_path=PHN204&start_date=2019-07-01&end_date=2020-06-30
```

To make requests against the MDS, you need an authentication token. Below is an
example of a short scipt to fetch an authentication token and use that in an
Authentication header for a subsequent request. Note that the authentication
service has a rate limit on the number of times you may authenticate in a 10
minute window, so you may wish to save the token rather then fetch a new token
for each request, also, authentication tokens are only valid for 15 minutes.


For this example, I'll put the credentials in a plain text file.
This is *not* secure. Use the secure tools your environment has available
for credential handling.

credentials.txt
```
user=value1&passwd=value2
```

api.sh
```
#!/bin/bash

auth-token() {
  curl -s -d @"$1" \
       -H "Content-Type: application/x-www-form-urlencoded" \
       -X POST "https://auth.strategicdata.com.au/login/?site=pmhcmds" \
  | grep login_token | sed 's/.*token=\(.\{36\}\).*/\1/'
}

token=$( auth-token "$1" )
curl -s -H "Authorization: Bearer ${token}" "${@:2}"
```

Call this as
```
./api.sh credentials.txt 'https://pmhc-mds.net/api/reports/a1?data_source=pmhc&level=R&option=PO&organisation_path=PHN204&start_date=2019-07-01&end_date=2020-06-30'
```

The json object returned will have `title`, `fields` and `data` keys. The
fields are ordered, permitting you to iterate over the elements of the data
to render the report.
