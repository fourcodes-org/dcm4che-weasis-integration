
## Reverse proxy implementation

```bash
docker build -t ui .
```

_http setting_
```console
https://dcm4che.januo.io/dcm4chee-arc/aets/DCM4CHEE/rs/studies
weasis://$dicom:rs --url "http://dcm4che.januo.io/dcm4chee-arc/aets/DCM4CHEE/rs" -r "patientID=583295" --query-ext "&includedefaults=false"

```
_https not working_

```console
weasis://$dicom:rs --url "https://dcm4che.januo.io/dcm4chee-arc/aets/DCM4CHEE/rs" -r "patientID=583295" --query-ext "&includedefaults=false"
```


![image](https://github.com/januo-org/dcm4che-weasis-integration/assets/57703276/af942700-c34b-4ba3-ab44-cc9b789799ca)

![image](https://github.com/januo-org/dcm4che-weasis-integration/assets/57703276/c1b9acaa-4f14-4b05-8dde-2c5fba11dda4)
