# Design Guidelines to create a new ODA Component

ODA Components are self-contained, independently deployable software modules that confirm to the [TM Forum Open Digital Architecture](https://tmforum.org/oda).

The business drivers and conceptual model for ODA Components is described in [IG1171 ODA Component Definition](https://projects.tmforum.org/wiki/display/PUB/IG1171%20ODA%20Component%20Definition%20R19.0.1).


The ODA Component concept builds on top of open standards like Docker and Kubernetes, and adds Telco-domain knowledge and meta-data. The starting point for building an ODA component is containerized, micro-service software described in a kubernetes manifest YAML file. To turn this into an ODA Component we will:

1. Add meta-data to the Kubernetes manifest describing the *Core Function*, *Notification/Reporting*, *Security* and *Management & Operation* of the software component.
2. Add labels to all the standard Kubernetes resources to label them as belonging to the component.
3. Test the deployment of the Component in an operating platform that has the ODA Canvas installed (for example the TM FOrum Open-Digital Lab). See the [Getting Started](https://github.com/tmforum-rand/oda-component-definitions#get-started) section to see how to install the ODA Canvas onto a kubernetes cluster environmnet.


## Step 1: ODA-Component Metadata

The ODA-Component metadata contains all the Telco-domain knowledge that makes the component a self-describing deployable software module. From [IG1171 ODA Component Definition](https://projects.tmforum.org/wiki/display/PUB/IG1171%20ODA%20Component%20Definition%20R19.0.1), this meta-data describes the Open-APIs, event data schemas as well as security and management & operations for the component. 

The meta-data is defined using a Kubernetes [CustomResourceDefinition](https://kubernetes.io/docs/tasks/access-kubernetes-api/custom-resources/custom-resource-definitions/). This allows us to extend the Kubernetes API with our custom-defined schema for Telco meta-data. The CustomResourceDefinition schema is in the [github.com/tmforum-rand/oda-component-definitions](https://github.com/tmforum-rand/oda-component-definitions/tree/master/custom-resource-definitions) folder.

There is an example of the metadata for the productcatalog example component broken-down into sections below:

```
apiVersion: oda.tmforum.org/v1alpha2
kind: component
metadata:
  name: vodafone-productcatalog
  labels:
    oda.tmforum.org/componentName: vodafone-productcatalog
```

This is the header information for the component, specifying the version of the CRD (Custom Resource Definition) that it is using, and providing a name and a label for the component.

```
spec:
  selector:
    matchLabels:
     oda.tmforum.org/componentName: vodafone-productcatalog
  componentKinds:
    - group: core
      kind: Service
    - group: apps
      kind: Deployment  
  type: productcatalog  
  version: "0.0.1"
  description: "Vodafone simple reference implementation of ODA Product Catalog." 
  maintainers:
    - name: Lester Thomas
      email: lester.thomas@vodafone.com
  owners:
    - name: Lester Thomas
      email: lester.thomas@vodafone.com  
```

This next section starts the `spec` for the component. THe `selector` and `matchLabels` are used in the section 2 to *label the standard kubernetes resources as belonging to the component*.

The `type` allows us to specify that this is an implementation of a standard type of Component. The target is for the Open-Digital Architecture Reference Implementation to define a catalog of standard component types.

The `version` and `description`, `maintainers` and `owners` are self-descriptive.

```
  coreFunction:
    exposedAPIs: 
    - name: productCatalog
      specification: https://open-api.tmforum.org/TMF620-ProductCatalog-v4.0.0.swagger.json
      implementation: productcatalog
      path: /admin/productCatalogManagement/v2/catalog
      port: 8080
      scopes:
      - name: admin
      - name: regular
    - name: processFlow
      specification: https://open-api.tmforum.org/TMF701-ProcessFlow-v4.0.0.swagger.json
      implementation: camunda
      path: /camunda
      port: 8080
    dependentAPIs: 
    - name: party      
      specification: https://open-api.tmforum.org/TMF632-Party-v4.0.0.swagger.json
```

The `coreFunction` describes the core purpose of the software component. It describes the list of APIs that the component exposes as well as the APIs it is dependant on. The definitions within the `exposedAPIs` and `dependentAPIs` are experimental at this point, and we will modify and enhance them as we build-out the ODA Canvas and assemble a representative set of ODA Components. The current definition has:
* an `implementation` which links to the Kubernetes service that implements the API, including the `port` where the http service is exposed. 
* The `path` shows the API resource end-point, and can be used, for example, to automatically configure any API Gateway that is included as part of the Canvas. Note that the `path` points to the root of the API (and you need to append the relavant path from the swagger document to get to an implemented API resource). 
* The `specification` points to the swagger documentation for the API. The Component CTK (Compliance Test Kit) will look inside this swagger for the `basePath` determine which Open-API CTK to execute for that API. The `basePath` is of format `"/tmf-api/productCatalogManagement/v4/"` which shows it is a `tmf-api` for `productCatalogManagement` with major version `4`. The swagger can be one of the TM Forum published swagger files (e.g. [https://raw.githubusercontent.com/tmforum-rand/Open_API_And_Data_Model/v4.0-Sprint-2020-03/apis/TMF620_Product_Catalog/swaggers/TMF620-ProductCatalog-v4.1.0.swagger.json](https://raw.githubusercontent.com/tmforum-rand/Open_API_And_Data_Model/v4.0-Sprint-2020-03/apis/TMF620_Product_Catalog/swaggers/TMF620-ProductCatalog-v4.1.0.swagger.json?token=ACS2FQP4M3AEZBKQQEMIUQ3ABGIBM)) or can be an extension (conforming to the TMF630 Design Guidelines).

```
 eventNotification:
    publishedEvents:
    - name: Catalog
      href: https://schema.tmforum.org/Product/Catalog.schema.json   
    - name: Category
      href: https://schema.tmforum.org/Product/Category.schema.json
    - name: ProductOffering
      href: https://schema.tmforum.org/Product/ProductOffering.schema.json
    - name: ProductOfferingPrice
      href: https://schema.tmforum.org/Product/ProductOfferingPrice.schema.json
    - name: ProductSpecification
      href: https://schema.tmforum.org/Product/ProductSpecification.schema.json
    - name: Usage
      href: https://schema.tmforum.org/Product/Usage.schema.json
    subscribedEvents:
    - name: ImportJob
      href: https://schema.tmforum.org/Product/ImportJob.schema.json
```

The `eventNotification` describes the data events that the component publishes and sub-scribes to as part of its implementation. Again, the current definitions within these sections are experimental and we will modify and enhance them as we build-out the ODA Canvas and assemble a representative set of ODA Components.


```
  management:
  - name: serviceTest
    href: https://manager.local/healthCheck
    specification: https://open-api.tmforum.org/TMF653-ServiceTest-v4.0.0.swagger.json
  - name: alarm
    href: https://manager.local/alarm
    specification: https://open-api.tmforum.org/TMF642-Alarm-v4.0.0.swagger.json
  - name: serviceActivationConfiguration
    href: https://manager.local/serviceConfiguration
    specification: https://open-api.tmforum.org/TMF640-ServiceActivationConfiguration-v4.0.0.swagger.json
  security:
    controllerRole: secConAdmin
    securitySchemes:
      bearerAuth:
        type: http
        scheme: bearer
        bearerFormat: JWT
    partyrole:
      specification: https://raw.githubusercontent.com/tmforum-apis/TMF669_PartyRole/master/TMF669-PartyRole-v4.0.0.swagger.json
      implementation: r1-partyroleapi
      path: /r1-productcatalog/tmf-api/partyRoleManagement/v4
      developerUI: /r1-productcatalog/tmf-api/partyRoleManagement/v4/docs
      port: 8080  
```

The `management` and `security` sections describe management APIs the component exposes that are part of its management (rather than part of its business function). Examples of management APIs are for self-testing, raising operational alarms, or configuring the component itself. The `security` section provides meta-data on the security mechanisms used by the component. Again, the current definitions within these sections are experimental and we will modify and enhance them as we build-out the ODA Canvas and assemble a representative set of ODA Components. As of version `v1alpha2` the security definition should include a `partyrole` property that describes the TMF669 PartyRole Open-API that all components must support (future versions may support multiple mechanisms for components to expose the roles they support). As of version `v1alpha3` the security definition must include a `controllerRole` property that gives the name of a pre-existing role in the component that the security controller can use to a) POST a listener URL to the component partyRole notification endpoint so that it can receive notifications of events against party roles and b) GET the partyrole endpoint to query roles in the component.

## Step 2: Add labels to all the standard Kubernetes resources

By default, when you deploy a kubernetes manifest containing a number of different standard resources (Deployments, Pods, Services etc), they are deployed as independant resources with no reference to each-other. As part of the component standard, we add a label to every kubernetes resource to show that it belongs to a particular component.

```
spec:
  selector:
    matchLabels:
     oda.tmforum.org/componentName: vodafone-productcatalog
  componentKinds:
    - group: core
      kind: Service
    - group: apps
      kind: Deployment  
```

We saw earlier that the `spec` of the ODA Component included a `selector` and `componentKinds` sections: The `selector` defines the standard label that we will use throughout the manifest to label all the standard Kubernetes resources; The `componentKinds` shows the types of resources that this component includes.

In our productcatalog example component, if you look at the service definition below, it shows us adding the component label to this service.

```
apiVersion: v1
kind: Service
metadata:
  name: productcatalog
  labels:
    app: productcatalog
    oda.tmforum.org/componentName: vodafone-productcatalog
spec:
  ports:
  - port: 8080
    targetPort: productcatalog
    name: productcatalog
  type: NodePort
  selector:
    app: productcatalog
```

When this component is deployed, the [Component Operator](https://github.com/tmforum-rand/oda-component-definitions/tree/master/controllers/componentOperator) will use this information to build a parent relationship between the running instances of the component. This also means that when we delete a component using the `kubectl delete component <componentname>` comand, the Kubernetes garbage-collection will also delete all the standard resources within the component.
