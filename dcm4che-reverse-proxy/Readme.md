_HTTP WORKING_
```bash
https://dcm4che.januo.io/dcm4chee-arc/aets/DCM4CHEE/rs/studies
weasis://$dicom:rs --url "http://dcm4che.januo.io/dcm4chee-arc/aets/DCM4CHEE/rs" -r "patientID=583295" --query-ext "&includedefaults=false"

```
_HTTPS NOT WORKING_

```bash
weasis://$dicom:rs --url "https://dcm4che.januo.io/dcm4chee-arc/aets/DCM4CHEE/rs" -r "patientID=583295" --query-ext "&includedefaults=false"
```
