---
layout: post
title:  "How to Niagara debugging?"
---

### Debugging Niagara AX in Eclipse
```
wb -@Xrunjdwp:transport=dt_socket,address=8000,server=y,suspend=n 
```

### Niagara N4
```
wb -@agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=5005 
```

