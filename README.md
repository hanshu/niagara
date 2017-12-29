# Niagara
Niagara Framework





## How to debug
### Debugging Niagara AX in Eclipse
```
c:\niagara\niagara-3.8.111>wb -@Xrunjdwp:transport=dt_socket,address=8000,server=y,suspend=n  -profile:mhvrfAppliance:ApplianceProfile
```

### Niagara N4
```
c:\niagara\niagara-4.3.58.18>wb -@agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=5005  -profile:mhVrfConAX:ApplianceProfile
```

