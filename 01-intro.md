# 1. Intro
Cube is a universal semantic layer. Use it to organized the data from cloud data warehouses into centralized, consistent definitions, and deliver it downstream tools via its APIs. 

With Cube, you can:
- build data models
- manage access control, and caching
- expose data to applications via REST/GraphQL/SQL APIs. 
- then build custom UI, connect existing dashboards/reporting tools, etc

Everything within Cube, from configurations to data models, is managed through code. 

A complete, universal semantic layer should have 4 layers: data model, caching, access controls, APIs. 

Data model objects: 
1. cubes. Represent business entities. You define calculations in measures/dimensions of these entities. Also relationships between cubes. 
2. views. Sit on top of cubes, is the facade of your data model. Data consumer interact with views. When building views, you select measures and dimensions from different connected cubes, and present them as a single dataset to BI. 

Access control range from simple row-level access rules, to completely custom data models per tenants, backed by different data sources. 



















