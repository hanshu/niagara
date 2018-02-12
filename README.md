# Niagara
Niagara Framework

## How to debug
### Debugging Niagara AX in Eclipse
```
wb -@Xrunjdwp:transport=dt_socket,address=8000,server=y,suspend=n -profile:mhvrfAppliance:ApplianceProfile
```

### Niagara N4
```
wb -@agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=5005 -profile:mhVrfConAX:ApplianceProfile
```